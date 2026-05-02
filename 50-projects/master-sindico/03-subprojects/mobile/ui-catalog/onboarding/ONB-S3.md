---
title: ONB-S3 — Dados Pessoais (Mobile, Síndico)
type: ui-screen
tags: [master-sindico, ui-catalog, mobile, onboarding, sindico]
project: master-sindico
persona: sindico
screen_id: ONB-S3
sub_produto: governanca-institucional
plan_requirement: trial
status: specification
stack: mobile
created: 2026-04-24
---

# ONB-S3 — Dados Pessoais (Mobile, Síndico)

## Finalidade

Coleta dados do síndico: CPF, RG, contato, endereço residencial (PII sensível).

## Persona e ABAC

- Síndico em onboarding.

## Fluxo mobile

1. Form: CPF (mask), RG, celular, email adicional, endereço.
2. Validação client-side (CPF valid check digit).
3. Auto-save debounce 2s.

## Widget Flutter principal

`OnbS3Form` com `TextFormField` + `mask_text_input_formatter`.

## Platform-specific notes

- iOS: toolbar next/done.
- Android: imeAction sequencial.
- **NFC de RG** (opcional leitura) — M3.

## Estados

- Editing / ValidationError / Saving / Saved.

## Offline behavior

- Auto-save enfileira.

## Push notification trigger

Nenhum.

## Links

- [[_moc]] · [[ONB-S4]]
