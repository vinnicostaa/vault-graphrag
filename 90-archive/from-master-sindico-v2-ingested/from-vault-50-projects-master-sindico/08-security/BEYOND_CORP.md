---
title: BEYOND_CORP — Master Síndico
type: project
tags: [master-sindico, security, beyond-corp, zero-trust]
project: master-sindico
created: 2026-04-22
---

# BEYOND_CORP — Adaptado ao Master Síndico

Aplicação do modelo **BeyondCorp** (zero-trust, Google) ao contexto SaaS multi-tenant do Master Síndico. Não é um add-on de segurança — é o **core** do produto: governança condominial exige auditabilidade e controle de acesso forte.

---

## Princípios

1. **Nenhuma rede é confiável por padrão** — toda request autenticada E autorizada, independente da origem.
2. **Device-aware access** — device fingerprint parte do contexto de autorização (1 device ativo por conta).
3. **ABAC > RBAC** — decisão baseada em atributos (user, role, resource, action, tenant, plan), não só role.
4. **Context-aware decisions** — hora, localização (IP range), device trust level, plano ativo, reputação.
5. **Auditabilidade total** — todo access decision logado (IAuditLogger — DT-010).
6. **Secrets fora do código** — env vars com rotação; zitadel-key*.json gitignored.
7. **Encrypted everything** — TLS 1.3 em trânsito; at-rest em DB (Postgres) via cloud provider; Redis com AUTH.

---

## Stack middleware (ordem rígida)

A ordem importa — cada middleware adiciona ao contexto, o próximo consome.

```
1. RequestID           → gera / propaga X-Request-ID
2. Recovery            → recupera panic; retorna 500 estruturado
3. CORS                → bloqueia origin não-allowlisted (wildcard proibido em prod — A-005)
4. RateLimit           → token bucket por IP + path group
5. Authenticate        → Zitadel introspection (ou Bearer mobile); popula AuthUser
6. ABAC                → matriz (admin_bypass → role → action → tenant → scope)
7. TenantIsolation     → força WHERE tenant_id em queries subsequentes
8. DeviceFingerprint   → valida X-Device-Fingerprint; 1-device policy
9. AuditLog            → registra access decision (DT-010)
```

Implementação em `internal/shared/middleware/`.

---

## ABAC Matrix

### Atributos (input)

```yaml
user:
  id: UUID
  is_admin: bool
  role: sindico | morador | empresa | parceiro | admin
  tenant_ids: [UUID]           # condomínios aos quais pertence
  plan_tier: trial | basic | pro | enterprise
  reputation: Bronze | Prata | Ouro | Platina | Diamante

resource:
  type: condominium | unit | membership | video | assembly | subscription | ...
  id: UUID
  tenant_id: UUID              # condomínio dono (quando aplicável)
  owner_id: UUID               # user dono (quando aplicável)
  state: draft | published | archived

action: create | read | update | delete | approve | close | cancel | publish | ...

context:
  ip: string
  device_fingerprint: string
  time: timestamp
  session_id: UUID
```

### Política (pseudo-DSL)

```
PERMIT IF user.is_admin                                        (admin_bypass)
DENY   IF action NOT IN catalog[resource.type]                 (action_not_configured)
DENY   IF user.role NOT IN allowed_roles[resource.type][action] (role_not_allowed)
DENY   IF resource.tenant_id NOT IN user.tenant_ids            (tenant_mismatch)
DENY   IF required_plan[resource.type][action] > user.plan_tier (scope_restricted)
DENY   IF action=="publish" AND resource.type=="video" AND user.reputation < Prata (reputation_required)
PERMIT
```

Catálogo versionado em `shared/middleware/abac_catalog.go`.

### Cache

- Redis com TTL 5min por `{user_id, tenant_id}`
- Invalidação explícita ao receber webhook Stripe `customer.subscription.*` ou admin altera roles
- PBT em `abac_engine_pbt_test.go` (400 casos) cobrindo admin_bypass, role, action, tenant

---

## 1-Device Policy

### Fluxo
1. Login via Zitadel OIDC
2. Backend recebe token + `X-Device-Fingerprint` (hash do cliente)
3. `CreateSession` registra fingerprint
4. `InvalidateOtherSessions(user_id)` invalida sessões anteriores do mesmo user
5. Próxima request do device antigo → session revogada → 401
6. (A-023 Sprint 10) comparar fp novo vs anterior; logar `device_changed` se diferente

### Exceção
- Admin pode multi-device (role `admin`)
- `MAX_ACTIVE_SESSIONS` env var permite ajuste para testes/dev

---

## Tenant Isolation

### Regra
Toda query em recurso escopado a condomínio inclui `WHERE condominium_id = $tenant_id`.

Checado em:
- **Code review** (grep em PR)
- **sqlc queries** — convenção: parâmetro `$1 = condominium_id` primeiro
- **Testes de isolamento** — 100+ testes cross-tenant tentam ler/escrever entre tenants e esperam 403

### Anti-pattern detectado
```sql
-- RUIM
SELECT * FROM units WHERE id = $1;

-- BOM
SELECT id, number, floor_area, fracao_ideal, condominium_id
  FROM units
 WHERE id = $1 AND condominium_id = $2;
```

Aplicação pelo ABAC garante o `$2` preenchido com `user.tenant_id` do contexto.

---

## PKCE + Cookies

### PKCE
- `code_verifier` gerado no client (SPA/mobile) — 43-128 chars url-safe
- `code_challenge = SHA256(code_verifier)` enviado ao Zitadel
- No callback, verifier enviado + comparado — evita code interception

### Cookies (web)
- `session` httpOnly, Secure (prod), SameSite=Lax, domain=`.mastersindico.com.br`
- Value: JWT opaco + gorilla/securecookie encrypted
- Refresh automático antes de expirar (deve ser feito em middleware)

### Bearer (mobile)
- Access token curto (10min) + refresh token (30d)
- Refresh em endpoint dedicado `/auth/mobile/refresh` (a criar)
- Refresh revogado no logout ou 1-device trigger

---

## Webhook Security

### Stripe
- Signature header `Stripe-Signature`
- HMAC SHA-256 com `STRIPE_WEBHOOK_SECRET`
- Timestamp check (≤ 5min) pra evitar replay
- **Antes** de parsear body: verificar HMAC; rejeitar se inválido
- Idempotency: `event.id` único em `stripe_webhook_events`

### Mux
- Signature header `Mux-Signature`
- HMAC SHA-256 com `MUX_SIGNING_KEY`
- **Rota pública** em `POST /webhooks/mux` (A-022 fix — Authenticate bloqueava Mux)

### Livekit
- Webhook autenticado via JWT signing com `LIVEKIT_API_SECRET`
- Rota pública `POST /webhooks/livekit`

---

## PII / LGPD

### Proibido em logs
- CPF, CNPJ (exceto mask: `***.***.***-**`)
- Email (pode usar hash — `hash(email)[:8]`)
- Password, token, secret
- Endereço completo
- Child data

### Permitido
- `user_id` (UUID)
- `tenant_id`
- `request_id`
- `device_fingerprint_hash[:8]`

### Direito de exclusão
- Endpoint `DELETE /api/v1/me` pra user
- Hard delete: `User` + `Sessions` + `DeviceFingerprints`
- Soft archive: agregados em que user foi co-autor (votes, propostas) — anonymized name, `user_id` mantido pra auditoria
- Processo documentado em [[lgpd-compliance]]

### Audit trail (DT-010)
- `IAuditLogger` centralizado
- Eventos: login/logout, ABAC decision (deny), resource CRUD, admin override
- Retention: 5 anos (LGPD recomenda)

---

## Secrets Management

### Dev
- `.env` gitignored
- Stripe CLI webhook secret rotaciona por sessão
- Zitadel key JSON baixado localmente

### Staging/Prod
- Railway variables (encrypted at rest)
- Rotação trimestral dos secrets não-Zitadel
- Zitadel key: rotação via Zitadel console quando comprometido
- Plano: migrar para AWS Secrets Manager quando mover pra ECS

### Gitignore enforcement
```
# .gitignore mínimo — enforced em CI
.env
.env.*
!.env.example
zitadel-key*.json
*.pem
*.key
/migrate
/zitadel-setup
```

---

## Input Validation

### Camadas
1. **Handler** — shape validation (struct tags, DecodeJSON retorna 422)
2. **VO constructors** — regras de domínio (`NewEmail("bad")` retorna err)
3. **DB constraints** — CHECK, UNIQUE, FK

Proibido: validar **só** no frontend.

---

## CORS

`shared/middleware/cors.go`:
- Allowlist em `CORS_ALLOWED_ORIGINS` (csv)
- `Config.Validate()` bloqueia `*` em staging/prod (A-005)
- Credentials: allowed para `.mastersindico.com.br` domain
- Preflight cached 10min

---

## Rate Limiting

Ver [[PATTERNS#rate-limiting-token-bucket]].

- `/auth/*`: 20 rpm, burst 5 (por IP)
- `/api/v1/*`: 60 rpm, burst 10 (por IP + user_id quando auth)
- `/webhooks/*`: sem rate limit interno (confiamos em HMAC; provider externo tem rate próprio)

---

## Certificate / TLS

- **Web/Mobile → API**: TLS 1.3 obrigatório em prod (Railway provê)
- **Mobile**: certificate pinning em produção (hosts conhecidos); desabilitado em dev
- **API → providers externos**: TLS 1.2+ (Stripe, Mux exigem)
- **DB connection**: `sslmode=require` em staging/prod

---

## Incidents e Response

Ver [[incident-response]] e [[../06-execution/playbooks/incident-response]].

- Detection: Sentry + Grafana alertas
- Critical SLA: 30min de TTR
- Comms: canal do time; se LGPD impactado: DPO + usuários afetados em 72h

---

## Verificação contínua

### CI
- `gosec -severity=high` = 0
- `govulncheck ./...` = 0 reachable
- Testes de isolamento cross-tenant passam
- Testes PBT de ABAC passam

### Periódico
- **Semanal**: dep-audit via [[../../../30-agents/playbooks/dependency-audit]]
- **Mensal**: review de ABAC matrix (novas features, novas roles)
- **Trimestral**: rotação de secrets; simulação de incident
- **Anual**: pen-test externo

---

## Gates abertos (trabalho pendente)

- A-023: device change audit log comparativo (Sprint 10)
- DT-010: `IAuditLogger` cross-módulo (Sprint 10)
- D-### pendente: migrar secrets pra AWS Secrets Manager quando ECS migrar

---

## Links

- [[_moc]]
- [[threat-model]]
- [[cve-process]]
- [[owasp-top10]]
- [[lgpd-compliance]]
- [[secret-management]]
- [[../02-architecture/_moc]]
- [[../11-audit/AUDIT]]
- [[../../../10-knowledge/security/_moc]]
- [[../../../10-knowledge/security/security-principles]]
