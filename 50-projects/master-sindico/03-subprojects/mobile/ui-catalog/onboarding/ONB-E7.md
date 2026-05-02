---
title: ONB-E7 — Banco de Talentos Opt-in (Mobile, Empresa)
type: ui-screen
tags: [master-sindico, ui-catalog, mobile, onboarding, empresa, banco-talentos]
project: master-sindico
persona: empresa
screen_id: ONB-E7
sub_produto: banco-talentos
plan_requirement: trial
status: specification
stack: mobile
created: 2026-04-24
---

# ONB-E7 — Banco de Talentos Opt-in (Mobile)

## Finalidade

Checkbox explícito de opt-in para banco de talentos (D-099 revoga D-074 — 5 vínculos profissionais). Empresa aceita ser descoberta por síndicos como prestador.

## Persona e ABAC

- Empresa.

## Fluxo mobile

1. Texto explicativo banco de talentos.
2. Checkbox "Quero estar no Banco de Talentos" (default OFF).
3. Se ON → form extra (vídeo pitch 90s URL opcional, tagline).
4. Finalizar onboarding → ONB-EFIN.

## Widget Flutter principal

`OnbE7Page` com card explicativo + checkbox + form condicional.

## Platform-specific notes

- Acessibilidade: checkbox com label longo via `MergeSemantics`.

## Estados

- Idle / Enabled / Submitting / Done.

## Offline behavior

- Auto-save enfileira finalização.

## Push notification trigger

- Server pode enviar `banco-talentos.new.company.{id}` para síndicos (opt-in canal).

## Links

- [[_moc]] · [[ONB-E6]]
- [[../../../STATE#D-074|D-074 Banco de Talentos 3 vínculos]]
