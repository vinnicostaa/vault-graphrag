---
title: Aggregate — SessionState
type: spec
tags: [domain, ddd, aggregate, cross-domain, session, device-fingerprint, lifecycle, master-sindico, v2]
bc: cross-domain
source: 01-domain/invariants.md (INV-005..INV-010) + 01-domain/state-machines.md §7 + 04-requirements/functional/identity.md (IDN-009, IDN-010, IDN-032)
created: 2026-04-23
updated: 2026-04-23
---

# Aggregate — `SessionState`

**BC**: cross-domain (leitura transversal) · **Raiz**: ✅ (lifecycle transversal que une [[Session]] identity + device fingerprint + rate limit + ABAC cache)

Visão **transversal** do ciclo de vida de uma sessão: combina o aggregate [[Session]] (identity BC) com metadata cross-BC (device fingerprint, risk profile, rate limit bucket key, ABAC cache key, MFA state). Não duplica dados — linka. Usado por ABAC engine, rate limiter e webhooks Zitadel pra decisões consistentes.

Criado como fechamento do finding **A-DC-QA-040**. Diferente de [[Session]] (que é o aggregate raiz em identity) — este é o **view-model agregador cross-BC** usado por serviços transversais.

## Entidade raiz

```go
type SessionState struct {
    sessionID         SessionID              // FK → Session (identity BC)
    userID            UserID                 // denormalizado pra acesso rápido
    role              string                 // snapshot do momento do login
    planTier          string                 // snapshot
    isAdmin           bool                   // idem

    // Device / client
    deviceFingerprint DeviceFingerprint      // SHA-256 server-side (INV-009)
    ipAddress         string                 // último IP
    userAgent         string

    // Lifecycle
    createdAt         time.Time
    lastActivityAt    time.Time
    expiresAt         time.Time              // TTL
    state             SessionLifecycleState  // active | idle | revoked_other_device | logged_out | revoked | expired (alinha com [[../state-machines#7-user-session]])
    invalidationReason *string

    // Risk & MFA
    mfaActive         bool                   // sincronizado via webhook Zitadel user.mfa.enabled/disabled
    stepUpRequiredAt  *time.Time             // próxima vez que precisa re-MFA
    riskScore         float64                // 0-100 (device inconsistency, IP changes, velocity)

    // Cross-BC keys (derivados, denormalizados pra performance)
    abacCacheKeyPrefix string                // "ms:abac:{userID}:*"
    rateLimitKey      RateLimitKey           // pra middleware rate-limit coerente com session
}
```

## Value Objects

```go
type SessionLifecycleState string

const (
    SessionActive               SessionLifecycleState = "active"
    SessionIdle                 SessionLifecycleState = "idle"
    SessionRevokedOtherDevice   SessionLifecycleState = "revoked_other_device"
    SessionLoggedOut            SessionLifecycleState = "logged_out"
    SessionRevoked              SessionLifecycleState = "revoked"
    SessionExpired              SessionLifecycleState = "expired"
)

type RiskScore float64 // 0 = sem risco, 100 = muito arriscado
```

## Invariantes

- **INV-005**: ≤ MAX_ACTIVE_SESSIONS por User (default 1 — 1-device policy; BR-004)
- **INV-007**: `zitadelSessionID` nunca re-usado pós revogação (INSERT-only)
- **INV-009**: DeviceFingerprint SHA-256 server-side — cliente nunca envia hash direto
- **INV-010**: Zitadel sem role → middleware 403 sem UPSERT (fail-fast pre-Session)
- **INV-100**: Cookie httpOnly + Secure + SameSite=Lax (flags aplicadas via middleware)
- State machine: transições canônicas em [[../state-machines#7-user-session]] — transições proibidas enforceadas via factory
- `role`/`planTier` são **snapshot** do momento do login — mudanças pós-login invalidam session (force re-login ou step-up) via webhook Zitadel (`user.grant.added/removed`) → Redis `DEL ms:abac:{uid}:*` + marca `SessionLoggedOut`
- `stepUpRequiredAt` obrigatório pra operações sensíveis quando `riskScore > threshold` — ADR-0026 admin step-up M1

## Factories

```go
// NewSessionState — criada junto com Session em login flow
func NewSessionState(sid SessionID, uid UserID, role, planTier string, isAdmin bool, fp DeviceFingerprint, ip, ua string, ttl time.Duration, now time.Time) *SessionState

func ReconstructSessionState(...) *SessionState
```

## Métodos de domínio

- `Touch(now time.Time)` — atualiza `lastActivityAt`; transição `idle → active`
- `RequiresStepUp(action string, now time.Time) bool` — verifica se ação precisa re-MFA (admin action, refund, LGPD export, delegation revoke, etc)
- `UpdateRiskScore(signals []RiskSignal)` — recomputa score baseado em sinais (IP change, UA change, geo distance, velocity)
- `Revoke(reason string)` — transição para `revoked`
- `InvalidateForOtherDevice()` — INV-005 (1-device); transição `active/idle → revoked_other_device`
- `IsUsable(now time.Time) bool` — true se `state == active|idle` AND `NOW() < expiresAt`
- `Regenerate() (SessionState, error)` — rotate session ID pós mudança de auth state (A-DC-PEN-003 mitigação session fixation): INSERT nova session, invalida anterior, preserva userID/role

## Repository interface (Go)

```go
// domain/shared/session_state_view.go (view; não é repo que escreve)
type ISessionStateReader interface {
    LoadFromRequest(ctx context.Context, sessionID SessionID) (*SessionState, error)
    LoadByUserID(ctx context.Context, uid UserID) ([]*SessionState, error)  // admin AD5 list sessions
}

// Mutações vão no IsessionRepository (identity) — SessionState é view-model
```

## Composição

SessionState agrega (sem duplicar):
- [[Session]] (identity aggregate raiz) — id, state, timestamps, zitadelSessionID
- [[User]] role/planTier/isAdmin snapshot
- [[AbacDecision]] cache key prefix
- [[RateLimitEntry]] key
- Device fingerprint (VO compartilhado com User)

Construída pelo middleware `LoadSession` antes dos subsequentes: rate limit → ABAC → handler.

## Risk signals (exemplo)

```go
type RiskSignal struct {
    Type    string  // "ip_change" | "ua_change" | "geo_distance_km" | "velocity_rpm" | "device_reroute"
    Value   float64
    Weight  float64 // contribuição ao score
}
```

Ex: login de SP → 1min depois request de Tóquio → geo_distance_km = 18000 → RiskScore sobe → próxima operação sensível requer step-up MFA.

## Eventos emitidos

- `SessionStateRiskElevated` (E-???) — quando score cruza threshold → força step-up
- Consome: `SessionCreated` (E-004), `SessionInvalidated` (E-005), `DeviceChanged` (E-006), webhooks Zitadel user.mfa.enabled/disabled (A-DC-QA-050)

## Observability

- Métrica `session_risk_score_histogram{state}`
- Métrica `active_sessions_per_user` (gauge) — alerta se > MAX_ACTIVE_SESSIONS (INV-005 violation)
- Dashboard admin AD5: top N sessions by risk score, distribuição geográfica, device mix

## ⚠️ Pendências

- **P-SS-001**: regeneração de session ID (A-DC-PEN-003) em toda transição de auth state — definir lista canônica (login, step-up MFA, impersonate, role change).
- **P-SS-002**: risk model — pesos default de cada signal; revalidar com threat model real pós-M1.
- **P-SS-003**: session fixation — atualmente cookie `session_id` é do Zitadel; se atacante conseguir forçar cookie pré-login, regenerate resolve. Pen-tester confirmou gap; implementar antes de M1.

## Links

- [[../bounded-contexts#1-identity]] + [[../bounded-contexts#7-cross-domain]]
- [[../invariants#identity-inv-001-a-inv-010]] (INV-005..INV-010)
- [[../state-machines#7-user-session]] — 7 transições com test IDs
- [[../../04-requirements/functional/identity]] (IDN-009, IDN-010, IDN-032)
- [[Session]] — aggregate raiz identity
- [[User]] — role/planTier snapshot source
- [[AbacDecision]] — consumer desta estrutura
- [[RateLimitEntry]] — key derivada
- [[AuditEntry]] — mudanças relevantes (revoke, step-up, risk elevated) geram audit
