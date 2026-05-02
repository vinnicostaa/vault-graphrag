---
title: "LMS7 — Página do Treinamento"
type: ui-screen
tags: [master-sindico, ui-catalog, web, lms, academia, treinamento]
project: master-sindico
persona: any
category: LMS
screen_id: LMS7
sub_produto: lms-academia
plan_requirement: plus
status: specification
stack: web
created: 2026-04-23
---

# LMS7 — Página do Treinamento

## Finalidade
Player do treinamento (vídeo único) + descrição + objetivo + botão "Marcar como concluído".

## Fonte canônica
- `vault/90-archive/from-development-content-2026-04-23/contents/MÓDULO ACADEMIA MASTER SÍNDICO.md` §LMS7

## Persona e ABAC
- **Personas**: Síndico, Empresa, Morador, Admin
- **❌ BLOQUEADO**: Parceiro Vizinhança
- **Plan tier**: `plus`+ para assistir completo
- **ABAC action**: `content.training.watch`

## Elementos exibidos
- Vídeo (Mux HLS via `IVideoProvider`)
- Descrição
- Objetivo do treinamento
- Material complementar (opcional)

## Botão
- **Marcar como concluído**

## Componentes
- `VideoPlayer` (Mux HLS)
- `TrainingHero`
- `TrainingDescription`
- `MaterialList`
- `CompleteButton`

## Estados
- **Loading**: skeleton.
- **Watching**: tracking pulse (mesmo modelo de [[LMS4]]).
- **Completed**: badge ✓.
- **Sem permissão**: 403 + banner upgrade.
- **Error**: toast retry.

## Integração com backend
| Ação | Endpoint (inferido) | Método |
|---|---|---|
| Detalhe | `/api/v1/academy/trainings/{id}` | GET |
| Marcar concluído | `/api/v1/academy/trainings/{id}/complete` | POST |

## Regras de negócio críticas
- Sem quiz, sem certificado (M3 — backlog M4+).
- Sistema registra progresso e gera evento `TrainingCompleted` (sugerido — não formalizado em domain-events).
- Reaproveita infra Mux (signed URL TTL 24h).

## Ligações
- [[LMS6]] · [[LMS4]] (similar player) · [[_moc]]

## Gaps/ressalvas
- Endpoint REST inferido.
- Evento `TrainingCompleted` não formalizado em `domain-events.md` (gap G-04).
- Certificado de treinamento backlog M4+.
