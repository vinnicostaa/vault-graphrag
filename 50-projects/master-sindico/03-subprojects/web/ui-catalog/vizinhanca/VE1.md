---
title: "VE1 — Painel da Vizinhança (Empresa)"
type: ui-screen
tags: [master-sindico, ui-catalog, web, vizinhanca, empresa]
project: master-sindico
persona: empresa
category: VE
screen_id: VE1
sub_produto: vizinhanca
plan_requirement: plus
status: specification
stack: web
created: 2026-04-23
---

# VE1 — Painel da Vizinhança (Empresa)

## Finalidade
Hub de entrada do módulo Vizinhança para a empresa prestadora. Empresa pode explorar parceiros e ativar promoções **abertas do bairro** — NÃO vê exclusivas do condomínio.

## Fonte canônica
- `vault/90-archive/from-development-content-2026-04-23/03-Modulos/Vizinhanca-4-jornadas.md` §VE1

## Persona e ABAC
- **Persona**: Empresa (role `enterprise`)
- **Plan tier mínimo**: `plus` (trial vê limitado)
- **ABAC action**: `commercial.neighborhood.read.open_only`
- **Caminho**: Painel Empresarial → Vizinhança

## Seções (canônico — 3)
1. **Explorar parceiros** → [[VE2]]
2. **Promoções disponíveis** (apenas abertas do bairro) → [[VE4]]
3. **Histórico** → [[VE6]]

## Botões
- **Explorar parceiros** → [[VE2]]
- **Ver promoções** (atalho ao feed de promoções abertas)

## Componentes
- `SectionPreviewCard` × 3
- `PrimaryActionGrid`
- `EmptyState`

## Estados
- **Loading**: skeleton.
- **Empty**: foco em explorar parceiros.
- **Populated**: previews + KPIs simples (parceiros próximos / promoções ativas).
- **Error**: toast retry.

## Integração com backend
| Ação | Endpoint (inferido) | Método |
|---|---|---|
| Resumo painel empresa | `/api/v1/neighborhood/enterprise/dashboard` | GET (filter open_only) |

## Regras de negócio críticas
- **Empresa NÃO vê promoções exclusivas** (regra dura — derivada para [[VE4]] e contraparte de [[VS7]]).
- Métrica de ativação registrada com `actor_type=empresa`.
- Acesso via Painel Empresarial (não navegação geral).

## Ligações
- [[VE2]] · [[VE3]] · [[VE4]] · [[VE5]] · [[VE6]] · [[_moc]]
- Sub-produto: [[../../../../00-product/sub-produtos/08-vizinhanca]]

## Gaps/ressalvas
- Endpoint REST inferido.
- Sem ADR sobre tier mínimo (sugerido `plus` — trial pode ter view-only?).
- Sem decisão sobre KPIs específicos da empresa (engajamento da própria carteira de condomínios).
