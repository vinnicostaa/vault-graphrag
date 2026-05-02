---
title: Transaction Patterns — ACID e Saga
type: pattern
tags:
  - pattern
  - transaction
  - saga
  - acid
  - concurrency
source: .kiro/steering/transaction-patterns.md
created: 2026-04-22T00:00:00.000Z
aliases:
  - Unit of Work
  - UoW
  - Saga Pattern
updated: '2026-04-24'
---

# Transaction Patterns — ACID e Saga

Duas ferramentas pra consistência. **ACID local** (UnitOfWork + driver nativo) resolve 80% dos casos. **Saga** resolve os 20% que cruzam bounded contexts e providers externos. **Usar a ferramenta errada** é origem de race condition, cobrança duplicada, estado inconsistente.

## Quando usar cada um (regra de ouro)

| Cenário | Padrão | Motivo |
|---|---|---|
| Operação em **1 banco** | **UnitOfWork / ACID** | Driver garante atomicidade real |
| Operação cruza **banco + cache** | UnitOfWork + eventual consistency | Cache não é source of truth |
| Operação cruza **bounded contexts no mesmo DB** | UoW se dentro de tx curta; **Saga** se houver chamada externa ou operação longa |
| Operação cruza **banco + provider externo** (Stripe, Mux, SES) | **Saga** | Provider externo não participa de tx local |
| Operação **longa** (> 5s) com múltiplos passos | **Saga** | Tx aberta longa causa lock + vacuum issues |
| Operação **eventual** (notificação, email, push) | Domain event + handler async | Não precisa atomicidade; falha não rollback |

## UnitOfWork (pattern padrão)

Toda operação que mexe em mais de uma tabela no mesmo banco usa `IUnitOfWork.Run()`.

### Propriedades garantidas

- **A**tomicity: callback retorna erro → rollback automático
- **C**onsistency: CHECK constraints + FKs + invariantes em código
- **I**solation: default `ReadCommitted` (suficiente pra 95% dos casos)
- **D**urability: WAL + fsync

### Exemplo canônico

```go
func (uc *CreateSubscriptionUseCase) Execute(ctx context.Context, cmd CreateSubscriptionCommand) (*SubscriptionDTO, error) {
    var output *SubscriptionDTO

    err := uc.unitOfWork.Run(ctx, func(ctx context.Context) error {
        // Passo 1: idempotency check
        existing, err := uc.subscriptionRepository.FindActiveByUserID(ctx, cmd.UserID)
        if err != nil && !errors.Is(err, apperrors.ErrNotFound) {
            return fmt.Errorf("FindActiveByUserID: %w", err)
        }
        if existing != nil {
            return apperrors.ErrConflict.WithMessage("user já tem subscription ativa")
        }

        // Passo 2: criar entidade
        subscription := entities.NewSubscription(cmd.UserID, cmd.PlanID, trialDays)

        // Passo 3: persistir
        if err := uc.subscriptionRepository.Save(ctx, subscription); err != nil {
            return fmt.Errorf("Save: %w", err)
        }

        // Passo 4: audit trail (append-only, mesma tx)
        if err := uc.auditRepository.Log(ctx, auditEntry); err != nil {
            return fmt.Errorf("audit Log: %w", err)
        }

        output = mappers.ToSubscriptionDTO(subscription)
        return nil
    })

    if err != nil {
        return nil, fmt.Errorf("CreateSubscription: %w", err)
    }
    return output, nil
}
```

Se qualquer passo falha: **toda a tx é revertida**. Banco nunca fica inconsistente.

### Limitações

**Não cobre**:
- Providers externos (Stripe, Mux, SES, Twilio, FCM) — fora do banco
- Operações que cruzam bancos
- Operações longas (tx aberta por 30s bloqueia vacuum, locks)
- Cross-bounded-context quando um lado dispara evento pra outro sistema

Pra esses casos: **Saga**.

## Saga Pattern

Saga decompõe uma transação de negócio em **passos locais**, cada um atômico, com **compensação** pra cada falha.

### Dois estilos

1. **Choreography** — cada passo emite evento; o próximo é listener do evento. Acoplamento mínimo, **difícil de debugar**.
2. **Orchestration** — um orchestrator (use case ou serviço dedicado) chama cada passo e coordena compensação. **Centralizado, claro**, preferido pra operações críticas (billing, checkout).

**Default recomendado: orchestration.** Choreography apenas pra reactive flows (ex: `SubscriptionActivated` → invalida cache + envia email — ambos idempotent, não precisam compensação).

### Anatomia de uma Saga orquestrada

```go
// Exemplo: checkout completo — Stripe customer + subscription + plan_tier + email

type SubscriptionCheckoutSaga struct {
    paymentGateway    IPaymentGateway
    subscriptionRepo  ISubscriptionRepository
    userRepo          IUserRepository
    emailProvider     IEmailProvider
    unitOfWork        IUnitOfWork
    logger            Logger
}

type checkoutState struct {
    stripeCustomerID     string
    stripeSubscriptionID string
    subscriptionID       string
}

func (s *SubscriptionCheckoutSaga) Execute(ctx context.Context, cmd CheckoutCommand) (*SubscriptionDTO, error) {
    state := &checkoutState{}

    // Passo 1: criar customer (idempotent via idempotency key)
    customer, err := s.paymentGateway.CreateCustomer(ctx, CreateCustomerInput{
        UserID: cmd.UserID,
        Email:  cmd.Email,
        IdempotencyKey: fmt.Sprintf("customer:%s", cmd.UserID),
    })
    if err != nil {
        return nil, fmt.Errorf("CreateCustomer: %w", err)
    }
    state.stripeCustomerID = customer.ID

    // Passo 2: criar subscription no provider
    stripeSub, err := s.paymentGateway.CreateSubscription(ctx, CreateSubscriptionInput{
        CustomerID:     customer.ID,
        PriceID:        cmd.PriceID,
        IdempotencyKey: fmt.Sprintf("subscription:%s:%s", cmd.UserID, cmd.PriceID),
    })
    if err != nil {
        // Compensação: customer idempotent, pode deixar
        return nil, fmt.Errorf("CreateSubscription: %w", err)
    }
    state.stripeSubscriptionID = stripeSub.ID

    // Passo 3: persistir localmente (ACID)
    err = s.unitOfWork.Run(ctx, func(ctx context.Context) error {
        sub := entities.NewSubscription(cmd.UserID, cmd.PlanID, stripeSub.TrialEnd)
        sub.LinkStripe(state.stripeCustomerID, state.stripeSubscriptionID)

        if err := s.subscriptionRepo.Save(ctx, sub); err != nil {
            return fmt.Errorf("Save subscription: %w", err)
        }
        if err := s.userRepo.UpdatePlanTier(ctx, cmd.UserID, cmd.PlanTier); err != nil {
            return fmt.Errorf("UpdatePlanTier: %w", err)
        }
        state.subscriptionID = sub.ID()
        return nil
    })
    if err != nil {
        // Compensação: cancelar subscription que acabamos de criar
        if cancelErr := s.paymentGateway.CancelSubscription(ctx, state.stripeSubscriptionID); cancelErr != nil {
            s.logger.Error().
                Err(cancelErr).
                Str("stripe_subscription_id", state.stripeSubscriptionID).
                Msg("compensação falhou — subscription órfã, reconciliar manualmente")
        }
        return nil, fmt.Errorf("persistir subscription: %w", err)
    }

    // Passo 4: email (fire-and-forget se falha)
    if err := s.emailProvider.SendWelcome(ctx, cmd.UserID, cmd.Email); err != nil {
        // NÃO compensa passos 1-3 — email é nice-to-have
        s.logger.Warn().Err(err).Str("user_id", cmd.UserID).Msg("email de boas-vindas falhou")
    }

    return &SubscriptionDTO{ID: state.subscriptionID}, nil
}
```

### Princípios de Saga bem-desenhada

1. **Cada passo é idempotent** — retry com mesma input produz mesmo resultado. Provider externo: `idempotency_key`. Banco: `INSERT ... ON CONFLICT DO UPDATE`. Email: `message_id` determinístico.
2. **Compensação explícita** — pra cada passo desfazível, definir operação inversa. `CreateSubscription` → `CancelSubscription`. `ChargeCard` → `Refund`. Alguns passos **não têm** compensação (email já enviado) — aceitar e documentar.
3. **Ordem importa** — passos **sem desfazer** vêm **depois** de passos que podem ser desfeitos. Email vai por último.
4. **Log tudo** — cada passo gera log estruturado com `saga_id` (correlação). Compensação falhada = alerta crítico (estado órfão em sistema externo).
5. **Estado intermediário persistido** (em Sagas longas) — salvar cada passo concluído em tabela `saga_state` com status. Permite retomar após crash.

### Saga simples reactive (choreography)

Quando passos são **naturalmente idempotent** e **recuperáveis por retry**:

```
1. User.UpdatePlan → emite SubscriptionActivated event
2. Listener A (audit): registra event em audit_trail (idempotent — key = event.ID)
3. Listener B (cache): invalida cache (idempotent — delete é idempotent)
4. Listener C (email): envia welcome (idempotent — message_id baseado em event.ID)
```

Se listener B falha: retry até sucesso (cache não é source of truth; miss não bloqueia). Sem compensação necessária.

## Árvore de decisão

```
Operação toca apenas 1 banco?
├── Sim → UnitOfWork (ACID local)
└── Não → toca provider externo / outro sistema?
    ├── Sim → Saga orchestrated (com compensação)
    └── Não (toca cache) → UnitOfWork + eventual consistency

Operação é longa (> 5s esperados)?
├── Sim → Saga com state persistido
└── Não → UnitOfWork

Falha no meio pode deixar estado inconsistente em sistema externo?
├── Sim → Saga com compensação explícita + alerta em falha de compensação
└── Não → UnitOfWork
```

## Domain Events + Eventual Consistency

Operações **sem** consistência forte → emitir domain event + handler async.

### Quando

- Notificações (email, push, SMS)
- Invalidação de cache
- Sync pra sistema de busca (OpenSearch, Elastic)
- Atualização de métricas / agregados não-críticos
- Webhooks de saída (notificar sistema parceiro)

### Princípios

- **Handler é idempotent** — event pode chegar 2× (retry, restart)
- **Falha de handler não bloqueia fluxo principal** — log + retry com backoff
- **Event ID único e estável** — idempotency key natural

## Isolation Levels

| Level | Quando |
|---|---|
| `ReadCommitted` (default) | 95% dos casos. Sem dirty reads. |
| `RepeatableRead` | Relatórios longos sem non-repeatable read |
| `Serializable` | Raro. Operações onde write skew é inaceitável (ex: quotas concorrentes). |

**Preferência**: `ReadCommitted` + **lock explícito** onde preciso (`SELECT ... FOR UPDATE`) em vez de subir isolation — menor contenção.

## Locks Explícitos

```go
err := uc.unitOfWork.Run(ctx, func(ctx context.Context) error {
    quota, err := uc.quotaRepository.FindByUserIDForUpdate(ctx, cmd.UserID)
    if err != nil {
        return err
    }
    if !quota.HasRemaining() {
        return apperrors.ErrBusiness.WithMessage("quota esgotada")
    }
    quota.Consume()
    return uc.quotaRepository.Save(ctx, quota)
})
```

```sql
-- name: FindQuotaByUserIDForUpdate :one
SELECT * FROM feature_usages
WHERE user_id = $1 AND feature_key = $2 AND period = $3
FOR UPDATE;
```

**Regra**: `FOR UPDATE` **só** em transação. Fora de tx, lock é liberado imediatamente (sem efeito).

### Advisory Locks (sem linha específica)

Mutex lógico (ex: "apenas 1 instância roda esse job"):

```sql
SELECT pg_try_advisory_lock(12345);
-- ... trabalho
SELECT pg_advisory_unlock(12345);
```

Útil pra migrations (goose já faz), jobs singleton, inicialização.

## Referências

- Chris Richardson. *Microservices Patterns* (2018) — capítulo 4 Saga
- Clemens Vasters. [Sagas (Microsoft)](https://learn.microsoft.com/en-us/azure/architecture/reference-architectures/saga/saga)
- Postgres: [Transaction Isolation Docs](https://www.postgresql.org/docs/current/transaction-iso.html)
- pgx: `BeginTxFunc` com `pgx.TxOptions`

## Links

- [[_moc]]
- [[../database/database-patterns]] — indexação, soft delete, upsert
- [[../../20-stacks/postgres/postgresql-conventions]] — pgx/pgxpool details
- [[../runtime-antipatterns/_moc]] — OP-001 (race), OP-003 (idempotência), OP-004 (retry)
- [[../security/security-principles]] — idempotency keys em escrita externa
