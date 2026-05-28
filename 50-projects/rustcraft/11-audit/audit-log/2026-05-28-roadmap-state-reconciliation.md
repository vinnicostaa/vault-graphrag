---
title: Audit log — roadmap and state reconciliation
type: audit
tags:
  - audit-log
  - rustcraft
  - roadmap
  - state
created: '2026-05-28'
updated: '2026-05-28'
---
# Audit log — roadmap and state reconciliation

## Contexto

Após o commit `28cff15`, uma revisão cruzada encontrou divergências entre código, vault e documentação.

## Correções aplicadas

- A-015 foi fechado: `rustcraft` agora usa `rc_voxel::block_pos_from_hit` diretamente em `interaction.rs`, então a dependência direta de `rc-voxel` é justificada.
- DT-004 foi fechado: o protótipo já foi provado em runtime e T-003 registra a validação.
- ROADMAP foi atualizado:
  - Fase 0 marcada como concluída.
  - Fase 1 marcada como concluída.
  - Fase 2 marcada como parcialmente concluída, com raycast/seleção de bloco feito e mouse look/quebrar-colocar/inventário ainda abertos.
- T-001 em `mvp-0-tasks` foi fechado para alinhar o arquivo de tasks ao ROADMAP.

## Itens que continuam abertos

- A-009 / DT-007: política final para `AUDIT.md` legado na raiz.
- A-014: `Block` e `GeneratedChunkBlock` continuam como componentes legados sem uso ativo.
- DT-008: preencher conteúdo substantivo das áreas 00-12 além dos MOCs iniciais.

## Evidência local

- `cargo test --workspace`: 33 testes passando após T-BEVY-005.
- `cargo clippy --workspace --all-targets --all-features -- -D warnings`: passou.
- Grafo de dependências continua sem ciclos; `rc-voxel` permanece sem Bevy.

## Links

- [[../../ROADMAP]]
- [[../../STATE]]
- [[../AUDIT]]
- [[../../05-tasks/mvp-0-tasks]]
- [[../../05-tasks/bevy-example-study-tasks]]
