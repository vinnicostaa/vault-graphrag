---
title: "content/ — ponteiro pós-sanitização 2026-04-22"
type: pointer
tags: [master-sindico, archive, pointer]
created: 2026-04-22
superseded: content/_MOC.md
---

# content/ — pasta removida em 2026-04-22

A pasta `content/` foi **desmontada** durante a sanitização de 2026-04-22. O conteúdo valioso foi promovido ao overlay canônico; o histórico foi preservado em `_archive/`.

Motivo: `content/` era um "vault Obsidian" paralelo criado por um agente-analista em sessão multi-agente anterior, apontando para um canônico externo (`backend/.kiro/`) que não existe neste vault. Isso fazia o agente se perder em três estruturas paralelas (`content/`, `specs/`, `04-requirements/`).

## Onde procurar o que estava em `content/`

### Promovido ao canônico

| Antes | Agora |
|---|---|
| `content/regras-de-negocio/regras-canonicas.md` | [[../01-domain/regras-canonicas]] |
| `content/regras-de-negocio/quebras-detectadas.md` | [[../11-audit/quebras-detectadas-2026-04-22]] |
| `content/sdd-gsd-aplicado.md` | [[../06-execution/sdd-gsd-aplicado]] — devolvido de `10-knowledge/methodology/sdd-gsd` (conteúdo era específico do projeto; genéricos destilados em [[../../../10-knowledge/methodology/sdd-maturity-checklist]] e [[../../../10-knowledge/methodology/sdd-adoption-gaps]]) |

### Arquivado (histórico preservado)

| Antes | Agora |
|---|---|
| `content/requirements.md` (204K monolítico) | `_archive/content-monolitico-2026-03/requirements.md` — recortado em `specs/requirements/` |
| `content/design.md` (84K monolítico) | `_archive/content-monolitico-2026-03/design.md` — recortado em `specs/design/` |
| `content/tasks.md` (68K monolítico) | `_archive/content-monolitico-2026-03/tasks.md` — recortado em `specs/tasks/` |
| `content/ARCHITECTURE_ANALYSIS_REPORT.md` | `_archive/content-monolitico-2026-03/architecture-analysis-report.md` |
| `content/excopo-tecnico-antigo.md` | `_archive/content-monolitico-2026-03/escopo-tecnico-antigo.md` |
| `content/main.go.md` | `_archive/content-monolitico-2026-03/main-go.md` |
| `content/_handoffs/*` | `_archive/handoffs/` |
| `content/contents/*` (23 jornadas/módulos) | `_archive/content-discovery-2026-03/` (nomes em kebab-case) |
| `content/.kiro/` (specs stale) | `_archive/content-kiro-stale/` |
| `content/.claude/settings.local_json.md` | `_archive/content-misc/` |
| `content/_MOC.md` | substituído por este arquivo |

## Log completo

Ver [[MOVE_LOG_2026-04-22]].

## Regra canônica derivada desta operação

A partir de agora, **novos artefatos de discovery/regra/quebra** vão em:

- Regras de negócio consolidadas → `01-domain/regras-canonicas.md` (append)
- Auditoria cross-doc → `11-audit/AUDIT.md` (A-###) ou arquivo dedicado datado
- Metodologia atemporal → `10-knowledge/methodology/`
- Discovery em andamento → `_archive/` só após virar canônico ou ser abandonado
