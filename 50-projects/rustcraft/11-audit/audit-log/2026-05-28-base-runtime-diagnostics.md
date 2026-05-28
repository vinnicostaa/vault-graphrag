---
title: Audit log — Base runtime diagnostics
type: audit-log
tags:
  - rustcraft
  - audit-log
  - bevy
  - diagnostics
created: '2026-05-28'
updated: '2026-05-28'
---
# Audit log — Base runtime diagnostics

Resumo:

- Criado `RustcraftDiagnosticsPlugin` em `crates/rustcraft/src/diagnostics.rs`.
- O plugin adiciona diagnosticos oficiais do Bevy: `LogDiagnosticsPlugin`, `FrameTimeDiagnosticsPlugin`, `EntityCountDiagnosticsPlugin` e `SystemInformationDiagnosticsPlugin`.
- `RustcraftPlugin` passou a instalar os diagnosticos antes dos plugins de dominio do jogo, mantendo `DefaultPlugins` como base em `run()`.
- `cargo run` confirmou logs de FPS, frame time, frame count, entity count, CPU e memoria.
- O valor observado de `entity_count` foi 20, coerente com a troca recente de spawn por bloco para spawn por chunk.

Validacao informada pelo usuario:

- `cargo fmt --all -- --check`
- `cargo check --workspace`
- `cargo test --workspace`
- `cargo clippy --workspace --all-targets --all-features -- -D warnings`
- `cargo run`

Validacao adicional local:

- `cargo fmt --all -- --check`
- `cargo check --workspace`
- `git diff --check`

Pendencias:

- Criar diagnosticos proprios para chunks renderizados/carregados e faces/vertices.
- Decidir se a visualizacao continua por log ou se entra overlay UI depois.

Links:

- [[../../05-tasks/bevy-example-study-tasks]]
- [[../../10-schedule/sprint-capability-map]]
- [[../../STATE]]
