---
title: MK2 — Empresas Atendidas
type: ui-screen
tags: [master-sindico, ui-catalog, web, marketing, marketing-agencias]
project: master-sindico
persona: marketing
category: MK
screen_id: MK2
sub_produto: marketing-agencias
plan_requirement: any
status: specification
stack: web
created: 2026-04-23
---

# MK2 — Empresas Atendidas

## Finalidade
Lista as empresas que autorizaram a agência a administrar conteúdos institucionais via [[../empresa/E14]]. Cada empresa é um "tenant" no qual a agência pode atuar (publicar vídeos, gerenciar biblioteca, ver dashboard). Empresas só aparecem APÓS aceitar convite.

## Fonte canônica
- `vault/90-archive/master-sindico-pdfs-cliente-originais/JORNADA DA EMPRESA DE MARKETING.md` (TELA MK2 — Empresas Atendidas)

## Persona e ABAC
- **Persona primária**: Marketing
- **Plan tier mínimo**: any
- **ABAC action**: `marketing.empresas.list`
- **Restrições**: lista filtrada por `agency_id`; só empresas com vínculo ativo (ou encerrado, com badge histórico).

## Fluxo da tela
1. Agência em [[MK1]] → clica card "Empresas atendidas".
2. Sistema lista empresas com vínculo ativo (e opcionalmente encerradas em aba separada).
3. Cada linha: Empresa (nome fantasia), Segmento, Cidade, Data de início da parceria, Status (Ativa/Encerrada).
4. Agência clica [Gerenciar empresa] → navega para [[MK3]] no contexto daquela empresa.

## Componentes
- `EmpresaTable` (linhas com badge status)
- `FilterBar` (status, segmento, cidade)
- `EmptyState`
- `BadgeStatus`
- `PaginationControl`

## Campos exibidos por linha (fonte PDF MK2)
- Empresa (nome fantasia + logo)
- Segmento da empresa
- Cidade
- Data de início da parceria
- Status (Ativa / Encerrada)
- Botão [Gerenciar empresa]

## Estados
- **Loading**: skeleton tabela
- **Empty**: "Nenhuma empresa atendida ainda. Empresas aparecem aqui após aceitar seu convite."
- **Error**: toast retry
- **Encerrada**: linha cinza com badge "Encerrada em DD/MM/AAAA" — read-only (R3)

## Integração com backend
| Ação UI | Endpoint | Método | Retorno |
|---|---|---|---|
| Listar empresas | `/api/v1/marketing/empresas` | GET | EmpresaAtendida[] |
| Filtrar | query string | GET | filtrado |

Use case `marketing.ListEmpresasAtendidas` (inferido).

## Regras de negócio críticas
- **Empresas só aparecem após aceitar convite** enviado pela empresa em [[../empresa/E14]]
- **Encerramento**: quando empresa revoga em [[../empresa/E14]], status vira "Encerrada" e agência perde grants ABAC (gestão futura bloqueada). Vídeos publicados permanecem (R3).
- **Histórico preservado**: empresas encerradas ficam visíveis em aba "Encerradas" (R3)
- **Multi-empresa**: agência pode atender muitas — sem limite explícito (negociar contrato)

## Ligações
- **Tela origem**: [[MK1]]
- **Tela destino**: [[MK3]] (contexto daquela empresa)
- **Sub-produto**: [[../../../../00-product/sub-produtos/10-marketing-agencias]]
- **Backend**: [[../../requirements]]
- **Inventário**: [[../../../../../../90-archive/from-development-content-2026-04-23/02-Jornadas/Inventario-Telas]]

## Gaps/ressalvas
- Tab "Encerradas" não está explícita no PDF — proposto para preservar princípio R3 (histórico visível)
- Limite máximo de empresas por agência: pendente de definição comercial
- Preview de últimas atividades por empresa (último vídeo, métricas) não está especificado — sugerir card de estatísticas no hover/expand

## Links
- [[../../../../_moc]]
- [[../../../../CLAUDE]]
- [[../../../../client-canvas/Modelo ABAC]]
