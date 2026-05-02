---
title: VM3 — Ativar Promoção (Mobile)
type: ui-screen
tags: [master-sindico, ui-catalog, mobile, vizinhanca, morador]
project: master-sindico
persona: morador
screen_id: VM3
sub_produto: vizinhanca
plan_requirement: any
status: specification
stack: mobile
created: 2026-04-24
---

# VM3 — Ativar Promoção (Mobile)

## Finalidade

Confirmação da ativação + exibição do código (texto + QR) + copy + share.

## Persona e ABAC

- Morador com oferta válida na região.
- `vizinhanca.coupon.activate.self`.

## Fluxo mobile

1. VM2 "Ativar" → modal confirm "Confirmar ativação? O cupom pode ser usado apenas uma vez."
2. `POST /offers/{id}/activate { }`.
3. Retorna `{ coupon_code, expires_at }`.
4. Tela com código fonte grande + QR (via `qr_flutter`) + copiar + compartilhar + "Ver todos os meus cupons" (VM4).
5. Cupom guardado em Hive `vizinhanca_coupons_active`.

## Widget Flutter principal

`VM3ActivatedPage` com `Center` + código + `QrImageView` + `FilledButton`s.

## Platform-specific notes

- Haptic medium ao copiar.
- iOS share sheet nativa.

## Estados

- Activating / Activated / Expired / AlreadyUsed / Error / Offline.

## Offline behavior

- Ativação = online. Após ativado, código visível offline (cached).

## Push notification trigger

- `vizinhanca.coupon.expiring.{user_id}` 24h antes.

## Links

- [[_moc]] · [[VM2]] · [[VM4]]
