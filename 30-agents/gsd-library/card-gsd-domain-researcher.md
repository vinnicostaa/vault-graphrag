---
title: gsd-domain-researcher
type: agent-contract
tags: [agent, gsd, research, external-reference]
source: https://github.com/gsd-build/get-shit-done
source_raw: 60-sources/get-shit-done/raw/agents/gsd-domain-researcher.md
source_date: 2026-04-22
created: 2026-04-24
updated: 2026-04-24
aliases: [domain-researcher]
---

# gsd-domain-researcher

**Família**: [[family-research]]. **Pipeline (nosso)**: Planner.

## Intenção

Pesquisa o **domínio de negócio** (não o framework técnico) de um sistema de AI. Responde: "o que um especialista do domínio realmente avalia quando olha o output deste AI?" Produz ingredientes de rubrica em linguagem de prática, antes do eval-planner transformar em métricas.

## Entrada

- `system_type` (RAG / Multi-Agent / Conversational / etc.)
- Path para `AI-SPEC.md`, `CONTEXT.md`, `REQUIREMENTS.md`
- Nome e goal da fase

## Saída

Seção `## 1b. Domain Context` de `AI-SPEC.md`:
- Industry vertical, user population, stakes level
- 3-5 rubric ingredients (Dimension / Good / Bad / Stakes / Source)
- Known failure modes domain-specific
- Contexto regulatório (HIPAA, GDPR, FCA, ou "none identified")
- Papéis de domain experts para avaliação

## Ferramentas declaradas

`Read`, `Write`, `Bash`, `Grep`, `Glob`, `WebSearch`, `WebFetch`, `mcp__context7__*`.

## Padrão-chave replicável

- **Linguagem de prática, não de ML** — "Citation precision: response cites clause + section + jurisdiction" em vez de "accuracy" genérico. Dois especialistas do domínio deveriam concordar lendo o critério.
- **Failure modes domain-specific** — não repetir "hallucination" genérico; pesquisar postmortems reais da vertical.
- **Regulamentação só quando diretamente aplicável** — não lista todo o catálogo de compliance; menciona o que de fato morde neste deployment.

## Fonte

- Raw: [[../../60-sources/get-shit-done/raw/agents/gsd-domain-researcher]]
- Repo: `gsd-build/get-shit-done`

## Links

- [[family-research]]
- [[_moc]]
- [[pipeline-mapping]]
- [[../_moc]]
