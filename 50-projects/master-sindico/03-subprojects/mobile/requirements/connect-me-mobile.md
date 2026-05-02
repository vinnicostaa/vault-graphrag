---
title: Connect Me (Mobile) — Mobile Flutter
type: requirement
tags: [master-sindico, mobile, flutter, connect-me, feature-req]
project: master-sindico
stack: mobile
priority: m2
status: pending
bounded_context: content
created: 2026-04-24
---

# Connect Me — Mobile

Canal direto **M→S** (Morador → Síndico) para reportar/sugerir/elogiar. Morador envia mensagem estruturada; síndico recebe em inbox dedicada. Limite de envio controlado (plano morador = 0/ano free; premium = N/ano).

## Escopo M1/M2/M3

- **M1**: fora (não está no escopo enxuto M1 morador + timeline read).
- **M2**: morador envia (categoria + título + descrição + foto opcional) + síndico recebe e responde.
- **M3**: tópicos agregados + SLA + escalada se sem resposta em 7d.

## Telas envolvidas

- `CM-M1` (lista Connect Me do morador — enviados)
- `CM-M2` (novo Connect Me — form)
- `CM-M3` (detalhe envio + resposta do síndico)
- `CM-S1` (inbox síndico — lista recebidos)
- `CM-S2` (detalhe + ações)

Web equivalente: `[[../../web/requirements/_moc]]` (a criar).

## Funcionais (FR-MOB-CM-N)

- **FR-MOB-CM-1** `GET /api/v1/content/connect-me/sent` paginado.
- **FR-MOB-CM-2** Morador toca FAB "Novo" → form com: categoria (enum: `suggestion/complaint/compliment/other`), título (max 80), descrição (max 1000), foto opcional (via `image_picker`, presign).
- **FR-MOB-CM-3** Antes de submit, verificar cota disponível: `GET /api/v1/content/connect-me/quota/me` → `{ used, limit, reset_at }`.
- **FR-MOB-CM-4** Se `used >= limit` → modal "Cota atingida. Atualize seu plano." + CTA billing.
- **FR-MOB-CM-5** `POST /api/v1/content/connect-me { category, title, description, attachment_key? }`.
- **FR-MOB-CM-6** Após sucesso, audit + entry reflete em CM-M1 com status `sent`.
- **FR-MOB-CM-7** Morador **não pode editar nem deletar** após envio (append-only).
- **FR-MOB-CM-8** Resposta do síndico chega via push `connect-me.replied.{id}` + aparece em CM-M3.
- **FR-MOB-CM-9** Síndico em CM-S1 vê lista com filtro por categoria + chip "pendente resposta".
- **FR-MOB-CM-10** Síndico em CM-S2 pode responder (text livre até 2000) + marcar status `resolved/acknowledged/dismissed`.

## Não-funcionais (NFR-MOB)

- **NFR-MOB-UX-1** Contador de caracteres visível em descrição.
- **NFR-MOB-PERF-1** Upload foto comprimida client-side (max 2MB JPEG Q85).
- **NFR-MOB-A11Y-1** Categoria via `RadioListTile` navegável por leitor de tela.
- **NFR-MOB-LGPD-1** Dados do envio ficam visíveis para síndico + auditoria; moradores não veem envios de outros.

## Dados locais

- **Hive `connect_me_sent`** (TTL 30min, cap 100).
- **Hive `connect_me_quota`** (TTL 5min).
- **NÃO persistir** draft localmente antes de envio (evita vazamento se device compartilhado).
- **Invalidação**: após envio 201 → refetch lista + quota.

## Integração backend

- `GET /api/v1/content/connect-me/sent`
- `POST /api/v1/content/connect-me`
- `GET /api/v1/content/connect-me/{id}`
- `GET /api/v1/content/connect-me/quota/me`
- `GET /api/v1/content/connect-me/received` (síndico)
- `POST /api/v1/content/connect-me/{id}/reply` (síndico)
- `POST /api/v1/storage/presign` (foto).

Push: `connect-me.replied.{id}` (p/ morador), `connect-me.new.{condominium_id}` (p/ síndico).

## Padrões Flutter aplicáveis

- `ConnectMeBloc` (morador) + `ConnectMeInboxBloc` (síndico).
- `bloc_concurrency.droppable()` no `SubmitRequested` (D-048).
- `freezed` em `ConnectMeEntry` + `Category` sealed union.
- `image_picker` + `flutter_image_compress` para foto.
- Permission camera/gallery via `permission_handler`.

## Gaps/decisões abertas

- **Q-MOB-CM-01** Cota do morador base (plano free) = 0/ano (D-065/D-079 legado); premium = ? — valor exato depende de plano Stripe-style (D-057).
- **Q-MOB-CM-02** Resposta do síndico permite anexo? Assumido **não** em M2 (texto puro).
- **Q-MOB-CM-03** Moderação automática de texto (palavrão/ofensa)? Fora de escopo M2.

## Links

- [[../../../_moc]]
- [[../../../00-product/sub-produtos/02-connect-me]]
- [[push-notifications]]
- [[../../../STATE#D-057|D-057 Planos globais]]
