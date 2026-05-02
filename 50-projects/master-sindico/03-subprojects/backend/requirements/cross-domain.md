---
title: Cross-Domain — Backend Requirements (Go)
type: requirement
tags: [master-sindico, backend, go, cross-domain, events, abac, audit, lgpd, otel, feature-req]
project: master-sindico
stack: backend
bounded_context: cross-domain
milestone: m1
status: partial
created: 2026-04-24
---

# Cross-Domain — Backend Requirements (Go)

## Escopo do bounded context

Módulo Go `internal/shared/*` + `internal/infrastructure/*` responsável por **recursos transversais** que atravessam bounded contexts sem virar um BC próprio:

- **`IEventBus`** — publicação de domain events entre BCs; atualmente `InMemoryPublisher` (sync + in-process); NATS JetStream threshold via ADR (XD-001).
- **`IAuditLogger`** (DT-010 — **aberta**) — interface canônica em `internal/shared/audit`; hoje está acoplada em identity e usada como shim cross-BC.
- **Outbox pattern** (XD-002) — futura: `outbox_events` + drain worker p/ garantir at-least-once.
- **Shared types interfaces** — `ITrialChecker`, `IQuotaConsumer`, `IValidationSubmitter`, `ITimelinePublisher`, `IUserModerator`, `ICondominiumIDsProvider` (solução XD-003 no-cross-BC-import).
- **Provider abstractions** — `IPaymentGateway`, `IVideoProvider`, `ILiveProvider`, `IEmailProvider`, `ISMSProvider`, `IStorageProvider`, `ICEPLookup`, `ICNPJValidator`, `IPushProvider` (XD-012 a XD-020).
- **ABAC engine** em `internal/shared/middleware` — matriz ordenada (admin_bypass → role → action → tenant → scope), cache Redis, invalidação por webhook.
- **Audit trail append-only** em `audit_trail` table; trigger `prevent_mutation_audit_trail`.
- **LGPD logs** em `lgpd_logs` table — separada de audit trail; retenção 5 anos.
- **OpenTelemetry + Sentry** (XD-031, XD-032) — spans, metrics, traces, erros sanitizados (PII scrubber).
- **Health checks** `/healthz` (liveness) e `/readyz` (readiness com DB+Redis+NATS).
- **Linter / arch fitness function** (XD-003) — bloquear cross-BC import em CI.

**Não é responsabilidade**: nenhum aggregate de negócio vive aqui; recursos técnicos e contratos que múltiplos BCs consomem.

## Status atual (vault canônico)

- **Sprint 2 + 5 + 7 (legado)**: ABAC engine, middleware Authenticate, audit_trail append-only, sentry + OTel base.
- **A-###** fechados: A-001..A-006 (ABAC matrix + middleware), A-021 (DB trigger prevent_mutation_audit_trail), A-FASE10-FIT-001 (linter ArchUnit-like custom).
- **A-### abertos**:
  - **DT-010 aberta** — `IAuditLogger` interface extraction de identity → `internal/shared/audit` com handler async via channel buffered.
  - **A-023 aberto** — `IUserModerator` interface exposta via `internal/shared/types`.
  - **A-031 aberto** — `ITimelinePublisher` interface contract test cross-BC.
  - **XD-001 decisão** — NATS JetStream threshold (hoje InMemoryPublisher; critério: N events/s ou latência).
- **D-###**: ADR-0012 (ABAC Redis cache), ADR-0027 (webhook idempotency), ADR-0028 (LGPD keyed HMAC), ADR-0034 (RLS default-on), ADR-0042 (search real M1 — pending dual-check); D-026 (Resend vs SES) fechada por D-100 (Resend M1-M2, SES M3+); D-067 (tsvector vs OpenSearch) **PARCIAL-REVOGADA** por D-120 (search real M1, tsvector é stub); D-024 (Railway → ECS) ainda aberta.

## Agregados (domain)

Este BC tem "pseudo-agregados" técnicos:

- [[../../../01-domain/aggregates/DomainEvent]] — envelope canônico `{id, subject, payload, metadata, occurred_at}`.
- [[../../../01-domain/aggregates/AuditEntry]] — append-only `audit_trail`.
- [[../../../01-domain/aggregates/OutboxEntry]] — futuro XD-002.
- [[../../../01-domain/aggregates/AbacDecision]] — VO retornado pelo engine.
- `LGPDLog` — retenção 5 anos; purpose/legal_basis enum.
- `RateLimitEntry` — Redis-backed.
- `WebhookEventRecord` — UNIQUE (provider, event_id) (ADR-0027).
- `FeatureFlagRollout` — hash(user_id + feature_key) % 100 < percentage.

## Entidades e Value Objects

| Tipo | Elemento | Invariante chave |
|---|---|---|
| VO | `DomainEvent` | `subject = "{bc}.{aggregate}.{action}"` padronizado |
| VO | `AuditEntry` | append-only; retenção 10 anos + archive S3 Glacier pós-1 ano |
| VO | `LGPDLog` | `legal_basis ∈ {consent, contract, legal_obligation, vital_interest, public_task, legitimate_interest}` |
| VO | `Purpose` | taxonomia fixa: billing, fraud_prevention, customer_support, tax_reporting, marketing_optout, connect_me_reveal, content_publication, assembly_record, ... |
| VO | `ABACDecision` | `{allowed, role, plan_tier, reason}` imutável |
| VO | `RateLimitDecision` | `{allowed, retry_after_seconds}` |

## Invariantes (INV-XD-###)

1. **INV-XD-001**: Audit trail append-only — nunca DELETE, nunca UPDATE (trigger).
2. **INV-XD-002**: LGPD logs retenção 5 anos + purga pós-vencimento.
3. **INV-XD-003**: Governance Score: calc diário, histórico 12 meses.
4. **INV-XD-004**: Todas assinaturas (terms, proxy, deal): rastreadas `{timestamp, ip, version}`.
5. **INV-XD-005**: Integrações agnósticas (interfaces IPaymentGateway, IVideoProvider, etc.) — D-056/D-071 Fase 7.
6. **INV-XD-006**: Cache ABAC invalidação via webhook (não stale 5min).
7. **INV-XD-007** (INV-119): Nenhum cross-BC import (XD-003) — linter/ArchUnit bloqueia CI.
8. **INV-XD-008**: Webhook HMAC verify antes de parse; UNIQUE `(provider, event_id)` idempotência.
9. **INV-XD-009** (INV-109): Pseudonimização LGPD via keyed HMAC rotating (ADR-0028).
10. **INV-XD-010**: OpenTelemetry root span por request; spans filhos por use case + SQL query.

## Eventos de domínio

Este BC **expõe o transporte** mas não emite eventos próprios de negócio. Publica tecnicamente:

- `shared.cache.invalidated {key_pattern}` (quando invalidation ocorre).
- `shared.outbox.drained {count, duration_ms}` (drain worker).
- `shared.audit.flushed {entries_written}`.
- `shared.linter.violation_detected {rule, location}` (dev-only).

Consome **de TODOS** para:

- `IAuditLogger.Log` em cada evento crítico.
- `IEventBus.Publish` rotea; consumers registram handlers.

## Requirements funcionais (FR-BE-XD-###)

- **FR-BE-XD-001** (XD-001) — *Quando* BC produz evento, *o backend deve* expor `IEventBus.Publish(ctx, subject, payload)`; default `InMemoryPublisher` (sync in-process para M1-M2); NATS JetStream threshold via ADR quando criterion hit (ex: >100 events/s ou latência >50ms p95); at-least-once durability.
- **FR-BE-XD-002** (XD-002) — *Quando* handler escreve em DB + precisa publicar evento crítico, *o código deve* usar outbox: (a) INSERT em `outbox_events {id, subject, payload, status, attempts, next_attempt_at}` na mesma UoW, (b) drain worker (cron 10s) publica; retry com exponential backoff; idempotent.
- **FR-BE-XD-003** (XD-003 + INV-XD-007) — *Quando* CI roda, *a arch fitness function deve* grep import statements em `internal/modules/<bc>/` e bloquear imports cruzados com `internal/modules/<other_bc>/domain/|application/|infrastructure/`; apenas `internal/shared/types` e `internal/shared/providers` permitidos.
- **FR-BE-XD-004** (XD-004) — *Quando* request chega handler protegido, *o middleware `RequirePermission(action, resource)` deve* avaliar ordem: (1) admin_bypass, (2) role_check → `403 ROLE_NOT_ALLOWED`, (3) action_check `plan.features[action]` → `402 PLAN_REQUIRED` ou `403 ACTION_NOT_CONFIGURED`, (4) tenant_check → `403 TENANT_MISMATCH`, (5) scope_check → `403 OUT_OF_SCOPE`, (6) rollout → `404 NOT_FOUND` (experimental hidden), (7) allow.
- **FR-BE-XD-005** (XD-005) — *Quando* ABAC resolve decisão, *o engine deve* escrever cache Redis `ms:abac:{user_id}:{tenant_id}` TTL 5min; hit < 1ms; miss → DB query + write cache.
- **FR-BE-XD-006** (XD-006) — *Quando* evento afeta permissões (`billing.subscription.changed`, `identity.user.role_changed`, `identity.user.banned`, `commercial.company.suspended`, `content.video.approved` → granting visibility), *o listener deve* emitir `DEL ms:abac:{user_id}:*` (Redis SCAN + DEL batched).
- **FR-BE-XD-007** (XD-007) — *Quando* seed migration roda, *o backend deve* popular `abac_policies {action, resource, roles_allowed[], plan_tiers_allowed[], scope_required}` com matriz canônica (ver [[../../../04-requirements/matrix-functional]]); frontend renderiza botão apenas se ABAC pré-resolve OK.
- **FR-BE-XD-008** (XD-008 + DT-010) — *Quando* evento crítico ocorre em qualquer BC, *o código deve* chamar `IAuditLogger.Log(ctx, AuditEntry {user_id, tenant_id, action, resource_type, resource_id, changes_before, changes_after, ip, user_agent, request_id, occurred_at})`; implementação default = channel buffered assíncrono + flush worker → INSERT `audit_trail` batched; **extração pendente** de identity → `internal/shared/audit` (DT-010).
- **FR-BE-XD-009** (XD-009 + INV-XD-001) — *Enquanto* row existe em `audit_trail`, *o DB trigger `prevent_mutation_audit_trail` deve* rejeitar `UPDATE|DELETE` exception; retenção 10 anos + archive S3 Glacier após 1 ano.
- **FR-BE-XD-010** (XD-010) — *Quando* dado pessoal é revelado/tratado, *o código deve* INSERT `lgpd_logs {id, user_id, data_subject_id, action, purpose, legal_basis, ip, terms_version, related_resource_type, related_resource_id, occurred_at, retention_until}`; append-only; retenção 5 anos; purga via cron mensal.
- **FR-BE-XD-011** (XD-011) — *Quando* cron noturno, *o job `CalcGovernanceScore` deve* consolidar 10 componentes (INS-032) por condomínio; alerta `< 60` ao síndico via `IPushProvider` + `IEmailProvider`.
- **FR-BE-XD-012** (XD-012) — *Quando* billing interage com Stripe, *o código deve* usar `IPaymentGateway` interface em `internal/shared/providers/payment`; SDK Stripe isolado em `internal/infrastructure/providers/stripe/`; métodos: `CreateCheckoutSession, CreateBillingPortalSession, UpsertSubscription, CreateRefund, ParseWebhookEvent, ListInvoices`; mock para tests.
- **FR-BE-XD-013** (XD-013) — *Quando* content interage com Mux, *o código deve* usar `IVideoProvider` em `internal/shared/providers/video`; métodos: `CreateDirectUpload, GetAsset, CreateSignedPlaybackURL, ParseWebhookEvent, DeleteAsset`; SDK Mux isolado.
- **FR-BE-XD-014** (XD-014) — *Quando* assembly (M4+) interage com Livekit, *o código deve* usar `ILiveProvider` em `internal/shared/providers/live`; métodos: `CreateRoom, CreateJoinToken, MuteParticipant, StartRecording, StopRecording, EndRoom`.
- **FR-BE-XD-015** (XD-015 + D-026) — *Quando* sistema envia email, *o código deve* usar `IEmailProvider`; default Resend (D-026 aberta vs SES); templates pt-BR embedded: `welcome, password_reset, assembly_notice, connect_me_interest, broadcast, refund_approved, step_up_mfa_fallback, harassment_admin_alert, dispute_alert, mandate_transfer_confirmation, co_admin_invite`.
- **FR-BE-XD-016** (XD-016) — *Quando* sistema envia SMS (OTP, notif crítica), *o código deve* usar `ISMSProvider` (Twilio default, Zenvia fallback); rate 1 SMS/min/user; consent opt-in (LGPD).
- **FR-BE-XD-017** (XD-017) — *Quando* sistema persiste objeto, *o código deve* usar `IStorageProvider` (MinIO dev, Cloudflare R2 ou AWS S3 prod); buckets: `videos-handled-by-mux`, `documents`, `media`, `public`, `assembly-minutes`, `dossiers`; presigned URL TTL 24h default; lifecycle archive 1 ano + delete 7 anos.
- **FR-BE-XD-018** (XD-018) — *Quando* backend valida CEP, *o código deve* usar `ICEPLookup` (ViaCEP default); cache Redis `cep:{cep}` TTL 30d; fallback manual se indisponível.
- **FR-BE-XD-019** (XD-019) — *Quando* backend envia push, *o código deve* usar `IPushProvider` (FCM); registro `device_token` ao login mobile; topics `assembly_{cond_id}`, `user_{user_id}`; opt-in LGPD.
- **FR-BE-XD-020** (XD-020) — *Quando* backend valida CNPJ, *o código deve* usar `ICNPJValidator`; M1 stub (algoritmo dígito verificador); M2+ adapter Receita Federal (ReceitaWS? SERPRO? — ADR pendente).
- **FR-BE-XD-021** (XD-025) — *Quando* user aceita termo, *o código deve* persistir `user_term_acceptances` append-only (IDN-024); re-aceite em nova versão; audit.
- **FR-BE-XD-022** (XD-031) — *Quando* erro não-operacional ocorre, *o logger deve* enviar Sentry com `request_id` tag, stack sanitizado via scrubber regex PII (email, CPF, cartão, nome completo); projects: backend, web, mobile.
- **FR-BE-XD-023** (XD-032) — *Quando* request entra, *o middleware deve* criar root span `http.request`; cada use case → span filho `usecase.{bc}.{name}`; cada SQL query → span `sql.query`; metrics: counters `events_published_total`, histograms `usecase_duration_ms`, `sql_query_duration_ms`; exporter OTel collector gRPC.
- **FR-BE-XD-024** (XD-034) — *Quando* `GET /healthz`, *o handler deve* retornar `200 {status: "ok"}` se processo vivo; *quando* `GET /readyz`, *o handler deve* verificar DB (`SELECT 1`) + Redis (`PING`) + NATS (se habilitado); todos OK → `200`; senão → `503`.
- **FR-BE-XD-025** (XD-035) — *Quando* admin liga feature flag com `rollout_percentage`, *o código deve* decidir via `hash(user_id + feature_key) % 100 < percentage`; default `false` (fail-closed); tabela `feature_flags`; admin UI.
- **FR-BE-XD-026** (INV-XD-008 + ADR-0027) — *Quando* webhook chega de qualquer provider (Stripe, Mux, Livekit), *o handler deve* (a) HMAC verify via SDK provider antes de parse, (b) `INSERT INTO webhook_events (provider, event_id, ...) ON CONFLICT DO NOTHING`, (c) processar dentro da mesma UoW marcando `status=processed`; replay protection via tolerance 5min.
- **FR-BE-XD-027** — RLS default-on (ADR-0034): enable RLS em toda tabela com `tenant_id|condominium_id|user_id|company_id`; policies `USING (col = current_setting('app.<context>')::type)`; role app **sem** `BYPASSRLS`; role admin com bypass explícito.
- **FR-BE-XD-028** — **Pseudonimização LGPD** (ADR-0028 + INV-XD-009): helper `Pseudonymize(entity_id, purpose, rotation_date) string` aplica HMAC keyed com `lgpd_pseudonym_keys` current; `purpose ∈ {timeline_anon, vote_anon, audit_anon, contract_anon, lgpd_log_anon}`; keys rotacionadas 1/jan anualmente.
- **FR-BE-XD-029** — **Shared types interfaces** em `internal/shared/types/`:
  - `ITrialChecker.StartTrial|IsTrialActive`
  - `IQuotaConsumer.Consume|Release`
  - `IQuotaCheck.Check`
  - `IFeatureFlagChecker.IsEnabled`
  - `ITimelinePublisher.Publish`
  - `IValidationSubmitter.SubmitExecutionRecord`
  - `IUserModerator.BanUser|SuspendUser|RevokeSessions`
  - `ICondominiumIDsProvider.ListByUser|GetMembershipType`
  - `IReputationReader.ReadTier|ReadBreakdown`
  - `IAuditLogger.Log`
- **FR-BE-XD-030** — **Rate limit middleware** unificado via Redis token bucket; config per-endpoint via tabela `rate_limit_rules`; resposta 429 com `Retry-After` header; logged em OTel metrics.

## Endpoints REST expostos

Este BC expõe principalmente **endpoints técnicos**, não de negócio:

| Método | Path | Auth | Descrição | Status |
|---|---|---|---|---|
| GET | /healthz | público | Liveness | ✅ |
| GET | /readyz | público | Readiness (DB+Redis+NATS) | ✅ |
| GET | /metrics | admin | Prometheus metrics | ✅ |
| GET | /version | público | Build version + commit SHA | ✅ |
| POST | /webhooks/:provider | público + HMAC + UNIQUE | Webhook router (stripe, mux, livekit) | ✅ |
| GET | /admin/v1/audit-trail | Bearer + admin | Query audit_trail paginado | 🟡 |
| GET | /admin/v1/lgpd-logs | Bearer + admin | Query lgpd_logs | 🟡 |
| GET | /admin/v1/feature-flags | Bearer + admin | Lista + rollout % | ⏳ |
| PATCH | /admin/v1/feature-flags/:key | Bearer + admin | Toggle + rollout | ⏳ |
| GET | /admin/v1/outbox-status | Bearer + admin | Drain worker metrics | ⏳ XD-002 |
| GET | /admin/v1/system-health | Bearer + admin | DB+Redis+NATS+providers status | 🟡 |

## Integração com outros BCs

Este BC **expõe interfaces** consumidas por **TODOS** os BCs. Não consome nada de BC específico (por design — cross-BC sem imports).

**Expõe**:

- `IEventBus` (+ InMemoryPublisher / NATSPublisher)
- `IAuditLogger` (DT-010)
- `IABACEngine` (via middleware)
- `RequirePermission(action, resource)` factory
- `IRateLimiter`
- `IHealthChecker`
- Todos os `I*Provider` (payment, video, live, email, sms, storage, cep, cnpj, push)
- Todas as `Shared Types` interfaces (ITrialChecker, IQuotaConsumer, etc.)
- `Pseudonymize(entity_id, purpose, rotation_date)` helper

**Consome eventos para infra**: recebe eventos de TODOS para `IAuditLogger.Log` + cache invalidation.

## Persistência

**Tabelas principais**:

- `audit_trail` — append-only; trigger `prevent_mutation_audit_trail`; retenção 10 anos; archive S3 Glacier pós-1 ano; index `(actor_id, occurred_at)`, `(entity_type, entity_id, occurred_at)`.
- `lgpd_logs` — append-only; retenção 5 anos; `purpose` + `legal_basis` enum; purga via cron mensal.
- `abac_policies` — matriz seed.
- `outbox_events` — XD-002 futura.
- `webhook_events` — UNIQUE `(provider, event_id)`.
- `feature_flags`, `feature_flags_rollout` — admin toggles.
- `rate_limit_rules` — per-endpoint config.
- `lgpd_pseudonym_keys` — rotacionadas 1/jan; append-only.
- `user_term_acceptances` — cross-BC termos versionados.
- `cep_cache` — opcional (Redis é primário).
- `system_config` — configs globais editáveis (CNT-029).

**Migrations**: `00000..00099` (core) + `01100..01199` (cross-domain seeds).

**RLS default-on**: `audit_trail` sem RLS (admin-only read); `lgpd_logs` com RLS por `user_id`; `webhook_events` sem RLS.

## Providers externos

Todos os providers externos são encapsulados aqui via interfaces:

- **Stripe** (XD-012 — ADR-0004).
- **Mux** (XD-013).
- **Livekit Cloud** (XD-014 — M4+).
- **Resend / AWS SES** (XD-015 — D-026 aberta).
- **Twilio / Zenvia fallback** (XD-016).
- **Cloudflare R2 / AWS S3 / MinIO dev** (XD-017).
- **ViaCEP** (XD-018).
- **ReceitaWS / SERPRO** (XD-020 — ADR pendente M2+).
- **FCM** (XD-019).
- **Sentry** (XD-031).
- **OpenTelemetry Collector** (XD-032).

**Sagas cross-domain**:

- **AccountDeletionSaga** (ADR-0030) — orquestra identity + billing + content + institutional + assembly + commercial para soft-flag + pseudonimização + hard-delete no D+15.
- **DrainOutboxSaga** (futuro XD-002) — cron worker lê outbox → publish via NATS → mark processed; idempotent.

## Testes

**Coverage target**: Middleware 95%, Shared types contracts 100%, Providers adapters 70%.

**PBT**:

- ABAC ordem fixa: `∀ request → reason ∈ {admin_bypass, matrix_deny, plan_missing, quota_exceeded, tenant_out_of_scope, experimental_hidden, allowed}` exatamente 1.
- Webhook idempotência: `∀ N deliveries same event_id → 1 processamento`.
- Cross-BC no import: ArchUnit linter em CI testa cada módulo (INV-XD-007).
- Pseudonymize determinismo + rotação: `Pseudonymize(id, purpose, date)` determinístico por `(id, purpose, key_version)`; diferente após rotação.
- Feature flag rollout: `∀ user_id → hash determinístico constante entre deploys`.

**Integration**:

- Webhook handler end-to-end (Stripe + Mux fixtures) com HMAC + UNIQUE.
- OutboxDrainSaga recovering após crash.
- AuditLogger async flush → DB batched write.
- Health check com DB down / Redis down.
- ArchUnit linter CI fails se import cross-BC introduzido.

## Segurança específica do BC

**LGPD compliance**:

- `lgpd_pseudonym_keys` rotation annual (1/jan) — histórico preservado para replay.
- Helper `Pseudonymize(entity_id, purpose, rotation_date)` central — toda pseudonimização passa aqui.
- `lgpd_logs` purpose taxonomia fixa (lint CI rejeita free-form).
- Sentry scrubber regex: email, CPF (`\d{3}\.\d{3}\.\d{3}-\d{2}`), cartão (Luhn-like), nome completo (heurística).

**BeyondCorp server-side** (ver [[../security]]):

- Toda request atravessa: `Authenticate` → `TenantScope` → `RequirePermission` → handler.
- `Authenticate` valida Zitadel introspection + cookie/Bearer; cache 5min.
- `TenantScope` seta `app.current_user_id` + `app.user_condominium_ids[]` via `SET LOCAL` na conexão pg (RLS).

**Rate limits globais**:

- `/webhooks/*` — sem RL (provider controla).
- `/api/v1/*` — 100 rpm per user base; overrides per-endpoint.
- `/admin/v1/*` — 30 rpm per admin + audit trail de cada call.
- `/auth/*` — ver identity.md.

## Decisões e dívidas

**D-###**:

- ADR-0012 (ABAC Redis cache) — fechado.
- ADR-0027 (webhook idempotency DB) — fechado.
- ADR-0028 (LGPD keyed HMAC rotating) — fechado.
- ADR-0030 (saga orchestration default) — fechado.
- ADR-0034 (RLS default-on) — fechado.
- D-024 (Railway → AWS ECS Fargate migration) — **aberta**; dependente de load.
- D-026 (Resend vs AWS SES) — **aberta**.
- D-067 (tsvector substitui OpenSearch temporariamente) — **PARCIAL-REVOGADA** por D-120 Fase 14 (search real M1 obrigatório; tsvector vigente é stub).
- D-080 (Stack stale reconciliação SES→Resend / OpenSearch→tsvector / AWS→Railway) — fechada Fase 7; SES→Resend e AWS→Railway permanecem; OpenSearch→tsvector revogado por D-120.
- **XD-001 threshold decisão NATS** — research-loop: hoje InMemoryPublisher; trigger migração para NATS JetStream quando N events/s atingido.

**DT-###**:

- **DT-010 aberta** — `IAuditLogger` interface extraction de identity → `internal/shared/audit`. Shim atual em identity causa:
  - Cross-BC import implícito via import identity por outros BCs.
  - Sync logging que bloqueia request path (deve ser async via channel).
  - Falta de reusable interface para mock em testes.

**Pendências**:

- XD-001 (NATS threshold criterion).
- XD-015 (Resend vs SES — D-026).
- XD-017 (Cloudflare R2 vs AWS S3 direto — custo).
- XD-020 (ReceitaWS vs SERPRO — ADR M2+).
- XD-033 (cronograma Railway → ECS).

## Gaps e bloqueadores Sprint 10 (M1)

1. **⚠️ DT-010 `IAuditLogger` extraction** — prioridade alta; melhora testabilidade cross-BC e desacopla identity.
2. **`IUserModerator`, `ITimelinePublisher`, `ICondominiumIDsProvider` interfaces** (A-023, A-031) — expor via `internal/shared/types` para evitar cross-BC import em commercial/content/assembly.
3. **Outbox pattern** (XD-002) — 🟡 InMemoryPublisher suficiente para M1 mas events críticos (deal.signed_complete, assembly.closed) precisam outbox antes de M3.
4. **RLS default-on completude** — `audit_trail` sem RLS (by design); verificar cobertura em tabelas tenant-scoped novas (units, fraction_snapshots).
5. **ArchUnit/linter CI** (A-FASE10-FIT-001) — implementado mas **não é fail-hard em todos repos**; promover a fail em M1 lockdown.
6. **LGPD purge cron** — `lgpd_logs` com `retention_until < NOW()` → purga mensal; cron **não configurado** ainda em prod.
7. **Sentry PII scrubber** — regex implementado mas **não cobre todos os casos** (endereços, telefones formatos diversos); audit pré-M1.

## Ligações

- [[../../../04-requirements/functional/cross-domain]] — requirement canônico global
- [[../../../01-domain/aggregates/DomainEvent]]
- [[../../../01-domain/aggregates/AuditEntry]]
- [[../../../01-domain/aggregates/OutboxEntry]]
- [[../../../01-domain/aggregates/AbacDecision]]
- [[../../../01-domain/aggregates/_moc]]
- [[../../../02-architecture/adr/0012-abac-redis-cache]]
- [[../../../02-architecture/adr/0027-webhook-idempotency-db]]
- [[../../../02-architecture/adr/0028-lgpd-keyed-hmac]]
- [[../../../02-architecture/adr/0030-saga-orchestration-default]]
- [[../../../02-architecture/adr/0034-postgres-rls-default-on-tenant-guard]]
- [[../../../02-architecture/_moc]]
- [[../../../11-audit/AUDIT]]
- [[../architecture]] — Clean Arch 4 camadas
- [[../patterns]] — UoW, Saga, Repository, Outbox
- [[../security]] — BeyondCorp server-side + ABAC + tenant isolation
- [[../tasks]] — Sprint 2, 5, 7 (cross-cutting)
- Correspondente web: [[../../web/requirements/_moc]] (shared infra)
- Correspondente mobile: a definir

## Links
- [[../../../_moc]]
- [[../../../CLAUDE]]
