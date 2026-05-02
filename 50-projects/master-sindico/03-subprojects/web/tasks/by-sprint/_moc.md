---
title: Web Sprints — MOC
type: moc
tags: [moc, web, sprints, master-sindico]
project: master-sindico
stack: web
created: 2026-04-24
updated: 2026-04-24
---

# Web Sprints — Map of Content

Índice dos sprints web. **Atenção**: a maioria dos sprints pré-10 está vazia — web arrancou o scaffolding em 2026-04-13 (sprint-0) e só retoma atividade real em Sprint 10. Esse é o cerne do finding [[../../../11-audit/AUDIT#a-fase10-dev-005|A-FASE10-DEV-005 🔴]] bloqueador M1.

## Cronograma

| Sprint | Período | Status | Observação |
|---|---|---|---|
| [[sprint-0]] | 2026-04-13 | ✅ parcial | Scaffold 6 apps + packages; apps ficam placeholders |
| [[sprint-1]] | 2026-04-07 → 2026-04-20 | ⏳ N/A | Web não existia ainda |
| [[sprint-2]] | 2026-04-21 → 2026-04-27 | ⏳ sem entrega | Apps placeholders |
| [[sprint-3]] | 2026-04-28 → 2026-05-04 | ⏳ sem entrega | Apps placeholders |
| [[sprint-4]] | 2026-05-05 → 2026-05-11 | ⏳ sem entrega | Apps placeholders |
| [[sprint-5]] | 2026-05-12 → 2026-05-18 | ⏳ sem entrega | Apps placeholders |
| [[sprint-6]] | 2026-05-19 → 2026-05-25 | ⏳ sem entrega | Apps placeholders |
| [[sprint-7]] | 2026-05-26 → 2026-06-01 | ⏳ sem entrega | Apps placeholders |
| [[sprint-8]] | 2026-06-02 → 2026-06-08 | ⏳ sem entrega | Apps placeholders |
| [[sprint-9]] | 2026-04-21 → 2026-04-22 (autônomo) | ✅ audit | axios CVE + AGENTS.md limpa; diagnóstico A-FASE10-DEV-005 |
| [[sprint-10]] | 2026-04-23 → 2026-05-18 | 🟡 em execução | **M1 inteiro concentrado** — bloqueador crítico |

## Roadmap web (marcos)

- **M1 (2026-05-08 ou 05-18 conforme Opção)**: auth + cms onboarding + dashboard básico → Sprint 10
- **M2 (2026-06-20)**: Connect Me + Content + Agências + Forum → Sprint 11 (FE-100..FE-193)
- **M3 (2026-07-20)**: Assembly + LMS + polish → Sprint 12 (FE-200..FE-253)

## Dívidas técnicas (DT-WEB-###)

Ver [[../../tasks#dividas-tecnicas-identificadas-dt-web]] — 15 DTs catalogados. Principais:

- **DT-WEB-001** railway.json turbo → FE-002
- **DT-WEB-002** btn-destructive ausente → FE-012
- **DT-WEB-004** uno.config duplicado 6× → FE-017
- **DT-WEB-005** `packages/schemas` stub → FE-010
- **DT-WEB-011** sem `packages/core` → M2

## Findings abertos web

| A-### | Severidade | Sprint alvo |
|---|---|---|
| A-FASE10-DEV-005 | 🔴 M1 | Sprint 10 (decisão stakeholder) |

## Links transversais

- [[../../tasks|tasks monolítico web]]
- [[../../../../../50-projects/master-sindico/ROADMAP|ROADMAP]]
- [[../../../../../50-projects/master-sindico/STATE|STATE]]
- [[../../../../../50-projects/master-sindico/AUDIT|AUDIT]]
- [[../../../backend/tasks/by-sprint/_moc|Backend sprints MOC]]
- [[../../../mobile/tasks/by-sprint/_moc|Mobile sprints MOC]]
