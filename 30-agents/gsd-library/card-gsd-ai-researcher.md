---
title: gsd-ai-researcher
type: agent-contract
tags: [agent, gsd, research, external-reference]
source: https://github.com/gsd-build/get-shit-done
source_raw: 60-sources/get-shit-done/raw/agents/gsd-ai-researcher.md
source_date: 2026-04-22
created: 2026-04-24
updated: 2026-04-24
aliases: [ai-researcher]
---

# gsd-ai-researcher

**Família**: [[family-research]]. **Pipeline (nosso)**: Planner.

## Intenção

Dado um framework de AI já escolhido (CrewAI, LangGraph, Claude Agent SDK, etc.), destila docs oficiais em guia implementation-ready: quick reference, padrão central, pitfalls e best practices de AI systems. Escreve seções 3, 4 e 4b de `AI-SPEC.md`.

## Entrada

- `framework` (nome + versão), `system_type` (RAG / Multi-Agent / Conversational / Extraction / etc.)
- `model_provider` (OpenAI / Anthropic / model-agnostic)
- Paths para `AI-SPEC.md`, `CONTEXT.md`, contexto da fase

## Saída

Três seções de `AI-SPEC.md`:
- **3. Framework Quick Reference** — install, imports, entry point runnable, abstrações, pitfalls, folder structure, sources
- **4. Implementation Guidance** — modelo específico, padrão central em código, tool config, context window
- **4b. AI Systems Best Practices** — Pydantic structured outputs, async-first, prompt discipline, context management, cost/latency budget

## Ferramentas declaradas

`Read`, `Write`, `Bash`, `Grep`, `Glob`, `WebFetch`, `WebSearch`, `mcp__context7__*`.

## Padrão-chave replicável

- **Depth over breadth no fetch de docs** — 2 a 4 páginas, priorizando quickstart + página do `system_type` + pitfalls (preferindo GitHub Issues a docs oficiais quando se trata de armadilhas).
- **Section 4b é framework-específica, não boilerplate** — exemplo Pydantic é escrito *para* este framework + system_type, não um genérico `class Foo(BaseModel)`.
- **Nunca alucinar métodos** — se não confirmou na doc, anota "verify in docs" em vez de inventar.

## Fonte

- Raw: [[../../60-sources/get-shit-done/raw/agents/gsd-ai-researcher]]
- Repo: `gsd-build/get-shit-done`

## Links

- [[family-research]]
- [[_moc]]
- [[pipeline-mapping]]
- [[../_moc]]
