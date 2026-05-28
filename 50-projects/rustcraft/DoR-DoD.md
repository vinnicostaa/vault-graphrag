---
title: DoR-DoD — rustcraft
type: note
tags:
  - dor
  - dod
  - rustcraft
  - quality
created: '2026-05-27'
updated: '2026-05-27'
---
# DoR-DoD — rustcraft

Definition of Ready e Definition of Done para tasks do `rustcraft`.

## Definition of Ready

Uma task esta pronta para comecar quando:

- objetivo observavel esta escrito;
- arquivos ou areas provaveis foram indicados;
- conceito de aprendizado esta claro;
- criterio de validacao esta definido;
- impacto no vault foi identificado;
- se for decisao arquitetural, ha D-### ou ADR planejado.

## Definition of Done

Uma task esta concluida quando:

- comportamento esperado foi implementado ou documentado;
- `cargo check --workspace` passa;
- `cargo test --workspace` passa quando houver logica testavel;
- `cargo fmt --all -- --check` passa;
- `cargo clippy --workspace --all-targets --all-features -- -D warnings` passa;
- README/ARCHITECTURE/vault foram atualizados se a estrutura mudou;
- `[[STATE]]` registra decisoes/dividas novas;
- `[[11-audit/AUDIT]]` registra findings descobertos ou fechados.

## Modo tutor-first

Para tasks de aprendizado, Done tambem exige que o usuario consiga explicar o que mudou e por que.

## Links

- [[STEERING]]
- [[05-tasks/mvp-0-tasks]]
- [[11-audit/AUDIT]]
- [[../../40-templates/project-scaffold/_moc]]
