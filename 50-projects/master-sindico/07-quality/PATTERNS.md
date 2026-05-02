> **Ver knowledge global atemporal** — este documento traz **patterns aplicados no Master Síndico (Go)**. Conceitos canônicos:
> - [[../../../10-knowledge/patterns/transaction-patterns]] — ACID/UoW/Saga
> - [[../../../10-knowledge/architecture/clean-architecture-layers]] — 4 camadas
> - [[../../../10-knowledge/architecture/ddd-strategic-tactical]] — DDD estratégico + tático
> - [[../../../10-knowledge/architecture/cqrs]] — Command-Query
> - [[../../../10-knowledge/principles/solid]] — 5 princípios
> - [[../../../10-knowledge/principles/code-calisthenics]] — 9 regras
>
> Este arquivo documenta **como esses patterns se materializam no código MS** — instâncias concretas.

---

title: Patterns aplicados — Master Síndico (Go)
type: guide
tags: [patterns, ddd, cqrs, saga, master-sindico, v2, go]
source: 02-architecture/patterns.md + 90-ingested/from-vault-50-projects-master-sindico/07-quality/PATTERNS.md
created: 2026-04-23
updated: 2026-04-23
aliases: [patterns-master-sindico-go]
---

# Patterns aplicados — Master Síndico v2 (Go)

Especialização de [[../02-architecture/patterns]] com exemplos Go concretos, dobrando o mapa canônico em snippets usáveis. Cada pattern tem **quando usar**, **quando NÃO**, **exemplo concreto** e **anti-pattern relacionado** (cross-link pra [[antipatterns]]).

> Herda filosofia [[../CLAUDE]] §14 (YAGNI): pattern só entra quando tem 2+ usos reais.

---

## 1. DDD tático (aggregate + VO + event + service + spec)

### Quando usar no master-sindico
- Todo BC com invariantes não-triviais (institutional, assembly, commercial, billing).
- Invariantes: Σ FracaoIdeal ≤ 100%, vote único, timeline imutável, quota persona-aware.

### Quando NÃO usar
- Tabelas técnicas (sessions cache, rate limit counters, outbox_events) — CRUD sqlc basta.
- Scripts one-off (migração de dado).

### Exemplo Go — aggregate `Assembly` com invariante

```go
// domain/entities/assembly.go
type Assembly struct {
    id            ULID
    tenantID      ULID
    status        AssemblyStatus // draft|published|in_progress|closed|homologated
    editalPublishedAt *time.Time
    items         AgendaItems
    votes         Votes
    presidentID   ULID
    scheduledAt   time.Time
}

// INV-074: assembleia não inicia sem edital publicado
func (a *Assembly) Start(at time.Time) error {
    if a.editalPublishedAt == nil {
        return apperrors.ErrConflict.WithMessage("edital não publicado")
    }
    if a.status != AssemblyPublished {
        return apperrors.ErrConflict.WithMessage("status inválido para iniciar")
    }
    a.status = AssemblyInProgress
    a.recordEvent(AssemblyStarted{AssemblyID: a.id, At: at})
    return nil
}

// INV-081: síndico presidente não vota
func (a *Assembly) CastVote(voter VoterInfo, itemID ULID, option VoteOption) error {
    if voter.UserID == a.presidentID {
        return apperrors.ErrForbidden.WithMessage("presidente síndico não vota")
    }
    // ...resto
}
```

### Anti-pattern relacionado
- **AP-009** (anemic domain) — DTOs nus com getters/setters; invariantes no use case.
- **AP-012** (God aggregate) — aggregate com 20 entidades; quebrar via domain events.
- Violação da regra de dependência ([[CLEAN_ARCH]] §3): domain importando Gin/pgx/SDK.

---

## 2. Repository (interface em domain, impl em infra)

### Quando usar
- Todo aggregate tem repositório. Interface em `domain/repositories.go`, impl em `infrastructure/database/repositories/`.

### Quando NÃO usar
- Query pura com `QueryHandler` dedicado — lê sqlc direto via `IReadableStore` sem passar por repository.

### Exemplo Go

```go
// domain/repositories.go
type IVoteRepository interface {
    Save(ctx context.Context, v *Vote) error
    Get(ctx context.Context, tenantID, id ULID) (*Vote, error)
    ExistsForVoter(ctx context.Context, tenantID, agendaItemID, voterID ULID) (bool, error)
}

// infrastructure/database/repositories/vote_repo.go
type voteRepo struct{ q *sqlcgen.Queries }

func (r *voteRepo) Save(ctx context.Context, v *Vote) error {
    db := database.DBFromContext(ctx, r.pool) // tx-aware
    return r.q.InsertVote(ctx, db, mappers.VoteToRow(v))
}
```

### Anti-pattern relacionado
- **AP-003** (tipo ORM vazando): `func (r *voteRepo) Get(...) (*sqlcgen.Vote, error)` — fix: mapper → `*domain.Vote`.
- **Generic Repository `IRepository[T]`** — perde intenção, força métodos sem sentido (`DeleteAll` em timeline imutável).

---

## 3. Factory Method (`NewX` valida; `ReconstructX` hidrata)

### Quando usar
- Aggregate com invariantes (validação obrigatória na criação).
- Hidratação do repo → `ReconstructX` sem revalidar (DB já garantiu).

### Quando NÃO usar
- VO trivial sem invariante — construtor literal.

### Exemplo Go

```go
// NewUser valida tudo — entrada externa
func NewUser(cpf CPFVO, email EmailVO, role Role) (*User, error) {
    if role == "" { return nil, apperrors.ErrValidation.WithMessage("role obrigatório") }
    return &User{id: ULIDNew(), cpf: cpf, email: email, role: role, createdAt: time.Now().UTC()}, nil
}

// ReconstructUser vem do DB — confia
func ReconstructUser(id ULID, cpf CPFVO, email EmailVO, role Role, createdAt time.Time) *User {
    return &User{id: id, cpf: cpf, email: email, role: role, createdAt: createdAt}
}
```

### Anti-pattern relacionado
- Construtor exportado que permite estado inválido (`User{}` zero value utilizável).
- Validação no setter em vez do factory.

---

## 4. Value Object imutável

### Quando usar
- Conceito sem identidade: `EmailVO`, `CPFVO`, `CNPJVO`, `Money`, `FracaoIdeal`, `Percentage`, `ULID`, `CouponCode`, `DeviceFingerprint`.
- Comparação por valor.

### Quando NÃO usar
- Conceito com identidade (`User`, `Condominium`, `Vote`) — é aggregate/entity.

### Exemplo Go

```go
// pkg/money/money.go
type Money struct{ cents int64 }

func NewMoneyBRL(cents int64) Money { return Money{cents: cents} }
func (m Money) Cents() int64 { return m.cents }
func (m Money) Add(o Money) Money { return Money{m.cents + o.cents} }
func (m Money) IsZero() bool { return m.cents == 0 }
func (m Money) String() string { return fmt.Sprintf("R$ %.2f", float64(m.cents)/100) }
```

### Anti-pattern relacionado
- **AP-###** (float64 pra dinheiro/fração): precisão instável; usar `int64` centavos ou basis points.
- VO com método `Set...` — viola imutabilidade.

---

## 5. CQRS (command ≠ query)

### Quando usar
- Todo use case. Arquivos separados `*_command.go` e `*_query.go`.
- Permite escalar leitura (materialized view, Redis, OpenSearch) sem refactor write side.

### Quando NÃO usar
- Event Sourcing full (M1/M2 não; M3+ avaliar se LGPD/auditoria exigir).

### Exemplo Go

```go
// application/use_cases/cast_vote_command.go
type CastVoteCommand struct {
    TenantID, AssemblyID, AgendaItemID, VoterID ULID
    Option VoteOption
}

type CastVoteHandler struct {
    assemblyRepo IAssemblyRepository
    voteRepo     IVoteRepository
    uow          IUnitOfWork
    clock        Clock
}

func (h *CastVoteHandler) Handle(ctx context.Context, cmd CastVoteCommand) (*VoteDTO, error) {
    return WithTxReturning(ctx, h.uow, func(ctx context.Context) (*VoteDTO, error) {
        a, err := h.assemblyRepo.Get(ctx, cmd.TenantID, cmd.AssemblyID)
        if err != nil { return nil, err }
        vote, err := a.CastVote(cmd.VoterID, cmd.AgendaItemID, cmd.Option, h.clock.Now())
        if err != nil { return nil, err }
        if err := h.voteRepo.Save(ctx, vote); err != nil { return nil, err }
        return mappers.VoteToDTO(vote), nil
    })
}
```

### Anti-pattern relacionado
- **AP-011** (if/else hell de dispatch) — `UpdateAndReturnList` que mistura write+read.
- Event sourcing prematuro em M1/M2.

---

## 6. UnitOfWork (intra-BC)

### Quando usar
- Command toca 2+ aggregates do mesmo BC atomicamente.
- Outbox pattern: INSERT `outbox_events` na mesma tx do write principal (INV-095).

### Quando NÃO usar
- Operação atravessa BC ou provider externo → **Saga**.

### Exemplo Go

```go
// core/contracts/uow.go
type IUnitOfWork interface {
    Run(ctx context.Context, fn func(ctx context.Context) error) error
}

// application/use_cases/publish_announcement_command.go
func (h *PublishAnnouncementHandler) Handle(ctx context.Context, cmd ...) error {
    return h.uow.Run(ctx, func(ctx context.Context) error {
        ann, _ := h.annRepo.Get(ctx, cmd.ID)
        if err := ann.Publish(h.clock.Now()); err != nil { return err }
        if err := h.annRepo.Save(ctx, ann); err != nil { return err }
        if err := h.timelineRepo.AppendEntry(ctx, "announcement_published", ann.ID); err != nil { return err }
        return h.outboxRepo.Enqueue(ctx, AnnouncementPublishedEvent{ID: ann.ID})
    })
}
```

### Anti-pattern relacionado
- Abrir tx no handler HTTP (tx pertence ao use case).
- UoW cruzando provider externo (Stripe/Mux não rollback) → Saga.

Referência: [[../02-architecture/adr/_moc]] ADR-0015.

---

## 7. Saga com compensação (inter-BC/provider)

### Quando usar
- Fluxo atravessa provider externo (Stripe, Mux, LiveKit).
- Fluxo cruza BCs e precisa rollback semântico.

### Quando NÃO usar
- Tudo cabe numa tx PG → UoW.

### Exemplo Go — UploadVideo (A-027 fix)

```go
func (h *UploadVideoHandler) Handle(ctx context.Context, cmd UploadVideoCommand) (*VideoDTO, error) {
    // 1. Cria upload no Mux
    upload, err := h.videoProvider.CreateDirectUpload(ctx, DirectUploadReq{
        TenantID: cmd.TenantID, PlaybackPolicy: "signed",
    })
    if err != nil { return nil, fmt.Errorf("mux create upload: %w", err) }

    // 2. Salva entidade local; se falhar, compensar
    video, _ := domain.NewVideo(cmd, upload.AssetID)
    if err := h.videoRepo.Create(ctx, video); err != nil {
        if cErr := h.videoProvider.CancelUpload(ctx, upload.ID); cErr != nil {
            h.logger.Error().Err(cErr).
                Str("original_err", err.Error()).
                Str("mux_upload_id", upload.ID).
                Msg("saga compensation failed — manual cleanup needed")
        }
        return nil, err
    }
    return mappers.VideoToDTO(video), nil
}
```

### Anti-pattern relacionado
- **AP-###** (saga sem compensação, A-027/A-028 legado) — orquestração otimista sem rollback.
- Distributed transaction 2PC — Stripe/Mux não suportam.
- Saga síncrona bloqueando request HTTP — usar worker + state machine.

Referência: [[../13-research/netflix/_destilado]] §2, [[../02-architecture/adr/_moc]] ADR-0015.

---

## 8. Strategy (trial persona-aware, notificações, moderação cascade)

### Quando usar
- N variantes com mesma interface.

### Quando NÃO usar
- 1 variante só. Extrair interface é over-engineering.

### Exemplo Go — TrialPolicy (INV-017, BR-001)

```go
// domain/services/trial_policy.go
type TrialPolicy interface {
    TrialDays(persona Persona) int
    Eligible(user User) bool
}

// Strategy registry — sem if/else
var trialRegistry = map[Persona]TrialPolicy{
    PersonaSindico:   SindicoTrial{},
    PersonaEmpresa:   EmpresaTrial{},
    PersonaParceiro:  ParceiroTrial{},
    PersonaMorador:   NoTrialPolicy{},
}

type SindicoTrial struct{}
func (SindicoTrial) TrialDays(_ Persona) int { return 15 }
func (SindicoTrial) Eligible(u User) bool   { return u.HasRole(RoleSindico) }

func TrialDaysFor(p Persona) int {
    policy, ok := trialRegistry[p]
    if !ok { return 0 }
    return policy.TrialDays(p)
}
```

### Anti-pattern relacionado
- **AP-011** (if/else hell): `if p==síndico {15} else if p==empresa {7} else ...` → Strategy registry.

Referência: [[../13-research/tiktok/_destilado]] §3 (cascade moderation 2-stage).

---

## 9. Specification (busca Connect Me + filtros compostos)

### Quando usar
- Critérios combinados (categoria + região + reputação + disponibilidade).
- Testabilidade: `And`/`Or`/`Not`.

### Quando NÃO usar
- 1-2 filtros — `WHERE x = $1` direto.

### Exemplo Go

```go
// domain/specifications/company_spec.go
type CompanySpec interface {
    SQL() (where string, args []any)
}

type InCategorySpec struct{ Cat string }
func (s InCategorySpec) SQL() (string, []any) {
    return "category = ?", []any{s.Cat}
}

type NearbyWithinSpec struct{ Lat, Lng, KM float64 }
func (s NearbyWithinSpec) SQL() (string, []any) {
    return "ST_DWithin(location::geography, ST_MakePoint(?,?)::geography, ?)",
        []any{s.Lng, s.Lat, s.KM * 1000}
}

type MinReputationSpec struct{ Tier ReputationTier }

// composição
spec := And(
    InCategorySpec{"hidraulica"},
    NearbyWithinSpec{Lat: -23.5, Lng: -46.6, KM: 20},
    MinReputationSpec{Tier: ReputationOuro},
)
```

### Anti-pattern relacionado
- Specification que virou mini-ORM; se passou do trivial, query sqlc nomeada.

---

## 10. Anti-Corruption Layer (ACL) — SDKs externos

### Quando usar
- Sempre que SDK externo entra (Stripe, Mux, LiveKit, Zitadel, Twilio, SES, FCM, OpenAI Moderation).
- `infrastructure/providers/<name>/<name>_adapter.go` implementa `application/ports/IXxxGateway`.

### Quando NÃO usar
- Biblioteca stdlib ou pkg neutro (zerolog, ULID).

### Exemplo Go

```go
// application/ports/video_provider.go
type IVideoProvider interface {
    CreateDirectUpload(ctx context.Context, req DirectUploadReq) (*DirectUploadResult, error)
    IssueSignedPlayback(ctx context.Context, assetID string, ttl time.Duration) (string, error)
    CancelUpload(ctx context.Context, uploadID string) error
}

// infrastructure/providers/mux/mux_adapter.go
type MuxAdapter struct{ client *mux.Client; cb *gobreaker.CircuitBreaker }

func (a *MuxAdapter) CreateDirectUpload(ctx context.Context, req DirectUploadReq) (*DirectUploadResult, error) {
    result, err := a.cb.Execute(func() (any, error) {
        return a.client.Video.DirectUploads.Create(ctx, &mux.CreateDirectUploadRequest{
            NewAssetSettings: &mux.CreateAssetRequest{
                PlaybackPolicy: []mux.PlaybackPolicy{mux.PlaybackPolicy(req.PlaybackPolicy)},
            },
            CORSOrigin: req.Origin,
        })
    })
    if err != nil { return nil, fmt.Errorf("mux direct upload: %w", err) }
    mr := result.(*mux.DirectUpload)
    return &DirectUploadResult{ID: mr.ID, URL: mr.URL, AssetID: mr.AssetID}, nil
    // ← ZERO vazamento de tipo Mux pra camada superior
}
```

### Anti-pattern relacionado
- Stripe/Mux import no domain — blocker absoluto.
- Port com `mux.Asset` no retorno — vazamento. Devolva tipo neutro.

Referência: [[../13-research/netflix/_destilado]] §2.

---

## 11. Idempotency (webhooks inbound + commands retry-safe)

### Quando usar
- Todo webhook inbound (Stripe, Mux, LiveKit, Zitadel).
- Commands retentáveis por cliente/Saga.

### Quando NÃO usar
- Query — idempotente por natureza.

### Exemplo Go — Stripe webhook

```go
func (h *StripeWebhook) Handle(c contracts.Context) error {
    body, _ := io.ReadAll(c.Request().Body)

    // 1. HMAC antes de parse (INV-018, INV-093)
    if !verifyStripeHMAC(body, c.Header("Stripe-Signature"), h.secret) {
        return c.Status(400)
    }

    var evt stripe.Event
    if err := json.Unmarshal(body, &evt); err != nil {
        return c.Status(400)
    }

    // 2. Idempotency ledger Redis (INV-015)
    idemKey := fmt.Sprintf("webhook:stripe:%s", evt.ID)
    seen, err := h.redis.SetNX(c.Ctx(), idemKey, "1", 24*time.Hour).Result()
    if err != nil || !seen {
        return c.Status(200) // já processado — não retry
    }

    if err := h.cmd.Handle(c.Ctx(), ProcessStripeEvent{Event: evt}); err != nil {
        // 3. Remover ledger em erro permite retry
        h.redis.Del(c.Ctx(), idemKey)
        return c.Status(500)
    }
    return c.Status(200)
}
```

### Anti-pattern relacionado
- **AP-003** (webhook sem idempotência) — duplicate processing.
- **AP-023** (webhook sem HMAC) — MITM inject payload.
- Idempotency em DB sem TTL — cresce indefinidamente; Redis 24h basta (Stripe retry até 72h mas payload dedup cobre).

Referência: [[../13-research/google-arch/_destilado]] §5 (Pub/Sub at-least-once).

---

## 12. Circuit Breaker (`sony/gobreaker`)

### Quando usar
- Toda call a provider externo (Stripe, Mux, LiveKit, Zitadel, Twilio, SES, FCM, OpenAI).

### Quando NÃO usar
- Call interna (PG, Redis cluster) — retry + timeout direto.

### Exemplo Go

```go
// infrastructure/providers/stripe/stripe_adapter.go
var stripeCB = gobreaker.NewCircuitBreaker(gobreaker.Settings{
    Name:        "stripe",
    MaxRequests: 5,
    Interval:    30 * time.Second,
    Timeout:     30 * time.Second,
    ReadyToTrip: func(c gobreaker.Counts) bool {
        return c.Requests >= 20 && float64(c.TotalFailures)/float64(c.Requests) > 0.5
    },
})

func (a *StripeAdapter) CreateSubscription(ctx context.Context, req CreateSubReq) (*SubDTO, error) {
    result, err := stripeCB.Execute(func() (any, error) {
        return a.client.Subscriptions.New(ctx, mappers.ToStripeReq(req))
    })
    if err != nil { return nil, fmt.Errorf("stripe create sub: %w", err) }
    return mappers.FromStripeSub(result.(*stripe.Subscription)), nil
}
```

### Configuração canônica (de [[../13-research/netflix/_destilado]] §2)

| Provider | Abre quando | Racional |
|---|---|---|
| Stripe | >50% erro em 20 req / 30s | Confiável, tolera glitch curto |
| Mux upload | >30% erro / 30s | Mais sensível a degradação |
| LiveKit room create | >50% erro / 30s | Não idempotente, não retry |
| Zitadel introspect | >30% erro / 30s | Auth crítico, abrir cedo |

### Anti-pattern relacionado
- CB sem fallback — abre e user vê 500. Regra-ouro: fail-closed em auth/billing; fail-open em features cosméticas.
- CB por endpoint — explode cardinalidade. CB por **provider**.

---

## 13. Retry + Exponential Backoff + Jitter + DLQ

### Quando usar
- Providers idempotentes (Stripe com `Idempotency-Key`, SES, webhooks).
- Consumo de eventos NATS / outbox.

### Quando NÃO usar
- Não-idempotentes sem key (LiveKit room create).
- 4xx (cliente) — retry não resolve.

### Exemplo Go

```go
err := retry.Do(
    func() error { return a.adapter.Send(ctx, msg) },
    retry.Attempts(3),
    retry.Delay(200*time.Millisecond),
    retry.DelayType(retry.CombineDelay(retry.BackOffDelay, retry.RandomDelay)),
    retry.MaxDelay(5*time.Second),
    retry.RetryIf(func(err error) bool { return isRetryable(err) }),
)
if err != nil {
    h.dlq.Enqueue(ctx, msg, err) // stream events.dlq (NATS) ou outbox_dlq
}
```

### DLQ canônica
- Stream `events.dlq` (NATS JetStream) ou `outbox_dlq` (PG) com contador + último erro.
- Alerta Grafana quando `size > 10`.
- Processo manual de reprocess + post-mortem.

### Anti-pattern relacionado
- **AP-004** (sem retry strategy) — perda silenciosa.
- Retry sem jitter — sincroniza clientes (thundering herd).
- Retry infinito — cap em 3-5 tentativas; depois DLQ.

---

## 14. Outbox pattern (pré-NATS garantia)

### Quando usar
- **Sempre.** Default M1. Garante side-effects acontecem se e só se tx de domínio commitou (INV-095).

### Quando NÃO usar
- Nunca. Mesmo com NATS JetStream, outbox continua como fonte; publisher consome outbox.

### Exemplo Go — schema

```sql
CREATE TABLE outbox_events (
    id            BYTEA PRIMARY KEY,            -- UUIDv7 bytes
    tenant_id     BYTEA NOT NULL,
    topic         TEXT NOT NULL,                -- "institutional.timeline_entry_added"
    payload       JSONB NOT NULL,
    metadata      JSONB NOT NULL,               -- request_id, actor, ts
    created_at    TIMESTAMPTZ NOT NULL DEFAULT now(),
    dispatched_at TIMESTAMPTZ,
    attempts      INT NOT NULL DEFAULT 0,
    last_error    TEXT
);
CREATE INDEX ON outbox_events (dispatched_at, created_at)
    WHERE dispatched_at IS NULL;
```

Worker poll:

```sql
SELECT * FROM outbox_events
WHERE dispatched_at IS NULL AND attempts < 5
ORDER BY created_at LIMIT 100
FOR UPDATE SKIP LOCKED;
```

### Anti-pattern relacionado
- INSERT outbox + SEND direto em memória (`go publish(...)`) — send falha, tx commitou → side-effect perdido.
- Dispatcher bloqueando request — rodar em worker dedicado.

---

## 15. Observer (domain events)

### Quando usar
- Ação secundária não bloqueante ao write principal.
- Cross-BC communication (sempre via event, nunca import direto).

### Quando NÃO usar
- Ação síncrona crítica que não pode falhar separadamente — mantenha no use case/UoW.

### Exemplo Go

Event `TimelineEntryAdded` → subscribers: `NotificationDispatcher`, `OpenSearchIndexer`, `AuditLogger`, `WebhookOutDispatcher`.

Event `SubscriptionCreated` (billing) → subscriber `IdentityUserActivator` (ativa recursos no user).

### Anti-pattern relacionado
- Domain event contendo aggregate inteiro — acoplamento. Payload = IDs + campos necessários.
- Subscriber chamando use case cross-BC direto — viola Clean Arch. Emitir outro event.

---

## 16. Matriz resumo

| Pattern | Camada | Quando usar | Quando NÃO | Anti-pattern |
|---|---|---|---|---|
| DDD tático | domain | BC com invariantes | Tabela técnica | AP-009, AP-012 |
| Repository | domain iface + infra impl | Todo aggregate | Query pura | AP-003 |
| Factory NewX/ReconstructX | domain | Aggregate c/ invariante | VO trivial | Construtor público inválido |
| Value Object | domain | Conceito sem identidade | Com identidade | float64 em money/fração |
| CQRS | application | Todo use case | Event sourcing M1 | Command+Query mixed |
| UoW | application | Intra-BC atomic | Cruza BC/provider | Tx no handler |
| Saga | application | Provider externo | Cabe numa tx | Saga sem compensação |
| Strategy | application | N variantes | 1 variante | AP-011 (if/else hell) |
| Specification | domain | Filtros compostos | 1-2 filtros | Spec virou ORM |
| ACL provider | infra | Todo SDK externo | stdlib/pkg neutro | SDK no domain |
| Idempotency | infra | Webhook + retry-safe | Query | AP-003, AP-023 |
| Circuit Breaker | infra | Provider externo | Serviço interno | CB sem fallback |
| Retry+Backoff+DLQ | infra | Idempotente transiente | 4xx cliente | AP-004, sem jitter |
| Outbox | infra (PG) | Sempre (M1+) | — | Send direto após commit |
| Observer | domain+app | Side-effect não bloqueante | Ação crítica síncrona | Event com aggregate inteiro |

---

## 17. Links

- [[_moc]]
- [[CLEAN_CODE]]
- [[CLEAN_ARCH]]
- [[CODE_CALISTHENICS]]
- [[antipatterns]]
- [[../02-architecture/patterns]]
- [[../02-architecture/clean-arch-mapping]]
- [[../02-architecture/event-driven]]
- [[../02-architecture/adr/_moc]]
- [[../01-domain/invariants]]
- [[../13-research/netflix/_destilado]]
- [[../13-research/google-arch/_destilado]]
- [[../CLAUDE]]
