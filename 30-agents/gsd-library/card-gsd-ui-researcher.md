---
title: gsd-ui-researcher
type: agent-contract
tags: [agent, gsd, audit, external-reference]
source: https://github.com/gsd-build/get-shit-done
source_raw: 60-sources/get-shit-done/raw/agents/gsd-ui-researcher.md
source_date: 2026-04-22
created: 2026-04-24
updated: 2026-04-24
aliases: [ui-researcher]
---

# gsd-ui-researcher

**Família**: [[family-audit]]. **Pipeline (nosso)**: Support (produz contrato que será auditado por ui-checker e ui-auditor).

## Intenção

Responde "quais contratos visuais e de interação esta fase precisa?" e produz o UI-SPEC.md que planner/executor/ui-auditor consomem. Scouta codebase antes de perguntar (shadcn init? tokens Tailwind existentes? componentes?), pergunta só o que upstream (CONTEXT/RESEARCH/REQUIREMENTS) não respondeu. Prescritivo — "16px body, line-height 1.5" — não exploratório.

## Entrada

- CONTEXT.md (decisões travadas de UX).
- RESEARCH.md (standard stack, patterns).
- REQUIREMENTS.md (success criteria, requirements visuais).
- Scan codebase: `components.json`, `tailwind.config.*`, `postcss.config.*`, componentes existentes.

## Saída

- `.planning/phases/<fase>/<phase>-UI-SPEC.md` com status inicial `draft`:
  - Spacing scale (múltiplos de 4).
  - Typography (3-4 sizes, 2 weights max).
  - Color (60/30/10 + accent reserved-for list específica).
  - Copywriting contract (CTA, empty, error, destructive + confirmation).
  - Registry (shadcn official + third-party com Safety Gate timestamped).

## Ferramentas declaradas

Read, Write, Bash, Grep, Glob, WebSearch, WebFetch, `mcp__context7__*`, `mcp__firecrawl__*`, `mcp__exa__*`.

## Padrão-chave replicável

- **shadcn gate antes de contract questions**: se projeto é React/Next/Vite e não tem `components.json`, ofereça init; se já existe, lê `npx shadcn info` e pré-popula — pergunta só o que upstream não respondeu. Reduz drasticamente o ruído de entrevista.
- **Registry vetting é gate executável, não checklist**: third-party block declarado dispara `npx shadcn view {block} --registry {url}` com scan por `fetch(`, `eval(`, `process.env`, `Function(`. Se developer recusa a inspeção, bloco NÃO entra no UI-SPEC — evidência é contrato, não intenção.

## Fonte

- Raw: [[../../60-sources/get-shit-done/raw/agents/gsd-ui-researcher]]
- Repo: `gsd-build/get-shit-done`

## Links

- [[family-audit]]
- [[_moc]]
- [[pipeline-mapping]]
- [[../_moc]]
