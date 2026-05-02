---
title: API Design — REST + OpenAPI 3.1 + WebSocket + Webhook
type: guide
tags: [architecture, api, rest, openapi, websocket, webhook, master-sindico, v2]
source: CLAUDE.md §3.11 + 13-research/instagram (GraphQL no) + 13-research/netflix (API Gateway)
created: 2026-04-23
updated: 2026-04-23
aliases: [api-contract, rest-openapi]
---

# API Design — Master Síndico

REST + OpenAPI 3.1 é o contrato canônico (§11 de [[../CLAUDE]]). GraphQL **NÃO** em M1/M2 ([[../13-research/instagram/_destilado]] §8).

> Versioning, envelope, pagination, webhook, WebSocket — padrões inegociáveis. Decisão: [[adr/0001-clean-architecture-ddd-cqrs]] + §3.11 do CLAUDE.

---

## 1. Princípios fundadores

1. **Contrato-first** — OpenAPI 3.1 spec existe **antes** do handler; handler é gerado ou conferido contra spec.
2. **Versioning explícito em URL** — `/api/v1/...`; breaking change vira `/v2/...` com deprecation 6 meses do v1.
3. **JSON-only** (content-type `application/json`); sem XML, sem form-urlencoded (exceto OAuth).
4. **TLS 1.3 + HSTS** obrigatórios.
5. **Idempotency-Key header** em todo POST crítico (pagamento, voto, publicação).
6. **HATEOAS opcional** — não dogmático; usar quando simplifica cliente (ex: `_links` em resource mutável).
7. **PII nunca em path/query** — CPF, CNPJ, email jamais em URL (vazam para logs de proxy).
8. **Rate limit headers** sempre devolvidos: `X-RateLimit-Limit`, `X-RateLimit-Remaining`, `X-RateLimit-Reset`.

---

## 2. Versionamento

### 2.1 Prefixo `/api/v1/`

- Todos endpoints públicos vivem sob `/api/v1/`.
- Admin back-office: `/admin/v1/` (separado; evita confusão com cliente).
- Webhooks inbound: `/webhook/<provider>` (sem versão — contrato do provider externo governa).

### 2.2 Breaking changes → `/api/v2/`

Critério para criar v2 (não é "mudança pequena" que merece v2):

- Mudança no shape de response em endpoint core.
- Remoção de campo.
- Renomeação com significado diferente.
- Mudança de semântica (ex: POST agora é idempotente).

### 2.3 Deprecation policy

- Ao lançar v2, v1 entra em **deprecation window de 6 meses**.
- Header `Deprecation: true` + `Sunset: <RFC 7231 date>` em toda response v1 após anúncio.
- Changelog público em `/api/docs/changelog`.
- Endpoint `/api/health` reporta versões ativas.

---

## 3. Envelope de response

### 3.1 Success envelope

```json
{
  "data": { ... },                      // payload principal (object ou array)
  "meta": {
    "request_id": "01HZ...ULID",
    "ts": "2026-04-23T14:00:00Z",
    "pagination": {                       // presente em endpoints list
      "next_cursor": "opaque-string",
      "has_more": true,
      "limit": 20
    }
  }
}
```

### 3.2 Error envelope

```json
{
  "error": {
    "code": "VALIDATION_ERROR",           // enum canônico
    "message": "Validation failed.",      // humano (pt-BR no future)
    "fields": [                            // opcional, só em 400 com erros por campo
      { "path": "cpf", "rule": "invalid_cpf" },
      { "path": "email", "rule": "email_already_exists" }
    ],
    "request_id": "01HZ...ULID",
    "docs_url": "https://docs.mastersindico.com.br/errors/VALIDATION_ERROR"
  }
}
```

### 3.3 Códigos de erro canônicos

| Code | HTTP | Significado |
|---|---|---|
| `VALIDATION_ERROR` | 400 | Payload inválido; `fields[]` com detalhes |
| `UNAUTHENTICATED` | 401 | Sem credenciais ou expiradas |
| `FORBIDDEN` | 403 | Autenticado mas sem permissão |
| `NOT_FOUND` | 404 | Recurso não existe (ou não pra você — tenant_id mismatch) |
| `CONFLICT` | 409 | UNIQUE constraint violada (voto duplicado, email já cadastrado) |
| `RATE_LIMITED` | 429 | Rate limit estourado; `Retry-After` header |
| `INTERNAL_ERROR` | 500 | Falha server-side; `request_id` para suporte |
| `SERVICE_UNAVAILABLE` | 503 | Circuit breaker aberto ou maintenance |
| `PAYMENT_REQUIRED` | 402 | Quota excedida, precisa upgrade plano |

---

## 4. HTTP verbs e semântica

| Verbo | Uso | Idempotente |
|---|---|---|
| `GET` | Leitura | ✅ |
| `POST` | Criação + ações | ❌ (usa `Idempotency-Key`) |
| `PUT` | Substituição full do recurso | ✅ |
| `PATCH` | Update parcial (JSON Merge Patch RFC 7396) | ❌ (usa `Idempotency-Key`) |
| `DELETE` | Remoção | ✅ (soft-delete ou 404 se já removido) |

**Timeline, atas, votos** — INSERT-only; não têm PUT/PATCH/DELETE.

---

## 5. Pagination — cursor opaco

### 5.1 Request

```
GET /api/v1/condominiums/:id/timeline?limit=20&cursor=eyJpZCI6IjAxSFo...In0
```

### 5.2 Response

```json
{
  "data": [ {...}, {...} ],
  "meta": {
    "pagination": {
      "next_cursor": "eyJpZCI6IjAxSFo...In0",
      "has_more": true,
      "limit": 20
    }
  }
}
```

### 5.3 Regras

- `limit` default 20, max 100.
- `cursor` é **opaco** (base64 JSON com `{id, ts}`). Cliente não decodifica.
- Sem offset-based pagination (performance ruim em tabelas grandes).
- Endpoints list obrigatoriamente cursor.
- Ordenação default: `created_at DESC` (timeline), `score DESC` (ranking Connect Me), `reputation DESC` (empresas).

---

## 6. Autenticação

### 6.1 Cookie httpOnly (web)

- Domain: `.mastersindico.com.br`
- `HttpOnly`, `Secure`, `SameSite=Lax`.
- Content: session ID opaco (16 bytes random); `sid → user_id + claims` em Redis.
- Renova via OIDC refresh token server-side.

### 6.2 Bearer token (mobile Flutter)

- `Authorization: Bearer <jwt>` (acesso curto 15min).
- Refresh via `/auth/refresh` → novo access + rotaciona refresh.
- Refresh bound a `device_fingerprint`.

### 6.3 OAuth Client Credentials (API pública agência — M2+)

- Agência obtém `client_id` + `client_secret` via onboarding.
- Token request em `/auth/oauth/token`.
- Scope limited: `read:timeline` + `post:announcement` (com consent do síndico).

---

## 7. Endpoints canônicos M1 (amostra)

```
# Identity
POST /api/v1/auth/register         (registro via OIDC callback)
POST /api/v1/auth/login            (Authorization Code PKCE)
POST /api/v1/auth/refresh
POST /api/v1/auth/logout
GET  /api/v1/me                    (session atual)
GET  /api/v1/me/devices
POST /api/v1/me/devices/:id/revoke

# Billing
POST /api/v1/billing/subscribe
GET  /api/v1/billing/subscription
GET  /api/v1/billing/invoices?cursor=...
POST /webhook/stripe                (inbound)

# Institutional
POST /api/v1/condominiums
GET  /api/v1/condominiums/:id
POST /api/v1/condominiums/:id/units
GET  /api/v1/condominiums/:id/timeline?cursor=...
POST /api/v1/condominiums/:id/announcements
POST /api/v1/condominiums/:id/plano-diretor

# Commercial (Connect Me)
POST /api/v1/companies                         (cadastro)
POST /api/v1/companies/:id/verify              (admin)
GET  /api/v1/companies?q=...&near_lat=&near_lng=&cursor=...
POST /api/v1/rfps
GET  /api/v1/rfps/:id
POST /api/v1/rfps/:id/proposals
POST /api/v1/rfps/:id/proposals/:pid/accept
POST /api/v1/contracts/:id/milestones/:mid/complete
POST /api/v1/contracts/:id/reviews

# Content
POST /api/v1/videos/upload-url                 (direct upload Mux)
POST /webhook/mux
GET  /api/v1/videos/:id/playback               (signed URL)
POST /api/v1/videos/:id/moderate               (admin ou auto)

# Assembly
POST /api/v1/assemblies
POST /api/v1/assemblies/:id/start
POST /api/v1/assemblies/:id/agenda/:itemId/vote
GET  /api/v1/assemblies/:id/quorum-live
POST /api/v1/assemblies/:id/end
POST /api/v1/assemblies/:id/minutes/publish
POST /webhook/livekit
```

---

## 8. WebSocket `/ws/...`

### 8.1 Uso

- Atualizações live em assembleias: quórum, votos tabulando, fila de fala.
- Feed Connect Me (M2+): novas propostas em RFP do síndico.

### 8.2 Auth

- Upgrade inicial carrega cookie/Bearer; WebSocket herda claims.
- Sem auth = 401 antes do upgrade.

### 8.3 Mensagens

```json
// Server → Client
{ "topic": "assembly.01HZ...", "event": "vote_cast", "data": { "agenda_item_id": "01HZ...", "option_counts": {"sim": 42, "nao": 18} } }
{ "topic": "assembly.01HZ...", "event": "quorum_update", "data": { "current_pct": 62.5 } }
```

- **Topic-based**: client subscribe ao topic (`assembly.{id}`, `user.{id}.notifications`).
- **Heartbeat** 30s; drop se ausente.
- **Reconnect** exponencial backoff client-side.

### 8.4 NÃO usar WebSocket para

- Fonte de verdade de voto — voto persistido em Postgres via POST HTTP. WebSocket apenas reflete. [[../13-research/google-meet/_destilado]] §4.

---

## 9. Webhook inbound (pattern canônico)

### 9.1 Handler pattern — DB UNIQUE primary, Redis L1 cache

**Reconciliação 2026-04-23**: pattern anterior usava Redis `SetNX` como **primary** idempotency. Finding [[../11-audit/audit-log/2026-04-23-pentest-specs#A-DC-PEN-016]] identificou que Redis é volátil e não durável — replay pós-crash = fraude financeira. Padronizado via [[adr/0027-webhook-idempotency-db]]: **DB UNIQUE `(provider, event_id)`** é fonte-de-verdade; Redis é cache L1 opcional. Alinha com [[../01-domain/invariants#INV-015]] e BIL-010.

```go
func (h *Webhook) Handle(c *gin.Context) {
    // 0) Body size guard (GR-036 + INV-102 — cobre A-DC-SEC-002)
    r := http.MaxBytesReader(c.Writer, c.Request.Body, 64*1024)
    body, err := io.ReadAll(r)
    if err != nil { c.Status(http.StatusRequestEntityTooLarge); return }

    // 1) HMAC verificado ANTES de parse (GR-008 + INV-093)
    if !verifyHMAC(body, c.GetHeader(hmacHeader), h.secret) {
        c.Status(http.StatusBadRequest); return
    }
    // 2) Parse
    var evt ProviderEvent
    if err := json.Unmarshal(body, &evt); err != nil {
        c.Status(http.StatusBadRequest); return
    }
    payloadHash := sha256Hex(body)

    // 3) Redis fast-path (L1 cache — opcional, correção não depende)
    cacheKey := fmt.Sprintf("ms:webhook-dedup:%s:%s", h.provider, evt.ID)
    if cached, err := h.redis.Get(c, cacheKey).Result(); err == nil && cached == payloadHash {
        c.Status(http.StatusOK); return // hit: já processado
    }

    // 4) DB UNIQUE = fonte-de-verdade (INV-105)
    err := h.repo.InsertEventOrSkip(c, WebhookEvent{
        Provider:    h.provider,
        EventID:     evt.ID,
        EventType:   evt.Type,
        PayloadHash: payloadHash,
    })
    switch {
    case errors.Is(err, ErrConflict):
        _ = h.redis.Set(c, cacheKey, payloadHash, 24*time.Hour).Err()
        c.Status(http.StatusOK); return // idempotência kick in
    case err != nil:
        c.Status(http.StatusInternalServerError); return
    }

    // 5) Soft-flag LGPD check (INV-104 — cobre A-DC-SEC-004)
    if evt.UserID != nil {
        pendingDelete, _ := h.repo.IsUserPendingHardDelete(c, *evt.UserID)
        if pendingDelete {
            _ = h.repo.LogSkippedWebhook(c, evt, "user_pending_hard_delete")
            _ = h.repo.MarkProcessed(c, evt.ID)
            c.Status(http.StatusOK); return
        }
    }

    // 6) Processa dentro de UoW (inclui audit + effects)
    if err := h.cmd.Handle(c, ProcessWebhookCommand{Provider: h.provider, Event: evt}); err != nil {
        h.repo.MarkFailed(c, evt.ID, err.Error())
        c.Status(http.StatusInternalServerError); return
    }

    _ = h.repo.MarkProcessed(c, evt.ID)
    _ = h.redis.Set(c, cacheKey, payloadHash, 24*time.Hour).Err()
    c.Status(http.StatusOK)
}
```

**Regras inegociáveis**:

1. **DB UNIQUE `(provider, event_id)` sempre composta** — nunca só `event_id` (previne colisão cross-provider — [[../11-audit/audit-log/2026-04-23-pentest-specs#A-DC-PEN-018]]).
2. **MaxBytesReader 64KB** em todo webhook (GR-036).
3. **HMAC antes de parse** (GR-008).
4. **Janela timestamp HMAC 2 min** (reduzida de 5 min — [[../11-audit/audit-log/2026-04-23-pentest-specs#A-DC-PEN-017]]).
5. **Soft-flag `users._pending_hard_delete`** consultada antes de UPDATE user-scoped (INV-104).
6. **Redis é cache opcional** — degraded mode (Redis down) mantém correção; latência sobe ~5ms.

### 9.2 Providers suportados (M1)

- `/webhook/stripe` — HMAC via `Stripe-Signature`.
- `/webhook/mux` — HMAC via `Mux-Signature`.
- `/webhook/livekit` — HMAC via `Authorization: Bearer <signed_payload>`.
- `/webhook/zitadel` — HMAC via `X-Zitadel-Signature` (Actions).

### 9.3 Webhook outbound (M2+)

- Master Síndico emite webhooks para clientes enterprise (agências, administradoras).
- Assinatura HMAC com segredo compartilhado.
- Retry exponencial + DLQ.
- Endpoint `/api/v1/webhooks` para cliente configurar URL + eventos subscritos.

---

## 10. Rate limiting

### 10.1 Tiers

| Escopo | Limite | Burst | Algoritmo |
|---|---|---|---|
| `/auth/*` | 20 rpm | 5 | Token bucket por IP |
| `/api/v1/*` | 60 rpm | 10 | Token bucket por `user_id` |
| `/webhook/*` | 1000 rpm | 100 | Token bucket por IP origem do provider |
| `/admin/v1/*` | 120 rpm | 20 | Token bucket por `admin_user_id` |

### 10.2 Headers devolvidos

```
X-RateLimit-Limit: 60
X-RateLimit-Remaining: 43
X-RateLimit-Reset: 1714838400
```

### 10.3 Enforcement

- Redis `INCR + EXPIRE` com cleanup (resolvido histórico A-006, A-032).
- 429 + `Retry-After: <seconds>` quando exceder.

---

## 11. OpenAPI 3.1 spec — estrutura

```
api/openapi/
├── openapi.yaml                  — entrypoint
├── paths/
│   ├── identity.yaml
│   ├── billing.yaml
│   ├── institutional.yaml
│   ├── commercial.yaml
│   ├── content.yaml
│   └── assembly.yaml
├── components/
│   ├── schemas/                   — shared DTOs
│   ├── parameters/                — pagination, filters comuns
│   ├── responses/                 — error envelope padrão
│   └── securitySchemes/           — cookie, bearer, oauth
└── examples/                      — examples para docs
```

- **Tooling**: `oapi-codegen` gera tipos Go; Redoc gera docs.
- **CI**: `spectral lint` + `openapi-diff` contra main (breaking change vira PR `/v2`).

---

## 11.5 CORS e cookies (hardening 2026-04-23)

Reforço pós-[[../11-audit/dual-check-log/2026-04-23-security-gaps#A-DC-SEC-003]] (🔴 M1 bloqueante — CORS credentials + subdomain wildcard). Codifica GR-020 no nível API:

**Allowlist canônica M1/M2** (estrita, sem wildcard subdomain):

| Env | Allowlist |
|---|---|
| `prod` | `https://app.mastersindico.com.br`, `https://admin.mastersindico.com.br` |
| `staging` | `https://staging-app.mastersindico.com.br`, `https://staging-admin.mastersindico.com.br` |
| `dev` | `http://localhost:3000`, `http://localhost:3001` (wildcard opcional via env) |

**Regras**:

1. **Wildcard subdomain** (`*.mastersindico.com.br`) **proibido em staging/prod** — fail-fast no `Config.Validate()` se detectado em env.
2. **Cookie session** setado com `Domain=app.mastersindico.com.br` (sem ponto-inicial) — evita compartilhamento com siblings (user-upload.mastersindico.com.br, CDN aliases).
3. **Cookie admin** setado com `Domain=admin.mastersindico.com.br` separado — isolation entre console admin e app usuário.
4. **`__Host-` prefix** quando possível (elimina `Domain` atributo completamente — limita cookie ao exato host que o setou).
5. **Pre-flight `OPTIONS`** valida `Origin` exato contra allowlist; qualquer mismatch → response sem headers CORS.
6. **Teste integration**: `Origin: evil.mastersindico.com.br` → pre-flight rejeitado.

---

## 12. Anti-patterns explícitos

- ❌ **SOAP / XML** — banido.
- ❌ **GraphQL** em M1/M2 — ver [[../13-research/instagram/_destilado]] §8.
- ❌ **PII em query string** — CPF/CNPJ/email nunca.
- ❌ **Offset pagination** — não escala.
- ❌ **Verbos em URL** — `/api/v1/getUsers`, `/api/v1/createCondominium`. Usar REST.
- ❌ **Trailing slash inconsistente** — decidir: sem trailing slash (redirect 301 se vier).
- ❌ **Erro sem `request_id`** — suporte cego.
- ❌ **Webhook sem HMAC** — vetor CSRF/spoofing.
- ❌ **Webhook sem MaxBytesReader 64KB** — vetor DoS ([[../11-audit/dual-check-log/2026-04-23-security-gaps#A-DC-SEC-002]]).
- ❌ **Webhook idempotência Redis-only** — replay pós-crash ([[../11-audit/audit-log/2026-04-23-pentest-specs#A-DC-PEN-016]]).
- ❌ **CORS wildcard subdomain em staging/prod** — cookie theft cross-subdomain ([[../11-audit/dual-check-log/2026-04-23-security-gaps#A-DC-SEC-003]]).
- ❌ **WebSocket como fonte de verdade** — [[../13-research/google-meet/_destilado]] §4.
- ❌ **`role`/`is_admin`/`tenant_id` em body de PATCH** — mass assignment ([[../11-audit/audit-log/2026-04-23-pentest-specs#A-DC-PEN-010]] + [[../11-audit/audit-log/2026-04-23-pentest-specs#A-DC-PEN-012]]).

---

## 13. ⚠️ Pendências

- **OAuth Client Credentials scopes** para agências — definir enum exato em M2.
- **JSON:API** vs envelope custom — decidido envelope custom; reavaliar se cliente pedir JSON:API.
- **GraphQL BFF** — M3+ só se 3+ clientes com shapes muito diferentes.

---

## 14. Vizinhos

- [[clean-arch-mapping]]
- [[c4-components]]
- [[patterns]] §11 (Idempotency)
- [[event-driven]] — outbox dispara webhook outbound
- [[adr/0001-clean-architecture-ddd-cqrs]]
- [[adr/0002-gin-abstracted-router]]
