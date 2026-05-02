---
title: Functional Requirements — Identity BC
type: requirement
tags:
  - requirements
  - functional
  - identity
  - auth
  - oidc
  - abac
  - zitadel
  - master-sindico
  - v2
  - ears
source: >-
  legacy specs/requirements/identity + identity-mirror-pattern +
  contextos/ONBOARDING_CADASTRO_VS_PERFIL + _chaos/requirements(6).md
  (2026-04-13)
created: 2026-04-23T00:00:00.000Z
updated: '2026-04-25'
fase_c_absorvido: true
---

> **Banner Fase C (2026-04-25)**: IDN-026 reescrito por D-135 (rejeitado bloqueio por IP; device fingerprint progressivo adotado). Demais Reqs: cobertura confirmada vs `_chaos/requirements(6).md` Req 1-3, 76, 102.

# Functional Requirements — Identity BC

Requisitos funcionais do **bounded context `identity`**. Responsabilidade: usuários, sessões, autenticação OIDC+PKCE (Zitadel), 1-device policy, multi-context membership, LGPD (exclusão, portabilidade, consentimento).

Formato **EARS**: "Quando/Enquanto `<trigger/state>`, o sistema deve `<behavior>`".

Prioridade: M (Must M1), S (Should M2), C (Could M3+), W (Won't).

> Destilado de `../../90-ingested/from-vault-50-projects-master-sindico/specs/requirements/identity.md` + `identity-mirror-pattern.md` + `04-requirements/global.md` + `contextos/ONBOARDING_CADASTRO_VS_PERFIL.md`.

---

## IDN-001 — Signup Inteligente (parametrizado)

**Prioridade**: M  
**EARS**: *Quando* usuário acessa URL parametrizada `mastersindico.com/cadastro?role=<role>&plan=<plan_slug>`, *o sistema deve* interpretar `role ∈ {syndic, resident, enterprise, local_company}` e `plan_slug` válido, criar registro inicial no DB com `role` travado (imutável pós-registro — ver IDN-029), disparar email de verificação Zitadel. *Quando* URL falta `role` → redireciona pra página de seleção de persona.

**Dependências**: GR-001, Zitadel tenant provisionado.

**Critério de aceite**:
- Personas auto-registráveis: `syndic`, `resident`, `enterprise`, `local_company`.
- `marketing` NÃO auto-registra (só via invite token — IDN-025).
- `admin` NÃO auto-registra (provisionado manualmente no Zitadel).
- URL sem `role` → tela de seleção renderiza.
- Teste: `?role=syndic&plan=syndic_n2` cria user com `role=syndic` travado + subscription trial `syndic_n2`.

**Fonte**: legado `novo escopo-7.md` §2.5, `identity.md` Req 3.

---

## IDN-002 — Verificação de email obrigatória

**Prioridade**: M  
**EARS**: *Enquanto* usuário não confirmou email via link Zitadel, *o sistema deve* bloquear login (`401 email_not_verified`). *Quando* click no link → Zitadel atualiza claim `email_verified=true` → próximo login permitido.

**Critério de aceite**:
- Token de verificação gerado por Zitadel (não pelo backend).
- Link válido 24h (configuração Zitadel).
- Reenvio: `POST /auth/resend-verification` com rate limit 1 req/2min.

**Fonte**: legado `identity.md` Req 3.

---

## IDN-003 — Login OIDC com PKCE

**Prioridade**: M  
**EARS**: *Quando* usuário clica "Login", *o sistema deve* iniciar Authorization Code Flow + PKCE via Zitadel: gera `code_verifier`, armazena em cookie `gorilla/securecookie`, redireciona pra Zitadel com `code_challenge`. *Quando* callback chega com `code`, *o sistema deve* trocar por access/refresh token, criar sessão local (IDN-009), setar cookie `access_token` httpOnly.

**Dependências**: GR-001, GR-004.

**Critério de aceite**:
- Cookie: `Secure`, `SameSite=Lax`, domain `.mastersindico.com.br`.
- Fallback: `Authorization: Bearer <token>` pro mobile.
- Refresh gerenciado por Zitadel; backend só valida via introspection.
- Cache Redis `ms:auth:{zitadelUserID}` TTL 5 min.

**Fonte**: legado `identity.md` Req 1, T9/T11/T13/T14.

---

## IDN-004 — Guard clause: tokens sem role

**Prioridade**: M  
**EARS**: *Quando* introspection Zitadel retorna `role = ""` ou ausente, *o sistema deve* responder `403 NO_ROLE_ASSIGNED` **antes** do UPSERT em `users` local.

**Critério de aceite**:
- T15: token sem role não cria user local.
- Teste: usuário Zitadel com role removido → próximo login → 403.

**Fonte**: legado `identity.md` Req 1, T15.

---

## IDN-005 — Logout

**Prioridade**: M  
**EARS**: *Quando* usuário emite `POST /auth/logout`, *o sistema deve* (1) revogar access token em Zitadel, (2) remover `sessions` local (`UPDATE sessions SET revoked_at=now()`), (3) limpar cookies (`access_token`, `code_verifier`), (4) invalidar cache Redis.

**Critério de aceite**:
- Rota pública (fora de `/api/v1`, sem ABAC).
- Idempotente: logout de sessão já revogada → 204.
- Audit trail: `identity.session.revoked`.

**Fonte**: legado `identity.md` Req 1.

---

## IDN-006 — Password reset (via Zitadel)

**Prioridade**: M  
**EARS**: *Quando* usuário emite `POST /auth/password-reset-request`, *o sistema deve* delegar pro Zitadel (API admin ou flow built-in) que envia email com link de reset. *Quando* user completa reset em Zitadel → próximo login novo access token.

**Critério de aceite**:
- Backend não gerencia hash de senha (delegado 100% a Zitadel — Argon2 interno).
- Rate limit: 3 requests/hora por email.
- Sessões ativas do user: auto-revogadas pós-reset (GR-004 + audit).

**Fonte**: legado `identity.md` Req 3.

---

## IDN-007 — MFA (Multi-Factor Authentication)

**Prioridade**: **M** (admin M1 — reclassificado — + empresa `plan_tier ∈ {pro}` M1 + step-up críticos M1) / S (síndico `plan_tier=pro`, morador opt-in M3)  

**EARS**: *Quando* usuário com `role = admin` OU `role_type ∈ {principal, administrator}` de empresa plan_tier ∈ {pro, platinum, diamante} faz login, *o sistema deve* exigir MFA TOTP antes de emitir session. *Quando* usuário tenta acessar endpoint auth-crítico (ver IDN-036), *o sistema deve* exigir `X-Step-Up-Token` fresh (`≤ 5min`). *Quando* MFA setup → Zitadel armazena backup codes hasheados (10 códigos one-time-use).

**Critério de aceite**:
- **Admin**: MFA obrigatório **desde M1** (reclassificado M2→M1 em [[../../02-architecture/adr/0026-admin-mfa-m1]], cobre [[../../11-audit/audit-log/2026-04-23-pentest-specs#A-DC-PEN-007]]).
- **Empresa Pro/Platinum/Diamante**: MFA obrigatório M1 para role_type administrator/principal.
- **Síndico N3 + Morador**: MFA opcional M1, recomendado M2, passkey default M3.
- Backup codes: 10 códigos one-time-use.
- Enforce Zitadel: project config `mfa_type = required` para subset acima.
- Teste: MFA ativo → login sem código → 403 `mfa_required`.

**Dependências**: GR-001, IDN-036, [[../../02-architecture/adr/0026-admin-mfa-m1]].

**Fonte**: legado `identity.md` Req 1 "SHOULD", [[../../00-product/personas]] Admin; reclassificado pós-[[../../11-audit/audit-log/2026-04-23-pentest-specs#A-DC-PEN-007]].

---

## IDN-036 — Step-up MFA em endpoints auth-críticos

**Prioridade**: M (M1)  
**EARS**: *Quando* request chega em endpoint auth-crítico, *o sistema deve* validar header `X-Step-Up-Token` (JWT curto 5min, claim `amr=["mfa"]` + `acr=2` OIDC). *Quando* token ausente/expirado/revogado → responde `401 step_up_required` com campo `step_up_url = /auth/step-up`. *Quando* válido → prossegue handler e adiciona `jti` ao denylist Redis (burn after use).

**Endpoints auth-críticos** (lista canônica):
- `DELETE /api/v1/me` (LGPD Art. 18 VI)
- `DELETE /api/v1/me/devices/:id` (cobre [[../../11-audit/dual-check-log/2026-04-23-security-gaps#A-DC-SEC-001]])
- `GET /api/v1/me/export` (cobre oracle LGPD)
- `POST /api/v1/assemblies/:id/minutes/publish` (Lei 4.591/64)
- `POST /api/v1/contracts/:id/approve` quando `value ≥ R$ 10.000`
- `POST /api/v1/condominiums/:id/transfer-mandate`
- `PATCH /api/v1/me` quando altera `email` ou `phone`
- `POST /admin/v1/*` (qualquer rota admin)

**Critério de aceite**:
- Denylist `ms:stepup-denylist:<jti>` TTL = token_exp - now.
- Rate limit `/auth/step-up` 5 rpm por user_id.
- Audit trail: `step_up.emitted`, `step_up.consumed`, `step_up.rejected`.
- INV-110 é a invariante.

**Dependências**: GR-001, IDN-007, [[../../02-architecture/adr/0026-admin-mfa-m1]], INV-110.

**Fonte**: [[../../11-audit/audit-log/2026-04-23-pentest-specs#A-DC-PEN-007]] + [[../../11-audit/dual-check-log/2026-04-23-security-gaps#Gap-SEC-002]].

---

## IDN-037 — Ownership validation em endpoints user-scoped (IDOR hardening)

**Prioridade**: M (M1)  
**EARS**: *Quando* handler recebe `DELETE /api/v1/me/devices/:id` ou similar endpoint user-scoped de Delete/Update, *o sistema deve* buscar o recurso com query tenant-aware `WHERE id = :id AND user_id = :ctx_user_id` **antes** de qualquer ação de mutação. *Quando* recurso não existe OU user_id mismatch → responde `404 not_found` (não 403 — nunca vaza existência cross-user).

**Endpoints cobertos** (não-exaustivo, padrão aplicável):
- `DELETE /api/v1/me/devices/:id`
- `DELETE /api/v1/me/sessions/:id`
- `DELETE /api/v1/me/consents/:type`
- `PATCH /api/v1/me/proposals/:id` (se existir self-service)
- Qualquer endpoint com path `/me/*/:id` em que `:id` é recurso user-scoped.

**Critério de aceite**:
- Middleware helper `OwnershipCheck(resource_type, id_param)` injetável.
- PBT: `∀ user_A, resource_B onde resource_B.owner_id != user_A → DELETE /me/<res>/:id == 404`.
- Audit test em cross-user test suite ≥ 100 combinações.
- INV-101 é a invariante.

**Dependências**: GR-002, GR-003, INV-101.

**Fonte**: [[../../11-audit/dual-check-log/2026-04-23-security-gaps#A-DC-SEC-001]].

---

## IDN-038 — Session regeneration em transições de auth-state

**Prioridade**: M (M1)  
**EARS**: *Quando* user completa login bem-sucedido, MFA completion, step-up MFA, role change ou logout → re-login no mesmo device, *o sistema deve* gerar **novo** `session_id` opaco (via `SessionStore.Regenerate()`) e **invalidar** o anterior incondicionalmente. *Quando* cookie não-existente ou expirado → emite novo sem revogação prévia.

**Critério de aceite**:
- Cookies com flag `__Host-` prefix quando possível (sem `Domain` attribute — limita escopo).
- Teste automatizado de session fixation: planta cookie pré-login → loga → confirma que cookie antigo foi invalidado.
- Audit trail: `identity.session.regenerated` em cada transição.
- INV-106 é a invariante.

**Dependências**: GR-001, GR-004, IDN-009, IDN-010, INV-106.

**Fonte**: [[../../11-audit/audit-log/2026-04-23-pentest-specs#A-DC-PEN-003]].

---

## IDN-039 — DTO estrito em PATCH profile (mass assignment hardening)

**Prioridade**: M (M1)  
**EARS**: *Quando* usuário emite `PATCH /api/v1/me` ou qualquer endpoint de atualização de profile, *o sistema deve* aceitar **apenas** fields do DTO `UpdateProfileRequest { name?, avatar_url?, bio?, phone?, email? }`. Campos `role`, `is_admin`, `tenant_id`, `user_id`, `documento`, `banned`, `status`, `plan_tier` → **rejeitados com `422 validation_error`** (DTO nem aceita o field; strict schema). Role change só via endpoint dedicado com MFA step-up.

**Endpoints cobertos**:
- `PATCH /api/v1/me` (profile self)
- `PATCH /api/v1/me/profile` (se distinto)
- `PATCH /api/v1/memberships/:id` (cobre [[../../11-audit/audit-log/2026-04-23-pentest-specs#A-DC-PEN-012]])
- Qualquer PATCH mutating em resources user/tenant-scoped.

**Critério de aceite**:
- OpenAPI DTOs tipados strict (`additionalProperties: false`).
- Handler Go usa struct tagged + sem decoder que "injeta" unknown fields.
- CI grep bloqueia handlers que aceitem `role` em body.
- PBT: fuzz body com `{role: "admin"}` → sempre 422.
- INV-107 é a invariante.

**Dependências**: IDN-029 (imutabilidade de role), INV-107, [[../../02-architecture/adr/0026-admin-mfa-m1]].

**Fonte**: [[../../11-audit/audit-log/2026-04-23-pentest-specs#A-DC-PEN-010]] + [[../../11-audit/audit-log/2026-04-23-pentest-specs#A-DC-PEN-012]].

---

## IDN-008 — Social login (Google/Apple)

**Prioridade**: C (M3+)  
**EARS**: *Quando* usuário clica "Entrar com Google" ou "Entrar com Apple", *o sistema deve* delegar pro Zitadel (providers habilitados via painel). *Quando* provider retorna → UPSERT em `users` com `zitadel_id` vinculado.

**Critério de aceite**:
- Só após M3 (sprint 5+ legado).
- Zitadel suporta nativo (só habilitar provider).

**Fonte**: legado `identity.md` Req 1 "SHOULD".

---

## IDN-009 — Sessions vinculadas a device fingerprint

**Prioridade**: M  
**EARS**: *Quando* login é bem-sucedido, *o sistema deve* criar registro em `sessions` com `{id, user_id, zitadel_session_id, device_fingerprint, ip_address, user_agent, created_at, revoked_at}`. `device_fingerprint = hash(ip || user_agent || canvas_hash)`.

**Critério de aceite**:
- PK UUIDv7.
- `zitadel_session_id` UNIQUE (nunca re-usado; nova session = INSERT, não UPDATE).
- Index em `(user_id, revoked_at)`.

**Fonte**: legado `identity.md` Req 1 notas.

---

## IDN-010 — 1-device policy enforcement

**Prioridade**: M  
**EARS**: *Quando* user com sessão ativa em device A faz login em device B (fingerprint diferente), *o sistema deve* (1) `UPDATE sessions SET revoked_at=now() WHERE user_id=$1 AND revoked_at IS NULL`, (2) criar nova session pra device B, (3) emitir evento `session.invalidated` pra notificar device A (WebSocket ou FCM), (4) audit trail `identity.device.switched`.

**Dependências**: GR-004.

**Critério de aceite**:
- Próxima request do device A → `401 SESSION_REVOKED`.
- Device A recebe push "Sua sessão foi encerrada (login em novo dispositivo)".
- Teste: duas requests simultâneas de device B → 1 session criada, 1 revogada.

**Fonte**: legado `identity.md` Req 1 rule 8, matriz funcional observação final.

---

## IDN-011 — Sessão expirada

**Prioridade**: M  
**EARS**: *Enquanto* session tem `revoked_at IS NOT NULL` ou `access_token expirado`, *o sistema deve* responder `401 session_expired` e redirecionar pra `/login` (frontend interpretação).

**Critério de aceite**:
- TTL: access token 1h (configurável Zitadel), refresh 30d.
- Middleware tenta refresh silencioso antes de expirar.

**Fonte**: legado `identity.md`, GR-001.

---

## IDN-012 — User mirror pattern (UPSERT no primeiro login)

**Prioridade**: M  
**EARS**: *Quando* introspection Zitadel retorna user novo (não existe em `users` local), *o sistema deve* UPSERT `users {zitadel_id, email, name, phone, role, status='active', created_at}`. *Quando* user já existe → UPDATE campos mutáveis (`email`, `name`, `phone` — `role` é imutável; ver IDN-029).

**Critério de aceite**:
- `users.zitadel_id` UNIQUE.
- UPSERT transacional (evita race em logins concorrentes).
- Campos imutáveis validados: `role`, `documento` (CPF/CNPJ).

**Fonte**: legado `identity-mirror-pattern.md`, `identity.md` Req 3.

---

## IDN-013 — Multi-context support (mesmo user, múltiplos memberships)

**Prioridade**: M  
**EARS**: *Quando* user autentica, *o sistema deve* permitir que o MESMO `user_id` tenha múltiplos memberships ativos em tenants distintos (ex: síndico no Condomínio X + morador no Y + representante da Empresa Z). *Quando* user opera, *o sistema deve* exigir seleção de contexto ativo (`X-Tenant-Id` header ou query param) ou inferir default (último usado).

**Critério de aceite**:
- Tabela `memberships {id, user_id, tenant_id, tenant_type, role, role_type, active, created_at}`.
- UNIQUE `(user_id, tenant_id, role, active)` — 1 membership ativo por tripla.
- ABAC consulta memberships ativos pra autorização.

**Fonte**: [[STEERING]] §2, legado `personas-and-plans.md`, [[../01-domain/context-map]] (a destilar).

---

## IDN-014 — Seletor de contexto (frontend)

**Prioridade**: M  
**EARS**: *Quando* user tem múltiplos memberships ativos, *o sistema deve* expor endpoint `GET /me/contexts` listando `{tenant_id, tenant_type, role, role_type, display_name}`. *Quando* user troca contexto, *o sistema deve* atualizar `X-Tenant-Id` em próximas requests e invalidar cache ABAC local.

**Critério de aceite**:
- Frontend dropdown/seletor no header.
- Persistência em cookie/localStorage.
- Troca invalida cache Redis auth (`ms:abac:{user_id}:{tenant_id}`).

**Fonte**: legado `institutional.md` Req 8.

---

## IDN-015 — Role types (sub-contextos operacionais)

**Prioridade**: M  
**EARS**: *Quando* membership é criado, *o sistema deve* aceitar `role_type` derivado:
- Síndico (`role=syndic`): `principal`, `auxiliary`, `assistant`.
- Morador (`role=resident`): `titular`, `dependent`.
- Empresa (`role=enterprise`): `administrator`, `operator`.
- Parceiro (`role=local_company`): `administrator`.
- Agência (`role=marketing`): `administrator`, `operator`.

**Dependências**: IDN-013.

**Critério de aceite**:
- `role_type` é lógica INTERNA (não existe no Zitadel — T8).
- ABAC usa `role_type` pra fine-grained (ex: `principal` cria comunicado, `auxiliary` não pode).

**Fonte**: legado `identity.md` Req 2, `personas-and-plans.md` T8.

---

## IDN-016 — Cadastro mínimo (etapa 1)

**Prioridade**: M  
**EARS**: *Quando* user submete formulário inicial de cadastro, *o sistema deve* aceitar campos mínimos `{name, email, password, role}`, criar User no Zitadel (via admin API ou flow) e disparar verificação de email. **Nenhum perfil é criado nesta etapa.**

**Critério de aceite**:
- Validação email: regex + DNS MX check opcional.
- Senha: políticas Zitadel (min 8 chars, 1 maiúscula, 1 número, 1 especial).
- Nome: 2+ chars, trim.

**Fonte**: legado `ONBOARDING_CADASTRO_VS_PERFIL.md`, [[registration-data]].

---

## IDN-017 — Cadastro obrigatório (etapa 2 — por persona)

**Prioridade**: M  
**EARS**: *Quando* user faz 1º login pós email-verified, *o sistema deve* exigir preenchimento de dados obrigatórios por persona antes de liberar app (ver [[registration-data]]). *Enquanto* dados obrigatórios incompletos, *o sistema deve* redirecionar pra `/onboarding/step/<n>` e retornar `403 onboarding_incomplete` em rotas protegidas.

**Critério de aceite**:
- Status: `users.status = 'incomplete'` até completude.
- Dados obrigatórios por persona em [[registration-data]].
- Documento (CPF/CNPJ) imutável pós-cadastro (IDN-029).

**Fonte**: legado `ONBOARDING_CADASTRO_VS_PERFIL.md`, `billing.md` Req 5, `novo escopo-7.md` §1.2.

---

## IDN-018 — Onboarding auto-save

**Prioridade**: M  
**EARS**: *Quando* user preenche parcialmente onboarding, *o sistema deve* auto-salvar em Redis `onboarding:{user_id}:{step}` com TTL 30 dias. *Quando* user retorna → carrega progresso e permite continuar onde parou.

**Critério de aceite**:
- TTL Redis 30 dias.
- Auto-save a cada blur de campo (frontend debounce 500ms).
- Validação frontend (Zod) + backend no POST final.

**Fonte**: legado `billing.md` Req 5.

---

## IDN-019 — Onboarding: telas por persona

**Prioridade**: M  
**EARS**: *Quando* onboarding inicia, *o sistema deve* exibir telas por persona:
- Morador (4 telas): Buscar condo por ID → Registrar unidade → Termo LGPD → Foto.
- Síndico (6 telas): Dados pessoais → Tipo síndico → Governance markers → Criar condo → Mandato → Empresa síndica (opcional).
- Empresa (7 telas): Dados → CNPJ → Categorias → Compliance markers → Logo/fotos → Termos → Dashboard.
- Parceiro (4 telas): Dados estabelecimento → Endereço/CEP → Logo/descrição → Dashboard.

**Critério de aceite**:
- Termo LGPD obrigatório em todas as personas (rule 10 legado).
- Foto de perfil via `IStorageProvider`.
- Persistência pós-conclusão: migra Redis → tabelas de profile (institutional/commercial).

**Fonte**: legado `billing.md` Req 5.

---

## IDN-020 — Status de user/perfil

**Prioridade**: M  
**EARS**: *Quando* user existe, *o sistema deve* manter estado em `users.status ∈ {pending_verification, incomplete, pending_payment, active, suspended, deleted}`:
- `pending_verification` → email não confirmado.
- `incomplete` → cadastro obrigatório pendente.
- `pending_payment` → cadastro ok, falta pagamento (quando aplicável).
- `active` → tudo ok.
- `suspended` → bloqueio admin.
- `deleted` → soft delete (LGPD exclusion).

**Critério de aceite**:
- Transições válidas (máquina de estados).
- `suspended`/`deleted` → login retorna 403 imediatamente.

**Fonte**: legado `identity.md` Req 3, `ONBOARDING_CADASTRO_VS_PERFIL.md`.

---

## IDN-021 — Flag `visibilityReady`

**Prioridade**: M  
**EARS**: *Quando* cadastro mínimo + dados de visibilidade estão completos, *o sistema deve* setar `users.visibility_ready = true`. *Quando* ausente → perfil NÃO aparece em busca/discovery.

**Critério de aceite**:
- Mínimos por persona em [[registration-data]].
- Index em `(visibility_ready, role)` pra busca.

**Fonte**: legado `ONBOARDING_CADASTRO_VS_PERFIL.md`.

---

## IDN-022 — Account deletion (LGPD SLA 15d) — workflow async com soft-flag + keyed HMAC pseudonymization

**Prioridade**: M  
**EARS**: *Quando* user emite `DELETE /api/v1/me` (requer step-up MFA IDN-036), *o sistema deve*:
1. Criar ticket `account_deletion_request` (async saga — ver [[../../02-architecture/adr/0030-saga-orchestration-default]] `AccountDeletionSaga`).
2. Marcar `users._pending_hard_delete = true` + `users.deletion_scheduled_at = NOW() + 15d` + `users.status = 'pending_deletion'`.
3. Pre-cancelar Stripe subscription via API (aguardar `customer.subscription.deleted` webhook).
4. Durante a janela 15d, **todo webhook handler** (Stripe/Mux/Livekit) consulta soft-flag — se `true` + user match, skip + log em `lgpd_logs` (cobre [[../../11-audit/dual-check-log/2026-04-23-security-gaps#A-DC-SEC-004]]).
5. Day 15: worker verifica flag ainda setado + nenhum webhook pendente (`webhook_events` com user match em status pending) → else aguarda +48h.
6. Pseudonimização via **keyed HMAC rotating** (não SHA256 determinístico) — `Pseudonymize(user_id, purpose, NOW())` com `purpose ∈ {timeline_anon, vote_anon, audit_anon, contract_anon}` — ver [[../../02-architecture/adr/0028-lgpd-keyed-hmac]] (cobre [[../../11-audit/audit-log/2026-04-23-pentest-specs#A-DC-PEN-038]]).
7. Hard-delete `users`, `sessions`, `device_fingerprints`, `user_consents`; pseudonimiza `votes`, `timeline_entries`, `minutes`, `contracts`, `evaluations` (preserva utilidade legal Lei 4.591/64).
8. `users.status = 'deleted'`, `deleted_at` setado.

**Dependências**: GR-024, GR-026, IDN-036 (step-up), INV-104 (soft-flag), INV-109 (keyed HMAC), [[../../02-architecture/adr/0028-lgpd-keyed-hmac]], [[../../02-architecture/adr/0030-saga-orchestration-default]].

**Critério de aceite**:
- Audit trail: `identity.deletion.requested` e `identity.deletion.completed` em `lgpd_logs` com `user_id_hash = Pseudonymize(user_id, "audit_anon", NOW())`.
- Dados institucionais (timeline, ata, audit_trail, votos) NÃO são removidos — só pseudonimizados com `purpose` scope distinto.
- FKs com `ON DELETE RESTRICT` protegem.
- Cancelamento: usuário pode cancelar dentro de 15 dias via link email (remove soft-flag).
- Race test: webhook Stripe chegando pós-delete → soft-flag check impede UPDATE em row inexistente + log em `lgpd_logs`.
- Keys `lgpd_pseudonym_keys` rotacionadas 1/jan anualmente.

**Fonte**: legado `cross-domain.md` Req 39, LGPD Art. 18, reconstruído pós-[[../../11-audit/dual-check-log/2026-04-23-security-gaps#A-DC-SEC-004]] + [[../../11-audit/audit-log/2026-04-23-pentest-specs#A-DC-PEN-038]].

---

## IDN-023 — Data portability (export JSON)

**Prioridade**: S  
**EARS**: *Quando* user emite `GET /me/export`, *o sistema deve* gerar ZIP com JSON contendo: perfil, memberships, timeline_entries vinculadas (visible_to_user), votes emitidos, proposals recebidas/emitidas, reviews, certificates LMS. *Quando* tamanho > 50 MB → async + email com link presigned (TTL 48h).

**Dependências**: GR-025.

**Critério de aceite**:
- Formato documentado (schema JSON publicado em `/docs/portability-schema.json`).
- Teste: user com 1000 entries → ZIP gerado < 10s.
- Rate limit: 1 request/24h por user.

**Fonte**: LGPD Art. 18 II, GR-025.

---

## IDN-024 — Consentimento versionado (LGPD)

**Prioridade**: M  
**EARS**: *Quando* user aceita termos, *o sistema deve* persistir `user_term_acceptances {user_id, terms_version, accepted_at, ip, user_agent, term_type}`. *Quando* nova versão publicada, *o sistema deve* exigir re-aceite via modal bloqueante no próximo login.

**Dependências**: GR-023.

**Critério de aceite**:
- Tipos de termo: `terms_of_service`, `privacy_policy`, `code_of_conduct`, `assembly_legal` (Req 60.12).
- Tabela append-only.
- Retenção 5 anos (LGPD).

**Fonte**: legado `cross-domain.md` Req 35, GR-023.

---

## IDN-025 — Agência de marketing: invite token (sem auto-registro)

**Prioridade**: S (M2)  
**EARS**: *Quando* admin de uma Empresa Plus/Pro emite convite pra agência de marketing, *o sistema deve* gerar token único com TTL 90 dias + scope limitado (`videos:upload`, `videos:edit`, `analytics:read`). *Quando* agência aceita → cria `user` com `role=marketing`, membership delegado à empresa. *Quando* TTL expira → token invalidado automaticamente.

**Dependências**: IDN-013, GR-002.

**Critério de aceite**:
- Tabela `marketing_agency_tokens {id, enterprise_id, email, scopes, expires_at, accepted_at, revoked_at}`.
- Audit log de ações da agência.
- Revogação manual imediata pela empresa.

**Fonte**: legado `content.md` Req 30.

---

## IDN-026 — Anti-fraude: device fingerprint progressivo (D-135 — REVOGA bloqueio IP)

**Prioridade**: M (M1 — Fase C reclassificado por D-135)  
**EARS**: *Quando* request de signup chega com `X-Device-Fingerprint`, *o sistema deve* (D-135 Fase C ratificação) consultar `device_signup_log {device_fingerprint, user_id, created_at}` e aplicar **rate-limit progressivo** (não bloqueio absoluto):

| Detecção | Resposta |
|---|---|
| 1-2 contas mesmo device em 24h | `silent_log` — registra, permite |
| 3 contas mesmo device em 24h | `captcha_required` — Cloudflare Turnstile (D-117) |
| 5+ contas mesmo device em 24h | `manual_review` — admin alert + bloqueio temp 24h |

**Decisão Fase C — REJEITADO bloqueio por IP** (D-135 + R102 §1):
- Famílias compartilham wifi do condomínio → falsos positivos seriam regra (família 4 pessoas + visitas + funcionários).
- Block por IP é **rejeitado** (D-135).
- Adotado: device fingerprint hash SHA-256 server-side (já canônico STEERING §17 + IR-011).

**Critério de aceite**:
- Tabela `device_signup_log {device_fingerprint, user_id, created_at}` append-only com TTL 90d.
- `POST /signup` consulta count em window 24h e aplica resposta progressiva.
- Hash de dispositivo é **dado técnico pseudonimizado** (LGPD Art. 7º IX legítimo interesse de segurança).
- DPIA-001 documenta uso (LGPD-M1-005 pendente, ver D-107).
- Privacy notice público explicar uso de fingerprint pra segurança.
- Direito ao esquecimento: ao excluir user, dropar registros de fingerprint correspondentes (não retém 5y como audit).
- Anti-evadabilidade: combinar com email verification (IDN-002) + step-up MFA (IDN-036) em ações críticas.

**Dependências**: STEERING §17 (1-device per account), IR-011 (hash SHA-256), D-117 (Cloudflare Turnstile), ADR-0028 (LGPD HMAC), DPIA-001 (a criar M1 conforme D-107).

**Fonte**: ~~legado `cross-domain.md` Req 102~~ (bloqueio IP **REJEITADO**); **D-135 absorção Fase C** (device fingerprint progressivo); R102 §1 do `_chaos/requirements(6).md` rejeitado.

---

## IDN-027 — CAPTCHA em login após falhas

**Prioridade**: S  
**EARS**: *Quando* > 3 tentativas de login falhadas por email ou IP em < 5 min, *o sistema deve* exigir CAPTCHA (reCAPTCHA v3 ou Cloudflare Turnstile) na próxima tentativa.

**Critério de aceite**:
- Reset após login bem-sucedido.
- Frontend integra reCAPTCHA v3 com score mínimo 0.5.

**Fonte**: legado `cross-domain.md` Req 102.

---

## IDN-028 — Detecção de anomalia (login de IP novo)

**Prioridade**: C (M3)  
**EARS**: *Quando* login ocorre de IP não conhecido (nunca visto pelo user), *o sistema deve* exigir confirmação adicional via SMS ou email ("Você está tentando entrar de um novo local?"). *Quando* confirmado → prossegue; *quando* negado → revoga session imediatamente e notifica user.

**Critério de aceite**:
- Tabela `user_known_ips` atualizada em cada login bem-sucedido.
- Fallback: se SMS falhar → email.

**Fonte**: legado `cross-domain.md` Req 102.

---

## IDN-029 — Imutabilidade de role e documento

**Prioridade**: M  
**EARS**: *Quando* tentativa de `PATCH /users/{id}` altera `role` ou `documento` (CPF/CNPJ), *o sistema deve* rejeitar com `422 validation_error` `{"rule":"immutable","path":"role"}`. *Quando* usuário precisa trocar de persona → exige deletion + novo cadastro (suporte manual em M1-M2).

**Critério de aceite**:
- DB CHECK ou application-level guard.
- Audit trail de tentativas bloqueadas.

**Fonte**: legado `ONBOARDING_CADASTRO_VS_PERFIL.md`.

---

## IDN-030 — Ban local (independe de status Zitadel)

**Prioridade**: M  
**EARS**: *Enquanto* `users.banned = true` ou `ban_expires_at > now()`, *o sistema deve* responder `403 account_banned` em todas as rotas autenticadas, mesmo se Zitadel introspection retornar `active`.

**Critério de aceite**:
- Campo `users.banned_until TIMESTAMPTZ NULL`.
- Middleware Authenticate consulta antes de liberar.
- Audit trail: `identity.user.banned` e `unbanned`.

**Fonte**: legado `identity.md` Req 3 edge cases.

---

## IDN-031 — Sync Zitadel → Local (job noturno)

**Prioridade**: S  
**EARS**: *Quando* job noturno roda, *o sistema deve* comparar users Zitadel × users locais; se user deletado no Zitadel → marca `users.deleted_at` localmente; se user com role mudada → atualiza (exceto se imutável por IDN-029 → loga alerta).

**Critério de aceite**:
- Job cron diário (3h da manhã SP).
- Dry-run mode pra safety.

**Fonte**: legado `identity.md` Req 3 edge cases.

---

## IDN-032 — Invite de co-admin de condomínio (sindico_auxiliar / assistente)

**Prioridade**: S (M2)  
**EARS**: *Quando* síndico principal convida co-admin, *o sistema deve* gerar token de aceitação (TTL 7d), enviar email. *Quando* aceito → cria membership com `role=syndic`, `role_type ∈ {auxiliary, assistant}`, permissões limitadas (IDN-015).

**Dependências**: IDN-013, IDN-015.

**Critério de aceite**:
- Limite: 1 principal + até 2 auxiliares + até 3 assistentes por condomínio.
- Revogação imediata invalida sessão do co-admin.

**Fonte**: legado `institutional.md` Req 10.

---

## IDN-033 — Revogação de dependente (morador)

**Prioridade**: S  
**EARS**: *Quando* titular de unidade emite revogação de dependente, *o sistema deve* marcar membership `active=false`, revogar sessões ativas daquele dependente.

**Critério de aceite**:
- Dependente: acesso leitura-apenas, sem voto em assembleia.
- Máximo 2 dependentes por unidade.

**Fonte**: legado `institutional.md` Req 16.

---

## IDN-034 — Conflict of interest declaration

**Prioridade**: C (M3)  
**EARS**: *Quando* síndico é convocado a votar/aprovar em pauta que envolve empresa onde ele tem conflito declarado, *o sistema deve* marcar voto como "abstenção obrigatória" e alertar UI.

**Critério de aceite**:
- Declaração obrigatória no onboarding (síndico).
- Atualização anual.

**Fonte**: legado `cross-domain.md` Req 36.

---

## IDN-035 — Sessão WebSocket atrelada a session HTTP

**Prioridade**: M (M3 pra assembly live)  
**EARS**: *Quando* client abre WebSocket em `/ws/assemblies/{id}`, *o sistema deve* validar session HTTP ativa via cookie, associar WS ao `session_id`. *Quando* session revogada → WS fecha com code 1008.

**Critério de aceite**:
- Handshake valida cookie.
- Ping/pong 30s.

**Fonte**: legado `assembly.md` Req 57 notas, [[STEERING]] §15.

---

## Invariantes protegidas

1. `users.zitadel_id` UNIQUE.
2. `users.role` imutável pós-criação (IDN-029).
3. `users.documento` (CPF/CNPJ) imutável pós-primeira definição.
4. 1 sessão ativa por user (default) — `revoked_at IS NULL` UNIQUE por `user_id`.
5. `sessions.zitadel_session_id` nunca re-usado.
6. Toda session FK `user_id` em `users`.
7. Audit trail de troca de device sempre gravado.

---

## Pendências (dual-check)

- ⚠️ **PENDÊNCIA IDN-007**: política de MFA por persona (obrigatório vs opt-in) — dual-check Fase 3.
- ⚠️ **PENDÊNCIA IDN-013**: estrutura exata do `X-Tenant-Id` header — validar contra frontend UX.
- ⚠️ **PENDÊNCIA IDN-025**: renovação de token de agência (auto vs manual) — dual-check.
- ⚠️ **PENDÊNCIA IDN-028**: escolha entre reCAPTCHA v3 vs Cloudflare Turnstile — ADR M2.

---

## Referências

- Legado: `specs/requirements/identity.md` + `identity-mirror-pattern.md` + `cross-domain.md` Req 102.
- [[../global]] GR-001, GR-004, GR-023, GR-024, GR-025.
- [[../../01-domain/bounded-contexts#identity]]
- [[../../00-product/personas]]
- [[../registration-data]]
- [[../matrix-functional]]
- [[_moc]]
