---
title: Implementation Rules — Master Síndico
type: project
tags: [master-sindico, domain, implementation-rules]
project: master-sindico
created: 2026-04-22
---

# Implementation Rules

**Como** implementamos pra garantir as regras de [[business-rules]] e [[domain-rules]].

Separação útil porque: business rules mudam por vontade de produto; domain rules são lei do negócio; implementation rules mudam por decisão técnica (D-### / ADR).

---

## IR-001 — Clean Architecture em camadas

Dependência aponta **para dentro**: `infrastructure → application → domain`.

- `domain/` sem import externo
- `application/` só importa `domain/` do próprio módulo + `core/`
- `infrastructure/` importa `application/` + `domain/` + SDKs externos

Ver [[../07-quality/CLEAN_ARCH]].

## IR-002 — CQRS em use cases

- Command ≠ Query em arquivos separados
- `create_x_use_case.go` (cmd) e `list_x_use_case.go` (query)
- Input: struct `XCommand` / `XQuery`
- Output: `(Result, error)` onde Result é ID ou DTO

## IR-003 — Repositório retorna entidade de domínio

```go
// NUNCA
func (r *UserRepository) FindByID(...) (*sqlc.User, error)

// SEMPRE
func (r *UserRepository) FindByID(...) (*User, error)
```

Mapper obrigatório entre row e entidade.

## IR-004 — UoW para atômico local; Saga para cruzar externo

Ver [[../07-quality/PATTERNS#uow-unit-of-work]] e [[../07-quality/PATTERNS#saga-compensação]].

## IR-005 — sqlc com colunas explícitas

Proibido `SELECT *` em queries novas. Exceção: `SELECT COUNT(*)`.

## IR-006 — Migrations particionadas por módulo

- identity: 001-099
- billing: 100-199
- institutional: 200-299
- commercial: 300-399
- content: 400-499
- assembly: 500-599

Migrations forward-only por default. DROP via expand-contract (2 sprints).

## IR-007 — UUIDv7 como PK

`pkg/utils/uuid7.go` — gera UUIDv7 (prefixo timestamp), ordenável.

## IR-008 — Money em int64 centavos

`pkg/money` expõe só `Money` struct com int64; operações safe via métodos.

## IR-009 — Error wrapping

```go
// Error wrapping preserva contexto
return fmt.Errorf("create video %s: %w", id, err)

// Sentinel tipado em fronteira
return apperrors.ErrNotFound.WithMessage("vídeo não encontrado")
```

## IR-010 — ABAC: middleware + cache

- Middleware `ABAC` popula `AbacDecision` em `contracts.Context`
- Cache Redis por `{user_id, tenant_id}` TTL 5min
- Invalidação via handler de webhook Stripe `customer.subscription.*`

## IR-011 — DeviceFingerprint: hash server-side

Client envia string; server hashia (SHA-256) antes de persistir. Nunca reverter.

## IR-012 — Webhook signature antes de parse

```go
// CORRETO
body, _ := io.ReadAll(ctx.Request().Body)
if !verifyStripeSignature(body, signature, secret) {
    return apperrors.ErrUnauthorized
}
var event stripe.Event
json.Unmarshal(body, &event)
```

## IR-013 — Bearer vs Cookie

- `Authenticate` middleware:
  - Se Cookie `session` → sessão web
  - Se `Authorization: Bearer ...` → sessão mobile
  - Prioridade: Cookie sobre Bearer (se ambos)

## IR-014 — Paginação cursor-based

Sem offset (não escala). Cursor opaco (base64 de `{last_id, last_created_at}`).

Impl: `pkg/pagination`.

## IR-015 — Error global handler

- Adapter (ex: `GinAdapter`) captura panics e erros
- Mapeia `AppError` → HTTP status:
  - `ErrValidation` → 422
  - `ErrNotFound` → 404
  - `ErrConflict` → 409
  - `ErrForbidden` → 403
  - `ErrUnauthorized` → 401
  - `ErrInternal` / panic → 500 (logado)

## IR-016 — Goroutines com lifecycle

Toda goroutine de longo prazo aceita `context.Context`:

```go
func (r *RateLimiter) cleanup(ctx context.Context) {
    ticker := time.NewTicker(30 * time.Minute)
    defer ticker.Stop()
    for {
        select {
        case <-ctx.Done():
            return
        case <-ticker.C:
            r.purgeStale()
        }
    }
}
```

A-032 fix (F28).

## IR-017 — Context propagation em queries

Repos pegam tx ativa do context via `database.DBFromContext(ctx, pool)`:

```go
func (r *UserRepo) Save(ctx context.Context, u *User) error {
    db := database.DBFromContext(ctx, r.pool)  // tx se em UoW; pool caso contrário
    return r.queries.UpsertUser(ctx, db, ...)
}
```

## IR-018 — `//go:build integration` para integration tests

Rodar: `go test -tags=integration ./...`. Helper em `internal/shared/testutil`.

## IR-019 — Logger estruturado com contexto obrigatório

```go
logger.Info().
    Str("request_id", ctx.RequestID()).
    Str("tenant_id", ctx.Tenant().String()).
    Str("user_id", ctx.User().ID.String()).
    Msg("user logged in")
```

Campos obrigatórios em logs de handler: request_id, tenant_id (se auth), user_id (se auth).

## IR-020 — Sem mock de DB em integration

testcontainers-go obrigatório. `internal/shared/testutil/pg.go`.

## IR-021 — Gates bloqueiam merge

```
go build + go vet + go test -race + go test -tags=integration + coverage + gosec + govulncheck
```

Ver [[../CLAUDE#8-coverage-thresholds]].

## IR-022 — Sprint-auditor ao fim do sprint

Skill local em `.claude/skills/sprint-auditor.md`. Bloqueia próximo sprint se AUDIT 🔴🟡.

## IR-023 — Commit imperativo com escopo claro

```
billing: Add Stripe webhook signature validation
identity: Fix TOCTOU in session invalidation
```

Sem `fix:`/`feat:`. Corpo explica **porquê**.

---

## Relação com outros padrões

- Aplicação de [[../../../10-knowledge/patterns/_moc]]
- Consolidado em [[../07-quality/PATTERNS]]

## Links

- [[_moc]]
- [[business-rules]]
- [[domain-rules]]
- [[invariants]]
- [[../07-quality/PATTERNS]]
- [[../07-quality/CLEAN_ARCH]]
- [[../07-quality/CLEAN_CODE]]
- [[../CLAUDE]]
