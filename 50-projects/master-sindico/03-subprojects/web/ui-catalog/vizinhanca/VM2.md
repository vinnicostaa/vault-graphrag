---
title: "VM2 — Detalhe do Parceiro (Morador)"
type: ui-screen
tags: [master-sindico, ui-catalog, web, vizinhanca, morador]
project: master-sindico
persona: morador
category: VM
screen_id: VM2
sub_produto: vizinhanca
plan_requirement: base
status: specification
stack: web
created: 2026-04-23
---

# VM2 — Detalhe do Parceiro (Morador)

## Finalidade
Página de detalhe do estabelecimento com nome, categoria, endereço, telefone, descrição e promoções disponíveis (abertas + exclusivas do próprio condomínio).

## Fonte canônica
- `vault/90-archive/from-development-content-2026-04-23/03-Modulos/Vizinhanca-4-jornadas.md` §VM2
- `vault/90-archive/master-sindico-pdfs-cliente-originais/MÓDULO VIZINHANÇA - MORADOR.md` §TELA VM2

## Persona e ABAC
- **Persona**: Morador (role `resident`)
- **Plan tier**: `base`
- **ABAC action**: `commercial.neighborhood_partner.read`

## Informações exibidas
- Nome do estabelecimento
- Categoria
- Endereço
- Telefone / WhatsApp
- Descrição
- Promoções disponíveis (abertas + exclusivas do próprio condomínio)

## Botões
- **Ver promoção** → [[VM3]]
- **Entrar em contato**
- **Ir até o local** (abre maps)

## Componentes
- `PartnerHero`
- `PartnerInfoBlock`
- `PromotionList`
- `ActionGroup` (3 botões)
- `MapLink`

## Estados
- **Loading**: skeleton.
- **Sem promoções**: bloco "Sem promoções" + ações de contato/mapa ativas.
- **Error**: toast retry.

## Integração com backend
| Ação | Endpoint (inferido) | Método |
|---|---|---|
| Detalhe parceiro | `/api/v1/neighborhood/partners/{id}?audience=resident` | GET |
| Registrar clique | `/api/v1/neighborhood/partners/{id}/contact-click` | POST |

## Regras de negócio críticas
- Morador vê promoções exclusivas **do próprio condomínio** (cross com [[VS7]]).
- Cliques de contato/maps registram métrica `actor_type=morador`.
- "Ir até o local" abre `geo:` URI ou Google Maps fallback.

## Ligações
- [[VM1]] · [[VM3]] · [[VM5]] · [[_moc]]

## Gaps/ressalvas
- Endpoint REST inferido.
- Provedor de mapa (Mapbox vs Google Maps vs OSM) sem ADR.
- Sem decisão sobre revelação de endereço completo vs aproximado (privacidade do parceiro).
