---
title: "VM7 — Histórico de Ativações (Morador)"
type: ui-screen
tags: [master-sindico, ui-catalog, web, vizinhanca, morador, historico]
project: master-sindico
persona: morador
category: VM
screen_id: VM7
sub_produto: vizinhanca
plan_requirement: base
status: specification
stack: web
created: 2026-04-23
---

# VM7 — Histórico de Promoções Utilizadas (Morador)

## Finalidade
Listar todas as promoções que o morador ativou. Controle de benefícios usados.

## Fonte canônica
- `vault/90-archive/from-development-content-2026-04-23/03-Modulos/Vizinhanca-4-jornadas.md` §VM7
- `vault/90-archive/master-sindico-pdfs-cliente-originais/MÓDULO VIZINHANÇA - MORADOR.md` §TELA VM7

## Persona e ABAC
- **Persona**: Morador (role `resident`)
- **Plan tier**: `base`
- **ABAC action**: `commercial.neighborhood_benefit.history.own`
- **Caminho**: Vizinhança → Histórico

## Lista exibida (canônico)
| Coluna |
|---|
| Promoção |
| Estabelecimento |
| Data da ativação |

## Componentes
- `ActivationTable`
- `FilterBar` (Período + Estabelecimento)
- `EmptyState`

## Estados
- **Loading**: skeleton.
- **Empty**: "Nenhum benefício ativado ainda" + CTA [[VM1]].
- **Error**: toast retry.

## Integração com backend
| Ação | Endpoint (inferido) | Método |
|---|---|---|
| Listar ativações próprias | `/api/v1/neighborhood/activations?actor=resident&actor_id=me` | GET |

## Regras de negócio críticas
- Lista somente ativações com `actor_type=resident`.
- Append-only.
- Cross-tenant strict — morador A não vê ativações de morador B.
- LGPD: dados retidos 5 anos (alinhamento com C9).

## Ligações
- [[VM1]] · [[VM4]] · [[_moc]]

## Gaps/ressalvas
- Endpoint REST inferido.
- Sem ADR sobre exportação CSV.
- Sem decisão sobre exibir cupom anterior na lista (privacidade vs conveniência).
