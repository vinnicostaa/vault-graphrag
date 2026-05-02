---
title: "VM1 — Feed do Bairro (Morador)"
type: ui-screen
tags: [master-sindico, ui-catalog, web, vizinhanca, morador]
project: master-sindico
persona: morador
category: VM
screen_id: VM1
sub_produto: vizinhanca
plan_requirement: base
status: specification
stack: web
created: 2026-04-23
---

# VM1 — Feed do Bairro (Morador)

## Finalidade
Feed prioritário de parceiros da vizinhança para o morador. Ordenação canônica: parceiros do mesmo bairro + promoções ativas + parceiros indicados pelo síndico.

## Fonte canônica
- `vault/90-archive/from-development-content-2026-04-23/03-Modulos/Vizinhanca-4-jornadas.md` §VM1
- `vault/90-archive/master-sindico-pdfs-cliente-originais/MÓDULO VIZINHANÇA - MORADOR.md` §TELA VM1

## Mensagem institucional canônica
> "Conheça estabelecimentos do seu bairro que oferecem benefícios para moradores da comunidade."

## Persona e ABAC
- **Persona**: Morador (role `resident`)
- **Plan tier**: `base` (acesso ao feed liberado a base)
- **ABAC action**: `commercial.neighborhood.read.resident`
- **Caminho**: Painel do Morador → Vizinhança

## Card do parceiro
- Nome do estabelecimento
- Categoria
- Bairro
- Promoção ativa (se houver)
- **Badge "Indicado pelo Síndico"** (quando aplicável — diferenciação canônica)

## Filtros
- Categoria (multi-select)

## Ordenação canônica do feed
1. `same_neighborhood` (mesmo bairro do condomínio)
2. `active_promotions` (promoções ativas)
3. `sindico_indicated_partners` (badge sindico)

## Componentes
- `PartnerFeedList` (priorizado)
- `CategoryFilter`
- `SyndicBadge` (chip distintivo)
- `EmptyState`

## Estados
- **Loading**: skeleton 9 cards.
- **Empty**: "Sem parceiros na sua região ainda. Volte em breve."
- **Filtered**: contagem visível.
- **Error**: toast retry.

## Integração com backend
| Ação | Endpoint (inferido) | Método |
|---|---|---|
| Feed | `/api/v1/neighborhood/feed?audience=resident` | GET |

## Regras de negócio críticas
- Morador vê **abertas do bairro + exclusivas do próprio condomínio + indicados pelo síndico**.
- Promoções **exclusivas** do **próprio condomínio** aparecem aqui (regra de visibilidade rígida).
- Promoções exclusivas de **outros condomínios** NÃO aparecem.
- Métrica `actor_type=morador`.

## Ligações
- [[VM2]] · [[VM5]] · [[VM6]] · [[_moc]]
- Sub-produto: [[../../../../00-product/sub-produtos/08-vizinhanca]]

## Gaps/ressalvas
- Endpoint REST inferido.
- Sem ADR sobre algoritmo exato de ranking (peso de cada critério).
- Sem decisão sobre paginação infinita vs paginated.
