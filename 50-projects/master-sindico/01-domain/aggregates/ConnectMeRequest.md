---
title: Aggregate — ConnectMeRequest
type: spec
tags: [domain, ddd, aggregate, commercial, connect-me, master-sindico, v2]
bc: commercial
source: 90-ingested/.../specs/requirements/commercial.md Req 19-19.3
created: 2026-04-23
updated: 2026-04-23
---

# Aggregate — `ConnectMeRequest`

**BC**: commercial · **Raiz**: ✅ · **Unidirecional** (não chat)

RFP formal. Quatro fluxos (Decision 5):

1. Síndico → Empresa
2. Morador → Síndico
3. Empresa → Empresa (Plus/Pro)
4. Síndico → Parceiro (Vizinhança)

## Entidade raiz

```go
type ConnectMeRequest struct {
    id                  ConnectMeRequestID
    fluxo               ConnectMeFluxo          // SindicoEmpresa | MoradorSindico | EmpresaEmpresa | SindicoParceiro
    publisherUserID     UserID
    targetScope         TargetScope             // empresa específica | categoria | parceiro específico | síndico do próprio condo
    category            *CategoriaID
    subject             string
    message             string
    urgency             ConnectMeUrgency        // baixa | media | alta | urgente
    contactName         string
    contactEmail        Email
    contactPhone        *Phone
    state               ConnectMeState          // draft | published | interest_expressed | proposals_received | auto_closed | selected | closed_deal | evaluated | canceled
    dataRevealedAt      *time.Time
    lgpdLogID           *LGPDLogID
    autoClosedAt        *time.Time
    publishedAt         *time.Time
    createdAt           time.Time
    updatedAt           time.Time
}
```

## Value Objects

- `ConnectMeRequestID`, `ConnectMeFluxo`, `ConnectMeUrgency`, `ConnectMeState`
- `TargetScope`

## Invariantes

- **INV-022, INV-037**: unidirecional — sem reply, sem thread (Rule 3)
- **INV-045**: quota consumida **no publish** (não no draft)
- **INV-047**: auto-close após 48h sem interesse
- **INV-048**: LGPD log obrigatório ao revelar dados

## Factories

```go
func NewConnectMeRequest(fluxo ConnectMeFluxo, publisherID UserID, target TargetScope, subject, message string, urgency ConnectMeUrgency, contactName string, contactEmail Email, contactPhone *Phone) (*ConnectMeRequest, error) {
    // state=draft
}

func ReconstructConnectMeRequest(...) *ConnectMeRequest
```

## Métodos de domínio

- `EditDraft(...)` — só se `state=draft`
- `Publish(now time.Time) error` — `draft → published`; valida quota (caller decrementa em UoW); seta `publishedAt`
- `ExpressInterest(companyID CompanyID, lgpdLogID LGPDLogID) error` — `published → interest_expressed`; reveal dados
- `AutoClose()` — `published → auto_closed` (job 48h)
- `MarkProposalsReceived()` — `interest_expressed → proposals_received`
- `Select(proposalID ProposalID)` — `proposals_received → selected`
- `CloseDeal(dealID DealID)` — `selected → closed_deal`
- `MarkEvaluated(evaluationID EvaluationID)` — `closed_deal → evaluated`
- `Cancel(reason string)` — terminal

## Repository interface

```go
type IConnectMeRequestRepository interface {
    Save(ctx context.Context, r *ConnectMeRequest) error
    FindByID(ctx context.Context, id ConnectMeRequestID) (*ConnectMeRequest, error)
    ListByPublisher(ctx context.Context, uid UserID, state *ConnectMeState) ([]*ConnectMeRequest, error)
    ListPublishedByCategory(ctx context.Context, catID CategoriaID) ([]*ConnectMeRequest, error)
    ListExpiredPublished(ctx context.Context, now time.Time) ([]*ConnectMeRequest, error)   // job 48h
}
```

## Eventos emitidos

- `ConnectMeRequestPublished` (E-031)
- `ConnectMeInterestExpressed` (E-032)
- `ConnectMeAutoClosed` (E-033)
- (indiretos via Proposal/ClosedDeal)

## Links

- [[../bounded-contexts#4-commercial]]
- [[../invariants#commercial-inv-036-a-inv-055]]
- [[../state-machines#4-connectmerequest]]
- [[../business-rules]] BR-019..BR-023
- [[Proposal]] · [[ClosedDeal]] · [[Company]]
