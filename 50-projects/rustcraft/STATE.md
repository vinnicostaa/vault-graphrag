---
title: STATE — rustcraft
type: state
tags:
  - state
  - rustcraft
created: '2026-05-27'
updated: '2026-05-28'
---
# STATE — rustcraft

## Decisões vivas

| ID | Status | Decisão |
| --- | --- | --- |
| D-001 | accepted | Bevy continua como runtime principal para o MVP de aprendizado. |
| D-002 | accepted | O repositório usa Cargo workspace com múltiplas packages internas para fins didáticos. |
| D-003 | accepted | Crates iniciais permitidas: `rustcraft`, `rc-input`, `rc-player`, `rc-voxel`, `rc-world`, `rc-render`, desde que o grafo de dependências continue sem ciclos. |
| D-004 | accepted | Não usar `protocol` para ações de jogador. Preferir `actions`, `controls`, `input` ou `player_controller`. |
| D-005 | accepted | O agente atua como tutor; o usuário codifica. Mudança de código pelo agente exige pedido explícito. |
| D-006 | accepted | Organização inicial recomendada: multi-crate workspace com `crates/rustcraft` como bin principal e libraries internas `rc-*`. |
| D-007 | accepted | O projeto deve seguir o contrato raiz do vault: leitura de `../../CLAUDE`, mutação via MCP Obsidian, frontmatter, MOCs e links. |
| D-008 | accepted | O path canônico de audit do projeto é `11-audit/AUDIT.md`; o `AUDIT.md` raiz é legado/compatibilidade até decisão final de double-write ou remoção. |
| D-009 | accepted | `SESSION_CHARTER.md`, `STEERING.md`, `DoR-DoD.md` e MOCs 00-12 fazem parte do mínimo operacional do projeto no vault. |
| D-010 | accepted | `20-stacks/rust` permanece lazy por enquanto; padrões Rust/Bevy descobertos no `rustcraft` ficam locais até estabilizarem ou servirem a outro projeto. |
| D-011 | accepted | O projeto nasceu no historico git como package `minecraft` (`e7df14e`), mas `rustcraft` e o nome canonico atual do workspace/app. |
| D-012 | accepted | Blocos comuns de terreno nao devem ser entidades Bevy; o modelo alvo e dados de chunk + uma entidade renderizavel/collider por chunk. Ver [[02-architecture/voxel-entity-rendering-model]]. |
| D-013 | accepted | O planejamento de capacidades deve usar o mapa de exemplos Bevy em [[02-architecture/bevy-examples-capability-map]] e o mapa de sprints em [[10-schedule/sprint-capability-map]]. |
| D-014 | accepted | `BlockType` enum e apenas prototipo; o modelo alvo e `BlockId` + `BlockState { id, variant }` + `BlockDefinition` + `BlockRegistry`. Ver [[02-architecture/minecraft-block-states-lessons]] e [[02-architecture/minecraft-modding-block-state-comparison]]. |
| D-015 | accepted | Estado finito pequeno fica em `BlockState`; dados grandes/arbitrarios ficam em BlockEntity/auxiliar ligado a `BlockPos`; objetos com comportamento proprio viram entidades Bevy. Ver [[02-architecture/minecraft-modloader-api-lessons]]. |
| D-016 | accepted | IDs nomespaceados estaveis devem existir para blocos/definicoes, mesmo que o runtime use IDs compactos (`u16`) para chunks, save e render. |
| D-017 | accepted | `rc-render` e responsavel por transformar dados voxel (`Chunk`/`BlockState`) em buffers de mesh e `Mesh` Bevy; `rc-world` deve consumir essa API em vez de montar geometria manualmente. |
| D-018 | accepted | Diagnosticos de runtime usam plugins oficiais do Bevy na composicao do app e `DiagnosticPath` customizado nos dominios internos; metricas de chunk inicial sao proxy ate existir streaming/chunk map. |
| D-019 | accepted | Conversao de hit de raycast para `BlockPos` pertence ao dominio voxel puro (`rc-voxel`); sistemas Bevy de interacao devem usar essa API em vez de duplicar matematica de coordenadas. |
| D-020 | accepted | Mouse look pertence ao dominio `rc-player` e consome movimento acumulado do mouse do Bevy; captura/trava de cursor e uma preocupacao separada de janela/input. |

## Dívidas técnicas / pendências

| ID | Status | Pendência |
| --- | --- | --- |
| DT-001 | closed | Código atual foi revisado e `protocol` foi removido do fluxo local; o módulo agora é `actions.rs`. |
| DT-002 | closed | Executável principal escolhido: `crates/rustcraft/src/bin/rustcraft.rs`, com composição em `crates/rustcraft/src/app.rs` e export por `lib.rs`. |
| DT-003 | closed | README e ARCHITECTURE documentam a árvore multi-crate implementada. |
| DT-004 | closed | Protótipo original/progressivo foi provado em estrutura limpa: `cargo run` validou janela/mundo/player, e T-003 registra métricas runtime depois do spawn por chunk. |
| DT-005 | closed | ADR-0003 implementada no código com `rc-input`, `rc-player`, `rc-voxel`, `rc-render`, `rc-world` e `rustcraft` como package de composição. |
| DT-006 | closed | `context-1.md` e `context-2.md` nao existem no workspace atual; `.gitignore` passa a ignorar `context-*.md` como artefato de sessao. |
| DT-007 | open | Decidir política definitiva para `AUDIT.md` raiz: remover, transformar em ponteiro, ou manter double-write temporário com `11-audit/AUDIT.md`. |
| DT-008 | open | Completar conteúdo substantivo das áreas 00-12 além dos MOCs iniciais. |
| DT-010 | closed | T-BEVY-001 concluida: `Chunk` existe em `rc-voxel` e `rc-world::generate_chunk` gera dados de chunk por `ChunkCoord`. |
| DT-011 | closed | Spawn principal trocado para uma entidade renderizavel por chunk usando `rc-render::build_chunk_mesh`; material unico temporario permanece como limitacao visual ate atlas/vertex color. |
| DT-009 | closed | `20-stacks/rust/_moc` foi estudado; permanece lazy por D-010, com padrões Rust/Bevy locais ao `rustcraft` por enquanto. |

## Notas

A estrutura deve ensinar Cargo/Bevy corretamente: workspace ≠ qualquer pasta; crate/package ≠ módulo; binary target ≠ library target.

Em 2026-05-27, o repositório passou a refletir essa decisão: root workspace, `crates/rustcraft` como package de composição e crates internas `rc-*` por responsabilidade.

Historico confirmado: em `e7df14e`, o package inicial se chamava `minecraft`; isso e origem historica, nao nome canonico atual.
