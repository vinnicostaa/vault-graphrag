---
title: ONB-E4 — Endereço + Contato (Mobile)
type: ui-screen
tags: [master-sindico, ui-catalog, mobile, onboarding, empresa]
project: master-sindico
persona: empresa
screen_id: ONB-E4
sub_produto: banco-talentos
plan_requirement: trial
status: specification
stack: mobile
created: 2026-04-24
---

# ONB-E4 — Endereço + Contato (Mobile)

## Finalidade

Endereço comercial + telefone + email corporativo.

## Persona e ABAC

- Empresa.

## Fluxo mobile

1. CEP (mask) + autocomplete ViaCEP.
2. Telefone + email.

## Widget Flutter principal

Form com mask CEP + lookup.

## Platform-specific notes

- Keyboards apropriados.

## Estados

- Editing / Saving.

## Offline behavior

- Auto-save enfileira.

## Push notification trigger

Nenhum.

## Links

- [[_moc]] · [[ONB-E5]]
