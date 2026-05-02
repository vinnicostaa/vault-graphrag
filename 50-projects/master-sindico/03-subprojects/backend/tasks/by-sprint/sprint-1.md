---
title: Sprint 1 — Backend (Identity & Billing)
type: sprint-spec
tags: [master-sindico, sprint-1, backend, tasks]
project: master-sindico
stack: backend
sprint: 1
period: "2026-04-07 → 2026-04-20"
status: completed
milestone_target: m1
created: 2026-04-24
---

# Sprint 1 Backend — Identity & Billing

**Período**: 2026-04-07 → 2026-04-20 (2 semanas)
**Status**: ✅ concluído
**Objetivo**: Entregar o coração regulatório do M1 — Identity (sincronização Zitadel, sessões, ABAC engine) e Billing (plans, subscriptions, trial por persona, Stripe, feature_usage/quotas).

## Escopo desta sprint (backend)

Backend puro novamente (web/mobile ainda não haviam começado). O ABAC engine entrega deny-by-default policy-based access control sobre o `Subject` (user + role + tier) × `Action` × `Resource`. Billing sobe Stripe como gateway de assinaturas + webhook signing validado antes do body parse.

## Tasks

| ID | Título | Status | Link detalhe | A-### relacionado | D-### |
|---|---|---|---|---|---|
| backend-1.1 | `User` aggregate + `SyncUserUseCase` (Zitadel → DB) | ✅ | `internal/modules/identity/application/use_cases/sync_user_use_case.go` | — | D-010 |
| backend-1.2 | `Session` aggregate + 1-device policy (device fingerprint) | ✅ | `internal/modules/identity/domain/entities/session.go` | A-023 (Sprint 10) | D-011 |
| backend-1.3 | `Authenticate` middleware (introspection + cache + request ctx) | ✅ | `internal/shared/middleware/authenticate.go` | — | D-008 |
| backend-1.4 | ABAC engine policy-based deny-by-default | ✅ | `internal/shared/abac/` | — | D-012 |
| backend-1.5 | `Plan` aggregate + enum `{trial,base,plus,pro}` + fixtures | ✅ | `internal/modules/billing/domain/entities/plan.go` | A-FASE8-PLAN-001 | D-013, ADR-0032 |
| backend-1.6 | `Subscription` + `Customer` + estados (`trial`, `active`, `past_due`, `canceled`, `paused`, `expired`) | ✅ | `internal/modules/billing/domain/entities/subscription.go` | A-FASE8-SUB-001 | D-014 |
| backend-1.7 | Stripe adapter (create customer + checkout session + portal + webhook signing) | ✅ | `internal/modules/billing/infrastructure/providers/stripe/` | — | D-015 |
| backend-1.8 | Webhook handler `POST /webhooks/stripe` (raw body + HMAC antes de parse) | ✅ | `internal/modules/billing/infrastructure/http/routes/webhook_handler.go` | A-024, A-DC-PEN-016 | D-015 |
| backend-1.9 | Trial por persona (síndico 15d, empresa 7d, parceiro 30d, morador 0d) | ✅ | `internal/modules/billing/application/use_cases/start_trial.go` | — | D-016 |
| backend-1.10 | `FeatureUsage` + quotas + enforcement em middleware `QuotaGate` | ✅ | `internal/modules/billing/domain/entities/feature_usage.go` | A-FASE8-FU-001 | D-017 |
| backend-1.11 | PBT ABAC engine + quotas (rapid) | ✅ | `internal/shared/abac/engine_pbt_test.go` (F32) | — | — |

## Dependências

- Sprint anterior: base Go/PG/Zitadel ([[sprint-0]]).
- Cross-stack: nenhuma — web e mobile não iniciaram.
- Decisões: D-010..D-017; INV-SUB-001..010 (state machine Subscription).

## Entregáveis

- `POST /api/v1/billing/checkout` → Stripe Checkout URL.
- `POST /webhooks/stripe` processa `customer.subscription.*` + `invoice.*` idempotente.
- ABAC decide allow/deny em todas as rotas `/api/v1/*` com base em `subscription.status + plan.tier`.
- Trial expira automaticamente (job noturno ou check lazy em read).

## Gates (`<verify>`)

- `go build ./... && go vet ./... && go test -race -count=1 ./...` ✅
- PBT ABAC (400 casos, ~22ms) ✅
- PBT quotas (300 casos) ✅
- gosec severity=high = 0 ✅

## Pós-sprint

- **O que deu certo**: ABAC policy-based permitiu evolução incremental (adicionar resource/action sem refactor); Stripe webhook assinatura-antes-de-parse virou pattern.
- **Bloqueadores**: status `trialing` vs `trial` ficou divergente (reconciliado na sessão autônoma 22/04 via A-FASE8-SUB-001 — alias adapter).
- **Dívidas criadas**: DT-003 (circuit breaker Zitadel introspection), DT-010 (IAuditLogger cross-BC).
- **Próxima sprint**: [[sprint-2|Sprint 2 — Institutional base]].

> Nota histórica: sprint executado cronologicamente em 07→20/04/2026. Reconciliações A-FASE8-PLAN-001, A-FASE8-SUB-001, A-FASE8-FU-001 foram diagnosticadas na sessão autônoma 22/04 e ficam para Sprint 10 backend.

## Links

- [[sprint-0]]
- [[sprint-2]]
- [[../../../../_moc]]
- [[../../../../STATE]]
- [[../../../11-audit/AUDIT]]
