---
title: MK7 — Dashboard Geral (consolidado)
type: ui-screen
tags: [master-sindico, ui-catalog, web, marketing, marketing-agencias]
project: master-sindico
persona: marketing
category: MK
screen_id: MK7
sub_produto: marketing-agencias
plan_requirement: any
status: specification
stack: web
created: 2026-04-23
---

# MK7 — Dashboard Geral (consolidado)

## Finalidade
Dashboard consolidado da agência: visão geral do desempenho de **todas as empresas atendidas**. Indicadores agregados servem para a agência avaliar sua operação total e identificar a empresa com maior engajamento.

## Fonte canônica
- `vault/90-archive/master-sindico-pdfs-cliente-originais/JORNADA DA EMPRESA DE MARKETING.md` (TELA MK7 — Dashboard Geral da Agência)

## Persona e ABAC
- **Persona primária**: Marketing (Administrador da agência preferencialmente)
- **Plan tier mínimo**: any
- **ABAC action**: `marketing.dashboard.read_consolidated`
- **Restrições**: scoped a `agency_id`; agrega apenas empresas com vínculo ativo (e opcionalmente histórico de encerradas).

## Fluxo da tela
1. Agência em [[MK1]] → clica card "Dashboard de desempenho".
2. Sistema agrega métricas de todas as empresas atendidas.
3. Sistema renderiza indicadores numéricos em grid + ranking de empresas + gráficos.
4. Agência pode aplicar filtros: período, tipo de vídeo (cross-empresa).
5. Cards/linhas de empresas são clicáveis → navegam para [[MK6]] daquela empresa.

## Componentes
- `IndicatorGrid` (5 cards)
- `EmpresaRankingTable` (top empresas por engajamento)
- `ChartLineConsolidated` (evolução total visualizações)
- `ChartBarByType` (cross-empresa por tipo)
- `TopVideoCardGlobal` (vídeo mais acessado de toda a agência)
- `FilterToolbar` (período, tipo)
- `ExportPDFButton` (futuro)

## Indicadores (fonte PDF MK7)
1. **Número de empresas atendidas** (total ativo)
2. **Total de vídeos publicados** (agregado de todas as empresas)
3. **Total de visualizações** (agregado)
4. **Vídeo mais acessado** (de toda a agência — com nome empresa + thumbnail)
5. **Empresa com maior engajamento** (cálculo: views/vídeos × retenção)

## Estados
- **Loading**: skeleton dashboard
- **Empty**: "Sem dados consolidados ainda. Aguarde primeiras publicações em [[MK4]]."
- **Error**: toast retry
- **Single-empresa**: agência atendendo apenas 1 empresa — exibe banner "Adicione mais empresas para comparativos"

## Integração com backend
| Ação UI | Endpoint | Método | Retorno |
|---|---|---|---|
| Dashboard consolidado | `/api/v1/marketing/dashboard/consolidated` | GET | { indicators, empresa_ranking, top_video } |
| Filtrar | query string | GET | filtrado |
| Exportar PDF | `/api/v1/marketing/dashboard/export` | POST | PDF binary (futuro) |

Use case `marketing.GetConsolidatedDashboard`. Computa cross-empresa scoped por `agency_id`.

## Regras de negócio críticas
- **Privacidade entre empresas**: agência VÊ dados de suas empresas (próprio escopo) — nunca de empresas de outras agências
- **Empresas encerradas**: aparecem em filtro opcional "Incluir encerradas" (R3); por padrão só ativas
- **Cálculo de engajamento**: fórmula sugerida `(views × retention) / videos_count` (a definir com PO)
- **Privacidade do dado individual**: ranking exibe nome fantasia da empresa — empresas estão cientes ao convidar agência (consent implícito)
- **Exportação PDF**: relatório formatado para apresentação à diretoria da agência ou clientes

## Ligações
- **Tela origem**: [[MK1]]
- **Tela destino**: [[MK6]] (drill-down por empresa), [[MK2]] (gestão empresas)
- **Sub-produto**: [[../../../../00-product/sub-produtos/10-marketing-agencias]]
- **Backend**: [[../../requirements]]
- **Inventário**: [[../../../../../../90-archive/from-development-content-2026-04-23/02-Jornadas/Inventario-Telas]]

## Gaps/ressalvas
- Fórmula de "Empresa com maior engajamento" não definida no PDF — proposta acima, validar com PO
- Comparativos com benchmarks de mercado (média de outras agências) NÃO previstos — fora escopo (privacidade)
- Filtros "tipo de vídeo cross-empresa" requerem joins; performance precisa de cache/agregação pré-calculada (Sprint 6+)

## Links
- [[../../../../_moc]]
- [[../../../../CLAUDE]]
- [[../../../../client-canvas/Master Canvas]]
