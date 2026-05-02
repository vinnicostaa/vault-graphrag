---
title: VM5 — Histórico de Cupons (Mobile)
type: ui-screen
tags: [master-sindico, ui-catalog, mobile, vizinhanca, morador]
project: master-sindico
persona: morador
screen_id: VM5
sub_produto: vizinhanca
plan_requirement: any
status: specification
stack: mobile
created: 2026-04-24
---

# VM5 — Histórico de Cupons (Mobile)

## Finalidade

Histórico de cupons já utilizados ou expirados (audit local + server).

## Persona e ABAC

- Morador, `vizinhanca.coupon.history.self`.

## Fluxo mobile

1. Entrada via VM4 → "Ver histórico".
2. `GET /vizinhanca/coupons/me?status=used|expired`.
3. Lista read-only ordenada desc por `used_at`.

## Widget Flutter principal

`CouponHistoryPage` com `ListView.builder`.

## Platform-specific notes

- Nada específico.

## Estados

- Loading / Loaded / Empty / Error / Offline.

## Offline behavior

- Histórico cached 30d.

## Push notification trigger

Nenhum.

## Links

- [[_moc]] · [[VM4]]
