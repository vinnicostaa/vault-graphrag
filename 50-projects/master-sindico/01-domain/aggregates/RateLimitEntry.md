---
title: Aggregate — RateLimitEntry
type: spec
tags: [domain, ddd, aggregate, cross-domain, rate-limit, token-bucket, throttle, master-sindico, v2]
bc: cross-domain
source: 01-domain/invariants.md (INV-097) + 04-requirements/functional/cross-domain.md (XD-014) + AUDIT A-006, A-032 legado
created: 2026-04-23
updated: 2026-04-23
---

# Aggregate — `RateLimitEntry`

**BC**: cross-domain · **Raiz**: ✅ (estado de token bucket persistido em Redis + fallback memory)

Estado de um token bucket por par `(user_id | IP, scope)`. Middleware `RateLimiter` consulta+decrementa átomo via Lua script Redis. Fallback in-memory (sync.Map) com cleanup goroutine pra evitar memory leak (A-032 legado fix).

Criado como fechamento do finding **A-DC-QA-040**.

## Entidade raiz

```go
type RateLimitEntry struct {
    key             RateLimitKey    // VO composta
    tokens          float64         // tokens atuais no bucket (decimal pra preserv. precisão)
    capacity        int             // max tokens
    refillRate      float64         // tokens/segundo
    lastRefillAt    time.Time       // ts do último refill
    windowStart     time.Time       // início da janela (pra rate limiters sliding-window)
    firstSeenAt     time.Time       // observability
    blockedCount    int             // contagem cumulativa de requests bloqueados (alerta)
    ttl             time.Duration   // Redis EXPIRE — auto-cleanup
}
```

## Value Objects

```go
type RateLimitKey struct {
    Subject   string  // "user:<uid>" | "ip:<ip>" | "session:<sid>"
    Scope     string  // "/auth/*" | "/api/v1/*" | "webhook:stripe" | "connect_me:publish" | ...
}

type RateLimitDecision struct {
    Allowed       bool
    Remaining     int          // tokens restantes pós-decrement
    RetryAfter    time.Duration // quando pode tentar de novo
    Reason        string       // "exceeded" | "allowed" | "burst_exceeded"
}
```

## Invariantes

- **INV-097**: `/auth/*` 20rpm burst 5; `/api/v1/*` 60rpm burst 10 (A-006, A-032 legado; default baselines)
- `tokens ∈ [0, capacity]` sempre (CHECK + atomic update)
- `refillRate` é constante por scope; ajustável via config (YAML) — admin pode sobrescrever em AD4 pra scope específico
- Cleanup goroutine **obrigatória** no fallback memory (A-032 legado) — sem ela, `sync.Map` cresce indefinidamente
- Atomic update via Lua script (Redis) — lê+atualiza numa operação só pra evitar race

## Scopes canônicos (defaults)

| Scope | Capacity | Refill rate | Observação |
|---|---|---|---|
| `/auth/*` | 5 | 20/min (0.333/s) | Anti brute-force login/signup |
| `/api/v1/*` | 10 | 60/min (1/s) | API geral per-user |
| `webhook:stripe` | 100 | 100/min | Por IP Stripe (whitelist IPs) |
| `webhook:mux` | 100 | 100/min | idem |
| `connect_me:publish` | 5 | 1/hora | Anti-spam Connect Me (BIL-016 tem quota separada) |
| `otp:send` | 3 | 1/10min | Twilio SMS anti-abuse |
| `coupon:generate` | 1 | 1/4h | Cooldown cupom (INV-051 — mas é por (user, partner) — tabela dedicada) |

## Factories

```go
// NewRateLimitEntry — criado sob demanda no primeiro request
func NewRateLimitEntry(key RateLimitKey, capacity int, refillRate float64, now time.Time, ttl time.Duration) *RateLimitEntry

func ReconstructRateLimitEntry(...) *RateLimitEntry
```

## Métodos de domínio

- `Refill(now time.Time)` — atualiza `tokens = min(capacity, tokens + (now - lastRefillAt) * refillRate)`
- `TryConsume(n int, now time.Time) RateLimitDecision` — `Refill` + decrementa se `tokens >= n`; senão retorna `Allowed=false` + `RetryAfter`
- `IncrementBlocked()` — métrica

## Repository interface (Go)

```go
// domain/shared/rate_limit_repository.go
type IRateLimitStore interface {
    // Atomic: Refill + TryConsume dentro de Lua script (Redis) ou lock local (memory)
    TryConsume(ctx context.Context, key RateLimitKey, n int, capacity int, refillRate float64) (RateLimitDecision, error)

    // Admin / observability
    Get(ctx context.Context, key RateLimitKey) (*RateLimitEntry, error)
    Reset(ctx context.Context, key RateLimitKey) error          // admin reset manual
    ListBlocked(ctx context.Context, scope string, page PageArgs) ([]RateLimitEntry, error) // dashboard top abusers
}
```

## Implementações

### Redis (produção)

Lua script atômico:
```lua
-- KEYS[1] = bucket_key
-- ARGV[1] = capacity, ARGV[2] = refill_rate, ARGV[3] = now_ms, ARGV[4] = cost
local bucket = redis.call("HGETALL", KEYS[1])
local tokens = tonumber(bucket.tokens or ARGV[1])
local last = tonumber(bucket.last or ARGV[3])
local elapsed = (ARGV[3] - last) / 1000
tokens = math.min(ARGV[1], tokens + elapsed * ARGV[2])
if tokens >= ARGV[4] then
  tokens = tokens - ARGV[4]
  redis.call("HSET", KEYS[1], "tokens", tokens, "last", ARGV[3])
  redis.call("EXPIRE", KEYS[1], 3600)
  return {1, tokens}
else
  redis.call("HSET", KEYS[1], "tokens", tokens, "last", ARGV[3])
  redis.call("EXPIRE", KEYS[1], 3600)
  return {0, tokens}
end
```

### Memory (fallback / dev)

`sync.Map[RateLimitKey]*RateLimitEntry` + cleanup goroutine a cada 1min remove keys com `lastRefillAt < NOW - ttl`.

## Eventos emitidos

Não emite events. Emite métricas:
- `rate_limit_checked_total{scope, allowed}`
- `rate_limit_blocked_total{scope, subject_type}`
- `rate_limit_bucket_size` (gauge)

Quando `blockedCount > threshold` por `subject` → alerta Sentry + potencialmente bloqueio estendido (anti-abuse automático).

## Observability

- Dashboard Grafana: top 10 blocked per scope last hour.
- Admin AD4 ABAC/RateLimit override: permite resetar bucket manual ou ajustar scope config temporário.
- Log rate-limit por user (A-DC-SEC-005 — Pentester pass log rate limit per user recomendado como melhoria).

## ⚠️ Pendências

- **P-RL-001**: distribuir rate limit entre réplicas API — Redis centralizado resolve; confirmar que staging/prod têm Redis com alta disponibilidade (sentinel/cluster).
- **P-RL-002**: scope `/auth/*` é por IP ou por email tentado? Proposta: ambos (login tenta como IP+email combinados pra evitar credential stuffing).
- **P-RL-003**: webhook scopes com whitelist IP — manter? Ou validar via mTLS? Default atual: whitelist IP Stripe/Mux.

## Links

- [[../bounded-contexts#7-cross-domain]]
- [[../invariants#cross-domain-inv-086-a-inv-100]] (INV-097)
- [[../../04-requirements/functional/cross-domain]] (XD-014)
- [[SessionState]] — rate limit combina com session state pra detectar bot
- [[AbacDecision]] — rate limit é primeira camada; ABAC é segunda
- [[../../11-audit/AUDIT]] A-006, A-032 (legado memory leak fix)
