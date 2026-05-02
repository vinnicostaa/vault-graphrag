---
title: "LMS10 — Replay de Lives"
type: ui-screen
tags: [master-sindico, ui-catalog, web, lms, academia, live, replay]
project: master-sindico
persona: any
category: LMS
screen_id: LMS10
sub_produto: lms-academia
plan_requirement: base
status: specification
stack: web
created: 2026-04-23
---

# LMS10 — Replay de Lives

## Finalidade
Listar replays de lives já transmitidas, disponíveis por 30 dias pós-evento. Ao clicar, abre player VOD do replay.

## Fonte canônica
- `vault/90-archive/from-development-content-2026-04-23/contents/MÓDULO ACADEMIA MASTER SÍNDICO.md` §LMS10

## Persona e ABAC
- **Personas**: Síndico, Empresa, Morador (mesmo `base` para lives abertas), Marketing, Admin
- **❌ BLOQUEADO**: Parceiro Vizinhança
- **Plan tier**: `base` (replays abertos) / `plus`+ (exclusivos)
- **ABAC action**: `content.live.replay.watch`

## Lista exibida
- Tema da live
- Data
- Palestrante
- LiveType
- Acesso (aberta / exclusiva)
- TTL restante (X dias para expirar)

## Botão
- **Assistir replay**

## Componentes
- `ReplayList`
- `ReplayCard`
- `TTLBadge`
- `VideoPlayer` (VOD HLS — quando aberto)
- `EmptyState`

## Estados
- **Loading**: skeleton.
- **Empty**: "Nenhum replay disponível no momento."
- **Watching**: player VOD (controles padrão).
- **Expirado**: card removido (TTL > 30d).
- **Sem permissão (exclusiva)**: 403 com mensagem upgrade.
- **Error**: toast retry.

## Integração com backend
| Ação | Endpoint (inferido) | Método |
|---|---|---|
| Listar replays | `/api/v1/academy/lives/replays` | GET |
| Detalhe replay | `/api/v1/academy/lives/{id}/replay` | GET |

## Regras de negócio críticas
- **Disponível por 30 dias** pós-evento (CNT-023).
- Mesmo ABAC de live ao vivo (Parceiro 403; tier governa exclusivas).
- Replay tem URL signed TTL ≤ 24h (re-issued on demand).
- Provedor pode armazenar VOD nativamente (Mux) ou em MinIO/R2.

## Ligações
- [[LMS8]] · [[LMS9]] · [[_moc]]

## Gaps/ressalvas
- Endpoint REST inferido.
- Pós-30d: archived em storage frio (Glacier?) sem ADR — replay some da UI mas backend mantém.
- Sem decisão sobre permitir compartilhamento público de link de replay.
