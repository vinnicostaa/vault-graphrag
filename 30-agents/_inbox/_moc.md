---
title: MOC — _inbox (filas como ponteiros)
type: moc
tags: [moc, agents, inbox, queue, pipeline]
created: 2026-04-24
updated: 2026-04-24
---

# _inbox — filas de tasks como ponteiros (Fase 1 instrumentada)

Pasta operacional do pipeline multi-agente. **Contrato redefinido em D-vault-015.a** (ajuste #6 do dual-check): `_inbox/` é **fila de ponteiros**, nunca duplicação do estado oficial.

## Contrato canônico

- **`<role>/<sub>/<task-id>.md`** = **estado oficial** (source of truth). Contém conteúdo completo. Paths: `planner/plans/`, `worker/outputs/`, `reviewer/reviews/`, `orchestrator/logs/`.
- **`_inbox/<next-role>/<task-id>.md`** = **ponteiro/fila** apenas. Frontmatter mínimo + 1 linha de corpo com link Obsidian. **Zero duplicação de conteúdo**.

## Frontmatter canônico do ponteiro

```yaml
---
title: "Task-<N> para <next-role>"
type: note
tags: [pipeline-artifact, pointer]
task_id: <N>
from_role: <planner | worker | reviewer>
to_role: <worker | reviewer | orchestrator>
enqueued_at: <ISO datetime>
pointer_to: "[[30-agents/<from-role>/<sub>/<N>]]"
status: pending
---
```

Corpo:

```markdown
# Task-<N> pendente para <next-role>

Estado oficial: [[30-agents/<from-role>/<sub>/<N>]]

Consumir: ler o link acima, processar, mover este ponteiro para `consumed/` ou atualizar `status: consumed`.
```

## Sub-pastas operacionais

- `planner/<task-id>.md` — briefing pendente para Planner (origem: operador humano)
- `worker/<task-id>.md` — plan pendente para Worker (origem: Planner)
- `reviewer/<task-id>.md` — output pendente para Reviewer (origem: Worker) — Fase 2 pendente

Cada sub-pasta pode ter `consumed/` onde ponteiros consumidos são movidos (ou marcados via frontmatter `status: consumed`).

## Regras duras

1. **Zero duplicação de conteúdo**: ponteiro nunca tem o plan/output/review inteiro. Apenas frontmatter + link.
2. **Ordem canônica de enqueue** (ajuste #9): escrever estado oficial primeiro, ponteiro depois.
3. **Idempotência de consume**: se `status: consumed` e próxima sessão tenta consumir de novo, detectar e pular.
4. **Rotação**: ponteiros consumidos > 7 dias podem ser arquivados (`90-archive/pipeline/inbox/<yyyy-ww>/`).

## Fluxo canônico (Fase 1: Planner → Worker)

```
1. Operador escreve briefing em _inbox/planner/<N>.md (ponteiro pra briefing oficial)
   OU direto em planner/briefings/<N>.md + _inbox/planner/<N>.md como ponteiro
2. Planner consome: lê inbox → resolve pointer → processa briefing
3. Planner escreve planner/plans/<N>.md (OFICIAL)
4. Planner escreve _inbox/worker/<N>.md (PONTEIRO pro plan)
5. Planner marca _inbox/planner/<N>.md como consumed
6. Worker consome: lê _inbox/worker/<N>.md → resolve pointer → lê plan
7. Worker escreve worker/outputs/<N>.md (OFICIAL)
8. Worker marca _inbox/worker/<N>.md consumed + (Fase 2) escreve _inbox/reviewer/<N>.md
```

## Status Fase 1

🟢 **Instrumentada para Planner → Worker**. Fase 2 (adicionar Reviewer) pendente como D-vault-015.b.

## Vizinhos

- [[../_moc]]
- [[../planner/_moc]] · [[../worker/_moc]] · [[../reviewer/_moc]] · [[../orchestrator/_moc]]
- [[../../40-templates/pipeline/_moc]]
- [[../../00-meta/STATE]] — D-vault-015.a
- [[../../00-meta/AUDIT]]
