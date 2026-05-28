---
title: MOC — rustcraft 09-testing
type: moc
tags:
  - moc
  - rustcraft
  - testing
created: '2026-05-27'
updated: '2026-05-27'
---
# MOC — 09-testing

Estrategia de testes do `rustcraft`.

## Estado atual

Testes unitarios existem para:

- mapeamento de camada para `BlockType`;
- faixa esperada de `terrain_height`.

## Proxima direcao

- Testar `rc-voxel`/dados voxel antes de render.
- Testar mapeamento input -> actions sem abrir janela.
- Separar testes de dominio puros de testes Bevy/runtime.

## Gates

Ver `[[../STEERING#Gates minimos]]`.

## Links

- [[../07-quality/_moc]]
- [[../../10-knowledge/testing/_moc]]
- [[../05-tasks/mvp-0-tasks]]
