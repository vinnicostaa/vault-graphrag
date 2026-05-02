---
title: Pattern — Recurring Required Action
type: pattern
tags:
  - pattern
  - ui
  - web
  - master-sindico
  - compliance
created: '2026-04-27'
updated: '2026-04-27'
---

# Pattern — Recurring Required Action

Ação obrigatória recorrente que aparece como **banner/modal bloqueante** em frequência definida (bimestral, mensal, anual) — ex.: avaliação de gestão (`REQ-GOV-AVALIACAO-GESTAO`), reaceite de termos LGPD, declaração NR-1.

## Estrutura

- **Trigger**: ao logar OU ao acessar tela específica, se data alvo passou.
- **Banner não-fechável** (com `Adiar 24h` opcional) que persiste até completion.
- **Bloqueio funcional**: ações estratégicas bloqueadas até completion (ex.: export dossier bloqueado se compliance_status ≠ em_dia).
- **Audit**: `last_action_at`, `next_action_due_at` em domain aggregate (Membership/Mandate/etc.).
- **Notification cascata**: D-7, D-3, D-0, D+1 (atrasado).

## Quando usar

- Compliance regulatório (LGPD, NR-1).
- Engagement obrigatório (avaliação morador → governance score).

## Quando evitar

- "Pesquisa de satisfação" não-obrigatória — usa banner dispensável, não bloqueante.
- Ação que pode ser feita por admin no lugar do user — não vire bloqueio do user.

## Stack

SolidJS + Kobalte (Dialog modal não-fechável quando crítico, Sheet quando adiável) + signal global de `pending_actions[]`.

## Cross-refs

- [[forms]] — modal contém form
- [[legal-terms-checkbox]] — quando ação é aceite
- [[../../../04-requirements/functional/compliance]]
- [[../../../04-requirements/functional/governance]]

## Links

- [[_moc]]
