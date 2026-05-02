---
title: Backend Sprints — MOC
type: moc
tags: [moc, backend, sprints, master-sindico]
project: master-sindico
stack: backend
created: 2026-04-24
updated: 2026-04-24
---

# Backend Sprints — Map of Content

Índice dos sprints backend. Cada sprint é um recorte da entrega com tasks `backend-<N>.M`.

## Cronograma

| Sprint | Período | Status | Objetivo |
|---|---|---|---|
| [[sprint-0]] | 2026-03-23 → 2026-04-06 | ✅ | Fundação Clean Arch + PG + Redis + Gin abstraído + OIDC PKCE |
| [[sprint-1]] | 2026-04-07 → 2026-04-20 | ✅ | Identity & Billing (ABAC, Stripe, plans, trial, quotas) |
| [[sprint-2]] | 2026-04-21 → 2026-04-27 | ✅ | Institutional base (Condomínio, Unit, Timeline, Plano Diretor) |
| [[sprint-3]] | 2026-04-28 → 2026-05-04 | ✅ | Institutional completa (Events, Announcements, Polls, Compliance) |
| [[sprint-4]] | 2026-05-05 → 2026-05-11 | ✅ | Commercial (Connect Me, Company, Proposals, Supplier Vote) |
| [[sprint-5]] | 2026-05-12 → 2026-05-18 | ✅ | Content (Video Mux, Marketing Agencies) |
| [[sprint-6]] | 2026-05-19 → 2026-05-25 | ✅ | Infra (Railway, OTel, CI/CD, Zitadel dev) |
| [[sprint-7]] | 2026-05-26 → 2026-06-01 | ✅ | Assembly foundation (Assembly, Vote, Livekit, Saga) |
| [[sprint-8]] | 2026-06-02 → 2026-06-08 | ✅ | Security hardening (ABAC matriz, rate limit, tenant isolation) |
| [[sprint-9]] | 2026-04-21 → 2026-04-22 (autônomo 10h+) | ✅ | F1-F33 reconciliação + gates verdes + BeyondCorp 12/14 |
| [[sprint-10]] | 2026-04-23 → 2026-05-18 | 🟡 | M1 blockers (14 🔴, LGPD, DT-004, DT-010, BE-001..BE-015) |

## Dívidas técnicas (DT-###) backend

- **DT-003** — circuit breaker Zitadel → Sprint 10 BE-005
- **DT-004** — matriz N1/N2/N3 → Sprint 10 backend-10.9 (reescrita canônico 4 tiers)
- **DT-010** — IAuditLogger cross-BC → Sprint 10 BE-004

## Findings abertos backend

| A-### | Severidade | Sprint alvo |
|---|---|---|
| A-023 | 🟡 | 10 (BE-002) |
| A-024 | 🟡 | 10 (BE-007) |
| A-039 | 🟡 | 10 (BE-003) |
| A-DC-SEC-001..004 | 🔴 | 10 |
| A-DC-PEN-003/010/012/016/038 | 🔴 | 10 |

## Links transversais

- [[../../tasks|tasks monolítico backend]]
- [[../../../../../50-projects/master-sindico/ROADMAP|ROADMAP]]
- [[../../../../../50-projects/master-sindico/STATE|STATE]]
- [[../../../../../50-projects/master-sindico/AUDIT|AUDIT]]
- [[../../../../../50-projects/master-sindico/SESSION_CHARTER|SESSION_CHARTER]]
- [[../../../web/tasks/by-sprint/_moc|Web sprints MOC]]
- [[../../../mobile/tasks/by-sprint/_moc|Mobile sprints MOC]]
