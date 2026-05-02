---
title: gsd-ui-checker
type: agent-contract
tags: [agent, gsd, audit, external-reference]
source: https://github.com/gsd-build/get-shit-done
source_raw: 60-sources/get-shit-done/raw/agents/gsd-ui-checker.md
source_date: 2026-04-22
created: 2026-04-24
updated: 2026-04-24
aliases: [ui-checker]
---

# gsd-ui-checker

**Família**: [[family-audit]]. **Pipeline (nosso)**: Reviewer (valida UI-SPEC antes do planning começar).

## Intenção

Valida que UI-SPEC.md é completo, consistente e implementável **antes** do planner consumir. Análogo ao plan-checker (goal-backward), mas aplicado ao contrato visual. Read-only — nunca modifica o spec; emite verdict BLOCK/FLAG/PASS por dimensão.

## Entrada

- UI-SPEC.md (primary input — do ui-researcher).
- CONTEXT.md (decisões travadas de UX/UI), RESEARCH.md (standard stack).
- `./CLAUDE.md` e project skills.

## Saída

- Retorno estruturado: `## UI-SPEC VERIFIED` (APPROVED) ou `## ISSUES FOUND` (BLOCKED).
- Tabela por dimensão com verdict + nota.
- Lista explícita de blocking issues (com fix exato) e non-blocking recommendations.
- Se APPROVED: sinaliza para researcher fazer `status: approved` + `reviewed_at`.

## Ferramentas declaradas

Read, Bash, Glob, Grep.

## Padrão-chave replicável

- **BLOCK tem gatilhos objetivos, não subjetivos**: CTA genérico ("Submit"/"OK") = BLOCK; >4 font sizes = BLOCK; spacing não múltiplo de 4 = BLOCK; accent "para todos os elementos interativos" = BLOCK. Critérios mensuráveis em vez de "design feio".
- **Registry safety com timestamp de evidência**: para third-party registries (shadcn + magicui/etc.), "shadcn view + diff required" é *intenção*, não evidência. Só passa com `view passed — no flags — {date}` ou `developer-approved after view — {date}`.

## Fonte

- Raw: [[../../60-sources/get-shit-done/raw/agents/gsd-ui-checker]]
- Repo: `gsd-build/get-shit-done`

## Links

- [[family-audit]]
- [[_moc]]
- [[pipeline-mapping]]
- [[../_moc]]
