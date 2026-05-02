---
title: CQRS — Command Query Responsibility Segregation
type: concept
tags: [architecture, cqrs, command, query, design-pattern, ddd]
source: Greg Young (2010), Bertrand Meyer CQS (1988)
created: 2026-04-23
aliases: [CQRS, Command Query Responsibility Segregation]
---

# CQRS — Command Query Responsibility Segregation

**CQRS** separa os modelos de **escrita** (Commands) e **leitura** (Queries). Cunhado por Greg Young (2010) como evolução do CQS (*Command-Query Separation*, Bertrand Meyer, 1988). Aplicável em vários níveis — desde convenção de métodos até arquiteturas com bancos separados para read/write.

> **Espírito**: **ler e escrever têm requisitos diferentes**. Forçar o mesmo modelo para ambos gera compromissos ruins em ambos os lados.

## CQS — o nível mínimo (sempre aplicar)

Bertrand Meyer:
- **Command**: muda estado, **não retorna** (ou retorna só void/error)
- **Query**: retorna dado, **nenhum efeito colateral**

```go
// ❌ Mistura — faz ambos e confunde
func (s *SubscriptionService) ActivateAndGet(id string) (*Subscription, error) { ... }

// ✅ Separados
func (s *SubscriptionService) Activate(id string) error { ... }           // command
func (s *SubscriptionService) GetByID(id string) (*Subscription, error) { // query
    // nunca muta estado
}
```

Método que é **query** nunca pode mutar (LSP — [[solid#3-lsp]]). Chamador espera idempotência de leitura.

## CQRS — o nível arquitetural

Vai além de método: separa **modelos** inteiros.

- **Command model**: otimizado para **consistência**, invariantes, transações. É o modelo DDD com aggregates (ver [[ddd-strategic-tactical]]).
- **Query model**: otimizado para **leitura**, com shapes que a UI/consumer precisa. Pode ser desnormalizado, materializado, cacheado.

```
┌──────────────┐     Write path
│  Command     │───▶ Aggregate.Method() ───▶ Repo.Save() ───▶ DB normalizado
│  (escrita)   │        │
└──────────────┘        │
                    ┌───▼────┐
                    │ Domain │
                    │ Event  │  (emite após commit)
                    └───┬────┘
                        │
                        ▼
                ┌───────────────┐   Read path
                │ Projector     │───▶ DB desnormalizado
                │ (projeta pra  │    ou read-model materialized
                │  view)        │    ou cache
                └───────────────┘
                        │
                        ▼
                ┌───────────────┐
                │ Query         │◀─── endpoint direto
                │ (leitura)     │      skip domain layer
                └───────────────┘
```

### Nível 1 — CQRS leve (o comum)

**Mesmo banco**, **classes separadas**:

- `<Action>Command` / `<Action>CommandHandler` / `<Action>UseCase` — escrita via aggregate
- `<Name>Query` / `<Name>QueryHandler` — leitura pode fazer SQL direto, skip do aggregate

```go
// Write — usa aggregate, invariantes protegidas
type ActivateSubscriptionCommand struct { SubscriptionID string }
type ActivateSubscriptionUseCase struct {
    repo ISubscriptionRepository
    uow  IUnitOfWork
}
func (uc *ActivateSubscriptionUseCase) Execute(ctx context.Context, cmd ActivateSubscriptionCommand) error {
    return uc.uow.Run(ctx, func(ctx context.Context) error {
        sub, _ := uc.repo.FindByID(ctx, cmd.SubscriptionID)
        if err := sub.Activate(); err != nil { return err }
        return uc.repo.Save(ctx, sub)
    })
}

// Read — SQL direto, DTO simples, sem carregar aggregate inteiro
type GetSubscriptionDashboardQuery struct { UserID string }
type SubscriptionDashboardDTO struct {
    PlanName     string
    Status       string
    DaysLeft     int
    UsedQuota    int
    TotalQuota   int
}
type GetSubscriptionDashboardHandler struct { db *pgxpool.Pool }
func (h *GetSubscriptionDashboardHandler) Execute(ctx context.Context, q GetSubscriptionDashboardQuery) (*SubscriptionDashboardDTO, error) {
    var dto SubscriptionDashboardDTO
    err := h.db.QueryRow(ctx, `
        SELECT p.name, s.status, ...
        FROM subscriptions s
        JOIN plans p ON p.id = s.plan_id
        WHERE s.user_id = $1
    `, q.UserID).Scan(&dto.PlanName, &dto.Status, ...)
    return &dto, err
}
```

**Benefícios**:
- Query não paga custo de hidratar aggregate inteiro
- Query pode fazer JOIN/aggregation/projection específica da view
- Write path permanece puro DDD
- **Testável**: commands testam via aggregate mock; queries testam via DB real (testcontainers)

### Nível 2 — CQRS com read model materializado

Read model é **tabela/view separada** populada por projectors (handlers de eventos). Útil quando:
- Read model combina dados de **múltiplos aggregates/bounded contexts**
- Query seria lenta contra tabelas normalizadas
- Read model precisa de pré-cálculos (contadores, agregações)

```
CommandHandler ─▶ aggregate save ─▶ DomainEvent ─▶ Projector ─▶ read_model table
                                                                        ▲
                                                                        │
QueryHandler ─────────────────────────────────────────────────────────────
```

Trade-off: **consistência eventual** entre write e read model (janela de ms a segundos). OK para dashboards, **não** OK para verificações de invariante (essas ficam no write path).

### Nível 3 — CQRS com bancos separados + Event Sourcing

Write DB (append-only eventos) ≠ Read DB (projeções materializadas). Greg Young original.

**Não aplicar** a menos que:
- Audit trail legal é requisito (LGPD, SOX, compliance)
- Regressão de estado ("o que aconteceu nesse dia?") é recorrente
- Time tem maturidade para lidar com eventual consistency como default

Trade-off: complexidade enorme, **replays** de eventos, versionamento, schema evolution. Maioria de projetos nunca precisa.

## Quando aplicar cada nível

| Sintoma | Nível recomendado |
|---|---|
| "Toda task CRUD" | CQS (command/query separados por convenção) |
| "Tela de dashboard lenta por carregar aggregate inteiro" | Nível 1 — SQL direto no query path |
| "Read combina muitos domínios ou precisa de rollup" | Nível 2 — read model materializado |
| "Preciso audit trail completo" ou "replay de estado" | Nível 3 — Event Sourcing + CQRS |

Erro mais comum: **começar no Nível 3**. Complexidade explode. Começar no CQS e **escalar conforme sintoma** aparece.

## CQRS sobre banco único — padrão mínimo

Estrutura canônica de diretório:

```
application/use_cases/
├── <action>_command.go       # write: mutações via aggregate
├── <action>_use_case.go      # idem — nome depende da convenção
├── <query>_query.go          # read: SQL direto, DTOs finos
└── <query>_query_handler.go
```

ou, mais granular por CQRS rígido:

```
application/
├── commands/
│   ├── activate_subscription/
│   │   ├── command.go
│   │   ├── handler.go
│   │   └── handler_test.go
└── queries/
    ├── get_subscription_dashboard/
    │   ├── query.go
    │   ├── handler.go
    │   └── handler_test.go
```

## Commands — regras

- **Um command = uma intenção de negócio**: `ActivateSubscription`, `CancelAssembly`, `RegisterVote` — nunca `UpdateSubscription` genérico
- **Imutável** após criação
- Validação **estrutural** na construção; **de invariante** dentro do handler (via aggregate)
- Retorno: `error` + opcionalmente ID gerado (se command cria novo aggregate)
- **Idempotency key** quando expõe via HTTP (header `Idempotency-Key`)

## Queries — regras

- **Retornam DTO**, nunca entidade de domínio
- **Zero efeito colateral** (não escreve audit log, não incrementa contador) — exceção: cache de leitura (OK, é observabilidade)
- Podem fazer SQL direto sem passar pelo aggregate (performance)
- **Paginação sempre** em lista (ver [[../database/database-patterns]])
- Versão mutável é proibida: `<Get|List|Search|Find|Find...>` como prefixo

## CQRS + Domain Events

Evento de domínio é **o lugar natural** para manter read model atualizado:

```go
// command path
func (uc *CancelSubscriptionUseCase) Execute(ctx context.Context, cmd Cmd) error {
    return uc.uow.Run(ctx, func(ctx context.Context) error {
        sub, _ := uc.repo.FindByID(ctx, cmd.ID)
        sub.Cancel(cmd.Reason)
        uc.repo.Save(ctx, sub)
        uc.events.Publish(SubscriptionCanceled{SubscriptionID: sub.ID(), CanceledAt: now})
        return nil
    })
}

// projector (handler do evento)
func (p *DashboardProjector) Handle(ctx context.Context, e SubscriptionCanceled) error {
    return p.db.Exec(ctx, `
        UPDATE dashboard_subscription_view SET status='canceled', canceled_at=$1
        WHERE subscription_id=$2
    `, e.CanceledAt, e.SubscriptionID)
}
```

## CQRS + CAP theorem

Leitura e escrita têm **trade-offs diferentes** sob CAP:
- **Write** prioriza consistência (CP) — perder escrita é inaceitável
- **Read** pode priorizar disponibilidade (AP) — staleness de 5s geralmente OK

Read model com réplica read-only: **leitura continua** durante manutenção do write. Write faz failover controlado.

## Anti-patterns

- **Command que retorna DTO** ("e já me dá o state novo"): vira CQS-confuso. Separa: command → ok; client faz query depois.
- **Query que incrementa contador** (ex: "view count"): é command disfarçado. Torna operação não-idempotente silenciosamente.
- **CRUD mascarado de CQRS**: `UpdateUserCommand { ... 20 campos opcionais ... }` — é update form, não command. Quebra em `ChangeUserEmail`, `UpdateUserPhone`, `PromoteUserToAdmin`.
- **CQRS sem eventos quando aparece read model materializado**: projection fica com estado divergente.
- **Read model eventual consistente usado em write path**: viola invariante ("verificar cota no dashboard DTO antes de autorizar consumo").

## Checklist de review

- [ ] Cada use case é claramente **command** OU **query**?
- [ ] Nenhum método marcado "query" muta estado?
- [ ] Nenhum command retorna entidade carregada (só `error` ou ID novo)?
- [ ] Queries retornam DTOs, nunca entidades?
- [ ] Commands têm nome de **intenção de negócio** (não genéricos `UpdateX`)?
- [ ] Read model materializado (se existe) é alimentado **só por eventos**?

## Referências

- Bertrand Meyer. *Object-Oriented Software Construction* (1988) — CQS original
- Greg Young. *CQRS Documents* (2010) — https://cqrs.wordpress.com/
- Martin Fowler. [CQRS](https://martinfowler.com/bliki/CQRS.html) — visão pragmática
- Vaughn Vernon. *Implementing DDD* (2013) — capítulo CQRS + Event Sourcing

## Links

- [[_moc]]
- [[clean-architecture-layers]]
- [[ddd-strategic-tactical]]
- [[../patterns/transaction-patterns]]
- [[../principles/solid]]
- [[../principles/clean-code]]
- [[../database/database-patterns]]
