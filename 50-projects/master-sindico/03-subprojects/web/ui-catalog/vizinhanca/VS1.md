---
title: "VS1 — Painel da Vizinhança (Síndico)"
type: ui-screen
tags: [master-sindico, ui-catalog, web, vizinhanca, sindico]
project: master-sindico
persona: sindico
category: VS
screen_id: VS1
sub_produto: vizinhanca
plan_requirement: trial
status: specification
stack: web
created: 2026-04-23
---

# VS1 — Painel da Vizinhança (Síndico)

## Finalidade
Hub de entrada do módulo Vizinhança para o síndico. Reúne 4 seções (explorar parceiros, promoções exclusivas do condomínio, parceiros indicados, histórico) e CTAs para ações de gestão (indicar parceiro, criar benefício exclusivo).

## Fonte canônica
- `vault/90-archive/from-development-content-2026-04-23/03-Modulos/Vizinhanca-4-jornadas.md` §VS1
- `vault/50-projects/master-sindico/00-product/sub-produtos/08-vizinhanca.md`
- `vault/50-projects/master-sindico/client-canvas/Fluxo Vizinhanca.md`

## Persona e ABAC
- **Persona**: Síndico (role `syndic`)
- **Plan tier mínimo**: `trial`
- **ABAC action**: `commercial.neighborhood.read`
- **Caminho**: Governança → Vizinhança

## Seções (canônico — 4)
1. **Explorar parceiros** (preview + link [[VS2]])
2. **Promoções exclusivas do condomínio** (preview + link [[VS8]])
3. **Parceiros indicados pelo síndico** (preview + link [[VS9]])
4. **Histórico de promoções ativadas** (preview + link [[VS10]])

## Botões
- **Explorar parceiros** → [[VS2]]
- **Indicar parceiro local** → [[VS6]]
- **Criar benefício exclusivo** → [[VS7]]

## Componentes
- `SectionPreviewCard` × 4 (cada com 2-3 itens + "Ver tudo")
- `PrimaryActionGrid` (3 botões CTA)
- `EmptyState` (se sem promoções/indicações)

## Estados
- **Loading**: skeleton de 4 cards.
- **Empty geral**: mensagem institucional + foco no CTA "Indicar parceiro".
- **Populated**: cards com último item de cada seção.
- **Error**: toast com retry.

## Integração com backend
| Ação | Endpoint (inferido) | Método |
|---|---|---|
| Resumo do painel | `/api/v1/neighborhood/syndic/dashboard` | GET |

## Regras de negócio críticas
- Acessível a partir de Governança → Vizinhança (não navegação principal).
- Síndico vê **promoções exclusivas que ele criou** + abertas do bairro do condomínio.
- Métricas separadas por `actor_type` no backend (sindico).

## Ligações
- [[VS2]] · [[VS6]] · [[VS7]] · [[VS8]] · [[VS9]] · [[VS10]] · [[_moc]]
- Sub-produto: [[../../../../00-product/sub-produtos/08-vizinhanca]]

## Gaps/ressalvas
- Endpoint REST inferido.
- Sem ADR sobre quantos itens preview exibir por seção.
- Sem decisão sobre painel multi-condomínio (síndico de mais de um).
