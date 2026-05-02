---
title: gsd-eval-planner
type: agent-contract
tags: [agent, gsd, audit, external-reference]
source: https://github.com/gsd-build/get-shit-done
source_raw: 60-sources/get-shit-done/raw/agents/gsd-eval-planner.md
source_date: 2026-04-22
created: 2026-04-24
updated: 2026-04-24
aliases: [eval-planner]
---

# gsd-eval-planner

**Família**: [[family-audit]]. **Pipeline (nosso)**: Planner (bias de Reviewer — cria contrato que será auditado).

## Intenção

Desenha a estratégia de avaliação de uma fase AI **antes** da implementação — responde "como saberemos que este sistema de LLM está funcionando?". Transforma ingredientes de rubric do domínio em critérios mensuráveis com ferramental específico, guardrails, dataset de referência e monitoring. Escreve as Seções 5–7 do AI-SPEC.md que o eval-auditor depois valida.

## Entrada

- `system_type` (RAG / Multi-Agent / Conversational / Extraction / Autonomous / Content / Code / Hybrid).
- `framework` e `model_provider` (do framework-selector).
- AI-SPEC.md (Seção 1: failure modes; Seção 1b: rubric ingredients do domain-researcher).
- CONTEXT.md, REQUIREMENTS.md.

## Saída

- AI-SPEC.md com:
  - Seção 5 (Evaluation Strategy): dimensões + rubrics PASS/FAIL + measurement (Code/LLM Judge/Human) + tooling + dataset spec + CI/CD command.
  - Seção 6 (Guardrails): online (real-time) vs offline flywheel.
  - Seção 7 (Production Monitoring): tracing + métricas + alert thresholds.

## Ferramentas declaradas

Read, Write, Bash, Grep, Glob, AskUserQuestion.

## Padrão-chave replicável

- **Rubric em linguagem do domínio, não genérica**: cada dimensão é "PASS: comportamento específico / FAIL: comportamento específico" — rótulos genéricos como "factual accuracy" não são critério, são categoria.
- **Detect antes de default**: scan por libs de eval já instaladas antes de recomendar tooling; opinionated defaults (Phoenix, RAGAS, Promptfoo, LangSmith) são fallback, não imposição.

## Fonte

- Raw: [[../../60-sources/get-shit-done/raw/agents/gsd-eval-planner]]
- Repo: `gsd-build/get-shit-done`

## Links

- [[family-audit]]
- [[_moc]]
- [[pipeline-mapping]]
- [[../_moc]]
