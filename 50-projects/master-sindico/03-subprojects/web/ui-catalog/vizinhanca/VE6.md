---
title: "VE6 — Histórico de Promoções (Empresa)"
type: ui-screen
tags: [master-sindico, ui-catalog, web, vizinhanca, empresa, historico]
project: master-sindico
persona: empresa
category: VE
screen_id: VE6
sub_produto: vizinhanca
plan_requirement: plus
status: specification
stack: web
created: 2026-04-23
---

# VE6 — Histórico de Promoções Utilizadas (Empresa)

## Finalidade
Listar todas as promoções que a empresa ativou. Controle de cupons usados.

## Fonte canônica
- `vault/90-archive/from-development-content-2026-04-23/03-Modulos/Vizinhanca-4-jornadas.md` §VE6

## Persona e ABAC
- **Persona**: Empresa (role `enterprise`)
- **Plan tier**: `plus`
- **ABAC action**: `commercial.neighborhood_benefit.history.own`

## Lista exibida (canônico)
| Coluna |
|---|
| Promoção |
| Estabelecimento |
| Data da ativação |
| Cupom (código) |

## Componentes
- `ActivationTable`
- `FilterBar` (Período + Estabelecimento)
- `EmptyState`

## Estados
- **Loading**: skeleton.
- **Empty**: "Nenhuma promoção ativada ainda" + CTA [[VE2]].
- **Filtered**: contagem visível.
- **Error**: toast retry.

## Integração com backend
| Ação | Endpoint (inferido) | Método |
|---|---|---|
| Listar ativações da empresa | `/api/v1/neighborhood/activations?actor=enterprise&actor_id=me` | GET |

## Regras de negócio críticas
- Lista somente ativações com `actor_type=enterprise`.
- Append-only — sem delete (LGPD/compliance).
- Cross-tenant strict — empresa A nunca vê ativações de empresa B.

## Ligações
- [[VE1]] · [[VE2]] · [[VE5]] · [[_moc]]

## Gaps/ressalvas
- Endpoint REST inferido.
- Sem ADR sobre exportação CSV.
- Sem decisão sobre métricas analíticas avançadas (Pro vs Plus).
