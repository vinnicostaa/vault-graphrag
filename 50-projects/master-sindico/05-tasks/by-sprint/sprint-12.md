---
title: Sprint 12 — M2 Polish (Vizinhança + Moderação + Connect Me E↔E)
type: spec
tags: [tasks, sprint, sprint-12, m2, master-sindico, v2]
source: backlog.md + ROADMAP
created: 2026-04-23
updated: 2026-04-23
---

# Sprint 12 — M2 Polish

Janela: **2026-05-13 → 2026-05-25** (13 dias).

Meta: fechar features M2 restantes — Vizinhança + moderação manual UI + Connect Me fluxos avançados (E→E Plus/Pro, S→Parceiro) + OpenSearch prod.

---

## Tasks

### Vizinhança (BC commercial)

| ID | Título | Estimate | Dono |
|---|---|---|---|
| T-500 | Mapa + cupons (lock 4h) | 2.5d | web+backend |
| T-501 | Seal "Síndico-aprovado" UI + fluxo | 1d | web |
| T-510 | Geo search (PostGIS ou lat/long + raio 2km) | 1d | backend |

### Connect Me fluxos avançados

| ID | Título | Estimate | Dono |
|---|---|---|---|
| T-102 | Connect Me empresa→empresa (Plus/Pro) | 1.5d | web |
| T-103 | Connect Me síndico→parceiro Vizinhança (acordo leve) | 1d | web |

### Moderação de vídeos (admin)

| ID | Título | Estimate | Dono |
|---|---|---|---|
| T-201 | Moderação manual UI (admin) — aprovar/rejeitar em 48h | 2d | web |
| T-203 | Privacidade métricas cross-tenant (ABAC + integration tests 10+) | 1.5d | backend |
| T-204 | `VideoVisibilityGrant` (unlock durante votação fornecedor) | 1d | backend |

### Connect Me denúncia + auto-close

| ID | Título | Estimate | Dono |
|---|---|---|---|
| T-110 | Denúncia assédio UI + admin action | 1d | web |
| T-111 | Auto-close 48h (cron) | 0.25d | backend |

### OpenSearch prod

| ID | Título | Estimate | Dono |
|---|---|---|---|
| T-210 | OpenSearch prod deploy + reindex job | 2d | ops+backend |

### Observabilidade

| ID | Título | Estimate | Dono |
|---|---|---|---|
| T-703 | k6 load test 1000 concurrent users | 1d | ops |
| T-704 | Chaos test Mux/Livekit down — Saga compensation validada | 1.5d | backend+ops |

### Dívidas

| ID | Título | Estimate | Dono |
|---|---|---|---|
| T-811 | A-024 — `rawBodyReader` elevado ao contracts/core | 0.5d | backend |
| T-812 | A-031 — Event Response TOCTOU fechar via dual-check | 0.5d | arch |
| T-813 | A-032 — Goroutines lifecycle pattern "context-aware shutdown" | 1d | backend |

---

## Gates de saída

- [ ] AUDIT 🔴 = 0
- [ ] Chaos test passing (Saga compensation Mux/Livekit)
- [ ] k6 1000 users: API p95 < 500ms, p99 < 2s
- [ ] OpenSearch prod indexado
- [ ] Moderação UI demo-ready

---

## Links

- [[_moc]]
- [[sprint-11]]
- [[sprint-13]]
- [[../backlog#must-m2]]
