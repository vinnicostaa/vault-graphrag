---
title: Aggregate — HarassmentReport
type: spec
tags: [domain, ddd, aggregate, commercial, moderation, master-sindico, v2]
bc: commercial
source: 90-ingested/.../specs/requirements/commercial.md Req 19 anti-assédio
created: 2026-04-23
updated: 2026-04-23
---

# Aggregate — `HarassmentReport`

**BC**: commercial · **Raiz**: ✅

Denúncia formal de abuso em Connect Me (assédio, linguagem imprópria, fraude). Moderação admin em 48h. Confirmação → `Company.suspended=true` por 72h (BR-031).

## Entidade raiz

```go
type HarassmentReport struct {
    id                  ReportID
    reportedUserID      UserID                  // alvo (empresa ou user)
    reporterUserID      UserID                  // quem reportou (hashed na UI)
    context             ReportContext           // connect_me | proposal | closed_deal | evaluation | other
    contextRefID        *UUID                   // RFP/proposal/deal alvo
    reason              string
    evidence            []AttachmentRef
    state               ReportState             // reported | in_review | confirmed | dismissed | escalated | closed
    reviewedBy          *UserID
    reviewedAt          *time.Time
    action              *ReportAction           // warning | suspension_72h | ban | escalate
    reportedAt          time.Time
    closedAt            *time.Time
}
```

## Value Objects

- `ReportID`, `ReportContext`, `ReportState`, `ReportAction` (enums)

## Invariantes

- **INV-055**: confirmado → `Company.suspended=true` por 72h (flag sobre tier)
- Retenção 5 anos (LGPD)
- `reporter_user_id` nunca exposto ao reportado

## Factories

```go
func NewHarassmentReport(reportedUserID, reporterUserID UserID, context ReportContext, ctxRefID *UUID, reason string, evidence []AttachmentRef) (*HarassmentReport, error) {
    // state=reported
}

func ReconstructHarassmentReport(...) *HarassmentReport
```

## Métodos de domínio

- `StartReview(by UserID)` — `reported → in_review`
- `Confirm(action ReportAction)` — `in_review → confirmed` → publica evento pra `Company.ApplyHarassmentSuspension`
- `Dismiss(reason string)` — `in_review → dismissed`
- `Escalate(to string)` — `in_review → escalated` (legal/DPO)
- `Close()` — terminal

## Repository interface

```go
type IHarassmentReportRepository interface {
    Save(ctx context.Context, r *HarassmentReport) error
    FindByID(ctx context.Context, id ReportID) (*HarassmentReport, error)
    ListPendingForAdmin(ctx context.Context) ([]*HarassmentReport, error)
    ListByReportedUser(ctx context.Context, uid UserID) ([]*HarassmentReport, error)
}
```

## Eventos emitidos

- `HarassmentReported` (E-044)
- (indireto via handler `CompanySuspended`)

## Links

- [[../bounded-contexts#4-commercial]]
- [[../invariants#commercial-inv-036-a-inv-055]]
- [[../state-machines#10-harassmentreport]]
- [[../business-rules]] BR-023, BR-031
- [[Company]]
