---
title: MOC — Orchestrator (pipeline operacional stub)
type: moc
tags: [moc, agents, orchestrator, pipeline, stub]
created: 2026-04-24
updated: 2026-04-24
---

# Orchestrator — pipeline operacional (stub)

Pasta operacional do pipeline multi-agente. **Atualmente stub** (A-vault-001) — decisão de materialização será tomada em D3 (Passada 4).

## Papel previsto

Agente que **coordena o pipeline**:

- Despacha tasks para Planner/Worker/Reviewer
- Gerencia filas em `_inbox/{planner,worker,reviewer}/`
- Mantém `_session/active-task.md` e `_memory/decisions.md`
- Trata checkpoints e pausas obrigatórias (5 categorias em [[../autonomous-execution]])
- Aplica ciclo SDD 5-fase ([[../../10-knowledge/methodology/sdd-workflow]])
- Consolida verdicts dos 2 reviewers (A+B); divergência → 3º ciclo ou escalona pro humano

## Observação

GSD **não** tem "orchestrator agent" explícito — orquestração é papel do **operador humano** ou de comando slash. Nosso Orchestrator pode adotar mesmo modelo (skill em vez de agent) conforme D3.

## Status

⏳ **Não instrumentado**. A-vault-001 pendente.

## Vizinhos

- [[../_moc]]
- [[../_inbox/_moc]]
- [[../_memory/_moc]]
- [[../_session/_moc]]
- [[../autonomous-execution]]
- [[../gsd-library/pipeline-mapping]] — crosswalk com GSD
