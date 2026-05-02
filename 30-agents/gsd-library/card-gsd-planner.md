---
title: gsd-planner
type: agent-contract
tags: [agent, gsd, planning, external-reference]
source: https://github.com/gsd-build/get-shit-done
source_raw: 60-sources/get-shit-done/raw/agents/gsd-planner.md
source_date: 2026-04-22
created: 2026-04-24
updated: 2026-04-24
aliases: [planner]
---

# gsd-planner

**Família**: [[family-planning]]. **Pipeline (nosso)**: Planner.

## Intenção

Transforma uma fase (goal de alto nível) em PLAN.md executáveis, com tarefas pequenas, grafo de dependências, ondas paralelas e critérios `must_haves` derivados por *goal-backward*. O plano É o prompt — executor aplica sem reinterpretar.

## Entrada

- `ROADMAP.md` da fase (goal + requirement IDs).
- `CONTEXT.md` com decisões travadas (D-01…D-NN), ideias diferidas e áreas de discretion.
- `RESEARCH.md` / `DISCOVERY.md` quando existem (standard stack, responsibility map, ASVS/STRIDE inputs).
- `STATE.md` + digest de fases anteriores; retrospectivas; intel (`graph.json`, PATTERNS.md).

## Saída

- Um ou mais `PLAN.md` em `.planning/phases/<fase>/` com frontmatter (`wave`, `depends_on`, `files_modified`, `autonomous`, `requirements`, `must_haves`, `user_setup` opcional).
- Atualização da `ROADMAP.md` (goal consolidado, contagem e lista de plans).
- Retorno estruturado ao orchestrator (`## PLANNING COMPLETE` com tabela de ondas).

## Ferramentas declaradas

Read, Write, Bash, Glob, Grep, WebFetch, `mcp__context7__*`.

## Padrão-chave replicável

- **Goal-backward**: não lista tarefas — extrai *truths* observáveis pelo usuário, depois artifacts, depois wiring, depois key_links (pontos de quebra cascata).
- **Fidelidade de decisão**: cada D-XX precisa virar task; qualquer simplificação ("v1", "static for now") é proibida — se não cabe, retornar `## PHASE SPLIT RECOMMENDED`. Planner não tem autoridade para julgar algo "difícil demais".

## Fonte

- Raw: [[../../60-sources/get-shit-done/raw/agents/gsd-planner]]
- Repo: `gsd-build/get-shit-done`

## Links

- [[family-planning]]
- [[_moc]]
- [[pipeline-mapping]]
- [[../_moc]]
