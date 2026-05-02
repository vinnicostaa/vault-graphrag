---
title: VM6 — Filtros (Mobile)
type: ui-screen
tags: [master-sindico, ui-catalog, mobile, vizinhanca, morador]
project: master-sindico
persona: morador
screen_id: VM6
sub_produto: vizinhanca
plan_requirement: any
status: specification
stack: mobile
created: 2026-04-24
---

# VM6 — Filtros (Mobile)

## Finalidade

Bottom sheet com filtros de feed: categoria (multi-select chip), distância (slider), preço/desconto.

## Persona e ABAC

- Morador.

## Fluxo mobile

1. FAB "Filtros" em VM1 → abre bottom sheet.
2. Aplica → fecha + refetch VM1 com novos params.
3. "Limpar filtros" reseta.

## Widget Flutter principal

`FiltersBottomSheet` com `showModalBottomSheet(isScrollControlled: true)`.

## Platform-specific notes

- Handle bar; swipe-down dismiss.

## Estados

- Open / Applying (spinner) / Applied (close).

## Offline behavior

- Filtros persistem Hive `vizinhanca_filters` mesmo offline.

## Push notification trigger

Nenhum.

## Links

- [[_moc]] · [[VM1]]
