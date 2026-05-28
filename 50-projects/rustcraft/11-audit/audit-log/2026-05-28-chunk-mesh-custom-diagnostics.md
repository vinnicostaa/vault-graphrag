---
title: Audit log — chunk mesh custom diagnostics
type: audit
tags:
  - audit-log
  - rustcraft
  - diagnostics
  - bevy
created: '2026-05-28'
updated: '2026-05-28'
---
# Audit log — chunk mesh custom diagnostics

## Contexto

A task T-BEVY-004 precisava sair de diagnosticos gerais do Bevy e registrar metricas proprias do mundo voxel.

## Mudancas verificadas

- `rc-world/src/diagnostics.rs` passou a registrar `rustcraft/chunks_rendered`, `rustcraft/chunk_faces` e `rustcraft/chunk_vertices`.
- `WorldPlugin` chama o registro dos diagnosticos antes do spawn inicial.
- `spawn_initial_chunk` constroi `ChunkMeshData`, mede faces e vertices, registra as medicoes e depois converte os dados para `Mesh`.
- `README.md` registra que diagnosticos de runtime existem e que os diagnosticos proprios ainda cobrem apenas o chunk inicial.

## Evidencias

Com `cargo run`, o console registrou:

- `rustcraft/chunks_rendered = 1`
- `rustcraft/chunk_faces = 1776`
- `rustcraft/chunk_vertices = 7104`
- `entity_count = 20`

A relacao `1776 faces * 4 vertices = 7104 vertices` confirma coerencia basica entre face count e vertex count.

## Validacao

- `cargo fmt --all -- --check`: passou.
- `cargo check --workspace`: passou.
- `cargo test --workspace`: passou no log do usuario.
- `cargo clippy --workspace --all-targets --all-features -- -D warnings`: passou no log do usuario.
- `git diff --check`: passou localmente.

## Limitacao registrada

Enquanto o mundo tem apenas o chunk inicial, `chunks_rendered` e uma metrica agregada suficiente. A separacao entre chunks carregados, visiveis, dirty e renderizados deve entrar junto com streaming/chunk map.

## Links

- [[../../05-tasks/bevy-example-study-tasks]]
- [[../../10-schedule/sprint-capability-map]]
- [[2026-05-28-base-runtime-diagnostics]]
- [[2026-05-28-chunk-mesh-spawn]]
