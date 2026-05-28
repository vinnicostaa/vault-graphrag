---
title: MOC — rustcraft 03-subprojects
type: moc
tags:
  - moc
  - rustcraft
  - subprojects
created: '2026-05-27'
updated: '2026-05-27'
---
# MOC — 03-subprojects

Subprojetos/packages do `rustcraft`.

## Estado atual

Repositorio local usa Cargo workspace multi-crate conforme ADR-0003.

## Packages atuais

- `crates/rustcraft` — package de composição do app; expõe `app.rs`/`lib.rs` e bin target em `src/bin/rustcraft.rs`.
- `crates/rc-input` — input bruto Bevy para ações semânticas (`PlayerAction`, `ActionState`).
- `crates/rc-player` — spawn e movimento do player/câmera consumindo `rc-input`.
- `crates/rc-voxel` — tipos voxel puros e testáveis (`BlockType`).
- `crates/rc-render` — luz, mesh/material compartilhados e recursos visuais.
- `crates/rc-world` — geração/spawn inicial do mundo usando `rc-voxel` e `rc-render`.

Ver `[[../11-audit/AUDIT#A-006]]` e `[[../05-tasks/mvp-0-tasks#T-009 — Reconciliar ADR-0003 com o código atual]]` para o fechamento da divergência.

## Links

- [[../02-architecture/adr/0003-multi-crate-workspace-for-learning]]
- [[../02-architecture/workspace-and-crate-organization]]
- [[../STATE]]
