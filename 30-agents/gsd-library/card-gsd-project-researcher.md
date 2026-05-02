---
title: gsd-project-researcher
type: agent-contract
tags: [agent, gsd, research, external-reference]
source: https://github.com/gsd-build/get-shit-done
source_raw: 60-sources/get-shit-done/raw/agents/gsd-project-researcher.md
source_date: 2026-04-22
created: 2026-04-24
updated: 2026-04-24
aliases: [project-researcher]
---

# gsd-project-researcher

**Família**: [[family-research]]. **Pipeline (nosso)**: Planner.

## Intenção

Mapeia o **ecossistema do domínio** (não uma fase específica) antes do roadmap ser criado. Serve `/gsd-new-project` e `/gsd-new-milestone` em 3 modos: ecosystem (default, "o que existe"), feasibility ("dá pra fazer?"), comparison ("A vs B vs C").

## Entrada

- Nome e descrição do projeto
- Modo de pesquisa (ecosystem / feasibility / comparison)
- Perguntas específicas do orquestrador

## Saída

Arquivos em `.planning/research/`:
- `SUMMARY.md` (sempre), `STACK.md` (sempre), `FEATURES.md` (sempre), `PITFALLS.md` (sempre)
- `ARCHITECTURE.md` (se patterns achados), `COMPARISON.md` (modo comparison), `FEASIBILITY.md` (modo feasibility)

**Não commita** — spawn em paralelo, orquestrador commita tudo depois.

## Ferramentas declaradas

`Read`, `Write`, `Bash`, `Grep`, `Glob`, `WebSearch`, `WebFetch`, `mcp__context7__*`, `mcp__firecrawl__*`, `mcp__exa__*`.

## Padrão-chave replicável

- **"Training data = hypothesis"** — disciplina explícita: o que Claude "sabe" é hipótese até verificado em fonte atual. Context7/docs oficiais > training data.
- **Modos como especialização** — o mesmo agente vira 3 agentes distintos via parâmetro; cada modo tem output template próprio. Reduz proliferação de agentes para a mesma função cognitiva.
- **Anti-features explícitos** — `FEATURES.md` lista **o que não construir** com rationale, não só table stakes + diferenciadores. Força disciplina de escopo cedo.

## Fonte

- Raw: [[../../60-sources/get-shit-done/raw/agents/gsd-project-researcher]]
- Repo: `gsd-build/get-shit-done`

## Links

- [[family-research]]
- [[_moc]]
- [[pipeline-mapping]]
- [[../_moc]]
