---
title: Pattern — Dashboard Cards Pivot
type: pattern
tags:
  - pattern
  - ui
  - web
  - master-sindico
  - dashboard
created: '2026-04-27'
updated: '2026-04-27'
---

# Pattern — Dashboard Cards Pivot

Conjunto de cards num dashboard que **alterna entre visões** (período, scope, persona) sem trocar de tela. Pivot via tab-bar ou segmented control no topo do bloco.

## Estrutura

- **Pivot control**: tabs (`Hoje | 7d | 30d | 90d | YTD`) ou segmented (`Meu condo | Todos`).
- **Cards** (filhos de [[dashboard-kpi-cards]]) re-renderizam com dados do pivot.
- **Persistência**: pivot atual em URL (`?period=30d`) e `localStorage` para próxima visita.

## Quando usar

- Dashboards com múltiplas perspectivas naturais (período, escopo).
- Comparação rápida (mudança de pivot = recálculo).

## Quando evitar

- Pivot que muda dado fundamental (dashboard ≠ relatório). Para dado incompatível, navegação dedicada.
- > 5 opções de pivot — vira menu lateral.

## Stack

SolidJS + Kobalte (Tabs ou ToggleGroup). Estado em signal global ou URL params.

## Cross-refs

- [[dashboard-kpi-cards]] — cards filhos
- [[forms]] — pivot é "filtro" simplificado
- [[../../07-quality/PATTERNS]]

## Links

- [[_moc]]
