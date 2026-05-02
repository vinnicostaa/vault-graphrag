---
title: "VZ4 — Promoções Ativas (Parceiro)"
type: ui-screen
tags: [master-sindico, ui-catalog, web, vizinhanca, parceiro]
project: master-sindico
persona: parceiro
category: VZ
screen_id: VZ4
sub_produto: vizinhanca
plan_requirement: trial
status: specification
stack: web
created: 2026-04-23
---

# VZ4 — Promoções Ativas (Parceiro)

## Finalidade
Listar promoções ativas do parceiro. Mostra título, tipo, validade, visualizações e ativações. Permite editar e encerrar.

## Fonte canônica
- `vault/90-archive/from-development-content-2026-04-23/03-Modulos/Vizinhanca-4-jornadas.md` §VZ4
- `vault/90-archive/master-sindico-pdfs-cliente-originais/Jornada do Parceiro da Vizinhança.md` §TELA VZ4

## Persona e ABAC
- **Persona**: Parceiro (role `local_company`)
- **Plan tier**: `trial`
- **ABAC action**: `commercial.neighborhood_benefit.list.own`

## Lista exibida (canônico)
| Coluna |
|---|
| Título da promoção |
| Tipo de promoção |
| Validade |
| Visualizações (contador) |
| Ativações (contador agregado) |
| Ações (Editar / Encerrar) |

## Componentes
- `PromotionTable`
- `EditButton` (abre [[VZ3]] em edit)
- `EndConfirmModal`
- `EmptyState`

## Estados
- **Loading**: skeleton.
- **Empty**: "Nenhuma promoção ativa" + CTA [[VZ3]].
- **Encerrando**: spinner; após confirmação some da lista.
- **Error**: toast retry.

## Integração com backend
| Ação | Endpoint (inferido) | Método |
|---|---|---|
| Listar próprias | `/api/v1/neighborhood/promotions?partner_id=me&status=active` | GET |
| Editar | `/api/v1/neighborhood/promotions/{id}` | PATCH |
| Encerrar | `/api/v1/neighborhood/promotions/{id}/end` | POST |

## Regras de negócio críticas
- Parceiro só vê e gerencia **as próprias** promoções.
- Ativações exibidas são **soma** dos 3 actor_types (morador + síndico + empresa); discriminação fica em [[VZ5]] métricas.
- Encerrar é registro auditável; promoção fica `ended` mas ativações já registradas permanecem.
- Edição de campos críticos pode invalidar cupons emitidos? — pendência.

## Ligações
- [[VZ2]] · [[VZ3]] · [[VZ5]] · [[VZ6]] · [[_moc]]

## Gaps/ressalvas
- Endpoint REST inferido.
- Sem decisão sobre impacto de edição em cupons já gerados (segurança vs flexibilidade).
- Visualizações vs ativações: definição precisa de "visualização" (impressão de card vs abertura de detalhe).
