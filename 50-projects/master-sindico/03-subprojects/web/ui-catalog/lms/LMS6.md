---
title: "LMS6 — Treinamentos"
type: ui-screen
tags: [master-sindico, ui-catalog, web, lms, academia, treinamentos]
project: master-sindico
persona: any
category: LMS
screen_id: LMS6
sub_produto: lms-academia
plan_requirement: trial
status: specification
stack: web
created: 2026-04-23
---

# LMS6 — Treinamentos (Lista)

## Finalidade
Lista de treinamentos curtos e práticos (vídeo único, sem módulos): porteiros, prevenção de pragas, segurança, primeiros socorros, atendimento ao morador, etc.

## Fonte canônica
- `vault/90-archive/from-development-content-2026-04-23/contents/MÓDULO ACADEMIA MASTER SÍNDICO.md` §LMS6

## Mensagem institucional canônica
> "Treinamentos rápidos para melhorar a rotina do condomínio."

## Persona e ABAC
- **Personas**: Síndico, Empresa, Morador, Admin
- **❌ BLOQUEADO**: Parceiro Vizinhança
- **Plan tier**: `trial` (catálogo); assistir completo `plus`+
- **ABAC action**: `content.training.list`

## Exemplos canônicos
- Treinamento para porteiros
- Treinamento de prevenção de pragas
- Treinamento de segurança
- Treinamento de primeiros socorros
- Treinamento de atendimento ao morador

## Botão por item
- **Assistir treinamento** → [[LMS7]]

## Componentes
- `TrainingCardGrid`
- `FilterBar` (categoria, duração)
- `EmptyState`

## Estados
- **Loading**: skeleton.
- **Empty**: "Catálogo de treinamentos em construção."
- **Error**: toast retry.

## Integração com backend
| Ação | Endpoint (inferido) | Método |
|---|---|---|
| Listar treinamentos | `/api/v1/academy/trainings` | GET |

## Regras de negócio críticas
- Treinamento = vídeo único, sem módulos, sem quiz, sem certificado por padrão (M3).
- Certificado de treinamento backlog M4+.
- Reaproveita infra de vídeo (`IVideoProvider` Mux).
- Cross-domain: Req 23 — registro de execução de serviço pode recomendar treinamento relacionado.

## Ligações
- [[LMS1]] · [[LMS7]] · [[_moc]]

## Gaps/ressalvas
- Endpoint REST inferido.
- Sem ADR sobre certificação de treinamentos (M4+).
- Categorias de treinamento (independentes das de curso) sem catálogo final.
