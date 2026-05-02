---
title: MOC — Planner (Fase 1 instrumentada)
type: moc
tags: [moc, agents, planner, pipeline]
created: 2026-04-24
updated: 2026-04-24
---

# Planner — pipeline operacional (Fase 1)

Pasta operacional do role **Planner**. Instrumentada em D-vault-015.a (Fase 1 — Planner + Worker básico).

## Papel

Agente que **decompõe briefing em plano executável**:

- Lê briefing em `30-agents/_inbox/planner/<task-id>.md` (ponteiro; estado oficial em `briefings/`)
- Consulta spec + STATE + AUDIT + steering relevantes do projeto
- Aplica research-loop ([[../playbooks/research-loop]]) — 5 fontes
- Produz **plan** em `plans/<task-id>.md` usando [[../../40-templates/pipeline/plan-template]]
- Escreve **ponteiro** em `_inbox/worker/<task-id>.md` (frontmatter + link)

## Sub-pastas operacionais

- `briefings/<task-id>.md` — briefing oficial (copiado de `_inbox/planner/` ao consumir)
- `plans/<task-id>.md` — **estado oficial do plan** (source of truth)

## Regras duras

1. **Um Planner ativo por task-id** (ajuste #8 do dual-check D-vault-015). Se 2 sessões concorrentes para o mesmo task-id: convenção quebrada; reportar em AUDIT.
2. **Ordem canônica de handoff** (ajuste #9):
   - (a) Escrever `plans/<task-id>.md` (estado oficial) primeiro.
   - (b) Escrever `_inbox/worker/<task-id>.md` (ponteiro) depois.
   - Nunca inverter.
3. **Path canônico obrigatório**. Artefato fora de `planner/plans/` **não é plan** — `/task-executor` aborta se não encontrar o path esperado (ajuste #7).
4. **Frontmatter canônico** conforme template `plan-template`. Faltando `task_id`/`role`/`phase` → refutado.

## Referência externa (biblioteca GSD)

[[../gsd-library/family-planning]] — 5 agents desta família (planner, plan-checker, roadmapper, framework-selector, intel-updater). Nosso Planner **adota a essência** (goal-backward, 2-3 tasks/plan, must_haves declarativos) sem forçar os 5 agents separados.

## Regra dura crítica — extensão reviewer

Plan define `<verify>` declarativo. Reviewer (Fase 2) valida contra esse contrato. Se plan fica ambíguo, dual-check amostral pode ser disparado na hora do plan (pre-execução) em vez de só no review pós-execução.

## Status

🟢 **Fase 1 instrumentada** (D-vault-015.a) — estrutura + templates prontos. Execução real depende de skill `/task-executor` do Claude Code que consuma esta estrutura.

## Vizinhos

- [[../_moc]] — pipeline
- [[../_inbox/_moc]] — fila de entrada (ponteiros)
- [[../worker/_moc]] — próxima etapa (recebe ponteiro → lê plan daqui)
- [[../gsd-library/family-planning]] — referência biblioteca GSD
- [[../playbooks/plan-and-execute]]
- [[../playbooks/research-loop]]
- [[../../40-templates/pipeline/plan-template]]
- [[../../40-templates/pipeline/_moc|pipeline templates]]
- [[../../00-meta/STATE]] — D-vault-015.a
