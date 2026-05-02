---
title: gsd-eval-auditor
type: agent-contract
tags: [agent, gsd, audit, external-reference]
source: https://github.com/gsd-build/get-shit-done
source_raw: 60-sources/get-shit-done/raw/agents/gsd-eval-auditor.md
source_date: 2026-04-22
created: 2026-04-24
updated: 2026-04-24
aliases: [eval-auditor]
---

# gsd-eval-auditor

**Família**: [[family-audit]]. **Pipeline (nosso)**: Reviewer (específico para sistemas AI/LLM).

## Intenção

Auditoria retroativa de cobertura de avaliação em fases AI: a estratégia de avaliação prometida no AI-SPEC.md foi realmente implementada? Pontua cada dimensão (COVERED/PARTIAL/MISSING) + 5 componentes de infra, produz score ponderado e veredicto sobre prontidão para produção.

## Entrada

- AI-SPEC.md (Seções 5/6/7: eval strategy, guardrails, monitoring).
- SUMMARY.md de cada plan.
- Codebase scan: arquivos de teste/eval, tracing (langfuse/arize/phoenix), libs de eval (ragas, promptfoo), guardrails, datasets de referência.

## Saída

- `.planning/phases/<fase>/<phase>-EVAL-REVIEW.md` com:
  - Tabela dimensão → status + measurement + finding.
  - Infra audit (tooling, dataset, CI/CD, guardrails, tracing).
  - Scores: coverage 60% + infra 40% → overall.
  - Verdict: PRODUCTION READY (80+) / NEEDS WORK (60-79) / SIGNIFICANT GAPS (40-59) / NOT IMPLEMENTED (<40).

## Ferramentas declaradas

Read, Write, Bash, Grep, Glob.

## Padrão-chave replicável

- **PARTIAL é suspeito**: reviewer soft marca PARTIAL para "amaciar" — a regra é quantificar o gap. Se uma dimensão crítica tem teste existente mas não cobre os rubrics, é MISSING até o gap ser fechado.
- **Infraestrutura checada pelo uso real**: lib instalada ≠ usada. O auditor grep por *chamadas* das libs, não só por presença em `package.json`; guardrail stubado no path de request conta como MISSING.

## Fonte

- Raw: [[../../60-sources/get-shit-done/raw/agents/gsd-eval-auditor]]
- Repo: `gsd-build/get-shit-done`

## Links

- [[family-audit]]
- [[_moc]]
- [[pipeline-mapping]]
- [[../_moc]]
