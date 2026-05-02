---
title: "LMS13 — Discussão do Fórum"
type: ui-screen
tags: [master-sindico, ui-catalog, web, lms, academia, forum, discussao]
project: master-sindico
persona: any
category: LMS
screen_id: LMS13
sub_produto: lms-academia
plan_requirement: base
status: specification
stack: web
created: 2026-04-23
---

# LMS13 — Discussão do Fórum

## Finalidade
Tela do tópico aberto: pergunta original + respostas + curtidas. Permite responder, curtir, marcar como resolvido (autor ou admin) e denunciar.

## Fonte canônica
- `vault/90-archive/from-development-content-2026-04-23/contents/MÓDULO ACADEMIA MASTER SÍNDICO.md` §LMS13

## Persona e ABAC
- **Personas**: Síndico, Empresa, Morador (`base`+ ler; `plus` comentar idealmente — matriz §6), Marketing, Admin
- **❌ BLOQUEADO**: Parceiro Vizinhança
- **Plan tier**: `base` ler; `plus` comentar
- **ABAC action**: `content.forum.topic.read` · `content.forum.reply.create` · `content.forum.like.create`

## Elementos exibidos
- Pergunta (título, descrição, autor, data, categoria)
- Respostas (lista paginada com curtidas)
- Status do tópico (aberto / resolvido)

## Botões
- **Responder** (`plus`+) — moradores podem comentar/responder
- **Curtir** (todas as personas autorizadas)
- **Marcar como resolvido** (autor do tópico ou admin)
- **Denunciar** (botão discreto em cada resposta)
- **Subscrever** (notificação para novas respostas)

## Componentes
- `TopicHeader`
- `ReplyList`
- `ReplyComposer` (markdown limitado)
- `LikeButton`
- `ResolveButton` (condicional)
- `ReportButton`
- `SubscribeToggle`

## Estados
- **Loading**: skeleton.
- **Empty (sem respostas)**: "Seja o primeiro a responder."
- **Resolved**: badge verde + bloqueio de novas respostas (configurável).
- **Sem permissão**: 403 (Parceiro).
- **Error**: toast retry.

## Integração com backend
| Ação | Endpoint (inferido) | Método |
|---|---|---|
| Detalhe tópico | `/api/v1/academy/forum/{id}` | GET |
| Responder | `/api/v1/academy/forum/{id}/reply` | POST |
| Curtir | `/api/v1/academy/forum/{id}/like` | POST |
| Marcar resolvido | `/api/v1/academy/forum/{id}/resolve` | POST |
| Denunciar | `/api/v1/academy/forum/{id}/report` | POST |
| Subscrever | `/api/v1/academy/forum/{id}/subscribe` | POST |

## Regras de negócio críticas
- Moderação reativa via Denunciar (CNT-027).
- Notificação a quem subscreveu quando nova resposta (CNT-028).
- Marcar resolvido: autor ou admin.
- Moradores: podem **comentar/responder** mas não criam tópicos (INV-LMS-E é para CRIAR; comentar ok com `plus`).

## Ligações
- [[LMS11]] · [[LMS12]] · [[_moc]]

## Gaps/ressalvas
- Endpoint REST inferido.
- Aggregate `ForumTopic`/`ForumPost` ausente (G-01).
- Eventos de domínio do fórum não formalizados (G-04).
- SLA de moderação reativa sem ADR (G-06).
- Markdown editor sem provedor escolhido.
