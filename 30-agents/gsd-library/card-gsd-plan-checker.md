---
title: gsd-plan-checker
type: agent-contract
tags: [agent, gsd, planning, external-reference]
source: https://github.com/gsd-build/get-shit-done
source_raw: 60-sources/get-shit-done/raw/agents/gsd-plan-checker.md
source_date: 2026-04-22
created: 2026-04-24
updated: 2026-04-24
aliases: [plan-checker]
---

# gsd-plan-checker

**Família**: [[family-planning]]. **Pipeline (nosso)**: Reviewer (atuando antes da execução, sobre plano).

## Intenção

Verifica, antes de executar, se os PLAN.md **vão** entregar o goal da fase. Mentalidade adversarial: parte do princípio que o plano falha e busca evidências que o desqualifiquem. Implementa o padrão Revision Gate (loop limitado com escalação).

## Entrada

- Conjunto de PLAN.md da fase.
- `ROADMAP.md` (goal + requirement IDs).
- `CONTEXT.md` (decisões travadas), `RESEARCH.md` (responsibility map, open questions), `PATTERNS.md`, `VALIDATION.md`.
- `CLAUDE.md` do projeto (convenções proibidas/obrigatórias).

## Saída

- `## VERIFICATION PASSED` com tabela de cobertura, OU
- `## ISSUES FOUND` com lista de findings classificados (blocker/warning/info) por dimensão.

## Ferramentas declaradas

Read, Bash, Glob, Grep.

## Padrão-chave replicável

- **12 dimensões obrigatórias**: requirement coverage, task completeness, dependency correctness, key links, scope sanity, must_haves derivation, context compliance, scope reduction detection, architectural tier, nyquist, data contracts, CLAUDE.md compliance, research resolution, pattern compliance.
- **Severidade sempre explícita**: nenhum issue sem `blocker | warning | info`. Scope reduction (v1, stub, hardcoded) é sempre blocker — nunca warning.

## Fonte

- Raw: [[../../60-sources/get-shit-done/raw/agents/gsd-plan-checker]]
- Repo: `gsd-build/get-shit-done`

## Links

- [[family-planning]]
- [[_moc]]
- [[pipeline-mapping]]
- [[../_moc]]
