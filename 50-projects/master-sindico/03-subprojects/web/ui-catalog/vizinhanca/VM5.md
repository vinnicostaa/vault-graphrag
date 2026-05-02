---
title: "VM5 — Benefícios do Condomínio (Morador)"
type: ui-screen
tags: [master-sindico, ui-catalog, web, vizinhanca, morador, exclusivo]
project: master-sindico
persona: morador
category: VM
screen_id: VM5
sub_produto: vizinhanca
plan_requirement: base
status: specification
stack: web
created: 2026-04-23
---

# VM5 — Benefícios do meu Condomínio (Morador)

## Finalidade
Lista das **promoções exclusivas** criadas pelo síndico em parceria com estabelecimentos para o condomínio do morador. Aparece apenas se houver exclusivas vigentes.

## Fonte canônica
- `vault/90-archive/from-development-content-2026-04-23/03-Modulos/Vizinhanca-4-jornadas.md` §VM5
- `vault/90-archive/master-sindico-pdfs-cliente-originais/MÓDULO VIZINHANÇA - MORADOR.md` §TELA VM5

## Mensagem institucional canônica
> "Alguns parceiros oferecem benefícios exclusivos para moradores do seu condomínio."

## Persona e ABAC
- **Persona**: Morador (role `resident`)
- **Plan tier**: `base`
- **ABAC action**: `commercial.exclusive_benefit.view` (com escopo `unit.condominium_id == benefit.condominium_id`)
- **Caminho**: Vizinhança → Benefícios do meu condomínio

## Lista exibida (canônico)
| Coluna |
|---|
| Promoção |
| Parceiro |
| Validade |

## Componentes
- `ExclusiveBenefitList`
- `InstitutionalNote` (mensagem fixa)
- `EmptyState`

## Estados
- **Loading**: skeleton.
- **Empty (canônico)**: "Aparece apenas se houver promoções exclusivas." — esconde a entrada/menu se nenhuma promoção exclusiva ativa.
- **Populated**: lista cards.
- **Error**: toast retry.

## Integração com backend
| Ação | Endpoint (inferido) | Método |
|---|---|---|
| Listar exclusivas do condomínio | `/api/v1/neighborhood/exclusive-benefits?condominium_id=mine` | GET |

## Regras de negócio críticas
- ABAC enforce: morador só vê promoções **do próprio condomínio**.
- Outros condomínios: 403 mesmo com URL direta.
- Empresa NÃO vê esta tela.
- Síndicos de outros condomínios NÃO veem.
- Métrica `actor_type=morador` quando ativa.

## Ligações
- [[VM1]] · [[VM3]] · [[VS7]] (criado por síndico) · [[VS8]] (visão do síndico) · [[_moc]]

## Gaps/ressalvas
- Endpoint REST inferido.
- Sem ADR sobre persistência da entrada do menu (esconder vs sempre mostrar com EmptyState).
- Sem decisão sobre push automático ao morador quando síndico cria exclusiva nova.
