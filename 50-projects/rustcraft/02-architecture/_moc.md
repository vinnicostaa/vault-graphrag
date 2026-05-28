---
title: MOC — rustcraft 02-architecture
type: moc
tags:
  - moc
  - rustcraft
  - architecture
created: '2026-05-27'
updated: '2026-05-27'
---
# MOC — 02-architecture

Arquitetura do `rustcraft`.

## Notas principais

- [[workspace-and-crate-organization]] — organizacao Cargo workspace/crates.
- [[crate-module-organization-plan]] — plano de organização interna das crates por domínio.
- [[rust-project-organization-research]] — evidências de Rust oficial, Bevy, Zed, COSMIC, 40tude e referências externas para organização de projeto.
- [[bevy-examples-capability-map]] — mapa de exemplos oficiais Bevy por capacidade/sprint.
- [[voxel-entity-rendering-model]] — decisao sobre blocos, chunks, entidades e renderizacao visivel.
- [[engine-entity-culling-comparison]] — comparacao direta entre Bevy, Unity, Unreal, Godot e padroes de empresas grandes.
- [[minecraft-block-states-lessons]] — leitura do modelo de Block States do Minecraft e implicacoes para `BlockId`/`BlockState`.
- [[minecraft-modding-block-state-comparison]] — comparacao Forge, NeoForge, Fabric, Quilt e Bedrock para estados/permutations/block entities.
- [[minecraft-modloader-api-lessons]] — licoes de registries, lifecycle, client/server, data-driven APIs, eventos e erros historicos dos mod loaders.
- [[adr/_moc]] — decisoes arquiteturais.

## Estado atual

- ADR-0003 foi implementada no repositorio em 2026-05-27. Ver [[../11-audit/AUDIT#A-006]] e [[../11-audit/audit-log/2026-05-27-multi-crate-restructure]].

## Links

- [[../STATE]]
- [[../03-subprojects/_moc]]
- [[../../20-stacks/rust/_moc]]
