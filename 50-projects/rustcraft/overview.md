---
title: Overview — rustcraft
type: project
tags:
  - project
  - rustcraft
  - rust
  - bevy
  - voxel
created: '2026-05-27'
updated: '2026-05-27'
---
# Overview — rustcraft

`rustcraft` e um projeto de estudo em Rust/Bevy para aprender fundamentos de jogos 3D voxel: ECS, input, camera/player, mundo, renderizacao, dados voxel, fisica, UI/debug, performance e, futuramente, multiplayer/modding.

## Intencao

A referencia inicial e Minecraft como base mental de sandbox/voxel, mas o objetivo nao e recriar Minecraft 1:1. A direcao desejada e aprender a construir uma base tecnica que possa evoluir para sistemas mais abertos, com inspiracoes de sobrevivencia, construcao, automacao e simulacao.

## Foco imediato

MVP-0:

- movimento e camera;
- geracao simples de terreno;
- UI/debug minima;
- organizacao correta de Cargo workspace;
- separacao didatica de responsabilidades.

## Modo de trabalho

Tutor-first:

- o usuario codifica;
- o agente explica, pesquisa, revisa, planeja e registra;
- alteracoes grandes exigem decisao documentada.

## Riscos atuais

- Politica final para `AUDIT.md` raiz vs `11-audit/AUDIT.md` ainda nao decidida.
- Scaffold do vault ainda precisa de conteudo substantivo alem dos MOCs iniciais.
- Possibilidade de overengineering se novas crates forem extraidas antes das fronteiras estarem claras.

## Links

- [[_moc]]
- [[STATE]]
- [[11-audit/AUDIT]]
- [[ROADMAP]]
- [[04-requirements/mvp-0-learning-prototype]]
