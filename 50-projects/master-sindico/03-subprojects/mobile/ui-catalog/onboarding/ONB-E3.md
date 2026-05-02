---
title: ONB-E3 — CNPJ + Razão Social (Mobile)
type: ui-screen
tags: [master-sindico, ui-catalog, mobile, onboarding, empresa]
project: master-sindico
persona: empresa
screen_id: ONB-E3
sub_produto: banco-talentos
plan_requirement: trial
status: specification
stack: mobile
created: 2026-04-24
---

# ONB-E3 — CNPJ + Razão Social (Mobile)

## Finalidade

CNPJ com auto-lookup (Receita/Serasa) preenchendo razão social, UF, data abertura.

## Persona e ABAC

- Empresa em onboarding.

## Fluxo mobile

1. Input CNPJ (mask).
2. `GET /integrations/cnpj/{cnpj}` — server busca Receita.
3. Preenche read-only: razão social, UF, situação.
4. User confirma → próximo.

## Widget Flutter principal

Form com mask + `BlocListener` para auto-fetch on valid CNPJ.

## Platform-specific notes

- Keyboard numérico.

## Estados

- Idle / Fetching / Fetched / Error / Offline.

## Offline behavior

- Bloqueia lookup; banner "Conecte-se".

## Push notification trigger

Nenhum.

## Links

- [[_moc]] · [[ONB-E2]] · [[ONB-E4]]
