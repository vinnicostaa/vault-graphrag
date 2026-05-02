---
title: "VZ6 — Histórico de Promoções (Parceiro)"
type: ui-screen
tags: [master-sindico, ui-catalog, web, vizinhanca, parceiro, historico]
project: master-sindico
persona: parceiro
category: VZ
screen_id: VZ6
sub_produto: vizinhanca
plan_requirement: trial
status: specification
stack: web
created: 2026-04-23
---

# VZ6 — Histórico de Promoções (Parceiro)

## Finalidade
Listar todas as promoções já criadas pelo parceiro (ativas, encerradas, expiradas) com data de publicação, encerramento e número total de ativações.

## Fonte canônica
- `vault/90-archive/from-development-content-2026-04-23/03-Modulos/Vizinhanca-4-jornadas.md` §VZ6
- `vault/90-archive/master-sindico-pdfs-cliente-originais/Jornada do Parceiro da Vizinhança.md` §TELA VZ6

## Persona e ABAC
- **Persona**: Parceiro (role `local_company`)
- **Plan tier**: `trial`
- **ABAC action**: `commercial.neighborhood_benefit.history.own`

## Lista exibida (canônico)
| Coluna |
|---|
| Promoção |
| Data de publicação |
| Data de encerramento |
| Número de ativações |

## Componentes
- `PromotionHistoryTable`
- `FilterBar` (Período + Tipo + Scope)
- `EmptyState`

## Estados
- **Loading**: skeleton.
- **Empty**: "Nenhuma promoção encerrada ainda."
- **Filtered**: contagem.
- **Error**: toast retry.

## Integração com backend
| Ação | Endpoint (inferido) | Método |
|---|---|---|
| Histórico próprio | `/api/v1/neighborhood/promotions?partner_id=me` | GET |

## Regras de negócio críticas
- Append-only — registros não somem após encerramento.
- Cross-tenant strict.
- Inclui ativações dos 3 `actor_types` (agregadas).

## Ligações
- [[VZ2]] · [[VZ4]] · [[VZ5]] · [[_moc]]

## Gaps/ressalvas
- Endpoint REST inferido.
- Sem ADR sobre exportação CSV.
- Sem decisão sobre re-ativar promoção do histórico (clone vs link).
