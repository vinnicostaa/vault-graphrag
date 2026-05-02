---
title: Aggregate — Session
type: spec
tags: [domain, ddd, aggregate, identity, master-sindico, v2]
bc: identity
source: 90-ingested/.../specs/requirements/identity.md
created: 2026-04-23
updated: 2026-04-23
---

# Aggregate — `Session`

**BC**: identity · **Raiz**: ✅ (ciclo de vida próprio; 1-device policy)

Sessão ativa de um User em um device. Rege enforcement BR-004 (1-device). Criada pós callback Zitadel; invalidada em logout, novo device, admin revoke, ou Zitadel revoga.

## Entidade raiz

```go
type Session struct {
    id                  SessionID               // UUIDv7
    userID              UserID
    zitadelSessionID    string
    deviceFingerprint   DeviceFingerprint
    ipAddress           string
    userAgent           string
    state               SessionState            // active | idle | revoked_other_device | logged_out | revoked | expired
    createdAt           time.Time
    lastActivityAt      time.Time
    expiresAt           time.Time               // TTL configurável
    invalidatedAt       *time.Time
    invalidationReason  *string
}
```

## Value Objects

- `SessionID`, `SessionState` (enum)
- `DeviceFingerprint` (compartilhado com `User`)

## Invariantes

- **INV-005**: ≤ `MAX_ACTIVE_SESSIONS` ativa por User (default 1)
- **INV-006**: FK `session.user_id` existe em `users`
- **INV-007**: `zitadel_session_id` nunca re-usado pós revogação (INSERT only)

## Factories

```go
func NewSession(userID UserID, zitadelSID string, fp DeviceFingerprint, ip, ua string, ttl time.Duration) (*Session, error) {
    // valida: userID != zero; fp não vazia; ttl > 0
}

func ReconstructSession(...) *Session
```

## Métodos de domínio

- `Touch()` — atualiza `lastActivityAt`
- `InvalidateForOtherDevice()` — `active/idle → revoked_other_device`
- `Logout()` — `active/idle → logged_out`
- `Revoke(reason string)` — `active/idle → revoked`
- `IsExpired(now time.Time) bool` — `NOW() >= expires_at`

## Repository interface (Go)

```go
type ISessionRepository interface {
    Save(ctx context.Context, s *Session) error
    FindByID(ctx context.Context, id SessionID) (*Session, error)
    FindActiveByUserID(ctx context.Context, uid UserID) ([]*Session, error)
    InvalidatePreviousByUserID(ctx context.Context, uid UserID, reason string) (invalidatedCount int, err error)
    InvalidateByZitadelSessionID(ctx context.Context, zSID string, reason string) error
}
```

**Uso crítico**: `InvalidatePreviousByUserID` chamado **dentro** de `UnitOfWork.Run(ctx, fn)` antes de criar nova Session, garantindo atomicidade 1-device (INV-005; A-023 legado fix).

## Eventos emitidos

- `SessionCreated` (E-004)
- `SessionInvalidated` (E-005)
- `DeviceChanged` (E-006) — quando fingerprint muda em novo login

## Links

- [[../bounded-contexts#1-identity]]
- [[../invariants#identity-inv-001-a-inv-010]]
- [[../state-machines#7-user-session]]
- [[User]]
