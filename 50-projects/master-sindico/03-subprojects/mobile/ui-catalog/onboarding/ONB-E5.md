---
title: ONB-E5 — Área de Atuação (Mobile)
type: ui-screen
tags: [master-sindico, ui-catalog, mobile, onboarding, empresa]
project: master-sindico
persona: empresa
screen_id: ONB-E5
sub_produto: banco-talentos
plan_requirement: trial
status: specification
stack: mobile
created: 2026-04-24
---

# ONB-E5 — Área de Atuação (Mobile)

## Finalidade

Multi-select de categorias de serviço (limpeza, jardinagem, reforma, segurança, etc).

## Persona e ABAC

- Empresa.

## Fluxo mobile

1. Lista chips categorias (10-20 default).
2. Permite tocar múltiplas (min 1, max 5).
3. Próximo.

## Widget Flutter principal

`Wrap` com `FilterChip`s.

## Platform-specific notes

- Touch targets ≥ 48dp.

## Estados

- Idle / Valid / InvalidMinMax.

## Offline behavior

- Categorias fetched 1x + cached.

## Push notification trigger

Nenhum.

## Links

- [[_moc]] · [[ONB-E6]]
