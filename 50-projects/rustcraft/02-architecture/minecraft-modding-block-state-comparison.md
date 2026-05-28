---
title: Minecraft modding block state comparison — rustcraft
type: architecture-note
tags:
  - rustcraft
  - minecraft
  - forge
  - neoforge
  - fabric
  - quilt
  - bedrock
  - block-state
  - architecture
created: '2026-05-27'
updated: '2026-05-27'
source_date: '2026-05-27'
---
# Minecraft modding block state comparison — rustcraft

Pesquisa sobre como Forge, NeoForge, Fabric/Quilt e Bedrock lidam com blocos, estados, variantes e dados extras.

## Conclusao curta

Todos os caminhos maduros convergem para a mesma regra:

```text
Block/type/definition = comportamento e propriedades estaticas
BlockState/permutation = estado finito e compacto salvo no mundo
BlockEntity/custom component/entity auxiliar = dados grandes, infinitos ou comportamento caro
Chunk/subchunk palette = armazenamento compacto dos estados usados localmente
```

Para o `rustcraft`, isso reforca o desenho:

```text
BlockId + BlockState compacto no Chunk
BlockDefinition/Registry fora do Chunk
Tile/BlockEntity auxiliar so para dados complexos
Palette por chunk/subchunk quando precisarmos compactar melhor
```

## Forge

Forge documenta a migracao de metadata numerica antiga para block states. O modelo Java moderno cria um `BlockState` como par unico entre um bloco e um conjunto de propriedades. Exemplos: `facing`, `powered`, `half`, etc.

Licoes:

- Metadata numerica pura e compacta, mas ruim de entender e evoluir.
- Block states tornam o estado nomeado e interpretavel.
- `BlockState`s sao imutaveis.
- Todas as combinacoes de propriedades sao geradas no startup.
- Muitos estados no mesmo bloco aumentam custo de load/memoria/complexidade.
- Se tem nome diferente, tende a ser bloco separado.
- Se tem dado grande ou quase infinito, usar TileEntity/BlockEntity.

## NeoForge

NeoForge segue o modelo moderno e deixa a regra bem explicita:

- blockstate e para estados finitos;
- block entity e para dados arbitrarios/grandes;
- chest usa blockstate para direcao/waterlogged/double chest, mas inventario e comportamento ficam em block entity;
- acima de poucos bits/algumas centenas de estados, considere block entity.

Essa regra e a mais util para o `rustcraft`.

## Fabric

Fabric usa o modelo vanilla: propriedades em `BlockState`, definidas no bloco, com arquivo de blockstate/model para visualizacao. A doc enfatiza que blockstates evitam guardar dados simples em block entity, reduzindo tamanho do mundo e problemas de TPS.

Exemplo: lampada com propriedade booleana `activated`, modelos diferentes e luminancia dependente do estado.

Licoes:

- Estado finito simples deve ficar no blockstate.
- Visual, luz e comportamento podem consultar o state.
- Se multiplas propriedades existem, as combinacoes precisam ser consideradas.

## Quilt

Quilt e construida sobre tecnologia de modding compativel com Fabric. A documentacao de blocos segue o mesmo modelo: registrar bloco, associar blockstates a modelos e usar estados para diferentes formas, como crescimento de crop ou orientacao.

Para nossa decisao, Quilt nao muda o modelo de dominio; e mais uma confirmacao de que o ecossistema Java converge para blockstates/properties.

## Bedrock

Bedrock expõe um modelo mais data-driven:

- custom block JSON define `description.identifier`;
- `states` define variaveis finitas do bloco;
- `permutations` escolhem componentes quando uma condicao baseada em `query.block_state()` e verdadeira;
- `components` sao propriedades/comportamentos base ou sobrescritos por permutation;
- `traits` reaproveitam estados/funcionalidades vanilla como direcao de colocacao, posicao de colocacao e conexoes;
- Script API usa `BlockPermutation` e permite trocar estado/permutacao via `withState`/`setPermutation`.

Bedrock parece lidar melhor para autoria porque evita escrever classe para cada bloco simples: muito vira JSON declarativo. Porem nao e ilimitado: a documentacao menciona limite global de 65.536 permutations por mapa por razoes de performance.

## Java vs Bedrock

| Tema | Java modding | Bedrock |
| --- | --- | --- |
| Definicao | Classe `Block` + registry | JSON `minecraft:block` + identifier |
| Estado finito | `BlockState` com `Property<T>` | `states` + `permutations` |
| Comportamento simples | Metodos da classe/blockstate | Components/traits/permutation components |
| Comportamento custom | Codigo Java | Custom components via Script API |
| Dados grandes | BlockEntity/TileEntity | Script/custom component/storage auxiliar, dependendo do caso |
| Visual | blockstate JSON + models | geometry/material_instances/permutations |
| Limite pratico | combinacoes geradas no startup | cap documentado de permutations |

## Como adaptar para rustcraft

Nao copiar Java literalmente: Rust/Bevy nao precisa de classe por bloco.

Nao copiar Bedrock literalmente: nao precisamos de JSON/permutations ja no MVP.

Modelo recomendado:

```rust
pub struct BlockId(pub u16);

pub struct BlockState {
    pub id: BlockId,
    pub variant: u16,
}

pub struct BlockDefinition {
    pub id: BlockId,
    pub key: &'static str,
    pub default_state: BlockState,
    pub solid: bool,
    pub opaque: bool,
    pub hardness: f32,
    pub render_kind: BlockRenderKind,
    pub state_schema: StateSchema,
}
```

`variant` deve representar uma permutation compacta. O `StateSchema` sabe interpretar bits/valores em nomes de propriedades quando necessario.

Exemplo:

```text
rustcraft:furnace
  state_schema:
    facing: 2 bits
    lit: 1 bit
  variants:
    0..7
```

Para o chunk:

```text
Primeira etapa: Vec<BlockState>
Segunda etapa: PalettedChunkSection { palette: Vec<BlockState>, indices: packed bits }
```

## Regra para decidir onde guardar dado

- `BlockDefinition`: propriedades iguais para todos os blocos daquele tipo.
- `BlockState.variant`: estado finito e pequeno, usado em render/collision/gameplay.
- `BlockEntity`/auxiliar: inventario, texto, energia, dono, timers complexos, dados arbitrarios.
- `Entity`: player, mob, item dropado, projectile, audio emitter, bloco ativo que precisa tick/comportamento proprio.

## Recomendacao atual

Antes de codar mesh por chunk, ajustar `rc-voxel` para:

1. `BlockId`;
2. `BlockState { id, variant }`;
3. `BlockDefinition`;
4. `BlockRegistry` estatico/minimo;
5. `StateSchema` simples ou placeholder;
6. helpers `is_air`, `is_solid`, `is_opaque`.

Nao implementar palette compacta agora. Mas desenhar `Chunk` de modo que depois possa trocar `Vec<BlockState>` por palette sem mudar mundo/render inteiro.

## Fontes

- Forge Block States: https://docs.minecraftforge.net/en/1.16.x/blocks/states/
- NeoForge Blockstates: https://docs.neoforged.net/docs/blocks/states/
- Fabric Block States: https://docs.fabricmc.net/develop/blocks/blockstates
- Quilt About/compatibilidade Fabric: https://quiltmc.org/en/about/
- Quilt simple block/blockstate: https://wiki.quiltmc.org/blocks/first-block
- Bedrock Block States and Permutations: https://learn.microsoft.com/en-us/minecraft/creator/reference/content/blockreference/examples/blockstatesandpermutations?view=minecraft-bedrock-stable
- Bedrock Block Traits: https://learn.microsoft.com/en-us/minecraft/creator/documents/intro-block-traits?view=minecraft-bedrock-stable
- Bedrock Custom Components: https://learn.microsoft.com/en-us/minecraft/creator/documents/scripting/custom-components?view=minecraft-bedrock-stable
- Minecraft Wiki — Block states: https://minecraft.wiki/w/Block_states
- Minecraft Wiki — Chunk format: https://minecraft.wiki/w/Chunk_format

## Links internos

- [[minecraft-block-states-lessons]]
- [[voxel-entity-rendering-model]]
- [[engine-entity-culling-comparison]]
