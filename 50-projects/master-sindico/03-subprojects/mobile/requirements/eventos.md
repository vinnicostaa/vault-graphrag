---
title: Eventos — Mobile Flutter
type: requirement
tags: [master-sindico, mobile, flutter, institutional, feature-req]
project: master-sindico
stack: mobile
priority: m2
status: pending
bounded_context: institutional
created: 2026-04-24
---

# Eventos — Mobile

Lista de eventos do condomínio + RSVP (ciente / confirmar participação / recusar). Em M1 apenas leitura; em M2 RSVP ativo.

## Escopo M1/M2/M3

- **M1**: listagem + detalhe (read-only).
- **M2**: marcar ciente + RSVP (confirmado/recusado/talvez) + push "lembrete 1h antes".
- **M3**: check-in presencial (código QR ou numérico — reuso do fluxo assembleia).

## Telas envolvidas

- `M10` (lista eventos morador; equivalente web `[[../../web/ui-catalog/morador/M10]]`)
- `M10-DETAIL` (detalhe + RSVP)
- `M10-RSVP-SHEET` (bottom sheet confirmar)

## Funcionais (FR-MOB-EVT-N)

- **FR-MOB-EVT-1** Lista ordenada asc por `starts_at` (próximos primeiro); passados via tab secundária.
- **FR-MOB-EVT-2** Cada item: data (dia/mês destacado) + título + local + chip status (`upcoming / live / past`).
- **FR-MOB-EVT-3** Tap abre detalhe: descrição, local com mapa estático, anexos, botão "Dar ciência" (M1) / "Confirmar presença" (M2).
- **FR-MOB-EVT-4** **Ciência** (M1): `POST /api/v1/institutional/events/{id}/acknowledge` registra timestamp + audit; UI vira "✓ Ciente em <data>".
- **FR-MOB-EVT-5** **RSVP** (M2): bottom sheet com 3 opções (Confirmado / Talvez / Recusado); `POST /api/v1/institutional/events/{id}/rsvp { status }`.
- **FR-MOB-EVT-6** Notificação push 1h antes (M2) via agendamento server-side tópico `event.{id}.reminder`.
- **FR-MOB-EVT-7** Compartilhar evento: share sheet nativa com link `https://app.mastersindico.com.br/events/{id}`.
- **FR-MOB-EVT-8** Offline: lista cacheada; ciência/RSVP enfileirados em sync queue (M2+).

## Não-funcionais (NFR-MOB)

- **NFR-MOB-PERF-1** Lista com 100+ eventos em 60fps.
- **NFR-MOB-A11Y-1** Data em formato ISO-like para leitor de tela ("24 de maio de 2026").
- **NFR-MOB-I18N-1** Datas formatadas pt_BR (intl).

## Dados locais

- **Hive `events_{condominium_id}`** (TTL 15min).
- **Hive `events_rsvp_local_queue`** (M2+): fila de RSVPs pendentes de sync.
- **hydrated_bloc `EventsBloc`** persiste filter (upcoming/past).

## Integração backend

- `GET /api/v1/institutional/events/{condominium_id}?range=upcoming|past`
- `GET /api/v1/institutional/events/{id}`
- `POST /api/v1/institutional/events/{id}/acknowledge`
- `POST /api/v1/institutional/events/{id}/rsvp` (M2)

Push tópicos: `event.{id}.reminder`, `event.{id}.cancelled`.

## Padrões Flutter aplicáveis

- `EventsBloc` + events `LoadRequested / AcknowledgeRequested / RsvpChanged`.
- `droppable()` no botão "Dar ciência" (evita double-tap).
- `freezed` em `Event` + `RsvpStatus` sealed union.
- `hydrated_bloc` cache local.

## Gaps/decisões abertas

- **Q-MOB-EVT-01** Calendar integration (adicionar ao calendário nativo) — M2 ou M3? Assumido M3 via `add_2_calendar`.
- **Q-MOB-EVT-02** Eventos recorrentes (toda 1ª terça do mês) — não no escopo do PDF cliente; ignorar.

## Links

- [[../../../_moc]]
- [[../../web/ui-catalog/morador/M10]]
- [[../../../00-product/sub-produtos/01-governanca-institucional]]
- [[push-notifications]]
