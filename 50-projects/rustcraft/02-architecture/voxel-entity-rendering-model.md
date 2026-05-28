---
title: Voxel entity and rendering model — rustcraft
type: architecture-note
tags:
  - rustcraft
  - voxel
  - ecs
  - rendering
  - chunks
  - architecture
created: '2026-05-27'
updated: '2026-05-27'
source_date: '2026-05-27'
---
# Voxel entity and rendering model — rustcraft

Nota de arquitetura para responder a duvida: o que deve ser entidade, o que deve ser dado puro, e como renderizar somente o que importa.

## Decisao principal

Bloco comum de terreno nao deve ser entidade Bevy.

No `rustcraft`, um bloco estatico deve ser dado dentro de um `Chunk`. A entidade Bevy deve aparecer no nivel em que existe identidade, ciclo de vida ou comportamento de runtime relevante.

## Entidade Bevy quando

Usar entidade para coisas que precisam participar do ECS como objetos individuais:

- player;
- camera;
- luzes;
- audio emitters;
- UI nodes;
- chunks renderizaveis;
- mobs/NPCs;
- itens soltos;
- projeteis;
- particulas/efeitos com ciclo de vida;
- blocos especiais ativos, como porta, bau, maquina, fornalha, fluido ativo ou bloco com tick proprio.

Para bloco especial, a entidade deve referenciar `BlockPos`/`ChunkCoord`, mas a existencia logica do bloco continua registrada no chunk.

## Dado puro quando

Nao usar entidade para:

- bloco de terra/pedra/grama parado;
- ar;
- face exposta;
- luz por voxel, metadata simples ou flags;
- dado de geracao procedural;
- celula de chunk;
- resultado intermediario de meshing.

Esses itens devem ficar em arrays compactos, structs puras e testes unitarios.

## Modelo recomendado

```text
rc-voxel
  BlockType / BlockId / BlockState
  BlockPos / ChunkCoord
  ChunkStorage

rc-world
  ChunkMap
  geracao de chunk
  fila de chunks dirty
  streaming/load/unload por distancia do player

rc-render
  materiais/texturas
  chunk meshing
  spawn/update de entidade renderizavel por chunk

Bevy ECS
  1 entidade por chunk visivel/renderizavel
  componentes: ChunkCoord, Mesh3d, MeshMaterial3d, Transform, Visibility, collider futuro
```

## Por que uma entidade por bloco nao escala

Uma entidade por bloco aumenta custo de ECS, spawn/despawn, queries, memoria, transform e render bookkeeping. Mesmo quando o renderer consegue instanciar meshes iguais, continuam existindo problemas importantes:

- faces internas continuam existindo se cada cubo for renderizado inteiro;
- colisao por bloco individual explode o numero de colliders;
- atualizar milhares de entidades por chunk dificulta streaming;
- draw/queue/culling overhead cresce antes de o mundo ficar grande.

A saida correta para voxel e gerar mesh por chunk com apenas faces expostas.

## Renderizar somente objetos visiveis

Existem tres camadas diferentes:

1. Culling do renderer: Bevy consegue descartar entidades fora do frustum quando elas tem bounds corretos. Isso funciona melhor quando ha uma entidade render por chunk, nao uma por voxel.
2. Visibilidade por distancia/LOD: `VisibilityRange` serve para trocar ou esconder representacoes por distancia, parecido com HLOD. Para voxel inicial, usar render distance e streaming de chunks primeiro.
3. Streaming do mundo: chunks longe do player devem ser descarregados, escondidos ou substituidos por LOD. Essa decisao e de `rc-world`, nao do renderer baixo nivel.

## Fluxo de chunk

```text
Player/Camera move
  -> calcula chunk central
  -> rc-world decide conjunto desejado de chunks
  -> gera/carrega chunks ausentes
  -> marca chunks dirty
  -> rc-render gera mesh so das faces expostas
  -> Bevy renderiza entidades de chunk dentro do frustum
```

## Meshing inicial

Primeira versao aceitavel:

- chunk 16x16x16 ou 16x128x16, decidir por simplicidade;
- `Vec<BlockType>` indexado por x/y/z;
- gerar face se vizinho for `Air` ou se vizinho estiver fora do chunk e for vazio/desconhecido;
- vertices/normals/UVs por face;
- um material simples para todos os blocos, depois atlas;
- teste unitario para indice e exposicao de faces.

Depois:

- greedy meshing;
- atlas/array texture por face;
- mesh async;
- collider por chunk;
- light propagation por voxel;
- LOD/HLOD.

## Paralelo com outras engines

Unity DOTS/ECS usa uma mentalidade parecida: identidade, dados e sistemas sao separados, e entidades com os mesmos componentes sao agrupadas em chunks/archetypes para processamento em massa. A licao para o `rustcraft`: dados repetitivos de voxel devem ser estruturados em massa, nao em objetos individuais soltos.

Unreal usa Actors como objetos colocaveis/spawnaveis e Components para render, colisao, audio, camera e comportamento. Em mundos grandes, World Partition + HLOD troca muitos atores/meshes distantes por proxies agregados para reduzir draw calls e melhorar performance. A licao: agregacao espacial e proxy renderizavel sao parte normal de engine grande.

Godot organiza jogos como arvores de scenes/nodes. Nodes sao otimos para objetos de jogo e composicao, mas isso nao implica criar um node por voxel. Para mundo grande, o equivalente pratico continua sendo agrupar geometria, usar visibilidade/culling e evitar granularidade excessiva.

## Implicacoes para as crates atuais

- `rc-voxel` deve crescer primeiro: `BlockState`, `Chunk`, indexacao e testes.
- `rc-world` deve parar de spawnar bloco individual e passar a produzir `Chunk`.
- `rc-render` deve parar de expor apenas `block_mesh` e passar a oferecer `build_chunk_mesh`/assets de chunk.
- `rc-player` deve fornecer posicao/camera para sistemas de render distance e raycast.
- `rc-input` ja esta no caminho certo: acao semantica separada de tecla fisica.

## Perguntas abertas

- Chunk vertical: usar chunks cubicos 16x16x16 ou colunas 16xHx16 no MVP?
- Block metadata: `BlockType` simples basta ou precisamos `BlockState` desde ja?
- Fisica: collider por chunk simples com faces expostas ou integrar Rapier somente apos meshing?
- Streaming: unload/despawn distante ja na Sprint 1 ou apenas apos raycast/player estabilizar?

## Fontes

- Bevy Generate Custom Mesh: https://bevy.org/examples/3d-rendering/generate-custom-mesh/
- Bevy Many Cubes: https://bevy.org/examples/stress-tests/many-cubes/
- Bevy Visibility Range: https://bevy.org/examples/3d-rendering/visibility-range/
- Bevy Mesh Ray Cast: https://bevy.org/examples/3d-rendering/mesh-ray-cast/
- Unity ECS overview: https://unity.com/ecs
- Unity DOTS ECS article: https://unity.com/blog/engine-platform/on-dots-entity-component-system
- Unreal Gameplay Framework: https://dev.epicgames.com/documentation/en-us/unreal-engine/gameplay-framework-in-unreal-engine
- Unreal Components: https://dev.epicgames.com/documentation/en-us/unreal-engine/components-in-unreal-engine
- Unreal World Partition HLOD: https://dev.epicgames.com/documentation/en-us/unreal-engine/world-partition---hierarchical-level-of-detail-in-unreal-engine
- Godot Nodes and Scenes: https://docs.godotengine.org/en/4.6/getting_started/step_by_step/nodes_and_scenes.html
- Godot Occlusion Culling: https://docs.godotengine.org/en/4.4/tutorials/3d/occlusion_culling.html

## Links internos

- [[bevy-examples-capability-map]]
- [[workspace-and-crate-organization]]
- [[../10-schedule/sprint-capability-map]]
- [[../ROADMAP]]
