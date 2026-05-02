---
title: MOC — 30-agents
type: moc
tags: [moc, agents, multi-agent]
created: 2026-04-22
---

# 30-agents — Sistema multi-agente

Operação do pipeline **Planner → Worker → Dual Reviewer** do automation-agents. Esta pasta é **dinâmica** — agentes escrevem aqui, não só leem.

## Pastas operacionais

- [[planner/_moc]] — briefings e plans gerados
- [[worker/_moc]] — outputs de implementação
- [[reviewer/_moc]] — reviews (A e B) + veredictos
- [[orchestrator/_moc]] — coordenação e logs do pipeline
- [[_memory/_moc]] — memória persistente cross-sessão (patterns, failures, decisions)
- [[_session/_moc]] — estado da sessão atual (active-task, agent-state)
- [[_inbox/_moc]] — fila de tasks por role (planner/worker/reviewer)
- [[playbooks/_moc]] — playbooks **obrigatórios** (research-loop, dual-check, dep-audit, cve-scan, onboarding, plan-and-execute, rollback, community-summary, gsd-resync)
- [[skills/_moc]] — skills Claude Code reutilizáveis
- [[gsd-library/_moc]] — biblioteca destilada dos 33 agents do ecossistema [gsd-build/get-shit-done](https://github.com/gsd-build/get-shit-done); informa este pipeline, não substitui (ver [[gsd-library/pipeline-mapping]] e [[../00-meta/STATE|D-vault-004]])

## Contratos operacionais (não-negociáveis)

- [[autonomous-execution]] — modo autônomo, limites, categorias destrutivas
- [[mcp-integration]] — uso de MCPs (Context7, Obsidian, Postgres, GitHub, etc.)
- [[claude-code-plugins]] — plugins por fase do SDD

## Fluxo canônico

```
user cria task
  ↓
_inbox/planner/<task-id>.md
  ↓  Planner consome
planner/plans/<task-id>.md → _inbox/worker/<task-id>.md
  ↓  Worker consome
worker/outputs/<task-id>.md → _inbox/reviewer/<task-id>.md
  ↓  Reviewer (A + B paralelos) consome
reviewer/reviews/<task-id>-A.md  +  reviews/<task-id>-B.md
  ↓  Orchestrator resolve
orchestrator/verdicts/<task-id>.md  (approved | revise | rejected)
  ↓
_memory/patterns.md + _memory/failures.md atualizados
```

## Regras duras (do [[../40-templates/AGENTS_SPEC]])

- Todos os SubAgents em **Opus**.
- Reviewer A e B rodam em **paralelo**, **independentes** — não veem a review um do outro.
- **Bootstrap de memória obrigatório** antes de qualquer decisão: ler `_memory/*` + `../00-meta/STATE` + `../00-meta/AUDIT`.
- Máximo **2 ciclos** de revisão por task — se ainda rejeitado, escalar pro humano.
- `.kiro/STATE.md` (ou equivalente no projeto em questão) **sempre vence** sobre o que está em `_memory/` em caso de divergência.

## Vizinhos no grafo

- [[../_index]] — raiz
- [[../00-meta/_moc]] — STATE/AUDIT/CHARTER que agentes consomem no bootstrap
- [[../40-templates/_moc]] — templates do pipeline e dos papéis
- [[../10-knowledge/methodology/graph-rag]] — como retrieval funciona pros agentes
