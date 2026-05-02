---
title: ONB-SFIN — Análise em Curso (Mobile, Síndico)
type: ui-screen
tags: [master-sindico, ui-catalog, mobile, onboarding, sindico]
project: master-sindico
persona: sindico
screen_id: ONB-SFIN
sub_produto: governanca-institucional
plan_requirement: trial
status: specification
stack: mobile
created: 2026-04-24
---

# ONB-SFIN — Análise em Curso (Mobile, Síndico)

## Finalidade

Tela de transição pós-envio: explica que KYB é async (2-5 dias úteis), libera acesso read-only a módulos enquanto aguarda.

## Persona e ABAC

- Síndico pending_review.

## Fluxo mobile

1. Animação check + mensagem "Estamos validando seus dados".
2. Info: "Você pode usar leitura de timeline enquanto aguarda."
3. CTA "Entrar" → S6 (read-only se not approved).

## Widget Flutter principal

`OnbSFinPage` com `Lottie` + texto + CTA.

## Platform-specific notes

Nenhuma específica.

## Estados

- Shown / Navigating.

## Offline behavior

Nada.

## Push notification trigger

- Recebe `syndic.kyc.approved.{user_id}` quando aprovado → toast in-app + refresh home.
- `syndic.kyc.rejected.{user_id}` → modal com motivo + re-upload.

## Links

- [[_moc]] · [[ONB-S6]]
- [[../sindico/S6]]
