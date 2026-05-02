---
title: gsd-executor
type: agent-contract
tags: [agent, gsd, executor, external-reference]
source: https://github.com/gsd-build/get-shit-done
source_raw: 60-sources/get-shit-done/raw/agents/gsd-executor.md
source_date: 2026-04-22
created: 2026-04-24
updated: 2026-04-24
aliases: [executor]
---

# gsd-executor

**Família**: [[family-executor]]. **Pipeline (nosso)**: Worker.

## Intenção

Executa um PLAN.md completo: aplica cada task atomicamente, commita por task, lida com desvios usando regras hierárquicas, pausa em checkpoints, produz SUMMARY.md e atualiza STATE/ROADMAP. O executor **não reinterpreta** — o plano já é o prompt.

## Entrada

- PLAN.md da fase (frontmatter + tasks + `must_haves`).
- `CONTEXT.md` (decisões travadas honradas na execução).
- `CLAUDE.md` do projeto (hard constraints — precedem instruções do plano).
- Estado anterior: `STATE.md`, commits já existentes (modo continuation).

## Saída

- Commits atômicos por task (`feat/fix/test/refactor/…({phase}-{plan}): …`).
- `.planning/phases/<fase>/<phase>-<plan>-SUMMARY.md` com deviations, auth gates, stubs, threat flags, self-check.
- Updates em `STATE.md` (decisões, métricas, session) e `ROADMAP.md` (progresso do plan).
- Retorno `## PLAN COMPLETE` ou `## CHECKPOINT REACHED`.

## Ferramentas declaradas

Read, Write, Edit, Bash, Grep, Glob, `mcp__context7__*`.

## Padrão-chave replicável

- **Regras de desvio hierárquicas**: Rule 1 (bug) / Rule 2 (missing critical — security, validation) / Rule 3 (blocker) são auto-fix sem pedir; Rule 4 (arquitetural) sempre pausa em checkpoint. Evita tanto paralisia quanto overreach.
- **Analysis paralysis guard**: se fizer 5+ Read/Grep seguidos sem Edit/Write/Bash, obrigado a declarar em 1 frase por quê não escreveu nada — sintoma de travamento, não de cautela.
- **Destructive git prohibition**: `git clean` proibido em worktree (destrói trabalho de waves anteriores); rollback é sempre `git checkout -- {file}` pontual.

## Fonte

- Raw: [[../../60-sources/get-shit-done/raw/agents/gsd-executor]]
- Repo: `gsd-build/get-shit-done`

## Links

- [[family-executor]]
- [[_moc]]
- [[pipeline-mapping]]
- [[../_moc]]
