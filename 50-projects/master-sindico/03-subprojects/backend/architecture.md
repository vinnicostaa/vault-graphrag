---
title: Backend — Arquitetura (destilação Fase 8, código real)
type: spec
tags: [architecture, backend, clean-arch, ddd, cqrs, master-sindico, v2, destilacao-fase-8]
source: /home/vinni/Documentos/Projetos/Clientes/Joao/MasterSindico/Development/backend
created: 2026-04-23
updated: 2026-04-23
aliases: [backend-architecture]
---

# Backend — Arquitetura aplicada

Destilação **fiel ao código real** em `Development/backend/`, sem inferir
componentes que não existem no ramo `main`. Divergências com spec legada
aparecem com `DIVERGE-DO-LEGADO`.

---

## 1. Fontes consultadas (2026-04-23)

| Path | Trecho |
|---|---|
| `cmd/api/main.go:1-236` | Wire-up DI manual, graceful shutdown |
| `internal/core/contracts/module.go:1-78` | `IModule`, `HTTPRouter`, `Context`, `HandlerFunc` |
| `internal/core/contracts/repository.go:1-14` | `IRepository[T, ID]` |
| `internal/core/contracts/use_case.go:1-10` | `IUseCase[I, O]` |
| `internal/core/contracts/unit_of_work.go:1-12` | `IUnitOfWork.Run(ctx, fn)` |
| `internal/server/server.go:1-247` | Gin engine adapter + setupMiddlewares + RegisterModules |
| `internal/server/routes.go:1-94` | Health, metrics, /auth routes, wrapHandler |
| `internal/shared/middleware/authenticate.go` + `abac_engine.go` + `abac_rules.go:1-789` | Stack BeyondCorp |
| `internal/shared/providers/database/unit_of_work.go` + `postgresql.go` + `migrator.go` | pgx pool + goose embed |
| `internal/modules/billing/application/use_cases/checkout_subscription_saga.go:1-221` | Padrão Saga com compensação |

---

## 2. Camadas (Clean Architecture estrita)

```
+-------------------------------------------------------------+
|  infrastructure/ (adapters + transport + repositórios pgx)  |  importa application + domain
+-------------------------------------------------------------+
|  application/    (use cases CQRS + mappers + event_handlers)|  importa domain
+-------------------------------------------------------------+
|  domain/         (entities + value_objects + enums +         |  ZERO imports de framework
|                   repositories interfaces + events +         |
|                   providers interfaces)                      |
+-------------------------------------------------------------+
```

**Dependency rule** (aplicada manualmente — build quebra se violar):
`infrastructure → application → domain`. Domain **nunca** importa:
- `gin-gonic/gin`
- `jackc/pgx`
- `stripe-go`, `mux-go`, `livekit/server-sdk-go`
- `zerolog` (o domínio retorna sentinels; logger vive na application/infra)

**Verificação preventiva** em code review: grep `import (` dentro de
`domain/` não deve listar nenhum dos pacotes acima. (Sem check automatizado
no CI atual — gap reportável.)

---

## 3. Core — contratos neutros (`internal/core/`)

### 3.1 `IModule` — entrada de bounded context

```go
// internal/core/contracts/module.go
type IModule interface {
    Name() string
    Register(router HTTPRouter) error
    Bootstrap() error
}
```

Todo módulo registra rotas em `/api/v1/*` via `Register(router)`. O router
chega já com `Authenticate` aplicado — módulos só adicionam
`RequirePermission` por rota.

### 3.2 `HTTPRouter` — abstração sobre Gin

```go
type HTTPRouter interface {
    Group(relativePath string) HTTPRouter
    Handle(method, path string, handlers ...HandlerFunc)
    GET(path string, handlers ...HandlerFunc)
    POST(path string, handlers ...HandlerFunc)
    PUT(path string, handlers ...HandlerFunc)
    PATCH(path string, handlers ...HandlerFunc)
    DELETE(path string, handlers ...HandlerFunc)
    Use(middleware ...HandlerFunc)
}

type HandlerFunc func(ctx Context)
```

`GinRouter` (`internal/server/gin_adapter.go`) é o **único** adapter.
Trocar para Echo/Chi = reimplementar adapter; módulos/handlers não mudam.

### 3.3 `Context` — wrapper request-agnostic

```go
type Context interface {
    // Request
    Param(key string) string
    Query(key string) string
    GetHeader(key string) string
    DecodeJSON(dst any) error      // equiv. gin.ShouldBindJSON

    // Response
    JSON(code int, obj any)
    Status(code int)

    // Flow
    Abort()
    AbortWithJSON(code int, obj any)

    // Cookies
    Cookie(name string) (string, error)
    SetCookie(name, value string, maxAge int, path, domain string, secure, httpOnly bool)

    // Propagation
    SetValue(key string, value any)
    GetValue(key string) (any, bool)
    SetError(err error)            // interceptado por ErrorHandler middleware
    RequestContext() context.Context
    ClientIP() string
}
```

A-024 registra: `rawBodyReader` ainda é interface privada redefinida em
`billing/webhook_handler.go` e `content/video_handler.go`. Proposta é promover
`RawBody() io.Reader` ao `Context` canônico.

### 3.4 `IRepository[T, ID]` — base genérica (Go 1.18+ generics)

```go
type IRepository[T any, ID comparable] interface {
    FindByID(ctx context.Context, id ID) (*T, error)
    Save(ctx context.Context, entity *T) error
    DeleteByID(ctx context.Context, id ID) error
}
```

Interfaces concretas estão em `modules/*/domain/repositories/I*Repository.go`
e normalmente **não** embedam o genérico (autores preferiram assinaturas
explícitas por agregado — decisão de legibilidade).

### 3.5 `IUseCase[I, O]` — contrato de use case CQRS

```go
type IUseCase[I any, O any] interface {
    Execute(ctx context.Context, input I) (O, error)
}
```

**CQRS rígido**: 1 arquivo = 1 use case. Arquivos com sufixo `_use_case.go`
(singular) para ações atômicas; `_use_cases.go` (plural) quando o autor
agrupou commands de um mesmo aggregate (ex: `assembly_use_cases.go` contém
Publish/Start/End). Sempre Command struct distinto de Query struct — sem
polimorfismo via flags.

### 3.6 `IUnitOfWork` — transação atômica

```go
type IUnitOfWork interface {
    Run(ctx context.Context, fn func(ctx context.Context) error) error
}
```

**Impl concreta** em `internal/shared/providers/database/unit_of_work.go`.
- Abre `tx` via `pool.BeginTx`.
- Injeta `tx` em `ctx` via `WithTx(ctx, tx)`.
- Repositories usam `database.DBFromContext(ctx, fallbackPool)` para pegar `tx`
  quando dentro do UoW, `pool` fora.
- Rollback automático se `fn` retornar erro **ou** se `ctx` for cancelado.
- **DT-004 aberto**: sem `context.WithTimeout` interno.

---

## 4. Bootstrap (`cmd/api/main.go`) — 18 passos determinísticos

Transcrito do arquivo real, `main.go:34-235`:

```
 1. config.Load()                                  # Viper + Validate (CORS non-* em staging/prod)
 2. logger.New(logger.Config{...})                 # zerolog wrapper
 3. if cfg.Telemetry.Enabled:
      telemetry.Setup()                            # OTLP HTTP + Prometheus bridge
      closers += telemetryShutdown
 4. database.New(&cfg.Database, appLogger)         # pgxpool — PgxTracer auto-aplicado
 5. closers += pool.Close
 6. if Env.IsDev() || Env.IsStaging():
      database.Migrate(ctx, pool)                  # goose Up via embed.FS
 7. unitOfWork = database.NewUnitOfWork(pool, log)
 8. cache.New(&cfg.Redis, appLogger)               # go-redis/v9
 9. closers += cacheClient.Close
10. zitadelProvider = NewZitadelProvider(ctx, cfg) # OAuth2 introspection client
11. abacEngine = NewABACEngine(DefaultMatrix(), log)
      # >> aborta se matrix vazia (antipattern A1 do legado — "fail closed")
12. condominiumIDsProvider = institutional.NewCondominiumIDsProvider(pool)
13. identityModule = identity.NewModule(pool, uow, &cfg.App, log, abacEngine, condominiumIDsProvider)
14. billingModule, _ = billing.NewModule(pool, uow, cfg, abacEngine, log)
15. billingModule.EventPublisher().Subscribe(
      events.EventSubscriptionActivated,
      func(e) { log.Info("... pending ABAC cache invalidation") },
    )  # handler noop — invalidação real em Sprint 9.x pendente
16. srv, _ = server.New(cfg, pool, uow, cacheClient, zitadelProvider,
                        identityModule.SyncUserUseCase(), log, telemetryProviders)
17. srv.RegisterPublicWebhook("/webhooks/stripe", billingModule.WebhookHandler())
    institutionalModule = institutional.NewModule(pool, uow, abacEngine,
                                                   billingModule.FeatureAccessChecker(), log)
    commercialModule = commercial.NewModule(pool, uow, abacEngine,
                                             billingModule.QuotaConsumer(),
                                             institutionalModule.ValidationSubmitter(),
                                             institutionalModule.TimelinePublisher(),
                                             identityModule.UserModerator(), log)
    contentModule = content.NewModule(pool, abacEngine, log, cfg)
    srv.RegisterPublicWebhook("/webhooks/mux", contentModule.MuxWebhookHandler())
    assemblyModule = assembly.NewModule(pool, abacEngine, log, cfg)
18. srv.RegisterModules(identity, billing, institutional, commercial, content, assembly)
    srv.RegisterDocs()                             # Scalar UI em /docs
    signal.NotifyContext(SIGINT, SIGTERM)          # graceful shutdown ShutdownTimeout
```

**Observações**:
- **Ordem importa**: `condominiumIDsProvider` precede `identity` porque o Sync
  depende dele para popular `AuthUser.CondominiumIDs`.
- **Webhook público ANTES dos módulos**: `RegisterPublicWebhook` grava em
  `s.engine` raiz (fora de `/api/v1`), nunca passa por `Authenticate`. Registrar
  depois funciona também, mas main.go ordena para clareza.
- **Cleanup stack** (linhas 50-57): closers empilhados, executam em LIFO no
  defer — garante ordem inversa: telemetria → cache → DB pool.
- **Nada de DI container** (sem wire, fx, dig) — intencional, explícito,
  testável.

---

## 5. Middleware stack (BeyondCorp ordenado)

`server.setupMiddlewares()` + `server.setupRoutes()` + `RegisterModules`:

```
engine.Use(Tracing(serviceName))      # se OTEL_ENABLED=true — PRIMEIRO para cobrir spans dos outros
engine.Use(Recovery(log))             # captura panics, retorna 500
engine.Use(RequestID())               # header X-Request-ID ecoado
engine.Use(Logger(log))               # 1 log estruturado por request com trace_id + span_id
engine.Use(CORS(&cfg.CORS))           # origin-específico; * recusado em staging/prod
engine.Use(ErrorHandler(log))         # intercepta ctx.SetError(err) e serializa AppError

# /auth/*
authGroup.Use(RateLimiter(ctx, AuthRequestsPerMinute=20, AuthBurst=5))

# /api/v1/*
protected := engine.Group("/api/v1")
protected.Use(RateLimiter(ctx, RequestsPerMinute=60, Burst=10))  # apenas se cfg.RateLimit.Enabled
protected.Use(AuthMiddleware())                                   # Authenticate(zitadel, userSyncer, cache, log)

# Por rota, handlers array:
#   middleware.RequirePermission(abacEngine, action, resource, tenantExtractor, log) → handler
```

**Ordem validada em BeyondCorp audit 2026-04-22** (12/14 invariantes verdes,
AUDIT.md §2026-04-22). Pendências: A-023 (1-device audit log), A-039
(rate-limit per-userID).

### 5.1 `Authenticate` — Zitadel introspection + 1-device

1. Lê `Authorization: Bearer <token>` OU cookie `ms_session`.
2. Consulta cache Redis `ms:auth:{zitadelUserID}` (TTL 5min) antes de bater em Zitadel.
3. Miss: chama `zitadelProvider.Introspect(token)` → se inválido, 401.
4. UPSERT em `users` (via `IUserSyncer.Execute`) com device fingerprint + IP + role do claim `urn:zitadel:iam:org:project:roles`.
5. Invalida outras sessões do mesmo `user_id` (1-device per account).
6. Popula `AuthUser` em `ctx.Value("auth_user")`.
7. **DT-003 pendente**: sem circuit breaker — se Zitadel cai, cada request falha com 503.

### 5.2 `RequirePermission(abacEngine, action, resource, tenantExtractor, log)`

- Lê `AuthUser` do contexto.
- Se `role == "admin"` → **bypass global** (admin é privilégio transversal, não listado nas rules).
- Senão, consulta `abacEngine.Decide(abacCtx, action)`:
  - `action` já conhece suas `[]Rule` via `Matrix` (O(1) map lookup).
  - Rule concede se `AllowedRoles ∩ userRole ≠ ∅` AND
    `AllowedRoleTypes ∩ userRoleType ≠ ∅` (ou vazio) AND
    `userTier >= MinPlanTier` (hierarquia `PlanTierOrder`) AND
    (`!RequireTenantMatch` OR `ResourceTenantID ∈ AuthUser.CondominiumIDs`).
- Múltiplas rules para mesma action = OR.
- Deny = 403 `insufficient_permissions`.

### 5.3 `Tracing(serviceName)` — OTel span por request

- Wraps `otelgin.Middleware(serviceName)`.
- `PgxTracer` (em `pkg/telemetry/pgx_tracer.go`) instrumenta queries pgx
  com spans `db.query`, `db.batch`, `db.copy_from` (semconv v1.39).
- Logger enriquece campos com `trace_id` + `span_id` via `SpanFromContext`.
- Default dev: `OTEL_ENABLED=false` — providers são noop, overhead zero.

### 5.4 `RateLimiter(ctx, rpm, burst)` — token bucket por IP

- `sync.Map[string]*rate.Limiter` (golang.org/x/time/rate).
- Goroutine de cleanup respeita `ctx.Done()` (A-032 fechado).
- **A-039 aberto**: spec pede per-userID também — registrar em Sprint 10.

---

## 6. Bounded context canônico (`modules/{ctx}/`)

Cada módulo é auto-contido e implementa `contracts.IModule`:

```go
type Module struct {
    handler1        *routes.Handler1
    handler2        *routes.Handler2
    useCase1        application.UseCase1
    abacEngine      *middleware.ABACEngine
    log             logger.Logger
    // ... outros state necessários
}

var _ contracts.IModule = (*Module)(nil)

func NewModule(pool *pgxpool.Pool, uow contracts.IUnitOfWork, /* deps específicas */) *Module {
    repo1 := database.NewXRepository(pool)
    repo2 := database.NewYRepository(pool)
    useCase1 := usecases.NewUseCase1(repo1, repo2, uow, log)
    handler1 := routes.NewHandler1(useCase1)
    return &Module{...}
}

func (m *Module) Name() string { return "nome-do-modulo" }

func (m *Module) Register(router contracts.HTTPRouter) error {
    grp := router.Group("/prefix")
    grp.POST("/",
        middleware.RequirePermission(m.abacEngine, "ctx.resource.action", /* ... */),
        m.handler1.Create,
    )
    return nil
}

func (m *Module) Bootstrap() error {
    // Subscriptions de eventos, jobs cron etc
    return nil
}

// Métodos exportados DE SAÍDA — expostos a outros módulos via shared/types/ interfaces
func (m *Module) SomeAdapter() types.ISomeAdapter { return m.someAdapter }
```

### 6.1 Estrutura de pastas (padrão rígido)

```
modules/{ctx}/
├── domain/
│   ├── entities/
│   │   ├── agregate.go              # struct privado + NewX + ReconstructX + métodos
│   │   └── agregate_test.go         # unit + property-based quando crítico
│   ├── value_objects/
│   │   └── cnpj.go                  # imutável, auto-valida no construtor
│   ├── enums/
│   │   └── status.go                # tipo nomeado + IsValid()
│   ├── repositories/
│   │   └── i_x_repository.go        # interface, nunca impl
│   ├── events/
│   │   └── events.go                # Event structs + IEventPublisher (só billing+institutional)
│   └── providers/
│       └── i_x_provider.go          # interface para SDK externo do módulo
├── application/
│   ├── use_cases/
│   │   ├── create_x_use_case.go     # 1 command por arquivo
│   │   └── list_x_use_case.go       # 1 query por arquivo
│   ├── mappers/
│   │   └── x_mapper.go              # entity → DTO JSON
│   └── event_handlers/
│       └── x_y_handler.go           # handler de evento cross-aggregate
├── infrastructure/
│   ├── database/
│   │   ├── queries/
│   │   │   └── x.sql                # sqlc input
│   │   ├── generated/
│   │   │   └── x.sql.go             # sqlc output (NÃO editar)
│   │   └── x_repository_impl.go     # pgx adapter
│   ├── http/routes/
│   │   └── x_handler.go             # contracts.HandlerFunc
│   ├── providers/
│   │   └── stripe_gateway.go        # SDK externo isolado
│   └── events/
│       └── in_memory_publisher.go
└── module.go                        # IModule impl
```

**Exceções reais** observadas no código: em `identity/`, os arquivos estão
colapsados por simplicidade (18 arquivos sem sub-pastas). Em `institutional/`,
entities e use_cases estão nos padrões. Em `commercial/`, idem. Isso é
um **drift de convenção** — sinalizar em code review quando entidades novas
entrarem.

---

## 7. Patterns aplicados (instância em produção)

### 7.1 Saga com compensação (Checkout, UploadVideo, StartLiveSession)

**Exemplo**: `billing/application/use_cases/checkout_subscription_saga.go:30-221`:

```
1. validate(cmd)
2. plan := planRepo.FindByID()                    # read puro
3. customer := gateway.CreateCustomer(idemKey)    # passo externo compensável
4. subStripe := gateway.CreateSubscription(idemKey)
      # falha (3)→(4) não compensa CreateCustomer (reutilizável)
5. unitOfWork.Run(ctx, func(ctx):
      sub := NewSubscription(...)
      sub.LinkStripe(customer, subStripe, priceID)
      subscriptionRepo.Save(ctx, sub)              # tx local
      # falha → compensa (4) via gateway.CancelSubscription(subStripe.ID, AtPeriodEnd=false)
   )
6. eventPublisher.Publish(SubscriptionActivated{...})
```

Outros Sagas confirmados:
- `content.UploadVideoUseCase` — `CreateUploadURL` + `videoRepo.Create`; falha compensa com `CancelUpload(uploadID)` (A-027 fechado em `c32ab5a`).
- `assembly.StartLiveSessionUseCase` — `CreateRoom` + `GenerateToken`; falha compensa com `EndRoom(roomName)` (A-028 fechado em `c32ab5a`).

**Regra**: passos internos ao DB vão em UoW. Passos externos ao DB **sempre**
têm compensação explícita + idempotência no provider (idempotency key, HMAC,
unique constraint).

### 7.2 Unit of Work ponteado por `context.Context`

**Exemplo**: `commercial/vote_session_use_cases.go:102-123`:

```go
err := uc.unitOfWork.Run(ctx, func(ctx context.Context) error {
    sess, err := uc.voteRepo.FindSessionByIDForUpdate(ctx, cmd.SessionID) // SELECT...FOR UPDATE
    if err != nil { return err }
    if err := uc.voteRepo.CastVote(ctx, vote); err != nil { return err }  // INSERT + Increment atômicos
    return uc.voteRepo.IncrementVotesFor(ctx, sess.ID)
})
```

Por dentro, `voteRepo` extrai `tx` do ctx via
`database.DBFromContext(ctx, r.pool)` — repo não sabe se está em UoW.

### 7.3 Event Handler reativo (MasterPlanTimelineHandler)

**Arquivo**: `institutional/application/event_handlers/master_plan_timeline_handler.go`.

- Subscribe no `Bootstrap()` do módulo (no constructor).
- Consome `EventTimelinePublished{entryType, entryID, condominiumID}`.
- Chama `masterPlanRepo.ListInProgressByCondominium(condominiumID)`.
- Itera aplicando `activity.AdvanceStatusFromTimeline(entryType, entryID)` (idempotente — status terminal = no-op).
- Persiste batch via `SaveBatchAdvance` — única UPDATE com UNNEST de 3 arrays (A-020 fechado).

### 7.4 Cross-module sem import direto

Interfaces em `shared/types/` são a ponte:

```go
// shared/types/timeline_publisher.go
type ITimelinePublisher interface {
    Publish(ctx context.Context, cmd PublishTimelineCommand) error
}

// institutional/application/timeline_publisher_adapter.go
type TimelinePublisherAdapter struct { createUC *usecases.CreateTimelineEntryUseCase }
func (a *TimelinePublisherAdapter) Publish(ctx, cmd) error { ... }

// commercial/application/use_cases/closed_deal_use_cases.go
uc.timelinePublisher.Publish(ctx, types.PublishTimelineCommand{...})
// uc.timelinePublisher é types.ITimelinePublisher — commercial não importa institutional
```

### 7.5 Repository pattern + sqlc

- Interface em `domain/repositories/i_x_repository.go`.
- Implementação em `infrastructure/database/x_repository_impl.go`.
- SQL em `infrastructure/database/queries/x.sql`.
- Código gerado em `infrastructure/database/generated/x.sql.go`.
- Repositório traduz entre **row** (sqlc struct) e **entity** (domain) via `mappers.go`.

---

## 8. Config (`internal/shared/config/config.go`)

Carregada via Viper com:
- `.env` prioridade em dev
- Env vars do container em prod (Railway fornece `DATABASE_URL`, `REDIS_URL` automaticamente)
- `Validate()` falha startup se:
  - `CORS_ALLOWED_ORIGINS` contém `*` em staging/prod
  - `STRIPE_SECRET_KEY` ausente em staging/prod
  - `ZITADEL_KEY_PATH` não-existente em staging/prod

Estrutura:
```go
type Config struct {
    App       AppConfig         // Env, Name, Port, Host, SessionTTL, SecureCookies, Timeouts
    Database  DatabaseConfig    // Host, Port, User, Password, Name, SSLMode, Pool sizes, ConnMaxIdleTime
    Redis     RedisConfig
    Zitadel   ZitadelConfig     // Domain, Port, ProjectID, WebClientID, KeyPath, RedirectURI, etc
    Stripe    StripeConfig      // SecretKey, WebhookSecret, CustomerPortalReturnURL
    Mux       MuxConfig         // TokenID, TokenSecret, WebhookSecret, EnvKey
    Livekit   LivekitConfig     // Host, APIKey, APISecret
    Resend    ResendConfig
    Storage   StorageConfig     // MinIO / S3-compatible
    CORS      CORSConfig        // AllowedOrigins, Credentials
    RateLimit RateLimitConfig   // Enabled, RPM global + Auth RPM + Burst
    Telemetry TelemetryConfig   // Enabled, OTLPEndpoint, ServiceVersion, SampleRate, PrometheusEnabled
}
```

---

## 9. Deploy Railway

Dockerfile multi-stage (`backend/Dockerfile`):

```dockerfile
FROM golang:1.26-alpine AS builder
# ... go mod download; go build -trimpath -ldflags="-s -w" cmd/api + cmd/migrate

FROM alpine:3.21
RUN apk add --no-cache ca-certificates tzdata    # timezone America/Sao_Paulo para logs
COPY --from=builder /bin/api ./api
COPY --from=builder /bin/migrate ./migrate
COPY docker-entrypoint.sh ./entrypoint.sh
EXPOSE 8000
ENTRYPOINT ["./entrypoint.sh"]
CMD ["./api"]
```

`railway.toml`:

```toml
[build]
builder          = "DOCKERFILE"
dockerfilePath   = "Dockerfile"

[deploy]
releaseCommand       = "/app/migrate up"
startCommand         = "/app/api"
healthcheckPath      = "/health"
healthcheckTimeout   = 30
restartPolicyType    = "ON_FAILURE"
restartPolicyMaxRetries = 3
```

**Entry point** (`docker-entrypoint.sh`): se `ZITADEL_KEY_JSON` env existe,
escreve em `/tmp/zitadel-key.json` antes de lançar `api`. Permite injetar
a service account key sem commitar arquivo.

**Env vars obrigatórias** comentadas no `railway.toml` linhas 18-81 — divididas
em: Database, Redis, App, Zitadel, Stripe, Mux, Resend, Storage, Livekit, CORS.

---

## 10. Observabilidade (OTel + Prometheus)

Sprint 7.9 (concluído 2026-04-21):

```
pkg/telemetry/
├── telemetry.go             # Setup(ctx, cfg) → TracerProvider + MeterProvider + shutdown
└── pgx_tracer.go            # PgxTracer emite spans db.query | db.batch | db.copy_from

internal/shared/middleware/
├── tracing.go               # otelgin.Middleware + TraceIDFromContext helper
└── logger.go                # enriquece log com trace_id + span_id

internal/server/
└── routes.go                # GET /metrics → gin.WrapH(prometheusHandler) quando habilitado
```

Config via env:

| Env | Default | Comentário |
|---|---|---|
| `OTEL_ENABLED` | `false` (dev) | true em staging/prod |
| `OTEL_EXPORTER_OTLP_ENDPOINT` | — | OTLP HTTP |
| `OTEL_SERVICE_VERSION` | — | ex: `git sha curto` |
| `OTEL_SAMPLE_RATE` | 1.0 dev/stg; 0.1 prod | |
| `OTEL_PROMETHEUS_ENABLED` | false (dev) | expõe `/metrics` no engine |

---

## 11. Eventos (in-process, NATS JetStream gap)

`billing/domain/events/events.go` define:

```go
type EventType string

const (
    EventSubscriptionActivated      EventType = "subscription.activated"
    EventSubscriptionCanceled       EventType = "subscription.canceled"
    EventSubscriptionTrialExpired   EventType = "subscription.trial_expired"
)

type IEventPublisher interface {
    Publish(ctx context.Context, event Event) error
    Subscribe(t EventType, handler func(Event) error)
}

type Event interface { EventType() EventType; OccurredAt() time.Time }
```

Implementação única: `infrastructure/events/in_memory_publisher.go` —
`sync.Map[EventType][]handler`, chamada síncrona.

**Gap vs STEERING**: NATS JetStream não instalado. Outbox pattern **não
existe** — se o processo cai entre commit local e publish, evento é perdido.
Promover Sprint 10 task "Outbox + NATS publisher" (ver [[tasks]]).

---

## 12. Ligações

- Padrões em detalhe: [[patterns]]
- Requisitos não-funcionais (observability, webhooks, rate limit): [[requirements]]
- Segurança: [[security]]
- Tasks Sprint 10+: [[tasks]]
- MOC: [[_moc]]
- STATE vivo: [[../../STATE]]
- Plan raiz: [[../../02-architecture/clean-arch-mapping]]
