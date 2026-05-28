---
title: MOC — rustcraft audit-log
type: moc
tags:
  - moc
  - audit-log
  - rustcraft
created: '2026-05-27'
updated: '2026-05-28'
---
# MOC — audit-log

Audits datados do `rustcraft`.

## Entradas

- [[2026-05-27-vault-usage-audit]] — auditoria do uso do vault e normalizacao inicial.
- [[2026-05-27-multi-crate-restructure]] — fechamento da divergencia entre ADR-0003 e a estrutura real do repositorio.
- [[2026-05-27-repository-consistency-correction]] — correcao de pendencias fantasma e registro do historico `minecraft` -> `rustcraft`.
- [[2026-05-27-bevy-examples-sprint-mapping]] — pesquisa dos exemplos Bevy e mapeamento de capacidades/sprints.
- [[2026-05-27-chunk-and-seeded-generation]] — implementacao inicial de `Chunk` e geracao deterministica com seed.
- [[2026-05-27-chunk-backed-spawn-and-readme]] — spawn inicial lendo de `Chunk`, README atualizado e registro da pendencia de CPU/mesh por chunk.
- [[2026-05-28-chunk-meshing-api]] — API de meshing por chunk em `rc-render`, alinhada ao padrao Bevy de custom mesh.
- [[2026-05-28-agent-and-coding-practices]] — AGENTS.md, CODING_PRACTICES.md e task guiada para spawn por chunk.
- [[2026-05-28-chunk-mesh-spawn]] — spawn principal trocado para uma entidade renderizavel por chunk.
- [[2026-05-28-base-runtime-diagnostics]] — diagnosticos oficiais Bevy ligados no app principal.
- [[2026-05-28-chunk-mesh-custom-diagnostics]] — diagnosticos customizados de chunk, faces e vertices.
- [[2026-05-28-block-raycast-highlight]] — raycast de bloco, conversao para `BlockPos` e highlight debug.
- [[2026-05-28-roadmap-state-reconciliation]] — reconciliacao de ROADMAP, STATE e AUDIT apos raycast.
- [[2026-05-28-player-mouse-look]] — mouse look no `rc-player`, rotacao antes do movimento e validacao da task T-BEVY-009.

## Links

- [[../AUDIT]]
- [[../../STATE]]
- [[../../SESSION_CHARTER]]
