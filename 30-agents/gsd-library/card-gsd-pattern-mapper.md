---
title: gsd-pattern-mapper
type: agent-contract
tags: [agent, gsd, research, external-reference]
source: https://github.com/gsd-build/get-shit-done
source_raw: 60-sources/get-shit-done/raw/agents/gsd-pattern-mapper.md
source_date: 2026-04-22
created: 2026-04-24
updated: 2026-04-24
aliases: [pattern-mapper]
---

# gsd-pattern-mapper

**Família**: [[family-research]]. **Pipeline (nosso)**: Planner.

## Intenção

Responde "que código existente os novos arquivos devem copiar padrões?" — classifica cada arquivo planejado por role (controller / service / middleware / etc.) e data flow (CRUD / streaming / event-driven / etc.), acha o análogo mais próximo no codebase e extrai excerpts concretos com linha.

## Entrada

- Phase directory, `CONTEXT.md`, `RESEARCH.md` (dos agentes upstream)
- Listagem explícita + implícita de arquivos a criar/modificar

## Saída

`PATTERNS.md` com:
- **File Classification** (tabela: arquivo → role → data flow → analog → match quality)
- **Pattern Assignments** por arquivo: imports, auth, core pattern, error handling — todos com `file:line` e código real
- **Shared Patterns** (cross-cutting: auth, error handling, validation)
- **No Analog Found** (planner usa RESEARCH.md nesses casos)

## Ferramentas declaradas

`Read`, `Bash`, `Glob`, `Grep`, `Write`. **Read-only no source** — só `PATTERNS.md` é escrito.

## Padrão-chave replicável

- **Ranking de analog em 4 níveis** — (1) same role + same data flow, (2) same role + diff flow, (3) diff role + same flow, (4) mais recentemente modificado. Força escolha sistemática, não "o que eu lembrei primeiro".
- **Stop at 3-5 strong analogs** — explicito no protocolo; evita gold-plating de pesquisa. Economia de contexto.
- **Concrete, not abstract** — "Copy auth pattern from `src/controllers/users.ts` lines 12-25" é utilizável pelo executor; "follow the auth pattern" não é.

## Fonte

- Raw: [[../../60-sources/get-shit-done/raw/agents/gsd-pattern-mapper]]
- Repo: `gsd-build/get-shit-done`

## Links

- [[family-research]]
- [[_moc]]
- [[pipeline-mapping]]
- [[../_moc]]
