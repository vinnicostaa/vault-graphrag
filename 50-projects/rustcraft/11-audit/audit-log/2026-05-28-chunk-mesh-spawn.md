---
title: Audit log — Chunk mesh spawn
type: audit-log
tags:
  - rustcraft
  - audit-log
  - bevy
  - voxel
  - chunk-mesh
created: '2026-05-28'
updated: '2026-05-28'
---
# Audit log — Chunk mesh spawn

Resumo:

- `spawn_initial_chunk` deixou de percorrer cada bloco solido para criar uma entidade renderizavel por bloco.
- O spawn inicial agora gera um `Chunk`, chama `rc_render::build_chunk_mesh`, adiciona a mesh em `Assets<Mesh>` e spawna uma entidade com `Mesh3d`, `MeshMaterial3d` e `GeneratedChunk`.
- `GeneratedChunk` foi adicionado para marcar a entidade de chunk gerado.
- `BlockRenderAssets::chunk_material` foi adicionado como material temporario unico do chunk.
- `T-BEVY-003` foi fechado como done.
- `DT-011` foi fechado: o caminho principal ja usa entidade renderizavel por chunk.

Validacao:

- `cargo fmt --all -- --check`
- `cargo check --workspace`
- `cargo test --workspace`
- `cargo clippy --workspace --all-targets --all-features -- -D warnings`

Limitacao remanescente:

- O chunk usa material unico temporario. A proxima frente visual deve usar atlas, vertex color ou outro mecanismo de material por bloco/face sem voltar ao spawn por bloco.

Links:

- [[../../STATE]]
- [[../../05-tasks/bevy-example-study-tasks]]
- [[../../06-execution/next-task-chunk-spawn]]
- [[../../10-schedule/sprint-capability-map]]
