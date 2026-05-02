---
title: Aggregate — User
type: spec
tags: [domain, ddd, aggregate, identity, master-sindico, v2]
bc: identity
source: 90-ingested/.../specs/requirements/identity.md + 01-domain/bounded-contexts.md
created: 2026-04-23
updated: 2026-04-23
---

# Aggregate — `User`

**BC**: identity · **Raiz**: ✅ · **Status legado**: produção

Identidade única por CPF (PF) ou CNPJ (PJ). **Mirror local** do Zitadel (UPSERT no primeiro login via callback OIDC). Um User pode operar em múltiplos tenants via `Membership` (em `institutional`).

## Entidade raiz

```go
type User struct {
    id              UserID              // UUIDv7
    zitadelID       string              // UNIQUE; sync chave
    email           Email               // VO
    name            string
    phone           Phone               // VO
    document        Document            // CPF | CNPJ (VO union)
    role            Role                // syndic|resident|enterprise|marketing|local_company|admin
    roleType        RoleType            // sub-contexto operacional (só local; T8)
    status          UserStatus          // pending_verification | active | suspended | deleted
    deviceFingerprint DeviceFingerprint // VO (SHA-256 hash)
    lastIP          string
    banned          bool
    banExpiresAt    *time.Time
    createdAt       time.Time
    updatedAt       time.Time
    deletedAt       *time.Time          // soft delete (LGPD SLA 15d → hard delete)
}
```

## Value Objects

- `Email` — RFC 5322 lowercase imutável
- `CPF` — 11 dígitos + verificador; `Masked()` pra log
- `CNPJ` — 14 dígitos + verificador; `Masked()` pra log
- `Phone` — E.164 BR
- `DeviceFingerprint` — SHA-256 server-side (IR-011; nunca reverter)
- `UserID`, `UserStatus`, `Role`, `RoleType` (enums)

## Invariantes

- **INV-001**: `zitadel_id` UNIQUE (1 User MS = 1 User Zitadel)
- **INV-002**: 1 CPF = 1 User (quando PF)
- **INV-003**: 1 CNPJ = 1 User (quando PJ)
- **INV-004**: `role` nunca string vazia (403 `NO_ROLE_ASSIGNED` antes do UPSERT)
- **INV-008**: Password **nunca** armazenado plain (delegado Zitadel Argon2)
- **INV-009**: DeviceFingerprint hash server-side SHA-256

## Factories

```go
// NewUser cria User novo com todas as validações (primeiro login pós Zitadel)
func NewUser(zitadelID string, email Email, name string, role Role, document Document, phone Phone) (*User, error) {
    // valida: role != "", document formato, email lowercase
}

// ReconstructUser reconstitui do repo (sem re-validar; assume dados consistentes)
func ReconstructUser(id UserID, zitadelID string, email Email, ..., createdAt time.Time, ...) *User
```

## Métodos de domínio

- `Activate()` — `pending_verification → active`
- `Suspend(reason string)` — `active → suspended`
- `SoftDelete()` — seta `deleted_at`; handler job hard-delete após 15d (LGPD BR-053)
- `UpdateDevice(fp DeviceFingerprint, ip string)` — atualiza device; dispara `DeviceChanged` event se mudou
- `Ban(until *time.Time)` / `Unban()` — tempor. ou permanente

## Repository interface (Go)

```go
// domain/identity/user_repository.go
type IUserRepository interface {
    Save(ctx context.Context, u *User) error                       // UPSERT por zitadel_id
    FindByID(ctx context.Context, id UserID) (*User, error)        // ErrNotFound
    FindByZitadelID(ctx context.Context, zID string) (*User, error)
    FindByEmail(ctx context.Context, email Email) (*User, error)
    FindByDocument(ctx context.Context, doc Document) (*User, error)
    SoftDelete(ctx context.Context, id UserID) error
}
```

## Eventos emitidos

- `UserCreated` (E-001) — pós UPSERT inicial
- `UserUpdated` (E-002) — dados alterados
- `UserDeleted` (E-003) — hard delete pós-LGPD

## Links

- [[../bounded-contexts#1-identity]]
- [[../invariants#identity-inv-001-a-inv-010]]
- [[../domain-events#identity]]
- [[../state-machines#7-user-session]]
- [[Session]]
