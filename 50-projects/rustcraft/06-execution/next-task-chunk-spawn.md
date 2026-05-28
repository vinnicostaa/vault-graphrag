---
title: Próxima task — spawn por chunk mesh
type: execution-note
tags:
  - rustcraft
  - task
  - voxel
  - bevy
  - chunk-mesh
created: '2026-05-28'
updated: '2026-05-28'
---
# Task concluida — spawn por chunk mesh

Status: done em 2026-05-28

Objetivo concluido: o usuario trocou o spawn principal de uma entidade por bloco para uma entidade renderizavel por chunk, usando a API pronta de `rc-render`.

## Arquivo alvo principal

- `crates/rc-world/src/spawn.rs`

## Arquivos que talvez precisem de ajuste

- `crates/rc-world/src/components.rs`
- `crates/rc-world/src/lib.rs`
- `crates/rc-render/src/assets.rs`, somente se faltar material temporario de chunk

## Estado antes

`spawn_initial_chunk` gera um `Chunk`, percorre cada bloco nao-ar e faz `commands.spawn` para cada bloco.

## Estado desejado

`spawn_initial_chunk` deve:

1. gerar o `Chunk` como hoje;
2. chamar `rc_render::build_chunk_mesh(&chunk, block_size)`;
3. adicionar a mesh em `Assets<Mesh>`;
4. spawnar uma entidade com `Mesh3d(handle)`, `MeshMaterial3d(material)` e `Transform` na origem do chunk.

## Conceitos Rust/Bevy envolvidos

- `ResMut<Assets<Mesh>>`: recurso mutavel para inserir uma mesh nova no asset storage do Bevy.
- `meshes.add(mesh)`: retorna `Handle<Mesh>`.
- `Mesh3d(handle)`: componente renderizavel 3D do Bevy 0.18.
- `MeshMaterial3d(material)`: componente que liga a entidade a um material 3D.
- `Transform::from_xyz(...)`: posiciona a mesh no mundo.

## Decisao temporaria

Como ainda nao existe atlas, vertex color ou material por face, a primeira versao pode usar um material unico temporario para o chunk. Isso fecha o gap estrutural de entidades demais sem resolver ainda a aparencia final por tipo de bloco.

## Criterios de pronto

- `spawn_initial_chunk` nao deve mais ter loops que fazem `commands.spawn` por bloco.
- Deve existir apenas um spawn renderizavel para o chunk inicial.
- O codigo deve compilar sem warnings.
- Gates:

```sh
cargo fmt --all -- --check
cargo test --workspace
cargo clippy --workspace --all-targets --all-features -- -D warnings
```

## Resultado da revisao

- `T-BEVY-003` fechado como done.
- `DT-011` fechado no `STATE`.
- Gates passaram no workspace.

## Proxima frente

- adicionar diagnosticos de chunk/render;
- recuperar visual por tipo de bloco sem voltar ao spawn por bloco.
