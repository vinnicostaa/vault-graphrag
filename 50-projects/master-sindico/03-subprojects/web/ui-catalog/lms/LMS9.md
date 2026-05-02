---
title: "LMS9 — Sala da Live"
type: ui-screen
tags: [master-sindico, ui-catalog, web, lms, academia, live]
project: master-sindico
persona: any
category: LMS
screen_id: LMS9
sub_produto: lms-academia
plan_requirement: base
status: specification
stack: web
created: 2026-04-23
---

# LMS9 — Sala da Live

## Finalidade
Sala de transmissão ao vivo: vídeo + chat + campo de perguntas ao palestrante. Após o evento, replay fica disponível por 30 dias em [[LMS10]].

## Fonte canônica
- `vault/90-archive/from-development-content-2026-04-23/contents/MÓDULO ACADEMIA MASTER SÍNDICO.md` §LMS9

## Persona e ABAC
- **Personas**: Síndico, Empresa, Morador, Marketing, Admin (conforme tier e access_level)
- **❌ BLOQUEADO**: Parceiro Vizinhança
- **Plan tier**: `base` (lives abertas) / `plus`+ (exclusivas)
- **ABAC action**: `content.live.join`

## Elementos exibidos
- Vídeo ao vivo (Mux Live ou Cloudflare Stream — pendência ADR CNT-022)
- Chat (moderado — admin pode toggle)
- Campo para perguntas ao palestrante
- CTA contextual (ex: "Saber mais" para lançamento de solução)

## Componentes
- `LiveStreamPlayer` (RTMP/HLS via provedor)
- `LiveChat` (WebSocket; moderação reativa)
- `QuestionForm`
- `LiveStateBadge` (AO VIVO / AGUARDANDO / ENCERRADA)

## Estados
- **Loading**: skeleton.
- **Aguardando**: countdown + chat aberto opcional.
- **Live**: vídeo + chat ativos.
- **Encerrada**: redirect [[LMS10]] replay com mensagem.
- **Sem permissão**: 403 (Parceiro Vizinhança ou tier).
- **Buffering / network**: indicador.
- **Error**: toast com fallback.

## Integração com backend
| Ação | Endpoint (inferido) | Método |
|---|---|---|
| Detalhe live | `/api/v1/academy/lives/{id}` | GET |
| Token de stream | `/api/v1/academy/lives/{id}/token` | POST (signed) |
| Chat (WS) | `/ws/academy/lives/{id}/chat` | WebSocket |
| Pergunta | `/api/v1/academy/lives/{id}/questions` | POST |

## Regras de negócio críticas
- **Tipos canônicos** (`LiveType`): debates_condominiais, lancamento_solucoes, palestras_tecnicas, entrevistas_especialistas — cada um pode ter UI auxiliar (sub-tela conceitual em `06-lms-academia.md` §9 LMS11-14).
- Ao encerrar, **grava replay automático** (CNT-023; 30d).
- Chat: moderação reativa; admin pode silenciar/banir.
- Perguntas filtradas; admin/moderador escolhe quais respondem.
- Sem criação direta da sala — só admin agenda em [[LMS8]] (D-062).

## Ligações
- [[LMS8]] · [[LMS10]] · [[_moc]]

## Gaps/ressalvas
- Provedor (Mux Live vs Cloudflare Stream) sem ADR (CNT-022).
- Aggregate formal `Live` LMS ausente (G-03).
- Sub-telas LMS11-14 do v2 (debates / lançamento / palestra / entrevista) não distintas no briefing — esta UI assume que LMS9 cobre todos os tipos com componentes contextuais.
- Endpoint REST inferido.
- WS chat: provedor (Pusher / Socket.io / Centrifugo?) sem decisão.
