---
title: MK6 — Dashboard da Empresa (por empresa)
type: ui-screen
tags: [master-sindico, ui-catalog, web, marketing, marketing-agencias, conteudo-videos]
project: master-sindico
persona: marketing
category: MK
screen_id: MK6
sub_produto: marketing-agencias
plan_requirement: any
status: specification
stack: web
created: 2026-04-23
---

# MK6 — Dashboard da Empresa (por empresa)

## Finalidade
Dashboard de desempenho dos vídeos publicados pela agência para uma empresa específica. Indicadores agregados: total de vídeos, visualizações, vídeo mais visto, tempo médio, taxa de retenção. Apoia a agência a otimizar produção e demonstrar valor à empresa cliente.

## Fonte canônica
- `vault/90-archive/master-sindico-pdfs-cliente-originais/JORNADA DA EMPRESA DE MARKETING.md` (TELA MK6 — Dashboard da Empresa)

## Persona e ABAC
- **Persona primária**: Marketing
- **Plan tier mínimo**: any
- **ABAC action**: `marketing.dashboard.read_empresa` (scoped)
- **Restrições**: agência precisa ter contexto da empresa ativo via [[MK3]]; só vê empresas do `agency_id`.

## Fluxo da tela
1. Agência em [[MK3]] → clica card "Dashboard da empresa".
2. Sistema busca métricas agregadas dos vídeos da empresa.
3. Sistema renderiza indicadores em grid + gráficos (linha de evolução, barra por tipo).
4. Agência pode aplicar filtros: período (último mês/trimestre/ano), tipo de vídeo.
5. Cada indicador é clicável (drill-down para [[MK5]] filtrado).

## Componentes
- `IndicatorGrid` (5 cards numéricos)
- `ChartLineEvolution` (visualizações ao longo do tempo)
- `ChartBarByType` (visualizações por tipo de vídeo)
- `TopVideoCard` (vídeo mais visto + thumbnail)
- `FilterToolbar` (período, tipo)
- `EmptyState`

## Indicadores (fonte PDF MK6)
1. **Total de vídeos publicados**
2. **Total de visualizações** (agregado dos vídeos)
3. **Vídeo mais visualizado** (com thumbnail + nome)
4. **Tempo médio de visualização** (em minutos)
5. **Taxa de retenção** (% que assistem até o final)

## Estados
- **Loading**: skeleton dashboard
- **Empty**: "Sem dados ainda. Publique vídeos em [[MK4]] para começar a coletar métricas."
- **Error**: toast retry
- **Filtered empty**: "Nenhum dado para os filtros aplicados"

## Integração com backend
| Ação UI | Endpoint | Método | Retorno |
|---|---|---|---|
| Métricas da empresa | `/api/v1/marketing/empresas/:id/metrics` | GET | { totals, top_video, retention } |
| Evolução temporal | `/api/v1/marketing/empresas/:id/metrics/timeseries` | GET | TimeSeriesPoint[] |

Use case `marketing.GetEmpresaDashboard`. Métricas computadas a partir de eventos Mux + tracking views.

## Regras de negócio críticas
- **Visualizações reais**: contam apenas views únicas (1 view = 1 sessão por usuário) durante exibição em assembleia ou consulta de síndico
- **Não inclui views internas da agência** (auto-exclui agency_user_ids)
- **Privacidade**: dados são agregados; não revela quem (qual síndico/morador) viu — apenas contagem
- **Retenção** calculada com base em watch-time (Mux fornece eventos)
- **Empresa pode acessar mesmo dashboard** via [[../empresa/E10]] (com perspectiva diferente)

## Ligações
- **Tela origem**: [[MK3]]
- **Tela destino**: [[MK5]] (drill-down), [[MK3]] (voltar)
- **Sub-produto**: [[../../../../00-product/sub-produtos/10-marketing-agencias]]
- **Backend**: [[../../requirements]]
- **Inventário**: [[../../../../../../90-archive/from-development-content-2026-04-23/02-Jornadas/Inventario-Telas]]

## Gaps/ressalvas
- Tracking de views: integração com Mux confirmada para encoding mas eventos de view (heartbeats) precisam de player customizado ou webhook Mux Data
- "Engajamento" (likes, comentários) não está nas métricas — fora do escopo MVP
- Comparativos vs outras empresas da agência: tela [[MK7]] cobre — aqui é apenas a empresa selecionada

## Links
- [[../../../../_moc]]
- [[../../../../CLAUDE]]
- [[../../../../client-canvas/Master Canvas]]
