---
title: Aggregate — ClosedDeal
type: spec
tags: [domain, ddd, aggregate, commercial, immutable, master-sindico, v2]
bc: commercial
source: 90-ingested/.../specs/requirements/commercial.md Req 22
created: 2026-04-23
updated: 2026-04-23
---

# Aggregate — `ClosedDeal` (Negócio Fechado)

**BC**: commercial · **Raiz**: ✅ · **Imutável pós-dupla-assinatura**

Contrato fechado entre síndico e empresa. Requer **dupla assinatura digital** (empresa → síndico). Após ambas: imutável (`is_immutable=true`); auto-publica `TimelineEntry` tipo `registro_execucao`.

## Entidade raiz

```go
type ClosedDeal struct {
    id                  DealID
    dealNumber          DealNumber              // DL-YYYYMMDD-NNN
    connectMeRequestID  ConnectMeRequestID
    proposalID          ProposalID
    companyID           CompanyID
    condominiumID       CondominiumID
    agreedCostCents     int64                   // Money
    startDate           time.Time
    endDate             time.Time
    companySignedAt     *time.Time
    sindicoSignedAt     *time.Time
    isImmutable         bool                    // true pós-dupla-assinatura
    executionRecords    []ExecutionRecord       // histórico de entregas
    timelineEntryID     *TimelineEntryID        // auto-criada
    createdAt           time.Time
    updatedAt           time.Time
}
```

## Value Objects

- `DealID`, `DealNumber` (formato `DL-YYYYMMDD-NNN`)
- `Money`
- `ExecutionRecord` (entity interna com fotos, datas, status validação)

## Invariantes

- **INV-025**: imutável após dupla assinatura (`company_signed_at` + `sindico_signed_at` ambos preenchidos)
- **INV-041**: auto-publica TimelineEntry pós dupla assinatura (dentro UoW)
- `end_date >= start_date`
- `agreed_cost_cents >= 0`

## Factories

```go
func NewClosedDeal(requestID ConnectMeRequestID, proposalID ProposalID, companyID CompanyID, condoID CondominiumID, costCents int64, startDate, endDate time.Time, dealNumberGenerator DealNumberGen) (*ClosedDeal, error) {
    // isImmutable=false; dealNumber gerado
}

func ReconstructClosedDeal(...) *ClosedDeal
```

## Métodos de domínio

- `SignByCompany(now time.Time) error` — seta `companySignedAt`; rejeita se já assinado; `isImmutable` só se sindico também assinou
- `SignBySindico(now time.Time) error` — seta `sindicoSignedAt`; se company já assinou → `isImmutable=true` + dispara evento
- `AddExecutionRecord(r ExecutionRecord) error` — só se `isImmutable=true`
- `ValidateExecution(recordID ExecutionRecordID, approved bool, reason *string)` — síndico valida; muda status da atividade MasterPlan (via evento)

## Repository interface

```go
type IClosedDealRepository interface {
    Save(ctx context.Context, d *ClosedDeal) error
    FindByID(ctx context.Context, id DealID) (*ClosedDeal, error)
    FindByDealNumber(ctx context.Context, dn DealNumber) (*ClosedDeal, error)
    ListByCondominium(ctx context.Context, cid CondominiumID) ([]*ClosedDeal, error)
    ListByCompany(ctx context.Context, cid CompanyID) ([]*ClosedDeal, error)
    CountImmutableByCompany(ctx context.Context, cid CompanyID) (int, error)            // pra Reputação
}
```

## Eventos emitidos

- `ClosedDealSigned` (E-037) — dispara handler em institutional pra criar TimelineEntry
- `ExecutionRecordSubmitted` (E-038)
- `ExecutionValidated` (E-039) — dispara handler em institutional pra avançar MasterPlan activity

## Links

- [[../bounded-contexts#4-commercial]]
- [[../invariants#commercial-inv-036-a-inv-055]]
- [[../business-rules]] BR-025, BR-026
- [[ConnectMeRequest]] · [[Proposal]] · [[Company]] · [[TimelineEntry]] · [[MasterPlan]]
