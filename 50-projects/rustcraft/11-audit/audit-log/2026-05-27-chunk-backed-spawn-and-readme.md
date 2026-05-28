---
title: Audit log — Chunk-backed spawn and README update
type: audit-log
tags:
  - audit
  - rustcraft
  - voxel
  - chunk
  - readme
  - performance
created: '2026-05-27'
updated: '2026-05-27'
---
# Audit log — Chunk-backed spawn and README update

## Contexto

O spawn inicial passou a usar `Chunk` como fonte de dados, mas ainda instancia uma entidade renderizavel por bloco nao vazio. Foi documentado que o consumo de CPU ainda pode ser perceptivel por causa desse caminho temporario.

## Alteracoes documentadas

- README atualizado com:
  - estrutura real dos módulos internos das crates;
  - estado atual do modelo `BlockId`/`BlockState`/`Chunk`;
  - `WorldSeed`, `TerrainGenerator` e `generate_chunk`;
  - limitacao atual de uma entidade por bloco;
  - proxima etapa: mesh por chunk com faces expostas.

- Tasks atualizadas:
  - T-BEVY-002 marcado como `done`, porque `rc-world` agora produz `Chunk` e o spawn le dele.
  - T-BEVY-003 permanece aberta para resolver renderizacao/performance com mesh por chunk.

- STATE atualizado:
  - DT-011 aberta para registrar que o custo de CPU atual ainda e esperado enquanto houver entidade por bloco.

## Implicacao tecnica

A fonte de verdade do mundo inicial ja e `Chunk`, mas a camada visual ainda e prototipo. A proxima mudanca relevante de performance deve ocorrer em `rc-render`, criando mesh agregada por chunk e emitindo apenas faces expostas.

## Links

- [[../../05-tasks/bevy-example-study-tasks]]
- [[../../STATE]]
- [[../../02-architecture/voxel-entity-rendering-model]]
