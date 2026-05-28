---
title: 'Research — organização de projetos Rust, Bevy e referências externas'
type: research-note
tags:
  - rustcraft
  - rust
  - bevy
  - zed
  - cosmic
  - 40tude
  - architecture
  - modules
created: '2026-05-27'
updated: '2026-05-27'
source_date: '2026-05-27'
---
# Research — organização de projetos Rust, Bevy e referências externas

Nota de evidência para a organização interna das crates do `rustcraft`.

## Status da pesquisa

Referências fortes já analisadas:

- Rust Book: packages, crates, modules, privacy, `use`, `pub use` e workspaces.
- Cargo Book: layout convencional de packages.
- Rust API Guidelines: documentação, nomes e API pública.
- Bevy docs/examples: plugins como unidade de composição; plugins pequenos e focados.
- Minecraft mod loaders/Bedrock: registry, lifecycle, data-driven design e separação de estado/dados pesados.
- Unity/Unreal/Godot: entidades/actors/nodes para objetos com identidade; dados massivos agregados em estruturas próprias.

Referências complementares, não normativas:

- Zed: monorepo Rust grande, muitas crates em `crates/`, separação por domínio/feature, tooling forte.
- COSMIC/libcosmic: toolkit Rust organizado por módulos públicos de domínio, reexports e documentação gerada por `cargo doc`.
- 40tude: artigos didáticos sobre Rust, modular monolith e spec-driven development. Útil para didática e fluxo, mas não é fonte oficial de Rust nem de engine/game architecture.

## Rust oficial

### Packages/crates/modules

O Rust Book separa claramente:

- package: unidade Cargo com `Cargo.toml`;
- crate: árvore de módulos compilada como lib ou bin;
- module: organização interna da crate;
- crate root: normalmente `src/lib.rs` ou `src/main.rs`.

A recomendação aplicada ao `rustcraft`:

- manter packages/crates por fronteira real (`rc-voxel`, `rc-render`, `rc-world`, etc.);
- dentro de cada crate, usar módulos por domínio;
- deixar `lib.rs` como raiz/API pública, não como arquivo gigante de implementação.

### Privacidade e `pub use`

O Rust Book enfatiza que módulos são privados por padrão e que `pub use` permite expor uma API pública diferente da organização interna.

A recomendação aplicada:

```rust
mod block;
mod registry;

pub use block::{BlockId, BlockState};
pub use registry::{BlockDefinition, block_definition};
```

Isso permite que usuários chamem `rc_voxel::BlockState`, enquanto o código continua organizado internamente em `block.rs`.

### Workspaces

O Rust Book e Cargo Book indicam workspaces para múltiplos packages relacionados, com `Cargo.lock` e `target/` compartilhados. Também reforçam comandos por package, como `cargo test -p rc-voxel`.

A recomendação aplicada:

- continuar usando workspace multi-crate;
- validar crate por crate enquanto refatora;
- manter dependências explícitas em cada `Cargo.toml`; workspace não significa dependência automática.

### Cargo package layout

Cargo usa convenções como:

```text
src/lib.rs
src/main.rs
src/bin/
tests/
examples/
benches/
```

A recomendação aplicada:

- `crates/rustcraft/src/bin/rustcraft.rs` está correto para bin principal;
- cada library crate usa `src/lib.rs` como raiz;
- módulos internos ficam em arquivos `snake_case.rs` dentro de `src/`.

### Rust API Guidelines

Aplicável principalmente a nomes e documentação:

- documentação de crate com `//!`;
- documentação de itens públicos com `///`;
- exemplos quando a API amadurecer;
- nomes idiomáticos (`snake_case` para funções/módulos, `UpperCamelCase` para tipos, `SCREAMING_SNAKE_CASE` para constantes).

A recomendação aplicada:

- documentação curta e orientada a contrato, não comentário linha-a-linha;
- nomes como `BlockState`, `BlockDefinition`, `block_definition`, `block_for_layer`.

## Bevy

Bevy trata plugins como unidade principal de modularidade: um plugin é um conjunto escopado de components, resources e systems. A própria documentação recomenda mover lógica para plugins para organização melhor.

A recomendação aplicada:

- cada crate Bevy expõe um plugin (`InputPlugin`, `PlayerPlugin`, `RenderPlugin`, `WorldPlugin`);
- `plugin.rs` registra systems/resources/sets;
- `config.rs`, `components.rs`, `assets.rs`, `movement.rs`, etc. guardam os domínios;
- `rustcraft` compõe plugins, não implementa regra de jogo.

## Zed

Zed é um monorepo Rust grande com muitos crates sob `crates/`, tooling próprio, lint/workspace config e organização por features/domínios. A análise útil aqui é estrutural, não copiar nomes.

Lições para o `rustcraft`:

- monorepo Rust grande tende a separar responsabilidades em crates pequenas;
- tooling/lints/workspace config ficam na raiz;
- crates de feature/domínio tornam dependências explícitas;
- `publish = false`/políticas de workspace são úteis para projetos internos.

Cuidado: Zed é editor/UI colaborativo, não engine voxel. Usar como referência de workspace e escala de projeto Rust, não de gameplay.

## COSMIC/libcosmic

libcosmic expõe módulos públicos por domínio: `app`, `config`, `theme`, `widget`, `task`, `surface`, `dialog`, etc., além de reexports convenientes. Isso é parecido com o padrão `lib.rs` como API pública e módulos internos por domínio.

Lições para o `rustcraft`:

- módulos devem corresponder a conceitos que o usuário da crate entende;
- reexports/prelude podem facilitar consumo quando a API crescer;
- docs geradas por `cargo doc` funcionam melhor quando os módulos têm responsabilidade clara.

Cuidado: COSMIC é toolkit de desktop, não jogo. A lição é organização de crate/API, não ECS/render voxel.

## 40tude

40tude é útil como material didático sobre Rust, modular monolith, spec-driven development e separação entre domínio/API/UI em projetos pequenos.

Lições para o `rustcraft`:

- documentar decisões antes de codar reduz retrabalho;
- separar domínio puro de framework ajuda testes e entendimento;
- manter specs/tasks vivas no vault combina com nosso fluxo tutor-first.

Cuidado: 40tude é blog/tutorial, não referência oficial. Usar como apoio didático, não como autoridade arquitetural.

## Grandes engines/empresas

A pesquisa anterior sobre Unity/Unreal/Godot e Minecraft modding aponta para o mesmo princípio:

- objetos com identidade viram entidade/actor/node;
- dados massivos ficam em estruturas compactas;
- renderização usa agregação, culling, streaming, LOD/HLOD;
- registries e IDs estáveis evitam enum gigante e acoplamento forte.

Para `rustcraft`, isso reforça:

```text
rc-voxel  -> dados puros e compactos
rc-world  -> mundo/chunks/lifecycle
rc-render -> render/meshing/assets
rc-player -> player/camera/controller
rc-input  -> input físico -> ação semântica
rustcraft -> composição do app
```

## Conclusão para a organização das crates

A organização proposta em [[crate-module-organization-plan]] está alinhada com:

- Rust oficial: módulos privados + `pub use`, crate root como API, workspace para packages relacionados;
- Cargo: layout convencional e validação por package;
- Bevy: plugins pequenos, escopados por funcionalidade;
- referências grandes: fronteiras por domínio, dependências explícitas, dados puros separados de integração/framework.

A decisão prática permanece:

1. Não criar novas crates agora.
2. Organizar internamente as crates existentes por domínio.
3. Manter `lib.rs` pequeno.
4. Reexportar API pública de forma intencional.
5. Validar uma crate por vez.

## Fontes

- Rust Book — Packages and Crates: https://doc.rust-lang.org/stable/book/ch07-01-packages-and-crates.html
- Rust Book — Modules and Privacy: https://doc.rust-lang.org/book/ch07-02-defining-modules-to-control-scope-and-privacy.html
- Rust Book — `use` and `pub use`: https://doc.rust-lang.org/book/ch07-04-bringing-paths-into-scope-with-the-use-keyword.html
- Rust Book — Cargo Workspaces: https://doc.rust-lang.org/book/ch14-03-cargo-workspaces.html
- Cargo Book — Package Layout: https://doc.rust-lang.org/cargo/guide/project-layout.html
- Rust API Guidelines — Documentation: https://rust-lang.github.io/api-guidelines/documentation.html
- Rust API Guidelines — Naming: https://rust-lang.github.io/api-guidelines/naming.html
- Bevy Plugins docs: https://bevy.org/learn/quick-start/getting-started/plugins/
- Bevy Plugin example: https://bevy.org/examples/application/plugin/
- Zed repository: https://github.com/zed-industries/zed
- COSMIC/libcosmic docs: https://pop-os.github.io/libcosmic/cosmic/
- libcosmic repository: https://github.com/pop-os/libcosmic
- 40tude Rust index: https://www.40tude.fr/
- 40tude Spec-Driven Development with Rust: https://www.40tude.fr/docs/06_programmation/rust/027_speckit/speckit.html

## Links internos

- [[crate-module-organization-plan]]
- [[workspace-and-crate-organization]]
- [[voxel-entity-rendering-model]]
- [[minecraft-modloader-api-lessons]]
