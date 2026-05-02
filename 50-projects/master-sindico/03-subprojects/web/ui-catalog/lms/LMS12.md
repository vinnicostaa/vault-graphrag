---
title: "LMS12 — Criar Tópico (Fórum)"
type: ui-screen
tags: [master-sindico, ui-catalog, web, lms, academia, forum]
project: master-sindico
persona: any
category: LMS
screen_id: LMS12
sub_produto: lms-academia
plan_requirement: plus
status: specification
stack: web
created: 2026-04-23
---

# LMS12 — Criar Tópico do Fórum

## Finalidade
Form para criar um tópico do fórum: título, categoria (1 das 7 canônicas), descrição.

## Fonte canônica
- `vault/90-archive/from-development-content-2026-04-23/contents/MÓDULO ACADEMIA MASTER SÍNDICO.md` §LMS12

## Persona e ABAC
- **Personas autorizadas**: `syndic | enterprise | admin` — **moradores NÃO criam**
- **❌ BLOQUEADO**: Parceiro Vizinhança, Morador (Base/Pagante)
- **Plan tier**: `plus`+ (admin bypass)
- **ABAC action**: `content.forum.topic.create`

## Campos
| Campo | Tipo | Notas |
|---|---|---|
| Título da discussão | Texto | Limite ~200 chars |
| Categoria | Select | 1 das 7 (`ForumCategory`) |
| Descrição | Textarea (markdown limitado) | Limite ~5000 chars |

## Botão
- **Publicar**

## Componentes
- `TopicForm`
- `CategorySelect`
- `MarkdownEditor` (limitado: bold, italic, listas, links)
- `PrimaryButton`

## Estados
- **Inicial**: form vazio.
- **Validating**: inline.
- **Submitting**: spinner.
- **Success**: redireciona [[LMS13]] discussão recém-criada + toast.
- **Sem permissão**: 403 com mensagem "Apenas síndicos, empresas e admin podem criar tópicos."
- **Error**: toast retry; preserva conteúdo.

## Integração com backend
| Ação | Endpoint (inferido) | Método |
|---|---|---|
| Criar tópico | `/api/v1/academy/forum` | POST `{ title, category, description }` |

## Regras de negócio críticas
- **INV-LMS-E**: morador (qualquer tier) não cria. Tela 403 com link para [[LMS11]].
- Tópico criado entra em moderação automática? — não no MVP (moderação reativa via Denunciar).
- Empresas também podem criar — observação canônica destaca valor técnico.

## Ligações
- [[LMS11]] · [[LMS13]] · [[_moc]]

## Gaps/ressalvas
- Endpoint REST inferido.
- Editor markdown: provedor (TipTap, Lexical, ProseMirror?) sem ADR.
- Limite de tamanho não definido oficialmente.
- Sem rate limit de criação (anti-spam).
