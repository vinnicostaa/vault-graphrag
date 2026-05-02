---
title: Comunicados — Mobile Flutter
type: requirement
tags: [master-sindico, mobile, flutter, institutional, feature-req]
project: master-sindico
stack: mobile
priority: m1
status: pending
bounded_context: institutional
created: 2026-04-24
---

# Comunicados — Mobile

Canal oficial do síndico para moradores. **Read-only no mobile** em M1 (publicação = web). Suporte a **push notification** obrigatório + **deep-link** para comunicado específico.

## Escopo M1/M2/M3

- **M1**: lista + detalhe + push + mark-as-read.
- **M2**: filtros (importantes/urgentes) + pesquisa full-text local.
- **M3**: resposta estruturada (ciente com comentário curto).

## Telas envolvidas

- `M11` (comunicados lista; `[[../../web/ui-catalog/morador/M11]]`)
- `M11-DETAIL` (detalhe)

## Funcionais (FR-MOB-COM-N)

- **FR-MOB-COM-1** `GET /api/v1/institutional/announcements/{condominium_id}?page&size` paginado.
- **FR-MOB-COM-2** Cada item: chip prioridade (normal/importante/urgente), título, preview, data, badge "novo" se não lido.
- **FR-MOB-COM-3** Tap → detalhe: markdown renderizado + anexos + botão "Marcar como lido".
- **FR-MOB-COM-4** **Auto-read** ao ver detalhe > 3s (debounce) via `POST /api/v1/institutional/announcements/{id}/read`.
- **FR-MOB-COM-5** Push FCM tópico `announcement.{condominium_id}.new` entrega notificação com payload `{ announcement_id, priority, title }`.
- **FR-MOB-COM-6** Tap no push → deep-link `/announcements/{id}` (abre detalhe direto mesmo se app killed).
- **FR-MOB-COM-7** Se prioridade = `urgent`, push canal alta (Android IMPORTANCE_HIGH + iOS interruption level `time-sensitive`).
- **FR-MOB-COM-8** Offline: cache 50 mais recentes; detalhe completo cached se já visto.

## Não-funcionais (NFR-MOB)

- **NFR-MOB-PUSH-1** Taxa de entrega push ≥ 95% (FCM baseline).
- **NFR-MOB-A11Y-1** Badge "novo" com `Semantics(label: 'não lido')`.
- **NFR-MOB-PERF-1** Deep-link resolve em < 500ms quando app warm.

## Dados locais

- **Hive `announcements_{condominium_id}`** (TTL 30min, cap 50).
- **Hive `announcement_detail_{id}`** cached após primeira leitura (TTL 7d).
- **hydrated_bloc `AnnouncementsBloc`** persiste lista + unread_count.

## Integração backend

- `GET /api/v1/institutional/announcements/{condominium_id}`
- `GET /api/v1/institutional/announcements/{id}`
- `POST /api/v1/institutional/announcements/{id}/read`
- `POST /api/v1/devices/fcm-token` — registro do FCM token (feito em login).

Push tópicos: `announcement.{condominium_id}.new` (prioridade normal), `announcement.{condominium_id}.urgent` (canal separado).

## Padrões Flutter aplicáveis

- `AnnouncementsBloc` com events `LoadRequested / OpenRequested(id) / MarkAsReadRequested`.
- `freezed` em `Announcement` + `Priority` enum.
- `hydrated_bloc` persiste unread count para badge no bottom nav.
- `go_router` rota `/announcements/:id` deep-linkable.
- Firebase Messaging handler (`onMessage`, `onMessageOpenedApp`, `getInitialMessage`) → dispara `OpenRequested` no Bloc.

## Gaps/decisões abertas

- **Q-MOB-COM-01** Unsubscribe por prioridade (desligar só "normais")? Assumido 1 toggle global em M1; granular em M2.
- **Q-MOB-COM-02** Comunicado com anexo PDF grande (> 10MB) em push: só link; download manual.
- **Q-MOB-COM-03** "Marcar todos como lido" — ausente no PDF; adicionar em M2.

## Links

- [[../../../_moc]]
- [[../../web/ui-catalog/morador/M11]]
- [[../../../00-product/sub-produtos/01-governanca-institucional]]
- [[push-notifications]]
