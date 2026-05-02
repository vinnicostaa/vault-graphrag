---
title: Pattern — Dashboard KPI Cards
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

# Pattern — Dashboard KPI Cards

Cards de KPI no topo de telas de governança e gestão (síndico, empresa, admin). Cada card exibe **número + label + delta vs período anterior + sparkline opcional + link drill-down**.

## Estrutura

- **Header**: ícone + label curto (ex.: "Avaliações pendentes", "Mandato — dias")
- **Number**: valor principal grande (formatação localizada pt-BR)
- **Delta**: ↑/↓/→ com cor semântica (verde/vermelho/cinza) + percentual ou absoluto
- **Sparkline** (opcional): mini-gráfico 30d
- **Drill-down**: link para tela detalhada

## Quando usar

- Tela inicial de persona (sindico-home, empresa-dashboard, admin-dashboard).
- Monitoring de saúde de gestão: governance score, compliance status, pendências.
- Vista executiva: 3-6 KPIs no máximo (mais que isso vira ruído).

## Quando evitar

- KPI sem delta interpretável (não vale número solto).
- Drill-down inexistente (KPI sem ação é decoração).

## Stack

SolidJS + Kobalte (Card) + UnoCSS. Animação de entrada com `@motionone`.

## Cross-refs

- [[forms]] — quando KPI tem filtros
- [[dashboard-cards-pivot]] — quando cards alternam visões
- [[../../07-quality/PATTERNS]] — registro de aplicação no projeto

## Links

- [[_moc]]
- [[ui-states]]
