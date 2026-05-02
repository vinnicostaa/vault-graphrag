---
title: gsd-phase-researcher
type: agent-contract
tags: [agent, gsd, research, external-reference]
source: https://github.com/gsd-build/get-shit-done
source_raw: 60-sources/get-shit-done/raw/agents/gsd-phase-researcher.md
source_date: 2026-04-22
created: 2026-04-24
updated: 2026-04-24
aliases: [phase-researcher]
---

# gsd-phase-researcher

**Família**: [[family-research]]. **Pipeline (nosso)**: Planner.

## Intenção

Responde "o que eu preciso saber para PLANEJAR bem esta fase?" — pesquisa stack, patterns, pitfalls, testing e security antes do `gsd-planner` criar tasks. Produz `RESEARCH.md` que o planner consome como verdade constrangida.

## Entrada

- Número, nome, goal e requisitos (`REQ-IDs`) da fase
- `CONTEXT.md` (decisões travadas + áreas de discrição + deferred)
- `./CLAUDE.md` se existe (diretivas do projeto)
- Knowledge graph se `graph.json` existe

## Saída

`RESEARCH.md` com: User Constraints, Architectural Responsibility Map, Standard Stack (versões verificadas), Architecture Patterns, Don't Hand-Roll, Common Pitfalls, Runtime State Inventory (rename/refactor only), Environment Availability, Validation Architecture, Security Domain (ASVS), Sources, Assumptions Log.

## Ferramentas declaradas

`Read`, `Write`, `Bash`, `Grep`, `Glob`, `WebSearch`, `WebFetch`, `mcp__context7__*`, `mcp__firecrawl__*`, `mcp__exa__*`.

## Padrão-chave replicável

- **Claim provenance obrigatório** — toda afirmação factual tagueada com `[VERIFIED]` / `[CITED: url]` / `[ASSUMED]`. Assumptions vão para `Assumptions Log` para confirmação humana.
- **Hierarquia explícita de fontes** — Context7 > Exa > Firecrawl > official GitHub > Brave/Web (verificado) > Web (unverified). Confidence segue a fonte, não a confiança do autor.
- **Runtime State Inventory para refactor/rename** — 5 categorias obrigatórias (stored data, live service config, OS state, secrets/env, build artifacts). Força enxergar estado que `grep` não pega (ex: Chroma collection names, Windows Task Scheduler, pip egg-info).

## Fonte

- Raw: [[../../60-sources/get-shit-done/raw/agents/gsd-phase-researcher]]
- Repo: `gsd-build/get-shit-done`

## Links

- [[family-research]]
- [[_moc]]
- [[pipeline-mapping]]
- [[../_moc]]
