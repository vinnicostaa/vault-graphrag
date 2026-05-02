---
title: ONB-M5 — Status da Unidade (Mobile)
type: ui-screen
tags: [master-sindico, ui-catalog, mobile, onboarding, morador]
project: master-sindico
persona: morador
screen_id: ONB-M5
sub_produto: governanca-institucional
plan_requirement: any
status: specification
stack: mobile
created: 2026-04-24
---

# ONB-M5 — Status da Unidade (Mobile)

## Finalidade

Último step morador: status (proprietário / inquilino / dependente) + dependências (opcional).

## Persona e ABAC

- Morador em onboarding.

## Fluxo mobile

1. RadioGroup: Proprietário / Inquilino / Dependente.
2. Se inquilino → campo extra "nome do proprietário" (opcional).
3. Finalizar → `POST /onboarding/sessions/me/complete` → cria Membership `active` → ONB-MFIN.

## Widget Flutter principal

`OnbM5Page` com `RadioListTile` + validação form.

## Platform-specific notes

- Botão finalizar 56dp altura.
- Haptic light no finalizar.

## Estados

- Idle / Submitting / Done / Error.

## Offline behavior

- **Online obrigatório** (finalização binds Membership).

## Push notification trigger

Nenhum.

## Links

- [[_moc]] · [[ONB-M4]] · [[ONB-MFIN]]
