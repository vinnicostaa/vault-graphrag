---
title: gsd-user-profiler
type: agent-contract
tags: [agent, gsd, research, external-reference]
source: https://github.com/gsd-build/get-shit-done
source_raw: 60-sources/get-shit-done/raw/agents/gsd-user-profiler.md
source_date: 2026-04-22
created: 2026-04-24
updated: 2026-04-24
aliases: [user-profiler]
---

# gsd-user-profiler

**Família**: [[family-research]]. **Pipeline (nosso)**: Support.

## Intenção

Analisa mensagens de sessões passadas de um dev e infere perfil comportamental em 8 dimensões (expertise, style, tolerância a risco, etc.) — com rating, confidence level e evidence quotes. Output é uma *instruction doc* consumível por Claude para ajustar comportamento ao usuário.

## Entrada

- JSONL de mensagens de sessões (filtradas para user-only, truncadas a 500 chars, amostradas proporcionalmente por projeto, com recency weighting)
- Volume típico: 100-150 mensagens

## Saída

JSON estruturado dentro de `<analysis>` tags, com 8 dimensões (schema no reference doc `user-profiling.md`). Cada dimensão tem rating, confidence (HIGH/MEDIUM/LOW/UNSCORED), evidence count, quotes e **claude_instruction** (diretiva imperativa).

## Ferramentas declaradas

`Read`. Apenas — intencionalmente isolado, sem side effects.

## Padrão-chave replicável

- **Rubrica externa ao agent** — toda regra de scoring (thresholds, signal patterns, sensitive filters) vive em `~/.claude/get-shit-done/references/user-profiling.md`. O agent lê e aplica; proíbido inventar dimensões. Separa policy de execução.
- **Output é imperativo, não descritivo** — `claude_instruction` obriga formato "Provide concise explanations with code" em vez de "You tend to prefer brief explanations". O output é instrução direta para o próximo prompt.
- **Filtro de sensíveis em 2 passes** — sampling filtra uma vez, o profiler re-checa quotes selecionadas por `sk-`, `Bearer`, `password`, paths com usernames. Registra exclusões em `sensitive_excluded` para auditoria.

## Fonte

- Raw: [[../../60-sources/get-shit-done/raw/agents/gsd-user-profiler]]
- Repo: `gsd-build/get-shit-done`

## Links

- [[family-research]]
- [[_moc]]
- [[pipeline-mapping]]
- [[../_moc]]
