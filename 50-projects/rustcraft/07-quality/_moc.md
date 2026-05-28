---
title: MOC — rustcraft 07-quality
type: moc
tags:
  - moc
  - rustcraft
  - quality
created: '2026-05-27'
updated: '2026-05-28'
---
# MOC — 07-quality

Qualidade tecnica do `rustcraft`.

## Gates atuais

```sh
cargo check --workspace
cargo test --workspace
cargo fmt --all -- --check
cargo clippy --workspace --all-targets --all-features -- -D warnings
```

## Regras iniciais

- Separar dominio voxel de renderizacao Bevy.
- Separar input fisico de acoes semanticas.
- Manter funcoes puras testaveis quando possivel.
- Evitar crates artificiais sem fronteira explicavel.
- Documentar trade-offs antes de refactor grande.

## Guias

- [[coding-practices]] — padrao de codificacao, documentacao, gates e fluxo didatico.

## Notas a criar

- `code-review-checklist.md`.
- `rust-bevy-quality-guide.md`.

## Links

- [[../STEERING]]
- [[../11-audit/AUDIT]]
- [[../../20-stacks/rust/_moc]]
