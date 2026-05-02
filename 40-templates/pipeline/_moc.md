---
title: MOC — Pipeline templates
type: moc
tags: [moc, templates, pipeline, operational]
created: 2026-04-24
updated: 2026-04-24
aliases: [Pipeline Templates]
---

# Pipeline templates

Templates replicáveis para **artefatos operacionais** do pipeline multi-agente (Planner → Worker → Reviewer → Orchestrator). Criados em D-vault-015.a (Fase 1 — Planner+Worker básico).

> **Distinção vs `30-agents/<role>/`**: esta pasta contém **contratos reutilizáveis** (shape + frontmatter). Os **artefatos reais** (plans, outputs, reviews, verdicts, logs) vivem em `30-agents/<role>/<sub>/<task-id>.md` ao longo da execução. Separação "spec vs estado" — ver ajuste #4 do dual-check D-vault-015.

## Templates ativos (Fase 1)

- [[plan-template]] — canônico para `30-agents/planner/plans/<task-id>.md`
- [[output-template]] — canônico para `30-agents/worker/outputs/<task-id>.md`

## Templates a adicionar (Fases 2-3, backlog)

### Fase 2 (D-vault-015.b — pendente)
- `review-A-template.md` — canônico para `30-agents/reviewer/reviews/<task-id>-A.md`
- `review-B-template.md` — análogo para variant B
- `verdict-template.md` — consolidação do orquestrador em `30-agents/reviewer/verdicts/<task-id>.md`

### Fase 3 (D-vault-015.c — pendente)
- `log-template.md` — canônico para `30-agents/orchestrator/logs/<yyyy-ww>/<task-id>.md`
- `skill-executor-template.md`, `skill-dual-validate-template.md`, `skill-sprint-auditor-template.md` — specs neutras para copiar em `~/.claude/skills/` (se preciso; instâncias MS já servem de base)

## Convenção de uso

1. **Operador ou skill `/task-executor`** copia template para `30-agents/<role>/<sub>/<task-id>.md`.
2. **Preenche frontmatter** — `task_id`, `phase`, `status`, `created`, `updated`.
3. **Preenche corpo** seguindo seções canônicas.
4. **Escreve ponteiro** em `_inbox/<next-role>/<task-id>.md` (frontmatter mínimo + link para artefato oficial — contrato ponteiro ver [[../../30-agents/_inbox/_moc]]).

## Ordem canônica de handoff (ajuste #9 do dual-check)

```
1. Write <role>/<sub>/<task-id>.md     (estado oficial — source of truth)
2. Write _inbox/<next-role>/<task-id>.md  (ponteiro — só frontmatter + link)
```

**Nunca invertida**. Se operação é interrompida após passo 1, próximo role não recebe trigger (correto). Se invertida e interrompida após passo 2, next-role consome ponteiro sem artefato oficial (quebra pipeline).

## Vizinhos

- [[../_moc]]
- [[../../30-agents/_moc]]
- [[../../30-agents/planner/_moc]] · [[../../30-agents/worker/_moc]] · [[../../30-agents/reviewer/_moc]] · [[../../30-agents/orchestrator/_moc]]
- [[../../30-agents/_inbox/_moc]] — contrato ponteiro
- [[../../30-agents/playbooks/plan-and-execute]]
- [[../../30-agents/playbooks/dual-check]]
- [[../../00-meta/STATE]] — D-vault-015.a, .b (pendente), .c (pendente)
