---
title: ONB-E2 — Dados Empresa (Mobile)
type: ui-screen
tags: [master-sindico, ui-catalog, mobile, onboarding, empresa]
project: master-sindico
persona: empresa
screen_id: ONB-E2
sub_produto: banco-talentos
plan_requirement: trial
status: specification
stack: mobile
created: 2026-04-24
---

# ONB-E2 — Dados Empresa (Mobile)

## Finalidade

Step 1 empresa: trade name + foto/logo.

## Persona e ABAC

- Empresa em onboarding. Fluxo mobile é **opcional** — primeiro cliente usa web; mobile = subset.

## Fluxo mobile

1. Input nome fantasia.
2. Upload logo (`image_picker`, presign).

## Widget Flutter principal

`OnbE2Form` form simples.

## Platform-specific notes

- Permissions gallery.

## Estados

- Editing / Saving / Saved.

## Offline behavior

- Auto-save enfileira.

## Push notification trigger

Nenhum.

## Links

- [[_moc]] · [[ONB-E3]]
