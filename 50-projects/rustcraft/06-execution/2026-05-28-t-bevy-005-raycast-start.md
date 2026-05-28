---
title: Execução — T-BEVY-005 raycast inicial
type: note
tags:
  - rustcraft
  - execution
  - bevy
  - raycast
created: '2026-05-28'
updated: '2026-05-28'
---
# Execução — T-BEVY-005 raycast inicial

## Escopo desta rodada

Implementar apenas o primeiro passo do raycast: sair da camera/player, bater na mesh do chunk e registrar ponto/normal/entidade atingidos.

A conversao completa para `BlockPos` fica para a rodada seguinte, porque deve ser testada separadamente e nao deve depender de entidade por bloco.

## Fontes verificadas

- Projeto usa `bevy = "0.18.1"`.
- Exemplo oficial local: `bevy-0.18.1/examples/3d/mesh_ray_cast.rs`.
- API oficial local: `bevy_picking-0.18.1/src/mesh_picking/ray_cast/mod.rs`.
- Camera API local: `bevy_camera-0.18.1/src/camera.rs`.

## API confirmada

- `MeshRayCast` e usado como `SystemParam`.
- `MeshRayCastSettings::default()` usa visibilidade `VisibleInView` e early exit no hit mais proximo.
- `RayMeshHit.point` esta em world space.
- `RayMeshHit.normal` representa a normal do triangulo atingido.
- `Camera::viewport_to_world(&GlobalTransform, Vec2)` cria `Ray3d` a partir de coordenadas de tela.

## Decisao local

Para o `rustcraft`, o primeiro ray deve sair do centro da camera ou da direcao `Transform::forward()` do player. Como ainda nao temos mouse look nem cursor capturado, usar a direcao da camera/player e mais simples e mais parecido com mira central de jogo voxel.

## Proxima fatia — T-BEVY-005B

Converter `RayMeshHit.point` + `RayMeshHit.normal` para `BlockPos`.

Regra escolhida:

- o ponto do hit esta exatamente na face do bloco;
- para descobrir o bloco atingido, deslocar o ponto um pouco para dentro da face usando `point - normal * epsilon`;
- converter coordenada de mundo para coordenada de bloco com `floor(world / block_size)`;
- implementar essa conta em `rc-voxel`, com arrays `[f32; 3]`, para manter a crate sem dependencia de Bevy.
