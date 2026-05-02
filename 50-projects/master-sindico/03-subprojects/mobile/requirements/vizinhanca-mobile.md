---
title: Vizinhança (Mobile) — Mobile Flutter
type: requirement
tags: [master-sindico, mobile, flutter, vizinhanca, feature-req]
project: master-sindico
stack: mobile
priority: m3
status: pending
bounded_context: commercial
created: 2026-04-24
---

# Vizinhança — Mobile

Feed de ofertas e serviços de comerciantes da vizinhança disponibilizados ao morador. Em M3 mobile: **feed + detalhe + ativar promoção** (VM1..VM7). Publicação de oferta = web (parceiro/admin).

## Escopo M1/M2/M3

- **M1/M2**: fora.
- **M3**: feed consumer (morador) + ativar cupom + geolocalização opcional.

## Telas envolvidas

- `VM1` (feed vizinhança — `[[../../web/ui-catalog/vizinhanca/VM1]]`)
- `VM2` (detalhe oferta)
- `VM3` (ativar promoção — confirmação + código)
- `VM4` (meus cupons ativos)
- `VM5` (histórico usado)
- `VM6` (filtros por categoria)
- `VM7` (permissão localização — onboarding)

## Funcionais (FR-MOB-VIZ-N)

- **FR-MOB-VIZ-1** `GET /api/v1/vizinhanca/feed?lat&lng&radius_km&category` — paginado por distância/relevância.
- **FR-MOB-VIZ-2** Se permissão localização negada → prompt em VM7; se usuário recusa → feed sem ordenação geo (por relevância).
- **FR-MOB-VIZ-3** Cada item: thumb, nome comerciante, distância (se geo), categoria, preço/% desconto.
- **FR-MOB-VIZ-4** Tap → detalhe VM2: descrição, termos, galeria, botão "Ativar promoção".
- **FR-MOB-VIZ-5** `POST /api/v1/vizinhanca/offers/{id}/activate` retorna `{ coupon_code, expires_at }`.
- **FR-MOB-VIZ-6** VM3 exibe código em fonte grande + QR + cópia clipboard + share sheet.
- **FR-MOB-VIZ-7** VM4 lista cupons ativos (locais Hive) com status `active / used / expired`.
- **FR-MOB-VIZ-8** Push `vizinhanca.offer.new` quando nova oferta relevante (opt-in canal).
- **FR-MOB-VIZ-9** Infinite scroll com ListView.builder.

## Não-funcionais (NFR-MOB)

- **NFR-MOB-PERF-1** Feed com 20 items visíveis < 300ms primeiro frame.
- **NFR-MOB-PERF-2** Imagens via `cached_network_image` + placeholder.
- **NFR-MOB-LGPD-1** Geolocalização uso mínimo — lat/lng arredondados a 2 casas decimais (~1km) antes de enviar.
- **NFR-MOB-PERM-1** Permissão localização `whileInUse` (nunca `always` em M3).

## Dados locais

- **Hive `vizinhanca_feed_{condominium_id}`** (TTL 10min, cap 50).
- **Hive `vizinhanca_coupons_active`** persistente até expirar.
- **Hive `vizinhanca_coupons_history`** persistente (audit local).
- **SharedPreferences**: categoria preferida, toggle "receber push novas ofertas".

## Integração backend

- `GET /api/v1/vizinhanca/feed`
- `GET /api/v1/vizinhanca/offers/{id}`
- `POST /api/v1/vizinhanca/offers/{id}/activate`
- `GET /api/v1/vizinhanca/coupons/me`

Push: `vizinhanca.offer.new`, `vizinhanca.coupon.expiring` (24h antes).

## Padrões Flutter aplicáveis

- `VizinhancaFeedBloc` + events `LoadRequested / NextPageRequested / CategoryFiltered / LocationUpdated`.
- `bloc_concurrency.droppable()` em `ActivateRequested` (evita doble cupom).
- `freezed` em `Offer` + `Coupon` + `ActivationResult`.
- `geolocator` package + `permission_handler`.
- `cached_network_image`.

## Gaps/decisões abertas

- **Q-MOB-VIZ-01** Cupons offline (resgatar sem internet) — não no escopo M3; exige código cryptographic assinado server-side.
- **Q-MOB-VIZ-02** Radius default (5km? 10km?) — a definir com comercial.
- **Q-MOB-VIZ-03** Comerciante cadastra oferta via app também? Assumido **não** em M3 (só web).

## Links

- [[../../../_moc]]
- [[../../web/ui-catalog/vizinhanca/VM1]] · [[../../web/ui-catalog/vizinhanca/VM2]] · [[../../web/ui-catalog/vizinhanca/VM3]]
- [[../../../00-product/sub-produtos/08-vizinhanca]]
