---
title: "LMS4 — Aula do Curso"
type: ui-screen
tags: [master-sindico, ui-catalog, web, lms, academia, aula, video]
project: master-sindico
persona: any
category: LMS
screen_id: LMS4
sub_produto: lms-academia
plan_requirement: plus
status: specification
stack: web
created: 2026-04-23
---

# LMS4 — Aula do Curso (Player + Material)

## Finalidade
Player de vídeo da aula (Mux HLS via `IVideoProvider`), sidebar com lista de módulos/aulas, resumo, material complementar e botões "Marcar como concluída" / "Próxima aula". Sistema rastreia progresso.

## Fonte canônica
- `vault/90-archive/from-development-content-2026-04-23/contents/MÓDULO ACADEMIA MASTER SÍNDICO.md` §LMS4
- `vault/50-projects/master-sindico/00-product/sub-produtos/06-lms-academia.md` §9 LMS5

## Persona e ABAC
- **Personas**: Síndico (plus+), Empresa (plus+), Morador (pagante), Admin
- **❌ BLOQUEADO**: Parceiro Vizinhança
- **Plan tier mínimo**: `plus` (assistir aulas completas; trial só preview)
- **ABAC action**: `content.lesson.watch`

## Elementos exibidos
- Player de vídeo (Mux HLS, controles, qualidade adaptativa, legendas — M2+)
- Sidebar com lista de módulos/aulas (✓ concluída, ● atual, ○ pendente)
- Resumo da aula (texto)
- Material complementar (PDFs / links)
- Botões: **Marcar aula como concluída** · **Próxima aula**

## Componentes
- `VideoPlayer` (Mux HLS via `IVideoProvider`)
- `LessonSidebar` (collapsible)
- `LessonSummary`
- `MaterialList` (signed URL TTL 24h)
- `CompleteButton`
- `NextLessonButton`

## Estados
- **Loading**: skeleton player.
- **Watching**: player ativo + tracking pulse.
- **Completed**: badge ✓ + Próxima aula automática (com countdown opcional).
- **Last lesson + quiz**: redireciona para Quiz / [[LMS5]] certificado.
- **Sem permissão**: 403 (banner upgrade).
- **Error stream**: toast + fallback qualidade.

## Tracking (CNT-011)
- Pulse frontend cada 15s com `watch_percentage`
- View conta se ≥ 70% assistido

## Integração com backend
| Ação | Endpoint (inferido) | Método |
|---|---|---|
| Detalhe aula | `/api/v1/academy/lessons/{id}` | GET |
| Pulse progresso | `/api/v1/academy/lessons/{id}/progress` | POST `{ watch_percentage }` |
| Marcar concluída | `/api/v1/academy/courses/{cid}/lessons/{lid}/complete` | POST |
| Signed playback URL | (interno via IVideoProvider) | — |

## Regras de negócio críticas
- Signed playback URL Mux: TTL ≤ 24h (INV-070; previne hotlinking).
- Ao marcar concluída: dispara `LessonCompleted` (E-059); se todas aulas do módulo done → libera quiz.
- Progresso 100% no curso → `CourseCompleted` (E-060) → handler `GenerateCertificate` → [[LMS5]].
- Storage MinIO/R2 para materiais; signed URL TTL 24h.

## Ligações
- [[LMS3]] · [[LMS5]] · [[LMS2]] · [[_moc]]
- Sub-produto vídeos: [[../../../../00-product/sub-produtos/04-conteudo-videos]]

## Gaps/ressalvas
- Quiz (LMS6 conceitual) não tem tela própria nesta numeração — embutido como modal pós-último vídeo do módulo.
- Endpoint REST inferido.
- Legendas + transcrições (acessibilidade) backlog M2+.
- Cooldown entre tentativas de quiz: nenhum (ilimitado MVP).
