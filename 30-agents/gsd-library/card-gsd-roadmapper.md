---
title: gsd-roadmapper
type: agent-contract
tags: [agent, gsd, planning, external-reference]
source: https://github.com/gsd-build/get-shit-done
source_raw: 60-sources/get-shit-done/raw/agents/gsd-roadmapper.md
source_date: 2026-04-22
created: 2026-04-24
updated: 2026-04-24
aliases: [roadmapper]
---

# gsd-roadmapper

**Família**: [[family-planning]]. **Pipeline (nosso)**: Orchestrator (com viés de Planner — inicializa estrutura de entrega).

## Intenção

Transforma `PROJECT.md` + `REQUIREMENTS.md` (+ research opcional) em um `ROADMAP.md` com fases derivadas das próprias requirements (não de um template). Cobertura 100% dos requirement IDs é condição não-negociável — sem órfãos, sem duplicatas.

## Entrada

- `PROJECT.md` (core value, constraints).
- `REQUIREMENTS.md` v1 (requirement IDs agrupados por categoria).
- `research/SUMMARY.md` opcional.
- `config.json` (granularity: coarse/standard/fine).

## Saída

- `.planning/ROADMAP.md` com dois blocos obrigatórios (checklist `## Phases` + detalhes `## Phase Details`) e tabela de progresso.
- `.planning/STATE.md` inicial.
- Atualização da seção Traceability em `REQUIREMENTS.md` (mapa REQ → Phase).
- Retorno `## ROADMAP CREATED` ao orchestrator.

## Ferramentas declaradas

Read, Write, Bash, Glob, Grep.

## Padrão-chave replicável

- **Goal-backward no nível de fase**: para cada fase, listar 2–5 *observable truths* (verificáveis por humano usando a app), cruzar com requirement IDs e resolver gaps antes de finalizar.
- **Fatias verticais > camadas horizontais**: fase entrega *feature completa* (auth funcionando ponta a ponta), nunca "todos os models" isolados. Headers `### Phase X:` são parseados downstream — obrigatórios.

## Fonte

- Raw: [[../../60-sources/get-shit-done/raw/agents/gsd-roadmapper]]
- Repo: `gsd-build/get-shit-done`

## Links

- [[family-planning]]
- [[_moc]]
- [[pipeline-mapping]]
- [[../_moc]]
