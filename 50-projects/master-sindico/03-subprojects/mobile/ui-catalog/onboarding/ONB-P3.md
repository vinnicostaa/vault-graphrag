---
title: ONB-P3 — Dados Básicos (Mobile, Parceiro)
type: ui-screen
tags: [master-sindico, ui-catalog, mobile, onboarding, parceiro]
project: master-sindico
persona: parceiro
screen_id: ONB-P3
sub_produto: marketing
plan_requirement: any
status: specification
stack: mobile
created: 2026-04-24
---

# ONB-P3 — Dados Básicos (Mobile, Parceiro)

## Finalidade

Dados do parceiro: CNPJ (ou CPF se MEI), razão social, contato, PIX para comissão.

## Persona e ABAC

- Parceiro.

## Fluxo mobile

1. Se Agência → CNPJ + razão.
2. Se Indicador PF → CPF + nome.
3. Chave PIX (email / CPF / celular / aleatória).
4. Próximo ONB-P4.

## Widget Flutter principal

Form condicional baseado em `OnbFlowBloc.state.partnerType`.

## Platform-specific notes

- Keyboard numérico em CNPJ/CPF/PIX numérico.

## Estados

- Editing / Validating / Saved.

## Offline behavior

- Auto-save enfileira.

## Push notification trigger

Nenhum.

## Links

- [[_moc]] · [[ONB-P2]] · [[ONB-P4]]
