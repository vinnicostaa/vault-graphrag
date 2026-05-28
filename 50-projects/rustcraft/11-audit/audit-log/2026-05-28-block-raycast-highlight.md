---
title: Audit log — block raycast highlight
type: audit
tags:
  - audit-log
  - rustcraft
  - raycast
  - interaction
created: '2026-05-28'
updated: '2026-05-28'
---
# Audit log — block raycast highlight

## Contexto

T-BEVY-005 adicionou a primeira interacao de bloco sem voltar ao modelo de entidade por bloco.

## Mudancas verificadas

- `RustcraftInteractionPlugin` foi adicionado na composicao do app.
- O sistema de interacao roda em `PostUpdate` depois de `TransformSystems::Propagate`.
- O raycast usa `MeshRayCast` com filtro para entidades `GeneratedChunk`.
- `rc-voxel::block_pos_from_hit` converte ponto/normal de hit em `BlockPos` usando deslocamento pequeno para dentro da face.
- O bloco mirado recebe highlight visual via `Gizmos::cube`.
- `ARCHITECTURE.md` e `README.md` foram atualizados para refletir o fluxo atual.

## Validacao

- `cargo fmt --all -- --check`: passou.
- `cargo check --workspace`: passou.
- `cargo test --workspace`: passou, com 33 testes no total.
- `cargo clippy --workspace --all-targets --all-features -- -D warnings`: passou.

## Limitacao registrada

A interacao ainda e apenas highlight debug. O proximo passo de gameplay deve separar estado de bloco mirado e depois implementar quebrar/colocar bloco com chunk dirty.

## Links

- [[../../05-tasks/bevy-example-study-tasks]]
- [[../../06-execution/2026-05-28-t-bevy-005-raycast-start]]
- [[2026-05-28-chunk-mesh-spawn]]
