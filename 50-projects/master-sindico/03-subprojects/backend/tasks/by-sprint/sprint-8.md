---
title: Sprint 8 — Backend (Security hardening)
type: sprint-spec
tags: [master-sindico, sprint-8, backend, tasks]
project: master-sindico
stack: backend
sprint: 8
period: "2026-06-02 → 2026-06-08"
status: completed
milestone_target: m1
created: 2026-04-24
---

# Sprint 8 Backend — Security hardening (ABAC + Rate Limit + Tenant Isolation)

**Período**: 2026-06-02 → 2026-06-08 (retrospectivo)
**Status**: ✅ concluído
**Objetivo**: Hardening de segurança transversal — matriz ABAC completa (persona × tier × resource × action), rate limit per-IP + per-user (parcial), isolamento tenant, sanitização `SELECT *`, fix N+1, preparar pentester pass.

## Escopo desta sprint (backend)

Sprint denso de hardening. Fecha A-019 (gosec 66→0 via safecast + pagination), A-020 (N+1 `BatchAdvanceMasterPlanStatus`), A-021 (7 `SELECT *` → lista explícita). Rate limit per-user fica em A-039 aberto (Sprint 10 BE-003).

## Tasks

| ID | Título | Status | Link detalhe | A-### relacionado | D-### |
|---|---|---|---|---|---|
| backend-8.1 | Matriz ABAC completa (persona × tier × resource × action) + PBT 100+ | ✅ | `internal/shared/abac/matrix_test.go` | — | — |
| backend-8.2 | `SafeCast` package (`Int32`, `Int32FromInt64`) para G115 | ✅ | `Development/backend/pkg/safecast/` | A-019 | — |
| backend-8.3 | `shared/pagination` package (PBT 1000 casos) | ✅ | `internal/shared/pagination/` | A-019 | — |
| backend-8.4 | `SELECT *` eliminar em commercial sqlc (7 queries) | ✅ | `internal/modules/commercial/.../queries/*.sql` commit `a0e9cf6` | A-021 | — |
| backend-8.5 | N+1 fix MasterPlan `BatchAdvanceMasterPlanStatus` (UNNEST 3 arrays) | ✅ | `internal/modules/institutional/.../master_plan_repository.go` commit `bfdd157` | A-020 | — |
| backend-8.6 | Rate limit per-IP (`sync.Map[ip]→limiter`) | ✅ | `internal/shared/middleware/rate_limiter.go` | A-032 | — |
| backend-8.7 | `RateLimiter(ctx, rpm, burst)` com lifecycleCtx + graceful shutdown | ✅ | idem acima | A-032 | commit `0b83094` |
| backend-8.8 | CORS wildcard bloqueado no startup em staging/prod (`Config.Validate()`) | ✅ | `internal/shared/config/config.go` | A-DC-SEC-003 | — |
| backend-8.9 | `govulncheck` + `gosec -severity=high` no CI (0 findings) | ✅ | `.github/workflows/ci.yml` | A-019 | — |
| backend-8.10 | `.gitignore` cobre `zitadel-key*.json`, `/migrate`, `/zitadel-setup` | ✅ | `.gitignore` | A-018 | — |
| backend-8.11 | Rate limit per-user (`RateLimitPerUser`) — deixado aberto | 🔴 aberto | — | A-039 | BE-003 Sprint 10 |

## Dependências

- Sprint anterior: assembly foundation ([[sprint-7]]).
- Cross-stack: nenhuma.
- Decisões: steering rate limit (300 rpm user, 60 rpm IP).

## Entregáveis

- Gate `gosec severity=high` zero findings em CI permanente.
- ABAC matrix validada com 100+ casos automatizados.
- Rate limit ativo em todas rotas `/api/v1/*` + `/auth/*`.

## Gates (`<verify>`)

- `go build ./... && go vet ./... && go test -race -count=1 ./...` ✅
- `gosec -severity=high` ✅ 0 (de 66)
- `govulncheck ./...` ✅ 0 CVEs
- `go test -tags=integration ./...` ✅

## Pós-sprint

- **O que deu certo**: safecast + pagination packages viram base para todos os novos handlers; matriz ABAC automatizada elimina classe inteira de bugs.
- **Bloqueadores**: rate limit per-user ficou aberto (A-039); BeyondCorp 12/14 (2 pendentes em A-023 + A-039).
- **Dívidas criadas**: A-023 + A-039 (Sprint 10 BE-002 + BE-003).
- **Próxima sprint**: [[sprint-9|Sprint 9 — Reconciliação autônoma 22/04]].

## Links

- [[sprint-7]]
- [[sprint-9]]
- [[../../../../_moc]]
- [[../../../../STATE]]
- [[../../../11-audit/AUDIT]]
