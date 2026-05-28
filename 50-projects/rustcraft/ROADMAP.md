---
title: ROADMAP — rustcraft
type: roadmap
tags:
  - roadmap
  - rustcraft
created: '2026-05-27'
updated: '2026-05-28'
---
# ROADMAP — rustcraft

## Fase 0 — Base didática e estrutura correta 🔨

Objetivo: preservar o protótipo original e reorganizar o projeto de modo idiomático para Cargo + Bevy.

Entregas:

- [x] Entender workspace/package/lib/bin/module.
- [x] Restaurar o protótipo jogável original em estrutura limpa.
- [x] Remover o nome `protocol` do fluxo de input local.
- [x] Separar input bruto de ações de jogo.
- [x] Separar blocos/mundo/render/player em crates internas por domínio.
- [x] Documentar estrutura no README.
- [x] Validar com `cargo check`, `test`, `fmt`, `clippy`.

Status: concluída no código e nas tasks MVP-0. Última validação registrada: 33 testes passando no workspace.

## Fase 1 — Voxel básico

Objetivo: sair de “uma entidade por bloco” para dados de chunk + mesh simples. Ver [[02-architecture/voxel-entity-rendering-model]] e [[10-schedule/sprint-capability-map]].

Entregas:

- [x] Modelo de bloco migrado para `BlockId`, `BlockState`, `BlockDefinition` e registry mínimo.
- [x] `ChunkCoord` e `Chunk` em memória.
- [x] Geração de terreno para um chunk.
- [x] Mesh por chunk com faces expostas.
- [x] Uma entidade renderizável por chunk.

Status: concluída. Limitação remanescente: material único temporário até atlas, vertex color ou abordagem equivalente por face/bloco.

## Fase 2 — Gameplay mínimo

Objetivo: começar interação jogador ↔ mundo.

Entregas:

- [x] Movimento de câmera/player melhor estruturado, incluindo mouse look.
- [x] Raycast ou seleção de bloco.
- [ ] Quebrar/colocar bloco.
- [ ] Inventário mínimo.

Status: parcialmente concluída. Mouse look e raycast já permitem mirar blocos; ainda falta capturar/travar o cursor, transformar o bloco mirado em estado de gameplay e implementar quebrar/colocar bloco.

## Fase 3 — Sistemas técnicos

Objetivo: começar a aproximar o projeto de um jogo real. O mapa de capacidades por exemplos Bevy esta em [[02-architecture/bevy-examples-capability-map]].

Entregas:

- [ ] Física/colisão com Rapier ou alternativa.
- [ ] Textura/atlas.
- [ ] Debug overlay.
- [ ] Config de render distance e geração.
- [ ] Profiling básico.

## Fase 4 — Escala e ferramentas

- [ ] Geração assíncrona.
- [ ] Streaming de chunks.
- [ ] Ferramentas/editor/debug panels.
- [ ] Save/load.
- [ ] Estrutura para multiplayer ou modding, se ainda fizer sentido.
