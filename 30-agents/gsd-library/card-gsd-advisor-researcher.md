---
title: gsd-advisor-researcher
type: agent-contract
tags: [agent, gsd, research, external-reference]
source: https://github.com/gsd-build/get-shit-done
source_raw: 60-sources/get-shit-done/raw/agents/gsd-advisor-researcher.md
source_date: 2026-04-22
created: 2026-04-24
updated: 2026-04-24
aliases: [advisor-researcher]
---

# gsd-advisor-researcher

**Família**: [[family-research]]. **Pipeline (nosso)**: Planner.

## Intenção

Pesquisa UMA única área cinzenta (gray area) de decisão e entrega UMA tabela comparativa 5-colunas com rationale. Não fala com o usuário — devolve markdown estruturado para o orquestrador sintetizar.

## Entrada

- Nome e descrição da gray area
- Contexto da fase (do ROADMAP)
- Contexto breve do projeto
- Tier de calibração: `full_maturity` | `standard` | `minimal_decisive`

## Saída

Bloco markdown com tabela de 5 colunas (Option / Pros / Cons / Complexity / Recommendation) + parágrafo de rationale. Número de opções varia por tier (2 a 5).

## Ferramentas declaradas

`Read`, `Bash`, `Grep`, `Glob`, `WebSearch`, `WebFetch`, `mcp__context7__*`.

## Padrão-chave replicável

- **Recomendação condicional, nunca ranking** — formato obrigatório é `Rec if X` (ex: "Rec if mobile-first", "Rec if SEO matters"). Proíbe declarar vencedor único.
- **Complexity = impact surface + risk**, nunca estimativas de tempo. Ex: `3 files, new dep -- Risk: memory, scroll state`.
- **Um research por gray area** — o spawn é fan-out via `Task()`; cada advisor vive numa janela de contexto isolada, o orquestrador consolida.

## Fonte

- Raw: [[../../60-sources/get-shit-done/raw/agents/gsd-advisor-researcher]]
- Repo: `gsd-build/get-shit-done`

## Links

- [[family-research]]
- [[_moc]]
- [[pipeline-mapping]]
- [[../_moc]]
