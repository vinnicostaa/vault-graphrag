---
title: gsd-codebase-mapper
type: agent-contract
tags: [agent, gsd, research, external-reference]
source: https://github.com/gsd-build/get-shit-done
source_raw: 60-sources/get-shit-done/raw/agents/gsd-codebase-mapper.md
source_date: 2026-04-22
created: 2026-04-24
updated: 2026-04-24
aliases: [codebase-mapper]
---

# gsd-codebase-mapper

**Família**: [[family-research]]. **Pipeline (nosso)**: Planner.

## Intenção

Explora o codebase com foco em UMA área (`tech` | `arch` | `quality` | `concerns`) e escreve documentos de análise direto em `.planning/codebase/`. Objetivo: reduzir carga de contexto do orquestrador — os docs são consumidos depois por `/gsd-plan-phase` e `/gsd-execute-phase`.

## Entrada

- Focus area (1 dos 4)
- Data de hoje (do prompt, nunca inferir)

## Saída

Documentos em UPPERCASE.md em `.planning/codebase/`, variando por focus:
- `tech` → `STACK.md`, `INTEGRATIONS.md`
- `arch` → `ARCHITECTURE.md`, `STRUCTURE.md`
- `quality` → `CONVENTIONS.md`, `TESTING.md`
- `concerns` → `CONCERNS.md`

Retorna apenas confirmação curta (~10 linhas), não o conteúdo.

## Ferramentas declaradas

`Read`, `Bash`, `Grep`, `Glob`, `Write`.

## Padrão-chave replicável

- **Prescriptive, não descriptive** — "Use camelCase for functions" é útil; "Some functions use camelCase" é inútil para o planner que escreverá código depois.
- **File paths em todo finding, em backticks** — `src/services/user.ts`, nunca "the user service". O consumidor navega direto para o código.
- **Forbidden files hardcoded** — `.env`, `*.pem`, `credentials.*`, `id_rsa*`, etc. são *listados* na existência mas **nunca lidos**. Escrito direto no prompt porque o output vai para git — leak = incidente de segurança.

## Fonte

- Raw: [[../../60-sources/get-shit-done/raw/agents/gsd-codebase-mapper]]
- Repo: `gsd-build/get-shit-done`

## Links

- [[family-research]]
- [[_moc]]
- [[pipeline-mapping]]
- [[../_moc]]
