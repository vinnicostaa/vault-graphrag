---
title: BeyondCorp — Master Síndico v2
type: spec
tags: [security, beyond-corp, zero-trust, master-sindico, v2]
source: 13-research/beyond-corp/_destilado.md + 90-ingested legacy BEYOND_CORP.md
created: 2026-04-23
updated: 2026-04-23
aliases: [BEYOND_CORP, beyond-corp-spec]
---

# BeyondCorp — Master Síndico v2

Pilar central de segurança. **BeyondCorp adaptado** ao SaaS governança condominial BR multi-tenant. Traduz NIST SP 800-207, Google BeyondCorp/BeyondProd, CISA ZTMM v2, Cloudflare/Tailscale/Okta em controles concretos para os 7 bounded contexts do master-sindico.

**Herança**: continua [[../90-ingested/from-vault-50-projects-master-sindico/08-security/BEYOND_CORP|BEYOND_CORP legado]] (v1, Sprint 10, vivo em prod). **Expansão**: incorpora research destilado em [[../13-research/beyond-corp/_destilado]] + [[../13-research/beyond-corp/source-02|NIST SP 800-207]] + [[../13-research/beyond-corp/source-05|BeyondProd]] + [[../13-research/beyond-corp/source-06|CISA ZTMM v2]].

**Não é**: add-on. É **core do produto**. Governança condominial exige auditabilidade forte por lei (LGPD + disputa judicial de ata).

---

## 1. Princípios — 7 tenets NIST SP 800-207 traduzidos

Fonte canônica: [[../13-research/beyond-corp/source-02]] §2.1.

### Tenet 1 — "All data sources and computing services are considered resources"

**Tradução master-sindico**: cada endpoint (`/api/v1/condominiums/:id`, `/api/v1/videos/:id`, `/webhooks/stripe`), cada tabela (users, timeline_entries, votes, minutes), cada integração (Stripe, Mux, Livekit, Zitadel, Twilio, SES) é um **resource** com política própria. Não existe "endpoint público desprotegido por conveniência".

**Controle**:
- Toda rota em OpenAPI 3.1 declara `security: [ABAC: [action, resource_type]]`.
- Webhooks têm política explícita (HMAC + idempotência).
- Health check `/health` é a única exceção (sem auth, sem PII, sem rate limit relevante).

### Tenet 2 — "All communication is secured regardless of network location"

**Tradução master-sindico**: TLS 1.3 em tudo norte-sul (cliente → API) + mTLS em tudo leste-oeste (BC ↔ BC) quando arquitetura evoluir pra microserviços M2+. No monolito-modular M1, comunicação inter-módulo é in-process, mas conexões DB/Redis/providers são TLS obrigatório (`sslmode=require`).

**Controle**:
- TLS 1.3 forçado pelo edge (Railway / Cloudflare / CloudFront).
- DB conn string: `sslmode=require` em staging/prod. Falha se plain TCP.
- Redis: `AUTH` + TLS (quando provedor suporta).
- Providers externos: TLS 1.2+ enforced (Stripe/Mux/Zitadel exigem por contrato SDK).
- Certificate pinning em mobile (produção).

### Tenet 3 — "Access to individual enterprise resources is granted on a per-session basis"

**Tradução master-sindico**: JWT curto (≤ 15 min access token) + refresh rotativo bound a `device_fingerprint`. Cada request revalida via Zitadel introspection quando o token é opaco, ou valida assinatura + denylist quando JWT self-contained.

**Controle**:
- Access token TTL: 10min (mobile) / 15min (web).
- Refresh token: 30d, rotativo, bound a device_id.
- Revogação: denylist Redis TTL curto + invalidação por `user_id` em webhook Stripe (subscription.cancelled).
- Session table: `sessions(id, user_id, device_fingerprint, expires_at, last_seen_at)`.

### Tenet 4 — "Access to resources is determined by dynamic policy"

**Tradução master-sindico**: ABAC engine com matriz ordenada (`admin_bypass → role → action → tenant → scope → plan → reputation`). Policy dinâmica: muda em tempo real (webhook Stripe altera `plan_tier` → cache invalidado → próxima request usa policy nova).

**Controle**: detalhado em §3 (ABAC Matrix).

### Tenet 5 — "The enterprise monitors and measures the integrity and security posture of all owned and associated assets"

**Tradução master-sindico**: nenhum dispositivo é confiável por default. `device_fingerprint` + `devices(id, user_id, fingerprint, trust_tier, first_seen_at, last_seen_at)` + trust_tier enum (`new | confirmed | suspicious | banned`).

**Controle**: detalhado em §4 (1-Device Policy + Device Posture).

### Tenet 6 — "All resource authentication and authorization are dynamic and strictly enforced before access is allowed"

**Tradução master-sindico**: middleware stack obrigatório em toda rota não-`/health`. Ordem rígida (§2). Para ações críticas: **step-up auth** (MFA on-the-fly) mesmo com sessão válida.

**Step-up auth obrigatório** em:
- Aprovar transação financeira (Stripe, contrato Connect Me acima de R$ 10k).
- Encerrar votação / publicar ata.
- Mudar síndico (transferir papel `sindico`).
- Deletar conta (direito LGPD Art. 18 V).
- Mudar email / telefone do user.
- Revogar device fingerprint de outro device.

### Tenet 7 — "The enterprise collects as much information as possible... and uses it to improve its security posture"

**Tradução master-sindico**: telemetria alimenta Policy Engine. Sinais: geolocalização nova, device novo, horário atípico, rate de requests anormal, falhas de ABAC seguidas. Elevam `risk_score` → dispara step-up auth ou soft-lock.

**Controle**:
- OpenTelemetry + Grafana/Sentry em todos os BCs.
- `audit_log` table append-only 5 anos (LGPD + disputa judicial de ata).
- `risk_signals` table (M2+) alimenta PDP em decisões de auth.

---

## 2. Stack de middleware (ordem rígida)

Cada middleware adiciona ao `context.Context`, o próximo consome. Qualquer reordenação é bug.

```
Request
  │
  ▼
1. RequestID                → gera / propaga X-Request-ID
2. Recovery                 → recupera panic; retorna 500 estruturado
3. CORS                     → bloqueia origin não-allowlisted (A-005 — wildcard proibido em staging/prod)
4. RateLimit                → token bucket (/auth/* 20rpm burst 5; /api/v1/* 60rpm burst 10)
5. Authenticate             → Zitadel introspection (ou Bearer mobile); popula AuthUser no ctx
6. TenantIsolation          → valida tenant_id no contexto; injeta WHERE condominium_id em queries
7. ABAC                     → matriz (admin_bypass → role → action → tenant → scope → plan → reputation)
8. DeviceFingerprint        → valida X-Device-Fingerprint; 1-device policy (A-023 Sprint 10: log comparativo)
9. AuditLog                 → registra decisão (DT-010 IAuditLogger)
10. StepUpGuard             → se rota marcada `auth_level: step_up`, valida MFA fresh (≤ 5min)
  │
  ▼
Handler
```

Implementação em `backend/internal/shared/middleware/`. Ver [[../02-architecture/patterns|patterns]] §middleware-stack.

### Webhooks (rota pública)

Middleware reduzido mas **HMAC ANTES de parse**:

```
Request /webhooks/{provider}
  │
  ▼
1. RequestID
2. Recovery
3. BodySizeLimit (≤ 1MB)
4. HMACVerify (Stripe-Signature / Mux-Signature / Livekit JWT) — rejeita 400 se inválido
5. IdempotencyGuard (dedupe por event.id em stripe_webhook_events / mux_events)
6. AuditLog
  │
  ▼
Handler (parse body só aqui)
```

Importante A-022 legado: **rota `/webhooks/mux` é pública**. Authenticate middleware deve pular essa rota explicitamente.

---

## 3. ABAC Matrix

### 3.1 Atributos (input)

```yaml
subject:
  id: UUID                    # user_id
  is_admin: bool              # flag plataforma (role admin)
  role: sindico | morador | empresa | parceiro | admin
  tenant_ids: [UUID]          # condomínios aos quais user pertence (memberships ativos)
  plan_tier: trial | basic | pro | enterprise
  reputation: Bronze | Prata | Ouro | Platina | Diamante  # só empresas
  mfa_fresh: bool             # MFA feito ≤ 5min atrás?

resource:
  type: condominium | unit | membership | video | assembly | vote | minutes | ...
  id: UUID
  tenant_id: UUID             # condomínio dono (quando aplicável)
  owner_id: UUID              # user dono (quando aplicável)
  state: draft | published | archived | locked

action: create | read | update | delete | approve | close | cancel | publish | vote | ...

context:
  ip: string
  device_fingerprint: string
  device_trust_tier: new | confirmed | suspicious | banned
  time: timestamp
  session_id: UUID
  risk_score: int             # 0-100 (M2+)
```

### 3.2 Política — pseudo-DSL ordenada

```
1. PERMIT IF subject.is_admin                                              (admin_bypass)
2. DENY   IF action NOT IN catalog[resource.type]                          (action_not_configured)
3. DENY   IF subject.role NOT IN allowed_roles[resource.type][action]      (role_not_allowed)
4. DENY   IF resource.tenant_id NOT IN subject.tenant_ids                  (tenant_mismatch)
5. DENY   IF required_plan[resource.type][action] > subject.plan_tier      (scope_restricted)
6. DENY   IF action == "publish" AND resource.type == "video"
            AND subject.reputation < Prata                                 (reputation_required)
7. DENY   IF auth_level[resource.type][action] == "step_up"
            AND NOT subject.mfa_fresh                                      (step_up_required)
8. DENY   IF context.device_trust_tier == "banned"                         (device_banned)
9. DENY   IF context.risk_score >= 80                                      (risk_threshold)  # M2+
10. PERMIT                                                                  (default allow após todas as checks)
```

**Catálogo versionado** em `backend/internal/shared/middleware/abac_catalog.go`. Alterações versionadas via Git + dual-check obrigatório.

### 3.3 Cache

- **Redis** com TTL 5min por `{user_id, tenant_id}`.
- **Invalidação explícita**:
  - Webhook Stripe `customer.subscription.*` → invalida todos os tenants do user.
  - Webhook Zitadel `user.role.changed` → invalida o user globalmente.
  - Admin altera membership → invalida {user, tenant}.
  - Trust tier mudou (fingerprint novo, bot detectado) → invalida user.
- **Cache versioning (AP-005)**: key format `abac:v{N}:{user}:{tenant}` — bump N invalida em massa.

### 3.4 Property-Based Testing (obrigatório)

`abac_engine_pbt_test.go` com ≥ 400 casos cobrindo:
- **admin_bypass**: `is_admin=true` sempre PERMIT (propriedade).
- **tenant_mismatch**: resource.tenant ∉ subject.tenants → sempre DENY (propriedade).
- **role_not_allowed**: combinação role × action fora do catálogo → sempre DENY.
- **step_up_required**: rota step_up + mfa_fresh=false → sempre DENY.
- **reputation_required**: Video.publish + reputation < Prata → sempre DENY.

Ver [[../09-testing/_moc]] (a escrever).

---

## 4. 1-Device Policy + Device Posture

### 4.1 Fluxo

1. Login via Zitadel OIDC (PKCE obrigatório).
2. Backend recebe token + header `X-Device-Fingerprint` (hash client-side: IP + UA + canvas hash).
3. `CreateSession` registra: `{session_id, user_id, device_fingerprint, ip, ua, created_at, expires_at}`.
4. `InvalidateOtherSessions(user_id)` invalida sessões anteriores (soft: `revoked_at=NOW()`).
5. Próxima request do device antigo → session revogada → 401 + forçar logout.
6. **A-023 (Sprint 10)**: comparar fingerprint novo vs anterior; se diferente, log `device_changed` em `audit_log` com `{old_fp_hash[:8], new_fp_hash[:8], ip_diff, ua_diff}`.

### 4.2 Exceções

- Role `admin` pode multi-device (ops, SRE, devs).
- Env `MAX_ACTIVE_SESSIONS` permite ajuste em dev/staging.
- **⚠️ PENDÊNCIA**: avaliar exceção pra role `empresa` com múltiplos representantes (mesmo CNPJ usa app de celulares diferentes). Proposta M2: cada CNPJ tem N `CompanyRepresentative` → cada representative tem seu User → 1-device aplica ao User, não ao CNPJ.

### 4.3 Device Posture (pilar herdado do research)

**M1 (base)**:
- `device_fingerprint` hash (IP + UA + canvas).
- `devices(id, user_id, fingerprint, trust_tier, first_seen_at, last_seen_at)`.
- trust_tier inicial: `new` → `confirmed` após 7d de uso estável + 0 falhas ABAC.
- `suspicious` se: IP muda mais de 3x/24h; fingerprint muda 2x/7d; falha ABAC > 5/h.
- `banned` = manual por ops (ou ML M4+).

**M2 (hardening — research pilar)**:
- **Atestação de plataforma** em mobile:
  - Android: **Play Integrity API** (atesta: app not tampered, device não-rooted, boot verified).
  - iOS: **DeviceCheck** + App Attest.
  - Web: **reCAPTCHA Enterprise** + fingerprint JS.
- Obrigatório em: votar em assembleia (anti-bot), aprovar transação financeira, publicar vídeo (anti-spam).

**M3 (optimal — CISA ZTMM)**:
- Continuous attestation — reavalia postura a cada session refresh.
- Risk score dinâmico alimenta ABAC tenet 7.

Ver [[../13-research/beyond-corp/source-03]] (BeyondCorp Design to Deployment — device inventory + trust tiers).

---

## 5. PKCE + Cookies

### 5.1 PKCE (OAuth 2.1)

- `code_verifier`: 43-128 chars url-safe, gerado no client (SPA/mobile).
- `code_challenge = SHA256(code_verifier)` enviado a Zitadel no authorize request.
- No callback, `code_verifier` enviado + comparado → evita code interception attack.
- **Obrigatório** em todos os flows OIDC (web + mobile).

### 5.2 Cookies (web)

- Nome: `ms_session`.
- Flags: `httpOnly`, `Secure` (prod), `SameSite=Lax`, `Domain=.mastersindico.com.br`.
- Conteúdo: access_token + refresh_token + code_verifier — **encrypted** via `gorilla/securecookie` (AES-256-GCM + HMAC-SHA256).
- Refresh automático: middleware renova access_token antes de expirar (grace 2min).

### 5.3 Bearer (mobile)

- Access: 10min, em memória (não persistir em disk sem cifra adicional).
- Refresh: 30d, persistido em **Keychain (iOS) / EncryptedSharedPreferences (Android)**.
- Endpoint refresh dedicado: `POST /auth/mobile/refresh` (rate limit `/auth/*` 20rpm).
- Revogado em logout + em trigger 1-device (login em outro device).

---

## 6. Rate Limiting escalonado

Token bucket com cleanup (A-032 resolvido Sprint 10).

| Path group | Rate | Burst | Chave |
|---|---|---|---|
| `/auth/*` | 20 rpm | 5 | IP |
| `/api/v1/*` (anonymous) | 20 rpm | 5 | IP |
| `/api/v1/*` (authenticated) | 60 rpm | 10 | IP + user_id |
| `/webhooks/*` | sem rate interno | — | confia em HMAC + provider rate |
| `/health` | 600 rpm | 20 | IP |
| WebSocket `/live` | 5 conn / user | — | user_id |

**M2+**: adicionar rate limit distribuído via Redis (INCR + EXPIRE) quando houver mais de 1 réplica da API. M1 `sync.Map` + cleanup goroutine é suficiente.

**Anti-DDoS**: WAF no edge (Cloudflare ou CloudFront+AWS WAF — ⚠️ PENDÊNCIA D-### a decidir).

Ver [[../13-research/beyond-corp/source-10|Cloudflare One]] pra reference arch.

---

## 7. PII masking — logs sem dados pessoais

### 7.1 Proibido em logs

- CPF, CNPJ (apenas mask: `***.***.***-**`, `**.***.***/****-**`).
- Email (opcional mask: `hash(email)[:8]` ou `u***@d***.com`).
- Password, token, API key, secret.
- Endereço completo.
- Child data (moradores < 18 anos em dados de dependentes).
- Telefone completo (mask: `(**)****-**NN`).

### 7.2 Permitido

- `user_id` UUID.
- `tenant_id` / `condominium_id` UUID.
- `request_id`.
- `device_fingerprint_hash[:8]`.
- `session_id`.
- Domínio de email sem o local-part (`@mastersindico.com.br` ok).

### 7.3 CI enforce

- Grep regex em PR bloqueando commit de logs com padrões PII:
  - `\d{3}\.\d{3}\.\d{3}-\d{2}` (CPF literal)
  - `\d{2}\.\d{3}\.\d{3}/\d{4}-\d{2}` (CNPJ literal)
  - `"email":\s*"[^"]*@` (email em JSON log)
- VOs com método `Masked() string` (já herdado do legado — `pkg/cpf/`, `pkg/cnpj/`, `pkg/email/`).
- Review: zerolog/slog `fmt` strings revisadas em PR.

---

## 8. HMAC webhook verification

### 8.1 Stripe

- Header: `Stripe-Signature` (`t=timestamp,v1=signature,v0=deprecated`).
- HMAC-SHA256 com `STRIPE_WEBHOOK_SECRET`.
- **Timestamp check ≤ 5min** (evita replay).
- Verificação **ANTES** de parse body → rejeita 400 se inválido.
- Idempotency: `stripe_webhook_events(event_id UNIQUE)` + INSERT ON CONFLICT DO NOTHING.

### 8.2 Mux

- Header: `Mux-Signature`.
- HMAC-SHA256 com `MUX_SIGNING_KEY`.
- **Rota pública** `/webhooks/mux` (A-022 fix — Authenticate middleware deve skipar).
- Idempotency via `mux_events(event_id UNIQUE)`.

### 8.3 Livekit

- Webhook body é JWT signed com `LIVEKIT_API_SECRET`.
- Verifica claims: `iss`, `iat`, `exp`, `nbf`.
- Rota pública `/webhooks/livekit`.

### 8.4 Timing attack protection

- Comparação de HMAC via `hmac.Equal()` (Go) — tempo constante.
- Nunca usar `==` direto em byte slices de signature.

---

## 9. CORS allowlist estrita

Implementação em `backend/internal/shared/middleware/cors.go`.

- **Allowlist** via env `CORS_ALLOWED_ORIGINS` (CSV).
- **Wildcard `*` proibido** em staging/prod → `Config.Validate()` falha boot (A-005).
- **Credentials** allowed apenas pra `.mastersindico.com.br`.
- **Preflight** cached 10min (`Access-Control-Max-Age: 600`).
- Methods: `GET, POST, PUT, PATCH, DELETE, OPTIONS`.
- Headers: `Authorization, Content-Type, X-Request-ID, X-Device-Fingerprint`.

---

## 10. Secrets gitignored + rotação

### 10.1 Dev

- `.env` em `.gitignore`.
- Stripe CLI rotaciona webhook secret por sessão (dev only).
- Zitadel key JSON baixado localmente via `zitadel-setup` CLI.

### 10.2 Staging / Prod

- Railway Variables (encrypted at rest) em M1.
- **AWS Secrets Manager** em M2+ (quando migrar pra ECS — D-024).
- Rotação trimestral dos secrets não-Zitadel.
- Rotação Zitadel: via console quando comprometido + playbook incident response.

### 10.3 Gitignore mínimo (enforced em CI)

```gitignore
.env
.env.*
!.env.example
zitadel-key*.json
*.pem
*.key
stripe-webhook-secret.txt
mux-signing-key.txt
livekit-api-secret.txt
/migrate
/zitadel-setup
```

### 10.4 Secret scanning

- **gitleaks** em pre-commit hook + CI job obrigatório.
- **GitHub Secret Scanning** ativado no repo.
- A-018 (resolvido): zitadel-key*.json chegou a ser commitado → rotacionado + gitignore reforçado.

---

## 11. Audit trail LGPD append-only

### 11.1 Eventos auditados

- Auth: login, logout, token refresh, 1-device trigger, MFA step-up.
- ABAC: decisão DENY (sempre) + decisão PERMIT em recurso sensível (votos, atas, contratos > R$ 10k).
- Resource CRUD: create/update/delete em aggregates críticos (Assembly, Minutes, Vote, ConnectMeRequest, Contract, User).
- Admin override: `is_admin=true` bypass registrado com justificativa (campo `admin_reason`).
- LGPD events: consent_given, consent_revoked, data_export_requested, data_deletion_requested, data_deleted.

### 11.2 Schema (DT-010 + LGPD)

```sql
CREATE TABLE audit_log (
    id              UUID PRIMARY KEY DEFAULT uuidv7(),
    occurred_at     TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    actor_user_id   UUID,                              -- nullable (sistema)
    actor_role      TEXT,
    tenant_id       UUID,
    action          TEXT NOT NULL,                     -- 'login', 'vote.cast', 'minutes.publish', ...
    resource_type   TEXT,
    resource_id     UUID,
    outcome         TEXT NOT NULL,                     -- 'permit' | 'deny' | 'error'
    deny_reason     TEXT,                              -- se outcome='deny'
    ip              INET,
    device_fp_hash  TEXT,                              -- hash[:8]
    request_id      UUID,
    metadata        JSONB                              -- sem PII
);

-- Append-only enforcement
REVOKE UPDATE, DELETE ON audit_log FROM app_role;
-- Particionamento mensal pra retention 5 anos
-- Partition pruning após 5 anos → export pra S3 Object Lock
```

### 11.3 Retention

- **5 anos** online (LGPD Art. 16).
- Pós-5 anos: export WORM S3 Object Lock **Compliance mode** 5 anos adicionais (disputa judicial de ata).
- Purge automático via partition drop + export.

### 11.4 Imutabilidade (defesa em profundidade)

- Sem UPDATE/DELETE grants pra app_role.
- M2: **hash-chain rolling** — `hash(prev_hash || row_content)` em cada insert; permite provar tampering.
- M3: **S3 Object Lock Compliance mode** imediato (WORM desde o insert — dual-write).

---

## 12. Pilares adicionais herdados do research (gaps v1)

Ver [[../13-research/beyond-corp/_destilado]] §3 pra gaps identificados. Tradução em pilares a introduzir:

### 12.1 Device posture evaluation (M2)

Ver §4.3. Fonte: [[../13-research/beyond-corp/source-11|Tailscale]] + [[../13-research/beyond-corp/source-12|Okta Device Assurance]] + [[../13-research/beyond-corp/source-07|Google IAP context-aware]].

### 12.2 Policy Engine / Policy Administrator / Policy Enforcement Point (M1+)

**NIST SP 800-207 §3.1**. Separação explícita:

- **PE (Policy Engine)**: OPA (Open Policy Agent) sidecar ou Cedar (AWS Verified Permissions).
  - ⚠️ PENDÊNCIA ADR: **OPA vs Cedar** — recomendação provisória OPA (portável, multi-cloud futuro). Ver [[../13-research/beyond-corp/_destilado]] §6.
- **PA (Policy Administrator)**: serviço de sessão em BC-identity que emite/renova/revoga tokens.
- **PEP (Policy Enforcement Point)**: middleware ABAC em cada BC + API Gateway central.

**M1**: lógica ABAC centralizada em `shared/middleware/abac.go` (PEP+PE in-process).
**M2**: extrai PE pra sidecar OPA — policies em Rego versionadas em Git.
**M3**: PE distribuído com policy bundle + telemetry feedback.

### 12.3 Continuous authentication (M2)

Não autentica só no login — **reavalia a cada request crítica**.

- Sinais: IP mudou, device mudou, horário atípico, falhas ABAC.
- Ação: eleva `risk_score` → dispara step-up ou soft-lock.
- Infraestrutura: `risk_signals` table + worker que atualiza `users.risk_score`.

### 12.4 Least-privilege via RBAC + ABAC híbrido (M1)

- **RBAC** define conjunto de actions permitidas pra role (catálogo).
- **ABAC** refina com atributos do resource + context.
- Nunca "role X pode tudo". Sempre intersection RBAC ∩ ABAC.

### 12.5 SPIFFE/SPIRE — inter-service identity (M3+)

Quando arquitetura virar microserviços, cada serviço tem SVID (SPIFFE Verifiable Identity Document) X.509 curto rotacionado automaticamente. **Não** identidade por hostname/IP.

Ver [[../13-research/beyond-corp/source-05|BeyondProd]] (ALTS = SPIFFE do Google).

### 12.6 mTLS entre serviços internos (M2+)

- **M1**: monolito-modular + TLS em providers externos.
- **M2**: mTLS manual nos 3 caminhos críticos (identity ↔ billing, identity ↔ assembly, gateway ↔ BCs).
- **M3**: service mesh (Linkerd — mais leve que Istio).

### 12.7 JIT/JEA — just-in-time / just-enough-access (M2+)

Admin access a prod via **elevação temporária** (ex: TTL 1h) com justificativa + approval:
- Ops precisa SSH em DB → requisita via Teleport / AWS Session Manager → aprovação async + TTL.
- **Sem** credenciais admin permanentes em workstations de dev.
- Cloudflare Access / Tailscale são candidatos (ver [[../13-research/beyond-corp/source-10|Cloudflare]], [[../13-research/beyond-corp/source-11|Tailscale]]).

---

## 13. Mapping pilar × BC

| BC | Identity | Device | Network | App | Data |
|---|---|---|---|---|---|
| **identity** | IdP Zitadel + PKCE + MFA | fingerprint + 1-device + A-023 audit | TLS 1.3 edge | PDP central (OPA M2) | audit_log + sessions |
| **billing** | tenant check + role=sindico/financeiro | 1-device herda | mTLS Stripe | ABAC + step-up em approve | stripe_webhook_events (idempotent) |
| **institutional** | role=sindico/morador + condo_id | herda | TLS | ABAC + tenant_isolation | timeline append-only imutável |
| **commercial** | role=empresa + reputation | fingerprint + Play Integrity (M2) | TLS + ranking queries tenant-filtered | ABAC + plano + reputação | company + evaluations + contracts |
| **content** | role + reputation ≥ Prata pra publish | Play Integrity (mobile upload) | Mux signed URLs | ABAC + video state (draft/moderation/published) | videos + moderation_queue |
| **assembly** | role=morador elegível + unit_id | **Play Integrity obrigatório M2** (anti-bot voting) | Livekit JWT com ACL mínimo | ABAC + vote_session_state + UNIQUE vote | votes + minutes (imutável) |
| **cross-domain** | domain events typed | fingerprint propagado | mTLS inter-BC (M2+) | IAuditLogger DT-010 | lgpd_logs |
| **neighborhood** | role=parceiro/morador + geo | herda | TLS | ABAC + raio geo + seal síndico | coupons (lock 4h) + seals |

---

## 14. Roadmap de controles

### 14.1 M1 (2026-05-08) — fundações inegociáveis

- [x] IdP Zitadel + OAuth 2.1 + PKCE (D-001 legado).
- [x] MFA em Zitadel (opcional no morador, obrigatório no admin).
- [x] ABAC matriz coarse (admin_bypass → role → action → tenant → scope).
- [x] Cache Redis 5min + invalidação webhook.
- [x] 1-device + fingerprint IP+UA+canvas.
- [x] Rate limit token bucket /auth + /api/v1.
- [x] HMAC webhook Stripe + Mux + Livekit.
- [x] CORS strict (`Config.Validate()`).
- [x] Secrets gitignored + gitleaks.
- [x] audit_log append-only (DT-010 a finalizar Sprint 10).
- [ ] **A-023 device change audit comparativo** (pendente).
- [ ] TLS `sslmode=require` staging/prod.
- [ ] LGPD base: consent versionado, GET /me, DELETE /me/data.
- [ ] Gates CI: gosec -sev high 0 + govulncheck 0 reachable.
- [ ] PBT ABAC ≥ 400 casos.

### 14.2 M2 (2026-06-20) — hardening

- [ ] **MFA obrigatório em síndico + empresa + financeiro**.
- [ ] **Step-up auth** em ações críticas (§1 Tenet 6).
- [ ] **Play Integrity / DeviceCheck** em assembleia + approve transaction.
- [ ] **Device trust_tier** + suspicious detection heurística.
- [ ] **PDP extraído** em OPA sidecar (ADR OPA vs Cedar decidido).
- [ ] **mTLS nos 3 caminhos críticos** (manual, cert rotation trimestral).
- [ ] **hash-chain rolling** no audit_log.
- [ ] **LGPD pleno**: export /me, DELETE /me SLA 15d, breach notify 72h.
- [ ] WAF (Cloudflare ou AWS WAF — ADR D-### decidido).
- [ ] Secret Manager (AWS Secrets Manager quando migrar D-024).
- [ ] Pen-test externo 1º ciclo.

### 14.3 M3 (2026-07-20) — maturidade Advanced CISA ZTMM

- [ ] **Service mesh** Linkerd (mTLS automático + observabilidade leste-oeste).
- [ ] **SPIFFE/SPIRE** identidade workload (se multi-cloud / on-prem).
- [ ] **Binary Authorization** (cosign + Kyverno/OPA Gatekeeper) no pipeline.
- [ ] **S3 Object Lock Compliance** (audit WORM imediato).
- [ ] **Continuous risk scoring** (behavior anomaly → step-up dinâmico).
- [ ] **Game days / red team** — validar assume breach.
- [ ] Certificações ISO 27001 / SOC 2 — kickoff (auditor externo).

### 14.4 Futuro (M4+) — Optimal

- [ ] Sandboxing workloads sensíveis (gVisor/Kata).
- [ ] ZTNA pra integrações externas (Open Banking, registros, CRM).
- [ ] ML anomaly detection alimentando PDP.
- [ ] Automação SOAR (revogar session + banir device em incident).
- [ ] ISO 27001 / SOC 2 com auditor certificado.

---

## 15. Verificação contínua

### 15.1 CI gates (hard fail)

```bash
gosec -severity=high ./...                   # 0 findings
govulncheck ./...                            # 0 CVEs reachable
gitleaks detect --source .                   # 0 secrets
go test -race -count=1 ./...                 # todos verdes
go test -tags=integration -count=1 ./...     # incluindo cross-tenant tests
go test -tags=pbt ./internal/shared/middleware  # PBT ABAC
```

### 15.2 Periódico

- **Diário**: Dependabot alerts + GitHub Secret Scanning.
- **Semanal**: dep-audit via [[../06-execution/playbooks/dependency-audit]] (a criar).
- **Mensal**: review de ABAC matrix (novas features, novas roles) + revisão de allowlist CORS + catálogo.
- **Trimestral**: rotação de secrets; simulação de incident response; review STRIDE por BC.
- **Anual**: pen-test externo; audit LGPD (DPO + auditor); review ZTMM (maturity assessment).

---

## 16. Gates abertos (trabalho pendente — ⚠️ PENDÊNCIAS)

Findings herdados [[../AUDIT]] + novas decisões:

- **A-023**: device change audit log comparativo (Sprint 10 ativo).
- **DT-010**: `IAuditLogger` cross-módulo finalização (Sprint 10).
- **⚠️ ADR-### pendente**: IdP final (Cognito vs Keycloak vs Auth0 vs Entra ID vs Zitadel continua).
- **⚠️ ADR-### pendente**: PDP (OPA vs Cedar).
- **⚠️ ADR-### pendente**: Edge/WAF (Cloudflare vs AWS WAF).
- **⚠️ ADR-### pendente**: Audit storage (S3 Object Lock vs TimescaleDB+hash-chain).
- **⚠️ ADR-### pendente**: mTLS em M1 ou M2? — recomendação provisória: M2 com manual 3-caminhos.
- **⚠️ A-SEC-### novos** esperados na Fase 5 (pentester pass): ver §16 de [[threat-model]].

---

## 17. Links

- [[_moc]]
- [[threat-model]] — STRIDE por BC + LINDDUN + gaps
- [[owasp-top10]] — mapping OWASP → controles
- [[cve-process]] — SLA + tools + workflow
- [[lgpd]] — controles LGPD 13.709/2018
- [[pentest-checklist]] — checklist de auditoria
- [[beyond-corp-references]] — ponte com research
- [[../STEERING]] §princípios-de-segurança
- [[../CLAUDE]] §9
- [[../AUDIT]]
- [[../13-research/beyond-corp/_destilado]]
- [[../13-research/beyond-corp/source-02|NIST SP 800-207]]
- [[../13-research/beyond-corp/source-05|BeyondProd]]
- [[../13-research/beyond-corp/source-06|CISA ZTMM v2]]
