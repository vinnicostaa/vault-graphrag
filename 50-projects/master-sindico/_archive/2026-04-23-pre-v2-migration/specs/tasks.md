# Tasks — Master Síndico

> **Status do Projeto (2026-04-21)**:
> Backend Sprints 0-8: ✅ Dev1 completo — API Go, ABAC, BeyondCorp, DDD, CQRS, assembly, billing, content, commercial, institutional
> Frontend/Mobile: ⏳ pendente — SolidJS + Flutter
> Deploy/Integrações: ⏳ Sprint 9 (Railway, Mux SDK real, Livekit, Resend, MinIO)

> **⚠️ Em migração**: formato de lista aqui é **canônico para Sprints 2-8** e pós-lançamento. Sprints ativos usam **formato 5-fase** em arquivos dedicados:
>
> - [`tasks/README.md`](tasks/README.md) — formato 5-fase (Discuss → Plan → Execute → Verify → Ship)
> - [`tasks/sprint-0.md`](tasks/sprint-0.md) — fundação (concluído)
> - [`tasks/sprint-1.md`](tasks/sprint-1.md) — Identity & Billing (concluído)
>
> Conforme cada Sprint se aproxima, é migrado para o formato detalhado. Este monolítico permanece como visão geral do roadmap.

> Organização: **9 Sprints de 14 dias** + buffer QA · Vertical Slices (backend + frontend + mobile juntos)
> Stack: Go (Gin) · SolidJS (TanStack + Rsbuild + UnoCSS) · Flutter (Clean Arch) · Zitadel · Stripe · **Mux** (vídeo VOD + live CDN + storage) · **Livekit** (WebRTC assembly ao vivo) · PostgreSQL · Redis · **Resend** (email — futuro: Twilio) · Twilio (SMS) · NATS · **Railway** (deploy) · **MinIO** (storage S3-compatible)
> Arquitetura: Clean Architecture + DDD + CQRS + Vertical Slices — BeyondCorp adaptado (Zero Trust, ABAC, tenant isolation, 1 device por conta)
> Equipe: Dev1 (full-stack/arquiteto) + Dev2 (mobile/web) · Dev1 sempre 1 sprint à frente no backend

---

## Sprint 0 — Fundação (23/03 → 06/04) — ✅ Dev1 backend completo

**Dev1 (Main Track):** Monorepo, Zitadel, DB, Redis, Core contracts
**Dev2:** Flutter setup, OIDC integration

- [x] 0.1 Setup monorepo web: workspaces bun, biome, tsconfig base, turbo pipeline
- [x] 0.2 Setup Go: Clean Architecture (`cmd/api/main.go`, `internal/core/contracts`, `internal/core/errors`, `internal/modules/{identity,billing,institutional,commercial,content,assembly}/{domain,application,infrastructure}`, `internal/shared/`, `pkg/`), Gin server, middleware base (logger, CORS, recovery, rate limit)
- [x] 0.3 PostgreSQL + migrations base: enums (`user_role`, `user_role_type`, `plan_tier`, `user_status`, `subscription_status`), tabelas `plans` + `plan_features`, `users`, `sessions`, `subscriptions` — migrations com goose v3 + embed.FS
- [x] 0.4 Redis setup: conexão, helpers de cache, TTL patterns
- [ ] 0.5 Zitadel deploy: **Railway** (container ou VM), domínio `auth.mastersindico.com.br`, SSL automático, Organization + Projects + Roles (syndic, resident, enterprise, marketing, local_company, admin) — *dev local pronto via docker-compose (commit 218a7a3 — Zitadel v4 + Traefik + zitadel-login); Railway managed ainda pendente — migrar para 9.19*
- [x] 0.6 Core contracts Go: `IModule`, `IUseCase[Input, Output]`, `IRepository[T, ID]`, `IUnitOfWork`, `IDomainEvent`, `AppError` + sentinels ErrNotFound/ErrForbidden/ErrConflict
- [x] 0.7 Gin plugins: CORS, Recovery, Logger, RequestID, ErrorHandler global, GinAdapter (contracts.HTTPRouter), response helpers
- [x] 0.8 CI/CD pipeline: GitHub Actions → build + vet + test → **Railway deploy** — entregue no Sprint 9 (commit 5114325: `.github/workflows/ci.yml` + deploy automático); o pipeline existe e passa; deploy staging pendente configuração do Railway token.
- [x] 0.9 Railway Foundation: entregue parcial no Sprint 9 (commit 5114325): `Dockerfile` multi-stage distroless + `railway.toml` + health check. **Pendente**: Railway managed Postgres/Redis, MinIO Railway volume, domínios custom + SSL — permanece em 9.19.
- [ ] 0.10 Flutter setup: estrutura Clean Architecture por feature, Dio + interceptors, flutter_secure_storage, Zitadel OIDC integration

---

## Sprint 1 — Identity & Billing (07/04 → 20/04) — ✅ Dev1 backend completo

**Dev1 (Main Track):** Correções técnicas Sprint 0 → Billing, ABAC, Trial
**Dev2:** Flutter auth + onboarding

### Correções Técnicas (bloqueadores herdados do Sprint 0)

- [x] 1.0.1 Mover `AuthUser` para `internal/shared/types/`
- [x] 1.0.2 Remover `zerolog.SetGlobalLevel()` do `pkg/logger/logger.go`
- [x] 1.0.3 Chamar `database.Migrate(ctx, pool)` no `main.go`
- [x] 1.0.4 Injetar `logger.Logger` no `UnitOfWork`
- [x] 1.0.5 Mover `tierOrder` para package-level em `authorize.go`
- [x] 1.0.6 Adicionar `Abort()`, `Cookie()`, `SetCookie()` ao `contracts.Context`
- [x] 1.0.7 Adicionar `ZITADEL_PROJECT_ID` ao `ZitadelConfig`; renomear `ShouldBindJSON` para `DecodeJSON`
- [x] 1.0.8 Adotar sqlc v1.30.0 — geração de código tipado a partir de SQL
- [x] 1.0.9 Helper `DBFromContext(ctx, pool)` — padrão UnitOfWork sem repetição
- [x] 1.0.10 Entidades de domínio `User` e `Session`
- [x] 1.0.11 Interfaces `IUserRepository` e `ISessionRepository`
- [x] 1.0.12 Atualizar `ZitadelClaims` com `Username` e `Phone`

### BeyondCorp Middleware Stack

- [x] 1.1 Zitadel ↔ Go middleware BeyondCorp completo: JWT httpOnly cookie + Bearer fallback, introspection, UPSERT user, Redis cache 5min, 1-device rule, PKCE OIDC (`/auth/login|callback|logout`), `GET /api/v1/auth/me`
- [x] 1.2 ABAC engine Go: `RequirePermission(action, resource string)`, matriz role × planTier, retorna 403 INSUFFICIENT_PERMISSION

### Billing

- [x] 1.3 Stripe Adapter Go: `IPaymentGateway` → `StripeGateway` — CreateCustomer, CreateSubscription, CancelSubscription, VerifyWebhookSignature, ParseWebhookEvent
- [x] 1.4 Plans + Subscriptions Go: GetPlanByRoleAndTier, GetActiveSubscription, webhook handler (subscription.created/updated, invoice.paid/failed)
- [x] 1.5 Trial logic Go: `CheckFeatureAccess(userID, feature string)` — consulta `plan_features`, bloqueia por persona
- [x] 1.6 Quotas + Feature Usage Go: Redis tracking (`ms:quota:{userID}:{feature}:{period}`), IncrementUsage, GetUsage

### Frontend Web (SolidJS)

- [ ] 1.7 Onboarding framework SolidJS: stepper, auto-save Redis via `PATCH /onboarding/progress`, validação Zod por step
- [ ] 1.8 Telas auth SolidJS: Splash, Login, Cadastro, OTP, Recuperação, Seleção de Perfil
- [ ] 1.9 Onboarding Síndico SolidJS (6 telas)
- [ ] 1.10 Onboarding Morador SolidJS (4 telas)
- [ ] 1.11 Onboarding Empresa SolidJS (7 telas)
- [ ] 1.12 Onboarding Parceiro SolidJS (4 telas)
- [ ] 1.13 `TrialBlocker` component SolidJS + tela de upgrade → Stripe Customer Portal

### Mobile (Flutter)

- [ ] 1.14 Flutter: telas auth (Splash, Login, Cadastro, OTP, Recuperação), onboarding por persona, biometric login

---

## Sprint 2 — Institutional I: Registry + Governance Base (21/04 → 04/05) — ✅ Dev1 backend completo

**Dev1 (Main Track):** Condominium Registry, Timeline, Master Plan
**Dev2:** Flutter home + timeline (leitura morador)

- [x] 2.1 Condominium Registry Go: `POST /condominiums` (ID alfanumérico 8 chars, QR Code), mandato (tipo, datas), limite 15 por síndico
- [x] 2.2 Unit Registry + Membership Go: `POST /condominiums/:id/units`, 1 titular + 2 dependentes max, encerramento de vínculo (4 motivos)
- [ ] 2.3 Telas S3-S5 SolidJS: Cadastrar Condomínio, Dados da Gestão, Identidade (ID + QR + compartilhar)
- [x] 2.4 Timeline Go: `POST /condominiums/:id/timeline` (append-only, sem DELETE/UPDATE), versionamento, adendo, filtros
- [x] 2.5 Timeline imutabilidade: após publicação record é locked; edição cria nova versão
- [x] 2.6 Master Plan Go: status automático via evento `TimelinePublished`
- [ ] 2.7 Telas S11-S14 SolidJS: Linha do Tempo (lista + filtros), Criar Atividade (26 tipos), Editar (cria versão), Histórico
- [ ] 2.8 Context Confirmation modal SolidJS: síndico multi-condo
- [ ] 2.9 Telas S7-S10 SolidJS: Plano Diretor, Cadastrar Ação, Lista Ações, Detalhe
- [ ] 2.10 Telas M1-M5 SolidJS: Painel Morador, Selecionar Condomínio, Buscar por ID, Cadastro Unidade, Home
- [ ] 2.11 Flutter: Home morador (M1-M5), Timeline leitura (M6-M7), Plano Diretor leitura (M8-M9)

---

## Sprint 3 — Institutional II: Governance Completa (05/05 → 18/05) — ✅ Dev1 backend completo

**Dev1 (Main Track):** Eventos, Comunicados, Validações, Dashboard, Compliance, Avaliação
**Dev2:** Flutter painel morador completo

- [x] 3.1 Eventos Go: CRUD + `POST /events/:id/respond` (ciente/confirmado), 13 tipos, métricas de participação
- [x] 3.2 Comunicados Go: CRUD + tracking de leitura por morador, comunicado de empresa entra como pendente de validação
- [x] 3.3 Validation Hub Go: lista pendentes, validate/reject/request-adjustment → publica Timeline automaticamente
- [x] 3.4 Pergunta aos Moradores Go: embutida em Criar Atividade, 4 tipos, resultado consolidado → Timeline
- [x] 3.5 Avaliação da Gestão Go: 3 perguntas, escala 1-10, anônima, bimestral obrigatória
- [x] 3.6 Compliance Go: termo de responsabilidade (IP + timestamp), declaração anual, audit trail append-only (5 anos LGPD)
- [x] 3.7 Dashboard Síndico Go: 13 indicadores, filtros por período
- [ ] 3.8 Telas S15-S17 SolidJS: Eventos (lista + métricas), Criar Evento (13 tipos)
- [ ] 3.9 Telas S23-S25 SolidJS: Comunicados, Criar Comunicado (8 tipos)
- [ ] 3.10 Tela S26 SolidJS: Validações Pendentes (3 seções), badge sidebar
- [ ] 3.11 Tela S27 SolidJS: Dashboard Síndico (charts, filtros)
- [ ] 3.12 Telas S28-S31 SolidJS: Compliance hub, Termo, Declaração Anual, Auditoria (imutável)
- [ ] 3.13 Telas M10-M15 SolidJS: Eventos, Comunicados, Avaliar Gestão, Meu Vínculo, Gerenciar Acessos, Encerrar Vínculo
- [ ] 3.14 Flutter: M10-M15 completos, push notifications setup (FCM)

---

## Sprint 4 — Commercial: Marketplace (19/05 → 01/06) — ✅ Dev1 backend completo

**Dev1 (Main Track):** Connect Me, Service & Contract, Company Profile
**Dev2:** Flutter painel empresa

- [x] 4.1 Connect Me Go: 4 fluxos, auto-close 48h, quotas anuais Redis, log LGPD (timestamp + IP + consent_version)
- [x] 4.2 Botão "Denunciar Assédio" Go: log com evidence, notifica admin, ações (warning/suspension/ban)
- [x] 4.3 Service & Contract Go: execution_records, technical_activities, workflows submit/validate/reject/adjust
- [x] 4.4 Proposals + Supplier Voting Go: CRUD, quórum configurável, resultado real-time
- [x] 4.5 Closed Deal Go: dupla assinatura → `is_immutable=true` → auto-publicação Timeline
- [x] 4.6 Company Profile + Users Go: perfil institucional, 2 roles (admin + operador técnico)
- [x] 4.7 Empresa Síndica Vinculada Go: invite via token, 5 permissões parametrizáveis, todas as ações auditadas
- [ ] 4.8 Telas S18-S20 SolidJS: Connect Me hub, Criar (LGPD checkbox), Recebido (interesse/recusar com motivo)
- [ ] 4.9 Telas E1-E12 SolidJS: Home Painel, Oportunidades, Registrar Execução, Atividade Técnica, Dashboard Empresarial, Perfil
- [ ] 4.10 Tela S5 SolidJS: Cadastro/Vínculo Empresa Síndica (campos + permissões toggle)
- [ ] 4.11 Flutter: E1 (Home), E2-E4 (Oportunidades), E5 (Registrar Execução + câmera), E10 (Dashboard)

---

## Sprint 5 — Content: Vídeos + Mux (02/06 → 15/06) — ✅ Dev1 backend (SDK real → Sprint 9)

**Dev1 (Main Track):** Video Library, Mux integration, Marketing Agency
**Dev2:** Flutter vídeo

- [x] 5.1 Mux Adapter Go: `IVideoProvider` → `MuxProvider` stub implementado — **SDK real (github.com/muxinc/mux-go) migrado no Sprint 9 task 9.1**
- [x] 5.2 Video Library Go: upload com trava trimestral (90 dias), limites mensais por plano (N1:4, N2:8, N3:12, Plus:8, Pro:12), 12 tipos de vídeo, `autorizar_exibicao_votacao` flag
- [x] 5.3 Regra de visibilidade Go: vídeos de empresa NÃO aparecem para moradores; apenas votação quando `autorizar_exibicao_votacao=true`
- [ ] 5.4 Player SolidJS: HLS.js, paywall 25% para Base em vídeos de empresa, contagem view (>70% = registra)
- [x] 5.5 Agência de Marketing Go: invite via token, permissões restritas (só vídeos + analytics), audit trail
- [ ] 5.6 Telas MK1-MK8 SolidJS: Painel Agência, Empresas, Publicar Vídeo (12 tipos + checkbox votação), Biblioteca, Dashboard
- [ ] 5.7 Telas E13-E16 SolidJS: Gestão Agência, Convidar, Gerenciar Acessos, Marketing Link
- [ ] 5.8 OpenSearch queries Go: swap provider busca (PostgreSQL tsvector → OpenSearch)
- [ ] 5.9 Search SolidJS: autocomplete, filtros, cards resultado, middleware visibilidade por plano
- [ ] 5.10 Flutter: player nativo (video_player), upload vídeo/foto, busca com filtros

---

## Sprint 6 — Integrações (16/06 → 29/06) — ⏳ email/push/sms → Sprint 9

**Dev1 (Main Track):** Push, SMS, Email, PDF, Mandato, Avaliação Empresa
**Dev2:** Flutter paridade completa

- [ ] 6.1 Push Notifications Go: FCM adapter, `IPushProvider`, device token management, notificações por tipo de evento
- [ ] 6.2 Twilio SMS Go: `ISMSProvider` → `TwilioProvider` (já stubado), OTP, notificações transacionais
- [ ] 6.3 Email Go: **Resend** adapter (github.com/resend/resend-go/v2) implementado Sprint 9 task 9.9 — `IEmailProvider`, templates pt-BR; futuro: Twilio como provider principal
- [ ] 6.4 CEP Lookup Go: ViaCEP adapter, `ICEPLookup`, caching Redis
- [ ] 6.5 PDF Generator Go: dossiê exportável, relatório de gestão
- [x] 6.6 Mandato Go: encerrar (requer compliance em dia), transferir (passa histórico, não passa avaliações pessoais)
- [x] 6.7 Company Evaluation Go: 5 critérios escala 1-10, anônima para empresa, pública para outros síndicos
- [x] 6.8 Post-Deal Comunicados Go: 5 tipos, validação síndico → Timeline + notifica moradores
- [ ] 6.9 Telas de proposta, votação, negócio fechado (termos + dupla confirmação), avaliação SolidJS
- [ ] 6.10 Flutter: paridade completa com web (Sprints 1-5), push notifications integradas
- [ ] 6.11 [Background] PDF templates: dossiê, relatório

---

## Sprint 7 — Assembly + Polish (30/06 → 13/07) — ✅ Dev1 backend completo

**Dev1 (Main Track):** Assembly assíncrona, bugs acumulados, performance, OTel
**Dev2:** Flutter assembly + QA

- [x] 7.1 Assembly assíncrona Go: configuração (tipo, modalidade, quórum, tempo de fala), pauta lock jurídico, edital (PDF), ciência bloqueante (tracking 6 meses)
- [x] 7.2 Assembly pré-assembleia Go: procurações (validação, 24h pós-votação), enquetes preliminares, simulador quórum, inadimplentes (planilha + deadline 1h antes)
- [x] 7.3 Assembly votação Go: simples, fração ideal (DECIMAL/NUMERIC — nunca float), fornecedor, eleição; voto presencial assistido; ausente = abstenção automática
- [x] 7.4 Assembly pós-assembleia Go: ata automática, 6 relatórios (CSV + PDF), score transparência 0-100 (10 componentes), auto-publicação Timeline
- [ ] 7.5 Telas de assembly SolidJS: configuração, pauta (drag-and-drop), edital, ciência, procurações, simulador, votação, relatórios
- [x] 7.6 Bug fixes acumulados: multi-tenancy, imutabilidade Timeline, PD reativo, trial retroativo, quotas Connect Me
- [x] 7.7 Performance tuning: N+1 queries, index optimization, Redis caching, connection pooling
- [ ] 7.8 Flutter: telas de assembly, QA, testes dispositivos reais (iOS + Android)
- [x] 7.9 OpenTelemetry: traces, metrics, logs via OTel SDK (pgx tracer, otelgin middleware, Prometheus bridge)

---

## Sprint 8 / Buffer — QA + Deploy Prod (14/07 → 08/08) — ⏳ E2E e deploy pendentes

**Dev1 + Dev2:** Testes E2E, security review, deploy produção, monitoring

- [ ] 8.1 Testes E2E: fluxo completo síndico (cria condo → timeline → PD → eventos → connect me → deal → execução → avaliação)
- [ ] 8.2 Testes E2E: fluxo morador (vincula → vê timeline → avalia → connect me → votação)
- [ ] 8.3 Testes E2E: fluxo empresa (recebe oportunidade → proposta → deal → execução → validação → Timeline)
- [ ] 8.4 Testes E2E: billing (trial → expiração → bloqueio retroativo → upgrade → desbloqueio)
- [x] 8.5 Testes ABAC: cada role vê apenas o que deve; operador técnico vs admin; cross-tenant isolation
- [x] 8.6 Security review BeyondCorp: 1 device, cookie httpOnly, rate limiting, LGPD logs, audit trail
- [ ] 8.7 Deploy produção: **Railway** (PostgreSQL + Redis + API container + MinIO), Zitadel prod, Stripe live mode, smoke tests, domínios custom + SSL
- [ ] 8.8 Monitoring: OTel → Railway metrics, alertas críticos, runbooks operacionais
- [ ] 8.9 Flutter: submit TestFlight (iOS) + Google Play Internal Testing (Android)
- [x] 8.10 Handoff docs: API docs (routes.http + OpenAPI/Swagger), README, runbook de deploy

---

## Sprint 9 — Integrações de Produção (iniciando 2026-04-22)

**Dev1 (Main Track):** Mux SDK real, Livekit, Resend, MinIO, Railway deploy, CI/CD
**Marco:** Deploy staging funcionando com todos os providers + Livekit rodando localmente

**Status 2026-04-22**: maioria entregue nos commits f8ce6d4, 58f171e, 48f842f, 39873f1 e 5114325; reconciliação feita contra o código real (wire-up em `content/module.go:42,105` e `assembly/module.go:42,95-97`).

### Mux (vídeo VOD + live CDN)

- [x] 9.1 Mux SDK Go: `MuxProvider` real com `github.com/muxinc/mux-go` — upload direto (signed URL), GetAsset, DeleteAsset, GetPlaybackURL, GetThumbnailURL. Wirado em `content/module.go:42`.
- [x] 9.2 Mux Live Streams Go: `ILiveStreamProvider` → `MuxProvider` — CreateLiveStream, GetLiveStream, EndLiveStream (RTMP→HLS). Arquivo `content/infrastructure/providers/i_live_stream_provider.go`.
- [x] 9.3 Mux Webhooks Go: `POST /webhooks/mux` em `content/module.go:105` + validação HMAC-SHA256 do header `Mux-Signature` (commit 39873f1).
- [x] 9.4 Mux Config: `VideoProviderConfig` com `MUX_TOKEN_ID`, `MUX_TOKEN_SECRET`, `MUX_WEBHOOK_SECRET` em `config.go`.

### Livekit (WebRTC assembly ao vivo)

- [x] 9.5 Livekit no Docker Compose: service `livekit/livekit-server:latest`, portas 7880/7881/7882 UDP, `livekit.yaml` — feito 2026-04-22.
- [x] 9.6 Livekit Provider Go: `ILiveProvider` (em `assembly/domain/providers/`) → `LivekitProvider` em `assembly/infrastructure/providers/livekit_provider.go` com `github.com/livekit/server-sdk-go/v2` — decisão arquitetural de manter o provider dentro do bounded context de assembly em vez de `shared/providers/live/`.
- [x] 9.7 Assembly Live Use Cases Go: `StartLiveSessionUseCase`, `JoinLiveSessionUseCase`, `EndLiveSessionUseCase` em `assembly/application/use_cases/live_use_cases.go`, wirados em `assembly/module.go:95-97`.
- [x] 9.8 Livekit Config: `LiveConferenceConfig` com `LIVE_HOST`, `LIVE_API_KEY`, `LIVE_API_SECRET` em `config.go:67`; defaults dev em `config.go:466-474`.

### Resend (email transacional)

- [x] 9.9 Resend Provider Go: `ResendProvider` com `github.com/resend/resend-go/v2` — `Send`, `SendTemplate`, substitui SES.
- [x] 9.10 Email Templates Go: **7 templates pt-BR** em `resend_provider.go:renderLocalTemplate` — `verificacao-conta`, `connect-me`, `boas-vindas`, `assembly-convocacao`, `assembly-ata-publicada`, `billing-trial-expirando`, `billing-cobranca-falhou`. Cada um devolve trio (html, text, subject). **Migração para `embed.FS templates/*.html`** fica para quando volume passar de ~15 templates.
- [x] 9.11 Resend Config: `EmailProviderConfig` com `RESEND_API_KEY`, `EMAIL_FROM` em `config.go`.

### MinIO (storage S3-compatible)

- [x] 9.12 MinIO no Docker Compose: service `minio/minio:latest`, portas 9000/9001 — feito 2026-04-22.
- [x] 9.13 Storage Provider Go: `IStorageProvider` → `MinIOProvider` com `github.com/minio/minio-go/v7` — UploadFile, GetPresignedUploadURL, GetPresignedDownloadURL, DeleteFile.
- [x] 9.14 MinIO Config: `StorageProviderConfig` em `config.go` com endpoint, access/secret key, buckets nomeados e SSL flag.
- [x] 9.15 Buckets iniciais: `docs`, `images`, `assembly` — auto-criados pelo init do docker-compose.

### Railway Deploy + CI/CD

- [x] 9.16 Dockerfile otimizado: multi-stage (builder + distroless runner), labels OCI, health check `GET /health` (commit 5114325).
- [x] 9.17 railway.toml: deploy config, start command, health check, env vars referenciando Railway secrets.
- [x] 9.18 GitHub Actions CI: `.github/workflows/ci.yml` com build + vet + test; deploy Railway no push para main.
- [ ] 9.19 Zitadel em Railway: atualmente rodando em `docker-compose.yml` via Traefik + zitadel-login UI (commit 218a7a3). **Pendente**: promover para Railway managed, domínio `auth.mastersindico.com.br`, configuração Organization + Projects + Roles via `cmd/zitadel-setup`.

### Restante do Sprint 9 — tasks abertas

1. **9.19** — deploy Zitadel em Railway com DNS próprio (requer ação fora do código — provisionar Railway service + DNS).
2. **DT-010** (STATE.md) — `IAuditLogger` cross-módulo (billing, commercial, identity); alvo declarado Sprint 9.
3. **A-027 + A-028** (AUDIT.md) — Saga orquestrada para UploadVideo + StartLiveSession (providers externos órfãos). Sprint 10 target.

### Sprint 9 — Hardening entregue na sessão autônoma 2026-04-22 (sub-tasks novas)

- [x] 9.H1 Error handling crítico: 8 `_ = fn()` fixados (A-001 a A-008: webhook close, event publish, audit, DecodeJSON). `go vet` + gosec limpos.
- [x] 9.H2 Pacote `internal/shared/pagination` + `pkg/safecast` — substituiu `strconv.Atoi + int32()` em 11 handlers e 10 repositories. Gosec G109/G115: 66 → 0.
- [x] 9.H3 Logger err.Error() → err direto em 6 arquivos (preserva stack). Webhook Stripe/Mux agora incluem request_id + ip nos logs.
- [x] 9.H4 Code Calisthenics: else block eliminado em `register_membership_use_case` (strategy com `checkTitularCapacity`/`checkDependentCapacity`).
- [x] 9.H5 CORS wildcard: `Config.Validate()` rejeita `*` em ambiente staging/production.
- [x] 9.H6 `.gitignore` hardening: `zitadel-key*.json` + binários `/migrate`, `/zitadel-setup` (A-018).
- [x] 9.H7 PBT com `pgregory.net/rapid` v1.2.0: primeira suíte em `agenda_item.SetFractionIdeal` (fração ideal, regra steering/testing.md #1) + PBT em `pagination.parseBounded`.
- [x] 9.H8 Integration tests com `testcontainers-go`: infra compartilhada `internal/shared/testutil/pg.go` (SetupPostgres + TruncateAll CASCADE) + 6 tests com tag `//go:build integration` (condominium, user, subscription, feature_usage, timeline, supplier_vote).
- [x] 9.H9 Mux webhook movido para rota pública `POST /webhooks/mux` fora de `/api/v1` (A-022; antes `Authenticate` bloqueava Mux porque não envia token Zitadel).
- [x] 9.H10 A-025 Assembly Vote: double-voting impossível agora — migration 00504 `UNIQUE(agenda_item_id, voter_id)` + VoteRepository mapeia 23505 para `ErrConflict`.
- [x] 9.H11 A-026 Commercial Vote contador: `CastVoteUseCase` envolve `FindSession + CastVote + Increment*` em `unitOfWork.Run` — atomicidade garantida.
- [x] 9.H12 Recortes `requirements/*.md`: 8 arquivos criados (identity, billing, institutional, commercial, content, assembly, cross-domain, functional-matrix) extraindo do monolítico.
- [x] 9.H13 `backend/CLAUDE.md` enxugado + `backend/.kiro/SESSION_CHARTER.md` criado para rastrear ordens de sessões autônomas longas.

Detalhes: ver `.kiro/AUDIT.md` (A-017 a A-034) e `.kiro/SESSION_CHARTER.md` §9.

---

## Pós-Lançamento (Sprints 10+)

- [ ]* P.1 Assembly ao vivo completa: Livekit WebRTC (SDK do Sprint 9), telão 2 áreas, fila de fala com cronômetro, votação real-time, contingência, NATS JetStream fan-out — 8-10 semanas
- [ ]* P.2 Academia LMS: 15 telas, 5 áreas (cursos, treinamentos, lives, fórum, biblioteca), 3 trilhas — 4-5 semanas
- [ ]* P.3 Vizinhança completa: 29 telas, 4 jornadas (Síndico, Empresa, Morador, Parceiro) — 3-4 semanas
- [ ]* P.4 Banco de Talentos: 11 telas morador + 7 funcionalidades empresa, vídeo-currículo 90s (Mux upload) — 3-4 semanas
- [ ]* P.5 Painel Admin MasterSíndico: gestão global de tenants, moderação, financeiro, broadcasts — 3-4 semanas
- [ ]* P.6 Mux Live para LMS/Academy: RTMP ingest → HLS CDN para lives de cursos
- [ ]* P.7 Twilio como provider principal de email: migrar de Resend para Twilio SendGrid quando volume justificar
- [ ]* P.8 OpenSearch swap: PostgreSQL tsvector → OpenSearch para busca avançada (companies, sindicos, parceiros, videos)
