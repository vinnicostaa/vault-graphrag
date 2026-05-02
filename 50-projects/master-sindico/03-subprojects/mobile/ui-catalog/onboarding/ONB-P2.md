---
title: ONB-P2 — Tipo de Parceiro (Mobile)
type: ui-screen
tags: [master-sindico, ui-catalog, mobile, onboarding, parceiro]
project: master-sindico
persona: parceiro
screen_id: ONB-P2
sub_produto: marketing
plan_requirement: any
status: specification
stack: mobile
created: 2026-04-24
---

# ONB-P2 — Tipo de Parceiro (Mobile)

## Finalidade

Seletor tipo de parceiro: Agência / Indicador. Define fluxo comercial/comissão.

## Persona e ABAC

- Parceiro primeira entrada.

## Fluxo mobile

1. RadioGroup Agência / Indicador.
2. Tooltip explica diferenças.
3. Próximo ONB-P3.

## Widget Flutter principal

`OnbP2Page` com 2 `RadioListTile` + texto explicativo.

## Platform-specific notes

Nenhuma específica.

## Estados

- Idle / Selected.

## Offline behavior

- Auto-save enfileira.

## Push notification trigger

Nenhum.

## Gaps/ressalvas

- **Q-FLOW-01**: parceiro mobile pode ser descontinuado (web-first); manter spec.

## Links

- [[_moc]] · [[ONB-P3]]
