---
title: ONB-M3 — Identificação da Unidade (Mobile)
type: ui-screen
tags: [master-sindico, ui-catalog, mobile, onboarding, morador]
project: master-sindico
persona: morador
screen_id: ONB-M3
sub_produto: governanca-institucional
plan_requirement: any
status: specification
stack: mobile
created: 2026-04-24
---

# ONB-M3 — Identificação da Unidade (Mobile)

## Finalidade

Segunda etapa morador. Bloco/apto/fração ideal (read-only).

## Persona e ABAC

- Morador em onboarding.

## Fluxo mobile

1. Bloco: dropdown com opções do condomínio.
2. Apto: dropdown filtrado por bloco.
3. Fração ideal: exibição read-only (vem da unidade selecionada).
4. Próximo → ONB-M4.

## Widget Flutter principal

`OnbM3Page` com `DropdownButtonFormField` encadeados.

## Platform-specific notes

- Dropdown Material 3.
- Teclado irrelevante (só dropdowns).

## Estados

- LoadingUnits / Ready / Error.

## Offline behavior

- Lista unidades cached Hive quando possível.
- Sem rede + sem cache → bloquear com banner.

## Push notification trigger

Nenhum.

## Links

- [[_moc]] · [[ONB-M2]] · [[ONB-M4]]
