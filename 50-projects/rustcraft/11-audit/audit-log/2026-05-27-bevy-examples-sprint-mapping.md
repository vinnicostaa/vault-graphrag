---
title: Audit log — Bevy examples sprint mapping
type: audit-log
tags:
  - audit
  - rustcraft
  - bevy
  - architecture
created: '2026-05-27'
updated: '2026-05-27'
---
# Audit log — Bevy examples sprint mapping

## Contexto

Pedido: estudar os exemplos oficiais do Bevy e referencias de engines maiores para mapear funcionalidades futuras do `rustcraft`: audio, ECS, input, render, shapes, animation, color, texture, lighting, lightmap, spotlight, easing, diagnostics, UI, camera, entities, player, physics, shaders, transforms, window/settings e escalabilidade de blocos.

## Alteracoes no vault

Criado:

- [[../../02-architecture/bevy-examples-capability-map]]
- [[../../02-architecture/voxel-entity-rendering-model]]
- [[../../10-schedule/sprint-capability-map]]
- [[../../05-tasks/bevy-example-study-tasks]]

Atualizado:

- [[../../02-architecture/_moc]]
- [[../../10-schedule/_moc]]
- [[../../05-tasks/_moc]]
- [[../../ROADMAP]]
- [[../../STATE]]

## Decisoes registradas

- D-012: bloco comum de terreno nao e entidade Bevy; chunk e a unidade de dados/render/collider.
- D-013: exemplos Bevy passam a guiar o mapa de capacidades e sprints.

## Fontes externas principais

- Bevy examples: https://bevy.org/examples/
- Unity ECS: https://unity.com/ecs
- Unreal Gameplay Framework: https://dev.epicgames.com/documentation/en-us/unreal-engine/gameplay-framework-in-unreal-engine
- Unreal World Partition HLOD: https://dev.epicgames.com/documentation/en-us/unreal-engine/world-partition---hierarchical-level-of-detail-in-unreal-engine
- Godot nodes/scenes: https://docs.godotengine.org/en/4.6/getting_started/step_by_step/nodes_and_scenes.html
- Godot occlusion culling: https://docs.godotengine.org/en/4.4/tutorials/3d/occlusion_culling.html
