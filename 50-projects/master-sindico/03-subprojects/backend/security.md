---
title: Backend — Security posture (destilação Fase 8, código real)
type: spec
tags: [security, backend, beyondcorp, abac, lgpd, master-sindico, v2, destilacao-fase-8]
source: /home/vinni/Documentos/Projetos/Clientes/Joao/MasterSindico/Development/backend + .kiro/AUDIT.md + .kiro/STATE.md
created: 2026-04-23
updated: 2026-04-23
---

# Backend — Security posture (BeyondCorp adaptado)

Destilação direta do código + AUDIT.md + STATE.md. Snapshot 2026-04-22.

---

## 1. Fontes consultadas (2026-04-23)

| Path | Trecho / uso |
|---|---|
| `backend/.kiro/AUDIT.md:170-205` | BeyondCorp audit 14 invariantes — 12/14 verdes |
| `internal/shared/middleware/authenticate.go` | Zitadel introspection + 1-device enforcement |
| `internal/shared/middleware/abac_engine.go` + `abac_rules.go:1-789` | ABAC matrix decision engine |
| `internal/shared/providers/auth/oidc_handler.go:110-202` | Cookies HttpOnly + Secure + SameSite + PKCE |
| `internal/shared/providers/auth/zitadel_provider.go` | Introspection via OAuth2 |
| `internal/shared/middleware/rate_limiter.go` | Token bucket + cleanup A-032 |
| `internal/modules/billing/infrastructure/providers/stripe_gateway.go` | HMAC Stripe |
| `internal/modules/content/infrastructure/providers/mux_provider.go` | HMAC Mux |
| `.gitignore` + `internal/shared/config/config.go` | Secrets + CORS validation |
| `pkg/safecast/` + `internal/shared/pagination/` | Clamp overflow (G115) |

---

## 2. BeyondCorp — 14 invariantes

Auditoria manual 2026-04-22 linha-a-linha. Resultado:

| # | Invariante | Status | Evidência |
|---|---|---|---|
| 1 | Never trust frontend | ✅ | Todos handlers validam server-side; `DecodeJSON` + guards |
| 2 | Zero Trust | ✅ | `Authenticate` em todo `/api/v1/*` via `RegisterModules` |
| 3 | 1-device per account | ✅ | `SyncUserUseCase.Execute` chama `InvalidatePreviousByUserID` (`sync_user_use_case.go:106`) |
| 4 | Cookie httpOnly + Secure + SameSite=Lax | ✅ | 4 pontos em `oidc_handler.go:110-202` |
| 5 | Least privilege credentials | ✅ | Zitadel JWT Profile; Stripe test em dev; `.gitignore` cobre `zitadel-key*.json` |
| 6 | Secrets nunca em repo | ✅ | Gosec G101 = 0 findings; `.gitignore` reforçado em A-018 |
| 7 | Middleware stack ordenado | ✅ | Recovery → RequestID → Logger → CORS → ErrorHandler → RateLimiter → Authenticate → RequirePermission |
| 8 | Zitadel JWT Profile | ✅ | `WithInsecure` só para `ZITADEL_PORT != 443` (dev) |
| 9 | ABAC engine com matriz explícita | ✅ | `abac_rules.go` 70+ actions + 400 PBT cases + 100+ table-driven |
| 10 | Webhook signature antes de parse | ✅ | Stripe + Mux verificam HMAC-SHA256 antes de parsear body |
| 11 | Device fingerprint via `X-Device-Fingerprint` | ✅ | Lido em `authenticate.go:110` + propagado em `SyncUserCommand` |
| 12 | Vulnerability scanning | ✅ | `gosec -severity=high` 0 findings; `govulncheck` 0 CVEs |
| 13 | **1-device change audit log** | 🔴 A-023 | Sem comparação de fingerprint anterior vs atual; sem `invalidation_reason="device_changed"` |
| 14 | **Rate limit per-userID** | 🔴 A-039 | Só por IP; spec pede 300 rpm por userID |

---

## 3. Stack middleware (ordem rígida)

```
request →
  Tracing(serviceName)                 # se OTEL_ENABLED
  → Recovery                           # panic → 500
  → RequestID                          # gera X-Request-ID se não vier
  → Logger                             # 1 log com method, path, status, duration, trace_id, span_id
  → CORS                               # origin-específico; config.Validate rejeita "*" em stg/prod
  → ErrorHandler                       # ctx.SetError(err) → JSON {code, message, status}
  → [rota /auth/*]
      RateLimiter(ctx, 20 rpm, burst 5)
      → OIDCHandler.Login/Callback/Logout
  → [rota /api/v1/*]
      RateLimiter(ctx, 60 rpm, burst 10)  # se RateLimit.Enabled
      → Authenticate                      # Zitadel introspection + UPSERT + 1-device
      → [por rota]
          RequirePermission(action, ...)   # ABAC engine
          → handler.Execute
```

Verificação Green no audit 2026-04-22.

---

## 4. Autenticação (Zitadel OIDC)

### 4.1 PKCE flow (`auth/oidc_handler.go`)

1. `GET /auth/login` gera `code_verifier` + `state` + cookies criptografados (gorilla/securecookie HMAC + AES).
2. Redireciona para Zitadel login UI.
3. Zitadel → `GET /auth/callback?code=...&state=...`.
4. Handler valida `state` cookie; troca `code` por access_token + refresh_token via `zitadel-go/v3`.
5. Emite cookie `ms_session` com token (HttpOnly, Secure, SameSite=Lax, Domain `.mastersindico.com.br` em prod).
6. Redirect para frontend.

### 4.2 Middleware `Authenticate`

```
header Authorization=Bearer OR cookie ms_session
  → cache Redis ms:auth:{zitadelUserID} (TTL 5min)
     cache hit → AuthUser reconstituído → ctx
  → cache miss:
     zitadelProvider.Introspect(token)
     invalid → 401 unauthenticated
     valid:
       userSyncer.Execute(SyncUserCommand{..., DeviceFingerprint, IPAddress})
         → UPSERT users
         → sessionRepository.InvalidatePreviousByUserID(userID, sessionID)  # 1-device
         → Popula AuthUser.CondominiumIDs via condominiumIDsProvider
         → Redis cache
     → AuthUser em ctx
```

**Gap DT-003**: sem circuit breaker (sony/gobreaker) — fail em Zitadel = 503 em cada request. Sprint 10.

### 4.3 Token sem PII em log

Tokens **nunca** logados em claro. Apenas prefixo anonimizado quando estritamente necessário em log de erro (`token_prefix: "Bearer e..."` mascarado).

---

## 5. Autorização (ABAC engine custom)

### 5.1 Design

- **Matriz declarativa** em `abac_rules.go` (789 linhas, 70+ actions × ~130 rules).
- `Matrix[Action] = []Rule` — O(1) lookup.
- `Rule` com condições não-vazias AND, múltiplas rules OR.
- `role=="admin"` bypass global hardcoded no engine (nunca listar admin rules; admin é transversal).
- **Fail-closed**: `NewABACEngine` retorna `ErrEmptyMatrix` se matriz vazia → startup aborta (evita A1 do legado "API sem autorização").

```go
type Rule struct {
    Name              string
    AllowedRoles      []string  // ex: {"syndic"}
    AllowedRoleTypes  []string  // ex: {"principal", "auxiliary"}
    MinPlanTier       string    // ex: "base"
    RequireTenantMatch bool
}
```

### 5.2 Decisão

```
Decide(abacCtx, action):
  if abacCtx.Role == "admin":
    return true                            # bypass

  rules := matrix[action]
  if len(rules) == 0:
    return false                           # default deny

  for rule in rules:
    if !checkRoles(rule, abacCtx) : continue
    if !checkRoleTypes(rule, abacCtx) : continue
    if rule.MinPlanTier != "" && !types.HasMinimumPlanTier(tier, rule.MinPlanTier) : continue
    if rule.RequireTenantMatch && !containsString(abacCtx.CondominiumIDs, abacCtx.ResourceTenantID) : continue
    return true                            # qualquer rule concedeu

  return false                             # default deny
```

### 5.3 Tenant isolation (fix Sprint 7.6 bug 1)

- `ABACContext.CondominiumIDs []string` substitui `TenantID` — o bug era que `TenantID=OrgZitadelID` nunca igual a `condominium_id` UUIDv7 do DB.
- 14+ rules com `RequireTenantMatch: true` agora funcionam em produção.
- `ICondominiumIDsProvider` em `shared/types/` → `CondominiumIDsProvider` em institutional → JOIN syndic_mandates + memberships para popular.

### 5.4 Testes ABAC

- `abac_engine_pbt_test.go` — **400 casos** rapid (admin_bypass + role + action + tenant).
- `abac_matrix_test.go` — **100+** sub-testes cobrindo happy paths, cross-tenant isolation, plan-tier enforcement, multi-condominium, operator vs administrator.

---

## 6. Webhooks (segurança 100% via HMAC)

Rotas `POST /webhooks/*` **não** passam por `Authenticate` — segurança é
verificação HMAC-SHA256 antes de qualquer parse.

### 6.1 Stripe

```go
// billing/infrastructure/providers/stripe_gateway.go
func (g *StripeGateway) VerifyWebhookSignature(payload []byte, sigHeader string) error {
    _, err := webhook.ConstructEvent(payload, sigHeader, g.webhookSecret)
    if err != nil { return ErrInvalidWebhookSignature }   // sentinel
    return nil
}
```

**Pentest 2026-04-21**: `/webhooks/stripe` rejeita 400 sem signature ou com signature forjada.

### 6.2 Mux

```go
// content/infrastructure/providers/mux_provider.go
func (m *MuxProvider) verifyMuxSignature(payload []byte, sigHeader string) error {
    // t=timestamp,v1=hex — HMAC-SHA256(secret, "t.payload")
    // também valida timestamp < 5 minutos (anti-replay)
}
```

**A-022 fechado 2026-04-22**: rota movida de `/api/v1/content/webhooks/mux`
(que seria barrada por Authenticate — Mux não envia Zitadel token) para
`/webhooks/mux` público.

---

## 7. Rate limiting

### 7.1 Impl atual (IP only)

```go
// middleware/rate_limiter.go
func RateLimiter(ctx context.Context, rpm, burst int) contracts.HandlerFunc {
    limiters := sync.Map{}   // ip → *rate.Limiter

    // Cleanup de IPs inativos > 2h (roda a cada 30 min)
    go func() {
        ticker := time.NewTicker(30 * time.Minute)
        for {
            select {
            case <-ctx.Done():   // A-032 — respeita lifecycle
                return
            case <-ticker.C:
                // limpa entries com lastSeen < now - 2h
            }
        }
    }()

    return func(c contracts.Context) {
        ip := c.ClientIP()
        l, _ := limiters.LoadOrStore(ip, rate.NewLimiter(rate.Limit(rpm/60.0), burst))
        if !l.(*rate.Limiter).Allow() {
            c.AbortWithJSON(429, /* rate_limited */)
        }
    }
}
```

Valores:

| Grupo | RPM | Burst | Config key |
|---|---|---|---|
| `/auth/*` | 20 | 5 | `AUTH_RATE_LIMIT_RPM` / `AUTH_RATE_LIMIT_BURST` |
| `/api/v1/*` | 60 | 10 | `RATE_LIMIT_RPM` / `RATE_LIMIT_BURST` |

### 7.2 Gap A-039

Spec `steering/security.md` §"Rate Limiting" exige:
- 300 req/min por userId em rotas protegidas (além do IP).
- IP corporativo NAT (escritório com 50 síndicos) não deve esgotar quota compartilhada.

Fix proposto: segundo tier `userLimiters sync.Map[userID]→limiter` aplicado **depois** de `Authenticate` via `RateLimitPerUser(userRpm, userBurst)`. Sprint 10.

---

## 8. Input validation

### 8.1 DecodeJSON obrigatório

`contracts.Context.DecodeJSON(dst any) error` retorna erro em:
- JSON malformado
- Campos de tipo incompatível

**F3**: zero handlers com `_ = ctx.DecodeJSON(...)`. Sempre `if err := ctx.DecodeJSON(&req); err != nil { ctx.SetError(...) ; return }`.

### 8.2 Guards pós-decode

```go
// event_handler.go (A-016 fix)
if req.StartsAt.IsZero() {
    ctx.SetError(apperrors.ErrValidation.WithMessage("starts_at obrigatório"))
    return
}
```

### 8.3 Domain entities validam em construtor

```go
func NewCondominium(...) (*Condominium, error) {
    if !publicIDPattern.MatchString(publicID) { return nil, ErrInvalidPublicID }
    if l := len(strings.TrimSpace(name)); l < 2 || l > 200 { return nil, ErrInvalidCondominiumName }
    // ...
}
```

Defesa em profundidade: DB CHECK + domain validation + request schema (OpenAPI).

### 8.4 Integer overflow (G115)

`pkg/safecast/` clampa `int → int32/int16` em [0, MaxIntN]. `gosec -severity=high` de **66 → 0 findings**.

---

## 9. Secrets

### 9.1 `.gitignore` protege (A-018 fechado)

```
.env
.env.*
zitadel-key*.json
[0-9]*[0-9].json       # service account keys numéricas
/migrate               # binário compilado
/zitadel-setup         # binário compilado
```

`git check-ignore zitadel-key.json` → retorna caminho (protegido).

### 9.2 Env-first em produção

Config via Viper:
- `.env` em dev
- Env vars do Railway em prod (`DATABASE_URL`, `REDIS_URL` fornecidos automaticamente quando Postgres/Redis vinculados)
- `config.Validate()` rejeita startup se:
  - `CORS_ALLOWED_ORIGINS` contém `*` em staging/prod (CVE de browser-side)
  - `STRIPE_SECRET_KEY` ausente em staging/prod
  - `ZITADEL_KEY_PATH` não-existente em staging/prod

### 9.3 Zitadel key injetada via entrypoint

```sh
# docker-entrypoint.sh
if [ -n "$ZITADEL_KEY_JSON" ]; then
    echo "$ZITADEL_KEY_JSON" > /tmp/zitadel-key.json
    chmod 600 /tmp/zitadel-key.json
    export ZITADEL_KEY_PATH=/tmp/zitadel-key.json
fi
exec "$@"
```

Permite passar conteúdo JSON como env var sem commitar arquivo.

### 9.4 Gates scan

- `gosec -severity=high -exclude-dir=generated -exclude-dir=cmd/zitadel-setup ./...` → **0 findings** (2026-04-22)
- `govulncheck ./...` → **No vulnerabilities found**
- Dependabot: 3 CVEs OTel 1.40→1.43 fechados em `d410547`

---

## 10. PII em logs (enforcement por grep)

### 10.1 Canônico

```go
logger.F("err", err)                 # zerolog aceita error direto; preserva stack; NUNCA err.Error()
logger.F("user_id", user.ID())        # UUIDv7 anônimo — OK
logger.F("condominium_id", cid)       # idem
logger.F("cnpj", cnpj.Masked())       # "**.***.***/****-**"
```

### 10.2 Proibido

```
logger.F("email", user.Email())
logger.F("cpf", cpf.Digits())
logger.F("token", tok)
logger.F("password", ...)
logger.F("cnpj", cnpj.Digits())       # A-004 fix
logger.F("invite_token", link.Token()) # A-003 fix
```

### 10.3 Grep de auditoria

```bash
grep -rn 'logger\.F("email\|logger\.F("cpf\|logger\.F("token\|logger\.F("password' internal/
# esperado: zero matches
```

A-003 e A-004 (invite_token e CNPJ em log) fechados em commit 104fec7.

---

## 11. CSRF / CORS

### 11.1 CORS (`middleware/cors.go`)

- Config `CORSConfig.AllowedOrigins` lista explícita (não wildcard em staging/prod).
- Whitelist típica: `https://cms.mastersindico.com.br`, `https://lms.mastersindico.com.br`, outros subdomínios.
- Preflight (OPTIONS) permite somente métodos e headers listados.
- `Access-Control-Allow-Credentials: true` apenas quando origem está na whitelist — cookies cross-subdomain só com `Origin` específico.

### 11.2 CSRF

- Cookies `SameSite=Lax` → mitigar CSRF em rotas de side-effect.
- Backend exige `Content-Type: application/json` em endpoints de mutação — bloqueia formulários HTML maliciosos.
- Pentest 2026-04-21: confirmado `application/x-www-form-urlencoded` rejeitado em `/api/v1/*`.

---

## 12. Webhook patterns — defesas em profundidade

### 12.1 Idempotency Stripe

- `evt.ID` é estável (`evt_XXX`) — handler pode aplicar o mesmo evento 2x sem efeito colateral.
- Exemplo: `customer.subscription.updated` → `Subscription.UpdateFromProvider(...)` é idempotente (substitui estado, não incrementa).
- **MarkCanceled não é idempotente**: retorna `ErrSubscriptionAlreadyCanceled` — handler checa antes de chamar.

### 12.2 UNIQUE + `ON CONFLICT DO UPDATE`

Para respostas, leituras, votos:
- `event_responses(event_id, user_id)` UNIQUE + UPSERT `ON CONFLICT DO UPDATE` — morador troca ciente↔confirmado sem criar linha nova.
- `announcement_reads(announcement_id, user_id)` UNIQUE + UPSERT — marca 2x = no-op.
- `votes(agenda_item_id, voter_id)` UNIQUE — double-vote barrado no DB; use case captura 23505 → `ErrConflict`.
- `supplier_vote_casts(session_id, voter_user_id)` idem + `IncrementVotesFor/Against/Abstentions` via SQL-side atomic.
- `company_evaluations(company_id, evaluator_user_id, condominium_id)` UNIQUE + `isUniqueViolation(err)` → `ErrConflict`.

### 12.3 Anti-replay timestamp

- Mux webhook rejeita se `|now - timestamp| > 5 minutes`.
- Stripe usa mecânica similar por default no SDK.

---

## 13. Multi-tenancy row-based

### 13.1 Enforcement

- Toda tabela institutional + commercial (exceto `companies` globais) + assembly tem `condominium_id` NOT NULL.
- Query pattern canônico:
  ```sql
  SELECT … WHERE condominium_id = $1 AND id = $2;
  ```
- `$1` vem **sempre** de `AuthUser.CondominiumIDs` (não do path/body) — impede IDOR.

### 13.2 Teste pentest (2026-04-21)

- Síndico A tenta `GET /api/v1/condominiums/CID-de-B` → 404 (isolamento correto).
- `RequireTenantMatch: true` + ID inexistente na lista → rule nega → 403.

### 13.3 CI grep

Proposta Sprint 10 (a adicionar): grep automatizado em PR
```bash
grep -rnE 'WHERE.*id.*=.*\$[0-9]' internal/modules/*/infrastructure/database/queries/
# confere se todas têm condominium_id no WHERE (manual hoje, automatizar)
```

---

## 14. Threat model STRIDE — snapshot

| Ameaça (STRIDE) | Mitigação atual | Abertos |
|---|---|---|
| **Spoofing** (auth bypass) | Zitadel introspection, 1-device, PKCE, token curto | DT-003 sem circuit breaker |
| **Tampering** (webhook forgery) | HMAC Stripe + Mux antes de parse | — |
| **Repudiation** | Audit trail LGPD (institutional) | DT-010: só compliance persiste — billing/commercial/identity usam zerolog |
| **Information disclosure** | Tenant isolation, PII never logged, CNPJ.Masked() | — |
| **Denial of service** | Rate limit (/auth e /api/v1 por IP) + timeouts HTTP Server (Read/Write/Idle/ReadHeader) | A-039 per-user; DT-003/A-032 goroutines |
| **Elevation of privilege** | ABAC default-deny; admin bypass isolado; role é do Zitadel (não modificável via API interna) | A-023 1-device change audit |

---

## 15. Vulnerabilidades históricas fechadas (2026-04-22)

| ID | Severidade | Fix |
|---|---|---|
| A-018 | high | `.gitignore` cobre `zitadel-key*.json` + binários |
| A-019 | high | Gosec 66→0 via `pkg/safecast` + `pagination.FromQuery` |
| A-022 | high | Mux webhook movido para rota pública (HMAC basta) |
| A-025 | critical | Assembly Vote TOCTOU: UNIQUE no DB + mapping 23505 → `ErrConflict` |
| A-026 | critical | Commercial Vote contador: UoW + `FindSessionByIDForUpdate` |
| A-027 | critical | UploadVideo Saga com `CancelUpload` compensation |
| A-028 | critical | StartLiveSession Saga com `EndRoom` compensation |
| A-029 | high | Company Evaluation: `isUniqueViolation` → `ErrConflict` |
| A-030 | high | Commercial Vote Session: `SELECT ... FOR UPDATE` |
| A-032 | medium | Rate limiter goroutine respeita `ctx.Done()` |
| A-033/A-034 | medium | Livekit retry backoff + reconciliação NotFound |
| A-003 | medium | invite_token removido de log (empresa_sindica_use_cases.go:53) |
| A-004 | medium | `CNPJ.Masked()` em logs |
| A-007 | medium | Audit trail LGPD — promovido a DT-010 Sprint 10 |
| A-006 | medium | Rate limiter cleanup via ticker 30min + TTL 2h |

---

## 16. Pendências (A-023 + A-039 + DT-003 + DT-010)

| ID | Sprint | Próximo passo |
|---|---|---|
| A-023 | 10 | Ler última session ativa antes de invalidar; comparar fingerprints; log `device_changed` + coluna `invalidation_reason` |
| A-024 | 10 | Elevar `RawBody() io.Reader` a método público em `contracts.Context` |
| A-039 | 10 | Tier 2 `RateLimitPerUser(userRpm=300, userBurst=30)` após Authenticate |
| DT-003 | 10 | `sony/gobreaker` em `zitadel_provider.go`; fallback cache stale até TTL 2x |
| DT-004 | 2+ | `context.WithTimeout(ctx, 30s)` em `UnitOfWork.Run`, configurável |
| DT-010 | 10 | `IAuditLogger` em `shared/types/`; adapter via `compliance_repository.CreateAuditLog`; DI em billing/commercial/identity |

---

## 17. Referências

- Auditoria vivo: `backend/.kiro/AUDIT.md`
- Steering security: `backend/.kiro/steering/security.md`
- Decisões: `backend/.kiro/STATE.md` (D-001..D-030)
- No vault v2: [[architecture]] · [[patterns]] · [[requirements]] · [[tasks]] · [[../../08-security/BEYOND_CORP]]
