---
title: Sprint 0 — Backend (Fundação Go Clean Arch)
type: sprint-spec
tags: [master-sindico, sprint-0, backend, tasks]
project: master-sindico
stack: backend
sprint: 0
period: "2026-03-23 → 2026-04-06"
status: completed
milestone_target: m1
created: 2026-04-24
---

# Sprint 0 Backend — Fundação Clean Architecture

**Período**: 2026-03-23 → 2026-04-06 (2 semanas)
**Status**: ✅ concluído
**Objetivo**: Estabelecer a fundação do backend Go com Clean Architecture, PostgreSQL migrations, Redis, HTTP server abstraído sobre Gin e OIDC PKCE via Zitadel.

## Escopo desta sprint (backend)

O Sprint 0 é 100% backend: não há entrega paralela web/mobile. Foco em destravar o time com uma base sólida que permita paralelizar os próximos módulos (identity, billing, institutional). A escolha por Gin **abstraído** (contrato `contracts.Context`) foi decisão deliberada para trocar por Fiber/Echo futuramente sem tocar handlers (D-067 anterior à renomeação, ver STATE).

## Tasks

| ID | Título | Status | Link detalhe | A-### relacionado | D-### |
|---|---|---|---|---|---|
| backend-0.1 | Estrutura pastas Clean Arch (`internal/{domain,application,infrastructure,shared}`) | ✅ | `Development/backend/internal/` | — | D-001 |
| backend-0.2 | `go.mod` Go 1.26 + deps base (pgx/v5, sqlc, zerolog, gin, testify, rapid) | ✅ | `Development/backend/go.mod` | — | D-002, D-003 |
| backend-0.3 | Migration runner (golang-migrate) + schema base `00001_init.sql` | ✅ | `internal/infrastructure/db/migrations/` | — | D-004 |
| backend-0.4 | Redis client singleton + config env (`REDIS_URL`, `REDIS_POOL_SIZE`) | ✅ | `internal/infrastructure/cache/redis.go` | — | D-005 |
| backend-0.5 | Gin adapter sob `contracts.Context` (abstrai framework) | ✅ | `internal/server/gin_adapter.go` | — | D-006 |
| backend-0.6 | Middleware stack base (recovery, logger, request-id, cors, security-headers) | ✅ | `internal/shared/middleware/` | — | D-007 |
| backend-0.7 | OIDC PKCE start + callback + exchange (Zitadel provider) | ✅ | `internal/modules/identity/infrastructure/providers/zitadel/` | — | D-008 |
| backend-0.8 | Config loader (envconfig) + `Config.Validate()` com fail-fast | ✅ | `internal/shared/config/config.go` | — | D-009 |
| backend-0.9 | Logger zerolog estruturado + `X-Request-Id` propagation | ✅ | `internal/shared/logger/` | — | D-009 |
| backend-0.10 | `cmd/api/main.go` wiring + graceful shutdown (SIGTERM → ctx cancel) | ✅ | `Development/backend/cmd/api/main.go` | — | — |

## Dependências

- Sprint anterior: nenhuma (sprint 0 é ponto de partida).
- Cross-stack: nenhuma — web e mobile não existiam ainda.
- Decisões: D-001..D-009 tomadas antes/durante (Clean Arch, Go 1.26, pgx/v5, sqlc, Zitadel OIDC PKCE, middleware stack ordenado).

## Entregáveis

- Repositório backend bootstrapável em ambiente limpo com `docker compose up` + `make migrate up` + `make run`.
- Healthcheck `GET /healthz` retorna 200.
- OIDC PKCE flow start→callback funcionando contra Zitadel local via docker-compose.
- Migrations idempotentes aplicam em PG17+ limpo.

## Gates (`<verify>`)

- `go build ./...` ✅
- `go vet ./...` ✅
- `go test -race -count=1 ./...` ✅ (cobertura inicial domain ~60%, application ~40%)
- `golangci-lint run` ✅
- PostgreSQL up via docker-compose + Redis up + Zitadel up

## Pós-sprint

- **O que deu certo**: abstração `contracts.Context` permitiu refactor Fiber POC em Sprint 1 sem tocar handlers; middleware stack ordenado ficou correto de primeira; OIDC PKCE com Zitadel local liberou auth para Sprint 1.
- **Bloqueadores**: nenhum crítico; ajustes finos de config env foram feitos inline.
- **Dívidas criadas**: nenhuma nesta sprint; testcontainers ficou para Sprint 9 (F7).
- **Próxima sprint**: [[sprint-1|Sprint 1 — Identity & Billing]].

> Nota histórica: Sprint 0 foi executado cronologicamente em 23/03→06/04/2026. Os refinamentos pós-auditoria (F1-F33) ocorreram na sessão autônoma 22/04 (ver [[../../../STATE#d-092]] e [[../../../../SESSION_CHARTER]]).

## Links

- [[../../../../_moc]]
- [[../../../../_moc]]
- [[../../../../STATE]]
- [[../../../11-audit/AUDIT]]
- [[../../tasks|backend tasks monolítico]]
