---
title: "VS9 — Parceiros Indicados pelo Síndico"
type: ui-screen
tags: [master-sindico, ui-catalog, web, vizinhanca, sindico]
project: master-sindico
persona: sindico
category: VS
screen_id: VS9
sub_produto: vizinhanca
plan_requirement: trial
status: specification
stack: web
created: 2026-04-23
---

# VS9 — Parceiros Indicados pelo Síndico

## Finalidade
Listar parceiros que o síndico convidou via [[VS6]] e acompanhar status do funil: convite enviado → parceiro cadastrado → parceiro ativo (publicou primeira promoção).

## Fonte canônica
- `vault/90-archive/from-development-content-2026-04-23/03-Modulos/Vizinhanca-4-jornadas.md` §VS9

## Persona e ABAC
- **Persona**: Síndico (role `syndic`)
- **Plan tier**: `trial`
- **ABAC action**: `commercial.neighborhood_partner.list.invited`

## Status canônicos
- `convite_enviado`
- `parceiro_cadastrado`
- `parceiro_ativo` (já publicou promoção)

## Lista exibida
| Coluna |
|---|
| Nome do estabelecimento |
| Categoria |
| Telefone (do convite) |
| Status (chip) |
| Data do convite |
| Ação ("Reenviar convite" / "Ver perfil" / "Indicar para moradores") |

## Componentes
- `InvitedPartnerTable`
- `StatusChip` (3 cores)
- `BulkResendButton` (reenviar para todos `convite_enviado` antigos)
- `EmptyState`

## Estados
- **Loading**: skeleton.
- **Empty**: "Você ainda não indicou parceiros. Comece em [[VS6]]."
- **All active**: banner verde "Todos os indicados estão ativos."
- **Error**: toast retry.

## Integração com backend
| Ação | Endpoint (inferido) | Método |
|---|---|---|
| Listar indicados | `/api/v1/neighborhood/partners/invited?syndic_id=me` | GET |
| Reenviar | `/api/v1/neighborhood/partners/invite/{id}/resend` | POST |
| Recomendar aos moradores | `/api/v1/neighborhood/partners/{id}/recommend` | POST |

## Regras de negócio críticas
- Status muda automaticamente conforme eventos: `partner.registered`, `partner.first_promotion_published`.
- Sem direito de alterar/deletar manualmente o registro de convite (auditoria).
- "Recomendar para moradores" só ativo quando `parceiro_cadastrado` ou `parceiro_ativo` (não enquanto só convite enviado).
- Métricas administrativas agregam estes números (parceiros cadastrados via síndicos).

## Ligações
- [[VS1]] · [[VS3]] · [[VS6]] · [[VM6]] · [[_moc]]

## Gaps/ressalvas
- Endpoint REST inferido.
- Sem decisão sobre TTL de convite "esquecido" (sugerido 30d → status `expirado` futuro).
- Sem cancelamento manual de convite.
