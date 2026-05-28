---
title: Tasks — Bevy example study and voxel architecture
type: tasks
tags:
  - rustcraft
  - tasks
  - bevy
  - voxel
  - sprints
created: '2026-05-27'
updated: '2026-05-28'
---
# Tasks — Bevy example study and voxel architecture

Tasks derivadas da pesquisa dos exemplos Bevy e da duvida sobre entidades/blocos.

## T-BEVY-000 — Migrar modelo de bloco prototipo

Status: done

Resultado em 2026-05-27:

- `BlockId`, `BlockState`, `BlockDefinition` e registry minimo foram criados em `rc-voxel`.
- `BlockType` deixou de ser o contrato principal.
- `rc-render` passou a receber `BlockState`.
- `rc-world` passou a usar `block_for_layer` retornando `BlockState`.
- `rc-voxel` foi organizado em módulos: `block`, `registry`, `position`, `generation`, `chunk`.


Objetivo: substituir o `enum BlockType` como contrato principal por `BlockId`, `BlockState`, `BlockDefinition` e `BlockRegistry` minimo.

Criterios:

- `BlockId` compacto (`u16`) existe;
- `BlockState { id, variant }` existe;
- constantes/definicoes iniciais para `air`, `grass`, `dirt`, `stone` existem;
- `BlockDefinition` centraliza `solid`, `opaque`, `hardness` e render kind minimo;
- `BlockRegistry` responde consultas como `is_air`, `is_solid`, `is_opaque`;
- `rc-voxel` continua sem dependencia de Bevy;
- `rc-world` e `rc-render` deixam de depender de `match BlockType` como fonte final.

## T-BEVY-001 — Implementar `Chunk` puro

Status: done

Resultado em 2026-05-27:

- `Chunk` puro foi criado em `rc-voxel/src/chunk.rs`.
- O armazenamento inicial usa `Vec<BlockState>`.
- `new_filled`, `get`, `set`, `size` e indexacao local `x/y/z` foram implementados e testados.
- `Chunk` foi exportado pela API pública de `rc-voxel`.
- `generate_chunk` em `rc-world` usa `ChunkCoord::origin_block_pos` para gerar dados de chunk por coordenada global.

Observação:

- A renderizacao principal ainda usa o spawn prototipo de entidade por bloco; a troca para entidade/mesh por chunk fica para T-BEVY-003.

Objetivo: criar armazenamento de blocos sem Bevy, usando `BlockState` em vez de entidade por bloco.

Arquivos provaveis:

- `crates/rc-voxel/src/lib.rs`

Criterios:

- `ChunkCoord` usado para origem;
- indexacao x/y/z testada;
- `get`/`set` seguros;
- nenhum import de Bevy em `rc-voxel`.

## T-BEVY-002 — Gerar terreno como dado, nao entidade

Status: done

Resultado em 2026-05-27:

- `rc-world::generate_chunk` retorna `Chunk` como dado puro.
- A geracao usa `TerrainGenerator`, `WorldSeed`, `ChunkCoord`, `Chunk::set` e `block_for_layer`.
- Testes de altura continuam existindo.
- `BlockType::Air` foi substituido por `BlockState::air()` no caminho de chunk data.
- `spawn_initial_chunk` passou a ler os blocos a partir de `Chunk`, mantendo temporariamente uma entidade renderizavel por bloco nao vazio.

Observacao:

- A renderizacao ainda nao e por mesh de chunk; o custo de CPU alto continua esperado ate T-BEVY-003.

Objetivo: `rc-world` deve produzir `Chunk`, nao spawnar bloco individual.

Criterios:

- geracao retorna dados de chunk;
- testes de altura continuam existindo;
- `BlockType::Air` representado sem entidade.

## T-BEVY-003 — Criar mesh de chunk com faces expostas

Status: done

Resultado em 2026-05-28:

- `rc-render::ChunkMeshData` foi criado para armazenar positions, normals, UVs e indices.
- `rc-render::build_chunk_mesh_data` gera apenas faces expostas de um `Chunk`.
- `rc-render::build_chunk_mesh` converte os dados em `bevy::mesh::Mesh` usando `PrimitiveTopology::TriangleList`, atributos `POSITION`, `NORMAL`, `UV_0` e `Indices::U32`.
- O uso de `RenderAssetUsages::MAIN_WORLD | RenderAssetUsages::RENDER_WORLD` segue o padrao dos exemplos oficiais Bevy para mesh editavel.
- `spawn_initial_chunk` passou a criar uma entidade renderizavel por chunk, em vez de uma entidade por bloco nao-ar.
- Testes cobrem chunk vazio, bloco isolado, blocos adjacentes, borda de chunk, escala de vertices, consistencia de buffers e conversao final para `Mesh`.
- Validacao realizada: `cargo fmt --all -- --check`, `cargo check --workspace`, `cargo test --workspace`, `cargo clippy --workspace --all-targets --all-features -- -D warnings`.

Limitacao remanescente:

- o chunk usa material unico temporario ate existir atlas, vertex color ou outro caminho de material por bloco/face.

Objetivo: gerar `Mesh` a partir de `Chunk`.

Criterios:

- faces internas entre blocos solidos nao sao emitidas;
- normals e UVs existem;
- uma entidade por chunk renderizavel;
- teste cobre face exposta simples.

Referencia: Bevy `generate-custom-mesh`.

## T-BEVY-004 — Adicionar diagnosticos de chunk/render

Status: done

Resultado em 2026-05-28:

- `crates/rustcraft/src/diagnostics.rs` criado com `RustcraftDiagnosticsPlugin`.
- O app passou a registrar `LogDiagnosticsPlugin`, `FrameTimeDiagnosticsPlugin`, `EntityCountDiagnosticsPlugin` e `SystemInformationDiagnosticsPlugin`.
- `rc-world/src/diagnostics.rs` registra `rustcraft/chunks_rendered`, `rustcraft/chunk_faces` e `rustcraft/chunk_vertices` via `DiagnosticPath` customizado.
- `spawn_initial_chunk` mede faces e vertices a partir de `ChunkMeshData` antes de converter os dados em `Mesh` Bevy.
- `cargo run` confirmou logs de `fps`, `frame_time`, `frame_count`, `entity_count`, CPU, memoria, chunks, faces e vertices no console.
- Valores observados: `entity_count = 20`, `chunks_rendered = 1`, `chunk_faces = 1776`, `chunk_vertices = 7104`.
- A relacao `1776 faces * 4 vertices = 7104 vertices` confirma consistencia basica dos contadores de mesh.

Limitacao remanescente:

- enquanto existe apenas chunk inicial, `chunks_rendered` funciona como proxy de chunk carregado/renderizado; a separacao entre carregado, visivel e dirty entra quando streaming existir.

Objetivo: enxergar custo antes de otimizar.

Criterios:

- contador de chunks carregados;
- contador de chunks visiveis/renderizados;
- contador de faces/vertices da mesh;
- log ou overlay de FPS.

Referencias: Bevy `custom-diagnostic`, `log-diagnostics`, `fps-overlay`.

## T-BEVY-005 — Raycast de bloco

Status: done

Resultado em 2026-05-28:

- `crates/rustcraft/src/interaction.rs` criou `RustcraftInteractionPlugin`.
- O raycast sai da camera/player usando `Ray3d` e `Transform::forward()`.
- `MeshRayCast` usa filtro por `GeneratedChunk`, mantendo a interacao independente de entidade por bloco.
- `rc-voxel::block_pos_from_hit` converte `RayMeshHit.point` + normal da face em `BlockPos` sem depender de Bevy.
- O bloco mirado recebe highlight debug com `Gizmos::cube`.
- Testes em `rc-voxel::position` cobrem face superior, face +X, coordenada negativa e tamanho de bloco invalido.
- Validacao realizada: `cargo fmt --all -- --check`, `cargo check --workspace`, `cargo test --workspace`, `cargo clippy --workspace --all-targets --all-features -- -D warnings`.

Limitacao remanescente:

- a interacao ainda e visual/debug; quebrar, colocar bloco, estado de bloco mirado e chunk dirty entram em tasks futuras.

Observacao tecnica:

- o projeto usa Bevy `0.18.1`; a API confirmada vem de `MeshRayCast`, `MeshRayCastSettings`, `Ray3d` e `RayMeshHit` nos fontes oficiais locais do Bevy/Cargo.

Objetivo: mirar em um bloco e descobrir `BlockPos`.

Criterios:

- ray sai da camera/player;
- converte hit em coordenada de bloco;
- mostra gizmo/highlight temporario;
- nao depende de entidade por bloco.

Referencia: Bevy `mesh-ray-cast`.

## T-BEVY-009 — Mouse look / câmera de primeira pessoa

Status: done

Objetivo: permitir que o mouse controle a direção da câmera/player, para o raycast de bloco ficar usável.

Resultado em 2026-05-28:

- `PlayerConfig` ganhou `mouse_sensitivity` para parametrizar a velocidade de rotação.
- `rc-player/src/look.rs` foi criado com o sistema `look_player`.
- O sistema usa `AccumulatedMouseMotion`, `EulerRot::YXZ` e clamp de pitch para evitar virar a câmera além da vertical.
- `PlayerPlugin` registra `(look_player, move_player).chain()`, garantindo que a rotação do mouse seja aplicada antes do movimento WASD no frame.
- O movimento continua relativo ao `Transform` atual do player/câmera.
- Validação realizada: `cargo fmt --all -- --check`, `cargo check --workspace`, `cargo test --workspace`, `cargo clippy --workspace --all-targets --all-features -- -D warnings`.

Limitacao remanescente:

- o cursor ainda não é capturado/travado na janela; isso fica para a próxima task de controle de janela/input.

Criterios:

- mover mouse gira a câmera/player;
- pitch nao passa de quase 90 graus para cima/baixo;
- WASD continua movendo relativo à direção atual;
- raycast/highlight acompanha a mira;
- sem log por frame.

Referencias: Bevy `first_person_view_model`, `physics_in_fixed_timestep`, `mouse_grab`.

## T-BEVY-006 — Settings iniciais

Status: open

Objetivo: resources para configuracoes que serao expostas no menu.

Criterios:

- render distance;
- mouse sensitivity;
- master volume;
- shadows on/off;
- window title/resolution basicos.

Referencias: Bevy `game-menu`, `window-settings`, `audio-control`.

## T-BEVY-007 — Seed e gerador deterministico de terreno

Status: done

Objetivo: introduzir seed no contrato de geracao procedural sem alterar ainda o render.

Resultado em 2026-05-27:

- `WorldSeed` foi criado em `rc-world::config`.
- `WorldConfig` passou a incluir `seed`.
- `TerrainGenerator` foi criado em `rc-world::generation`.
- `height_at(&self, x, z)` gera altura deterministica com seed.
- `terrain_height` foi mantida como compatibilidade chamando a implementacao com `WorldSeed(0)`.
- Testes cobrem mesma seed gerando mesma altura e seeds diferentes gerando altura diferente no ponto testado.

## T-BEVY-008 — Organizar crates por módulos de domínio

Status: done

Objetivo: deixar `lib.rs` como API pública e mover implementação para módulos por responsabilidade.

Resultado em 2026-05-27:

- `rc-input`: `actions`, `bindings`, `state`, `plugin`.
- `rc-player`: `components`, `config`, `camera`, `movement`, `plugin`.
- `rc-render`: `assets`, `config`, `lighting`, `materials`, `plugin`.
- `rc-world`: `components`, `config`, `generation`, `spawn`, `plugin`.
- `rc-voxel`: `block`, `registry`, `position`, `generation`, `chunk`.
- Validação realizada: `cargo fmt --all`, `cargo test --workspace`, `cargo check --workspace`, `cargo clippy --workspace --all-targets --all-features -- -D warnings`.

## Links internos

- [[../02-architecture/bevy-examples-capability-map]]
- [[../02-architecture/voxel-entity-rendering-model]]
- [[../10-schedule/sprint-capability-map]]
