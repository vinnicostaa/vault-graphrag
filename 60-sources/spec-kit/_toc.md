---
title: spec-kit — TOC curado
type: moc
tags: [sources, spec-kit, sdd, toc]
created: 2026-04-22
---

# spec-kit — Índice curado

Navegação por **tópico** (não por layout do repo). Material bruto vive em `raw/`. Destilação local em `_notes/` antes de promover pra `10-knowledge/methodology/` ou `40-templates/`.

## Leitura obrigatória (ordem)

1. [[raw/spec-driven|Spec-Driven Development]] — **manifesto do método**, base conceitual
2. [[raw/docs/index|docs/index]] — visão geral
3. [[raw/docs/quickstart|Quickstart]] — como aplicar hoje
4. [[raw/docs/installation|Installation]]

## Por tópico

### Método SDD (conceitos)

- [[raw/spec-driven|spec-driven.md]] — white paper do método
- [[raw/docs/index|docs/index]] — entry point
- [[raw/docs/reference/overview|reference/overview]] — referência conceitual
- [[raw/docs/upgrade|upgrade]] — migração entre versões

### Templates de artefato (CORE para 40-templates/)

- [[raw/templates/spec-template|spec-template]] — estrutura de **spec**
- [[raw/templates/plan-template|plan-template]] — estrutura de **plan**
- [[raw/templates/tasks-template|tasks-template]] — estrutura de **tasks**
- [[raw/templates/constitution-template|constitution-template]] — **constitution** (princípios do projeto)
- [[raw/templates/checklist-template|checklist-template]]

### Slash commands de referência (7 + 2 auxiliares)

- [[raw/templates/commands/specify|/specify]] — captura requisitos
- [[raw/templates/commands/clarify|/clarify]] — esclarece ambiguidades
- [[raw/templates/commands/plan|/plan]] — gera plano técnico
- [[raw/templates/commands/tasks|/tasks]] — decompõe em tasks
- [[raw/templates/commands/implement|/implement]] — executa
- [[raw/templates/commands/analyze|/analyze]] — análise
- [[raw/templates/commands/checklist|/checklist]]
- [[raw/templates/commands/constitution|/constitution]]
- [[raw/templates/commands/taskstoissues|/taskstoissues]] — converte tasks em GitHub issues

### Sistema de extensões

- [[raw/extensions/RFC-EXTENSION-SYSTEM|RFC do sistema de extensões]]
- [[raw/extensions/EXTENSION-API-REFERENCE|API reference]]
- [[raw/extensions/EXTENSION-DEVELOPMENT-GUIDE|Development guide]]
- [[raw/extensions/EXTENSION-PUBLISHING-GUIDE|Publishing guide]]
- [[raw/extensions/EXTENSION-USER-GUIDE|User guide]]
- Extensão git: [[raw/extensions/git/commands/speckit.git.initialize|git.initialize]], [[raw/extensions/git/commands/speckit.git.feature|git.feature]], [[raw/extensions/git/commands/speckit.git.commit|git.commit]], [[raw/extensions/git/commands/speckit.git.remote|git.remote]], [[raw/extensions/git/commands/speckit.git.validate|git.validate]]

### Presets (bundles pré-configurados)

- [[raw/presets/ARCHITECTURE|Architecture dos presets]]
- [[raw/presets/PUBLISHING|Como publicar]]
- Preset **lean** — só essencial: `raw/presets/lean/commands/speckit.{constitution,specify,plan,tasks,implement}`
- Preset **scaffold** — extensão de exemplo: `raw/presets/scaffold/`
- Preset **self-test** — validação: `raw/presets/self-test/`

### Workflows

- [[raw/workflows/ARCHITECTURE|Workflows architecture]]
- [[raw/workflows/PUBLISHING|Publishing workflow]]

### Newsletters (updates mensais)

- [[raw/newsletters/2026-February|Feb 2026]]
- [[raw/newsletters/2026-March|Mar 2026]]

### Referência técnica

- [[raw/docs/reference/core|core]]
- [[raw/docs/reference/workflows|workflows]]
- [[raw/docs/reference/extensions|extensions]]
- [[raw/docs/reference/integrations|integrations]]
- [[raw/docs/reference/presets|presets]]

### Testing

- [[raw/tests/hooks/TESTING|TESTING.md]]
- Hooks: [[raw/tests/hooks/spec|spec]], [[raw/tests/hooks/plan|plan]], [[raw/tests/hooks/tasks|tasks]]

## Prioridade de destilação

**P0 — destilar em `_notes/` já:**
1. `spec-driven.md` → conceitos pros 10-knowledge/methodology/spec-driven-development.md
2. `templates/spec-template` + `templates/plan-template` + `templates/tasks-template` → 40-templates/sdd/

**P1:**
3. Comparação ciclo spec-kit × GSD × SDD nosso
4. Sistema de extensões (se formos adotar)

## Vizinhos no grafo

- [[_index]] — metadata + como consultar
- [[../get-shit-done/_toc]] — fonte complementar (execução)
- [[../../10-knowledge/methodology/_moc]] — destino do destilado
- [[../../40-templates/_moc]] — destino dos templates
