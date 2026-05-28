---
title: Revisão dos contextos Zed — rustcraft
type: audit
tags:
  - review
  - rustcraft
  - context
  - cargo
  - bevy
created: '2026-05-27'
updated: '2026-05-27'
---
# Revisão dos contextos Zed — rustcraft

## Escopo

Revisão cruzada de:

- `context-1.md`;
- `context-2.md`;
- código local em `/home/vinni/Documentos/Projetos/Privados/rustcraft`;
- notas do vault em `50-projects/rustcraft`;
- documentação primária consultada na sessão.

Objetivo: tratar os inputs do usuário nos contextos como instruções válidas para a continuidade do projeto, removendo respostas rasas e registrando decisões verificáveis.

Atualização posterior em 2026-05-27: a divergência registrada aqui como F-001 foi resolvida pela reestruturação multi-crate documentada em `[[../11-audit/audit-log/2026-05-27-multi-crate-restructure]]`.

## Inputs do usuário que governam a direção

1. O projeto é um estudo de jogo voxel 3D inspirado em Minecraft, mas não uma cópia 1:1.
2. A ambição é aprender base de jogos 3D/voxel e depois explorar sistemas mais abertos, com referências como ICARUS e Satisfactory.
3. O foco imediato é básico: movimento, câmera, geração, interface; depois itens, lógica, física, textura, áudio e multiplayer.
4. A análise precisa separar fato verificado de inferência.
5. Zed e COSMIC são referências para entender organização e padrões, não para copiar nomes de outro domínio.
6. `protocol` não é bom nome para input local de jogo; preferir `actions`, `controls`, `input` e `player_controller`.
7. O vault deve ser usado como fonte de verdade para decisões, estado, auditoria, requisitos e tasks.
8. O modo desejado é tutor-first: o agente ajuda a planejar, revisar, pesquisar e decompor; mudanças grandes de código precisam ser explícitas e registradas.

## Dupla checagem feita

### Código local

Comandos executados:

```sh
cargo check --workspace
cargo test --workspace
cargo fmt --all -- --check
cargo clippy --workspace --all-targets --all-features -- -D warnings
```

Resultado depois das correções pequenas:

- `cargo check`: passou;
- `cargo test`: passou, 5 testes unitários;
- `cargo fmt --all -- --check`: passou;
- `cargo clippy ... -D warnings`: passou.

### Correções aplicadas

- `crates/rustcraft/src/protocol.rs` foi renomeado para `crates/rustcraft/src/actions.rs`.
- Imports em `input.rs` e `player.rs` agora usam `crate::actions`.
- `lib.rs`, `README.md` e `ARCHITECTURE.md` foram atualizados para remover `protocol` do fluxo local.
- `GameConfig` agora deriva `Default`, resolvendo `clippy::derivable_impls`.
- `Cargo.toml` agora usa `members = ["crates/*"]`, mantendo `default-members = ["crates/rustcraft"]`.

## Findings principais

### F-001 — Repositório ainda não implementava ADR-0003

Status: resolvido em 2026-05-27.

O vault aceitava uma direção multi-crate didática com `crates/rc-input`, `crates/rc-player`, `crates/rc-voxel`, `crates/rc-render` e `crates/rc-world`, mas o código local ainda tinha apenas uma package real: `crates/rustcraft`.

A divergência foi corrigida: o repositório agora implementa as crates `rc-*` e `rustcraft` ficou como package de composição.

### F-002 — `protocol` estava errado para o domínio atual

O nome vinha de analogia com COSMIC/Wayland/compositor, mas o contexto de `rustcraft` é jogo local. Para input semântico, `actions` é mais claro.

Resolvido no código local e registrado em `AUDIT.md`/`STATE.md`.

### F-003 — Workspace atual é válido, mas incompleto para o objetivo pedagógico

Pelo Cargo Book, workspace é uma coleção de packages gerenciadas em conjunto. O repositório já é um workspace válido, mas ainda não ensina plenamente dependências entre packages internas porque só há um membro real.

### F-004 — Separação Bevy por plugins e SystemSet está no caminho certo

A organização atual por `RustcraftPlugin`, plugins de domínio e sets explícitos (`StartupSet`, `GameSet`) é coerente com o modelo de Bevy 0.18.1: plugins configuram `App`, e `SystemSet` serve para agrupar/ordenar sistemas.

### F-005 — A referência 40tude deve influenciar o método de trabalho, não a arquitetura final diretamente

O valor principal dos artigos 40tude para este projeto é o estilo incremental: começar simples, modularizar, testar, adicionar logging/argumentos/métricas e só depois otimizar. Isso combina com MVP-0 e com a ideia de tasks pequenas.

## Próxima task recomendada

T-009 foi executada em 2026-05-27: ADR-0003 continuou aceita e o repositório foi reorganizado em `crates/rustcraft`, `crates/rc-input`, `crates/rc-player`, `crates/rc-voxel`, `crates/rc-render` e `crates/rc-world`.

Próxima task técnica recomendada agora: validar visualmente o protótipo em janela Bevy e confirmar que terreno, câmera e movimento continuam equivalentes após a reorganização.

## Fontes consultadas

- Rust Book, Chapter 7 — packages, crates and modules: https://doc.rust-lang.org/book/ch07-00-managing-growing-projects-with-packages-crates-and-modules.html
- Cargo Book — workspaces: https://doc.rust-lang.org/cargo/reference/workspaces.html
- Bevy 0.18.1 docs — `Plugin`: https://docs.rs/bevy/latest/bevy/app/trait.Plugin.html
- Bevy 0.18.1 docs — `SystemSet`: https://docs.rs/bevy/latest/bevy/prelude/trait.SystemSet.html
- Bevy 0.18.1 docs — `ButtonInput<KeyCode>`: https://docs.rs/bevy/latest/bevy/input/struct.ButtonInput.html
- Zed `crates/` tree: https://github.com/zed-industries/zed/tree/main/crates
- 40tude Game of Life with Winit/Pixels: https://www.40tude.fr/docs/06_programmation/rust/017_game_of_life/game_of_life_00.html
