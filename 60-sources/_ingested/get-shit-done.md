---
title: _ingested/get-shit-done
type: note
tags:
  - ingested
  - get-shit-done
  - gsd
  - tracking
source: ../get-shit-done
created: 2026-04-22T00:00:00.000Z
updated: '2026-04-24'
---

# Ingestão — get-shit-done

**Tracker** das destilações. Fila viva em [[../get-shit-done/_toc#Prioridade de destilação]].

## Dívidas prioritárias (P0/P1)

| Prio | Origem | Destino | Status |
|---|---|---|---|
| P0 | 33 `gsd-*` agents em `60-sources/get-shit-done/raw/agents/` | `30-agents/gsd-library/card-gsd-<name>.md` + 6 famílias | ✅ 33/33 + pipeline-mapping + gsd-resync (2026-04-24, D-vault-004) |
| P0 | [[../get-shit-done/raw/docs/ARCHITECTURE]] + [[../get-shit-done/raw/get-shit-done/references/agent-contracts]] | contexto em `30-agents/_moc` + `30-agents/agent-contracts.md` | ⏳ pendente |
| P0 | Workflows ciclo 5 fases ([[../get-shit-done/raw/get-shit-done/workflows/discuss-phase\|discuss]]/plan/execute/verify/ship) | `10-knowledge/methodology/sdd-workflow` ampliado | ⏳ pendente |
| P0 | [[../get-shit-done/raw/get-shit-done/templates/claude-md]] vs `40-templates/CLAUDE.md.template` | reconciliar | ⏳ pendente |
| P1 | SPEC/STATE/UAT templates | `40-templates/gsd-*` | ⏳ pendente |
| P1 | [[../get-shit-done/raw/get-shit-done/references/context-budget]] + [[../get-shit-done/raw/docs/context-monitor]] | `30-agents/autonomous-execution` (complementar) | ⏳ pendente |
| P1 | [[../get-shit-done/raw/get-shit-done/references/gates]] + [[../get-shit-done/raw/get-shit-done/references/gate-prompts]] | `30-agents/gates.md` | ⏳ pendente |

## Destiladas em staging local (`_notes/`)

*(nenhuma ainda)*

## Promovidas pro vault canônico

### 2026-04-24 — GSD Agents Library (D-vault-004)

Biblioteca em `30-agents/gsd-library/`:

- [[../../30-agents/gsd-library/_moc]] — índice da biblioteca
- [[../../30-agents/gsd-library/pipeline-mapping]] — crosswalk Planner/Worker/Reviewer/Orchestrator × GSD agents
- **33 reference-cards** (`card-gsd-<name>.md`) — 1 por agent, 40-60 linhas, destilando intenção + I/O + ferramentas + padrão-chave replicável.
- **6 famílias** (fiel ao TOC original):
  - [[../../30-agents/gsd-library/family-research]] — 10 agents (investigar/mapear/sintetizar)
  - [[../../30-agents/gsd-library/family-planning]] — 5 agents (decompor/ordenar)
  - [[../../30-agents/gsd-library/family-executor]] — 4 agents (implementar/corrigir/revisar)
  - [[../../30-agents/gsd-library/family-audit]] — 8 agents (verificar/auditar/certificar)
  - [[../../30-agents/gsd-library/family-debug]] — 2 agents (investigação post-mortem)
  - [[../../30-agents/gsd-library/family-docs]] — 4 agents (classificar/sintetizar/verificar/escrever docs)

Playbook associado: [[../../30-agents/playbooks/gsd-resync]] — processo de re-sincronização quando upstream evolui.

## Vizinhos

- [[../get-shit-done/_toc]]
- [[_moc]]
- [[../../00-meta/STATE]] — D-vault-004
