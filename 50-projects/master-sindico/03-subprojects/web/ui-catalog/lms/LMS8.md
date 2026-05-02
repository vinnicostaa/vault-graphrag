---
title: "LMS8 — Agenda de Lives"
type: ui-screen
tags: [master-sindico, ui-catalog, web, lms, academia, lives, d-097-expanded]
project: master-sindico
persona: any
category: LMS
screen_id: LMS8
sub_produto: lms-academia
plan_requirement: trial
status: specification
stack: web
created: 2026-04-23
---

# LMS8 — Agenda de Lives

## Finalidade
Listar próximas lives (data, tema, palestrante) e replays disponíveis. Botões "Participar" / "Lembrar-me".

## Fonte canônica
- `vault/90-archive/from-development-content-2026-04-23/contents/MÓDULO ACADEMIA MASTER SÍNDICO.md` §LMS8
- `master-sindico-v2/STATE.md` D-062 (Lives admin-only MVP)

## Mensagem institucional canônica
> "Encontros ao vivo com especialistas do universo condominial."

## Persona e ABAC
- **Personas**: Síndico, Empresa, Morador, Marketing, Admin
- **❌ BLOQUEADO**: Parceiro Vizinhança
- **Plan tier**: `trial` (visualizar agenda); participar conforme tier (`base` só abertas; `plus`/`pro` exclusivas)
- **ABAC action**: `content.live.list`
- **Criação/agendamento**: **APENAS `admin`** — D-062 (Q28)

## Lista exibida
- Data
- Tema
- Palestrante
- LiveType (badge: debate / lançamento / palestra técnica / entrevista)
- LiveAccessLevel (aberta / exclusiva)

## Botões por item
- **Participar** (quando `live_state=live`) → [[LMS9]]
- **Lembrar-me** (subscription para notificação push/email)

## Componentes
- `LiveAgendaList`
- `LiveCard`
- `RemindButton`
- `JoinButton`
- `AdminScheduleButton` (visível só para `admin` no MVP D-062)

## Estados
- **Loading**: skeleton.
- **Empty**: "Próximas lives em breve."
- **Live now**: card destacado com countdown / "AO VIVO".
- **Replay disponível**: card secundário → [[LMS10]].
- **Lock 30d expirado**: replay some.
- **Error**: toast retry.

## Integração com backend
| Ação | Endpoint (inferido) | Método |
|---|---|---|
| Listar lives | `/api/v1/academy/lives` | GET |
| Lembrar-me | `/api/v1/academy/lives/{id}/remind` | POST |
| **Agendar** (admin) | `/api/v1/academy/lives` | POST (apenas role=admin) |

## Regras de negócio críticas
- **D-062**: criação/agendamento **APENAS admin** no MVP. Primeira live = apresentação oficial da plataforma.
- Replay disponível por **30 dias** pós-evento (depois some).
- ABAC: `local_company` (Parceiro) → 403; `trial` ⏳ (preview); `base` morador → só lives `abertas`; `plus`/`pro` → abertas + exclusivas.
- Lives `exclusivas` exigem tier ou whitelist admin (convite).
- Provedor de Live LMS: Mux Live ou Cloudflare Stream — pendência CNT-022.

## Ligações
- [[LMS1]] · [[LMS9]] · [[LMS10]] · [[_moc]]

## Gaps/ressalvas
- Provedor Live LMS sem ADR (CNT-022).
- Q28: pós-MVP, Empresa Pro pode agendar?
- Endpoint REST inferido.
- Aggregate formal `Live` LMS ausente (G-03).
