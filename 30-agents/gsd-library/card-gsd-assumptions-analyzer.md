---
title: gsd-assumptions-analyzer
type: agent-contract
tags: [agent, gsd, research, external-reference]
source: https://github.com/gsd-build/get-shit-done
source_raw: 60-sources/get-shit-done/raw/agents/gsd-assumptions-analyzer.md
source_date: 2026-04-22
created: 2026-04-24
updated: 2026-04-24
aliases: [assumptions-analyzer]
---

# gsd-assumptions-analyzer

**Família**: [[family-research]]. **Pipeline (nosso)**: Planner.

## Intenção

Lê profundo o codebase de UMA fase e produz assumptions estruturadas com evidência (file paths) e nível de confiança. Serve à etapa discuss-phase — pré-planejamento — para forçar explicitação de decisões tácitas que "estavam no ar".

## Entrada

- Fase (número + nome + goal do ROADMAP)
- Decisões já travadas de fases anteriores
- Scout results (arquivos relevantes já pré-encontrados)
- Tier de calibração

## Saída

Seção `## Assumptions` com itens no formato:
```
- Assumption: <decisão>
  - Why this way: <evidência com file paths>
  - If wrong: <consequência concreta>
  - Confidence: Confident | Likely | Unclear
```
Mais uma seção `## Needs External Research` para gaps que o codebase sozinho não resolve.

## Ferramentas declaradas

`Read`, `Bash`, `Grep`, `Glob`. **Sem web, sem Context7** — intencionalmente isolado ao codebase.

## Padrão-chave replicável

- **Toda assumption cita file path como evidência** — sem referência a código específico, a assumption é descartada (disciplina anti-alucinação).
- **"If wrong" é concreto, não vago** — "causa bug no login" ≠ "could cause issues". Força accountability.
- **Flag explícito quando o codebase não basta** — em vez de especular, escreve em `Needs External Research` e entrega para o advisor/phase-researcher resolver.

## Fonte

- Raw: [[../../60-sources/get-shit-done/raw/agents/gsd-assumptions-analyzer]]
- Repo: `gsd-build/get-shit-done`

## Links

- [[family-research]]
- [[_moc]]
- [[pipeline-mapping]]
- [[../_moc]]
