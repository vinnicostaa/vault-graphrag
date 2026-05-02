---
title: Aggregate — AbacDecision
type: spec
tags: [domain, ddd, aggregate, cross-domain, abac, cache, security, master-sindico, v2]
bc: cross-domain
source: 01-domain/invariants.md (INV-086..INV-089) + 04-requirements/functional/cross-domain.md (XD-004, XD-007) + ADR 0029
created: 2026-04-23
updated: 2026-04-23
---

# Aggregate — `AbacDecision`

**BC**: cross-domain · **Raiz**: ✅ (cache entry + outcome auditável)

Decisão canônica de ABAC (Attribute-Based Access Control) em resposta a uma tentativa de ação. Persistida em cache Redis TTL curto + eventualmente loggada em `AuditEntry` quando é `allow` bypass ou `deny` sensível. Implementa a ordem estrita INV-086: `admin_bypass → action_configured → role_allowed → tenant_matches → plan_tier_sufficient`.

Criado como fechamento do finding **A-DC-QA-040**.

## Entidade raiz

```go
type AbacDecision struct {
    id              DecisionID         // UUIDv7 — só quando persistido em AuditEntry
    subject         AbacSubject        // quem (user_id, role, plan_tier, session, is_admin)
    action          string             // ex: "institutional.timeline.create"
    resource        AbacResource       // tipo + id + tenant_id
    outcome         AbacOutcome        // allow | deny
    reason          AbacReason         // enum: admin_bypass, action_configured, role_allowed, tenant_matches, plan_tier_sufficient, default_deny, tenant_mismatch, plan_required, role_forbidden, action_not_configured
    evaluatedAt     time.Time
    cacheKey        string             // "ms:abac:{userID}:{action}:{resourceID}"
    cachedUntil     *time.Time         // TTL expiration — Redis
    metadata        AbacMetadata       // trace_id, ip, ua, request_id
}
```

## Value Objects

```go
type AbacSubject struct {
    UserID       string
    Role         string    // role canônico (syndic, resident, enterprise, marketing, local_company, admin)
    PlanTier     string    // n1 | n2 | n3 | plus | pro | base | pagante | ...
    IsAdmin      bool
    IsImpersonating bool   // admin atuando como outro user
    SessionID    string
    DeviceFingerprint string
}

type AbacResource struct {
    Type      string  // "communication" | "vote" | "connect_me_request" | ...
    ID        string  // id do recurso alvo
    TenantID  string  // condominium_id | company_id (contexto de isolamento — INV-089)
    OwnerID   *string // user_id dono, quando aplicável (IDOR check)
}

type AbacOutcome string // "allow" | "deny"
type AbacReason string  // enum (ver campo `reason`)
type DecisionID string  // UUIDv7
type AbacMetadata struct {
    TraceID    string
    IP         string
    UserAgent  string
    RequestID  string
}
```

## Invariantes

- **INV-086**: Ordem estrita de avaliação (`admin_bypass → action_configured → role_allowed → tenant_matches → plan_tier_sufficient`) — garantido pela engine sequencial
- **INV-087**: Default-deny — ação não configurada → `Deny(action_not_configured)` (nunca `allow` implícito)
- **INV-088**: Admin bypass emite `AuditEntry{action:'admin_bypass',...}` sempre que usa
- **INV-089**: Tenant isolation — decisão negada se `subject.tenant != resource.tenantID` salvo admin bypass explícito
- **404 (não 403) em cross-tenant** (Gap-SEC confirmada, STATE): a API retorna 404 pra evitar enumeração, mas a decisão interna é `deny(tenant_mismatch)` — logged pra dashboard (INV-089)
- Cache key determinístico: `ms:abac:{userID}:{action}:{resourceID}` — invalidação por mudança de role/plan/tenant via webhooks (Zitadel, Stripe)

## Factories

```go
// NewAbacDecision — engine retorna após avaliar
func NewAbacDecision(sub AbacSubject, action string, res AbacResource, outcome AbacOutcome, reason AbacReason, md AbacMetadata) *AbacDecision {
    // validações mínimas: action != "", outcome em {allow, deny}
}

// ReconstructAbacDecision — quando lido do cache Redis ou AuditEntry
func ReconstructAbacDecision(...) *AbacDecision
```

## Métodos de domínio

- `IsAllowed() bool` — helper
- `RequiresAudit() bool` — `true` se `reason == admin_bypass` OR `outcome == deny && action in sensitive_list`
- `CacheKey() string` — retorna key canônica Redis
- `ShouldCache() bool` — `true` exceto pra `admin_bypass` (sempre re-avaliar) e outcomes com `deny(action_not_configured)` (bug-finding, não cachear)

## Repository interface (Go)

```go
// domain/shared/abac_decision_repository.go
type IAbacDecisionCache interface {
    Get(ctx context.Context, key string) (*AbacDecision, bool)     // Redis GET; false = miss
    Set(ctx context.Context, d *AbacDecision, ttl time.Duration) error // Redis SETEX
    Invalidate(ctx context.Context, userID string) error           // DEL pattern ms:abac:{userID}:*
    InvalidateByTenant(ctx context.Context, tenantID string) error // quando tenant muda (rare)
    InvalidateByAction(ctx context.Context, action string) error   // quando policy muda (admin edit)
}
```

**Uso crítico**: cache invalidado em handlers de:
- `billing.subscription.changed` (webhook Stripe) → invalida user
- `zitadel.user.grant.added/removed` → invalida user (ver [[../../90-ingested/_consolidado-providers-fluxos#fluxos-inbound-webhooks-zitadel-a-dc-qa-050-expandido]])
- admin edita policy na tela AD4 ABAC matrix override → invalida `action`
- user muda de tenant ativo (contexto) → invalida user

## Eventos emitidos

- **Não emite eventos próprios** (decisão é stateless).
- Efeito colateral **obrigatório quando `RequiresAudit()`**: cria `AuditEntry` (ver [[AuditEntry]]).
- Efeito colateral em métricas (Prometheus): `abac_decisions_total{outcome, reason, action}` — INV-089 dashboard alimenta de `tenant_mismatch` counter.

## Performance

- Target p95: < 5ms por decisão (INV-086 PBT com 400 casos ~22ms / 400 ≈ 55µs por decisão em teste; produção tem cache miss penalty).
- Cache hit rate esperado: > 90% em steady-state (chaves com TTL 5min + invalidação surgical).

## ⚠️ Pendências

- **P-ABAC-001**: admin bypass **deve** sempre emitir AuditEntry (INV-088); confirmar se é síncrono (dentro da chamada ABAC) ou via evento async (risco de perder audit se crash).
- **P-ABAC-002**: decisões `deny` sensíveis → qual lista? Proposta: `admin.*`, `billing.refund`, `lgpd.*`, `content.video.remove` pós-90d.
- **P-ABAC-003**: TTL default 5min; discutir caso especial para operações LGPD (forçar re-avaliar sempre).

## Links

- [[../bounded-contexts#7-cross-domain]]
- [[../invariants#cross-domain-inv-086-a-inv-100]] (INV-086..INV-089)
- [[../../04-requirements/functional/cross-domain]] (XD-004, XD-007)
- [[AuditEntry]] — decisões que requerem audit
- [[../../02-architecture/c4-components]] §7 ABAC engine
- [[../../09-testing/pbt]] — 400 casos PBT INV-086
