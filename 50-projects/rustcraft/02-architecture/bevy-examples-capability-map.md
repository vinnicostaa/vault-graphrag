---
title: Bevy examples capability map — rustcraft
type: architecture-note
tags:
  - rustcraft
  - bevy
  - examples
  - sprint-planning
  - architecture
created: '2026-05-27'
updated: '2026-05-27'
source_date: '2026-05-27'
---
# Bevy examples capability map — rustcraft

Mapa sistematico dos exemplos oficiais do Bevy consultados em 2026-05-27 para orientar sprints do `rustcraft`. O workspace atual usa `bevy = "0.18.1"`.

## Decisao de uso

Os exemplos oficiais devem ser usados como referencia de API e de forma de compor plugins/sistemas, mas nao como arquitetura final do jogo. Muitos exemplos sao intencionalmente pequenos; no `rustcraft`, cada area entra por plugin/crate com fronteira clara.

## Mapa por capacidade

| Capacidade | Exemplos Bevy relevantes | Uso no rustcraft | Sprint sugerida |
| --- | --- | --- | --- |
| App, plugins, logs | Examples index, Plugin, Plugin Group, Logs | `crates/rustcraft` continua sendo composicao; cada dominio exporta plugin. Logs entram cedo para debugging. | S0 |
| ECS e entidades | Entity disabling, Observers, Observer Propagation, Iter Combinations | Componentes como dados; sistemas como comportamento; observers para eventos pontuais; nao usar `Disabled` como substituto de visibilidade. | S0-S2 |
| Input | Fixed timestep physics, Gamepad Viewer | Manter `rc-input` convertendo input fisico em acao semantica; acumular input quando entrar fisica fixa. | S1-S2 |
| Player e camera | Free Camera controller, First person view model, 3D viewport to world, Mesh Ray Cast | Separar player/camera; mouse look; raycast para mirar bloco; view model depois. | S2 |
| Mundo voxel escalavel | Generate Custom Mesh, Many Cubes, Visibility Range, Automatic Instancing | Trocar entidade por bloco por dados de chunk + uma entidade render por chunk. Usar exemplos de mesh/culling para validar desempenho. | S1 |
| Renderizacao visivel | Visibility Range, Many Cubes, Shadow Caster and Receiver | Frustum culling do Bevy para entidades renderizaveis; render distance/streaming no app; `VisibilityRange` so quando houver LOD/HLOD. | S1-S4 |
| Meshes, shapes, primitivas | 3D Shapes, Rendering Primitives, Sampling Primitives, Generate Custom Mesh | Usar primitivas para prototipos/debug; custom mesh para chunks; sampling para efeitos/ferramentas, nao para terreno principal. | S1-S3 |
| Texturas e materiais | Texture, Repeated texture configuration, Array Texture, Extended Material, Animated Material | Comecar com `StandardMaterial`; evoluir para atlas/array texture por tipo de face; shader custom so depois que chunk mesh estabilizar. | S3-S6 |
| Luz e sombras | Lighting, Spotlight, Lightmaps, Mixed Lighting, Light Gizmos | MVP usa directional sun + ambient controlado; spotlight e light gizmos para debug; lightmaps so para cenas estaticas/art assets, nao para voxel dinamico inicial. | S3-S5 |
| Animacao e easing | Animated Transform, Eased Motion, Easing Functions, Color Animation | Usar para UI, feedback de interacao, quebrar/colocar bloco, transicoes de menu; nao misturar com movimento fisico do player. | S4-S5 |
| Audio | Audio, Audio Control, Soundtrack, Spatial Audio 3D | `AudioPlayer` para SFX/musica, `AudioSink` para volume/mute/pause, soundtrack por estado, spatial audio para eventos no mundo. | S5 |
| UI, menu e settings | Game Menu, Button, Text, CSS Grid/Flex, UI Scaling, Display and Visibility | Menu por `States`; settings como resources; debug overlay separado de HUD de jogo. | S4-S5 |
| Diagnostics/dev tools | FPS overlay, Custom Diagnostic, Log Diagnostics, 3D Gizmos, Light Gizmos, Picking Debug Tools | Medir FPS, numero de chunks, faces, entidades render, tempo de meshing; gizmos para chunk bounds, raycast e luzes. | S0-S5 |
| Fisica | Run physics in a fixed timestep | Bevy nao traz fisica completa; usar FixedUpdate/interpolacao como base e integrar Rapier/Avian depois. | S6 |
| Shaders | Array Texture, Automatic Instancing, Extended Material, Shader Material | Usar quando `StandardMaterial` e atlas simples nao bastarem; manter shader fora do caminho critico ate haver perfil de necessidade. | S6+ |
| Assets/glTF | Load glTF, Update glTF Scene, Multi-asset synchronization | Usar para player model, ferramentas, props; esperar assets carregarem em telas de loading quando necessario. | S4+ |
| Window | Window Settings, Window Resizing, Screenshot, Low Power | Settings iniciais: titulo, resolucao, vsync/present mode; screenshot para validacao visual futura. | S0-S4 |

## Exemplos que merecem leitura de codigo antes de implementar

1. `generate-custom-mesh`: base direta para gerar mesh de chunk com vertices, normals, UVs e indices.
2. `many-cubes`: alerta de performance para muitos meshes/entidades e referencia para diagnosticos de culling/batching.
3. `visibility-range`: referencia para HLOD/distancia, mas nao substitui streaming de chunk.
4. `mesh-ray-cast`: base para selecionar bloco mirando pela camera.
5. `physics-in-fixed-timestep`: base para separar input, fisica fixa e interpolacao visual.
6. `game-menu`: base para menu por estados e cleanup de telas.
7. `audio-control`, `soundtrack`, `spatial-audio-3d`: base para audio de jogo.
8. `lighting`, `spotlight`, `lightmaps`, `mixed-lighting`: base para luz dinamica, sombras e investigacao futura de bake.
9. `array-texture` e `automatic-instancing`: base futura para atlas/array texture e render massivo.

## Fontes

- Bevy examples index: https://bevy.org/examples/
- Generate Custom Mesh: https://bevy.org/examples/3d-rendering/generate-custom-mesh/
- Many Cubes: https://bevy.org/examples/stress-tests/many-cubes/
- Visibility Range: https://bevy.org/examples/3d-rendering/visibility-range/
- Mesh Ray Cast: https://bevy.org/examples/3d-rendering/mesh-ray-cast/
- Fixed timestep physics: https://bevy.org/examples/movement/physics-in-fixed-timestep/
- Entity Disabling: https://bevy.org/examples/ecs-entity-component-system/entity-disabling/
- Observers: https://bevy.org/examples/ecs-entity-component-system/observers/
- Game Menu: https://bevy.org/examples/games/game-menu/
- Audio Control: https://bevy.org/examples/audio/audio-control/
- Soundtrack: https://bevy.org/examples/audio/soundtrack/
- Spatial Audio 3D: https://bevy.org/examples/audio/spatial-audio-3d/
- Lighting: https://bevy.org/examples/3d-rendering/lighting/
- Lightmaps: https://bevy.org/examples/3d-rendering/lightmaps/
- Mixed Lighting: https://bevy.org/examples/3d-rendering/mixed-lighting/
- Spotlight: https://bevy.org/examples/3d-rendering/spotlight/
- Texture: https://bevy.org/examples/3d-rendering/texture/
- Animated Material: https://bevy.org/examples/3d-rendering/animated-material/
- Animated Transform: https://bevy.org/examples/animation/animated-transform/
- Color Animation: https://bevy.org/examples/animation/color-animation/
- Eased Motion: https://bevy.org/examples/animation/eased-motion/
- Easing Functions: https://bevy.org/examples/animation/easing-functions/
- Array Texture: https://bevy.org/examples/shaders/array-texture/
- Automatic Instancing: https://bevy.org/examples/shaders/automatic-instancing/
- Extended Material: https://bevy.org/examples/shaders/extended-material/

## Links internos

- [[voxel-entity-rendering-model]]
- [[../10-schedule/sprint-capability-map]]
- [[workspace-and-crate-organization]]
- [[../ROADMAP]]
