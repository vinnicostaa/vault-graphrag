# Master Síndico — Backend

API Go do ecossistema condominial Master Síndico.

## Stack

- **Go 1.26** + Gin (abstraído via `contracts.HTTPRouter`)
- **PostgreSQL 18** (pgx/v5 + sqlc — sem ORM)
- **Redis 7** (go-redis/v9 — sessões, ABAC cache, quotas)
- **NATS JetStream** (pub/sub assíncrono)
- **Zitadel** (OIDC + JWT introspection — Docker em dev, cloud em prod)
- **Stripe** (billing — via `IPaymentGateway`)
- **Mux** (vídeo VOD + live CDN — via `IVideoProvider` + `ILiveStreamProvider`)
- **MinIO** (storage S3-compatible — via `IStorageProvider`; Cloudflare R2/AWS S3 em prod)
- **Livekit** (WebRTC assembly ao vivo — via `ILiveProvider`; Mux Spaces futuramente)
- **Resend** (e-mail transacional — via `IEmailProvider`; Twilio SendGrid futuramente)
- **OpenTelemetry** (traces + métricas Prometheus)

## Arquitetura

Clean Architecture + DDD + CQRS + Vertical Slices. Cada bounded context é um módulo autocontido. Dependências apontam para dentro: `infrastructure → application → domain`.

```
cmd/api/main.go
internal/
  core/
    contracts/        ← IModule, IUseCase[I,O], IRepository[T,ID], IUnitOfWork
    errors/           ← AppError, DomainError e variantes
  modules/
    identity/         ← usuários, sessões, autenticação Zitadel
    billing/          ← planos, assinaturas, Stripe, trial, quotas
    institutional/    ← condomínio, timeline, plano diretor
    commercial/       ← connect me, contratos, marketplace
    content/          ← vídeos Mux, busca OpenSearch
    assembly/         ← assembleias, votações, atas
  shared/
    config/           ← Viper (todas as configs centralizadas)
    middleware/       ← CORS, logger, recovery, request_id
    providers/
      cache/          ← Redis (interface + implementação)
      database/       ← pgxpool + UnitOfWork (tx via context)
pkg/
  logger/             ← zerolog wrapper (interface Logger)
  server/             ← Gin engine, health checks, error handler
```

## Pré-requisitos

- Go 1.26+
- Docker + Docker Compose

## Ambiente local

```bash
# Subir PostgreSQL 18, Redis 7 e NATS JetStream
docker compose up -d

# Copiar e ajustar variáveis
cp .env .env.local

# Rodar a API
go run ./cmd/api
```

A API sobe em `http://localhost:8000`.

## Health checks

```
GET /health        → liveness (sempre 200)
GET /health/ready  → readiness (verifica PostgreSQL + Redis)
```

## Migrations

Gerenciadas com [goose](https://github.com/pressly/goose) + `embed.FS` (migrations dentro do binário).

```
internal/shared/providers/database/migrations/
  00001_create_enums.sql        ← user_role, user_role_type, plan_tier, subscription_status
  00002_create_plans.sql        ← plans + plan_features (quotas)
  00003_create_users.sql        ← users (identidade + role + plan_id + visibility_ready)
  00004_create_sessions.sql     ← sessions (BeyondCorp device tracking)
  00005_create_subscriptions.sql ← subscriptions (billing Stripe)
```

As migrations rodam automaticamente no startup em desenvolvimento. Em produção, use o binário separado `cmd/migrate`.

## Variáveis de ambiente

| Variável | Descrição | Default |
|---|---|---|
| `APP_ENV` | `development` / `staging` / `production` | `development` |
| `APP_PORT` | Porta da API | `8000` |
| `DB_HOST` | Host do PostgreSQL | `localhost` |
| `DB_NAME` | Nome do banco | `master_sindico` |
| `REDIS_HOST` | Host do Redis | `localhost` |
| `REDIS_PASSWORD` | Senha do Redis | `redis_password` |
| `ZITADEL_DOMAIN` | Domínio do Zitadel | — |
| `ZITADEL_KEY_PATH` | Path para o service account key.json | — |
| `ZITADEL_PROJECT_ID` | ID do projeto no console Zitadel (Projects > seu projeto) | — |
| `ZITADEL_WEB_CLIENT_ID` | Client ID do App Web PKCE | — |
| `ZITADEL_WEB_REDIRECT_URI` | URI de callback OIDC | `http://localhost:8000/auth/callback` |
| `ZITADEL_WEB_ENCRYPTION_KEY` | Chave AES para cookies PKCE (`openssl rand -hex 16`) | — |

Ver `.env` para a lista completa com todos os valores de desenvolvimento.

## Auth Flow (OIDC PKCE)

O fluxo de autenticação usa Authorization Code Flow com PKCE via `zitadel/oidc/v3`.

```
Browser → GET /auth/login → Zitadel Login UI → GET /auth/callback
       → backend seta cookie access_token httpOnly → redirect para frontend

Mobile → Flutter SDK (PKCE nativo) → access_token no flutter_secure_storage
       → requests com Authorization: Bearer <token>

Ambos → GET /api/v1/auth/me → middleware Authenticate valida via introspection
```

### Rotas de auth

```
GET  /auth/login     → inicia OIDC, redireciona para Zitadel (pública)
GET  /auth/callback  → recebe code, troca por token, seta cookie (pública)
GET  /auth/logout    → deleta cookie + end_session no Zitadel (pública)
GET  /api/v1/auth/me → retorna AuthUser do contexto (protegida)
```

### Variáveis de ambiente Zitadel

| Variável | Descrição |
|---|---|
| `ZITADEL_DOMAIN` | Domínio da instância Zitadel |
| `ZITADEL_KEY_PATH` | Path do service account key.json (introspection) |
| `ZITADEL_PROJECT_ID` | ID do projeto (para extrair roles do token) |
| `ZITADEL_WEB_CLIENT_ID` | Client ID do App Web PKCE |
| `ZITADEL_WEB_REDIRECT_URI` | URI de callback (`/auth/callback`) |
| `ZITADEL_WEB_POST_LOGOUT_URI` | URI de redirect após logout |
| `ZITADEL_WEB_ENCRYPTION_KEY` | Chave AES para cookies PKCE (gerar: `openssl rand -hex 16`) |

---

## Sprint 0 — Fundação ✅

| Task | Status |
|---|---|
| 0.1 Monorepo web (bun workspaces, biome, tsconfig) | ✅ |
| 0.2 Estrutura Go Clean Architecture + Gin server + middlewares base | ✅ |
| 0.3 PostgreSQL — migrations base (users, sessions, plans, subscriptions) | ✅ |
| 0.4 Redis — conexão + cache provider (interface + implementação) | ✅ |
| 0.5 Zitadel deploy | ✅ (dev local) / ⏳ (prod Railway) |
| 0.6 Core contracts (IModule, IUseCase, IRepository, IUnitOfWork) | ✅ |
| 0.7 Gin plugins — CORS, Recovery, Logger, RequestID | ✅ |
| 0.7 Rate limit (token bucket por IP, redis-backed) | ✅ |
| 0.7 Swagger/OpenAPI docs | ⏳ Sprint 9 |
| 0.8 CI/CD pipeline | ⏳ Sprint 9 |
| 0.9 Railway deploy (backend + DB + Redis) | ⏳ Sprint 9 |
| 0.10 Flutter setup (app/) | ✅ |
| 0.11 OpenSearch cluster | ⏳ Sprint 9 |

## Sprint 1 — Identity & Billing ✅

### Correções técnicas

| Task | Status |
|---|---|
| `AuthUser` movido para `shared/types/` (Clean Architecture) | ✅ |
| `zerolog.SetGlobalLevel()` removido (testes paralelos) | ✅ |
| `database.Migrate()` chamado no `main.go` | ✅ |
| Logger injetado no `UnitOfWork` (rollback silencioso) | ✅ |
| `planTierOrder` movido para package-level | ✅ |
| `contracts.Context` com `Abort()`, `Cookie()`, `SetCookie()`, `DecodeJSON()` | ✅ |
| `ZITADEL_PROJECT_ID` adicionado ao config + validação | ✅ |
| sqlc v1.30.0 configurado — queries tipadas em `identity/infrastructure/database/queries/` | ✅ |
| sqlc nullable overrides — enums e UUIDs nullable como `*string` | ✅ |
| Entidades `User` e `Session` criadas em `identity/domain/entities/` | ✅ |
| Interfaces `IUserRepository` e `ISessionRepository` criadas | ✅ |
| `ZitadelClaims` com `Username` e `Phone` (campos da introspection) | ✅ |
| `IZitadelProvider` movida para `shared/providers/auth/` | ✅ |
| `IUserSyncer` + `SyncUserCommand` + `SyncUserOutput` em `shared/types/` | ✅ |
| Wire-up completo no `main.go` (ZitadelProvider + SyncUserUseCase) | ✅ |
| `RegisterModules()` corrigido — módulos recebem router protegido com auth | ✅ |
| Guard clause `claims.Role == ""` no middleware Authenticate | ✅ |
| `BanExpiresAt` propagado até `AuthUser` | ✅ |

### BeyondCorp + Billing

| Task | Status |
|---|---|
| 1.1 Middleware BeyondCorp completo (user sync, Redis cache, 1 device) | ✅ |
| 1.1 OIDC PKCE flow (login/callback/logout + `GET /auth/me`) | ✅ |
| 1.1 Fallback `Authorization: Bearer` para mobile (RFC 6750) | ✅ |
| 1.1 `identity.Module` encapsula repositories + use cases | ✅ |
| 1.2 ABAC engine (role × role_type × plan_tier × action — 100+ testes) | ✅ |
| 1.3 Stripe adapter (`IPaymentGateway` → `StripeGateway`) | ✅ |
| 1.4 Plans + Subscriptions CRUD + webhook handler (Stripe events) | ✅ |
| 1.5 Trial logic por persona (Síndico 15d, Empresa 7d, Parceiro 30d) | ✅ |
| 1.6 Quotas + Feature Usage (plan_features) | ✅ |

## Sprint 2 — Institutional I ✅

| Task | Status |
|---|---|
| 2.1 Módulo institutional (condomínio, endereço, CNPJ VO) | ✅ |
| 2.2 Criação e consulta de condomínios | ✅ |
| 2.3 Mandatos de síndico (criar, transferir, encerrar) | ✅ |
| 2.4 Memberships (moradores, unidades, fração ideal) | ✅ |
| 2.5 Timeline condominial | ✅ |

## Sprint 3 — Institutional II ✅

| Task | Status |
|---|---|
| 3.1 Plano Diretor (atividades + listagem) | ✅ |
| 3.2 Comunicados (announcements) | ✅ |
| 3.3 Eventos com RSVP + métricas | ✅ |
| 3.4 Enquetes (múltipla escolha + sim/não) com votação | ✅ |
| 3.5 Compliance (assinar termo, declaração anual) | ✅ |
| 3.6 Avaliação de gestão (rating_q1/q2/q3) | ✅ |
| 3.7 Itens de validação condominial | ✅ |
| 3.8 Dashboard condominial | ✅ |

## Sprint 4 — Commercial ✅

| Task | Status |
|---|---|
| 4.1 Empresas (CNPJ VO, categorias, ABAC) | ✅ |
| 4.2 Empresa-Síndica Link (convite por token, aceite) | ✅ |
| 4.3 Propostas (criar, submeter, aceitar, rejeitar) | ✅ |
| 4.4 Execution Records (registro de execução de serviço) | ✅ |
| 4.5 Closed Deal (fechamento de negócio com custo final) | ✅ |
| 4.6 Supplier Votes (votar em fornecedor) | ✅ |
| 4.7 Harassment Reports (denúncia + revisão admin) | ✅ |

## Sprint 5 — Content ✅

| Task | Status |
|---|---|
| 5.1 Video Library (metadados, tipos em pt-BR, ABAC) | ✅ |
| 5.2 Listagem e busca de vídeos | ✅ |
| 5.3 Remoção de vídeos, hide/unhide, voting-auth | ✅ |
| 5.4 Marketing Agency Link (convite por token) | ✅ |

## Sprint 6 — Commercial Integrations ✅

| Task | Status |
|---|---|
| 6.6 Connect Me flow (empresa-síndica linking) | ✅ |
| 6.7 Supplier marketplace (votos, reputação) | ✅ |
| 6.8 Anti-assédio (reports + moderação admin) | ✅ |

## Sprint 7 — Assembly ✅

| Task | Status |
|---|---|
| 7.1 Lifecycle completo de assembleia (rascunho → publicada → em_andamento → encerrada) | ✅ |
| 7.2 Itens de pauta (adicionar, abrir/fechar votação, retirar) | ✅ |
| 7.3 Sistema de votação (cast vote, resultados por item) | ✅ |
| 7.4 Presença, procurações, ciência do edital, ata automática | ✅ |
| 7.5 Rota `POST /publish-edital` (fix A-013) | ✅ |

## Sprint 8 — Security & Observability ✅

| Task | Status |
|---|---|
| 8.1 OpenTelemetry (traces + métricas) — noop quando OTEL_ENABLED=false | ✅ |
| 8.2 Prometheus bridge + `/metrics` endpoint | ✅ |
| 8.3 W3C TraceContext propagation (Jaeger/X-Ray compatível) | ✅ |
| 8.4 pgx tracer (spans para queries PostgreSQL) | ✅ |
| 8.5 ABAC matrix — 100+ testes (role × plan_tier × action) | ✅ |
| 8.6 Security review BeyondCorp (gosec + govulncheck — 0 CVEs) | ✅ |
| 8.7 QA funcional completo (64 rotas testadas) | ✅ |
| 8.8 Pentest (IDOR, SQLi, XSS, CSRF, CORS, rate limit, webhook) | ✅ |

### Decisões técnicas Sprint 1

| Decisão | Resolução |
|---|---|
| `IZitadelProvider` + `ZitadelClaims` | Vivem em `shared/providers/auth/` — `modules/identity/domain/providers/` usa type aliases |
| `IUserSyncer` + `SyncUserCommand` + `SyncUserOutput` | Vivem em `shared/types/` — middleware não importa de `modules/` |
| `RegisterModules()` | Passa grupo `/api/v1` com `Authenticate` já aplicado — módulos nunca recebem router público |
| Guard clause de role vazia | Rejeita `403 NO_ROLE_ASSIGNED` antes do UPSERT — evita `role=""` no banco |
| sqlc nullable enums | `role_type`, `plan_id`, `user_status` são `*string` — `NullUserRoleType`/`pgtype.UUID` eliminados |
| `BanExpiresAt` | Propagado `User.GetBanExpiresAt()` → `SyncUserOutput` → `AuthUser` — bans temporários funcionam |
| `PlanTier` no `AuthUser` | `"trial"` como fallback seguro até task 1.4 implementar join com `plans` |
| Cache invalidação proativa | Implementada na task 1.4 (webhook Stripe) — delay de 5min é aceitável para billing |
| OIDC PKCE | Usa `zitadel/oidc/v3` diretamente (`rp.NewRelyingPartyOIDC`) — controle total sobre `ResponseWriter` no callback |
| Dual token strategy | Web: cookie httpOnly `access_token` · Mobile: `Authorization: Bearer` — middleware suporta ambos |
| Module encapsulation | `identity.Module` instancia repositories e use cases internamente — `main.go` só conhece `IModule` e `IUserSyncer` |
| Rotas OIDC | Fora do grupo `/api/v1` — são redirects de browser, não endpoints de API REST |

## Observabilidade (Sprint 7.9)

O backend expõe telemetria via OpenTelemetry SDK v1.40.0.

### Variáveis de ambiente OTel

| Variável | Padrão | Descrição |
|---|---|---|
| `OTEL_ENABLED` | `false` | Liga o SDK OTel. Quando `false`, providers são noop (overhead zero) |
| `OTEL_EXPORTER_OTLP_ENDPOINT` | — | URL base do OTLP HTTP collector (ex: `http://otelcol:4318`) |
| `OTEL_PROMETHEUS_ENABLED` | `false` (dev), `true` (staging/prod) | Habilita bridge OTel→Prometheus e endpoint `/metrics` |
| `OTEL_SAMPLE_RATE` | `1.0` (dev/staging), `0.1` (production) | Fração de traces a amostrar |
| `OTEL_SERVICE_VERSION` | `0.0.0` | Versão do artefato (semver) |

### Endpoint de métricas

```
GET /metrics     → Prometheus scrape endpoint (só disponível quando OTEL_PROMETHEUS_ENABLED=true)
```

### Instrumentação automática

- **HTTP**: todas as requisições recebem um span OTel via `otelgin` middleware. `trace_id` e `span_id` são injetados nos campos de log de cada request.
- **PostgreSQL**: todas as queries pgx/v5 (queries, batches, COPY FROM) geram spans `db.query`/`db.batch`/`db.copy_from` com `db.system.name=postgresql` e `db.query.text`.
- **Propagação**: W3C TraceContext + Baggage — compatível com AWS X-Ray e Jaeger.

### Configuração para produção (Railway → ADOT → CloudWatch)

```bash
OTEL_ENABLED=true
OTEL_EXPORTER_OTLP_ENDPOINT=http://adot-collector:4318
OTEL_PROMETHEUS_ENABLED=true
OTEL_SAMPLE_RATE=0.1
OTEL_SERVICE_VERSION=1.0.0
```

---

## Geração de código (sqlc)

Queries SQL tipadas geradas com [sqlc](https://sqlc.dev) v1.30.0.

```bash
# Regenerar após alterar arquivos em queries/
sqlc generate
```

Arquivos de query ficam em `internal/modules/{module}/infrastructure/database/queries/*.sql`.
Código gerado fica em `internal/modules/{module}/infrastructure/database/generated/` — **não editar manualmente**.

---

## Modelo de dados (Sprint 0)

```
users
  id, zitadel_id, email, username, phone
  role (syndic|resident|enterprise|marketing|local_company|admin)
  role_type (principal|auxiliary|titular|dependent|administrator|operator)
  plan_id → plans
  status, visibility_ready
  device_fingerprint, last_ip (BeyondCorp)
  banned, ban_reason, ban_expires_at

sessions
  id, user_id → users
  zitadel_session_id
  ip_address, user_agent, device_fingerprint
  expires_at

plans
  id, role, tier (trial|base|plus|pro)
  name, price_monthly, price_yearly
  trial_days
  stripe_product_id, stripe_price_monthly_id, stripe_price_yearly_id
  UNIQUE (role, tier)

plan_features
  plan_id → plans
  feature_key, feature_value, feature_type
  Ex: connect_me_quota=2, video_upload_monthly=4

subscriptions
  id, user_id → users, plan_id → plans
  status (trialing|active|past_due|canceled|paused|...)
  current_period_start, current_period_end
  trial_ends_at, canceled_at, cancel_at_period_end
  stripe_customer_id, stripe_subscription_id, stripe_price_id
```

---

## Status dos módulos (Sprint 9 — 2026-04-22)

| Módulo | Endpoints | Status |
|---|---|---|
| `identity` | `GET /api/v1/auth/me` + OIDC (`/auth/{login,callback,logout}`) | ✅ |
| `billing` | Plans, Subscription, Checkout, Customer Portal + Stripe webhook | ✅ |
| `institutional` | Condomínium, Units, Mandates, Timeline, Master Plan, Announcements, Events, Polls, Compliance, Assessments, Validations, Dashboard | ✅ |
| `commercial` | Companies, Empresa-Síndica, Connect Me, Proposals, Executions, Closed Deals, Evaluations, Post-Deal Comunicados, Harassment Reports | ✅ |
| `content` | Videos (Mux upload direto + webhook), Marketing Agencies | ✅ Sprint 9 |
| `assembly` | Assembleias, Agenda Items, Votes, Attendance, Proxies, Science, Minutes | ✅ |

> **106 endpoints** — ver `routes.http` para exemplos completos de request/response.

### Rotas de saúde e observabilidade

```
GET  /health          → liveness probe
GET  /health/ready    → readiness (PostgreSQL + Redis)
GET  /metrics         → Prometheus (quando OTEL_PROMETHEUS_ENABLED=true)
```

## Sprint 9 — Integrations (em andamento, 2026-04-22)

| Task | Status |
|---|---|
| 9.1 MuxProvider SDK real (VOD + live CDN) | ✅ |
| 9.2 MuxProvider wirado no content/module.go (IVideoProvider domain) | ✅ |
| 9.3 MinIOProvider (IStorageProvider — S3-compatible) | ✅ |
| 9.4 LivekitProvider (ILiveProvider — WebRTC assembly) | ✅ |
| 9.5 ResendProvider (IEmailProvider — e-mail transacional) | ✅ |
| 9.6 docker-compose atualizado (Zitadel, MinIO, Livekit, NATS) | ✅ |
| 9.7 Config agnóstica (VideoProviderConfig, EmailProviderConfig, etc.) | ✅ |
| 9.8 UploadVideoUseCase: retorna UploadURL + UploadID (upload direto ao Mux) | ✅ |
| 9.9 Assembly ao vivo: StartLiveSession, JoinLiveSession, EndLiveSession | ⏳ |
| 9.10 Mux webhook com validação de assinatura (header `Mux-Signature`) | ⏳ |
| 9.11 Email templates pt-BR completos | ⏳ |
| 9.12 Railway deploy: Dockerfile multi-stage, railway.toml, GitHub Actions CI | ⏳ |
| 9.13 Zitadel: Organization + Projects + Roles no console (:8080) | ⏳ |
| 9.14 OpenAPI/Swagger (`/docs`) | ⏳ |

### Ambiente local completo (Sprint 9)

```bash
# Subir todos os serviços: PostgreSQL 18, Redis 7, NATS, Zitadel, MinIO, Livekit
docker compose up -d

# Portas dos serviços:
# PostgreSQL: 5432  |  Redis: 6379  |  NATS: 4222
# Zitadel:   8080  (admin: zitadel-admin@zitadel.localhost / Password1!)
# MinIO API: 9000  |  MinIO Console: 9001  (minioadmin / minioadmin)
# Livekit:   7880 (ws)  |  7881 (TCP RTC)  |  7882 UDP RTC

# Testar webhooks Stripe (em terminal separado)
stripe listen --forward-to localhost:8000/webhooks/stripe

# Rodar a API
go run ./cmd/api
```

### Pendências (Sprint 9 → MVP Marco 1: 2026-05-08)

- [ ] Assembly ao vivo (LivekitProvider no assembly/module.go)
- [ ] Railway deploy (Dockerfile + railway.toml + GitHub Actions)
- [ ] Zitadel cloud config (Organization + Projects + Roles)
- [ ] OpenAPI/Swagger docs
- [ ] Mux webhook com assinatura HMAC (`Mux-Signature`)
