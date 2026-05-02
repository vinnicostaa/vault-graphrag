---
title: Sprint 6 — Backend (Infra — Railway + OTel + CI/CD)
type: sprint-spec
tags: [master-sindico, sprint-6, backend, tasks]
project: master-sindico
stack: backend
sprint: 6
period: "2026-05-19 → 2026-05-25"
status: completed
milestone_target: infra
created: 2026-04-24
---

# Sprint 6 Backend — Infra (Railway + OTel + CI/CD)

**Período**: 2026-05-19 → 2026-05-25 (retrospectivo — consolidado 22/04)
**Status**: ✅ concluído
**Objetivo**: Preparar infra de produção — Dockerfile multi-stage, Railway deploy, GitHub Actions CI, Zitadel Railway (parcial), OpenTelemetry completo (traces + metrics) + Prometheus + Sentry.

## Escopo desta sprint (backend)

Sprint infra-only. Zitadel Railway managed ficou com Task 9.19 aberta (dev local OK; prod pendente). OTel foi completo — tracer provider + PgxTracer + middleware + Prometheus exporter.

## Tasks

| ID | Título | Status | Link detalhe | A-### relacionado | D-### |
|---|---|---|---|---|---|
| backend-6.1 | Dockerfile multi-stage (build-cache + distroless + non-root) | ✅ | `Development/backend/Dockerfile` | — | commit `5114325` |
| backend-6.2 | `railway.toml` + healthcheck + restart policy | ✅ | `Development/backend/railway.toml` | — | — |
| backend-6.3 | GitHub Actions CI (`build + vet + test -race + gosec + govulncheck`) | ✅ | `.github/workflows/ci.yml` | — | — |
| backend-6.4 | Deploy staging step (token deploy pendente) | 🟡 | `.github/workflows/ci.yml` | — | — |
| backend-6.5 | Resend adapter (transactional email) | ✅ | `internal/infrastructure/providers/resend/resend_provider.go` | — | D-026, D-046 |
| backend-6.6 | MinIO adapter S3-compatible (uploads não-Mux; dossiês PDF) | ✅ | `internal/infrastructure/providers/minio/minio_provider.go` | — | — |
| backend-6.7 | OpenTelemetry: tracer provider + `middleware.Tracing` + PgxTracer + Prometheus | ✅ | `pkg/telemetry/` + `internal/shared/middleware/tracing.go` | — | D-067 |
| backend-6.8 | OpenAPI /docs (Scalar UI) auto-gerado | ✅ | `internal/server/docs.go` | — | — |
| backend-6.9 | Idempotency key em Stripe `Create*` calls | ✅ | `internal/modules/billing/infrastructure/providers/stripe/stripe_gateway.go` | — | ADR-0027 |
| backend-6.10 | `pgxpool` ConnMaxIdleTime env-configurável (`DB_CONN_MAX_IDLE_TIME`) | ✅ | `internal/infrastructure/db/pool.go` | — | — |
| backend-6.11 | docker-compose Zitadel v4 + Traefik + zitadel-login dev local | ✅ | `docker-compose.yml` commits `218a7a3`, `70dd90f` | — | — |
| backend-6.12 | **Zitadel managed Railway + DNS `auth.mastersindico.com.br`** | 🔴 aberto | — | — | Task 9.19 / BE-012 Sprint 10 |

## Dependências

- Sprint anterior: content ([[sprint-5]]).
- Cross-stack: nenhuma.
- Decisões: D-067 (infra Railway + Resend + R2/S3); ADR-0027 (webhook idempotency DB UNIQUE).

## Entregáveis

- Dockerfile reprodutível < 80MB final image.
- CI roda em 3-5min e bloqueia PR se gate falhar.
- Traces distribuídos via OTel em Grafana/Jaeger.
- Prometheus expõe métricas `http_requests_total`, `pgxpool_*`, `stripe_webhooks_*`.

## Gates (`<verify>`)

- `go build ./... && go vet ./... && go test -race -count=1 ./...` ✅
- `gosec -severity=high` ✅ 0 findings
- `govulncheck ./...` ✅ 0 CVEs
- Container scan trivy/grype zero críticos ✅

## Pós-sprint

- **O que deu certo**: OTel single source of truth para traces + metrics; Prometheus endpoint limpo.
- **Bloqueadores**: Zitadel Railway managed — dev local funciona mas prod não está no DNS final (BE-012 Sprint 10).
- **Dívidas criadas**: BE-012 (Zitadel Railway + DNS), observability maturity (M3).
- **Próxima sprint**: [[sprint-7|Sprint 7 — Assembly foundation]].

## Links

- [[sprint-5]]
- [[sprint-7]]
- [[../../../../_moc]]
- [[../../../../STATE]]
- [[../../../11-audit/AUDIT]]
