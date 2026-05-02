---
title: Sprint 13 — M2 Chaos Validation + Deploy
type: spec
tags: [tasks, sprint, sprint-13, m2, master-sindico, v2]
source: backlog.md + ROADMAP + cronograma-macro
created: 2026-04-23
updated: 2026-04-23
---

# Sprint 13 — M2 Chaos Validation + Deploy

Janela: **2026-05-20 → 2026-06-01** (13 dias, buffer pra M2 2026-06-20).

Meta: validar M2 em condições adversas (chaos), finalizar últimos ajustes, deploy staging estável 48h, preparar prod.

---

## Tasks

### Chaos validation

| ID | Título | Estimate | Dono |
|---|---|---|---|
| T-704a | Mux webhook drop → reprocessing funciona | 1d | backend+ops |
| T-704b | Stripe webhook duplicado → idempotency ledger rejeita | 0.5d | backend |
| T-704c | Livekit disconnection durante assembleia → contingency mode | 1d | backend |
| T-704d | Redis cache down → ABAC fallback DB (degradação graceful) | 1d | backend |

### Load + perf tuning

| ID | Título | Estimate | Dono |
|---|---|---|---|
| T-703b | Refinar load test — rotas críticas por sub-produto | 1d | ops |
| T-703c | pprof em endpoints com p99 > 1s; otimizar | 1d | backend |

### Moderação ML (research-only; não implementa)

| ID | Título | Estimate | Dono |
|---|---|---|---|
| T-220 | Research OpenAI Moderation vs AWS Rekognition (ADR em M4+) | 0.5d | arch |

### Polish UI M2

| ID | Título | Estimate | Dono |
|---|---|---|---|
| T-230 | Polish Connect Me UX (revisão jornada, atalhos) | 1d | web |
| T-231 | Polish Vizinhança mapa (loading states, empty states) | 0.5d | web |
| T-232 | Polish moderação admin (bulk actions, filtros) | 0.5d | web |

### Reputação dashboard V2

| ID | Título | Estimate | Dono |
|---|---|---|---|
| T-160 | Dashboard reputação V2 — breakdown por métrica + trending | 1.5d | web |

### Deploy M2 prep

| ID | Título | Estimate | Dono |
|---|---|---|---|
| T-240 | CHANGELOG M2 | 0.25d | ops |
| T-241 | Release notes na PR body | 0.25d | ops |
| T-242 | DB snapshot pré-migration | 0.25d | ops |
| T-243 | Feature flags M2 revisadas | 0.5d | backend |
| T-244 | Oncall rotation configurada | 0.25d | ops |

---

## Gates de saída

- [ ] Chaos tests 100% pass
- [ ] AUDIT 🔴 = 0; 🟡 com plano
- [ ] Load test 1000 users p95 < 500ms
- [ ] Staging 48h estável
- [ ] CHANGELOG + release notes prontos
- [ ] Freeze 2026-06-18 → deploy 2026-06-19 staging → prod promote 2026-06-20

---

## Ritual M2

- **Qui 2026-06-18**: code freeze.
- **Sex 2026-06-19**: deploy staging final; smoke; monitor 24h.
- **Sáb 2026-06-20**: **🟡 M2 ao vivo**. Monitor 48h (oncall).

---

## Links

- [[_moc]]
- [[sprint-12]]
- [[sprint-14]]
- [[../backlog]]
- [[../../ROADMAP#m2-connect-me--conteúdo]]
