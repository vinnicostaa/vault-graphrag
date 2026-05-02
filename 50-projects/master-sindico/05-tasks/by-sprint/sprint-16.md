---
title: Sprint 16 — M3 a11y + Relatórios + Deploy
type: spec
tags: [tasks, sprint, sprint-16, m3, master-sindico, v2]
source: backlog.md + ROADMAP + cronograma-macro
created: 2026-04-23
updated: 2026-04-23
---

# Sprint 16 — M3 a11y + Relatórios + Deploy

Janela: **2026-07-08 → 2026-07-20** (13 dias; deploy 2026-07-20).

Meta: fechar M3 — a11y WCAG 2.1 AA, 6 relatórios pós-assembleia + score transparência, SLO 99.5% monitorado, deploy.

---

## Tasks

### a11y (WCAG 2.1 AA)

| ID | Título | Estimate | Dono |
|---|---|---|---|
| T-270 | Audit a11y — screen reader + keyboard nav em 5 fluxos críticos | 1d | web |
| T-271 | Fix contraste, labels, foco visível, landmark roles | 2d | web |
| T-272 | Mobile TalkBack / VoiceOver smoke test | 1d | mobile |

### Relatórios pós-assembleia + transparência

| ID | Título | Estimate | Dono |
|---|---|---|---|
| T-312 | 6 relatórios (presença, votação, procurações, fala, consolidado, transparência) CSV+PDF | 2.5d | web+backend |
| T-313 | Fila fala cronometrada UI + alertas 30s | 1d | web |
| T-314 | Score transparência (10 componentes) dashboard | 1d | web |

### Telão (live streaming) — thin

| ID | Título | Estimate | Dono |
|---|---|---|---|
| T-308 | Telão 2 áreas (Presentation + Operational) — live streaming thin | 2d | web |

### Observabilidade final M3

| ID | Título | Estimate | Dono |
|---|---|---|---|
| T-710 | SLO dashboards Grafana por sub-produto | 1d | ops |
| T-711 | Alertas PagerDuty (error rate, p99, webhooks) | 0.5d | ops |

### Deploy M3 prep

| ID | Título | Estimate | Dono |
|---|---|---|---|
| T-280 | CHANGELOG M3 | 0.25d | ops |
| T-281 | Release notes | 0.25d | ops |
| T-282 | Oncall rotation 24h pós-deploy | 0.25d | ops |
| T-283 | Post-deploy runbook + dashboards revisados | 0.5d | ops |

---

## Gates de saída

- [ ] AUDIT 🔴 = 0; 🟡 com plano
- [ ] a11y audit pass (axe-core 0 violations em fluxos críticos)
- [ ] SLO 99.5% em rotas críticas (últimos 7d)
- [ ] 6 relatórios gerados corretamente em teste
- [ ] Staging 48h estável
- [ ] Freeze 2026-07-18 → deploy 2026-07-19 staging → prod promote 2026-07-20

---

## Ritual M3

- **Sex 2026-07-18**: code freeze.
- **Sáb 2026-07-19**: deploy staging final; smoke; monitor 24h.
- **Dom 2026-07-20**: **🔵 M3 ao vivo**. Monitor 48h (oncall).

---

## Links

- [[_moc]]
- [[sprint-15]]
- [[../backlog]]
- [[../../ROADMAP#m3-assembleia--lms--polish]]
