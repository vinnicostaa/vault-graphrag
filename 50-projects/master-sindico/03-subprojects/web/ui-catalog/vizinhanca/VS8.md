---
title: "VS8 — Benefícios do Condomínio (Síndico)"
type: ui-screen
tags: [master-sindico, ui-catalog, web, vizinhanca, sindico]
project: master-sindico
persona: sindico
category: VS
screen_id: VS8
sub_produto: vizinhanca
plan_requirement: trial
status: specification
stack: web
created: 2026-04-23
---

# VS8 — Benefícios do Condomínio

## Finalidade
Listar todas as promoções **exclusivas ativas** criadas pelo síndico em parceria com estabelecimentos. Permite editar e encerrar.

## Fonte canônica
- `vault/90-archive/from-development-content-2026-04-23/03-Modulos/Vizinhanca-4-jornadas.md` §VS8

## Persona e ABAC
- **Persona**: Síndico (role `syndic`)
- **Plan tier**: `trial`
- **ABAC action**: `commercial.exclusive_benefit.list.own`

## Lista exibida (canônico)
| Coluna |
|---|
| Promoção (título) |
| Parceiro |
| Validade |
| Ativações (contador) |
| Ações (Editar / Encerrar) |

## Componentes
- `ExclusiveBenefitTable`
- `EditButton` (abre [[VS7]] em modo edit)
- `EndConfirmModal` ("Tem certeza?")
- `EmptyState`

## Estados
- **Loading**: skeleton.
- **Empty**: "Nenhum benefício exclusivo ativo. Criar um?" + CTA [[VS7]].
- **Encerrando**: spinner inline; após confirmação, item some da lista.
- **Error**: toast retry.

## Integração com backend
| Ação | Endpoint (inferido) | Método |
|---|---|---|
| Listar exclusivos do condomínio | `/api/v1/neighborhood/exclusive-benefits?condominium_id=...` | GET |
| Editar | `/api/v1/neighborhood/exclusive-benefits/{id}` | PATCH |
| Encerrar | `/api/v1/neighborhood/exclusive-benefits/{id}/end` | POST |

## Regras de negócio críticas
- Síndico só edita/encerra **promoções que ele mesmo criou**.
- Encerramento é registro **imutável** (Compliance) — promoção fica em status `ended`, mas ativações já registradas permanecem em [[VS10]].
- Métricas de ativação separadas por `actor_type=morador` (esperado o público dessa promoção).
- Edição de campos críticos (tipo, validade) pode requerer aviso de impacto.

## Ligações
- [[VS1]] · [[VS7]] · [[VS10]] · [[VM5]] (visão dos moradores) · [[_moc]]

## Gaps/ressalvas
- Endpoint REST inferido.
- Sem decisão se editar título atualiza promoção exibida aos moradores (atualização imediata vs versionamento).
- Sem auditoria visível de "quem encerrou e quando" (registro existe em backend; não exposto na UI).
