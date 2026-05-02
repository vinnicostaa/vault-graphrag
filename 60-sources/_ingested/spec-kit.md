---
title: _ingested/spec-kit
type: note
tags:
  - ingested
  - spec-kit
  - tracking
source: ../spec-kit
created: 2026-04-22T00:00:00.000Z
updated: '2026-04-24'
---

# Ingestão — spec-kit

**Tracker** das destilações em andamento. Lista de prioridade viva em [[../spec-kit/_toc#Prioridade de destilação]].

## Dívidas prioritárias (P0/P1)

| Prio | Origem | Destino | Status |
|---|---|---|---|
| P0 | [[../spec-kit/raw/spec-driven]] | `10-knowledge/methodology/spec-driven-development.md` | ⏳ pendente |
| P0 | [[../spec-kit/raw/templates/spec-template]] | `40-templates/sdd/spec-template.md` | ⏳ pendente |
| P0 | [[../spec-kit/raw/templates/plan-template]] | `40-templates/sdd/plan-template.md` | ⏳ pendente |
| P0 | [[../spec-kit/raw/templates/tasks-template]] | `40-templates/sdd/tasks-template.md` | ⏳ pendente |
| P0 | [[../spec-kit/raw/templates/constitution-template]] | `40-templates/sdd/constitution-template.md` | ⏳ pendente |
| P0 | [[../spec-kit/raw/templates/checklist-template]] | `40-templates/sdd/checklist-template.md` | ⏳ pendente |
| P1 | spec-kit × GSD × SDD nosso | `10-knowledge/methodology/sdd-comparison.md` | ⏳ pendente |
| P1 | Sistema de extensões | avaliar adoção | ⏳ pendente |
| P1 | [[../spec-kit/raw/templates/commands/specify]] + /plan + /tasks + /implement + /clarify | `30-agents/playbooks/sdd-slash-commands.md` | ⏳ pendente |
| P1 | [[../spec-kit/raw/docs/reference/overview]] | contexto em `10-knowledge/methodology/` | ⏳ pendente |

## Destiladas em staging local (`_notes/`)

*(nenhuma ainda)*

## Promovidas pro vault canônico

### 2026-04-24 — Atualização SDD taxonomia 2026 (D-vault-002)

- [[../../10-knowledge/methodology/kiro-vs-spec-kit]] reescrito como v2.1: 3 escolas (Kiro + Spec Kit + Tessl), 9 slash commands divididos em 6 core + 3 optional, 3 fases macro do Spec Kit (Greenfield / Creative Exploration / Brownfield), taxonomia Fowler de 3 níveis (spec-first / spec-anchored / spec-as-source), 30+ AI agents suportados.
- [[../../10-knowledge/methodology/sdd-adoption-gaps]] expandido com **G-007** (workflow mismatch — spec com granularidade errada pro tamanho do problema) e **G-008** (review burden — verbosidade excede esforço de revisar código direto), ambas derivadas do [artigo Martin Fowler 2025-10-15](https://martinfowler.com/articles/exploring-gen-ai/sdd-3-tools.html).
- [[../../10-knowledge/methodology/spec-driven-development]] — já existia; continua sem mudança nesta sessão.

### Pendente (ver dívidas acima)

Templates em `40-templates/sdd/` (spec-template, plan-template, tasks-template, constitution-template, checklist-template) — dívida P0 ainda aberta.

## Vizinhos

- [[../spec-kit/_toc]]
- [[_moc]]
- [[../../00-meta/STATE]] — D-vault-002
