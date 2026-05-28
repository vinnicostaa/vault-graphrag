---
title: ADR-0003 — Multi-crate workspace didático
type: adr
tags:
  - adr
  - rustcraft
  - cargo
  - workspace
  - crates
status: accepted
date: '2026-05-27'
created: '2026-05-27'
updated: '2026-05-27'
---
# ADR-0003 — Multi-crate workspace didático

## Status

accepted — 2026-05-27

## Contexto

A primeira proposta manteve uma package jogável única com módulos internos. Isso é simples, mas não exercita plenamente Cargo workspace.

O usuário apontou corretamente que, seguindo a ideia de packages/crates/workspaces do Rust Book, uma estrutura com packages internas como `crates/rc-input` e `crates/rc-render` pode ser mais adequada ao objetivo de aprendizado.

## Decisão

Adotar uma estrutura multi-crate desde a Fase 0, mas com poucas crates e fronteiras explícitas:

```text
crates/rustcraft   # bin principal
crates/rc-input    # input bruto -> ações semânticas
crates/rc-player   # player/camera/controller
crates/rc-voxel    # dados voxel puros
crates/rc-render   # luz/material/mesh/render plugin
crates/rc-world    # geração/spawn inicial do mundo
```

O package `rustcraft` será o app principal e vai compor plugins das crates internas. A implementação usa `src/lib.rs`/`src/app.rs` para a composição e `src/bin/rustcraft.rs` como bin target.

## Implementação

Implementada no repositório em 2026-05-27 com:

- `crates/rustcraft`
- `crates/rc-input`
- `crates/rc-player`
- `crates/rc-voxel`
- `crates/rc-render`
- `crates/rc-world`

Validação registrada: `cargo check --workspace`, `cargo test --workspace`, `cargo fmt --all -- --check` e `cargo clippy --workspace --all-targets --all-features -- -D warnings` passaram.

## Regra de dependência

```text
rustcraft app
├── rc_input
├── rc_player ──→ rc_input
├── rc_voxel
├── rc_render ──→ rc_voxel
└── rc_world  ──→ rc_voxel, rc_render
```

Evitar ciclos. Se uma dependência circular aparecer, a fronteira está errada e deve ser redesenhada.

## Consequências positivas

- Ensina Cargo workspace corretamente.
- Cria APIs públicas pequenas entre crates.
- Permite rodar `cargo check -p rc-voxel`, `cargo test -p rc-input`, etc.
- Aproxima o projeto de workspaces Rust reais.
- Facilita extrair partes puras/testáveis, especialmente `rc-voxel`.

## Consequências negativas

- Mais `Cargo.toml`.
- Mais cuidado com `pub`, `pub(crate)` e exports.
- Mais chance de criar fronteiras artificiais se não houver disciplina.
- Refactors simples podem tocar vários packages.

## Decisão sobre nomes

Usar prefixo curto `rc-` para packages internas:

- package: `rc-input`;
- crate path no código: `rc_input`.

Reservar o nome `rustcraft` para o executável/app principal.

## Relação com ADR anterior

Esta ADR substitui a direção inicial de manter tudo em uma única package principal. A ideia de evitar overengineering continua válida, mas o objetivo pedagógico de aprender workspace justifica a separação inicial controlada.
