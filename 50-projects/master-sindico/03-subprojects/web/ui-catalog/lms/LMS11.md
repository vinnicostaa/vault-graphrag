---
title: "LMS11 — Fórum"
type: ui-screen
tags: [master-sindico, ui-catalog, web, lms, academia, forum]
project: master-sindico
persona: any
category: LMS
screen_id: LMS11
sub_produto: lms-academia
plan_requirement: base
status: specification
stack: web
created: 2026-04-23
---

# LMS11 — Fórum

## Finalidade
Espaço discursivo do ecossistema condominial em 7 categorias. Síndicos e empresas criam tópicos; moradores apenas comentam.

## Fonte canônica
- `vault/90-archive/from-development-content-2026-04-23/contents/MÓDULO ACADEMIA MASTER SÍNDICO.md` §LMS11
- `vault/50-projects/master-sindico/00-product/sub-produtos/06-lms-academia.md` §9 LMS8

## Mensagem institucional canônica
> "Compartilhe experiências, faça perguntas e construa conhecimento coletivo."

## Persona e ABAC
- **Personas (leitura/comentário)**: Síndico, Empresa, Morador (`plus`+ para comentar idealmente — matriz §6), Marketing, Admin
- **❌ BLOQUEADO**: Parceiro Vizinhança (INV-067)
- **Criar tópico**: **APENAS** `syndic`, `enterprise`, `admin` (matriz canônica) — moradores **não criam**
- **Plan tier**: `base` (ler); criar tópico exige `plus`+

## 7 Categorias canônicas (`ForumCategory`)
- gestão_condominial
- manutenção_predial
- segurança
- saúde_ambiental
- tecnologia
- convivência_moradores
- legislação_condominial

## Elementos
- Abas por categoria
- Lista de tópicos: título, autor, última resposta, curtidas, status (aberto/resolvido)
- Busca full-text

## Botão
- **Criar tópico** (visível apenas a `syndic | enterprise | admin`) → [[LMS12]]

## Componentes
- `CategoryTabs` (7)
- `TopicList`
- `SearchBar`
- `CreateTopicButton` (condicional)
- `EmptyState`

## Estados
- **Loading**: skeleton.
- **Empty (categoria)**: "Nenhum tópico nesta categoria."
- **Empty (busca)**: "Nenhum resultado."
- **Sem permissão**: 403 (Parceiro Vizinhança).
- **Error**: toast retry.

## Integração com backend
| Ação | Endpoint (inferido) | Método |
|---|---|---|
| Listar tópicos | `/api/v1/academy/forum?category=...` | GET |
| Busca | `/api/v1/academy/forum?q=...` | GET |

## Regras de negócio críticas
- **INV-LMS-E**: moradores **não criam tópicos** (apenas comentam).
- Empresa também cria — fonte canônica destaca: empresas têm conhecimento técnico, contribuem com soluções.
- Moderação reativa via "Denunciar" → fila admin (CNT-027).
- Notificação push/email a quem subscreveu o tópico (CNT-028).
- Busca full-text: `ISearchProvider` real (OpenSearch ou Meilisearch — ADR-0042 pending; D-120 Fase 14 revoga D-067 PG tsvector).

## Ligações
- [[LMS1]] · [[LMS12]] · [[LMS13]] · [[_moc]]

## Gaps/ressalvas
- Endpoint REST inferido.
- Aggregate `ForumTopic`/`ForumPost` ausente em `01-domain/aggregates/` (G-01).
- Eventos `ForumTopicCreated`, `ForumReplyPosted` não formalizados (G-04).
- SLA de moderação não definido (G-06).
