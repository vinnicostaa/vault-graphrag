---
title: Patterns aplicados — Master Síndico
type: guide
tags: [architecture, patterns, ddd, cqrs, saga, master-sindico, v2]
source: 13-research + 90-ingested/from-archive-specs-consolidation/architecture-refined.md
created: 2026-04-23
updated: 2026-04-23
aliases: [architecture-patterns]
---

# Patterns aplicados — Master Síndico

Cada padrão inclui: **quando usar**, **quando NÃO usar**, **exemplo concreto no master-sindico** e **anti-padrões relacionados**. Todo padrão deriva de research destilado (pasta `13-research` *(consolidada em [[../../../60-sources/master-sindico-research/_moc]])*) ou decisão explícita (ADRs em [[adr/_moc]]).

> Filosofia: padrão só entra quando tem 2+ usos reais (YAGNI §14 de [[../CLAUDE]]). Nada de abstração prematura.

---

## 1. DDD tático (aggregate + VO + domain event + domain service + specification + repository)

### Quando usar

- Todo BC do master-sindico (regra dura).
- Invariantes de negócio não-triviais (Σ fração ideal ≤ 100%, vote único por (agenda_item, voter), timeline imutável, quota persona-aware).

### Quando NÃO usar

- Tabelas puramente técnicas (sessions cache, rate limit counters, outbox_events) — CRUD simples via sqlc basta.
- Scripts one-off (migração de dado) — DDD é overhead.

### Exemplo no master-sindico

- Aggregate `Condominium` (root) encapsula `Unit[]` + enforcer `FracaoIdealValidator` (Σ ≤ 100% via PBT).
- Aggregate `Vote` com `VoteUniquenessEnforcer` e `VoteOption` VO.
- Domain service `ReputationCalculator` — determinístico, cross-entity (Review + Contract + Company).
- Specification `NearbyCompaniesInCategorySpec` — compõe filtros para query OpenSearch.

### Anti-padrões relacionados

- **Anemic Domain Model** — DTOs nus com getters/setters sem invariantes. Proibido.
- **God aggregate** — aggregate com 20 entidades. Refatorar em múltiplos aggregates via domain events.
- **Aggregate que importa framework** — domain zero import de gin/pgx/SDK.

### Referência

[[../13-research/linkedin/_destilado]] §5 (reputação determinística), [[../CLAUDE]] §3.1.

---

## 2. Repository (interface no domain, impl em infrastructure)

### Quando usar

- Todo aggregate precisa de repositório. Interface em `domain/repositories.go`, impl em `infrastructure/database/repositories/`.

### Quando NÃO usar

- Queries de leitura pura que já têm QueryHandler dedicado — `application/queries/` lê sqlc direto (via `IReadableStore`), evitando indireção desnecessária.

### Exemplo

```go
// domain/repositories.go
type VoteRepository interface {
    Save(ctx context.Context, v *Vote) error
    Get(ctx context.Context, tenantID, id ULID) (*Vote, error)
    ExistsForVoter(ctx context.Context, tenantID, agendaItemID, voterID ULID) (bool, error)
}

// infrastructure/database/repositories/vote_repo.go
type voteRepo struct{ q *sqlcgen.Queries }
func (r *voteRepo) Save(...) { ... } // impl
```

### Anti-padrões

- **Generic Repository `IRepository[T]`** — perde intenção, força métodos que não fazem sentido (ex: `DeleteAll` em timeline).
- **Repo que retorna tipo do ORM** — `SqlcUser` vazando para application.

---

## 3. Factory Method (`NewX` valida; `ReconstructX` hidrata)

### Quando usar

- Criação de aggregate com invariantes (validação obrigatória).
- Hidratação vinda do repo (mapper chama `ReconstructX`).

### Quando NÃO usar

- VOs triviais sem invariante — construtor literal serve.

### Exemplo

```go
// NewUser valida CPF, email, etc.
func NewUser(cpf CPF, email Email, ...) (*User, error) {
    if !cpf.IsValid() { return nil, ErrInvalidCPF }
    ...
}

// ReconstructUser vem do DB — confia
func ReconstructUser(id ULID, cpf CPF, email Email, ...) *User {
    return &User{id: id, cpf: cpf, email: email, ...}
}
```

### Anti-padrões

- **Construtor público exportado** que permite criar estado inválido.
- **Validação no setter** em vez de no factory — incentiva estado transitório inválido.

---

## 4. Value Object imutável

### Quando usar

- Conceito sem identidade (CPF, CNPJ, Money, FracaoIdeal, Email, Percentage, ULID).
- Comparação por valor (`==` em struct com campos comparáveis).

### Quando NÃO usar

- Conceito com identidade (User, Condominium, Vote) — é aggregate/entity.

### Exemplo

```go
type FracaoIdeal struct{ bps int64 } // basis points: 0-10000 → 0.00-100.00%
func NewFracaoIdeal(pct float64) (FracaoIdeal, error) {
    if pct < 0 || pct > 100 { return FracaoIdeal{}, ErrOutOfRange }
    return FracaoIdeal{bps: int64(pct * 100)}, nil
}
func (f FracaoIdeal) Add(o FracaoIdeal) FracaoIdeal { return FracaoIdeal{f.bps + o.bps} }
func (f FracaoIdeal) IsZero() bool { return f.bps == 0 }
```

### Anti-padrões

- **Float64 pra dinheiro/fração** — precisão instável. Use int64 centavos ou basis points.
- **VO com método `Set...`** — viola imutabilidade.

---

## 5. CQRS (Command Query Responsibility Segregation)

### Quando usar

- Todo use case — command ≠ query. Arquivos separados (`*_command.go`, `*_query.go`).
- Permite escalar leitura (materialized views, Redis, OpenSearch) sem refactor write side.

### Quando NÃO usar

- Event Sourcing full (M1/M2 não; avaliar em M3+ se LGPD/auditoria exigir além de timeline).

### Exemplo

- `CastVoteCommand` (write) + `GetQuorumLiveQuery` (read; lê via view materializada).
- `PublishAnnouncementCommand` (write) + `ListAnnouncementsQuery` (read; lê OpenSearch).

### Anti-padrões

- **CommandQueryMixed** — método `UpdateAndReturnList` que faz write + retorna lista nova.
- **Event sourcing prematuro** — em M1/M2 snapshots + append timeline bastam.

---

## 6. Unit of Work (UoW) — intra-BC

### Quando usar

- Um command toca 2+ aggregates no mesmo BC e precisa atomicidade.
- Outbox pattern: INSERT outbox_events na mesma tx do write principal.

### Quando NÃO usar

- Operação atravessa BC ou provider externo → usa **Saga**.

### Exemplo

```go
// application/ports/unit_of_work.go
type UnitOfWork interface {
    Run(ctx context.Context, fn func(ctx context.Context) error) error
}

// application/commands/cast_vote_command.go
func (h *CastVoteHandler) Handle(ctx, cmd) (*VoteDTO, error) {
    return WithTxReturning(ctx, h.uow, func(ctx) (*VoteDTO, error) {
        // INSERT vote + INSERT outbox_events numa tx
    })
}
```

### Anti-padrões

- **Abrir tx no handler HTTP** — transação pertence ao use case.
- **UoW que cruza provider externo** — Stripe/Mux não rollback; vira Saga.

Referência: [[adr/0015-uow-intra-saga-inter]].

---

## 7. Saga com compensação (inter-BC / provider externo)

### Quando usar

- Fluxo atravessa provider externo (Stripe, Mux, LiveKit).
- Fluxo cruza BCs e **precisa** de rollback semântico (compensação).

### Quando NÃO usar

- Tudo cabe em uma tx Postgres → UoW.

### Exemplo — `AssemblySaga` (criar assembleia + room LiveKit + Egress)

```
1. INSERT assembly (pg tx)
2. CREATE LiveKit room (via provider)     ─┐ se falhar → compensar: UPDATE assembly status=failed
3. INSERT egress_config (pg tx)             │
4. START Egress (via provider)            ─┤ se falhar → compensar: stop room LiveKit
5. PUBLISH timeline entry (pg tx + outbox)  │
```

Cada passo produz evento + registro de compensação. Se `GoBreaker` abre em (2) ou (4), saga roda compensação reversa.

### Anti-padrões

- **Distributed transaction 2PC** — Stripe/Mux não suportam.
- **Saga síncrona que bloqueia request HTTP** — saga é async via worker + state machine em pg; HTTP só aceita enqueue.
- **Saga sem compensação** — na prática é orquestração otimista. Documente compensação de cada passo.

Referência: [[../13-research/netflix/_destilado]] §2 (fallback strategy), [[adr/0015-uow-intra-saga-inter]].

---

## 8. Strategy (trial persona-aware, notificações multi-channel, moderação cascade)

### Quando usar

- N variantes de um algoritmo com mesma interface.
- Exemplo: `TrialCalculator` (síndico 15d / empresa 7d / parceiro 30d / morador sem trial).
- `NotificationDispatcher` (Email / SMS / Push — usuário escolhe canais).
- `ModerationStage` (heurística → LLM → humano).

### Quando NÃO usar

- 1 variante só. Extrair interface aí é over-engineering.

### Exemplo

```go
type TrialPolicy interface {
    TrialDays(persona Persona) int
    Eligible(u User) bool
}

type SindicoTrial struct{}
func (SindicoTrial) TrialDays(_ Persona) int { return 15 }

type EmpresaTrial struct{}
func (EmpresaTrial) TrialDays(_ Persona) int { return 7 }
```

### Anti-padrões

- **If/else hell** — "se persona==síndico retorna 15, senão se empresa retorna 7, senão..." → Strategy registry.
- **Strategy sem registry map** — voltar a if/else negando o pattern.

Referência: [[../13-research/tiktok/_destilado]] §3 (cascade 2-stage moderation).

---

## 9. Specification (busca Connect Me + filtros compostos)

### Quando usar

- Critérios de busca combinados (categoria + região + reputação + disponibilidade).
- Testabilidade: spec compõe como `And`/`Or`/`Not`.

### Quando NÃO usar

- Query simples com 1-2 filtros — `WHERE x = $1` direto.

### Exemplo

```go
type CompanySpec interface {
    SQL() (string, []any)
}
type InCategorySpec struct{ Cat string }
type NearbyWithinSpec struct{ Lat, Lng, KM float64 }
type TopReputationSpec struct{ MinTier ReputationTier }

spec := And(InCategorySpec{"hidraulica"}, NearbyWithinSpec{-23.5, -46.6, 20}, TopReputationSpec{Ouro})
```

### Anti-padrões

- **Specification que virou mini-ORM** — se já está complexo, vá direto em sqlc com query nomeada.

---

## 10. Anti-Corruption Layer (ACL) — SDKs externos

### Quando usar

- Sempre que um SDK externo entra no projeto (Stripe, Mux, LiveKit, Zitadel, Twilio, SES, FCM, OpenAI Moderation, Rekognition).
- Implementação: `infrastructure/providers/<name>/<name>_adapter.go` implementa `application/ports/IXxxGateway`.

### Quando NÃO usar

- Biblioteca stdlib ou pkg neutro (zerolog, ULID).

### Exemplo

```go
// application/ports/video_provider.go
type IVideoProvider interface {
    CreateDirectUpload(ctx context.Context, req DirectUploadReq) (*DirectUploadResult, error)
    IssueSignedPlayback(ctx context.Context, assetID string, ttl time.Duration) (string, error)
}

// infrastructure/providers/mux/mux_adapter.go
type MuxAdapter struct{ client *mux.Client }
func (a *MuxAdapter) CreateDirectUpload(ctx, req) (*DirectUploadResult, error) {
    // traduz req → chamada SDK Mux → traduz response → DirectUploadResult (sem vazar tipo SDK)
}
```

### Anti-padrões

- **Stripe import no domain** — proibido.
- **Port que tem `mux.Asset` no retorno** — vazamento; devolva tipo neutro.

Referência: [[../13-research/netflix/_destilado]] §2 (providers abstraídos).

---

## 11. Idempotency (webhooks inbound + commands retry-safe)

### Quando usar

- Todo webhook inbound (Stripe, Mux, LiveKit, Zitadel).
- Commands que podem ser retentados por cliente ou Saga.

### Quando NÃO usar

- Query pura — é idempotente por natureza.

### Exemplo

```go
// infrastructure/http/handlers/webhook_stripe.go
func (h *StripeWebhook) Handle(c *gin.Context) {
    body, _ := io.ReadAll(c.Request.Body)
    if !verifyHMAC(body, c.GetHeader("Stripe-Signature")) {
        c.Status(400); return
    }
    var evt stripe.Event
    _ = json.Unmarshal(body, &evt)

    idemKey := fmt.Sprintf("webhook:stripe:%s", evt.ID)
    seen, err := h.redis.SetNX(c, idemKey, "1", 24*time.Hour).Result()
    if err != nil || !seen {
        c.Status(200); return // já processado ou erro — devolve 200 pro Stripe não retry
    }
    // processa
    h.cmd.Handle(c.Request.Context(), ProcessStripeWebhookCommand{Event: evt})
    c.Status(200)
}
```

### Anti-padrões

- **Parse antes de HMAC** — MITM pode injetar payload; HMAC primeiro.
- **Idempotency em DB sem TTL** — cresce indefinidamente. Redis 24h suficiente para Stripe (retry até 72h mas idemkey dura enough).

Referência: [[../13-research/google-arch/_destilado]] §5 (Pub/Sub at-least-once + idempotency).

---

## 12. Circuit Breaker (sony/gobreaker em providers externos)

### Quando usar

- Toda call a provider externo (Stripe, Mux, LiveKit, Zitadel, Twilio, SES, FCM, OpenAI).
- Previne cascading failure + thundering herd.

### Quando NÃO usar

- Call interna (pg, Redis, OpenSearch cluster interno) — usa retry + timeout direto.

### Exemplo

```go
// infrastructure/providers/stripe/stripe_adapter.go
var stripeCB = gobreaker.NewCircuitBreaker(gobreaker.Settings{
    Name: "stripe",
    MaxRequests: 5,
    Interval: 30*time.Second,
    Timeout: 30*time.Second,
    ReadyToTrip: func(c gobreaker.Counts) bool {
        return c.Requests >= 20 && float64(c.TotalFailures)/float64(c.Requests) > 0.5
    },
})

func (a *StripeAdapter) CreateSubscription(ctx, req) (*Sub, error) {
    result, err := stripeCB.Execute(func() (any, error) {
        return a.client.CreateSubscription(req)
    })
    ...
}
```

### Configuração herdada de [[../13-research/netflix/_destilado]] §2

| Provider | Abre quando |
|---|---|
| Stripe | >50% erro em 20 req / 30s |
| Mux upload | >30% erro / 30s |
| LiveKit room create | >50% erro / 30s |
| Zitadel introspect | >30% erro / 30s |

### Anti-padrões

- **CB sem fallback** — abre e user vê 500. Defina: fail-closed (auth/billing) vs fail-open (cosmético).
- **CB por endpoint** — explode quantidade. CB por **provider**.

Referência: [[adr/0022-circuit-breaker-gobreaker]].

---

## 13. Retry + Exponential Backoff + Jitter + DLQ

### Quando usar

- Providers idempotentes (Stripe write com idempotency-key; Mux metadata; SES).
- Consumo de eventos NATS / outbox.

### Quando NÃO usar

- Operações não-idempotentes sem idempotency-key (ex: LiveKit room create).
- Erros 4xx (cliente-side) — retry não resolve.

### Exemplo

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
    h.dlq.Enqueue(ctx, msg, err)
}
```

### DLQ canônica

- Stream `events.dlq` (NATS) ou tabela `outbox_dlq` (pg) com contador de tentativas + último erro.
- Alerta Grafana quando size > 10.
- Processo manual de reprocess + post-mortem após investigação.

### Anti-padrões

- **Retry sem jitter** — sincroniza clientes (thundering herd).
- **Retry infinito** — cap em 3-5 tentativas; depois DLQ.

---

## 14. Outbox pattern (pré-messaging)

### Quando usar

- Sempre. É o default em M1. Garante que side-effects acontecem se e só se a tx de domínio commitou.

### Quando NÃO usar

- Quando você já tem NATS JetStream + effectively-once semantics confirmadas (M2+). Mesmo assim, **outbox continua** como fonte; NATS consome do outbox poller.

### Exemplo

```sql
-- migration
CREATE TABLE outbox_events (
    id          BYTEA PRIMARY KEY,         -- ULID bytes
    tenant_id   BYTEA NOT NULL,
    topic       TEXT NOT NULL,              -- ex: "institutional.timeline_entry_added"
    payload     JSONB NOT NULL,
    metadata    JSONB NOT NULL,             -- request_id, actor, ts
    created_at  TIMESTAMPTZ NOT NULL DEFAULT now(),
    dispatched_at TIMESTAMPTZ,
    attempts    INT NOT NULL DEFAULT 0,
    last_error  TEXT
);
CREATE INDEX ON outbox_events (dispatched_at, created_at)
    WHERE dispatched_at IS NULL;
```

Worker poll SQL:
```sql
SELECT * FROM outbox_events
WHERE dispatched_at IS NULL AND attempts < 5
ORDER BY created_at LIMIT 100 FOR UPDATE SKIP LOCKED;
```

### Anti-padrões

- **INSERT outbox + SEND direto em memory** — send falha, tx commitou → side-effect perdido. Sempre via tabela.
- **Dispatcher bloqueia request** — rodar em worker dedicado.

Referência: [[../13-research/linkedin/_destilado]] §1 (LinkedIn → "The Log"), [[event-driven]].

---

## 15. Observer (domain events)

### Quando usar

- Ação secundária não bloqueante ao write principal.
- Cross-BC communication (sempre via events, nunca import direto).

### Quando NÃO usar

- Ação síncrona crítica que não pode falhar separadamente — mantenha no use case.

### Exemplo

- `TimelineEntryAdded` → subscribers: `NotificationDispatcher`, `OpenSearchIndexer`, `AuditLogger`, `WebhookOutDispatcher`.
- `SubscriptionCreated` (billing) → subscriber: `IdentityUserActivator` (ativa recursos no user).

### Anti-padrões

- **Domain event que contém aggregate inteiro** — acoplamento excessivo. Payload = IDs + campos necessários.
- **Subscriber que faz chamadas cross-BC síncronas** — viola Clean Arch. Usa outro event.

---

## 16. Anti-pattern: Singleton

### Por que evitar

- Singletons escondem dependências e complicam testes.
- Em Go, `sync.Once` + variável package-level é tentador mas vira acoplamento global.

### Permitido (poucos casos)

- Logger package-level (`pkg/logger`) porque é ubíquo e imutável após init.
- Clock de produção — mas SEMPRE via interface `Clock` injetável em testes.

### Proibido

- Repository singleton global.
- Service/Handler package-level.

Injetar via DI explícito em `main.go` + `module.Register(deps)` é o padrão.

---

## 17. Matriz resumo

| Pattern | Camada | Quando | Não |
|---|---|---|---|
| DDD tático | domain | Todo BC com invariantes | Tabelas técnicas puras |
| Repository | domain iface + infra impl | Todo aggregate | Query pura (usa QueryHandler) |
| Factory `NewX/ReconstructX` | domain | Aggregate com invariante | VO trivial |
| Value Object imutável | domain | Conceito sem identidade | Com identidade (aggregate) |
| CQRS | application | Todo use case | Event sourcing M1/M2 |
| UoW | application | Intra-BC atomic | Cruza BC/provider (→ Saga) |
| Saga | application/cross-domain | Provider externo / inter-BC | Cabe em 1 tx |
| Strategy | application | N variantes com interface comum | 1 variante só |
| Specification | domain | Queries complexas compostas | 1-2 filtros |
| ACL (provider) | infrastructure | Todo SDK externo | stdlib/pkg neutro |
| Idempotency | infrastructure | Webhooks + commands retry-safe | Query pura |
| Circuit Breaker | infrastructure | Provider externo | Serviço interno |
| Retry + Backoff + DLQ | infrastructure | Idempotente, erros transientes | 4xx cliente |
| Outbox | infrastructure (pg) | Sempre — default M1 | — |
| Observer (events) | domain + app | Ação secundária / cross-BC | Ação crítica síncrona |
| Singleton | — | (evitar) | Repo, Service, Handler global |

---

## 18. Vizinhos

- [[clean-arch-mapping]] — camadas e regras
- [[c4-components]] — onde cada pattern se materializa
- [[event-driven]] — Observer + Outbox + NATS
- [[adr/0001-clean-architecture-ddd-cqrs]]
- [[adr/0015-uow-intra-saga-inter]]
- [[adr/0022-circuit-breaker-gobreaker]]
- [[research-inspirations]] — herança do research
