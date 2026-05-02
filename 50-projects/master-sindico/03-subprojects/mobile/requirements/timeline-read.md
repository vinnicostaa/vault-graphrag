---
title: Timeline Read — Mobile Flutter
type: requirement
tags: [master-sindico, mobile, flutter, institutional, feature-req]
project: master-sindico
stack: mobile
priority: m1
status: pending
bounded_context: institutional
created: 2026-04-24
---

# Timeline Read — Mobile

Linha do Tempo **append-only read-only** no mobile. Morador e síndico consomem; escrita (nova entrada) é web-only em M1.

## Escopo M1/M2/M3

- **M1**: listagem paginada + detalhe + filtros básicos + read mark.
- **M2**: compartilhamento externo (share sheet nativa com PDF/imagem).
- **M3**: voice narration (síndico no modo apresentação).

## Telas envolvidas

- `M6` (timeline lista morador; ver `[[../../web/ui-catalog/morador/M6]]` equivalente web)
- `M6-DETAIL` (detalhe de entrada)
- `S7` (timeline read síndico — similar M6 mas com metadados extras: autor, audit ref)

## Funcionais (FR-MOB-TL-N)

- **FR-MOB-TL-1** Lista ordenada desc por `published_at`; paginada `GET /api/v1/institutional/timeline/{condominium_id}?page=N&size=20`.
- **FR-MOB-TL-2** Infinite scroll com skeleton ao carregar próxima página.
- **FR-MOB-TL-3** Cada item: chip categoria (comunicado/ata/mural/evento) + título + preview 2 linhas + data relativa ("há 2 horas").
- **FR-MOB-TL-4** Tap abre detalhe full-screen com conteúdo markdown renderizado + anexos (PDF via `open_filex`, imagens via `photo_view`).
- **FR-MOB-TL-5** Primeira visualização marca `read_at` via `POST /api/v1/institutional/timeline/{id}/read`; badge "novo" some.
- **FR-MOB-TL-6** Filtro chip-bar: [todos] [comunicados] [atas] [eventos]; seleção salva em Hive user prefs.
- **FR-MOB-TL-7** Pull-to-refresh recarrega página 1.
- **FR-MOB-TL-8** Offline: exibe timeline cacheada (até 100 entradas); banner "Offline — cache de <X horas atrás>".
- **FR-MOB-TL-9** Append-only: UI **não permite** delete/edit (conceito "imutável + audit").

## Não-funcionais (NFR-MOB)

- **NFR-MOB-PERF-1** Scroll 60fps com lista > 200 itens (ListView.builder + key).
- **NFR-MOB-PERF-2** Imagens lazy-loaded via `cached_network_image`.
- **NFR-MOB-A11Y-1** Lista navegável com rotor VoiceOver/TalkBack.

## Dados locais

- **Hive `timeline_{condominium_id}`** (TTL 1h, cap 100 entradas).
- **hydrated_bloc `TimelineBloc`** persiste última page + filter state.
- **SharedPreferences**: último filter selecionado.
- **Invalidação**: push FCM categoria `timeline.new` → marca stale → fetch page 1 no próximo foreground.

## Integração backend

- `GET /api/v1/institutional/timeline/{condominium_id}?page&size&category`
- `GET /api/v1/institutional/timeline/{id}` (detail)
- `POST /api/v1/institutional/timeline/{id}/read`

Push FCM topic: `timeline.{condominium_id}.new`.

## Padrões Flutter aplicáveis

- `TimelineBloc` com events `LoadRequested / NextPageRequested / FilterChanged / RefreshRequested` (último com `droppable`).
- States freezed `TimelineLoading / TimelineLoaded(items, hasMore, filter) / TimelineError / TimelineOffline`.
- `hydrated_bloc` para cache local.
- `freezed` em `TimelineEntry` entity + union de categorias.
- `ListView.builder` com `itemExtent` hint quando possível.

## Gaps/decisões abertas

- **Q-MOB-TL-01** Scroll position preservada ao trocar tab bottom nav? Assumido sim via `PageStorageKey`.
- **Q-MOB-TL-02** Filtro combinado (multi-select chip) não está no PDF; M1 = single chip exclusivo.
- **Q-MOB-TL-03** Download de anexo grande (PDF > 5MB) em 3G: mostrar progress + cancelável.

## Links

- [[../../../_moc]]
- [[../../web/ui-catalog/morador/M6]] · [[../../web/ui-catalog/sindico/S7]]
- [[../../../00-product/sub-produtos/01-governanca-institucional]]
- [[push-notifications]] (tópico timeline.new)
