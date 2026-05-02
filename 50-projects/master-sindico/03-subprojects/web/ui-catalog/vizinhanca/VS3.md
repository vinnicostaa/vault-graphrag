---
title: "VS3 — Detalhe do Parceiro (Síndico)"
type: ui-screen
tags: [master-sindico, ui-catalog, web, vizinhanca, sindico]
project: master-sindico
persona: sindico
category: VS
screen_id: VS3
sub_produto: vizinhanca
plan_requirement: trial
status: specification
stack: web
created: 2026-04-23
---

# VS3 — Detalhe do Parceiro (Síndico)

## Finalidade
Página de detalhe de um parceiro da vizinhança com dados completos: nome, categoria, endereço, telefone, descrição, lista de promoções disponíveis. Permite ao síndico ver promoção, entrar em contato e indicar para moradores.

## Fonte canônica
- `vault/90-archive/from-development-content-2026-04-23/03-Modulos/Vizinhanca-4-jornadas.md` §VS3
- `vault/90-archive/master-sindico-pdfs-cliente-originais/MÓDULO VIZINHANÇA - MORADOR.md` §TELA VM2 (referência paralela)

## Persona e ABAC
- **Persona**: Síndico (role `syndic`)
- **Plan tier**: `trial`
- **ABAC action**: `commercial.neighborhood_partner.read`

## Informações exibidas
- Nome do estabelecimento
- Categoria (+ subcategoria se houver)
- Endereço completo
- Telefone / WhatsApp
- Descrição
- **Promoções disponíveis** (lista — abertas + exclusivas que o síndico pode ver)

## Botões
- **Ver promoção** → [[VS4]]
- **Entrar em contato** (abre WhatsApp/telefone)
- **Indicar para moradores** (push aos moradores do condomínio com badge "Indicado pelo Síndico")

## Componentes
- `PartnerHero` (nome + categoria + foto/logo se houver)
- `PartnerInfoBlock` (endereço, contatos, descrição)
- `PromotionList` (cards das promoções disponíveis)
- `ActionGroup` (3 botões)
- `MapPreview` (mini-mapa do endereço — opcional)

## Estados
- **Loading**: skeleton.
- **Sem promoções**: bloco "Sem promoções no momento" + ações de contato/indicação ativas.
- **Indicação enviada**: toast + botão muda para "Indicado em DD/MM" (não permite duplicar no mesmo dia).
- **Error**: toast com retry.

## Integração com backend
| Ação | Endpoint (inferido) | Método |
|---|---|---|
| Detalhe parceiro | `/api/v1/neighborhood/partners/{id}` | GET |
| Indicar aos moradores | `/api/v1/neighborhood/partners/{id}/recommend` | POST |
| Registrar clique de contato | `/api/v1/neighborhood/partners/{id}/contact-click` | POST |

## Regras de negócio críticas
- Síndico vê **promoções abertas** (bairro) + **exclusivas do próprio condomínio** (criadas por ele em [[VS7]]).
- Recomendar gera notificação aos moradores e adiciona o parceiro a [[VS9]] e [[VM6]] com badge "Indicado pelo Síndico".
- Cliques de contato geram métricas do parceiro (`actor_type=sindico`).

## Ligações
- [[VS2]] · [[VS4]] · [[VS6]] · [[VS9]] · [[VM6]] · [[_moc]]
- Connect Me Síndico → Parceiro (Req 19.3): [[../../../../00-product/sub-produtos/02-connect-me]]

## Gaps/ressalvas
- Endpoint REST inferido.
- Sem regra de cooldown da indicação (pode indicar mesmo parceiro 5x?).
- Mini-mapa: provedor não decidido (Mapbox vs Leaflet/OSM).
