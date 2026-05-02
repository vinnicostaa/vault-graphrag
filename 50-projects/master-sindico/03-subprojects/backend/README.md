---
title: Backend Master Síndico — README (destilação Fase 8, código real)
type: spec
tags: [subproject, backend, master-sindico, v2, go, destilacao-fase-8]
source: /home/vinni/Documentos/Projetos/Clientes/Joao/MasterSindico/Development/backend
created: 2026-04-23
updated: 2026-04-23
aliases: [backend-readme, backend-overview]
---

# Backend — Master Síndico (sub-projeto `backend/`)

> **Fonte canônica**: este README reflete **o código Go real em produção** em
> `Development/backend/` (ramo `main`, snapshot 2026-04-22). Divergências com
> `vault/50-projects/master-sindico` legado ficam sinalizadas explicitamente
> com tag `DIVERGE-DO-LEGADO` e são promovidas a D-### em [[../../STATE]].

API REST em Go 1.26 — Clean Architecture estrita + DDD tático + CQRS rígido +
BeyondCorp adaptado. Single binary, multi-tenant row-based, integra Stripe +
Mux + Livekit + Zitadel + Resend + MinIO + NATS + Redis + PostgreSQL 18.

**Marcos vivos**:
- **Marco 1 (MVP contratual — master-sindico cliente)**: 2026-05-08.
- **Sprint ativo**: 9 (Integrations) — 99% fechado; 2 tasks abertas (9.10 templates parciais, 9.19 Zitadel managed).
- **Sprint 10** (acumulado pós-MVP): A-023 (audit log 1-device) + A-039 (rate-limit per-userId).

---

## 1. Fontes consultadas (path:linha, 2026-04-23)

| Fonte | Uso |
|---|---|
| `backend/CLAUDE.md:1-97` | Orientação enxuta + stack canônico + comandos |
| `backend/AGENTS.md:1-234` | Contrato do agente + estrutura + princípios inegociáveis |
| `backend/.kiro/SESSION_CHARTER.md:1-309` | Ordens da sessão autônoma 2026-04-22 + F1-F33 |
| `backend/.kiro/STATE.md:1-400` | Decisões vivas (D-001..D-030) + dívidas (DT-001..DT-010) |
| `backend/.kiro/AUDIT.md:1-309` | Findings A-017..A-039 abertos/fechados |
| `backend/cmd/api/main.go:1-236` | Wire-up DI manual, ordem de inicialização |
| `backend/internal/core/contracts/*.go` | `IModule`, `HTTPRouter`, `Context`, `IRepository`, `IUseCase`, `IUnitOfWork` |
| `backend/internal/shared/types/auth_user.go:1-91` | `AuthUser` + `PlanTierOrder` + `HasMinimumPlanTier` |
| `backend/internal/shared/middleware/abac_rules.go:1-789` | Matriz ABAC (6 bounded contexts, 70+ actions) |
| `backend/internal/shared/providers/database/migrations/*.sql` | 39 migrations goose |
| `backend/internal/modules/*/domain/entities/*.go` | 38 entidades de domínio |
| `backend/internal/modules/*/application/use_cases/*.go` | ~48 arquivos de use cases (commands e queries separados) |
| `backend/internal/modules/*/module.go` | Registro de rotas (≈ 115 endpoints) |
| `backend/go.mod:1-210` | Deps: Gin 1.12.0, pgx/v5, goose/v3 3.27, stripe/v82, livekit 1.45, mux-go, zitadel/v3, rapid v1.2 |
| `backend/sqlc.yaml:1-140` | 6 packages sqlc (um por módulo), overrides de enums |
| `backend/Dockerfile:1-45` + `railway.toml:1-81` | Deploy Railway multi-stage + release migrate |

---

## 2. Stack real confirmada

| Camada | Tech (versão exata no go.mod) | Abstração no código |
|---|---|---|
| Linguagem | **Go 1.26.2** | — |
| HTTP framework | **Gin 1.12.0** | `contracts.HTTPRouter`, `contracts.Context`, `contracts.HandlerFunc` (adapter único em `internal/server/`) |
| Driver PG | **pgx/v5 5.9.1** | `contracts.IUnitOfWork` + `database.DBFromContext` |
| Query tipada | **sqlc v2** (6 packages) | Um output por módulo: `identitydb`, `billingdb`, `institutionaldb`, `commercialdb`, `contentdb`, `assemblydb` |
| Migrations | **goose/v3 3.27** | `internal/shared/providers/database/migrator.go` via `embed.FS` — 39 arquivos `internal/shared/providers/database/migrations/` |
| Auth OIDC | **zitadel-go/v3 3.29** + **zitadel/oidc/v3 3.45** | `auth.IZitadelProvider` + `auth.OIDCHandler` (PKCE) |
| Cache | **go-redis/v9 9.18** | `providers/cache/Cache` interface |
| Log estruturado | **zerolog 1.35** | `pkg/logger.Logger` wrapper (sem `SetGlobalLevel`) |
| Billing | **stripe-go/v82 82.5** | `IPaymentGateway` em `billing/domain/providers/` |
| Vídeo VOD | **mux-go 1.1.1** | `IVideoProvider` em `content/domain/providers/` |
| Live WebRTC | **livekit/protocol 1.45 + server-sdk-go/v2 2.16** | `ILiveProvider` em `assembly/domain/providers/` |
| Email | **resend-go/v2 2.28** | `IEmailProvider` em `shared/providers/email/` |
| Storage | **minio-go/v7 7.0.100** | `IStorageProvider` em `shared/providers/storage/` |
| Observabilidade | **otel v1.43 + otelgin v0.62 + prometheus v1.23** | `pkg/telemetry` + `middleware.Tracing` + pgx `PgxTracer` |
| Property-based testing | **pgregory.net/rapid v1.2.0** | 3 suites PBT + 1 abac_matrix table-driven |
| Integration testing | **testcontainers-go v0.42** | `internal/shared/testutil/pg.go` |
| NATS JetStream | **inegociável no STEERING** | Não presente no `go.mod` — providers atuais usam `InMemoryPublisher` (`billing/infrastructure/events/` + `institutional/infrastructure/events/`). Gap registrado em tasks.md. |
| SMS | **Twilio** — | Interface `ISMSProvider` em `shared/providers/sms/` com adapter `twilio_provider.go`; SDK não constante do go.mod (gap: stub). |
| Push | **FCM** — | `IPushProvider` com stub `fcm_provider.go` |
| PDF | — | `IPDFGenerator` com stub `gotpl_provider.go` |
| CEP | **ViaCEP** | `ICEPLookup` com adapter `viacep_provider.go` |
| Deploy | **Railway** (Dockerfile multi-stage + `railway.toml` + release `/app/migrate up`) | — |

**DIVERGE-DO-LEGADO** (atualizado 2026-04-24 — D-120): Ent/Casbin substituídos por ABAC engine próprio em `internal/shared/middleware/abac_engine.go`. Uber-FX substituído por wire-up DI manual em `cmd/api/main.go` (236 linhas). ECS/SES descartados em favor de Railway+Resend (D-067).

**Search (D-120 — revoga D-067 PG tsvector)**: `ISearchProvider` real desde M1, com adapter inicial **OpenSearch** ou **Meilisearch** (decisão Sprint 10 via ADR-0042 dual-check). Busca é vital pro produto (Vizinhança, Connect Me, Banco Talentos, biblioteca de atas, prestadores) — não pode ser tsvector subdimensionado. Backend atual ainda usa `ILIKE` em sqlc — gap a fechar em Sprint 10 (T-### a criar).

---

## 3. Arquitetura de alto nível

```
cmd/api/main.go                  # 236 linhas — entry, bootstrap, DI manual, graceful shutdown
internal/
  core/
    contracts/                   # Module, HTTPRouter, Context, IRepository, IUseCase, IUnitOfWork
    errors/                      # AppError + sentinels ErrNotFound/Forbidden/Conflict/Validation/Internal
  modules/                       # 6 bounded contexts — cada um isolado, nunca import cruzado direto
    identity/
    billing/
    institutional/
    commercial/
    content/
    assembly/
  server/                        # ÚNICO lugar que importa gin — GinRouter + GinContext adapters
  shared/
    config/                      # Viper + Validate() com deny-CORS-wildcard em staging/prod
    middleware/                  # Recovery, RequestID, Logger, CORS, ErrorHandler, RateLimiter,
                                 # Authenticate, RequirePermission (ABAC), Tracing
    pagination/                  # pagination.FromQuery(ctx) — cap 100, PBT 1000 casos
    providers/
      auth/                      # ZitadelProvider + OIDCHandler (PKCE, gorilla/securecookie)
      cache/                     # Redis adapter
      database/                  # pgxpool, Migrator (goose embed.FS), UnitOfWork, PgxTracer
      email/                     # Resend adapter
      storage/                   # MinIO adapter
      sms/                       # Twilio stub
      push/                      # FCM stub
      pdf/                       # gotpl stub
      cep/                       # ViaCEP adapter
    testutil/                    # testcontainers-go PG helper + TruncateAll
    types/                       # AuthUser, IUserSyncer, IQuotaConsumer, IValidationSubmitter,
                                 # ITimelinePublisher, IUserModerator, ITrialChecker,
                                 # ICondominiumIDsProvider — contratos cross-module
pkg/
  logger/                        # zerolog wrapper
  safecast/                      # Int32/Int16/Int32FromInt64 com clamp [0, MaxIntN] — resolve G115
  telemetry/                     # OTel Setup + PgxTracer + noop fallback
  utils/                         # UUIDv7
```

**Dependency rule** enforçado: `infrastructure → application → domain`;
domain zero import de framework (verificação manual em code review, build
breaks se violar).

Cada bounded context tem layout idêntico:

```
{ctx}/
├── domain/
│   ├── entities/        # estado privado, factories NewX + ReconstructX, métodos de domínio
│   ├── value_objects/   # imutáveis, auto-validados (CPF, CNPJ, Email, Address, BimonthlyPeriod)
│   ├── enums/           # tipos nomeados com IsValid() — evita strings mágicas
│   ├── repositories/    # interfaces I*Repository — NUNCA implementações
│   ├── events/          # Event + IEventPublisher (billing + institutional só)
│   └── providers/       # interfaces I* para SDK externo do próprio módulo
├── application/
│   ├── use_cases/       # 1 arquivo = 1 use case (CQRS); inputs = Command/Query structs
│   ├── mappers/         # entidade → DTO JSON
│   └── event_handlers/  # MasterPlanTimelineHandler, etc
├── infrastructure/
│   ├── database/
│   │   ├── queries/     # *.sql (sqlc input)
│   │   ├── generated/   # *.sql.go (sqlc output — nunca editado à mão)
│   │   └── *_impl.go    # adapter pgx → interface do domain
│   ├── http/routes/     # handler.go (contracts.HandlerFunc)
│   ├── providers/       # MuxProvider, LivekitProvider, StripeGateway, PublicIDGeneratorAdapter
│   └── events/          # InMemoryPublisher (in-process pub/sub)
└── module.go            # implementa contracts.IModule — Register(router) + Bootstrap()
```

---

## 4. Módulos em produção (6 bounded contexts)

Verificação via `find internal/modules/*/module.go`. Todos os 6 registram
rotas em `/api/v1/*` pós-Authenticate e possuem handler de erro global.

### 4.1 `identity` (Sprint 1)

- **Responsabilidade**: espelho Zitadel no DB local + 1-device + moderação.
- **Aggregates**: `User` (20 campos), `Session` (device fingerprint + IP + expiresAt).
- **Value objects**: `CPF`, `Email`.
- **Use cases**: `SyncUserUseCase` (UPSERT no login, invalida outras sessões), `ModerateUserUseCase` (warning | suspension | ban).
- **Endpoints**: `GET /api/v1/auth/me`. Auth/login/callback/logout são rotas públicas de OIDC em `/auth/*` (não `/api/v1/*`).
- **Cross-module**: expõe `IUserSyncer` (consumido pelo middleware Authenticate) + `IUserModerator` (consumido pelo commercial harassment).
- **Gap A-023**: 1-device change sem audit log comparativo.

### 4.2 `billing` (Sprint 1.3-1.6)

- **Responsabilidade**: Plans universais (role × tier), Subscriptions, Trial, Quotas, Stripe webhook.
- **Aggregates**: `Plan` (id, role, tier, priceMonthly/Yearly em centavos int64, trialDays, stripeProductID/PriceMonthlyID/PriceYearlyID), `Subscription` (status trialing|active|past_due|canceled|paused|incomplete|incomplete_expired), `FeatureUsage` (quotas por janela [start,end)).
- **Tiers universais (D-032 do vault legado e ADR-0032)**: `trial` → `base` → `plus` → `pro`. `shared/types.PlanTierOrder` é o **único** source of truth para comparação hierárquica.
- **Roles** (enum DB `user_role`): `syndic | resident | enterprise | marketing | local_company | admin`.
- **Use cases**: `CheckoutSubscriptionSaga` (orquestra Stripe + DB com compensação), `HandleStripeWebhookUseCase` (idempotente via evt_ID), `CheckFeatureAccessUseCase`, `ConsumeQuotaUseCase`, `ListPlansUseCase`, `GetMySubscription*UseCase`.
- **Providers**: `IPaymentGateway` → `StripeGateway` (HMAC-SHA256 verify before parse).
- **Endpoints**: `GET /billing/plans`, `GET /billing/subscription/me`, `POST /billing/checkout`, `POST /billing/customer-portal` + webhook público `POST /webhooks/stripe`.
- **Services**: `QuotaLimits` (matriz feature × role × tier → int64; `QuotaUnlimited = -1`), `TrialRules`.
- **Events**: `SubscriptionActivated`, `SubscriptionCanceled`, `SubscriptionTrialExpired` (domínio in-memory; NATS JetStream planejado).

### 4.3 `institutional` (Sprints 2-3 + 6.6)

- **Responsabilidade**: Condomínio, Unit, Membership, SyndicMandate, Timeline imutável, Master Plan, Events, Announcements, Polls, ManagementAssessment, Compliance, ValidationHub, Dashboard.
- **Aggregates** (14): `Condominium`, `Unit` (❌ sem `unit_type`/`fracao_ideal` — **gap crítico**), `Membership`, `SyndicMandate`, `TimelineEntry` (append-only), `MasterPlanActivity` (status evolui automaticamente via handler de TimelinePublished), `Event`, `EventResponse`, `Announcement` (+ reads com UNIQUE), `Poll`, `ManagementAssessmentResponse` (bimestral), `Compliance` (termos + declaração anual), `ValidationItem` (ponte com commercial).
- **Value objects**: `Address` (CEP + rua + número + bairro + cidade + estado), `BimonthlyPeriod`.
- **Enums**: `CondominiumType` (6 — residencial/comercial/misto/shopping_center/galeria_comercial/edificio_garagem), `MandateType` (3), `MandateStatus` (3), `MembershipStatus` (3), `MembershipEndReason` (4), `TimelineEntryType` (7 incl. `assembly_result`), `EventType` (13), `AnnouncementType` (8), `AnnouncementPriority` (4), `PollType` (3), `PollStatus` (2), `ValidationStatus` (4), `MasterPlanStatus` (7 — no arquivo dedicado `enums/master_plan.go`).
- **Eventos domínio**: `EventTimelinePublished` (consumido pelo `MasterPlanTimelineHandler` para advance automático de status).
- **Use cases (~18)**: Create/List Condominium/Unit/Timeline/MasterPlan/Event/Announcement/Poll/Assessment/Compliance + Membership register/end + Mandate end/transfer + Validation read/decide + Dashboard.
- **Endpoints**: ~36 registrados no `condominiums.Group("/condominiums")` + 1 standalone `GET /master-plan/catalogs` público (retorna 26 áreas + 26 tipos de atividade + 9 riscos + 8 próximas ações).
- **Cross-module**: expõe `IValidationSubmitter` + `ITimelinePublisher` + `ICondominiumIDsProvider`.

### 4.4 `commercial` (Sprint 4 + 6.7 + 6.8)

- **Responsabilidade**: Company, ConnectMe, ExecutionRecord, ClosedDeal, HarassmentReport, Proposal + SupplierVoteSession, EmpresaSindicaLink, CompanyEvaluation, PostDealComunicado.
- **Aggregates (10)**: `Company` (state machine pending_verification → active → suspended → blocked), `CompanyUser` (admin/operator), `ConnectMeRequest` (4 fluxos), `ExecutionRecord` (draft → submitted → validated/rejected/adjustment_requested), `ClosedDeal` (dupla assinatura — empresa → síndico), `HarassmentReport` (pending_review → reviewed + admin action), `Proposal`, `SupplierVoteSession` (quórum simple/absolute/qualified), `EmpresaSindicaLink` (invite token UUIDv7 + permissions 5 bool flags), `CompanyEvaluation` (UNIQUE company_id+evaluator+condominium), `PostDealComunicado`.
- **Value object**: `CNPJ` (com `Masked()` para log — A-004 corrigido).
- **Enums**: `CompanyStatus` (4), `CompanyRole` (2), `ConnectMeFlow` (4), `ConnectionStatus` (4), `ExecutionRecordStatus` (5), `ClosedDealStatus` (5), `HarassmentReportStatus` (2), `HarassmentAdminAction` (3 — warning/suspension/ban), `ProposalStatus` (5), `SupplierVoteQuorumType` (3), `SupplierVoteSessionStatus` (4), `SupplierVoteValue` (3 — for/against/abstain), `EmpresaSindicaLinkStatus` (3).
- **Use cases (~13)**: CompanyRegister/Activate/CreateUser/RemoveUser, ConnectMe Create/Respond, Execution Create/Submit, ClosedDeal Create/Submit/CompanySign/SyndicSign, Proposal CRUD + Submit/Review/Withdraw, VoteSession Open/Cast/Close, EmpresaSindica Invite/Accept/UpdatePermissions/Revoke, Harassment Report/Review, CompanyEvaluation Submit/List, Comunicado Create/Validate/List.
- **Endpoints**: ~41 sob grupos `/companies`, `/connect-me`, `/executions`, `/closed-deals`, `/proposals`, `/empresa-sindica`, `/harassment-reports`.
- **Cross-module**: consome `IQuotaConsumer`, `IValidationSubmitter`, `ITimelinePublisher`, `IUserModerator`.

### 4.5 `content` (Sprint 5 + 9)

- **Responsabilidade**: Vídeos VOD via Mux + MarketingAgencyLink.
- **Aggregates**: `Video` (12 tipos pt-BR), `MarketingAgencyLink`.
- **Enums**: `VideoType` (12), `VideoStatus` (4), `MarketingAgencyLinkStatus` (3).
- **Providers**: `IVideoProvider` → `MuxProvider` (CreateUploadURL, CancelUpload, GetAsset, GetPlaybackURL, GetThumbnailURL, DeleteAsset) + `ILiveStreamProvider` (não utilizado — Mux live cancelado em favor de Livekit assembly).
- **Use cases (~9)**: UploadVideo (Saga com compensação `CancelUpload` — A-027 fechado), GetVideo, ListVideos, HideVideo, UnhideVideo, UpdateVideoVotingAuth, Invite/Accept/Update/Revoke/ListMarketingAgency.
- **Endpoints**: 11 sob `/content/videos` + `/content/marketing-agencies`.
- **Webhook público**: `POST /webhooks/mux` (A-022 fechado — rota fora de `/api/v1`).

### 4.6 `assembly` (Sprint 7)

- **Responsabilidade**: Ciclo de vida completo de assembleia — rascunho → publicada → em_andamento → encerrada (+ cancelada).
- **Aggregates (7)**: `Assembly`, `AgendaItem` (bloqueia após edital via `Lock()`; fração ideal 0-100 validada — PBT `agenda_item_pbt_test.go`), `Vote` (UNIQUE(agenda_item_id, voter_id) — A-025 fechado), `AttendanceRecord`, `Proxy`, `ScienceRecord`, `Minutes`.
- **Enums**: `AssemblyType` (3), `AssemblyModality` (3), `AssemblyStatus` (5), `QuorumType` (3 — simple/qualified/unanimous), `AgendaItemType` (5), `AgendaItemStatus` (5), `VoteType` (3), `PresenceType` (3), `ProxyStatus` (4).
- **Providers**: `ILiveProvider` → `LivekitProvider` (retry backoff 100/300/900ms + reconciliação NotFound — A-033/A-034 fechados).
- **Use cases**: Assembly Create/Publish/PublishEdital/Start/End, AgendaItem Add/OpenVoting/CloseVoting/Withdraw, Vote Cast/List, Attendance Checkin/MarkAbsence/MarkReturn, Proxy Create/Validate/List, Science Acknowledge/List, Minutes Generate/Get, Live Start (Saga com `EndRoom` compensation — A-028 fechado)/Join/End/ListParticipants.
- **Endpoints**: ~25 sob `/condominiums/:condominium_id/assemblies`.

---

## 5. Endpoints — inventário canônico (115 rotas extraídas via grep em `module.go`)

Sistema integra 4 camadas de rota:

```
Camada                  | Auth aplicada?                | Origem
------------------------|--------------------------------|-----------------------------------
/health, /health/ready  | não                            | server/routes.go
/metrics (Prometheus)   | não                            | server/routes.go (quando OTEL on)
/auth/login|callback|logout | rate-limit /auth + OIDC   | server/routes.go
/webhooks/stripe        | HMAC Stripe — SEM Authenticate | main.go RegisterPublicWebhook
/webhooks/mux           | HMAC Mux    — SEM Authenticate | main.go RegisterPublicWebhook
/api/v1/*               | RateLimiter global → Authenticate → RequirePermission (ABAC) | módulos
/docs (Scalar/OpenAPI)  | não                            | server/docs.go
```

### 5.1 Identity (1 endpoint)

| Método | Path | Permission (ABAC) |
|---|---|---|
| GET | `/api/v1/auth/me` | `identity.user.view_me` |

### 5.2 Billing (4 + 1 webhook)

| Método | Path | Permission |
|---|---|---|
| GET | `/api/v1/billing/plans` | público-dentro-de-api (sem ABAC específica; qualquer autenticado) |
| GET | `/api/v1/billing/subscription/me` | `billing.subscription.view_own` |
| POST | `/api/v1/billing/checkout` | `billing.checkout.create` |
| POST | `/api/v1/billing/customer-portal` | `billing.subscription.view_own` |
| POST | `/webhooks/stripe` | HMAC (público) |

### 5.3 Institutional (36)

Grupo `/api/v1/condominiums`:

| Método | Path | Permission |
|---|---|---|
| POST | `/` | `institutional.condominium.create` |
| POST | `/:cid/units` | `institutional.unit.create` |
| POST | `/:cid/memberships` | `institutional.membership.register` |
| POST | `/:cid/memberships/:membership_id/end` | `institutional.membership.end` |
| POST | `/:cid/timeline` | `institutional.timeline.create` |
| GET | `/:cid/timeline` | `institutional.timeline.read` |
| POST | `/:cid/master-plan` | `institutional.master_plan.create` |
| GET | `/:cid/master-plan` | `institutional.master_plan.read` |
| POST | `/:cid/events` | `institutional.event.create` |
| GET | `/:cid/events` | `institutional.event.read` |
| POST | `/:cid/events/:event_id/respond` | `institutional.event.respond` |
| GET | `/:cid/events/:event_id/metrics` | `institutional.event.metrics` |
| POST | `/:cid/announcements` | `institutional.announcement.create` |
| GET | `/:cid/announcements` | `institutional.announcement.read` |
| POST | `/:cid/announcements/:announcement_id/read` | `institutional.announcement.mark_read` |
| GET | `/:cid/announcements/:announcement_id/stats` | `institutional.announcement.stats` |
| POST | `/:cid/assessments` | `institutional.assessment.submit` |
| GET | `/:cid/assessments/current/metrics` | `institutional.assessment.metrics` |
| GET | `/:cid/assessments/current/my-status` | `institutional.assessment.my_status` |
| POST | `/:cid/polls` | `institutional.poll.create` |
| GET | `/:cid/polls` | `institutional.poll.read` |
| POST | `/:cid/polls/:poll_id/vote` | `institutional.poll.vote` |
| POST | `/:cid/polls/:poll_id/close` | `institutional.poll.close` |
| GET | `/:cid/polls/:poll_id/metrics` | `institutional.poll.read` |
| POST | `/:cid/compliance/terms/sign` | `institutional.compliance.sign_term` |
| POST | `/:cid/compliance/annual-declaration` | `institutional.compliance.submit_declaration` |
| GET | `/:cid/compliance/audit` | `institutional.compliance.audit_read` |
| GET | `/:cid/dashboard` | `institutional.dashboard.read` |
| GET | `/:cid/validations` | `institutional.validation.read` |
| POST | `/:cid/validations/:validation_id/validate` | `institutional.validation.decide` |
| POST | `/:cid/validations/:validation_id/reject` | `institutional.validation.decide` |
| POST | `/:cid/validations/:validation_id/request-adjustment` | `institutional.validation.decide` |
| POST | `/:cid/mandates/:mandate_id/end` | `institutional.mandate.end` |
| POST | `/:cid/mandates/:mandate_id/transfer` | `institutional.mandate.transfer` |
| GET | `/master-plan/catalogs` | nenhuma (retorna catálogos fixos) |

### 5.4 Commercial (41)

Grupos: `/companies`, `/connect-me`, `/executions`, `/closed-deals`, `/proposals`, `/empresa-sindica`, `/harassment-reports`. Tabela resumo por ação:

| Grupo | Rotas |
|---|---|
| `/companies` | POST `/`, GET `/`, GET `/:id`, POST `/:id/activate`, POST `/:id/users`, DELETE `/:id/users/:uid`, POST `/:id/evaluations`, GET `/:id/evaluations` |
| `/connect-me` | POST `/`, GET `/`, POST `/:rid/respond` |
| `/executions` | POST `/`, POST `/:eid/submit`, GET `/` |
| `/closed-deals` | POST `/`, POST `/:did/submit`, POST `/:did/company-sign`, POST `/:did/syndic-sign`, GET `/`, POST `/:did/comunicados`, GET `/:did/comunicados`, POST `/:did/comunicados/:cid/validate` |
| `/proposals` | POST `/`, GET `/`, GET `/:pid`, PUT `/:pid`, POST `/:pid/submit`, POST `/:pid/review`, POST `/:pid/withdraw`, POST `/:pid/vote-sessions`, GET `/:pid/vote-sessions/:sid`, POST `/:pid/vote-sessions/:sid/cast`, POST `/:pid/vote-sessions/:sid/close` |
| `/empresa-sindica` | POST `/`, POST `/accept`, GET `/`, PUT `/:lid/permissions`, DELETE `/:lid` |
| `/harassment-reports` | POST `/`, GET `/` (admin), POST `/:rid/review` (admin) |

### 5.5 Content (11)

Grupos: `/content/videos`, `/content/marketing-agencies`.

| Método | Path | Permission |
|---|---|---|
| POST | `/content/videos` | `content.video.publish` |
| GET | `/content/videos` | `content.video.read` |
| GET | `/content/videos/:id` | `content.video.read` |
| PATCH | `/content/videos/:id/hide` | `content.video.manage` |
| PATCH | `/content/videos/:id/unhide` | `content.video.manage` |
| PATCH | `/content/videos/:id/voting-auth` | `content.video.update_voting_auth` |
| POST | `/content/marketing-agencies` | `content.marketing_agency.invite` |
| POST | `/content/marketing-agencies/accept` | `content.marketing_agency.accept` |
| GET | `/content/marketing-agencies` | `content.marketing_agency.read` |
| PATCH | `/content/marketing-agencies/:id/permissions` | `content.marketing_agency.update` |
| DELETE | `/content/marketing-agencies/:id` | `content.marketing_agency.revoke` |

### 5.6 Assembly (25)

Grupo `/condominiums/:cid/assemblies`.

| Método | Path | Permission |
|---|---|---|
| POST | `/` | `assembly.assembly.create` |
| GET | `/` | `assembly.assembly.read` |
| GET | `/:aid` | `assembly.assembly.read` |
| POST | `/:aid/publish` | `assembly.assembly.manage` |
| POST | `/:aid/publish-edital` | `assembly.assembly.manage` |
| POST | `/:aid/start` | `assembly.assembly.manage` |
| POST | `/:aid/end` | `assembly.assembly.manage` |
| POST | `/:aid/agenda-items` | `assembly.agenda_item.manage` |
| POST | `/:aid/agenda-items/:iid/open-voting` | `assembly.agenda_item.manage` |
| POST | `/:aid/agenda-items/:iid/close-voting` | `assembly.agenda_item.manage` |
| POST | `/:aid/agenda-items/:iid/withdraw` | `assembly.agenda_item.manage` |
| POST | `/:aid/agenda-items/:iid/votes` | `assembly.vote.cast` |
| GET | `/:aid/agenda-items/:iid/votes` | `assembly.vote.read` |
| POST | `/:aid/attendance` | `assembly.attendance.checkin` |
| POST | `/:aid/attendance/absence` | `assembly.attendance.manage` |
| POST | `/:aid/attendance/return` | `assembly.attendance.manage` |
| POST | `/:aid/proxies` | `assembly.proxy.create` |
| GET | `/:aid/proxies` | `assembly.proxy.read` |
| POST | `/:aid/proxies/validate` | `assembly.proxy.validate` |
| POST | `/:aid/science` | `assembly.science.acknowledge` |
| GET | `/:aid/science` | `assembly.science.read` |
| GET | `/:aid/minutes` | `assembly.minutes.read` |
| POST | `/:aid/live` | `assembly.assembly.manage` |
| POST | `/:aid/live/join` | `assembly.assembly.read` |
| DELETE | `/:aid/live` | `assembly.assembly.manage` |
| GET | `/:aid/live/participants` | `assembly.assembly.read` |

**Total**: **1 + 4 + 36 + 41 + 11 + 25 = 118 endpoints** protegidos `/api/v1/*` + **2** webhooks públicos + **3** `/auth/*` + **2** health + **1** `/metrics` + `/docs` ⇒ **~126 endpoints** expostos pela API. Steering do v2 referenciava "75+" — número real é **118 endpoints protegidos**.

---

## 6. Providers (inegociáveis em D-056 do STEERING)

### 6.1 Providers do domínio de módulos

| Interface | Arquivo domain | Implementação | Arquivo infra |
|---|---|---|---|
| `IPaymentGateway` | `billing/domain/providers/i_payment_gateway.go` | Stripe | `billing/infrastructure/providers/stripe_gateway.go` |
| `IVideoProvider` | `content/domain/providers/i_video_provider.go` | Mux | `content/infrastructure/providers/mux_provider.go` |
| `ILiveProvider` | `assembly/domain/providers/i_live_provider.go` | Livekit | `assembly/infrastructure/providers/livekit_provider.go` |

### 6.2 Providers compartilhados (`shared/providers/*`)

| Interface | Arquivo | Implementação |
|---|---|---|
| `IZitadelProvider` | `shared/providers/auth/zitadel_provider_interface.go` | `zitadel_provider.go` (JWT Profile, introspection OAuth2) |
| `Cache` (Redis) | `shared/providers/cache/cache.go` | `redis.go` |
| `IUnitOfWork` (no core) + `*pgxpool.Pool` | `shared/providers/database/postgresql.go` + `unit_of_work.go` + `migrator.go` | pgx + goose embed.FS |
| `IEmailProvider` | `shared/providers/email/i_email_provider.go` | Resend (`resend_provider.go`) — 7 templates pt-BR |
| `IStorageProvider` | `shared/providers/storage/i_storage_provider.go` | MinIO (`minio_provider.go`) |
| `ISMSProvider` | `shared/providers/sms/i_sms_provider.go` | Twilio (`twilio_provider.go` — stub) |
| `IPushProvider` | `shared/providers/push/i_push_provider.go` | FCM (`fcm_provider.go` — stub) |
| `IPDFGenerator` | `shared/providers/pdf/i_pdf_generator.go` | gotpl (`gotpl_provider.go` — stub) |
| `ICEPLookup` | `shared/providers/cep/i_cep_lookup.go` | ViaCEP (`viacep_provider.go`) |

**✅ Invariante D-056 atendido**: toda integração externa tem interface em
`domain/` ou `shared/providers/` + adapter em `infrastructure/providers/` ou
`shared/providers/<vendor>/`. **Stubs** (SMS/Push/PDF) são **assumidos pelo
STATE.md**; terminar implementação real é trabalho de Sprint 10+.

### 6.3 Providers cross-module (contratos em `shared/types/`)

| Interface | Exposta por | Consumida por |
|---|---|---|
| `IUserSyncer` | identity.Module | middleware.Authenticate |
| `IUserModerator` | identity.Module | commercial.HarassmentUseCases |
| `ITrialChecker` / `IFeatureAccessChecker` | billing.Module | commercial, institutional |
| `IQuotaConsumer` | billing.Module | commercial.ConnectMe, content.UploadVideo |
| `IValidationSubmitter` | institutional.Module | commercial.ExecutionRecord |
| `ITimelinePublisher` | institutional.Module | commercial.ClosedDeal, assembly.End |
| `ICondominiumIDsProvider` | institutional | identity.SyncUserUseCase (popula `AuthUser.CondominiumIDs`) |

Antipattern **A7 (cross-module direct import)** evitado rigorosamente — main.go
faz todo o wiring; nenhum módulo importa outro por path.

---

## 7. Migrations — goose v3 via embed.FS (39 arquivos)

Numeração particionada: `{módulo×100}{sequencial}`. Executadas no startup
(dev/staging) via `database.Migrate(context.Background(), pool)` ou explícito
via binário `cmd/migrate up` em prod (`railway.toml` define
`releaseCommand = "/app/migrate up"`).

| Faixa | Módulo | Arquivos |
|---|---|---|
| 00001-00004 | shared + identity | `create_enums`, `create_plans`, `create_users`, `create_sessions` |
| 00005 | billing | `create_subscriptions` |
| 00101 | billing | `create_feature_usages` |
| 00200-00210 | institutional | `create_condominium_enums`, `create_condominiums`, `create_units_memberships`, `create_timeline`, `create_master_plan`, `create_events`, `create_announcements`, `create_management_assessments`, `create_polls`, `create_compliance`, `create_validation_items` |
| 00300-00310 | commercial | `create_company_enums`, `create_companies`, `create_connect_me`, `create_execution_records`, `create_closed_deals`, `create_harassment_reports`, `create_proposals`, `create_empresa_sindica`, `create_company_evaluations`, `create_post_deal_comunicados`, `normalize_empresa_sindica_cnpj` |
| 00400-00402 | content | `create_video_enums`, `create_videos`, `create_marketing_agency_links` |
| 00500-00507 | assembly | `create_assembly_enums`, `create_assemblies`, `create_agenda_items`, `create_attendance_records`, `create_votes`, `create_proxies`, `create_science_records`, `create_minutes` |

**Nenhum enum legacy N1/N2/N3 encontrado** — todos os enums do DB usam
valores pt-BR ou trial/base/plus/pro. Confirmado via `grep 'N1\|N2\|N3'
migrations/*.sql` → zero matches no escopo relevante. DT-004 do legado (que
pedia limpeza de slugs compostos) é **não-aplicável** neste backend Go
(pode ter sido contaminação do full-stack TS legado).

---

## 8. Gaps críticos vs spec v2

### 8.1 Unit aggregate (crítico)

**Achado Fase 7d confirmado linha-a-linha**:

- `institutional/domain/entities/unit.go:17-29` — struct `Unit` tem apenas `id, condominiumID, unitNumber, block, floor *int, active`.
- `migrations/00202_create_units_memberships.sql:27-43` — tabela `units` tem apenas `unit_number TEXT`, `block TEXT`, `floor INTEGER`, `active BOOLEAN`.

**Faltam**:
- `unit_type` (residencial | comercial | vaga_garagem | deposito | sala_comercial | loja)
- `fracao_ideal` (NUMERIC(5,2) ou VARCHAR — usado na matriz de quórum em assembleias).

**Impacto**:
- Assembleia com quórum qualified/absolute por fração ideal **não é calculável**.
- Rateio de despesas/votos ponderado **não é possível**.
- Corner case crítico para os condomínios-cliente de Master Síndico (João explicitou).

**Proposta**: migration `00211_add_unit_type_and_fracao_ideal.sql` + enum DB
`unit_type` + column `fracao_ideal NUMERIC(5,2) CHECK (fracao_ideal BETWEEN 0
AND 100)`. Promover a **task Sprint 10 de prioridade alta** (pré-requisito
para quórum por fração em assembleia — atualmente hack em `AgendaItem.fractionIdeal`
string).

### 8.2 NATS JetStream ausente

- `go.mod` não inclui `github.com/nats-io/nats.go`.
- `billing/infrastructure/events/in_memory_publisher.go` e `institutional/infrastructure/events/in_memory_publisher.go` são **in-process só**.
- Consequência: eventos `SubscriptionActivated`, `EventTimelinePublished` **não cruzam réplicas** em horizontal scale.
- STEERING dita NATS JetStream como inegociável para outbox/domain events — discrepância registrada como D-### candidato em [[../../STATE]].

### 8.3 Audit log fragmentado (DT-010 ativo)

- Só `institutional/.../compliance_use_cases.go` persiste `AuditLogEntry`.
- Billing, commercial (deal.sign, empresa_sindica), identity (membership.register) usam `zerolog.Info` só.
- Sprint 9 deferrou para Sprint 10 como `IAuditLogger` em `shared/types/`.

### 8.4 Pendentes para Marco 1 (2026-05-08)

| ID | Origem | Severidade | Sprint |
|---|---|---|---|
| A-023 | AUDIT | medium | 10 — 1-device change audit log comparativo |
| A-024 | AUDIT | medium | 10 — `rawBodyReader` interface privada → `Context.RawBody()` |
| A-039 | AUDIT | medium | 10 — rate-limit per-userID (spec exige IP + user) |
| DT-003 | STATE | medium | 10 — sem circuit breaker para Zitadel introspection |
| DT-004 | STATE | low | 2+ (já tarde) — sem timeout explícito em `UnitOfWork.Run` |
| DT-010 | STATE | high | 10 — IAuditLogger cross-módulo |
| Sprint 9.10 | tasks.md | medium | 9 — 2 templates email (welcomes, billing-alerts) faltantes |
| Sprint 9.19 | tasks.md | high | 9 — Zitadel managed em Railway + DNS custom |

---

## 9. Coverage atual (snapshot 2026-04-22)

Extraído de `coverage.out` (1.1MB) + sessão autônoma F1-F33.

| Camada | Threshold | Real |
|---|---|---|
| `internal/modules/*/domain/entities` | ≥95% | commercial **96.1%**, assembly **100%**, institutional **98.7%** |
| `internal/modules/*/application/use_cases` | ≥90% | commercial **92.8%**, content **100%**, assembly **91.5%**, institutional **91.0%** |
| `internal/shared/middleware` | ≥90% | ✅ (abac_matrix_test + abac_engine_pbt 400 casos + require_permission + authenticate + rate_limiter) |
| `internal/shared/pagination` | ≥90% | ✅ PBT 1000 casos + table-driven parseBounded |
| `pkg/telemetry` | ≥90% | 4 testes; coverage não re-medido |
| Global (`coverage.out`) | ≥85% | último run bulk; precisa refresh após A-032/A-033/A-034 |

**Tipos de teste em produção**:
- Unit tests: 132 arquivos `*_test.go` (excluindo generated)
- Integration tests com tag `integration`: 9 arquivos (condominium, user, session, subscription, feature_usage, timeline, announcement, supplier_vote + template)
- **Property-Based Tests** (`pgregory.net/rapid`): **3 suites críticas**
  - `assembly/domain/entities/agenda_item_pbt_test.go` — fração ideal 0-100% (100 casos)
  - `billing/domain/entities/feature_usage_pbt_test.go` — quotas Consume / Unlimited / ZeroLimit (300 casos)
  - `shared/middleware/abac_engine_pbt_test.go` — ABAC admin_bypass + role + action + tenant (400 casos)
  - `shared/pagination/pagination_test.go` — pagination parseBounded (1000 casos)

**Regra 6/6** do `steering/testing.md`:
1. ✅ Fração ideal — PBT
2. ✅ Quotas — PBT
3. ✅ ABAC — PBT
4. ✅ Tenant isolation — covered em PBT ABAC + table-driven matrix
5. 🟡 Trial expiration — table-driven com `math/rand/v2`; não migrado para rapid ainda
6. 🟡 Money BRL int64 centavos — enforce em code review + steering; benefício PBT dinâmico baixo

---

## 10. Gates de qualidade (snapshot 2026-04-22 06:30)

Rodados em CI local + manual após F1-F33:

| Gate | Comando | Resultado |
|---|---|---|
| Build | `go build ./...` | ✅ exit 0 |
| Vet | `go vet ./...` | ✅ exit 0 |
| Test race | `go test -race -count=1 ./...` | ✅ todos verdes |
| Integration build | `go build -tags=integration ./...` | ✅ exit 0 |
| Gosec severity=high | `gosec -severity=high -exclude-dir=generated -exclude-dir=cmd/zitadel-setup ./...` | ✅ **0 findings** (de 66 iniciais — G101/G109/G115) |
| Govulncheck | `govulncheck ./...` | ✅ 0 CVEs em reachable deps |
| Coverage gate | `go-test-coverage --profile=coverage.out --threshold-file=.coverage.yml` | ✅ (último run bulk) |

**Dependabot**: 3 CVEs OTel 1.40→1.43 fechados no commit `d410547`.

---

## 11. Referências cruzadas

- Arquitetura: [[architecture]]
- Padrões e anti-patterns: [[patterns]]
- Requisitos não-funcionais: [[requirements]]
- Segurança (BeyondCorp + LGPD): [[security]]
- Tasks Sprint 10+: [[tasks]]
- MOC: [[_moc]]
- Steering raiz: [[../../STEERING]]
- State / ADR: [[../../STATE]]
- Domínio (linguagem ubíqua): [[../../01-domain/_moc]]
- Cliente master-sindico original: `/home/vinni/Documentos/Projetos/Clientes/Joao/MasterSindico/Development/backend/`
