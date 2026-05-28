---
title: MOC — rustcraft ADRs
type: moc
tags:
  - moc
  - adr
  - rustcraft
created: '2026-05-27'
updated: '2026-05-27'
---
# MOC — ADRs

Decisoes arquiteturais do `rustcraft`.

## ADRs aceitas

- [[0001-cargo-workspace-boundaries]] — Cargo workspace com fronteiras graduais.
- [[0002-game-actions-not-protocol]] — usar actions/controls em vez de protocol para input local.
- [[0003-multi-crate-workspace-for-learning]] — multi-crate workspace didatico.

## Atencao

ADR-0001 e ADR-0003 entraram em tensao: a primeira favorecia package unica inicial, a terceira aceita multi-crate desde Fase 0. A decisao mais recente e ADR-0003, e o codigo passou a acompanha-la em 2026-05-27. Ver [[../../11-audit/AUDIT#A-006]] e [[../../11-audit/audit-log/2026-05-27-multi-crate-restructure]].

## Links

- [[../../STATE]]
- [[../workspace-and-crate-organization]]
