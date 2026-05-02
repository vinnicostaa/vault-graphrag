---
title: ONB-S2 — Selecionar Condomínio (Mobile, Síndico)
type: ui-screen
tags: [master-sindico, ui-catalog, mobile, onboarding, sindico]
project: master-sindico
persona: sindico
screen_id: ONB-S2
sub_produto: governanca-institucional
plan_requirement: trial
status: specification
stack: mobile
created: 2026-04-24
---

# ONB-S2 — Selecionar Condomínio (Mobile, Síndico)

## Finalidade

Primeira etapa do onboarding síndico. Seleciona o condomínio onde foi eleito (lista pré-preenchida pelo admin do condomínio ou busca por CNPJ).

## Persona e ABAC

- Síndico recém-cadastrado, primeiro acesso.

## Fluxo mobile

1. Exibe lista de pendências "Você foi indicado como síndico em:" — pode ter 0 ou mais entradas pré-cadastradas.
2. Se 0 → busca manual por CNPJ do condomínio.
3. Seleciona → próximo ONB-S3.

## Widget Flutter principal

`OnbS2Page` com `ListView` de pendências + `SearchBar` fallback.

## Platform-specific notes

- Keyboard numérico para CNPJ.

## Estados

- Loading / Ready / Searching / Error.

## Offline behavior

- Lista cached se já tentou; busca exige rede.

## Push notification trigger

Nenhum.

## Links

- [[_moc]] · [[ONB-S3]]
