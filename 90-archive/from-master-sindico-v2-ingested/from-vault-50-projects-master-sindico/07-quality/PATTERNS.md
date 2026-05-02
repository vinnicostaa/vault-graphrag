---
title: PATTERNS aplicados — Master Síndico
type: project
tags: [master-sindico, patterns, ddd, cqrs, saga]
project: master-sindico
created: 2026-04-22
---

# PATTERNS aplicados — Master Síndico

Patterns que usamos **aqui e agora**, com lugar no código e justificativa. Patterns universais em [[../../../10-knowledge/patterns/_moc]].

---

## DDD Tático

### Aggregate Root
- `Condominium` (institutional) — raiz; contém Units, Memberships
- `Assembly` (assembly) — raiz; contém AgendaItems, Votes
- `Subscription` (billing) — raiz; contém Seats, Invoices
- `User` (identity) — raiz; contém Sessions, DeviceFingerprints

**Regra**: operações em agregado passam pela raiz. Repositório só busca/salva a raiz inteira.

### Value Object
Ver [[CODE_CALISTHENICS#3-envolver-primitivos-e-strings-em-value-objects]].

### Domain Event
- `CondominiumCreated`, `UnitRegistered`, `MembershipAssigned`
- `SubscriptionActivated`, `TrialExpired`, `QuotaExhausted`
- `VideoApproved`, `VideoPublished`, `VideoRemoved`
- `AssemblyStarted`, `VoteCast`, `AssemblyClosed`
- Publicados fora da tx; consumidores decidem handle.

### Anti-Corruption Layer
Aplicado em adapters de SDK externo: `IStripeAdapter`, `IMuxAdapter`, `IZitadelAdapter`, `ILivekitAdapter`.
- Converte shapes externos → entidades do domínio
- Escude mudanças de API externa

### Specification Pattern (quando critério complexo)
Exemplo: `ListApprovedCompaniesSpec` compondo `IsApproved` + `HasReputationTier(min)` + `MatchesCondominium(tenant)`.

---

## CQRS

### Command
- Mudam estado.
- 1 arquivo por command: `create_company_use_case.go`
- Retorno: `(CommandResult, error)` onde `CommandResult` tem ID novo ou void.
- Input: struct `XCommand` (`CreateCompanyCommand`).

### Query
- Só leem.
- 1 arquivo por query: `list_companies_by_condominium_use_case.go`
- Input: struct `XQuery`.
- Retorno: DTO + erro.

### Read Model (quando necessário)
- Materialized view em PG para reputação ranking (agregações caras)
- OpenSearch index para busca full-text

---

## UoW (Unit of Work)

Implementação: `pkg/providers/database/unit_of_work.go` (pseudo).

```go
type UnitOfWork interface {
    Run(ctx context.Context, fn func(ctx context.Context) error) error
}

// Repositórios descobrem a tx ativa via DBFromContext:
func (r *Repo) Save(ctx context.Context, e Entity) error {
    db := database.DBFromContext(ctx, r.pool) // tx ou pool
    return r.queries.Upsert(ctx, db, ...)
}
```

**Usar quando**: fluxo é atomicamente uma transação (mesmo módulo, banco único).

**Exemplos aplicados**:
- CloseVoteSession (commercial) — `A-026` fix: tudo em `uow.Run(...)`
- Create Unit + Timeline append — atomicamente no institutional

---

## Saga (compensação)

**Usar quando**: fluxo cruza serviço externo com estado próprio (Mux, Livekit, Stripe).

### Estrutura típica
```
1. Ação no provider externo (CreateRoom no Livekit)
2. Ação local (Save session no PG)
3. Se (2) falhar → compensar (1): EndRoom no Livekit
4. Logar falha de compensação (não sobrescrever erro original)
```

### Aplicações
- `A-027` UploadVideo: `CreateUploadURL (Mux) → videoRepo.Create`. Falha em Create → `CancelUpload (Mux)`.
- `A-028` StartLiveSession: `CreateRoom (Livekit) → GenerateToken`. Falha em GenerateToken → `EndRoom (Livekit)`.

### Quando não é saga
Se ambos são locais: usar UoW.
Se externa mas sem estado persistente: idempotência resolve (retry cegamente é ok).

---

## Strategy

Aplicado em:
- **Trial duration por persona** (Síndico 15d, Empresa 7d, Parceiro 30d): map persona → duration
- **BeyondCorp middleware chain**: policies configuráveis por rota
- **Connect Me 4 fluxos**: strategy por tipo (vizinhança, academia, marketplace, contratação direta)

---

## Factory

- `NewX` + `ReconstructX` em entidades:
  - `NewUser(email, password)` valida + cria; usado no caso de uso de signup
  - `ReconstructUser(id, email, password, ...)` sem validação; usado no repo ao hidratar

---

## Repository

Interface no `domain/`; impl em `infrastructure/database/`. Sempre retorna entidade/agregado, nunca row do sqlc. Mapper obrigatório.

Exemplo:

```go
// domain/interfaces.go
type IUserRepository interface {
    FindByID(ctx context.Context, id UserID) (*User, error)
    Save(ctx context.Context, user *User) error
    InvalidatePreviousByUserID(ctx context.Context, userID UserID) error
}

// infrastructure/database/user_repository_impl.go
type UserRepository struct {
    pool *pgxpool.Pool
    q    *sqlc.Queries
}
func (r *UserRepository) FindByID(ctx context.Context, id UserID) (*User, error) {
    row, err := r.q.GetUserByID(ctx, r.db(ctx), id.UUID())
    if err != nil { return nil, mapErr(err) }
    return mappers.UserFromRow(row), nil
}
```

---

## Dependency Injection (manual)

Sem container mágico. `main.go` compõe explicitamente:

```go
// config
cfg := config.Load()

// providers
pool := database.MustPool(cfg)
redis := cache.MustRedis(cfg)
zitadel := auth.MustZitadel(cfg)
uow := database.NewUnitOfWork(pool)

// modules
identityMod := identity.NewModule(identity.Deps{Pool: pool, Redis: redis, Zitadel: zitadel, UoW: uow})
billingMod := billing.NewModule(billing.Deps{...})
...

// server
srv := server.New(cfg, []contracts.IModule{identityMod, billingMod, ...})
srv.Start()
```

Vantagens: compile-time correctness, debug trivial, sem reflection.

---

## ABAC Engine

`shared/middleware/abac_engine.go` implementa matriz:

```
admin_bypass → (SE user é admin global, permite tudo)
role_not_allowed → (role não habilitada para action)
action_not_configured → (action não existe no catálogo)
tenant_mismatch → (recurso pertence a outro tenant)
scope_restricted → (plano não inclui feature)
```

Cache Redis 5min; invalidação via webhook Stripe (quando subscription muda).

PBT: `abac_engine_pbt_test.go` cobre as regras acima (F32).

---

## Idempotency

- Webhooks Stripe: `event.id` é idempotency key; registrado em `stripe_webhook_events` com status.
- Webhooks Mux: `request_id` + `object.id` compõem chave.
- POST com `Idempotency-Key` header aceito em `/api/v1/connect-me` e outros endpoints sensíveis.

---

## Rate Limiting (token bucket)

`shared/middleware/rate_limiter.go`:
- `sync.Map` de key → bucket (key = IP + user_id + path group)
- Cleanup goroutine com `time.Ticker(30min)` + context-aware shutdown (A-032 fix F28)
- Separado por superfície: `/auth` (20 rpm, burst 5) + `/api/v1` (60 rpm, burst 10)

---

## Specification pattern (busca)

Quando busca tem múltiplos critérios opcionais:

```go
type CompanySpec struct {
    CondominiumID *CondominiumID  // optional
    MinReputation *ReputacaoTier
    ServiceType   *ServiceType
    Limit         int
    Cursor        string
}
```

Repo traduz spec em WHERE condicionais; evita N queries distintas.

---

## Anti-padrões conscientemente evitados

| Antipattern | Por quê evitamos |
|---|---|
| **Anemic domain model** | Entidades com comportamento (DDD)? sim; ricas |
| **God Object** | AP-014; quebrar em agregados |
| **Table Module** (tudo no service) | Lógica vai na entidade; service só orquestra |
| **Active Record** | Domain não conhece pgx; mapper explícito |
| **Smart UI** (lógica no handler) | Handler só orquestra; use case processa |

---

## Links

- [[_moc]]
- [[CLEAN_ARCH]]
- [[CLEAN_CODE]]
- [[CODE_CALISTHENICS]]
- [[../02-architecture/cqrs-layout]]
- [[../02-architecture/adapter-layer]]
- [[../../../10-knowledge/patterns/_moc]]
- [[../../../10-knowledge/patterns/transaction-patterns]]
