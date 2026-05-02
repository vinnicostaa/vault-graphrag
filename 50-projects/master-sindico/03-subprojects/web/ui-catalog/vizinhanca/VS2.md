---
title: "VS2 — Explorar Parceiros (Síndico)"
type: ui-screen
tags: [master-sindico, ui-catalog, web, vizinhanca, sindico]
project: master-sindico
persona: sindico
category: VS
screen_id: VS2
sub_produto: vizinhanca
plan_requirement: trial
status: specification
stack: web
created: 2026-04-23
---

# VS2 — Explorar Parceiros (Síndico)

## Finalidade
Feed de parceiros da vizinhança visível ao síndico, com filtros por categoria e bairro. Card resumido com nome, categoria, bairro e indicador de promoção ativa.

## Fonte canônica
- `vault/90-archive/from-development-content-2026-04-23/03-Modulos/Vizinhanca-4-jornadas.md` §VS2
- `vault/90-archive/master-sindico-pdfs-cliente-originais/MÓDULO VIZINHANÇA - MORADOR.md` (lista mestra de categorias)

## Persona e ABAC
- **Persona**: Síndico (role `syndic`)
- **Plan tier**: `trial`
- **ABAC action**: `commercial.neighborhood_partner.list`

## Card do parceiro
- Nome do estabelecimento
- Categoria
- Bairro
- Indicador "Promoção ativa" (badge se houver)
- Botão "Ver detalhes" → [[VS3]]

## Filtros
- **Categoria** (dropdown / multi-select — 38+ categorias canônicas)
- **Bairro** (autocomplete sobre bairros da região)

## Componentes
- `PartnerCardGrid` (responsive 3-col desktop, 1-col mobile)
- `FilterBar` (Categoria + Bairro + busca free text)
- `Pagination` (server-side)
- `EmptyState`

## Estados
- **Loading**: skeleton de 9 cards.
- **Empty (filtros)**: "Nenhum parceiro com esses filtros."
- **Empty (geral)**: "Nenhum parceiro cadastrado na sua região ainda. Que tal indicar um?" + CTA [[VS6]].
- **Error**: toast com retry; preserva filtros.

## Integração com backend
| Ação | Endpoint (inferido) | Método |
|---|---|---|
| Listar parceiros | `/api/v1/neighborhood/partners` | GET (filtros, paginação) |
| Filtros disponíveis | `/api/v1/neighborhood/partners/filters` | GET |

## Regras de negócio críticas
- Síndico vê **parceiros do bairro** do condomínio + parceiros indicados (próprios e gerais).
- Filtragem por bairro respeita a região do condomínio em foco.
- Card NÃO revela contato direto; isso fica em [[VS3]].

## Ligações
- [[VS1]] · [[VS3]] · [[VS6]] · [[_moc]]
- Sub-produto: [[../../../../00-product/sub-produtos/08-vizinhanca]]

## Gaps/ressalvas
- Endpoint REST inferido.
- Lista mestra de categorias (38+) sem catálogo final unificado.
- Sem decisão sobre raio geográfico (filtro por km).
