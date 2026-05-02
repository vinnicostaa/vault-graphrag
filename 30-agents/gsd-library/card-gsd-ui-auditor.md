---
title: gsd-ui-auditor
type: agent-contract
tags: [agent, gsd, audit, external-reference]
source: https://github.com/gsd-build/get-shit-done
source_raw: 60-sources/get-shit-done/raw/agents/gsd-ui-auditor.md
source_date: 2026-04-22
created: 2026-04-24
updated: 2026-04-24
aliases: [ui-auditor]
---

# gsd-ui-auditor

**Família**: [[family-audit]]. **Pipeline (nosso)**: Reviewer (dimensão visual/UX).

## Intenção

Auditoria visual retroativa do frontend implementado contra o contrato declarado em UI-SPEC.md (ou contra os 6 padrões abstratos se o spec não existe). Pontua cada pilar 1–4 e produz UI-REVIEW.md com top 3 fixes prioritários. Captura screenshots via Playwright-MCP (preferido) ou CLI quando há dev server; senão, auditoria é code-only.

## Entrada

- UI-SPEC.md (contrato: spacing scale, typography, colors 60/30/10, copywriting, registry safety).
- SUMMARY.md + PLAN.md da fase.
- Código-fonte frontend (`.tsx`/`.jsx`/`.css`/`.scss`).
- Dev server opcional (localhost:3000/5173/8080) para screenshots.

## Saída

- `.planning/phases/<fase>/<phase>-UI-REVIEW.md` com:
  - Pillar scores (1-4 em Copywriting / Visuals / Color / Typography / Spacing / Experience Design).
  - Overall score `N/24`.
  - Top 3 priority fixes (issue específico + impacto no usuário + fix concreto).
  - Findings detalhados com `file:line`.
  - Registry Safety section se shadcn + third-party (flags de `fetch`/`eval`/`process.env`).
- `.gitignore` em `.planning/ui-reviews/` (gate obrigatório antes de screenshot).

## Ferramentas declaradas

Read, Write, Bash, Grep, Glob.

## Padrão-chave replicável

- **Gitignore gate antes de capturar**: `.planning/ui-reviews/.gitignore` é criado *antes* de qualquer screenshot — binários nunca entram no histórico, mesmo se o usuário fizer `git add .` antes do cleanup.
- **Evidence-based por pilar**: cada score cita classes Tailwind contadas, distribuição de font sizes, hardcoded hex colors — não é opinião. "Ajuste de classes" sobre `text-primary` usado em 12 elementos distintos é evidência; "cores parecem ruins" não é.

## Fonte

- Raw: [[../../60-sources/get-shit-done/raw/agents/gsd-ui-auditor]]
- Repo: `gsd-build/get-shit-done`

## Links

- [[family-audit]]
- [[_moc]]
- [[pipeline-mapping]]
- [[../_moc]]
