---
title: "VE2 — Explorar Parceiros (Empresa)"
type: ui-screen
tags: [master-sindico, ui-catalog, web, vizinhanca, empresa]
project: master-sindico
persona: empresa
category: VE
screen_id: VE2
sub_produto: vizinhanca
plan_requirement: plus
status: specification
stack: web
created: 2026-04-23
---

# VE2 — Explorar Parceiros (Empresa)

## Finalidade
Feed de parceiros da vizinhança para a empresa prestadora, com filtros por categoria e bairro. Igual ao VS2 do síndico em estrutura, mas o backend filtra apenas promoções **abertas** (sem exclusivas).

## Fonte canônica
- `vault/90-archive/from-development-content-2026-04-23/03-Modulos/Vizinhanca-4-jornadas.md` §VE2

## Persona e ABAC
- **Persona**: Empresa (role `enterprise`)
- **Plan tier**: `plus`
- **ABAC action**: `commercial.neighborhood_partner.list`

## Card do parceiro
- Nome
- Categoria
- Bairro
- Indicador "Promoção ativa" (apenas se houver **aberta**)
- Botão "Ver detalhes" → [[VE3]]

## Filtros
- Categoria (multi-select)
- Bairro (autocomplete)

## Componentes
- `PartnerCardGrid`
- `FilterBar`
- `Pagination`
- `EmptyState`

## Estados
- **Loading**: skeleton 9 cards.
- **Empty**: "Nenhum parceiro com esses filtros."
- **Error**: toast retry.

## Integração com backend
| Ação | Endpoint (inferido) | Método |
|---|---|---|
| Listar parceiros | `/api/v1/neighborhood/partners?audience=enterprise` | GET |

## Regras de negócio críticas
- Empresa **NÃO vê** promoções exclusivas — backend filtra (`audience=enterprise` ou ABAC enforce).
- Cards podem reutilizar componente de [[VS2]], mas com flag de filtro.
- Sem CTA "indicar para moradores" (essa ação é exclusiva do síndico).

## Ligações
- [[VE1]] · [[VE3]] · [[_moc]]

## Gaps/ressalvas
- Endpoint REST inferido.
- Sem decisão sobre filtro adicional "por carteira de condomínios da empresa".
- Lista mestra de categorias 38+ sem catálogo final.
