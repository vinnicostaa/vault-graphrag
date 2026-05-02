---
title: MOC — 60-sources
type: moc
tags: [moc, sources, ingestion]
created: 2026-04-22
updated: 2026-04-24
---

# 60-sources — Matéria-prima externa

Sites baixados, repos clonados, PDFs, cursos. **Não linkar diretamente nas outras áreas** — o que importa é o destilado em `_notes/` local de cada fonte e, depois, promovido pra `10-knowledge/` / `20-stacks/` / `30-agents/` / `40-templates/`.

## Estrutura canônica por fonte

Reorg 2026-04-22 padronizou **todas as 5 fontes** nesta forma:

```
60-sources/<fonte>/
├── _index.md     # o que é a fonte + como consultar
├── _toc.md       # índice curado por tópico ★ (entrada principal pro agente)
├── raw/          # material bruto (EN-only, ≤ 6 níveis a partir de 60-sources/)
└── _notes/       # staging pre-promoção (pt-BR, destilados atômicos)
    └── _moc.md
```

Além disso:
- `_ingested/` (esta pasta) — marcadores de status de destilação por fonte.
- `90-archive/2026-04-22-sources-raw/` — ruído admin + i18n não-EN arquivados nesta reorg.

## Status das fontes (atualizado 2026-04-24 pós-sessão Graph-RAG)

| Fonte | Raw (MDs) | Status destilação | Notas promovidas |
|---|---:|---|---|
| [[spec-kit/_index\|spec-kit]] + [[spec-kit/_toc]] | 79 | ✅ parcial (2026-04-24) | [[../10-knowledge/methodology/spec-driven-development]], [[../10-knowledge/methodology/kiro-vs-spec-kit]] v2.1 atualizada com 9 slash commands + 3 fases macro + taxonomia Fowler |
| [[clean-coder/_index\|clean-coder]] + [[clean-coder/_toc]] | 171 | ✅ parcial (2026-04-24) | Canonical Five: [[../10-knowledge/architecture/the-clean-architecture]], [[../10-knowledge/architecture/screaming-architecture]], [[../10-knowledge/architecture/a-little-architecture]], [[../10-knowledge/architecture/framework-bound]], [[../10-knowledge/architecture/no-db]] |
| [[get-shit-done/_index\|get-shit-done]] + [[get-shit-done/_toc]] | 346 | ✅ parcial (2026-04-24) | Biblioteca completa em [[../30-agents/gsd-library/_moc]]: 33 reference-cards + 6 famílias + `pipeline-mapping` + playbook `gsd-resync` |
| [[fernando-franco-valle/_index\|fernando-franco-valle]] + [[fernando-franco-valle/_toc]] | 71 | ⏳ 0/2 prioritárias (índice curatorial, baixo ROI de destilação atômica) | — |
| [[refactoring-guru/_index\|refactoring-guru]] + [[refactoring-guru/_toc]] | 376 | ✅ **22/22 GoF patterns destilados** (2026-04-24) + 3 notas de categoria | [[../10-knowledge/patterns/_moc]] com os 22 patterns + `creational-patterns`, `structural-patterns`, `behavioral-patterns` |

**Total ativo em 60-sources**: 1043 MDs úteis (vs 4869 originais — 78% era ruído/duplicação). **Total arquivado**: 3839 MDs em [[../90-archive/2026-04-22-sources-raw/_index|90-archive/2026-04-22-sources-raw/]].

**Pendências conhecidas (sessões futuras)**:
- `refactoring-guru/raw/smells/` — 23 code smells ainda não destilados em `10-knowledge/antipatterns/`.
- `refactoring-guru/raw/refactorings/` — 66 refactorings Fowler ainda não consolidados.
- `clean-coder` — posts de TDD, TPP, functional vs OO ainda não destilados.
- `spec-kit` — templates (spec, plan, tasks, constitution, checklist) ainda não replicados em `40-templates/sdd/`.
- `get-shit-done` — workflows (discuss/plan/execute/verify/ship) + references (gates, context-budget, agent-contracts) pendentes.

## Pipeline de ingestão (3 estágios)

```
raw/         →  _notes/       →   10-knowledge/ | 20-stacks/ | 30-agents/ | 40-templates/
(bruto)         (staging pt-BR)   (canônico)
```

1. **Consulta (retrieval-only)**: agente lê `_toc.md` pra achar o tópico, abre `raw/<arquivo>.md` pra referência. **Não linka raw nas outras áreas**.
2. **Destilação**: gera nota atômica em `_notes/<conceito>.md` em pt-BR com frontmatter `source: ../raw/<arquivo>.md`.
3. **Promoção**: quando amadurecida, **move** (não copia) pra destino final. Marca `_ingested/<fonte>.md` com pointer.

## Regras duras

- **EN é canônico** nas fontes externas (i18n arquivada na reorg 2026-04-22).
- **Fontes brutas** podem ficar gitignored no projeto; apenas `_index.md`, `_toc.md`, `_notes/` e `_ingested/` devem ser commitados. *(Hoje raw/ está versionado — avaliar se migra pra gitignore)*.
- **Mirror de site** (httrack, pandoc) não substitui consulta ao vivo pra material em mudança (docs, blogs).
- **Copyright**: citações curtas + atribuição. Nunca republicar conteúdo integral.
- **Máximo 3 níveis na navegação curada** (via `_toc.md`). Dentro de `raw/` a profundidade é tolerada, mas `_toc.md` sempre aponta direto.

## Vizinhos no grafo

- [[../_index]] — raiz
- [[_ingested/_moc]] — marcadores de ingestão por fonte
- [[../10-knowledge/_moc]] — destino canônico dos conceitos atemporais
- [[../20-stacks/_moc]] — destino dos exemplos específicos de stack
- [[../30-agents/_moc]] — destino dos patterns de agents/workflows
- [[../40-templates/_moc]] — destino dos templates de artefato
- [[../90-archive/2026-04-22-sources-raw/_index]] — material arquivado da reorg
- [[../00-meta/STATE]] — D-vault-001, 002, 004, 005 registram promoções desta sessão

## Histórico

- **2026-04-22** — Reorg canônica aplicada às 5 fontes: `_index + _toc + raw + _notes`. Arquivado 3839 MDs (i18n não-EN + admin repo + ruído mirror). Ver [[../90-archive/2026-04-22-sources-raw/_index]].
- **2026-04-24** — Sessão de expansão Graph-RAG: **62 promoções** de raw → canônico (22 GoF patterns + 3 categorias + 5 Canonical Five + 33 GSD cards + 6 famílias + 2 meta GSD + `gsd-resync` + `community-summary` playbook + atualização SDD). Ver [[../00-meta/STATE]] D-vault-001..005.
