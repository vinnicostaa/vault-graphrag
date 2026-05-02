---
title: "VE3 — Detalhe do Parceiro (Empresa)"
type: ui-screen
tags: [master-sindico, ui-catalog, web, vizinhanca, empresa]
project: master-sindico
persona: empresa
category: VE
screen_id: VE3
sub_produto: vizinhanca
plan_requirement: plus
status: specification
stack: web
created: 2026-04-23
---

# VE3 — Detalhe do Parceiro (Empresa)

## Finalidade
Página de detalhe de um parceiro com dados completos. Empresa vê **somente promoções abertas** desse parceiro. NÃO acessa exclusivas do condomínio.

## Fonte canônica
- `vault/90-archive/from-development-content-2026-04-23/03-Modulos/Vizinhanca-4-jornadas.md` §VE3

## Persona e ABAC
- **Persona**: Empresa (role `enterprise`)
- **Plan tier**: `plus`
- **ABAC action**: `commercial.neighborhood_partner.read`

## Informações exibidas
- Nome do estabelecimento
- Categoria
- Endereço
- Telefone / WhatsApp
- Descrição
- **Promoções disponíveis** (apenas abertas)

## Botões
- **Ver promoção** → [[VE4]]
- **Entrar em contato** (registra clique como `actor_type=empresa`)

## Componentes
- `PartnerHero`
- `PartnerInfoBlock`
- `PromotionList` (apenas abertas)
- `ActionGroup` (2 botões — sem "indicar para moradores", privilégio do síndico)

## Estados
- **Loading**: skeleton.
- **Sem promoções**: bloco "Sem promoções abertas" + ação contato.
- **Error**: toast retry.

## Integração com backend
| Ação | Endpoint (inferido) | Método |
|---|---|---|
| Detalhe parceiro | `/api/v1/neighborhood/partners/{id}?audience=enterprise` | GET |
| Registrar clique contato | `/api/v1/neighborhood/partners/{id}/contact-click` | POST (com actor_type derivado da auth) |

## Regras de negócio críticas
- Backend filtra promoções por `audience=enterprise` (= apenas abertas).
- Empresa NÃO recebe nenhuma promoção exclusiva nesta tela.
- Métricas de view/click registradas com `actor_type=empresa` (separação canônica).
- Sem ação "indicar para moradores" (privilégio do síndico em [[VS3]]).

## Ligações
- [[VE2]] · [[VE4]] · [[_moc]]

## Gaps/ressalvas
- Endpoint REST inferido.
- Sem ADR sobre se empresa vê "Indicado pelo Síndico" badge ou se isso é só para moradores.
- Cross-validação: garantir que o filtro `audience` é enforce em ABAC e não só na query (defesa em profundidade).
