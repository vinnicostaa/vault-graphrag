---
title: Audit log — Reestruturação multi-crate rustcraft
type: audit
tags:
  - audit
  - rustcraft
  - cargo
  - workspace
  - crates
created: '2026-05-27'
updated: '2026-05-27'
---
# Audit log — Reestruturação multi-crate rustcraft

## Escopo

Corrigir a base do projeto para refletir as decisões do vault, especialmente ADR-0003 e a separação entre workspace, package/crate, bin target, lib target e módulos internos.

## Evidência de implementação

Estrutura implementada no repositório:

```text
crates/rustcraft
crates/rc-input
crates/rc-player
crates/rc-voxel
crates/rc-render
crates/rc-world
```

Responsabilidades verificadas:

- `rustcraft` compõe o app e expõe o bin target `src/bin/rustcraft.rs`.
- `rc-input` converte input Bevy em ações semânticas.
- `rc-player` consome `rc-input` para spawn/movimento do player.
- `rc-voxel` contém dados voxel puros e testes.
- `rc-render` concentra luz, mesh/material e recursos visuais.
- `rc-world` gera/spawna o mundo inicial usando `rc-voxel` e `rc-render`.

## Validação

Executado em 2026-05-27:

```sh
cargo check --workspace
cargo test --workspace
cargo fmt --all -- --check
cargo clippy --workspace --all-targets --all-features -- -D warnings
```

Resultado inicial: todos passaram. `cargo test --workspace` executou 5 testes unitários relevantes (`rc-voxel` e `rc-world`) além dos targets sem testes.

## Snapshot git

Commit criado antes de prosseguir com a próxima etapa de aprendizado:

```text
28c419a feat: split rustcraft into workspace crates
```

Validação antes do commit:

```sh
cargo fmt --all -- --check
cargo check --workspace
cargo test --workspace
cargo clippy --workspace --all-targets --all-features -- -D warnings
```

Resultado: todos passaram. No momento do commit, `rc-voxel` continha 8 testes unitários e `rc-world` continha 2 testes unitários.

## Findings fechados

- A-001 — árvore Cargo agora separa workspace, packages/crates, lib target, bin target e módulos.
- A-004 — scaffold do vault está instanciado com áreas 00-12 e audit canônico.
- A-006 — ADR-0003 deixou de divergir do código.

## Pendências preservadas

- A-009 — decidir política definitiva para `AUDIT.md` raiz.
- T-003 — validação visual do protótipo em janela Bevy ainda não foi executada nesta rodada.
- T-001 — exercício pedagógico de explicar workspace/package/crate/module ainda pode ser feito pelo usuário.

## Correção posterior de consistência

Ver [[2026-05-27-repository-consistency-correction]]. A-007 foi fechado porque `context-1.md` e `context-2.md` nao existem no workspace atual; `.gitignore` passou a ignorar `context-*.md`. O histórico `minecraft` -> `rustcraft` também foi registrado.

## Links

- [[../AUDIT]]
- [[../../STATE]]
- [[../../02-architecture/adr/0003-multi-crate-workspace-for-learning]]
- [[../../02-architecture/workspace-and-crate-organization]]
