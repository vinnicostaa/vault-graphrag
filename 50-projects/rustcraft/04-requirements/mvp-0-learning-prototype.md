---
title: MVP-0 — Protótipo didático inicial
type: requirement
tags:
  - requirements
  - rustcraft
  - mvp0
created: '2026-05-27'
updated: '2026-05-27'
---
# MVP-0 — Protótipo didático inicial

## Objetivo

Reorganizar o protótipo original sem mudar o comportamento jogável: terreno simples de cubos, câmera/player e movimento básico.

## Requisitos funcionais

### R-001 — Executar o jogo

O usuário deve conseguir rodar o jogo com:

```sh
cargo run
```

Critérios:

- abre uma janela Bevy;
- renderiza o terreno inicial;
- cria uma câmera/player.

### R-002 — Movimento básico

O jogador deve conseguir mover a câmera com:

- `W`/`S`: frente/trás;
- `A`/`D`: esquerda/direita;
- `Space`: subir;
- `ShiftLeft`: descer.

Critérios:

- movimento independente de framerate usando delta time;
- input bruto não deve ficar misturado com lógica de mundo.

### R-003 — Terreno simples

O mundo deve gerar uma grade de blocos com altura procedural simples.

Critérios:

- superfície com grama;
- camadas próximas com terra;
- camadas profundas com pedra;
- comportamento equivalente ao protótipo original.

### R-004 — Estrutura Cargo correta

O projeto deve ensinar corretamente:

- workspace;
- package/crate;
- lib target;
- bin target;
- módulos internos.

Critérios:

- README documenta a árvore;
- não há crate prematura por cada domínio;
- se houver múltiplos binários, eles ficam em `src/bin`.

## Requisitos não funcionais

### RNF-001 — Didático antes de otimizado

A estrutura deve favorecer aprendizado. Performance avançada entra depois com chunks/meshing.

### RNF-002 — Nomes alinhados com jogos

Usar vocabulário de jogos/ECS:

- `actions`;
- `controls`;
- `player`;
- `world`;
- `voxel`;
- `rendering`.

Evitar `protocol` para input local.

### RNF-003 — Validação mínima

Antes de marcar MVP-0 como concluído:

```sh
cargo check --workspace
cargo test --workspace
cargo fmt --all -- --check
cargo clippy --workspace --all-targets --all-features -- -D warnings
```
