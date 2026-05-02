---
title: Identity — Backend Requirements (Go)
type: requirement
tags: [master-sindico, backend, go, identity, oidc, zitadel, abac, session, feature-req]
project: master-sindico
stack: backend
bounded_context: identity
milestone: m1
status: delivered
created: 2026-04-24
---

# Identity — Backend Requirements (Go)

## Escopo do bounded context

Módulo Go `internal/modules/identity/` responsável por:

- Autenticação via **Zitadel OIDC + PKCE** (Authorization Code Flow) — sem gestão própria de senha.
- **Users mirror pattern** (ADR-0036): UPSERT local no primeiro login via introspection.
- Gestão de `sessions` com `device_fingerprint` e política **1-device**.
- Middleware `Authenticate` (validação bearer/cookie + introspection + cache Redis).
- **ABAC engine** canônico: matriz `(role, plan_tier, action, resource) → allow/deny`, ordem determinística, cache Redis.
- `RequirePermission(action, resource)` factory para handlers downstream.
- Multi-context memberships (mesmo user em vários tenants).
- LGPD: consentimento versionado, exclusão async (AccountDeletionSaga + keyed HMAC).
- Step-up MFA em endpoints auth-críticos (IDN-036).

**Não é responsabilidade**: billing/trial (usa `ITrialChecker` de billing), profile por persona (institutional/commercial), storage de foto (usa `IStorageProvider`).

## Status atual (vault canônico)

- **Sprints 2 e 5 (legado)** implementaram base: Zitadel OIDC, users mirror, sessions, ABAC matrix, middleware `Authenticate`.
- **A-###** fechados: A-001..A-006 (OIDC + PKCE), A-DC-PEN-003 (session regeneration), A-DC-PEN-007 (admin MFA M1), A-DC-PEN-010 + A-DC-PEN-012 (DTO estrito PATCH).
- **A-023 aberto**: `IUserModerator` para content/commercial aplicarem ban/suspension sem cross-BC import (DT-010 parcial).
- **DT-010 aberta**: `IAuditLogger` ainda acoplado em identity (cross-BC shim); precisa extrair pra `internal/shared/audit`.
- **D-###**: ADR-0026 (admin MFA M1), ADR-0028 (LGPD keyed HMAC), ADR-0030 (saga orchestration default), ADR-0036 (identity-mirror-pattern).

## Agregados (domain)

- [[../../../01-domain/aggregates/User]] — mirror do Zitadel; `zitadel_id` UNIQUE; `role` imutável (IDN-029); status lifecycle.
- [[../../../01-domain/aggregates/Session]] — 1-device enforcement, device_fingerprint, TTL access 1h / refresh 30d.
- [[../../../01-domain/aggregates/Membership]] — compartilhado com institutional; identity valida scope.
- [[../../../01-domain/aggregates/AbacDecision]] — VO imutável retornado pelo engine.
- `MarketingAgencyToken` — invite token com scopes limitados (IDN-025).
- `UserConsent`, `UserKnownIP`, `IPRiskEvent`, `LGPDPseudonymKey` — LGPD/anti-fraude.

## Entidades e Value Objects

| Tipo | Elemento | Invariante chave |
|---|---|---|
| Entity | `User` | `zitadel_id` UNIQUE; `role` imutável; `documento` (CPF/CNPJ) imutável; `status` state machine |
| Entity | `Session` | `zitadel_session_id` UNIQUE (nunca re-usado); 1 ativa por user default |
| VO | `Email` | regex + MX opcional; case-insensitive |
| VO | `Role` | enum `{syndic, resident, enterprise, marketing, local_company, admin}` |
| VO | `RoleType` | sub-contexto operacional por role (IDN-015) |
| VO | `DeviceFingerprint` | `hash(ip || user_agent || canvas_hash)` |
| VO | `StepUpToken` | JWT curto 5min, `amr=["mfa"]`, `acr=2` |
| VO | `PseudonymKey` | keyed HMAC rotating (ADR-0028), rotacionada 1/jan |

## Invariantes (INV-IDN-###)

1. **INV-IDN-001**: `users.zitadel_id` UNIQUE (mirror pattern).
2. **INV-IDN-002**: `users.role` imutável pós-criação (IDN-029).
3. **INV-IDN-003**: `users.documento` imutável pós-primeira definição.
4. **INV-IDN-004**: 1 sessão ativa por user (default) — `revoked_at IS NULL` UNIQUE por `user_id`.
5. **INV-IDN-005**: `sessions.zitadel_session_id` nunca re-usado (nova session = INSERT).
6. **INV-IDN-006** (INV-106 global): session regenerada em toda transição de auth-state.
7. **INV-IDN-007** (INV-107 global): DTO estrito em PATCH; `role` nunca em body.
8. **INV-IDN-008** (INV-101 global): ownership check em endpoints `/me/*/:id` → 404 cross-user.
9. **INV-IDN-009** (INV-109 global): keyed HMAC rotating para pseudonimização LGPD.
10. **INV-IDN-010** (INV-110 global): step-up MFA denylist `ms:stepup-denylist:<jti>` burn-after-use.
11. **INV-IDN-011**: Máx 1 principal + 2 auxiliares + 3 assistentes por condomínio (membership cap).

Referência: [[../../../01-domain/invariants]].

## Eventos de domínio

Publica via `IEventBus` (XD-001):

- `identity.user.created` → consumido por billing (auto-start trial via `ITrialChecker`).
- `identity.user.role_changed` → invalida ABAC cache + emite `billing.upgrade_required` se aplicável (BIL-004).
- `identity.session.created` → audit log.
- `identity.session.revoked` → fecha WebSockets (IDN-035); invalida cache.
- `identity.session.regenerated` → audit (INV-106).
- `identity.device.switched` → push pro device antigo + audit.
- `identity.user.banned` / `identity.user.unbanned` → content/commercial re-avaliam.
- `identity.deletion.requested` → AccountDeletionSaga inicia.
- `identity.deletion.completed` → lgpd_logs final.
- `identity.mfa.enrolled`, `identity.step_up.emitted|consumed|rejected`.

Consome:

- `billing.subscription.changed` → invalida ABAC cache `ms:abac:{user_id}:*`.

## Requirements funcionais (FR-BE-IDN-###)

- **FR-BE-IDN-001** (IDN-001) — *Quando* `POST /signup` chega com `role+plan_slug`, *o backend deve* validar enum `role ∈ {syndic, resident, enterprise, local_company}`, criar User no Zitadel via admin API, disparar email verificação. `marketing`/`admin` → 403.
- **FR-BE-IDN-002** (IDN-002) — *Enquanto* `email_verified=false`, *o middleware Authenticate deve* responder `401 email_not_verified` antes do UPSERT local.
- **FR-BE-IDN-003** (IDN-003) — *Quando* callback OIDC chega com `code`, *o backend deve* trocar por tokens via `oauth2.Exchange`, UPSERT `users` (IDN-012), criar `sessions` com device_fingerprint, setar cookie `access_token` httpOnly+Secure+SameSite=Lax+`__Host-` quando possível.
- **FR-BE-IDN-004** (IDN-004) — *Quando* introspection retorna `role=""`, *o backend deve* responder `403 NO_ROLE_ASSIGNED` antes do UPSERT (guard clause T15).
- **FR-BE-IDN-005** (IDN-005) — *Quando* `POST /auth/logout`, *o backend deve* (a) Zitadel revoke, (b) `UPDATE sessions SET revoked_at=now()`, (c) clear cookies, (d) Redis DEL `ms:auth:{zitadelUserID}` + `ms:abac:{user_id}:*`.
- **FR-BE-IDN-006** (IDN-007 + IDN-036) — *Quando* handler protegido por endpoint auth-crítico, *o middleware deve* validar `X-Step-Up-Token` (JWT 5min, claim `amr=["mfa"]`, `acr=2`), adicionar `jti` ao denylist Redis burn-after-use; ausente → `401 step_up_required`.
- **FR-BE-IDN-007** (IDN-009) — *Quando* login OK, *o backend deve* criar `sessions` com PK UUIDv7, index `(user_id, revoked_at)`, UNIQUE `zitadel_session_id`.
- **FR-BE-IDN-008** (IDN-010) — *Quando* login em device B com session ativa em A, *o backend deve* (UoW única): (a) `UPDATE sessions SET revoked_at=now() WHERE user_id=$1 AND revoked_at IS NULL`, (b) INSERT nova session, (c) publicar `identity.session.invalidated` → FCM + WebSocket close 1008 device A.
- **FR-BE-IDN-009** (IDN-012) — *Quando* introspection OK e user não existe local, *o backend deve* UPSERT transacional `users {zitadel_id, email, name, phone, role, status='active', created_at}` com ON CONFLICT preservando imutáveis.
- **FR-BE-IDN-010** (IDN-013 + IDN-014) — *Quando* user autentica, *o backend deve* carregar todos memberships ativos (WHERE active=true) e permitir `X-Tenant-Id` header selecionar contexto; `GET /me/contexts` lista.
- **FR-BE-IDN-011** (IDN-017 + IDN-020) — *Quando* user tem `status='incomplete'`, *o middleware deve* permitir apenas rotas `/onboarding/*` + `/me`; demais → `403 onboarding_incomplete`.
- **FR-BE-IDN-012** (IDN-018) — *Quando* user auto-salva onboarding, *o backend deve* escrever em Redis `onboarding:{user_id}:{step}` TTL 30d; no POST final migra para tabelas de profile (commits via UoW).
- **FR-BE-IDN-013** (IDN-022) — *Quando* `DELETE /api/v1/me` chega com step-up MFA, *o backend deve* iniciar `AccountDeletionSaga`: (1) `users._pending_hard_delete=true` + `deletion_scheduled_at=NOW()+15d`, (2) Stripe subscription cancel, (3) hooks verificam soft-flag em todo webhook, (4) D+15 pseudonimização via `Pseudonymize(user_id, purpose, NOW())` keyed HMAC, (5) hard-delete rows pessoais, pseudonimiza FKs.
- **FR-BE-IDN-014** (IDN-023) — *Quando* `GET /me/export`, *o backend deve* gerar ZIP (perfil + memberships + timeline_entries visible + votes + proposals + certificates) async se > 50MB, retornar presigned URL TTL 48h; rate 1/24h.
- **FR-BE-IDN-015** (IDN-024 + XD-025) — *Quando* user aceita termos, *o backend deve* persistir `user_term_acceptances {user_id, terms_version, accepted_at, ip, user_agent, term_type}` append-only; nova versão → modal bloqueante próximo login.
- **FR-BE-IDN-016** (IDN-025) — *Quando* empresa Plus/Pro gera invite agência, *o backend deve* emitir token `marketing_agency_tokens {enterprise_id, email, scopes:["videos:upload","videos:edit","analytics:read"], expires_at=NOW()+90d}`.
- **FR-BE-IDN-017** (IDN-029) — *Quando* `PATCH /users/{id}` tenta alterar `role|documento`, *o backend deve* rejeitar `422 validation_error` via application-level guard (não depende só de DB CHECK).
- **FR-BE-IDN-018** (IDN-030) — *Enquanto* `users.banned=true` OR `ban_expires_at > now()`, *o middleware deve* responder `403 account_banned` mesmo se Zitadel retorna active.
- **FR-BE-IDN-019** (IDN-037) — *Quando* handler `DELETE /api/v1/me/devices/:id`, *o helper `OwnershipCheck` deve* executar `WHERE id=:id AND user_id=:ctx_user_id` antes de mutação; mismatch → `404 not_found` (nunca vaza existência).
- **FR-BE-IDN-020** (IDN-038) — *Quando* user completa login/MFA/step-up/role-change/logout+re-login, *o backend deve* chamar `SessionStore.Regenerate()` para emitir novo `session_id` opaco + invalidar anterior.
- **FR-BE-IDN-021** (IDN-039) — *Quando* handler recebe PATCH mutating, *o DTO deve* ter `additionalProperties: false` via schema OpenAPI + struct Go tagged; body com `role|is_admin|tenant_id|banned` → `422 validation_error`.
- **FR-BE-IDN-022** (XD-004 + XD-005) — *Quando* request chega handler protegido, *o middleware `RequirePermission(action, resource)` deve* consultar ABAC engine na ordem: admin_bypass → role_check → action_check → tenant_check → scope_check; cache Redis `ms:abac:{user_id}:{tenant_id}` TTL 5min.
- **FR-BE-IDN-023** (XD-006) — *Quando* `billing.subscription.changed` chega via `IEventBus`, *o listener deve* emitir `DEL ms:abac:{user_id}:*` (pattern).
- **FR-BE-IDN-024** (XD-007 + BIL-043) — *Quando* ABAC avalia `(role, tier, action)`, *o engine deve* retornar `ABACDecision {allowed, role, plan_tier, reason ∈ {admin_bypass, matrix_deny, plan_missing, quota_exceeded, tenant_out_of_scope, experimental_hidden, allowed}}`.
- **FR-BE-IDN-025** (IDN-035) — *Quando* client abre `WebSocket /ws/assemblies/{id}`, *o backend deve* validar cookie session ativa, associar WS ao `session_id`, fechar code 1008 se session revogada.
- **FR-BE-IDN-026** (IDN-031) — *Quando* cron 3h SP roda, *o job SyncZitadel deve* comparar Zitadel vs local: deleted → `deleted_at`; role change → alerta se imutável violado (INV-IDN-002).
- **FR-BE-IDN-027** (IDN-032) — *Quando* síndico principal emite invite co-admin, *o backend deve* gerar token TTL 7d, criar membership `role=syndic, role_type ∈ {auxiliary, assistant}` no aceite; revogação imediata → session revoked.
- **FR-BE-IDN-028** — `IUserModerator` interface (A-023) exposta em `internal/shared/types`: `BanUser(ctx, user_id, reason, until)`, `SuspendUser(ctx, user_id, duration)`; content/commercial chamam para harassment + reputation suspension.

## Endpoints REST expostos

| Método | Path | Auth | Descrição | Status |
|---|---|---|---|---|
| POST | /signup | público | Signup inteligente parametrizado | ✅ |
| POST | /auth/login | público | Inicia OIDC + PKCE | ✅ |
| GET | /auth/callback | público | Callback OIDC | ✅ |
| POST | /auth/logout | público | Logout + revoke | ✅ |
| POST | /auth/resend-verification | público | Reenvio email verify (rate 1/2min) | ✅ |
| POST | /auth/password-reset-request | público | Delega Zitadel (rate 3/h) | ✅ |
| POST | /auth/step-up | Bearer | Emite step-up token 5min (rate 5rpm) | ✅ |
| GET | /api/v1/me | Bearer | Perfil próprio | ✅ |
| PATCH | /api/v1/me | Bearer + step-up se email/phone | Update profile strict DTO | ✅ |
| DELETE | /api/v1/me | Bearer + step-up | AccountDeletionSaga | ✅ |
| GET | /api/v1/me/export | Bearer + step-up | LGPD export async | 🟡 |
| GET | /api/v1/me/contexts | Bearer | Lista memberships ativos | ✅ |
| GET | /api/v1/me/devices | Bearer | Sessions ativas | ✅ |
| DELETE | /api/v1/me/devices/:id | Bearer + step-up + OwnershipCheck | Revoga device | ✅ |
| DELETE | /api/v1/me/sessions/:id | Bearer + OwnershipCheck | Revoga session específica | ✅ |
| POST | /api/v1/me/consents | Bearer | Aceita termo versionado | ✅ |
| DELETE | /api/v1/me/consents/:type | Bearer + OwnershipCheck | Retira consentimento | 🟡 |
| GET | /api/v1/memberships/:id | Bearer + ABAC | Detalhe | ✅ |
| PATCH | /api/v1/memberships/:id | Bearer + ABAC + strict DTO | Update limitado (INS-007) | ✅ |
| POST | /api/v1/condominiums/:id/co-admin-invite | Bearer + ABAC(principal) | Invite IDN-032 | 🟡 |
| GET | /api/v1/auth/zitadel/jwks | público | JWKS público cacheado | ✅ |

## Integração com outros BCs

**Consome** (via `internal/shared/types`):

- `ITrialChecker` (de billing) — valida trial ativo no onboarding completion.
- `ICondominiumIDsProvider` (de institutional) — resolve scope para ABAC tenant_check.
- `IEventBus` (de cross-domain).
- `IAuditLogger` (DT-010 — atualmente shim local).
- `IEmailProvider` (XD-015).

**Expõe** (via shared interfaces):

- `IUserModerator` (A-023) — BanUser/SuspendUser.
- `IABACEngine` — `Evaluate(ctx, user_id, action, resource) ABACDecision`.
- `ISessionStore` — Get/Create/Revoke/Regenerate.
- `IIdentityMirror` — UpsertFromZitadel.

**Publica eventos**: `identity.user.*`, `identity.session.*`, `identity.device.*`, `identity.mfa.*`, `identity.deletion.*`, `identity.step_up.*`.

**Consome eventos**: `billing.subscription.changed` (cache invalidation).

## Persistência

**Tabelas principais** (sqlc + Postgres):

- `users` — PK UUIDv7, `zitadel_id` UNIQUE, `documento` UNIQUE (CPF/CNPJ), `role` enum, `status` enum, `banned_until TIMESTAMPTZ NULL`, `_pending_hard_delete BOOL`, `deletion_scheduled_at`, `visibility_ready BOOL`.
- `sessions` — PK UUIDv7, FK `user_id`, `zitadel_session_id` UNIQUE, `device_fingerprint`, `ip_address`, `user_agent`, `created_at`, `revoked_at`.
- `memberships` — UNIQUE `(user_id, tenant_id, role, active) WHERE active=true`; `role_type` enum.
- `user_term_acceptances` — append-only; retenção 5 anos.
- `marketing_agency_tokens` — `expires_at`, `accepted_at`, `revoked_at`.
- `user_consents`, `user_known_ips`, `ip_risk_events`.
- `lgpd_pseudonym_keys` — rotacionadas 1/jan.
- `stepup_denylist` — TTL Redis primário; DB backup para audit.

**Migrations** (goose embed): `00001..00099` identity.

**RLS default-on** (ADR-0034): `sessions`, `memberships`, `user_consents` com `policy USING (user_id = current_setting('app.current_user_id')::uuid)`. `users` **sem RLS** (cross-tenant lookup permitido a admin).

## Providers externos

- **Zitadel** — OIDC IdP; admin API (signup, reset, MFA enroll); JWKS para introspection.
- **Resend/SES** (`IEmailProvider`) — verificação email, reset, step-up (fallback).
- **Twilio** (`ISMSProvider`) — IDN-028 anomaly confirm.
- **Cloudflare Turnstile ou reCAPTCHA v3** (a definir ADR M2) — IDN-027.
- **FCM** (`IPushProvider`) — IDN-010 device switched notification.

**Sagas de compensação**:

- `AccountDeletionSaga` (ADR-0030) — 7 steps com compensações: Stripe cancel idempotente, pseudonym key rotation, hard-delete via UoW transacional.
- `SignupOidcSaga` (futuro) — rollback Zitadel user se DB falha.

## Testes

**Coverage target**:

- Domain: **≥90%** (funções puras).
- Application: **≥80%** (use cases, sagas).
- Infrastructure: **≥60%** (repositórios via testcontainers).

**PBT** (`gopter`/`rapid`):

- ABAC matrix: `∀ (role, tier, action) → determinístico`.
- Ownership: `∀ user_A, resource_B onde owner_id != A → DELETE /me/<res>/:id == 404`.
- Session regeneration: `∀ transição → session_id novo != anterior ∧ anterior invalidado`.
- Mass assignment: `fuzz body{role: "admin"} → 422`.

**Integration** (testcontainers): Postgres + Redis + Zitadel mock; saga AccountDeletion E2E.

## Segurança específica do BC

**LGPD** (INV-LGPD-*):

- Retenção `user_term_acceptances`: 5 anos.
- Pseudonimização via keyed HMAC rotating (ADR-0028); keys rotacionadas 1/jan.
- Soft-flag `_pending_hard_delete` consultado em TODO webhook handler (Stripe, Mux, Livekit) — evita race.
- Audit trail via `IAuditLogger`: `identity.deletion.requested`, `identity.deletion.completed` com `user_id_hash = Pseudonymize(user_id, "audit_anon", NOW())`.

**ABAC actions canônicos**:

- `identity.session.revoke.self`, `identity.session.revoke.other` (admin).
- `identity.user.ban`, `identity.user.unban` (admin).
- `identity.membership.create.syndic_co_admin` (principal only).
- `identity.consent.read`, `identity.consent.revoke.self`.

**Rate limits**:

- `/auth/login` — 10 rpm por IP.
- `/auth/resend-verification` — 1 req/2min por email.
- `/auth/password-reset-request` — 3 req/h por email.
- `/auth/step-up` — 5 rpm por user_id.
- `/signup` — 3 req/h por IP (IDN-026 integrado).
- `/api/v1/me/export` — 1 req/24h por user.

**Headers**: cookies com `__Host-` prefix quando possível (sem `Domain` attribute), `Secure`, `HttpOnly`, `SameSite=Lax`.

## Decisões e dívidas

**D-###**:

- D-026 (Resend vs AWS SES) — aberta.
- ADR-0026 (admin MFA reclassificado M2→M1) — fechado.
- ADR-0028 (LGPD keyed HMAC rotating) — fechado.
- ADR-0030 (saga orchestration default) — fechado.
- ADR-0036 (identity-mirror-pattern) — fechado.

**DT-###**:

- **DT-010 aberta**: `IAuditLogger` ainda shim local em identity; extrair para `internal/shared/audit` e promover interface.
- **A-023 aberto**: `IUserModerator` interface exposta via `internal/shared/types` para content/commercial sem cross-BC import.

## Gaps e bloqueadores Sprint 10 (M1)

1. **LGPD export async** (FR-BE-IDN-014) — 🟡 endpoint existe mas sem job async; bloqueia auditoria pré-M1.
2. **`DELETE /me/consents/:type`** (FR-BE-IDN-015) — 🟡 ownership check presente; falta purge em downstream caches.
3. **IDN-028 anomaly detection** — ⏳ C (M3); não bloqueia M1.
4. **DT-010 `IAuditLogger` extraction** — desacoplar shim identity → `internal/shared/audit`.
5. **IDN-032 co-admin invite** — 🟡 token gera mas revocation path não invalida WebSockets ativos imediatamente.
6. **IDN-026 IP risk + captcha rollout** — S (M2).

## Ligações

- [[../../../04-requirements/functional/identity]] — requirement canônico global
- [[../../../01-domain/aggregates/User]]
- [[../../../01-domain/aggregates/Session]]
- [[../../../01-domain/aggregates/AbacDecision]]
- [[../../../01-domain/aggregates/_moc]]
- [[../../../02-architecture/_moc]]
- [[../../../11-audit/AUDIT]]
- [[../architecture]] — Clean Arch backend
- [[../security]] — BeyondCorp + ABAC + tenant isolation
- [[../tasks]] — Sprint 2, 5 (legado) + 10 (M1 lockdown)
- Correspondente web: [[../../web/requirements/auth]]
- Correspondente mobile: a definir

## Links
- [[../../../_moc]]
- [[../../../CLAUDE]]
