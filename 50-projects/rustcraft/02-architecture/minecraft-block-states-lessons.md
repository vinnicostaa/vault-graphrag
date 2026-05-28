---
title: Minecraft block states lessons — rustcraft
type: architecture-note
tags:
  - rustcraft
  - minecraft
  - voxel
  - block-state
  - architecture
created: '2026-05-27'
updated: '2026-05-27'
source_date: '2026-05-27'
---
# Minecraft block states lessons — rustcraft

Referencia: https://minecraft.wiki/w/Block_states

## Ideia principal

Minecraft nao trata cada variante como um bloco completamente separado no sentido conceitual. Ele usa um bloco base mais propriedades de estado.

Exemplos:

- log: `axis=x|y|z`;
- furnace: `facing=north|east|south|west`, `lit=false|true`;
- slab: `type=bottom|top|double`, `waterlogged=false|true`;
- stairs: `facing`, `half`, `shape`, `waterlogged`;
- leaves: `distance`, `persistent`, `waterlogged`;
- water: `level` e fluid state.

Isso confirma que o nosso `enum BlockType { Air, Grass, Dirt, Stone }` e apenas prototipo. O modelo escalavel precisa distinguir identidade do bloco e estado concreto no mundo.

## Modelo mental

```text
Block kind / definition
  "minecraft:stairs" / "rustcraft:grass" / "rustcraft:furnace"

Block state
  bloco base + propriedades atuais
  ex: furnace[facing=north, lit=true]
```

No `rustcraft`, isso sugere:

```text
BlockId          -> identidade compacta do tipo base
BlockState       -> valor armazenado no chunk
BlockDefinition  -> propriedades estaticas do tipo base
BlockRegistry    -> catalogo de definicoes
State schema     -> propriedades permitidas por tipo de bloco
```

## Nao copiar literalmente agora

Minecraft tem muitas propriedades porque o jogo ja tem anos de blocos, redstone, agua, plantas, portas, slabs, stairs, sinais, etc. Para o `rustcraft`, copiar tudo agora seria overengineering.

A decisao pragmaticamente correta:

1. Trocar `BlockType` por `BlockId` + `BlockState`.
2. Criar `BlockRegistry` estatico com `air`, `grass`, `dirt`, `stone`.
3. Adicionar propriedades estaticas basicas: `solid`, `opaque`, `render_kind`, `hardness`.
4. Preparar `BlockState` para carregar variantes no futuro, mas nao implementar propriedade dinamica generica ainda.

## Design recomendado para a primeira versao

```rust
pub struct BlockId(pub u16);

pub struct BlockState {
    pub id: BlockId,
    pub variant: u16,
}
```

`variant` pode ser `0` para blocos simples. Depois, cada `BlockDefinition` decide como interpretar variantes.

Exemplos futuros:

```text
log variant:
  0 = axis_y
  1 = axis_x
  2 = axis_z

furnace variant:
  bits 0..1 = facing
  bit 2 = lit

slab variant:
  bits 0..1 = bottom/top/double
  bit 2 = waterlogged
```

Isso evita alocar hashmap de propriedades por bloco dentro do chunk. Chunk precisa ser compacto.

## Separar propriedades estaticas e estado dinamico

Nao misturar tudo em `BlockState`.

Propriedades estaticas ficam na definicao:

- nome;
- solidez padrao;
- opacidade padrao;
- dureza;
- render kind;
- textura/material base;
- se pode emitir luz;
- se tem entidade auxiliar.

Estado por bloco fica no chunk:

- id;
- variant compacta;
- talvez light level no futuro;
- talvez metadata auxiliar para casos raros.

Dados grandes ou inventario nao ficam no `BlockState`. Exemplo: bau nao guarda itens dentro do bloco compacto. O chunk guarda `BlockState` do bau e existe uma entidade/tile data ligada ao `BlockPos` para inventario.

## Regras para o rustcraft

- Bloco comum: `BlockState { id, variant: 0 }`.
- Bloco orientado: usar variant compacta.
- Bloco com comportamento pesado: `BlockState` + entidade auxiliar.
- Fluido: provavelmente sistema proprio depois, inspirado em fluid states.
- Render: meshing consulta `BlockRegistry` e interpreta `variant`.
- Gameplay: interacao consulta definicao e, se necessario, entidade auxiliar.

## Implicacao para a proxima codificacao

Antes de chunk mesh, vale ajustar `rc-voxel` para ter:

- `BlockId`;
- `BlockState`;
- constantes `AIR`, `GRASS`, `DIRT`, `STONE`;
- `BlockDefinition`;
- `BlockRegistry` estatico/minimo;
- helpers `is_air`, `is_solid`, `is_opaque`.

Depois disso o chunk ja nasce com o modelo certo.

## Links internos

- [[voxel-entity-rendering-model]]
- [[engine-entity-culling-comparison]]
- [[bevy-examples-capability-map]]
