---
title: Cargo workspace e organização de crates — rustcraft
type: guide
tags:
  - architecture
  - rustcraft
  - cargo
  - workspace
  - crates
created: '2026-05-27'
updated: '2026-05-27'
---
# Cargo workspace e organização de crates — rustcraft

## Correção conceitual

A referência correta é a distinção do Rust Book:

- **Package**: recurso do Cargo com um `Cargo.toml`.
- **Crate**: árvore de módulos que compila para uma library ou executable.
- **Module**: organização interna dentro de uma crate via `mod`.
- **Workspace**: conjunto de packages relacionados, com `Cargo.lock` e `target/` compartilhados.

Logo, se queremos exercitar Cargo workspace de verdade, uma estrutura como `crates/rc-input`, `crates/rc-render`, etc. é válida — cada pasta dessas é uma **package** que normalmente expõe uma **library crate**.

## Estrutura implementada para estudar workspace desde cedo

```text
rustcraft/
├── Cargo.toml
├── Cargo.lock
├── README.md
├── ARCHITECTURE.md
└── crates/
    ├── rustcraft/          # package principal: compõe o jogo
    │   ├── Cargo.toml
    │   └── src/
    │       ├── app.rs
    │       ├── lib.rs
    │       └── bin/
    │           └── rustcraft.rs
    ├── rc-input/           # library crate: input bruto -> actions
    │   ├── Cargo.toml
    │   └── src/
    │       └── lib.rs
    ├── rc-player/          # library crate: player/camera/controller
    │   ├── Cargo.toml
    │   └── src/
    │       └── lib.rs
    ├── rc-voxel/           # library crate pura: blocos/chunks/dados voxel
    │   ├── Cargo.toml
    │   └── src/
    │       └── lib.rs
    ├── rc-render/          # library crate: luz, materiais, meshes, plugins render
    │   ├── Cargo.toml
    │   └── src/
    │       └── lib.rs
    └── rc-world/           # library crate: geração/spawn do mundo Bevy
        ├── Cargo.toml
        └── src/
            └── lib.rs
```

Em Rust, packages com hífen são importadas com underscore no código:

```rust
// package: rc-input
// crate path no código:
use rc_input::InputPlugin;
```

## Grafo de dependências inicial

O ponto mais importante não é a árvore de pastas; é evitar ciclos.

```text
rustcraft app
├── rc_input
├── rc_player ──→ rc_input
├── rc_voxel
├── rc_render ──→ rc_voxel
└── rc_world  ──→ rc_voxel, rc_render
```

Regras:

- `rc-voxel` deve ser o mais puro possível: sem Bevy render, sem input, sem player.
- `rc-input` conhece Bevy input e produz ações semânticas.
- `rc-player` consome ações; não deve ler `KeyCode` diretamente.
- `rc-render` transforma dados/handles visuais em recursos Bevy.
- `rc-world` gera/spawna o terreno inicial.
- `rustcraft` só compõe plugins e roda o app.

## O que cada crate exporta

### `rc-input`

Responsabilidade:

- `PlayerAction`;
- `ActionState`;
- mapeamento `KeyCode` → `PlayerAction`;
- `InputPlugin`.

Evitar:

- mover câmera;
- gerar mundo;
- escolher material de bloco.

### `rc-player`

Responsabilidade:

- `Player` component;
- spawn da câmera/player;
- sistema de movimento;
- `PlayerPlugin`.

Depende de:

- `rc-input`;
- Bevy.

### `rc-voxel`

Responsabilidade:

- `BlockType`;
- futuramente `Chunk`, `ChunkCoord`, direção de faces, armazenamento voxel;
- funções puras testáveis.

Evitar:

- `Mesh`, `StandardMaterial`, `Color`, `Commands`, `Transform`.

### `rc-render`

Responsabilidade:

- iluminação;
- mesh/material compartilhados;
- assets visuais de blocos;
- futuramente meshing/render plugin.

Depende de:

- Bevy;
- `rc-voxel` para saber que tipos de bloco existem.

### `rc-world`

Responsabilidade:

- geração inicial do terreno;
- spawn de entidades do mundo;
- futuramente chunk lifecycle.

Depende de:

- Bevy;
- `rc-voxel`;
- talvez `rc-render` enquanto o spawn ainda usa mesh/material diretamente.

### `rustcraft`

Responsabilidade:

- `fn main()`;
- `App::new()`;
- `DefaultPlugins`;
- registrar plugins internos.

Não deve conter regra de jogo.

## Quando isso é melhor do que módulos internos?

Essa estrutura é melhor se o objetivo atual inclui aprender:

- Cargo workspace;
- dependências path entre packages;
- APIs públicas entre crates;
- limites reais impostos pelo compilador;
- `cargo check -p <package>`;
- organização parecida com projetos Rust maiores.

Ela custa mais boilerplate, mas ensina o modelo correto.

## Quando extrair mais crates?

Depois, podem surgir:

```text
crates/rc-worldgen/       # geração procedural pura
crates/rc-meshing/        # mesh por chunk, talvez sem Bevy inicialmente
crates/rc-physics/        # integração Rapier
crates/rc-ui/             # HUD/debug/menu
crates/xtask/             # automação do workspace
```

Mas só quando houver fronteira clara.

## Vocabulário recomendado

| Conceito | Nome recomendado | Evitar |
| --- | --- | --- |
| Ações semânticas do jogador | `actions` | `protocol` |
| Mapeamento teclado/gamepad → ação | `bindings`, `controls` | `keyboard_protocol` |
| Movimento/câmera | `player`, `controller` | `camera_hack` |
| Blocos/chunks puros | `voxel` | misturar em `render` |
| Luz/material/mesh Bevy | `render` / `rendering` | misturar em `world` |
| UI/HUD/debug | `ui` | `menu` genérico demais se crescer |

## Inspirações externas

- Rust Book Chapter 7: packages, crates, modules e privacidade.
- Rust Book Chapter 14: workspaces como conjunto de packages relacionados.
- Bevy: plugins como unidade natural de composição.
- Unity Input System: `Actions` separam significado lógico do controle físico.
- Unreal: Controller/Pawn separam decisão de controle da entidade controlada.
- Game Programming Patterns: componentes separam domínios como input, physics, rendering e audio.
