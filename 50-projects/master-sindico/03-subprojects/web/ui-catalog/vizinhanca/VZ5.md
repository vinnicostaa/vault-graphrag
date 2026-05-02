---
title: "VZ5 — Métricas do Parceiro"
type: ui-screen
tags: [master-sindico, ui-catalog, web, vizinhanca, parceiro, metricas]
project: master-sindico
persona: parceiro
category: VZ
screen_id: VZ5
sub_produto: vizinhanca
plan_requirement: trial
status: specification
stack: web
created: 2026-04-23
---

# VZ5 — Métricas do Parceiro

## Finalidade
Painel analítico do parceiro com indicadores e gráficos de engajamento.

## Fonte canônica
- `vault/90-archive/from-development-content-2026-04-23/03-Modulos/Vizinhanca-4-jornadas.md` §VZ5
- `vault/90-archive/master-sindico-pdfs-cliente-originais/Jornada do Parceiro da Vizinhança.md` §TELA VZ5

## Persona e ABAC
- **Persona**: Parceiro (role `local_company`)
- **Plan tier**: `trial`
- **ABAC action**: `commercial.neighborhood_partner.metrics.read.own`

## Indicadores canônicos
- Visualizações do perfil
- Visualizações das promoções
- Ativações (com discriminação por `actor_type`: morador/síndico/empresa)
- Cliques em contato

## Gráficos
- **Engajamento semanal** (line chart visualizações + ativações)
- **Promoções mais ativadas** (bar chart top 5)

## Componentes
- `KPICardRow` (4 cards)
- `EngagementLineChart`
- `TopPromotionsBarChart`
- `ActorTypeBreakdown` (mostra 3 contadores separados — diferencial canônico)
- `PeriodFilter`

## Estados
- **Loading**: skeleton dos cards e charts.
- **Empty (sem dados)**: "Crie sua primeira promoção para acompanhar métricas."
- **Populated**: gráficos com dados.
- **Error**: toast retry; preserva filtros.

## Integração com backend
| Ação | Endpoint (inferido) | Método |
|---|---|---|
| KPIs | `/api/v1/neighborhood/partners/me/metrics` | GET |
| Engajamento semanal | `/api/v1/neighborhood/partners/me/metrics/engagement-weekly` | GET |
| Top promoções | `/api/v1/neighborhood/partners/me/metrics/top-promotions` | GET |

## Regras de negócio críticas
- Métricas separadas por `actor_type` (morador / síndico / empresa) — separação canônica.
- Cross-tenant strict: parceiro vê apenas próprias métricas.
- Cliques em contato e visualizações de perfil são eventos rastreados em backend.
- Sem comparação com outros parceiros (privacidade competitiva).

## Ligações
- [[VZ2]] · [[VZ4]] · [[VZ6]] · [[_moc]]

## Gaps/ressalvas
- Endpoint REST inferido.
- Sem ADR sobre granularidade de tempo (diário vs semanal vs mensal).
- Sem decisão sobre exportação CSV.
- Sem definição de "conversion rate" (visualização → ativação).
- Provedor de gráficos (Chart.js, Recharts, ECharts?) sem ADR.
