---
title: ONB-S6 — Termo + Assinatura Digital (Mobile, Síndico)
type: ui-screen
tags: [master-sindico, ui-catalog, mobile, onboarding, sindico]
project: master-sindico
persona: sindico
screen_id: ONB-S6
sub_produto: governanca-institucional
plan_requirement: trial
status: specification
stack: mobile
created: 2026-04-24
---

# ONB-S6 — Termo + Assinatura Digital (Mobile, Síndico)

## Finalidade

Aceite de termo de uso síndico + assinatura digital via biometria (local_auth).

## Persona e ABAC

- Síndico em onboarding.

## Fluxo mobile

1. Texto termo (scroll).
2. Checkbox aceite.
3. Botão "Assinar" → `local_auth.authenticate` (FaceID / fingerprint).
4. `POST /onboarding/sessions/me/complete` com biometric token (server registra audit).
5. Navega ONB-SFIN.

## Widget Flutter principal

`OnbS6Page` com `MarkdownBody` + `Checkbox` + `FilledButton` → `BiometricAuth`.

## Platform-specific notes

- iOS: FaceID/TouchID.
- Android: BiometricPrompt.
- **Fallback PIN** se biometria indisponível? Sim (M2+).

## Estados

- Reviewing / Authenticating / Submitting / Done / BiometricFailed.

## Offline behavior

- **Online obrigatório** (finalize binding + compliance audit).

## Push notification trigger

Nenhum.

## Links

- [[_moc]] · [[ONB-S5]] · [[ONB-SFIN]]
- [[../../security]] §6 (biometria)
