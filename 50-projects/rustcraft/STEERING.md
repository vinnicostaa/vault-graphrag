---
title: STEERING — rustcraft
type: note
tags:
  - steering
  - rustcraft
  - governance
  - tutor-first
created: '2026-05-27'
updated: '2026-05-27'
---
# STEERING — rustcraft

Nao-negociaveis operacionais e arquiteturais para agentes trabalhando no `rustcraft`.

## Operacao no vault

- Ler `[[../../CLAUDE]]` antes de qualquer decisao relevante.
- Ler `[[CLAUDE]]`, `[[STATE]]`, `[[11-audit/AUDIT]]`, `[[SESSION_CHARTER]]` e `[[05-tasks/mvp-0-tasks]]` no inicio de sessoes do projeto.
- Toda mutacao no vault passa pelo MCP Obsidian.
- Decisao arquitetural vira entrada D-### em `[[STATE]]` e, se estrutural, ADR em `[[02-architecture/adr/_moc]]`.
- Finding de revisao vira A-### em `[[11-audit/AUDIT]]`.

## Modo tutor-first

- O usuario codifica de fato.
- O agente explica conceitos, cria planos, revisa, pesquisa fontes oficiais e decompoe tasks.
- O agente so altera codigo quando houver pedido explicito na sessao corrente.
- Refactor grande precisa de plano, criterios de verificacao e registro no vault.

## Linguagem de dominio

- Usar vocabulario de jogo: `actions`, `controls`, `player`, `world`, `voxel`, `rendering`, `ui`, `tools`.
- Nao usar `protocol` para input local.
- Reservar `protocol` para multiplayer, wire format, save format ou comunicacao externa real.

## Arquitetura tecnica

- `rustcraft` e estudo de jogo sandbox/voxel 3D em Rust/Bevy.
- Bevy pode ser runtime atual, mas conceitos devem ser explicados de forma transferivel.
- Separar input fisico de acao semantica.
- Separar dados voxel puros de renderizacao Bevy.
- Evitar otimizacao prematura, mas registrar desde cedo o caminho para chunks, mesh por chunk, culling e profiling.

## Gates minimos

```sh
cargo check --workspace
cargo test --workspace
cargo fmt --all -- --check
cargo clippy --workspace --all-targets --all-features -- -D warnings
```

## Links

- [[CLAUDE]]
- [[STATE]]
- [[11-audit/AUDIT]]
- [[02-architecture/workspace-and-crate-organization]]
- [[../../40-templates/project-scaffold/_moc]]
