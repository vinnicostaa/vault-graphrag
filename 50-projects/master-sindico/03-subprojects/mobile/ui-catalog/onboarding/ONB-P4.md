---
title: ONB-P4 — Termo de Comissão (Mobile, Parceiro)
type: ui-screen
tags: [master-sindico, ui-catalog, mobile, onboarding, parceiro]
project: master-sindico
persona: parceiro
screen_id: ONB-P4
sub_produto: marketing
plan_requirement: any
status: specification
stack: mobile
created: 2026-04-24
---

# ONB-P4 — Termo de Comissão (Mobile, Parceiro)

## Finalidade

Aceite do termo de comissão (percentual definido pelo plano/tipo).

## Persona e ABAC

- Parceiro.

## Fluxo mobile

1. Texto termo (escalonado por tier agência).
2. Checkbox aceite.
3. Assinatura biométrica (local_auth).
4. `POST /onboarding/sessions/me/complete`.
5. Navega ONB-FIN.

## Widget Flutter principal

Similar ONB-S6.

## Platform-specific notes

- Biometria.

## Estados

- Reading / Accepted / Authenticating / Done.

## Offline behavior

- Online obrigatório.

## Push notification trigger

Nenhum.

## Links

- [[_moc]] · [[ONB-P3]] · [[ONB-FIN]]
