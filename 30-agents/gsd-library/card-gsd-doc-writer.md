---
title: gsd-doc-writer
type: agent-contract
tags: [agent, gsd, docs, external-reference]
source: https://github.com/gsd-build/get-shit-done
source_raw: 60-sources/get-shit-done/raw/agents/gsd-doc-writer.md
source_date: 2026-04-22
created: 2026-04-24
updated: 2026-04-24
aliases: [doc-writer]
---

# gsd-doc-writer

**Família**: [[family-docs]]. **Pipeline (nosso)**: Worker (específico de documentação).

## Intenção
Escrever e atualizar docs de projeto (README, ARCHITECTURE, GETTING-STARTED, DEVELOPMENT, TESTING, API, CONFIGURATION, DEPLOYMENT, CONTRIBUTING, per-package, custom) em 4 modos: `create | update | supplement | fix`. Explora o codebase antes de escrever; nunca fabrica paths, funções ou comandos.

## Entrada
- `<doc_assignment>` com `type`, `mode`, `project_context`, `existing_content`, `scope` (per_package/monorepo), `failures[]` (modo fix), `description`+`output_path` (modo custom)
- Projeto-skills em `.claude/skills/` ou `.agents/skills/` (só SKILL.md index, nunca AGENTS.md full)

## Saída
- Arquivo `.md` escrito via Write tool, com marker `<!-- generated-by: gsd-doc-writer -->` como primeira linha (exceto `supplement`)
- Markers `<!-- VERIFY: {claim} -->` em claims de infra não verificáveis do repo (URLs externas, dashboard links)
- Confirmação apenas (não retorna conteúdo da doc ao orchestrator)

## Ferramentas declaradas
`Read, Bash, Grep, Glob, Write`

## Padrão-chave replicável
1. **4 modos com disciplinas distintas**: `supplement` **nunca** modifica linha existente, só anexa; `fix` é **cirúrgico** (modifica só as linhas listadas em `failures[]`); `update` preserva prosa user-authored em seções ainda precisas; `create` escreve do zero.
2. **Templates como contrato discovery-first**: cada `<template_*>` lista "Required Sections" + "Content Discovery" (quais arquivos grep/read pra encontrar os fatos) + "Format Notes" — separa estrutura (fixa) de descoberta (adaptável ao projeto).
3. **Doc tooling adaptation sem mudar conteúdo**: Docusaurus/VitePress/MkDocs/Storybook mudam apenas `output_path` e frontmatter; seções/headings continuam iguais. Zero vazamento de concerns de apresentação para conteúdo.
4. **Zero GSD methodology leak**: docs geradas descrevem o **target project**, nunca mencionam `/gsd-` commands, PLAN.md ou fases. CHANGELOG.md é intocável (dono: `/gsd-ship`).

## Fonte
- Raw: [[../../60-sources/get-shit-done/raw/agents/gsd-doc-writer]]
- Repo: `gsd-build/get-shit-done`

## Links
- [[family-docs]]
- [[_moc]]
- [[pipeline-mapping]]
- [[../_moc]]
