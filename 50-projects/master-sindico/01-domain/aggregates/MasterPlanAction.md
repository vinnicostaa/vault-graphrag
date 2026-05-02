---
title: MasterPlanAction (aggregate stub)
type: aggregate
tags:
  - aggregate
  - master-sindico
  - stub
  - institutional
bc: institutional
status: stub
created: '2026-04-27'
updated: '2026-04-27'
---

# MasterPlanAction

> 🟡 **Stub** criado em 2026-04-27 para cobrir refs em ui-catalog/jornadas/. Conteúdo a destilar junto com a expansão completa de [[MasterPlan]].

Entity ou aggregate filho de [[MasterPlan]] — representa **uma ação concreta** dentro do Plano Diretor (obra, melhoria, manutenção, política). Cada item tem status, prazo, responsável e gera `TimelineEntry` ao mudar de fase.

**BC**: institutional.

**Invariantes esperadas**:
- Status segue state machine (proposta → aprovada → em_execucao → concluida / cancelada).
- Mudança de status só via `TimelineEntry` (R2: PD não atualizado manualmente).
- Categorias predefinidas (`obra_melhoria`, `manutencao`, `politica`, `seguranca`, `comunicacao`).

**Eventos esperados**: `MasterPlanActionCreated`, `MasterPlanActionStatusChanged`, `MasterPlanActionCompleted`.

## Links

- [[_moc]]
- [[MasterPlan]]
- [[TimelineEntry]]
- [[../../04-requirements/functional/governance]]
- [[../../STATE]]
