---
title: ONB-MFIN — Tudo Pronto (Morador, Mobile)
type: ui-screen
tags: [master-sindico, ui-catalog, mobile, onboarding, morador]
project: master-sindico
persona: morador
screen_id: ONB-MFIN
sub_produto: governanca-institucional
plan_requirement: any
status: specification
stack: mobile
created: 2026-04-24
---

# ONB-MFIN — Tudo Pronto (Morador, Mobile)

## Finalidade

Tela institucional final de celebração + CTA "Ir para o painel".

## Persona e ABAC

- Morador.

## Fluxo mobile

1. Anima ilustração/check grande.
2. Mensagem "Você está conectado ao [condomínio]".
3. CTA → navega home M1 (clear stack).

## Widget Flutter principal

`OnbMFinPage` com `Lottie` ou `AnimatedBuilder` + `FilledButton`.

## Platform-specific notes

- iOS: haptic success.
- Android: confetti visual opcional.

## Estados

- Animating / Ready.

## Offline behavior

- Renderização pura; navegação segue.

## Push notification trigger

- Pode receber push "Boas-vindas ao MS" server-side (opcional).

## Links

- [[_moc]] · [[ONB-M5]]
- [[../morador/M1]]
