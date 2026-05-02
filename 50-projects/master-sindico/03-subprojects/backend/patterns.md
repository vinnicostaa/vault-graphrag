---
title: Backend — Patterns Go (destilação Fase 8, código real)
type: guide
tags: [backend, patterns, go, ddd, saga, uow, master-sindico, v2, destilacao-fase-8]
source: /home/vinni/Documentos/Projetos/Clientes/Joao/MasterSindico/Development/backend
created: 2026-04-23
updated: 2026-04-23
aliases: [backend-patterns, go-patterns]
---

# Backend — Patterns aplicados (código real)

Cada seção lista o **padrão**, **onde aparece no código**, **trechos literais
preservados**, e **quando NÃO usar** (as exceções são o que evita culto-cargo).

---

## 1. Fontes consultadas (2026-04-23)

| Path | Padrão exemplificado |
|---|---|
| `backend/AGENTS.md:82-92` | Princípios inegociáveis |
| `internal/modules/billing/application/use_cases/checkout_subscription_saga.go:30-221` | Saga com compensação |
| `internal/modules/billing/domain/entities/subscription.go:1-213` | Aggregate + state machine + sentinels |
| `internal/modules/billing/domain/services/quota_limits.go:1-124` | Domain service declarativo |
| `internal/modules/billing/domain/entities/feature_usage.go:1-119` | Idempotência + error-as-value |
| `internal/modules/institutional/domain/entities/unit.go:1-84` | Factory + Reconstruct |
| `internal/modules/institutional/domain/entities/condominium.go:1-123` | VO + regex + sentinels |
| `internal/modules/assembly/domain/entities/assembly.go:132-198` | State machine com guards |
| `internal/modules/assembly/domain/entities/agenda_item.go:133-147` | Fração ideal (PBT 100 casos) |
| `internal/modules/commercial/application/use_cases/vote_session_use_cases.go:102-123` | UoW + SELECT FOR UPDATE |
| `internal/shared/middleware/abac_engine.go` + `abac_rules.go:1-789` | ABAC declarativo |
| `internal/shared/types/auth_user.go:81-90` | Single source of truth `PlanTierOrder` |
| `pkg/safecast/` | Clamp int32/int16 para G115 |

---

## 2. DDD tático — Aggregate com estado privado

**Padrão canônico aplicado em todos os 38 agregados**:

```go
// internal/modules/institutional/domain/entities/condominium.go:29-42
type Condominium struct {
    id              string
    publicID        string
    name            string
    condoType       enums.CondominiumType
    address         value_objects.Address
    totalUnits      int
    blocksOrTowers  int
    createdBy       string
    createdAt       time.Time
    updatedAt       time.Time
    deletedAt       *time.Time
}
```

Invariantes:
- Campos são **todos lowercase** (privados ao package).
- Getters explícitos (`ID()`, `Name()`, ...) — nenhum `Set*(v)` público.
- **Duas factories**:
  - `NewCondominium(...)` — construtor canônico, **valida todas invariantes** antes de retornar. Usado por use cases de criação.
  - `ReconstructCondominium(...)` — hidratação a partir do banco, **sem revalidação** (dados já passaram pelos CHECKs do PG). Usado **exclusivamente** por repositories via `mappers.go`.

**Exemplo com validação**:

```go
// condominium.go:46-81
func NewCondominium(id, publicID, name string, /* ... */) (*Condominium, error) {
    if !publicIDPattern.MatchString(publicID) { return nil, ErrInvalidPublicID }
    if l := len(strings.TrimSpace(name)); l < 2 || l > 200 { return nil, ErrInvalidCondominiumName }
    if !condoType.IsValid() { return nil, ErrInvalidCondominiumType }
    if totalUnits < 1 || totalUnits > 10000 { return nil, ErrInvalidUnitCount }
    if blocksOrTowers < 0 || blocksOrTowers > 500 { return nil, ErrInvalidBlocksCount }
    // ... retorna agregado válido
}
```

---

## 3. State machines explícitas em entidades

`Subscription` tem 7 estados definidos em enum + métodos atômicos:

```go
// billing/domain/entities/subscription.go:11-26, 132-213
type SubscriptionStatus string

const (
    SubscriptionStatusTrialing          SubscriptionStatus = "trialing"
    SubscriptionStatusActive            SubscriptionStatus = "active"
    SubscriptionStatusPastDue           SubscriptionStatus = "past_due"
    SubscriptionStatusCanceled          SubscriptionStatus = "canceled"
    SubscriptionStatusPaused            SubscriptionStatus = "paused"
    SubscriptionStatusIncomplete        SubscriptionStatus = "incomplete"
    SubscriptionStatusIncompleteExpired SubscriptionStatus = "incomplete_expired"
)

func (s SubscriptionStatus) IsActiveStatus() bool {
    return s == SubscriptionStatusActive || s == SubscriptionStatusTrialing
}

// IsActive: corrige trial retroativo — Sprint 7.6 bug 2
func (s *Subscription) IsActive() bool {
    if s.status == SubscriptionStatusTrialing && !s.IsInTrial() {
        return false  // trialing com trialEndsAt no passado = bloqueia
    }
    return s.status.IsActiveStatus()
}

// MarkCanceled: transição não-idempotente que retorna ErrSubscriptionAlreadyCanceled
func (s *Subscription) MarkCanceled(reason string) error {
    if s.status == SubscriptionStatusCanceled {
        return ErrSubscriptionAlreadyCanceled
    }
    // ...
}
```

`Assembly` idem com 5 estados (`rascunho → publicada → em_andamento →
encerrada` / `cancelada`) + 8 sentinels de erro + guards:

```go
// assembly/domain/entities/assembly.go:146-161
func (a *Assembly) Start() error {
    if a.status == enums.AssemblyStatusEncerrada { return ErrAssemblyAlreadyEnded }
    if a.status != enums.AssemblyStatusPublicada { return ErrAssemblyNotPublished }
    if !a.IsEditalPublished() { return ErrAssemblyEditalRequired }
    now := time.Now()
    a.status = enums.AssemblyStatusEmAndamento
    a.startedAt = &now
    a.updatedAt = now
    return nil
}
```

---

## 4. Value Objects imutáveis com validação no construtor

`CNPJ`, `CPF`, `Email`, `Address`, `BimonthlyPeriod`.

```go
// commercial/domain/value_objects/cnpj.go (resumo)
type CNPJ struct { digits string }

func NewCNPJ(raw string) (*CNPJ, error) {
    cleaned := stripNonDigits(raw)
    if len(cleaned) != 14 { return nil, ErrInvalidCNPJLength }
    if !validateCNPJCheckDigits(cleaned) { return nil, ErrInvalidCNPJCheckDigit }
    return &CNPJ{digits: cleaned}, nil
}

func (c *CNPJ) Digits() string { return c.digits }

// Masked — LGPD: CNPJ identifica pessoa jurídica, nunca logar em claro (A-004).
func (c *CNPJ) Masked() string { return "**.***.***/****-**" }
```

Uso obrigatório em `logger.F("cnpj", cnpj.Masked())` — nunca `.Digits()`.

---

## 5. Error-as-value com sentinels exportados

Camada domain **nunca** retorna `errors.New` anônimo no caller path crítico —
use sentinels que permitem `errors.Is`:

```go
// billing/domain/entities/feature_usage.go:16
var ErrQuotaExhausted = errors.New("quota exhausted for current period")

// use case
if err := usage.Consume(limit); err != nil {
    if errors.Is(err, entities.ErrQuotaExhausted) {
        return nil, apperrors.ErrQuotaExhausted.WithMessage("limite mensal atingido")
    }
    return nil, fmt.Errorf("consume: %w", err)
}
```

**AppError** (no `internal/core/errors/errors.go`) é o tipo serializado para
HTTP. Sentinels: `ErrNotFound`, `ErrForbidden`, `ErrConflict`,
`ErrValidation`, `ErrInternal`, `ErrQuotaExhausted`.

---

## 6. Saga com compensação (inter-módulo + provider externo)

Canônico em `billing/application/use_cases/checkout_subscription_saga.go`.

Ciclo de 6 passos:

```
1. validate(cmd)
2. plan := planRepo.FindByID(planID)            # read puro — sem compensação
3. customer := gateway.CreateCustomer(idemKey)  # lado externo idempotente
4. subStripe := gateway.CreateSubscription(idemKey)
     # falha aqui = customer persiste no Stripe (reusável)
5. unitOfWork.Run(ctx, func(ctx):
      local := NewSubscription(...).LinkStripe(customer, subStripe, priceID)
      subscriptionRepo.Save(ctx, local)
      # falha = gateway.CancelSubscription(AtPeriodEnd=false) imediato
   )
6. eventPublisher.Publish(SubscriptionActivated)
     # falha = log warn; evento pode ser reemitido via worker outbox
```

**Regra operacional** (`steering/transaction-patterns.md`):
- **Intra-DB** → UoW sozinho basta.
- **Cross-DB + provider externo** → Saga + idempotency key no provider + compensação explícita.
- Passos não-compensáveis (ex: email enviado, push) vão **por último** no fluxo.

Outros Sagas em produção:
- `content.UploadVideoUseCase`: `CreateUploadURL` + `videoRepo.Create`; compensa com `CancelUpload(uploadID)` (A-027).
- `assembly.StartLiveSessionUseCase`: `CreateRoom` + `GenerateToken`; compensa com `EndRoom(roomName)` (A-028).

---

## 7. Unit of Work propagado por `context.Context`

Interface no core, impl em `shared/providers/database/unit_of_work.go`.

**Uso em use case** (atomicidade dentro do DB):

```go
// commercial/application/use_cases/vote_session_use_cases.go:108
err := uc.unitOfWork.Run(ctx, func(ctx context.Context) error {
    sess, err := uc.voteRepo.FindSessionByIDForUpdate(ctx, cmd.SessionID)
    if err != nil { return err }
    if sess.Status.IsTerminal() {
        return apperrors.ErrConflict.WithMessage("sessão encerrada")
    }
    if err := uc.voteRepo.CastVote(ctx, newVote); err != nil {
        if errors.Is(err, apperrors.ErrConflict) {
            return err       // UNIQUE violation = voto duplicado (A-026/A-029)
        }
        return err
    }
    return uc.voteRepo.IncrementVotesFor(ctx, sess.ID)
})
```

**Uso em repositório** (extrai `tx` ou usa pool):

```go
// padrão replicado em todos os *_repository_impl.go
func (r *VoteRepository) CastVote(ctx context.Context, v *entities.Vote) error {
    db := database.DBFromContext(ctx, r.pool)   // tx se dentro de UoW, pool fora
    q := generated.New(db)
    _, err := q.InsertVote(ctx, generated.InsertVoteParams{...})
    if err != nil {
        if isUniqueViolation(err) {
            return apperrors.ErrConflict.WithMessage("votante já registrou voto")
        }
        return fmt.Errorf("insert vote: %w", err)
    }
    return nil
}
```

---

## 8. Sem `else` — Code Calisthenics Go-flavor

Todo `else` block foi eliminado. Padrões substitutos:

### 8.1 Early return / guard clauses

```go
// identity/application/use_cases/sync_user_use_case.go
if err := validateInput(cmd); err != nil {
    return nil, err
}
user, err := uc.repo.FindByZitadelID(ctx, cmd.ZitadelID)
if err != nil && !errors.Is(err, apperrors.ErrNotFound) {
    return nil, err
}
if user == nil {
    user = entities.NewUser(...)
}
// segue sem else
```

### 8.2 Strategy via map/dispatcher

```go
// institutional/application/use_cases/register_membership_use_case.go (refatoração F4)
type membershipStrategy func(ctx, cmd) error

var strategies = map[string]membershipStrategy{
    "titular":   registerTitularStrategy,
    "dependent": registerDependentStrategy,
}

if fn, ok := strategies[cmd.RoleType]; ok {
    return fn(ctx, cmd)
}
return apperrors.ErrValidation.WithMessage("role_type inválido")
```

### 8.3 Default + override (em vez de if/else)

```go
status := entities.SubscriptionStatusActive
if trialEndsAt != nil && trialEndsAt.After(now) {
    status = entities.SubscriptionStatusTrialing
}
```

---

## 9. Domain Service declarativo (matriz de regras)

Quando regra é **muitos para muitos** e pode mudar por SKU/sprint:
matriz imutável construída no startup.

```go
// billing/domain/services/quota_limits.go:56-86
func defaultQuotaMatrix() QuotaMatrix {
    return QuotaMatrix{
        FeatureConnectMe: {
            "resident": { "base": 2, "plus": 4 },
            "syndic":   { "trial": 2, "base": 2, "plus": 4, "pro": entities.QuotaUnlimited },
        },
        FeatureVideoUpload: {
            "syndic":     { "base": 4, "plus": 8, "pro": 12 },
            "enterprise": { "plus": 8, "pro": 12 },
        },
    }
}

// LimitFor retorna 0 quando combinação não mapeada — default deny.
func (q *QuotaLimits) LimitFor(feature FeatureKey, role, tier string) int64 {
    roles, ok := q.matrix[feature]
    if !ok { return 0 }
    tiers, ok := roles[role]
    if !ok { return 0 }
    return tiers[tier]
}
```

**Por que não fica em config/env**: matriz está ligada à lógica de domínio
(ex: `QuotaUnlimited = -1` é sentinel). Constante em código = auditável em
PR + versionada com o resto.

**Mesmo padrão** aplicado em `abac_engine.DefaultMatrix()` (70+ actions) e
`TrialRules.defaultMatrix()`.

---

## 10. ABAC declarativo — matriz imutável

`internal/shared/middleware/abac_rules.go` (789 linhas) define uma única função `DefaultMatrix()`:

```go
return Matrix{
    "institutional.timeline.create": {
        {
            Name:               "syndic_creates_timeline_entry",
            AllowedRoles:       []string{"syndic"},
            AllowedRoleTypes:   []string{"principal", "auxiliary"},
            MinPlanTier:        "base",
            RequireTenantMatch: true,
        },
    },
    "commercial.connect_me.create": {
        { Name: "syndic_initiates_connect_me",     AllowedRoles: []string{"syndic"} },
        { Name: "resident_initiates_connect_me",   AllowedRoles: []string{"resident"} },
        { Name: "enterprise_initiates_connect_me", AllowedRoles: []string{"enterprise"}, MinPlanTier: "plus" },
    },
    // ...
}
```

Regras:
- **Dentro** de uma Rule: AND (todas condições não-vazias precisam bater).
- **Entre** Rules (mesma action): OR (qualquer uma concede).
- `role=="admin"` → bypass global (hardcoded no engine, não listado na matriz).
- `MinPlanTier` usa `types.PlanTierOrder` (única hierarquia oficial).
- `RequireTenantMatch=true` + `ResourceTenantID ∈ AuthUser.CondominiumIDs`.

Engine abort se matriz vazia (`ErrEmptyMatrix`) — fail-closed, evita A1 do legado.

---

## 11. Repository + sqlc + mapping row→entity

```
domain/repositories/i_vote_repository.go      # interface (apenas assinaturas)
    ↓
infrastructure/database/
  queries/vote.sql                             # sqlc input
  generated/vote.sql.go                        # sqlc output (NÃO editar)
  vote_repository_impl.go                      # adapter: pgx → entity
  mappers.go                                   # helpers row→entity (ex: numeric→string)
```

**Sobre `SELECT *`**: proibido em produção (A-021 corrigido — 7 queries commercial trocadas para colunas explícitas). `RETURNING *` também proibido.

**`isUniqueViolation(err)`** helper em `mappers.go`:

```go
func isUniqueViolation(err error) bool {
    var pgErr *pgconn.PgError
    return errors.As(err, &pgErr) && pgErr.Code == "23505"
}
```

Chamado em `Create` de Vote, CompanyEvaluation, EventResponse (via ON
CONFLICT), AgendaItem — mapeia 23505 → `ErrConflict`.

---

## 12. Property-Based Testing (`pgregory.net/rapid v1.2.0`)

Suites existentes:

### 12.1 `agenda_item_pbt_test.go` — Fração ideal 0-100%

```go
func TestSetFractionIdeal_PBT(t *testing.T) {
    rapid.Check(t, func(t *rapid.T) {
        raw := rapid.Float64Range(0, 100).Draw(t, "fraction")
        item, _ := entities.NewAgendaItem(/* ... */)
        err := item.SetFractionIdeal(fmt.Sprintf("%.2f", raw))
        require.NoError(t, err)
        require.Equal(t, fmt.Sprintf("%.2f", raw), *item.FractionIdeal())
    })
}
```

### 12.2 `feature_usage_pbt_test.go` — Quotas

3 generators (Consume / Unlimited / ZeroLimit); 300 casos rapid, ~10ms.

### 12.3 `abac_engine_pbt_test.go` — 400 casos

Generators produzem `(role, roleType, tier, tenantID, action)`. Verifica
invariantes:
- Admin role sempre concede (bypass).
- `RequireTenantMatch=true` + tenant mismatch → deny.
- `MinPlanTier=plus` + `tier=base` → deny.

### 12.4 `pagination_test.go` — 1000 casos

Valida `FromQuery(ctx)` clampa limit em [0, 100] e offset em [0, MaxInt32].

---

## 13. Integration tests com `testcontainers-go`

`internal/shared/testutil/pg.go`:

```go
func StartPG(t *testing.T) *pgxpool.Pool {
    container, err := postgres.Run(ctx, "postgres:18-alpine",
        postgres.WithDatabase("ms_test"), postgres.WithUsername("ms"), postgres.WithPassword("ms"),
        postgres.WithReuse(),   // reuse container entre testes — acelera rodadas
    )
    // ... inicializa pool + roda migrations
    return pool
}

func TruncateAll(t *testing.T, pool *pgxpool.Pool) { /* DELETE FROM all tables */ }
```

Uso:

```go
//go:build integration

func TestCondominiumRepository_SaveAndFind(t *testing.T) {
    pool := testutil.StartPG(t)
    t.Cleanup(func() { testutil.TruncateAll(t, pool) })

    repo := database.NewCondominiumRepository(pool)
    c, _ := entities.NewCondominium(/* ... */)
    require.NoError(t, repo.Save(ctx, c))
    // ...
}
```

Rodar com `go test -tags=integration ./...` após `docker compose up -d`.

---

## 14. Stubs hand-rolled (sem `mockery`/`testify/mock`)

**Decisão explícita**: sessão autônoma 2026-04-21 criou todos os mocks de use cases como **stubs hand-rolled** em arquivos `*_common_stubs_test.go`.

Motivação:
- Transparência — leitor vê o que cada stub faz sem regenerar nada.
- Zero-generation — sqlc já gera código; evitar mais ferramenta.
- Type-safe — compilador atrapalha se a interface mudar.

Exemplo:

```go
// institutional/application/use_cases/use_cases_common_stubs_test.go
type stubTimelineRepo struct {
    saveCalls int
    saveErr   error
    saved     []*entities.TimelineEntry
}

func (s *stubTimelineRepo) Save(ctx context.Context, e *entities.TimelineEntry) error {
    s.saveCalls++
    if s.saveErr != nil { return s.saveErr }
    s.saved = append(s.saved, e)
    return nil
}
```

---

## 15. Clamp para overflow (G115 do gosec)

`pkg/safecast/safecast.go`:

```go
package safecast

const (
    MaxInt32 int64 = 1<<31 - 1
    MaxInt16 int64 = 1<<15 - 1
)

// Int32 clampa valor int em [0, MaxInt32].
// Usado em todos os sites onde query/body vira int32 de sqlc.
func Int32(v int) int32 {
    if v < 0 { return 0 }
    if int64(v) > MaxInt32 { return int32(MaxInt32) }
    return int32(v)
}

func Int32FromInt64(v int64) int32 { /* idem */ }
func Int16(v int) int16 { /* idem */ }
```

Aplicado em **12 arquivos** de handlers/repositories — G115 de 66 → 0.

---

## 16. Pagination canônico (`internal/shared/pagination/`)

```go
type Params struct {
    Limit  int32
    Offset int32
}

func FromQuery(ctx contracts.Context) Params {
    limit := parseBounded(ctx.Query("limit"), 50, 1, 100)        // default 50, cap 100
    offset := parseBounded(ctx.Query("offset"), 0, 0, math.MaxInt32)
    return Params{Limit: safecast.Int32(limit), Offset: safecast.Int32(offset)}
}
```

PBT 1000 casos verifica clamp e defaults. Handler apenas chama `pagination.FromQuery(ctx)` — zero `strconv.Atoi(Query())` solto.

---

## 17. Logger estruturado sem PII

```go
logger.F("err", err)                 # zerolog aceita error direto — preserva stack trace; nunca err.Error()
logger.F("user_id", user.ID())        # ID UUIDv7 é anônimo — OK
logger.F("cnpj", cnpj.Masked())       # A-004 — nunca Digits()
# PROIBIDO: logger.F("email", user.Email()), logger.F("cpf", ...), logger.F("token", ...)
```

Audit regular via grep:

```bash
grep -rn 'logger\.F("email"\|logger\.F("cpf"\|logger\.F("token"\|logger\.F("password"' internal/
# esperado: zero matches
```

---

## 18. Idempotency em webhooks

### 18.1 Stripe

```go
func (u *HandleStripeWebhookUseCase) Execute(ctx, cmd) error {
    if err := u.gateway.VerifyWebhookSignature(cmd.RawBody, cmd.Signature); err != nil {
        return apperrors.ErrValidation.WithMessage("invalid signature")
    }
    evt, err := u.gateway.ParseWebhookEvent(cmd.RawBody, cmd.Signature)
    if err != nil { return err }

    // Dedup: evt.ID vem estável (evt_XXX). Persistência em webhook_events opcional.
    // ...switch por evt.Type
}
```

### 18.2 Mux

`MuxProvider.verifyMuxSignature` valida HMAC-SHA256 antes de parse; rota
pública `POST /webhooks/mux` fora de Authenticate (A-022 corrigido).

---

## 19. Cleanup de goroutines com `context.WithCancel`

A-032 corrigiu o `RateLimiter`:

```go
// internal/shared/middleware/rate_limiter.go (pós-fix)
func RateLimiter(ctx context.Context, rpm, burst int) contracts.HandlerFunc {
    limiters := &sync.Map{}
    go func() {
        ticker := time.NewTicker(30 * time.Minute)
        defer ticker.Stop()
        for {
            select {
            case <-ctx.Done():
                return                                    // cancelado em Shutdown
            case <-ticker.C:
                // limpa IPs inativos por > 2h
            }
        }
    }()
    return func(c contracts.Context) { /* ... */ }
}
```

Server em `Shutdown` cancela o `lifecycleCtx`, liberando todas as goroutines.

---

## 20. OpenAPI via swag + Scalar UI

`internal/server/docs.go` registra `GET /docs` (Scalar UI) que renderiza
spec compilada pelo `swaggo/swag`. Handlers têm comentários `@Summary`,
`@Param`, `@Success` em godoc — `swag init` gera `docs/swagger.json`.

---

## 21. Antipatterns proibidos (e onde atacam — grep-regex checklist)

| Antipattern | Regex de auditoria | Encontrado? |
|---|---|---|
| `_ = fn()` em I/O crítica | `grep -rn '_ = .*\.\(Save\|Publish\|Send\|Commit\)' internal/` | 0 após F1 (AUDIT A-001..A-008) |
| `else` block | `grep -rn '} else {' internal/` | 0 após F4 |
| `strconv.Atoi(Query()` solto | `grep -rn 'strconv\.Atoi(ctx\.Query' internal/modules/` | 0 após F10 (pagination.FromQuery) |
| `SELECT *` em sqlc | `grep -rn 'SELECT \*' internal/modules/*/infrastructure/database/queries/` | 0 após A-021 |
| `zerolog.SetGlobalLevel` | `grep -rn 'SetGlobalLevel' pkg/logger/` | 0 — mantido removido desde Sprint 1.0.2 |
| CORS `*` aceito em prod | `grep -A3 'AllowedOrigins' internal/shared/config/config.go` | Config.Validate rejeita |
| PII em log | `grep -rn 'logger\.F("email\|logger\.F("cpf\|logger\.F("token' internal/` | 0 (A-003/A-004 fechados) |
| `_ =` em DecodeJSON | `grep -rn '_ = ctx\.DecodeJSON' internal/` | 0 após F3 |
| `WITH CHECK OPTION` ausente em UPDATE multi-tenant | Code review manual | ongoing |

---

## 22. Quando NÃO usar cada padrão

| Padrão | NÃO usar quando... |
|---|---|
| Saga | Operação é puramente intra-DB → UoW é suficiente |
| UoW | Operação é pura-leitura → abrir tx é overhead |
| PBT | Invariante já é coberta por tipo (ex: enum `IsValid()`) |
| Domain event | Outra camada precisa do resultado **síncrono** (não é pub/sub) |
| ABAC rule | Regra é specific-per-user (cai em authz no use case) |
| Matriz declarativa | Menos de 3 combinações; código direto é mais claro |
| Generics `IRepository[T,ID]` | Assinatura por agregado tem parâmetros distintos — use interface explícita |

---

## 23. Referências

- Steering oficial: `backend/.kiro/steering/go-patterns.md`, `database.md`, `transaction-patterns.md`, `testing.md`, `do-dont.md`, `code-calisthenics.md`.
- Antipatterns legado: `backend/.kiro/specs/master-sindico/reference/antipatterns-to-avoid.md`.
- Neste vault: [[architecture]] · [[security]] · [[../../02-architecture/patterns]].
