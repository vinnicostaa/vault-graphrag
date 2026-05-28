---
title: Plano de organização interna das crates — rustcraft
type: architecture-note
tags:
  - rustcraft
  - architecture
  - modules
  - crates
  - organization
created: '2026-05-27'
updated: '2026-05-27'
---
# Plano de organização interna das crates — rustcraft

Este plano organiza as crates existentes seguindo o mesmo princípio aplicado em `rc-voxel`: `lib.rs` vira a porta de entrada da crate, e os arquivos internos representam domínios/responsabilidades.

## Regra geral

Cada crate deve ter:

```text
src/lib.rs          # API pública e composição interna
src/plugin.rs       # Plugin Bevy, quando a crate expõe plugin
src/config.rs       # Resources/configurações
src/components.rs   # Components ECS públicos da crate
src/systems.rs      # Sistemas Bevy pequenos ou agregadores
```

Nem toda crate precisa de todos esses arquivos. Arquivo só nasce quando há responsabilidade real.

## Critérios de boa separação

- `lib.rs` não deve virar arquivo de implementação grande.
- Tipos públicos ficam em arquivos de domínio e são reexportados por `lib.rs`.
- Sistemas Bevy ficam perto do plugin que os registra.
- Configuração entra em `config.rs` quando for recurso independente.
- Componentes ECS entram em `components.rs` quando são parte da linguagem pública da crate.
- Código puro/testável deve ficar longe de `Commands`, `Query`, `Res`, `Handle`, `Mesh`, `Material` quando possível.

## `rc-input`

Responsabilidade: transformar input físico do Bevy em ações semânticas de jogo.

Estrutura sugerida:

```text
crates/rc-input/src/
├── lib.rs
├── actions.rs
├── bindings.rs
├── plugin.rs
└── state.rs
```

Conteúdo:

- `actions.rs`: `PlayerAction`.
- `state.rs`: `ActionState` e métodos `press`, `pressed`, `clear`, `iter`.
- `bindings.rs`: `KEY_BINDINGS` e, no futuro, remapeamento teclado/gamepad.
- `plugin.rs`: `InputPlugin`, `InputSet`, `collect_keyboard_actions`.
- `lib.rs`: `mod ...` e `pub use ...`.

API pública recomendada:

```rust
pub use actions::PlayerAction;
pub use plugin::{InputPlugin, InputSet};
pub use state::ActionState;
```

## `rc-player`

Responsabilidade: player/câmera/controlador inicial.

Estrutura sugerida:

```text
crates/rc-player/src/
├── lib.rs
├── camera.rs
├── components.rs
├── config.rs
├── movement.rs
└── plugin.rs
```

Conteúdo:

- `components.rs`: `Player`.
- `config.rs`: `PlayerConfig`.
- `camera.rs`: `spawn_player` enquanto player e camera ainda nascem juntos.
- `movement.rs`: `move_player`.
- `plugin.rs`: `PlayerPlugin`.
- `lib.rs`: reexports.

Observação: quando entrar mouse look/raycast, criar `look.rs` ou `controller.rs`. Não colocar raycast em `rc-world`; o ray sai do player/camera, mas a mutação do mundo fica em `rc-world`.

## `rc-render`

Responsabilidade: recursos visuais Bevy, iluminação, materiais e, depois, meshing de chunk.

Estrutura sugerida agora:

```text
crates/rc-render/src/
├── lib.rs
├── assets.rs
├── config.rs
├── lighting.rs
├── materials.rs
└── plugin.rs
```

Conteúdo atual:

- `config.rs`: `RenderConfig`.
- `assets.rs`: `BlockRenderAssets` e `setup_block_assets`.
- `materials.rs`: `block_material` e, depois, mapeamento material/textura por bloco.
- `lighting.rs`: `setup_lighting`.
- `plugin.rs`: `RenderPlugin`, `RenderStartupSet`.
- `lib.rs`: reexports.

Estrutura futura, quando chunk mesh entrar:

```text
crates/rc-render/src/
├── meshing.rs        # build_chunk_mesh
├── mesh_data.rs      # structs auxiliares de vertices/faces, se necessário
└── chunk_render.rs   # spawn/update da entidade renderizável do chunk
```

Regra: `rc-render` pode conhecer `BlockState` e `BlockDefinition`, mas não deve decidir geração de mundo.

## `rc-world`

Responsabilidade: mundo, geração, lifecycle de chunks e spawn inicial enquanto o projeto ainda é protótipo.

Estrutura sugerida agora:

```text
crates/rc-world/src/
├── lib.rs
├── components.rs
├── config.rs
├── generation.rs
├── plugin.rs
└── spawn.rs
```

Conteúdo atual:

- `components.rs`: `Block`, `GeneratedChunkBlock` enquanto ainda existe entidade por bloco no protótipo.
- `config.rs`: `WorldConfig`.
- `generation.rs`: `terrain_height` e regras de geração de terreno.
- `spawn.rs`: `spawn_initial_chunk`.
- `plugin.rs`: `WorldPlugin`.
- `lib.rs`: reexports.

Estrutura futura, quando chunk entrar:

```text
crates/rc-world/src/
├── chunk_map.rs       # armazenamento dos chunks carregados
├── dirty.rs           # chunks alterados que precisam remesh
├── streaming.rs       # load/unload por distância do player
└── events.rs          # BlockChanged, ChunkGenerated, ChunkLoaded
```

Regra: `rc-world` decide quais blocos existem e onde. `rc-render` decide como eles aparecem.

## `rustcraft`

Responsabilidade: composição do app.

Estrutura atual está boa:

```text
crates/rustcraft/src/
├── lib.rs
├── app.rs
└── bin/rustcraft.rs
```

Por enquanto não precisa quebrar mais. Quando entrar estados de jogo, considerar:

```text
crates/rustcraft/src/
├── app.rs
├── plugins.rs
└── states.rs
```

Mas só depois de `GameState` existir.

## Ordem recomendada de aplicação

1. `rc-input`: menor e mais fácil; bom para treinar `mod`/`pub use`.
2. `rc-render`: já está sendo mexido por causa de `BlockState`.
3. `rc-world`: separar geração/spawn/config antes de chunks.
4. `rc-player`: separar componente/config/movement depois.
5. `rustcraft`: deixar como está até aparecer `GameState`.

## O que não fazer agora

- Não criar crate nova ainda.
- Não mover `rc-worldgen` ou `rc-meshing` para crate própria antes de haver chunk real.
- Não transformar tudo em `pub mod`; preferir módulos privados e reexports explícitos.
- Não criar `systems.rs` gigante se houver domínios mais claros como `movement.rs`, `lighting.rs`, `spawn.rs`.

## Links internos

- [[workspace-and-crate-organization]]
- [[voxel-entity-rendering-model]]
- [[minecraft-modloader-api-lessons]]
- [[../10-schedule/sprint-capability-map]]
