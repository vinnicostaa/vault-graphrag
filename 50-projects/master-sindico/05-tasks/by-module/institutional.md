---
title: Tasks — BC institutional
type: spec
tags: [tasks, by-module, institutional, master-sindico, v2]
source: 04-requirements/functional/institutional.md + AUDIT A-020 + backlog.md
created: 2026-04-23
updated: 2026-04-23
---

# Tasks — BC `institutional`

Tasks do bounded context **institutional** (Condomínio, Unidades, Memberships, Timeline imutável, Plano Diretor, Compliance, Governance Score).

Alinhamento: [[../../04-requirements/functional/institutional]] (INT-001..INT-026).

---

## M1

| ID | Título | Sprint | Status |
|---|---|---|---|
| T-001b | Dashboard condomínio — 13 cards (INT-025) | 10 | ⏳ |
| T-001c | Timeline view — append-only, 7 tipos, adendo (INT-007..009) | 10 | ⏳ |
| T-001d | Plano Diretor view — 26 áreas × 7 status (INT-011..012) | 10 | ⏳ |
| T-001e | Seletor contexto síndico 2+ condos (INT-005) | 10 | ⏳ |
| T-003 | A-020 — Fix N+1 UPDATE em `master_plan_timeline_handler` | 10 | ⏳ |
| T-008 | Governance Score job noturno em prod (INT-024) | 10 | ⏳ |

## M2

| ID | Título | Sprint | Status |
|---|---|---|---|
| T-020 | Poll UI (INT-015) — enquete + resultado consolidado → timeline | 11 | ⏳ |
| T-021 | Announcement UI (INT-014) — tracking leitura morador | 11 | ⏳ |
| T-021a | Ciência obrigatória antes assembleia (bloqueia app até `Acknowledge`) | 11 | ⏳ |

## M3

| ID | Título | Sprint | Status |
|---|---|---|---|
| T-050 | Event 13 tipos UI (INT-013) | 15 | ⏳ |
| T-051 | Compliance anual UI + bloqueio exportação dossiê | 15 | ⏳ |

---

## Invariantes cobertos

I-013..I-020 de [[../../01-domain/bounded-contexts#institutional]]:
- I-013 Σ `FracaoIdeal` ≤ 100%
- I-014 `FracaoIdeal ∈ (0, 100]`
- I-015 TimelineEntry imutável (sem UPDATE/DELETE/`deleted_at`)
- I-016 Announcement imutável pós-publish
- I-017 Membership ativa única por (user, condominium)
- I-018 ≤ 15 condomínios por síndico
- I-019 Mandate `end_date > start_date`
- I-020 Unit tem 1 titular + ≤ 2 dependentes

---

## Links

- [[_moc]]
- [[../../04-requirements/functional/institutional]]
- [[../../01-domain/bounded-contexts#institutional]]
- [[../backlog]]
