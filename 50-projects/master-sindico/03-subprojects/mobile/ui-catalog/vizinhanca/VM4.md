---
title: VM4 — Meus Cupons Ativos (Mobile)
type: ui-screen
tags: [master-sindico, ui-catalog, mobile, vizinhanca, morador]
project: master-sindico
persona: morador
screen_id: VM4
sub_produto: vizinhanca
plan_requirement: any
status: specification
stack: mobile
created: 2026-04-24
---

# VM4 — Meus Cupons Ativos (Mobile)

## Finalidade

Lista cupons ativos do morador com status (ativo / expirando / expirado).

## Persona e ABAC

- Morador, `vizinhanca.coupon.self.read`.

## Fluxo mobile

1. Entrada via tab "Meus cupons" no módulo.
2. `GET /vizinhanca/coupons/me`.
3. Lista cards com código, comerciante, expira em, status.
4. Tap → volta para VM3 (visualizar código).

## Widget Flutter principal

`MyCouponsPage` com `ListView` de `CouponCard`.

## Platform-specific notes

- Swipe para ver detalhe.

## Estados

- Loading / Loaded / Empty / Error / Offline (cached).

## Offline behavior

- Cupons ativos persistem Hive até expirar.

## Push notification trigger

- `vizinhanca.coupon.expiring` → badge.

## Links

- [[_moc]] · [[VM3]] · [[VM5]]
