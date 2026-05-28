---
title: Audit log — Chunk meshing API
type: audit-log
tags:
  - rustcraft
  - audit-log
  - bevy
  - voxel
  - render
created: '2026-05-28'
updated: '2026-05-28'
---
# Audit log — Chunk meshing API

Resumo:

- `rc-render` ganhou `meshing.rs` com API publica para meshing de chunks.
- `ChunkMeshData` guarda positions, normals, UVs e indices sem depender de ECS/Assets Bevy.
- `build_chunk_mesh_data` emite apenas faces expostas e remove faces internas entre blocos solidos do mesmo chunk.
- `build_chunk_mesh` converte o resultado para `bevy::mesh::Mesh` com `PrimitiveTopology::TriangleList`, atributos `POSITION`, `NORMAL`, `UV_0` e `Indices::U32`.
- O uso de `RenderAssetUsages::MAIN_WORLD | RenderAssetUsages::RENDER_WORLD` foi alinhado ao exemplo oficial Bevy `generate-custom-mesh` para meshes editaveis.
- Testes de `rc-render` subiram para 7 e cobrem a conversao final para `Mesh`.

Validacao local:

- `cargo fmt --all -- --check`
- `cargo test -p rc-render`
- `cargo test --workspace`
- `cargo clippy --workspace --all-targets --all-features -- -D warnings`

Pendencia mantida no momento deste audit:

- DT-011 continuava aberta: o spawn principal ainda criava uma entidade renderizavel por bloco; a proxima task deveria usar a mesh por chunk no spawn do mundo.

Atualizacao posterior em 2026-05-28:

- DT-011 foi fechada por [[2026-05-28-chunk-mesh-spawn]].

Links:

- [[../../STATE]]
- [[../../05-tasks/bevy-example-study-tasks]]
- [[../../10-schedule/sprint-capability-map]]
