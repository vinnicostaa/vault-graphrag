---
title: Práticas de codificação — rustcraft
type: quality-guide
tags:
  - rustcraft
  - quality
  - rust
  - bevy
  - coding-practices
created: '2026-05-28'
updated: '2026-05-28'
---
# Práticas de codificação — rustcraft

Esta nota espelha o guia do repositorio `CODING_PRACTICES.md` e registra o padrao esperado para desenvolvimento no projeto.

## Contrato didatico

- O usuario codifica quando a task for de implementacao.
- O agente atua como tutor, revisor e pesquisador tecnico.
- Mudanca de codigo pelo agente exige pedido explicito na sessao corrente.
- Toda explicacao de task deve apontar arquivos alvo, sintaxe Rust envolvida e criterios de validacao.

## Principios

- Separar dominio puro de runtime Bevy.
- Manter `rc-voxel` sem dependencia de Bevy.
- Usar `lib.rs` como API publica do crate e modulos internos por responsabilidade.
- Evitar entidade Bevy por bloco comum de terreno no modelo final.
- Documentar limitacoes temporarias quando elas afetam a proxima etapa.

## Bevy 0.18.1

Para mesh customizada, seguir o padrao validado nos exemplos oficiais/local source:

- `Mesh::new(PrimitiveTopology::TriangleList, RenderAssetUsages::MAIN_WORLD | RenderAssetUsages::RENDER_WORLD)`;
- atributos `POSITION`, `NORMAL`, `UV_0`;
- `Indices::U32`;
- spawn renderizavel com `Mesh3d(handle)` e `MeshMaterial3d(material)`.

## Gates

```sh
cargo fmt --all -- --check
cargo test --workspace
cargo clippy --workspace --all-targets --all-features -- -D warnings
```

## Artefatos no repositorio

- `AGENTS.md` — guia rapido para futuras sessoes de agentes.
- `CODING_PRACTICES.md` — praticas de codificacao e documentacao.
- `README.md` — estado atual e comandos.
- `ARCHITECTURE.md` — fronteiras e proximas etapas.

## Links

- [[../CLAUDE]]
- [[../STATE]]
- [[../05-tasks/bevy-example-study-tasks]]
- [[../10-schedule/sprint-capability-map]]
