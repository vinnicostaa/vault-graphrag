---
title: Minecraft mod loader API lessons — rustcraft
type: architecture-note
tags:
  - rustcraft
  - minecraft
  - modding
  - forge
  - neoforge
  - fabric
  - quilt
  - bedrock
  - architecture
created: '2026-05-27'
updated: '2026-05-27'
source_date: '2026-05-27'
---
# Minecraft mod loader API lessons — rustcraft

Pesquisa de arquitetura sobre como docs de Forge, NeoForge, Fabric, Quilt e Bedrock ajudam o `rustcraft` a evitar erros estruturais antes de codar sistemas grandes.

## Tese

As docs de mod loaders sao uteis porque mostram onde Minecraft precisou criar APIs publicas em cima de uma engine que cresceu por muitos anos. Elas revelam tres coisas:

1. limites do modelo interno do Minecraft;
2. pontos onde modders precisam de extensao estavel;
3. erros historicos que podemos evitar no `rustcraft`.

O objetivo nao e copiar Forge/NeoForge/Fabric/Bedrock. O objetivo e transformar as licoes em contratos simples para Rust/Bevy.

## Padroes que se repetem

### 1. Registry com IDs nomeados

Forge e NeoForge colocam registry no centro: blocos, itens, entidades, block entities, sons e outros objetos precisam ser registrados por IDs nomespaceados.

Licoes para o `rustcraft`:

- nao usar enum global como fonte final de verdade;
- usar chaves estaveis como `rustcraft:stone` e `rustcraft:grass`;
- `BlockId` pode ser compacto em runtime, mas deve mapear para uma chave estavel;
- carregamento/save/network nao deve depender de ordem acidental de enum;
- preparar remap/missing entries no futuro evita mundos quebrados ao remover/renomear blocos.

### 2. Deferred registration / lifecycle

Forge/NeoForge existem em torno de eventos de registro e `DeferredRegister` para evitar bugs de ordem de inicializacao.

Licoes:

- separar fase de declarar definicoes da fase de usar definicoes;
- `BlockRegistry` deve ser construido antes de worldgen/meshing;
- sistemas nao devem consultar registry incompleto;
- no futuro, packs/mods devem carregar antes de iniciar mundo.

### 3. BlockState finito, BlockEntity para dado grande

Java modding converge para:

```text
Block -> comportamento/propriedades base
BlockState -> estado finito e pequeno
BlockEntity -> dados grandes/arbitrarios
```

Licoes:

- `BlockState` deve ficar compacto no chunk;
- variantes finitas: `facing`, `axis`, `lit`, `waterlogged`, `age`, `shape`;
- inventario, energia, texto, dono, timers complexos e dados arbitrarios nao ficam em `BlockState`;
- criar `BlockEntity`/auxiliary data ligado a `BlockPos` quando necessario.

### 4. Evitar explosao combinatoria

Forge/NeoForge alertam que todas as combinacoes de propriedades geram estados. Fabric mostra que multiplas propriedades multiplicam variantes visuais. Bedrock tambem tem limite pratico global de permutations.

Licoes:

- propriedade so entra em `BlockState` se for pequena, finita e usada por render/collision/gameplay basico;
- se um bloco precisa de centenas/milhares de combinacoes, modelar como dado auxiliar;
- nao transformar cor, inventario, dono, nome custom, energia ou conteudo em permutation.

### 5. Data-driven resources

Bedrock e datapacks/registries do Java empurram muito para JSON/data. Fabric/Forge/NeoForge tambem usam data generation para evitar arquivos manuais gigantes.

Licoes:

- no MVP, registry pode ser Rust estatico;
- depois, migrar definicoes de blocos para arquivos (`ron`/`json`/asset Bevy) sem mudar chunk/render;
- gerar assets derivados quando possivel: modelos, atlas, lang, loot, recipes;
- o caminho moderno e data-driven, mas com validacao forte.

### 6. Tags e agrupamentos

Mods e datapacks usam tags para agrupar blocos/itens: mineable, logs, leaves, replaceable, etc.

Licoes:

- nao hardcodar listas como `is_wood` via match gigante;
- preparar `BlockTag`/`TagRegistry` depois do `BlockRegistry`;
- gameplay deve perguntar `registry.has_tag(block, tag)` quando fizer sentido.

### 7. Eventos e extension points

Mod loaders expõem eventos porque modificar engine interna diretamente quebra compatibilidade.

Licoes:

- no `rustcraft`, preferir eventos/sistemas claros para: bloco colocado, bloco removido, chunk gerado, chunk meshed, player interagiu;
- evitar que `rc-world` chame diretamente detalhes de UI/audio/render avançado;
- usar Bevy events/observers com cuidado para pontos de extensao, nao para fluxo massivo por bloco.

### 8. Capabilities / attachments / API lookup

Forge/NeoForge/Fabric tem mecanismos para anexar dados e consultar comportamento sem depender de classe concreta.

Licoes:

- no futuro, inventario/energia/fluidos nao devem ser `match BlockId` espalhado;
- usar uma ideia parecida com `capability`: perguntar se um bloco/entidade em `BlockPos` oferece `Inventory`, `Energy`, `Fluid`, etc.;
- attachments sao bons para dados extras em chunk/entity/world sem poluir structs centrais.

### 9. Client/server sides

Forge/NeoForge enfatizam separacao entre lado fisico e logico. Erros comuns incluem executar logica no client, acessar classes client-only no server, ou usar static compartilhado indevidamente.

Licoes:

- mesmo em singleplayer Bevy, separar simulacao e apresentacao;
- `rc-world` e `rc-voxel` nao devem depender de render;
- `rc-render` deve consumir estado autorizado, nao decidir regras de mundo;
- isso prepara multiplayer futuro sem reescrever tudo.

### 10. Networking/sync como contrato

Mod loaders deixam claro que dados precisam de sincronizacao explicita entre lados.

Licoes:

- mundo autoritativo deve emitir delta de bloco/chunk;
- dirty chunks e dirty block entities devem existir como conceito;
- render/client consome atualizacao, nao muta mundo por conta propria;
- diagnosticos devem medir bytes/eventos/dirty updates no futuro.

## Bedrock como referencia de caminho

Bedrock e especialmente interessante porque expoe um caminho mais moderno para autoria:

```text
identifier
states
permutations
components
traits
custom components
```

Licoes especificas:

- `states` sao variaveis finitas;
- `permutations` aplicam componentes diferentes conforme condicoes;
- componentes base podem ser sobrescritos por permutation;
- traits reaproveitam comportamento vanilla como direcao de colocacao, posicao e conexoes;
- custom components conectam JSON a comportamento via script;
- existe limite global de permutations por performance.

Para o `rustcraft`, a ideia equivalente seria:

```text
BlockDefinition
  key: rustcraft:furnace
  default_state
  components: [solid, opaque, hardness, render]
  state_schema: facing + lit
  permutations: regras que alteram render/luz/collision por state
  behavior_hooks: callbacks/sistemas futuros
```

Nao implementar isso tudo agora. Mas o modelo `BlockId + BlockState { variant } + BlockDefinition` deve deixar esse caminho aberto.

## Erros historicos que podemos evitar

- Metadata numerica sem significado: compacta, mas ruim para debugging e evolucao.
- Enum gigante como API publica: quebra modding, save e extensao.
- Estado demais em blockstate: explode combinacoes e custo de load/render.
- Block entities demais: aumenta tick/storage/sync quando dado simples bastaria em state.
- Acoplamento client/server: causa desync e crash em servidor dedicado.
- Assets manuais duplicados: blockstate/model/loot/lang divergindo entre si.
- Dependencia de internals: mod loaders sofrem a cada mudanca interna do Minecraft.
- Compatibilidade por camadas demais: Quilt/Fabric mostram que compatibilidade e poderosa, mas custa manutencao.

## Decisoes recomendadas para o rustcraft

### Agora

- Trocar `BlockType` por `BlockId` + `BlockState`.
- Criar `BlockDefinition` e `BlockRegistry` minimo em Rust.
- Usar IDs nomespaceados estaveis mesmo que runtime use `u16`.
- Manter chunk com `Vec<BlockState>` por simplicidade.
- Criar `BlockEntity` apenas como conceito documentado, nao implementacao completa.

### Logo depois

- `StateSchema` simples para interpretar `variant`.
- Dirty chunks/dirty block entities.
- Eventos: `BlockChanged`, `ChunkGenerated`, `ChunkMeshBuilt`.
- Tags basicas: `solid`, `opaque`, `terrain`, talvez como flags antes de tag registry completo.

### Mais tarde

- Registry data-driven por arquivo.
- Palette por chunk/subchunk.
- Sistema de capabilities para inventario/energia/fluidos.
- Networking/sync delta.
- Mod/plugin API propria.

## Fontes

- Forge Registries: https://docs.minecraftforge.net/en/latest/concepts/registries/
- Forge Sides: https://docs.minecraftforge.net/en/latest/concepts/sides/
- Forge Block States: https://docs.minecraftforge.net/en/latest/blocks/states/
- NeoForge Registries: https://docs.neoforged.net/docs/concepts/registries/
- NeoForge Sides: https://docs.neoforged.net/docs/concepts/sides/
- NeoForge Data Attachments: https://docs.neoforged.net/docs/datastorage/attachments/
- NeoForge Blockstates: https://docs.neoforged.net/docs/blocks/states/
- Fabric Block States: https://docs.fabricmc.net/develop/blocks/blockstates
- Fabric Block Entities: https://docs.fabricmc.net/develop/blocks/block-entities
- Fabric Data Attachments: https://docs.fabricmc.net/develop/data-attachments
- Fabric Networking: https://docs.fabricmc.net/develop/networking
- Fabric Events: https://docs.fabricmc.net/develop/events
- Quilt About: https://quiltmc.org/en/about/
- Quilt simple block docs: https://wiki.quiltmc.org/blocks/first-block
- Bedrock States and Permutations: https://learn.microsoft.com/en-us/minecraft/creator/reference/content/blockreference/examples/blockstatesandpermutations?view=minecraft-bedrock-stable
- Bedrock Traits: https://learn.microsoft.com/en-us/minecraft/creator/documents/intro-block-traits?view=minecraft-bedrock-stable
- Bedrock Custom Components: https://learn.microsoft.com/en-us/minecraft/creator/documents/scripting/custom-components?view=minecraft-bedrock-stable

## Links internos

- [[minecraft-modding-block-state-comparison]]
- [[minecraft-block-states-lessons]]
- [[voxel-entity-rendering-model]]
- [[bevy-examples-capability-map]]
