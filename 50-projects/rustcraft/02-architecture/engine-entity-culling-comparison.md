---
title: 'Engine comparison — entities, chunks and culling'
type: architecture-note
tags:
  - rustcraft
  - engines
  - ecs
  - culling
  - voxel
  - architecture
created: '2026-05-27'
updated: '2026-05-27'
source_date: '2026-05-27'
---
# Engine comparison — entities, chunks and culling

Resposta direta para a duvida de arquitetura: o que deve ser entidade, como renderizar so o que precisa, e como engines maiores tratam isso.

## Regra principal para o rustcraft

Entidade e para coisa com identidade de runtime. Voxel comum e dado.

No `rustcraft`, pedra, terra, grama e ar nao devem ser entidades Bevy uma a uma. Eles devem morar em `ChunkStorage`. A entidade Bevy deve representar o chunk renderizavel, o player, camera, luzes, UI, audio emitters, itens soltos, mobs e blocos especiais ativos.

## Tabela pratica

| Caso | Entidade? | Motivo |
| --- | --- | --- |
| Bloco comum parado | Nao | Dado massivo repetitivo; fica em chunk. |
| Ar | Nao | Ausencia de bloco; nao deve ocupar ECS. |
| Chunk visivel | Sim | Tem transform, mesh, material, visibilidade e collider futuro. |
| Bau/porta/maquina ativa | Sim, ou entidade auxiliar | Tem estado/comportamento individual. |
| Tocha estatica simples | Depende | Como bloco comum se for so textura/luz baked; entidade se tiver PointLight/flicker/audio. |
| Item dropado | Sim | Move, colide, despawna, pode ser coletado. |
| Player/camera | Sim | Input, transform, camera, controle. |
| Face exposta | Nao | E resultado de meshing, nao objeto de jogo. |

## Renderizar somente o visivel

Existem camadas diferentes:

1. Nao gerar geometria invisivel: faces internas entre blocos solidos nao entram na mesh.
2. Frustum culling: o renderer descarta entidades fora da camera. Funciona melhor com entidade por chunk do que por bloco.
3. Render distance/streaming: `rc-world` decide quais chunks existem perto do player.
4. Occlusion culling: esconder coisas atras de paredes/terreno. Isso e mais caro e normalmente vem depois.
5. LOD/HLOD: longe da camera, trocar muitos objetos por representacao agregada.

Para voxel dinamico, a ordem correta e: face culling -> chunk mesh -> frustum culling -> render distance -> async meshing -> LOD/HLOD/occlusion.

## Como Unity faz

Unity tradicional usa GameObjects como contêineres de Components; todo GameObject tem Transform. Isso e bom para objetos de jogo, mas ruim para milhoes de celulas voxel.

Unity DOTS/Entities usa archetypes e chunks: entidades com os mesmos componentes sao agrupadas em chunks de memoria. A licao para o `rustcraft` e data-oriented: se existe muito dado parecido, organizar em bloco contiguo de memoria, nao em objetos pequenos espalhados.

Unity tambem tem frustum culling por camera e occlusion culling baked. A propria documentacao observa que occlusion baked nao e adequada para geometria gerada em runtime. Para voxel dinamico, isso empurra a solucao para chunk streaming/meshing proprio.

## Como Unreal faz

Unreal separa Actor e Component. Actor e objeto de mundo; Components adicionam render, colisao, audio, camera e comportamento. Primitive Components criam estado de render/physics. Registrar componentes durante o jogo tem custo, entao nao faz sentido criar/remover milhares de componentes por bloco a todo momento.

Para mundo grande, Unreal usa World Partition e HLOD: o mundo e dividido em celulas, e objetos distantes podem virar HLOD Actors agregados. A licao e a mesma: engine grande agrupa espacialmente e cria proxies renderizaveis, em vez de tratar cada detalhe como unidade independente cara.

## Como Godot faz

Godot usa Scene/Node. Node e otimo para objetos de jogo, mas uma arvore com node por voxel seria pesada. Para 3D, Godot usa MeshInstance3D/MultiMeshInstance3D e culling por bounds. O occlusion culling oficial usa occluders e testa AABBs dos objetos escondidos; nao e uma licenca para modelar cada voxel como node.

A licao para o `rustcraft`: scenes/nodes/actors/gameobjects sao nivel de composicao; voxel terrain e dado/mesh agregada.

## Como empresas grandes normalmente pensam

O padrao e separar:

- dado de mundo em estruturas espaciais: chunks, cells, sectors, regions;
- objetos de gameplay com identidade: actors/entities/gameobjects;
- render em batches, instancing, merged meshes ou proxies;
- streaming por distancia/interesse;
- culling por frustum, distancia, oclusao e LOD;
- fisica agregada quando possivel.

Em jogo voxel, isso vira:

```text
World data: ChunkMap<ChunkCoord, Chunk>
Chunk data: Vec<BlockState>
Render: 1 Mesh por chunk visivel
Physics: 1 collider simplificado por chunk
Gameplay: entidades so para player, mobs, itens e blocos ativos
```

## Decisao aplicada ao projeto

- `rc-voxel`: `BlockState`, `Chunk`, indexacao, dados puros.
- `rc-world`: geracao, armazenamento, dirty chunks, streaming.
- `rc-render`: meshing, materiais, textura/atlas, spawn/update de chunk entity.
- `rc-player`: camera/player/raycast.
- `rc-input`: input fisico -> acoes semanticas.

## Fontes

- Bevy Generate Custom Mesh: https://bevy.org/examples/3d-rendering/generate-custom-mesh/
- Bevy Many Cubes: https://bevy.org/examples/stress-tests/many-cubes/
- Bevy Visibility Range: https://bevy.org/examples/3d-rendering/visibility-range/
- Unity GameObject: https://docs.unity3d.com/cn/2018.3/Manual/class-GameObject.html
- Unity Entities archetypes: https://docs.unity.cn/Packages/com.unity.entities%401.0/manual/concepts-archetypes.html
- Unity Occlusion Culling: https://docs.unity3d.com/es/2019.4/Manual/OcclusionCulling.html
- Unreal Components: https://dev.epicgames.com/documentation/en-us/unreal-engine/components-in-unreal-engine
- Unreal World Partition HLOD: https://dev.epicgames.com/documentation/en-us/unreal-engine/world-partition---hierarchical-level-of-detail-in-unreal-engine
- Godot Nodes and Scenes: https://docs.godotengine.org/en/4.6/getting_started/step_by_step/nodes_and_scenes.html
- Godot Occlusion Culling: https://docs.godotengine.org/en/4.4/tutorials/3d/occlusion_culling.html

## Links internos

- [[voxel-entity-rendering-model]]
- [[bevy-examples-capability-map]]
- [[../10-schedule/sprint-capability-map]]
