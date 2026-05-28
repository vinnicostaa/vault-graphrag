---
title: Sprint capability map — rustcraft
type: schedule
tags:
  - rustcraft
  - sprints
  - bevy
  - planning
created: '2026-05-27'
updated: '2026-05-28'
source_date: '2026-05-27'
---
# Sprint capability map — rustcraft

Mapa inicial de sprints baseado nos exemplos oficiais do Bevy e nas limitacoes atuais do projeto.

## Principio de ordem

A prioridade tecnica e resolver a escalabilidade do mundo voxel antes de empilhar sistemas cosmeticos. Audio, UI, shaders, luz avancada e menus entram melhor quando chunk/render/player ja tiverem uma base minimamente correta.

## S0 — Base observavel

Objetivo: manter o prototipo atual validavel.

Entregas:

- logs configurados;
- FPS overlay ou log diagnostics;
- window title/settings basicos;
- contadores: entidades render, chunks, faces, tempo de meshing quando existir;
- gizmo simples opcional para eixos/chunk bounds.

Referencias Bevy:

- Logs;
- FPS overlay;
- Custom Diagnostic;
- Log Diagnostics;
- Window Settings;
- 3D Gizmos.

## S1 — Chunk data + mesh por chunk

Objetivo: substituir entidade por bloco por dados de chunk e mesh agregada.

Estado em 2026-05-28:

- `ChunkCoord`, `Chunk`, armazenamento indexado, geracao de terreno para chunk e `build_chunk_mesh` com faces expostas ja existem.
- O spawn principal ja usa uma entidade renderizavel por chunk.
- A limitacao remanescente e visual: material unico temporario ate atlas/vertex color ou caminho equivalente de material por bloco.

Entregas:

- `ChunkCoord`, `Chunk`, armazenamento indexado;
- geracao de terreno para chunk;
- `build_chunk_mesh` com faces expostas;
- uma entidade renderizavel por chunk;
- testes para indexacao e geracao de faces;
- remover caminho principal de spawn por bloco individual.

Referencias Bevy:

- Generate Custom Mesh;
- Many Cubes;
- Visibility Range;
- Automatic Instancing, somente como leitura de performance futura.

## S2 — Player, camera e interacao com bloco

Objetivo: transformar o player em controlador util para gameplay voxel.

Estado em 2026-05-28:

- Mouse look da camera/player ja existe em `rc-player` usando `AccumulatedMouseMotion`.
- Raycast a partir da camera/player, conversao para `BlockPos` e highlight debug do bloco mirado ja existem.
- Ainda faltam captura/trava de cursor, estado persistente de bloco mirado e quebrar/colocar bloco marcando chunk dirty.

Entregas:

- mouse look / free camera controlado;
- input semantico expandido;
- raycast da camera para selecionar bloco;
- highlight/gizmo no bloco mirado;
- quebrar/colocar bloco marcando chunk dirty.

Referencias Bevy:

- Free Camera controller;
- First person view model;
- 3D viewport to world;
- Mesh Ray Cast;
- Picking Debug Tools.

## S3 — Materiais, textura e luz base

Objetivo: sair de cubos coloridos para material voxel minimamente extensivel.

Entregas:

- material por tipo ou atlas inicial;
- UVs por face no chunk mesh;
- directional light configuravel;
- sombras basicas;
- ambient light controlado;
- decidir se textura entra via atlas 2D ou array texture.

Referencias Bevy:

- Texture;
- Repeated texture configuration;
- Array Texture;
- Lighting;
- Spotlight;
- Shadow Caster and Receiver.

## S4 — UI, menu e settings

Objetivo: configurar jogo sem recompilar e separar estado de menu/jogo.

Entregas:

- `GameState`: Loading/Menu/InGame/Paused;
- menu inicial simples;
- settings resources: render distance, volume, mouse sensitivity, fullscreen/resolution depois;
- debug overlay toggleavel;
- cleanup de UI por estado.

Referencias Bevy:

- Game Menu;
- Button;
- Text;
- Display and Visibility;
- CSS Grid/Flex Layout;
- UI Scaling;
- UI Target Camera.

## S5 — Audio, feedback e diagnosticos de jogo

Objetivo: adicionar feedback de jogo sem esconder problemas de performance.

Entregas:

- SFX de colocar/quebrar bloco;
- soundtrack por estado;
- mute/volume/pause via `AudioSink`;
- spatial audio 3D para eventos no mundo;
- custom diagnostics: chunks visiveis, dirty chunks, faces renderizadas.

Referencias Bevy:

- Audio;
- Audio Control;
- Soundtrack;
- Spatial Audio 3D;
- Custom Diagnostic;
- Log Diagnostics.

## S6 — Fisica e colisao

Objetivo: colocar o player em mundo com colisao real.

Entregas:

- decidir Rapier ou Avian;
- rodar fisica em `FixedUpdate`;
- collider por chunk/mesh simplificada;
- interpolacao visual;
- separar input acumulado, simulacao fixa e render.

Referencias Bevy:

- Run physics in a fixed timestep.

## S7 — Streaming, async e escala

Objetivo: mundo alem de um chunk sem travar frame.

Entregas:

- geracao/meshing async;
- fila de jobs;
- load/unload por render distance;
- cache de chunks;
- LOD/HLOD experimental para distancia;
- screenshot/profiling para comparar antes/depois.

Referencias Bevy:

- Async Channel Pattern;
- Visibility Range;
- Many Cubes;
- Screenshot;
- Scene Viewer, se glTF entrar.

## Backlog avancado

- Lightmaps e Mixed Lighting: usar para cenas estaticas/props, nao para voxel dinamico inicial.
- Easing Functions, Eased Motion, Color Animation: feedback de UI, hit, mining progress, dano, transicoes.
- Animated Transform/Animated Material: entidades especiais, props, efeitos.
- Extended Material/Shader Material: shader voxel custom quando textura/atlas simples mostrar limite real.
- Light Gizmos/3D Gizmos: ferramentas de debug sempre que mexer em luz/culling/raycast.

## Links internos

- [[../02-architecture/bevy-examples-capability-map]]
- [[../02-architecture/voxel-entity-rendering-model]]
- [[../ROADMAP]]
- [[../05-tasks/mvp-0-tasks]]
