---
title: Aggregate — MasterPlan
type: spec
tags: [domain, ddd, aggregate, institutional, master-plan, master-sindico, v2]
bc: institutional
source: 90-ingested/.../specs/requirements/institutional.md + legacy design.md master_plan_actions
created: 2026-04-23
updated: 2026-04-23
---

# Aggregate — `MasterPlan` (Plano Diretor)

**BC**: institutional · **Raiz**: ✅

Plano plurianual (3-5 anos) do condomínio com atividades categorizadas. Status de atividade atualizado **apenas** via evento da Timeline (INV-031).

## Entidade raiz

```go
type MasterPlan struct {
    id              MasterPlanID
    condominiumID   CondominiumID
    title           string
    startYear       int
    endYear         int
    activities      []MasterPlanActivity
    createdAt       time.Time
    updatedAt       time.Time
}

type MasterPlanActivity struct {
    id              ActivityID
    masterPlanID    MasterPlanID
    title           string
    tipoAtividade   TipoAtividade           // 26 tipos (legado)
    areaImpactada   AreaImpactada           // 26 áreas
    tipoRisco       TipoRisco               // 9 tipos
    status          ActivityStatus          // planned | contracted | in_progress | delayed | completed | rejected | amended
    dueDate         time.Time
    budgetEstimatedCents int64              // Money
    createdAt       time.Time
    updatedAt       time.Time
}
```

## Value Objects

- `MasterPlanID`, `ActivityID`
- `TipoAtividade`, `AreaImpactada`, `TipoRisco`, `ActivityStatus` (enums)

## Invariantes

- **INV-031**: `activity.status` muda **apenas** via evento `ExecutionValidated` (ou `TimelineEntryAdded`)
- `endYear >= startYear`
- `Σ budget_estimated` é informativo (não restritivo)

## Factories

```go
func NewMasterPlan(condoID CondominiumID, title string, startYear, endYear int) (*MasterPlan, error)
func NewActivity(masterPlanID MasterPlanID, title string, tipo TipoAtividade, area AreaImpactada, risco TipoRisco, dueDate time.Time, budget int64) (*MasterPlanActivity, error)
func ReconstructMasterPlan(...) *MasterPlan
```

## Métodos de domínio

- `AddActivity(a MasterPlanActivity) error` — via síndico no planejamento
- `RemoveActivity(activityID ActivityID) error` — se `status = planned`; senão Amendment
- **`AdvanceActivityStatus(activityID ActivityID, newStatus ActivityStatus, sourceEventID UUIDv7) error`** — só chamado pelo handler de eventos (`ExecutionValidated`, etc); guard `sourceEventID != nil`

## Repository interface

```go
type IMasterPlanRepository interface {
    Save(ctx context.Context, mp *MasterPlan) error
    FindByID(ctx context.Context, id MasterPlanID) (*MasterPlan, error)
    FindCurrentByCondominium(ctx context.Context, cid CondominiumID) (*MasterPlan, error)
    SaveActivity(ctx context.Context, a *MasterPlanActivity) error
    ListActivitiesByPlan(ctx context.Context, pid MasterPlanID) ([]*MasterPlanActivity, error)
}
```

## Eventos emitidos

- `MasterPlanActivityAdvanced` (E-025)

## Links

- [[../bounded-contexts#3-institutional]]
- [[../invariants#institutional-inv-021-a-inv-035]]
- [[../business-rules]] BR-010
- [[../state-machines#8-masterplan-activity]]
- [[Condominium]] · [[TimelineEntry]]
