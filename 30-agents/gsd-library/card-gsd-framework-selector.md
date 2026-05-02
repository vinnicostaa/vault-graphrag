---
title: gsd-framework-selector
type: agent-contract
tags: [agent, gsd, planning, external-reference]
source: https://github.com/gsd-build/get-shit-done
source_raw: 60-sources/get-shit-done/raw/agents/gsd-framework-selector.md
source_date: 2026-04-22
created: 2026-04-24
updated: 2026-04-24
aliases: [framework-selector]
---

# gsd-framework-selector

**Família**: [[family-planning]]. **Pipeline (nosso)**: Planner (decisão arquitetural de stack AI/LLM).

## Intenção

Responde "qual framework AI/LLM cabe neste projeto?" com uma entrevista curta (≤6 perguntas), scoring ponderado e filtro por hard constraints. Produz recomendação primária + alternativa ranqueadas, pronta para virar decisão registrada.

## Entrada

- Sinais do codebase (scan `package.json`, `pyproject.toml`, `requirements.txt` — libs AI já presentes).
- Respostas da entrevista: tipo de sistema, provider de modelo, estágio/time, linguagem, prioridade, constraints.
- `CONTEXT.md` upstream (se já existe, pula perguntas redundantes).
- `ai-frameworks.md` (matriz de decisão de referência — required reading).

## Saída

- Bloco `FRAMEWORK_RECOMMENDATION` estruturado: primary + rationale, alternative + reason, system_type, model_provider, eval_concerns, hard_constraints, existing_ecosystem.
- Display humano ao usuário com pick primário e alternativa.

## Ferramentas declaradas

Read, Bash, Grep, Glob, WebSearch, AskUserQuestion.

## Padrão-chave replicável

- **Hard constraint primeiro, score depois**: elimina frameworks incompatíveis com restrições declaradas (ex: "no vendor lock-in") antes de pontuar os sobreviventes — evita recomendar algo que seria rejeitado.
- **Codebase como evidência**: scan prévio das libs já instaladas reduz perguntas e impede recomendar framework que o time já abandonou.

## Fonte

- Raw: [[../../60-sources/get-shit-done/raw/agents/gsd-framework-selector]]
- Repo: `gsd-build/get-shit-done`

## Links

- [[family-planning]]
- [[_moc]]
- [[pipeline-mapping]]
- [[../_moc]]
