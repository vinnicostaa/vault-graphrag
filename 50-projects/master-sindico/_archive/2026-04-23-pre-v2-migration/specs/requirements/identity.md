---
title: Identity — Autenticação, Autorização e Registro
version: 1.0
status: stable
type: requirement
domain: identity
tags: [auth, oidc, abac, zitadel, registration, role-based-access]
---

# Identity — Autenticação, Autorização e Registro

Domínio responsável por gestão de identidade, autenticação via Zitadel OIDC e autorização baseada em atributos (ABAC). Uma identidade única (1 CPF/CNPJ) com múltiplos contextos de operação.

## Req 1 — Autenticação

**Stack:** Zitadel OIDC (Authorization Code Flow + PKCE) com cache Redis de introspection.

### Requisitos MUST

- Authorization Code Flow com PKCE via Zitadel (`zitadel/oidc/v3` — T11)
- Refresh token gerenciado **exclusivamente** pelo Zitadel; backend valida access token via introspection
- Cache Redis `ms:auth:{zitadelUserID}` com TTL 5 minutos (T13)
- Cookie `access_token` httpOnly no domínio raiz `.mastersindico.com.br`, `Secure`, `SameSite=Lax` (T9)
- Fallback `Authorization: Bearer` (RFC 6750) para mobile além de cookie (T14)
- **1 dispositivo ativo por conta** (fingerprint + IP tracking) — login em segundo device invalida sessão anterior (Rule 8)
- Rate limiting contra brute force
- Audit log de todas as tentativas (timestamp + IP)
- Anti-fraude: bloqueia múltiplos cadastros do mesmo IP (Req 102)
- Guard clause: tokens sem role retornam `403 NO_ROLE_ASSIGNED` **antes** do UPSERT (T15)

### Requisitos SHOULD

- OAuth Google e Apple: pós-lançamento (Sprint 5+) — apenas habilitação no painel Zitadel, sem desenvolvimento adicional

### Notas de implementação

- Rotas públicas de auth (`GET /auth/login`, `/callback`, `/logout`) **fora** de `/api/v1` (sem middleware Authenticate) — T12
- Middleware Authenticate verifica cache Redis; fallback: introspection ao Zitadel
- Sessão local (`sessions` table) vinculada a `device_fingerprint` + `ip_address`
- Ver `identity-mirror-pattern.md` para fluxo completo de sincronização User/Session ↔ Zitadel

---

## Req 2 — Autorização ABAC

**Stack:** Engine próprio (sem Casbin) — avalia `(userId, tenantId, role, roleType, planTier, action)` em cada request.

### Requisitos MUST

- Middleware `RequirePermission(action, resource string)` intercepta toda rota protegida (Sprint 1, task 1.2)
- Retorna: `allowed` (boolean), `role`, `planTier`, features disponíveis
- **Roles primárias** (1:1 com Zitadel, em inglês): `syndic`, `resident`, `enterprise`, `marketing`, `local_company`, `admin` (T7)
- **Role types** (sub-contexto operacional, **apenas no banco**, não no Zitadel): 
  - `principal`, `auxiliary`, `assistant` (síndico)
  - `titular`, `dependent` (morador)
  - `administrator`, `operator` (empresa) (T8)
- Cross-tenant via grants explícitos (Connect Me, validações)
- Tenant isolation: usuários só acessam dados do próprio tenant (Rule 9)

### Cadeia de middleware BeyondCorp

```
Recovery → RequestID → Logger → CORS → ErrorHandler
  → [rotas protegidas]:
      → RateLimiter (por IP + userId)
      → Authenticate (JWT → Zitadel introspection → cache Redis → user sync → 1-device check)
      → RequirePermission(action, resource)
  → Handler
  → AuditLogger (Sprint 3)
```

### Requisitos SHOULD

- Feature flags para rollout gradual (Sprint 3+)
- Matriz de ABAC detalhada em `functional-matrix.md`

### Notas de implementação

- Implementação atual: `internal/shared/middleware/` — `RateLimiter`, `Authenticate`, `RequirePermission`
- Decisão em aberto D-001 (`.kiro/STATE.md`): purge instantâneo de cache Redis via webhook Zitadel vs aceitação de staleness de 5min

---

## Req 3 — Registro

**Stack:** Auto-registro integrado com Zitadel; criação de mirror local via UPSERT no primeiro login.

### Requisitos MUST

- **Personas que se registram diretamente**: `syndic`, `resident`, `enterprise`, `local_company`
- **Personas que NÃO se registram**: `marketing` (Agência) — é convidada por uma empresa via token (Sprint 5); `admin` — provisionada manualmente no Zitadel (sem fluxo de auto-registro)
- Campos obrigatórios: `email`, `password`, `name`, `phone`, `role`
- Verificação de email obrigatória (responsabilidade do Zitadel)
- Hash de senha: gerenciado **exclusivamente** pelo Zitadel (Argon2 internamente)
- Status: `pending_verification` → `active` → `suspended` → `deleted`
- User sync: UPSERT em `users` table no primeiro login via callback Zitadel (`zitadel_id` como chave de sincronização)

### Requisitos SHOULD

- Onboarding pós-registro com auto-save de progresso em Redis (Req 5)
- Campos extras por persona (Req 6.2, 6.3, 6.4)

### Edge cases

- Email mudou no Zitadel → próxima introspection traz novo email, UPSERT atualiza
- Role removida de user → `role = ''` nos claims → middleware rejeita `403 NO_ROLE_ASSIGNED` antes do UPSERT
- User banido localmente → `banned = true` + `ban_expires_at` → 403, mesmo que ativo no Zitadel
- User deletado no Zitadel → introspection falha, middleware rejeita 401; job noturno sincroniza `users.deleted_at`

### Notas de implementação

- Fluxo completo em `identity-mirror-pattern.md` seção "Fluxo de sincronização"
- Ver também `personas-and-plans.md` para tipos de persona e planos disponíveis por tipo
- FKs com `ON DELETE RESTRICT` protegem dados históricos (subscriptions, timeline, etc)

---

## Invariantes protegidas

1. `users.zitadel_id` é UNIQUE — 1 user no Master Síndico = 1 user no Zitadel
2. `users.role` nunca é string vazia — middleware rejeita antes do UPSERT
3. Toda `session` tem `user_id` que existe em `users` (FK)
4. Um user tem no máximo 1 session ativa — 1-device enforcement via `DELETE` antes do `INSERT`
5. `sessions.zitadel_session_id` nunca é re-usado após revogação — nova session é `INSERT`, não `UPDATE`

---

## Referências cruzadas

- **Identity Mirror Pattern**: `identity-mirror-pattern.md` — decisão e fluxo de sincronização
- **Personas e Planos**: `personas-and-plans.md` — mapeamento role → plano
- **Product Overview**: `product-overview.md` — decisões técnicas T1–T15, regras de ouro
- **Functional Matrix**: `functional-matrix.md` — matriz ABAC completa (role × action × plan_tier)
- **Steering docs**: `.kiro/steering/security.md`, `.kiro/steering/go-patterns.md`
