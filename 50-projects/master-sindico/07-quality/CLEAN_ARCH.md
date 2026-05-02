---
title: Clean Architecture aplicada — Master Síndico (Go)
type: note
tags:
  - principle-applied
  - clean-arch
  - master-sindico
  - quality
project: master-sindico
created: 2026-04-24T00:00:00.000Z
updated: 2026-04-24T00:00:00.000Z
aliases:
  - clean-arch-master-sindico
  - clean-arch-go-ms
---

# Clean Architecture aplicada — Master Síndico v2 (Go)

Regras concretas de Clean Architecture estrita para o backend Go 1.26 do Master Síndico. Especialização do canônico atemporal [[../../../10-knowledge/architecture/clean-architecture-layers]] com layout de pastas, regra de dependência, exemplos de violação e enforcement em CI.

> Cada regra **CA-###** previne pelo menos um antipattern do catálogo [[antipatterns]] (AP-001..AP-026) e/ou implementa uma decisão de [[../02-architecture/clean-arch-mapping]] / [[../03-subprojects/backend/architecture]]. Não duplica o knowledge global — linka.

---

## Sumário

Clean Architecture no MS é **regra-dura, não sugestão**. O legado TS tinha entidades importando Drizzle/Stripe (AP-014), repos retornando `typeof users.$inferSelect` direto (AP-003), e cada BC inventava sua estrutura (AP-008). A reescrita Go corta isso com 4 camadas estritas, layout canônico verificado em CI, e proibições enforcement-listed. **Domínio é puro Go (stdlib + `pkg/`); zero SDK externo.**

---

## Regras concretas

### CA-001 — 4 camadas obrigatórias por bounded context

**Regra**: cada BC sob `internal/modules/<bc>/` **deve** ter exatamente estas pastas:

```
internal/modules/<bc>/
├── domain/
│   ├── entities/            # aggregates + métodos + sentinels
│   ├── value_objects/       # imutáveis
│   ├── enums/               # tipos nomeados + IsValid()
│   ├── repositories/        # interfaces I*Repository
│   ├── events/              # struct + IEventPublisher (quando há)
│   ├── providers/           # interfaces de SDK externo (IPaymentGateway, etc.)
│   └── services/            # domain services puros (cálculos cross-aggregate)
├── application/
│   ├── use_cases/           # 1 arquivo por command/query (CQRS)
│   ├── mappers/             # entity → DTO de saída
│   └── event_handlers/      # subscribers de eventos
├── infrastructure/
│   ├── database/
│   │   ├── queries/         # *.sql (sqlc input)
│   │   ├── generated/       # *.sql.go (sqlc output — NÃO editar)
│   │   ├── repositories/    # *_repository_impl.go
│   │   └── mappers.go       # row → entity
│   ├── http/routes/         # handlers HTTP
│   ├── providers/           # adapters SDK externo
│   ├── events/              # publisher impl
│   └── jobs/                # crons, NATS consumers, outbox poller
└── module.go                # IModule wire-up
```

**Por quê**: previne **AP-008** (inconsistência estrutural entre módulos). Onboarding é instantâneo: dev sabe onde tudo vive em todo BC.

**✅ Exemplo**: `internal/modules/identity/`, `internal/modules/billing/`, `internal/modules/assembly/` seguem o layout (ver [[../03-subprojects/backend/architecture]] §6.1).

**❌ Anti-exemplo (AP-008)**: legado TS misturado tinha `helpers/`, `services/`, `utils/` espalhados sem convenção. BC novo era cópia do anterior + drift.

**Detecção**:
- CI script valida existência das pastas obrigatórias por BC:

```bash
for bc in identity billing institutional commercial content assembly; do
  for layer in domain application infrastructure; do
    [ -d "internal/modules/$bc/$layer" ] || { echo "missing $bc/$layer"; exit 1; }
  done
done
```

- Template `40-templates/project-scaffold/` em [[../../../40-templates/_moc]] — novo BC é cópia.

---

### CA-002 — Regra de dependência: setas apontam para dentro

**Regra**: dependência permitida segue exatamente:

```
domain/         ← imports apenas: stdlib, pkg/, core/errors
application/    ← imports: domain/ (mesmo BC), core/contracts, pkg/, application/ports/
infrastructure/ ← imports: application/ (ports), domain/ (mesmo BC), core/, pkg/, SDKs externos
cross-domain/   ← ÚNICO BC com permissão de importar use cases de múltiplos BCs (sagas)
```

**Por quê**: é a **regra fundadora** ([[../02-architecture/adr/_moc]] ADR-0001). Domínio puro = testável, portável, vida útil longa. Quebrar isso = AP-014 (vazamento framework).

**✅ Exemplo Go (correto)**:

```go
// domain/entities/vote.go — só stdlib
package entities

import (
    "errors"
    "time"

    "github.com/master-sindico/backend/internal/core/errors" // OK
)

// application/use_cases/cast_vote_use_case.go
package usecases

import (
    "context"

    "github.com/master-sindico/backend/internal/modules/commercial/domain/entities"
    "github.com/master-sindico/backend/internal/modules/commercial/domain/repositories"
    "github.com/master-sindico/backend/internal/core/contracts"  // OK: contract
)

// infrastructure/database/vote_repository_impl.go
package database

import (
    "github.com/jackc/pgx/v5"  // OK: infra pode importar driver

    "github.com/master-sindico/backend/internal/modules/commercial/domain/entities"
    "github.com/master-sindico/backend/internal/modules/commercial/domain/repositories"
    "github.com/master-sindico/backend/internal/modules/commercial/infrastructure/database/generated"
)
```

**❌ Anti-exemplo (AP-014, blocker)**:

```go
// domain/entities/subscription.go — PROIBIDO
package entities

import (
    "github.com/stripe/stripe-go/v74"   // ❌ SDK externo no domínio
    "github.com/jackc/pgx/v5"           // ❌ driver no domínio
    "github.com/gin-gonic/gin"          // ❌ framework no domínio
)
```

**Detecção**:
- `go-arch-lint` configurado em `.go-arch-lint.yml` (planejado Sprint 10);
- grep CI por arquivo:

```bash
# Falha se domain/ importa qualquer coisa proibida
forbidden='gin-gonic|jackc/pgx|stripe-go|mux-go|livekit|zitadel|go-redis|rs/zerolog'
if grep -rE "^\s*\"github.com/($forbidden)" internal/modules/*/domain/; then
    echo "FAIL: SDK externo no domain"
    exit 1
fi
```

---

### CA-003 — Domain layer: estado privado, factories, sentinels

**Regra**: aggregate em `domain/entities/`:
- Campos **lowercase** (privados ao package).
- Acesso via getters explícitos (`ID()`, `Name()`...). Zero `Set*(v)` público.
- **Duas factories**:
  - `NewX(...)` valida invariantes (criação).
  - `ReconstructX(...)` hidrata do DB sem revalidar (mapper).
- Sentinels exportados (`var ErrXxx = errors.New(...)`) — não `errors.New` anônimo no caller path.

**Por quê**: preserva invariantes. Construtor zero-value (`User{}`) inválido — força `New*` que valida; `Reconstruct` confia no DB (CHECK constraints já validaram).

**✅ Exemplo Go**:

```go
// internal/modules/institutional/domain/entities/condominium.go
type Condominium struct {
    id         string
    publicID   string
    name       string
    condoType  enums.CondominiumType
    address    value_objects.Address
    totalUnits int
    createdAt  time.Time
}

var (
    ErrInvalidPublicID         = errors.New("invalid public_id format")
    ErrInvalidCondominiumName  = errors.New("name must be 2-200 chars")
    ErrInvalidCondominiumType  = errors.New("invalid condominium type")
    ErrInvalidUnitCount        = errors.New("totalUnits must be 1-10000")
)

// NewCondominium valida invariantes (criação via use case)
func NewCondominium(id, publicID, name string, condoType enums.CondominiumType,
    addr value_objects.Address, totalUnits int) (*Condominium, error) {
    if !publicIDPattern.MatchString(publicID) { return nil, ErrInvalidPublicID }
    if l := len(strings.TrimSpace(name)); l < 2 || l > 200 {
        return nil, ErrInvalidCondominiumName
    }
    if !condoType.IsValid() { return nil, ErrInvalidCondominiumType }
    if totalUnits < 1 || totalUnits > 10000 { return nil, ErrInvalidUnitCount }
    return &Condominium{id, publicID, name, condoType, addr, totalUnits, time.Now().UTC()}, nil
}

// ReconstructCondominium hidrata do DB — sem validar (CHECK do PG já garantiu)
func ReconstructCondominium(id, publicID, name string, condoType enums.CondominiumType,
    addr value_objects.Address, totalUnits int, createdAt time.Time) *Condominium {
    return &Condominium{id, publicID, name, condoType, addr, totalUnits, createdAt}
}

// Getters
func (c *Condominium) ID() string                         { return c.id }
func (c *Condominium) Name() string                       { return c.name }
// ...
```

**❌ Anti-exemplo**:

```go
type Condominium struct {
    ID         string  // exportado e mutável — quebra invariante
    Name       string
    TotalUnits int
}
// uso: c := Condominium{}; c.TotalUnits = -1  // estado inválido
```

**Detecção**: code review; PR template item "domain entity tem campos privados + NewX/ReconstructX?".

---

### CA-004 — Application: CQRS leve, 1 use case por arquivo

**Regra**:
- Arquivos com sufixo `_use_case.go` (singular) para 1 ação atômica;
- `_use_cases.go` (plural) só quando 3+ commands compartilham deps reais (ex: `assembly_use_cases.go` com Publish/Start/End).
- **Command** (write) em arquivo separado de **Query** (read).
- Handler recebe deps via construtor (sem container DI).
- Retorna **DTO**, não aggregate (evita vazamento para HTTP).
- Transaction boundary é a **use case**, nunca o handler.

**Por quê**: previne **AP-010** (regra de negócio em handler) e **AP-011** (mistura de read+write tipo `UpdateAndReturnList`).

**✅ Exemplo Go**:

```go
// internal/modules/billing/application/use_cases/cancel_subscription_use_case.go
type CancelSubscriptionCommand struct {
    SubscriptionID  ULID
    Reason          string
    AtPeriodEnd     bool
}

type CancelSubscriptionUseCase struct {
    subscriptionRepo repositories.ISubscriptionRepository
    paymentGateway   providers.IPaymentGateway
    eventPublisher   events.IEventPublisher
    unitOfWork       contracts.IUnitOfWork
    log              logger.Logger
}

func NewCancelSubscriptionUseCase(/* deps */) *CancelSubscriptionUseCase { ... }

func (uc *CancelSubscriptionUseCase) Execute(ctx context.Context, cmd CancelSubscriptionCommand) error {
    return uc.unitOfWork.Run(ctx, func(ctx context.Context) error {
        sub, err := uc.subscriptionRepo.FindByID(ctx, cmd.SubscriptionID)
        if err != nil { return err }
        if err := sub.MarkCanceled(cmd.Reason); err != nil { return err }
        if err := uc.subscriptionRepo.Save(ctx, sub); err != nil { return err }
        if err := uc.paymentGateway.CancelSubscription(ctx, sub.StripeID(), cmd.AtPeriodEnd); err != nil {
            return fmt.Errorf("stripe cancel: %w", err)
        }
        return uc.eventPublisher.Publish(ctx, events.SubscriptionCanceledEvent{ID: sub.ID()})
    })
}
```

**❌ Anti-exemplo (AP-011)**:

```go
// UM handler que faz UPDATE + lista filtrada — write + read misturados
func (uc *MegaUseCase) UpdateAndReturnList(ctx context.Context, cmd Mixed) ([]DTO, error) {
    // ... atualiza X
    // ... query para retornar lista paginada
}
```

**Detecção**: PR review; `find internal/modules/*/application/use_cases -name "*.go" -exec wc -l {} \;` — arquivo > 250 linhas é sinal de juntar 5+ use cases — split.

---

### CA-005 — Infrastructure: mapper explícito row ↔ entity

**Regra**: repositories (`*_repository_impl.go`) **nunca** retornam tipo gerado pelo sqlc para fora. Mapper (`mappers.go`) traduz `generated.X` → `*entities.Y`.

**Por quê**: previne **AP-003** + **AP-021** (vazamento ORM). Trocar pgx → outro driver vira refactor de 1 arquivo, não 200.

**✅ Exemplo Go**:

```go
// infrastructure/database/repositories/condominium_repository_impl.go
func (r *CondominiumRepository) FindByID(ctx context.Context, id string) (*entities.Condominium, error) {
    db := database.DBFromContext(ctx, r.pool)
    q := generated.New(db)
    row, err := q.GetCondominiumByID(ctx, id)
    if err != nil {
        if errors.Is(err, pgx.ErrNoRows) {
            return nil, apperrors.ErrNotFound
        }
        return nil, fmt.Errorf("get condominium: %w", err)
    }
    return mappers.RowToCondominium(row), nil  // <-- nunca retorna `row`
}

// infrastructure/database/mappers.go
func RowToCondominium(r generated.Condominium) *entities.Condominium {
    return entities.ReconstructCondominium(
        r.ID, r.PublicID, r.Name,
        enums.CondominiumType(r.CondoType),
        ToAddressVO(r.Address),
        int(r.TotalUnits),
        r.CreatedAt.Time,
    )
}
```

**❌ Anti-exemplo (AP-003)**:

```go
func (r *CondominiumRepository) FindByID(ctx context.Context, id string) (*generated.Condominium, error) {
    return r.q.GetCondominiumByID(ctx, id)  // ❌ tipo sqlc vazando
}
```

**Detecção**: grep CI:

```bash
grep -rE "func \([a-z]+ \*[A-Z]\w+Repository\) \w+\(.*\) \(.*\*?(generated|sqlcgen|pgtype)\." \
    internal/modules/*/infrastructure/database/repositories/ && exit 1
```

---

### CA-006 — `domain/` proibições enforcement-listed

**Regra**: imports proibidos em `internal/modules/*/domain/`:

| Pacote | Razão |
|---|---|
| `github.com/gin-gonic/gin` | Framework HTTP em camada errada |
| `github.com/jackc/pgx/*` | Driver DB em camada errada |
| `github.com/redis/go-redis/*` | Cliente Redis em camada errada |
| `github.com/stripe/stripe-go/*` | SDK externo |
| `github.com/muxinc/mux-go` | SDK externo |
| `github.com/livekit/server-sdk-go` | SDK externo |
| `github.com/zitadel/*` | SDK externo |
| `github.com/rs/zerolog` | Logger só em application/infra |
| `github.com/master-sindico/backend/internal/modules/<outro_bc>/*` | Cross-BC import direto (use evento) |
| `github.com/master-sindico/backend/internal/modules/<this_bc>/application/*` | Inverte regra de dependência |
| `github.com/master-sindico/backend/internal/modules/<this_bc>/infrastructure/*` | Inverte regra de dependência |

**Por quê**: previne **AP-014** (vazamento framework) e **AP-021** (cross-BC import).

**Detecção**: CI script consolidado:

```bash
#!/bin/bash
# scripts/check_domain_purity.sh
forbidden=(
    'gin-gonic/gin'
    'jackc/pgx'
    'redis/go-redis'
    'stripe/stripe-go'
    'muxinc/mux-go'
    'livekit/server-sdk-go'
    'zitadel/'
    'rs/zerolog'
)
for f in $(find internal/modules/*/domain -name '*.go' -not -name '*_test.go'); do
    for pkg in "${forbidden[@]}"; do
        if grep -q "\"github.com/$pkg" "$f"; then
            echo "VIOLATION: $f imports github.com/$pkg"
            exit 1
        fi
    done
done
```

---

### CA-007 — `application/` não importa de outro BC's `domain/`

**Regra**: comunicação cross-BC acontece via **eventos de domínio** + interfaces em `internal/shared/types/`. `commercial/application/` **não** importa `billing/domain/entities/Subscription`.

**Por quê**: previne **AP-021** (cross-BC import). Cada BC evolui no seu ritmo; eventos são contratos versionáveis.

**✅ Exemplo Go**:

```go
// shared/types/quota_consumer.go
type IQuotaConsumer interface {
    Consume(ctx context.Context, cmd ConsumeQuotaCommand) error
}

// commercial/application/use_cases/initiate_connect_me_use_case.go
type InitiateConnectMeUseCase struct {
    quotaConsumer  types.IQuotaConsumer  // interface — não importa billing/domain
    // ...
}

// billing/application/quota_consumer_adapter.go
type QuotaConsumerAdapter struct { uc *ConsumeQuotaUseCase }
var _ types.IQuotaConsumer = (*QuotaConsumerAdapter)(nil)
```

**❌ Anti-exemplo (AP-021)**:

```go
// commercial/application/use_cases/initiate_connect_me_use_case.go
import "github.com/master-sindico/backend/internal/modules/billing/domain/entities"
// uses entities.Subscription directly — ❌ blocker
```

**Detecção**: grep CI cruzando BC's:

```bash
for bc in identity billing institutional commercial content assembly; do
    grep -rE "internal/modules/(?!$bc)\w+/(domain|application)" \
        internal/modules/$bc/application/ && exit 1
done
```

---

### CA-008 — `interface/` (HTTP) não chama `infrastructure/` direto

**Regra**: handler HTTP nunca chama repository ou provider direto. Sempre via use case (`application/`). Handler é decode → use case → encode.

**Por quê**: handler = plumbing. Lógica em handler = AP-010.

**✅ Exemplo Go** (já em CC-003).

**❌ Anti-exemplo**:

```go
// infrastructure/http/routes/condominium_handler.go — ERRADO
func (h *CondominiumHandler) Get(c contracts.Context) {
    cond, _ := h.condominiumRepo.FindByID(c.RequestContext(), c.Param("id"))  // ❌ pula use case
    c.JSON(200, cond)
}
```

**Detecção**: grep CI: handler chamando `*Repository.` direto. Exceção: query simples já tem `Get<X>QueryUseCase` mínimo.

---

### CA-009 — Use case naming `<Verb><Noun>UseCase`

**Regra**: arquivo `<verb>_<noun>_use_case.go` (snake_case); type `<Verb><Noun>UseCase` (PascalCase). Verbo no infinitivo inglês (Register, Cancel, Publish, Cast, Approve).

**Por quê**: convenção zero-ambiguidade. Claude autônomo gera com nome certo.

**✅ Exemplos reais do repo**:

```
register_condominium_use_case.go   → RegisterCondominiumUseCase
cast_vote_use_case.go              → CastVoteUseCase
cancel_subscription_use_case.go    → CancelSubscriptionUseCase
publish_announcement_use_case.go   → PublishAnnouncementUseCase
upload_video_use_case.go           → UploadVideoUseCase  (Saga)
checkout_subscription_saga.go      → CheckoutSubscriptionSaga (Saga é exceção de naming)
```

**❌ Anti-exemplo**: `service.go`, `helper.go`, `manager.go`, `do_stuff.go` — todos blockers.

**Detecção**: PR review.

---

### CA-010 — Aggregate root: invariantes no construtor + métodos como única fronteira de mutação

**Regra**: aggregate só muda via método público que retorna `error` quando invariante quebra. Sem `Set*Field()` exposto; sem mutação direta de campos.

**Por quê**: invariantes (INV-###) são a essência do domínio. Mutação não controlada quebra estado.

**✅ Exemplo Go**:

```go
// internal/modules/assembly/domain/entities/assembly.go
func (a *Assembly) Start() error {
    // INV-074 + INV-075: edital publicado + status válido
    if a.status == enums.AssemblyStatusEncerrada { return ErrAssemblyAlreadyEnded }
    if a.status != enums.AssemblyStatusPublicada { return ErrAssemblyNotPublished }
    if !a.IsEditalPublished() { return ErrAssemblyEditalRequired }
    now := time.Now()
    a.status = enums.AssemblyStatusEmAndamento
    a.startedAt = &now
    a.updatedAt = now
    return nil
}

// CastVote: INV-081 + INV-071
func (a *Assembly) CastVote(voter VoterInfo, itemID ULID, opt enums.VoteOption) error {
    if voter.UserID == a.presidentID { return ErrPresidentCannotVote } // INV-081
    if a.votes.AlreadyVoted(voter.UserID, itemID) { return ErrDuplicateVote } // INV-071
    a.votes = append(a.votes, NewVote(voter.UserID, itemID, opt))
    return nil
}
```

**❌ Anti-exemplo**:

```go
// uc.go — mutação direta, invariantes ausentes
assembly.Status = "em_andamento"
assembly.StartedAt = time.Now()
```

**Detecção**: code review; teste unit que tenta `assembly.status = ...` deve falhar (campo privado, build quebra).

---

### CA-011 — Mappers: `domain ↔ db` e `domain ↔ dto`, nunca `db ↔ dto`

**Regra**: 2 mappers por BC com use case ativo:
1. `infrastructure/database/mappers.go` — row sqlc ↔ domain entity.
2. `application/mappers/<entity>_mapper.go` — domain entity ↔ output DTO.

**Não** existe mapper direto db → dto (pula domínio = bypass de invariantes).

**Por quê**: domínio é o *centro*. Cada saída/entrada passa pela validação dele.

**✅ Exemplo**:

```go
// application/mappers/vote_mapper.go
func VoteToDTO(v *entities.Vote) *VoteDTO {
    return &VoteDTO{
        ID:        v.ID().String(),
        SessionID: v.SessionID().String(),
        Option:    string(v.Option()),
        CastAt:    v.CastAt(),
    }
}
```

**❌ Anti-exemplo**:

```go
// handler que mapeia generated → dto direto
dto := VoteDTO{ID: row.ID, Option: row.Option, ...}  // pulou Vote entity → invariantes não validados em response
```

**Detecção**: code review.

---

### CA-012 — sqlc queries: colunas explícitas, sem `SELECT *`, sem `RETURNING *`

**Regra**: **toda** query lista colunas. `SELECT *` proibido. `RETURNING *` proibido. Exceção: `SELECT COUNT(*)`.

**Por quê**: previne **AP-016** + **AP-021**. Adicionar coluna nova (ex: `password_hash`) não vaza para mapper sem revisão. Performance previsível.

**✅ Exemplo SQL**:

```sql
-- internal/modules/identity/infrastructure/database/queries/users.sql

-- name: GetUserByID :one
SELECT id, public_id, cpf_digits, email, role, created_at, updated_at
FROM users
WHERE id = $1 AND deleted_at IS NULL;

-- name: InsertVote :one
INSERT INTO votes (id, tenant_id, agenda_item_id, voter_id, option, weight, cast_at)
VALUES ($1, $2, $3, $4, $5, $6, $7)
RETURNING id, cast_at;
```

**❌ Anti-exemplo**:

```sql
-- name: GetUserByID :one
SELECT * FROM users WHERE id = $1;     -- ❌ AP-016
```

**Detecção**:

```bash
# CI: falha em SELECT * (exceto COUNT)
if grep -rnE 'SELECT \*' internal/modules/*/infrastructure/database/queries/ | grep -v 'COUNT(\*)'; then
    echo "FAIL: SELECT * encontrado"
    exit 1
fi
# CI: falha em RETURNING *
grep -rnE 'RETURNING \*' internal/modules/*/infrastructure/database/queries/ && exit 1
```

---

### CA-013 — Multi-tenant scoping obrigatório

**Regra**: toda query em tabela tenant-scoped recebe `tenant_id` como parâmetro **e** `WHERE tenant_id = $1` no SQL. Convenção: `tenant_id` é sempre o **primeiro** parâmetro.

**Por quê**: previne **AP-025** (cross-tenant leak). LGPD crítico, blocker absoluto.

**✅ Exemplo SQL**:

```sql
-- name: GetCondominiumByID :one
SELECT id, public_id, name, total_units, created_at
FROM condominiums
WHERE tenant_id = $1 AND id = $2 AND deleted_at IS NULL;

-- name: ListAnnouncementsByCondo :many
SELECT id, title, body, published_at
FROM announcements
WHERE tenant_id = $1 AND condominium_id = $2
ORDER BY published_at DESC LIMIT $3 OFFSET $4;
```

**❌ Anti-exemplo (AP-025)**:

```sql
-- ❌ sem tenant_id — IDOR cross-tenant
SELECT * FROM condominiums WHERE id = $1;
```

**Detecção**:
- `sqlc.yaml` marca tabelas com `tenant_scoped: true` (custom plugin planejado);
- grep CI: query em tabela marcada **deve** ter `WHERE tenant_id`;
- integration test obrigatório `cross_tenant_isolation_test.go` por repo (100+ casos cobertos em [[../09-testing/integration]]);
- PBT ABAC 400 casos em [[../09-testing/pbt]].

---

### CA-014 — Webhook público fora de `/api/v1`

**Regra**: webhook inbound (Stripe, Mux, futuros) registra em rota raiz (`/webhooks/<provider>`) **antes** de `RegisterModules`, fora do middleware `Authenticate`. HMAC verificado **antes** de parse body.

**Por quê**: webhook do provider não tem token de usuário; passar por `Authenticate` retorna 401 (A-022 legado). HMAC antes de parse evita JSON bomb / inject (AP-023).

**✅ Exemplo Go** (`cmd/api/main.go`):

```go
// Antes de RegisterModules
srv.RegisterPublicWebhook("/webhooks/stripe", billingModule.WebhookHandler())
srv.RegisterPublicWebhook("/webhooks/mux",    contentModule.MuxWebhookHandler())

// Handler
func (h *StripeWebhookHandler) Handle(c contracts.Context) {
    body, _ := io.ReadAll(c.Request().Body)
    if err := h.gateway.VerifyWebhookSignature(body, c.GetHeader("Stripe-Signature")); err != nil {
        c.JSON(400, apperrors.ErrValidation.WithMessage("invalid signature"))  // CC-014
        return
    }
    // só agora parse e processa
}
```

**❌ Anti-exemplo (AP-023)**:

```go
// rota dentro de /api/v1 → bloqueada por Authenticate
v1 := engine.Group("/api/v1", AuthMiddleware())
v1.POST("/webhooks/stripe", h.Handle)  // 401 sempre
```

**Detecção**: integration test que faz POST em `/webhooks/stripe` sem signature → expect 400; com signature válida → expect 200.

---

### CA-015 — Timeline / Ata = INSERT-only no DB (immutable)

**Regra**: tabelas `timeline_entries`, `assembly_atas`, `audit_logs` têm:
- `REVOKE UPDATE, DELETE` da role da app.
- Trigger `BEFORE UPDATE/DELETE` que `RAISE EXCEPTION`.
- Correção = nova entry com `correction_of_id` apontando para a original.

**Por quê**: ADR-0033 (immutable timeline). Audit trail LGPD requer não-repúdio. Previne soft-delete malicioso.

**✅ Exemplo Go**:

```go
// application/use_cases/correct_timeline_entry_use_case.go
func (uc *CorrectTimelineEntryUseCase) Execute(ctx context.Context, cmd CorrectCmd) error {
    return uc.uow.Run(ctx, func(ctx context.Context) error {
        original, err := uc.timelineRepo.FindByID(ctx, cmd.OriginalID)
        if err != nil { return err }
        correction, err := entities.NewTimelineEntry(
            entities.WithCorrectionOf(original.ID()),
            entities.WithBody(cmd.NewBody),
        )
        if err != nil { return err }
        return uc.timelineRepo.Insert(ctx, correction)  // INSERT-only
    })
}
```

**❌ Anti-exemplo**:

```sql
UPDATE timeline_entries SET body = $1 WHERE id = $2;  -- trigger RAISE EXCEPTION
DELETE FROM timeline_entries WHERE id = $1;            -- trigger RAISE EXCEPTION
UPDATE timeline_entries SET deleted_at = now();        -- ainda viola; sem soft-delete
```

**Detecção**: integration test tenta UPDATE/DELETE → expect erro PG. Migration valida trigger existe.

---

### CA-016 — `module.go`: única porta de entrada do BC

**Regra**: cada BC expõe `module.go` que implementa `contracts.IModule` com `Name()`, `Register(router)`, `Bootstrap()`. **Toda** wire-up DI vive aí. `main.go` chama `module.NewModule(deps).Register(router)`.

**Por quê**: encapsulamento. Toggling BC inteiro = remover 1 linha em `main.go`. Sem container DI mágico.

**✅ Exemplo Go** (`internal/modules/billing/module.go`):

```go
type Module struct {
    webhookHandler  *routes.WebhookHandler
    subscriptionUC  *usecases.CreateSubscriptionUseCase
    quotaConsumer   types.IQuotaConsumer
    eventPublisher  events.IEventPublisher
    abacEngine      *middleware.ABACEngine
    log             logger.Logger
}

var _ contracts.IModule = (*Module)(nil)

func NewModule(pool *pgxpool.Pool, uow contracts.IUnitOfWork, cfg *config.Config,
    abac *middleware.ABACEngine, log logger.Logger) (*Module, error) {
    // ... constrói repos, providers, use cases internos
}

func (m *Module) Name() string { return "billing" }

func (m *Module) Register(router contracts.HTTPRouter) error {
    grp := router.Group("/billing")
    grp.POST("/subscriptions",
        middleware.RequirePermission(m.abacEngine, "billing.subscription.create", ...),
        m.subHandler.Create)
    return nil
}

func (m *Module) Bootstrap() error {
    m.eventPublisher.Subscribe(events.EventSubscriptionActivated, m.onActivated)
    return nil
}

// Métodos exportados para outros BCs (interface em shared/types/)
func (m *Module) QuotaConsumer() types.IQuotaConsumer { return m.quotaConsumer }
```

**❌ Anti-exemplo**: `main.go` com 600 linhas de wire-up direto, BCs sem encapsulamento.

**Detecção**: `find internal/modules/*/module.go` — todo BC deve ter; `grep -L "var _ contracts.IModule" internal/modules/*/module.go` deve ser vazio.

---

### CA-017 — Testes por camada

**Regra**: cobertura mínima por camada (canônico [[../CLAUDE]] §9):

| Camada | Mínimo | Tipo de teste |
|---|---|---|
| `domain/` | 95% | Unit + PBT |
| `application/use_cases/` | 90% | Unit + integração |
| `infrastructure/database/` | 85% | Integration (testcontainers) |
| `infrastructure/http/` | 75% | Unit + contract |
| `shared/middleware/` | 90% | Unit |
| `core/`, `pkg/` | 95% | Unit |

Test files lado a lado: `vote.go` + `vote_test.go` + `vote_pbt_test.go` quando houver invariante crítica.

**Por quê**: previne **AP-009** (ausência de testes). Confiança em refactor agressivo.

**Detecção**:

```bash
go test -race -count=1 ./...
go test -tags=integration -count=1 ./...
go tool cover -func=coverage.out  # parse + check threshold
```

---

## Diagrama de dependência (visual)

```
┌─────────────────────────────────────────────────────────┐
│  infrastructure/                                         │
│  ├── http/routes      → handler (CC-003)                 │
│  ├── database/repos   → impl IXxxRepository              │
│  ├── providers/<sdk>  → impl IXxxGateway                 │
│  └── jobs             → cron, NATS consumers             │
└────────────────┬────────────────────────────────────────┘
                 │ depends on
                 ▼
┌─────────────────────────────────────────────────────────┐
│  application/                                            │
│  ├── use_cases  → CQRS handlers                          │
│  ├── ports      → IPaymentGateway, IVideoProvider, ...   │
│  ├── mappers    → entity ↔ DTO                           │
│  └── event_handlers                                       │
└────────────────┬────────────────────────────────────────┘
                 │ depends on
                 ▼
┌─────────────────────────────────────────────────────────┐
│  domain/  ← stdlib + pkg/ apenas                         │
│  ├── entities          → aggregates + invariantes        │
│  ├── value_objects     → imutáveis                       │
│  ├── enums             → tipos nomeados                  │
│  ├── repositories      → interface IXxxRepository        │
│  ├── services          → cross-aggregate puro            │
│  ├── events            → struct + IEventPublisher        │
│  └── providers         → interface IXxxGateway           │
└─────────────────────────────────────────────────────────┘

cross-domain/ → único módulo que pode importar use cases de múltiplos BCs (sagas inter-BC)
shared/types/ → contratos cross-BC (interfaces); evita import direto entre BCs
core/         → contracts (HTTPRouter, Context, IUnitOfWork, Clock), errors padronizados
pkg/          → libs neutras (logger, money, pagination, safecast)
```

---

## Checklist de PR (cole no template)

```markdown
## Clean Architecture (CA)

- [ ] CA-001: BC novo segue layout canônico (domain/application/infrastructure)
- [ ] CA-002: nenhum import de application/infra/SDK em domain/
- [ ] CA-003: aggregate tem campos privados + NewX/ReconstructX + sentinels
- [ ] CA-004: 1 use case por arquivo; CQRS write/read separado
- [ ] CA-005: repo retorna *entities.X, nunca generated.X
- [ ] CA-006: domain/ não importa nenhum SDK proibido (CI script verde)
- [ ] CA-007: nenhum import cross-BC direto (use shared/types/)
- [ ] CA-008: handler HTTP só chama use case (não repo direto)
- [ ] CA-009: arquivo `<verb>_<noun>_use_case.go` + type `<Verb><Noun>UseCase`
- [ ] CA-010: invariantes validadas no construtor + métodos retornam error
- [ ] CA-011: mappers domain↔db e domain↔dto presentes; sem db↔dto direto
- [ ] CA-012: zero `SELECT *` / `RETURNING *` em queries sqlc
- [ ] CA-013: queries em tabelas tenant-scoped têm `WHERE tenant_id = $1`
- [ ] CA-014: novo webhook registrado fora de /api/v1; HMAC antes de parse
- [ ] CA-015: tabela timeline/ata é INSERT-only (REVOKE + trigger)
- [ ] CA-016: BC novo expõe `module.go` implementando IModule
- [ ] CA-017: cobertura por camada respeita threshold (domain ≥ 95%, app ≥ 90%, ...)
```

---

## Exceções aceitas (com justificativa)

| Exceção | Justificativa |
|---|---|
| `core/contracts/` importa de qualquer canto | É o pacote de interfaces transversais por design |
| `cross-domain/sagas/` importa use cases de múltiplos BCs | Único módulo com permissão; documentado em ADR-0015 |
| `internal/modules/*/infrastructure/database/generated/` | Código sqlc; **não editar**; CA-### não aplicáveis a generated |
| `pkg/safecast`, `pkg/money`, `pkg/pagination` | Libs neutras; podem ser importadas por qualquer camada |
| Testes (`*_test.go`) | Podem importar livremente outros BCs para integration tests |
| `infrastructure/database/repositories/<entity>_repository_impl.go` importa `application/ports/` | Para verificar contrato `var _ ports.IXxx = (*XxxImpl)(nil)` é ok |
| Saga handler com 80-100 linhas | Compensação explícita justifica passar de CC-002 (50 linhas) — comentar #saga-exception |

---

## Vizinhos

- [[../../../10-knowledge/architecture/clean-architecture-layers]] — referência canônica atemporal
- [[../../../10-knowledge/architecture/_moc]]
- [[../02-architecture/clean-arch-mapping]] — mapping camadas (canônico do projeto)
- [[../02-architecture/patterns]] — patterns de arquitetura
- [[../02-architecture/adr/_moc]] — ADR-0001 (clean-arch+ddd+cqrs), ADR-0033 (timeline immutable), ADR-0034 (RLS)
- [[../03-subprojects/backend/architecture]] — arquitetura backend real (código)
- [[PATTERNS]] — patterns aplicados (DDD, CQRS, Saga, etc.)
- [[CLEAN_CODE]] — regras de Clean Code
- [[SOLID]] — princípios SOLID Go-flavor
- [[CODE_CALISTHENICS]] — 9 regras Calisthenics
- [[antipatterns]] — catálogo AP-001..AP-026
- [[../01-domain/invariants]] — INV-### testáveis
- [[../CLAUDE]] §5 — proibições
