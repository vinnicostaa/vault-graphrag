---
title: Audit log — player mouse look
type: audit-log
tags:
  - audit-log
  - rustcraft
  - player
  - camera
  - bevy
created: '2026-05-28'
updated: '2026-05-28'
---
# Audit log — player mouse look

## Contexto

T-BEVY-009 tinha como objetivo tornar o raycast de bloco usável com controle de câmera pelo mouse, sem voltar para entidade por bloco e sem gerar log por frame.

## Resultado

- `rc-player` ganhou o módulo `look.rs`.
- `PlayerConfig` ganhou `mouse_sensitivity`.
- `look_player` consome `AccumulatedMouseMotion` do Bevy.
- A rotação usa `EulerRot::YXZ`: yaw no eixo Y, pitch no eixo X e roll preservado.
- O pitch é limitado para evitar virar a câmera além da vertical.
- `PlayerPlugin` registra `(look_player, move_player).chain()`, aplicando rotação antes do movimento no frame.

## Validação

Comandos executados na raiz do workspace:

```sh
cargo fmt --all -- --check
cargo check --workspace
cargo test --workspace
cargo clippy --workspace --all-targets --all-features -- -D warnings
```

Resultado: todos passaram.

## Limitação remanescente

O mouse look ainda não captura/trava o cursor na janela. Essa responsabilidade deve ser tratada separadamente como controle de janela/input, usando a API de cursor do Bevy.

## Links

- [[../../05-tasks/bevy-example-study-tasks#T-BEVY-009 — Mouse look / câmera de primeira pessoa]]
- [[../../10-schedule/sprint-capability-map#S2 — Player, camera e interacao com bloco]]
- [[../../ROADMAP#Fase 2 — Gameplay mínimo]]
