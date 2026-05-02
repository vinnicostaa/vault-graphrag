---
title: Sprint 11 — Kick-off M2 (Connect Me + Reputação + Vídeos)
type: spec
tags: [tasks, sprint, sprint-11, m2, master-sindico, v2]
source: backlog.md + ROADMAP + cronograma-macro
created: 2026-04-23
updated: 2026-04-23
---

# Sprint 11 — Kick-off M2

Janela: **2026-05-06 → 2026-05-18** (13 dias).

Meta: **arrancar M2** = Connect Me frontend completo + Reputação dashboard + moderação de vídeos UI, aproveitando backend já entregue em Sprints 4-5.

---

## Tasks

### Connect Me — 4 fluxos UI (BC commercial)

| ID | Título | Estimate | Dono |
|---|---|---|---|
| T-100 | UI síndico→empresa — RFP form + lista propostas + escolha | 2d | web |
| T-100a | UI empresa — receber RFP + enviar proposta estruturada | 2d | web |
| T-101 | UI morador→síndico — form estruturado (5 categorias) | 1d | web+mobile |
| T-106 | Quotas Connect Me com reset anual (cron job) | 0.5d | backend |

### Reputação Bronze→Diamante (BC commercial)

| ID | Título | Estimate | Dono |
|---|---|---|---|
| T-150 | Dashboard reputação pública da empresa | 2d | web |
| T-151 | Recálculo deterministico pós-evento (contract closed, eval submitted) | 1d | backend |
| T-152 | PBT (degradação + promoção tier) | 0.5d | backend |
| T-155 | Fechar D-010 — ADR parametrização vs congelada | 0.5d | arch |

### AUDIT findings M2

| ID | Título | Estimate | Dono |
|---|---|---|---|
| T-104 | Fix A-029 — Company Evaluation TOCTOU (`SELECT ... FOR UPDATE` + PBT) | 1d | backend |
| T-105 | Fix A-030 — Supplier Vote race (state machine) | 1d | backend |

### Vídeos + Moderação (BC content)

| ID | Título | Estimate | Dono |
|---|---|---|---|
| T-200 | Player HLS (Mux) + upload presigned UI | 2d | web |
| T-200a | Upload vídeo-currículo Banco de Talentos 90s | 1d | mobile |
| T-202 | Lock 90d enforcement + removal guard | 0.5d | backend |
| T-250-pre | LMS — estrutura de áreas/trilhas (dados seed) | 0.5d | backend |

### Institutional evolução (BC institutional)

| ID | Título | Estimate | Dono |
|---|---|---|---|
| T-020 | Poll UI (enquete informal) + resultado consolidado → timeline | 1.5d | web |
| T-021 | Announcement UI — tracking leitura por morador | 1d | web |

### Observabilidade / monitoring M2

| ID | Título | Estimate | Dono |
|---|---|---|---|
| T-703-pre | k6 smoke test setup (pipeline); baseline p95 | 0.5d | ops |
| T-800 | DT-001 — gaps client-vault consolidados em `01-domain/_gaps-resolvidos.md` | 0.5d | arch |

---

## Gates de saída

- [ ] AUDIT 🔴 = 0 (A-029, A-030 fixados)
- [ ] Gates verdes
- [ ] Staging 48h estável
- [ ] Connect Me end-to-end demonstrável (síndico publica → empresa responde → síndico escolhe)
- [ ] Reputação dashboard visível pra empresa

---

## Links

- [[_moc]]
- [[sprint-10]]
- [[sprint-12]]
- [[../backlog#must-m2]]
- [[../../ROADMAP#m2-connect-me--conteúdo]]
