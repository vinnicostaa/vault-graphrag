---
title: Aggregate — Proposal
type: spec
tags: [domain, ddd, aggregate, commercial, immutable, master-sindico, v2]
bc: commercial
source: 90-ingested/.../specs/requirements/commercial.md Req 20
created: 2026-04-23
updated: 2026-04-23
---

# Aggregate — `Proposal`

**BC**: commercial · **Raiz**: ✅ · **Imutável pós-`sent`**

Resposta formal de empresa a `ConnectMeRequest`. Status progressivo; após `sent` → imutável (INV-039). Correção via `Amendment` *(entity a destilar quando emendas formais entrarem em escopo)*.

## Entidade raiz

```go
type Proposal struct {
    id                      ProposalID
    connectMeRequestID      ConnectMeRequestID
    companyID               CompanyID
    title                   string
    scope                   string                  // descrição do que será feito
    estimatedCostCents      int64                   // Money
    validUntil              time.Time
    attachments             []AttachmentRef
    status                  ProposalStatus          // draft | sent | awaiting_validation | validated | accepted | rejected | expired
    sentAt                  *time.Time
    validatedAt             *time.Time
    decisionAt              *time.Time
    version                 int
    previousProposalID      *ProposalID             // pra versões via Amendment
    createdAt               time.Time
    updatedAt               time.Time
}
```

## Value Objects

- `ProposalID`, `ProposalStatus`
- `Money` (int64 cents)
- `AttachmentRef`

## Invariantes

- **INV-023**: UNIQUE (connect_me_request_id, company_id) — 1 proposta por empresa por RFP
- **INV-039**: imutável após `status = sent` (Rule 2)
- `estimated_cost_cents >= 0` (CHECK)
- `valid_until > NOW()` na criação

## Factories

```go
func NewProposal(requestID ConnectMeRequestID, companyID CompanyID, title, scope string, costCents int64, validUntil time.Time, attachments []AttachmentRef) (*Proposal, error) {
    // status=draft, version=1
}

// NewProposalVersion cria nova versão via Amendment (link com previousProposalID)
func NewProposalVersion(prev *Proposal, title, scope string, costCents int64, validUntil time.Time, attachments []AttachmentRef) (*Proposal, error) {
    // version = prev.version + 1; previousProposalID = prev.id
}

func ReconstructProposal(...) *Proposal
```

## Métodos de domínio

- `EditDraft(...)` — só se `status = draft`
- `Send() error` — `draft → sent` (imutável depois)
- `MoveToValidation()` — `sent → awaiting_validation` (síndico abre período análise)
- `Validate(by UserID)` — `awaiting_validation → validated` (pré-req pra votação fornecedor)
- `Accept()` — `validated → accepted` (terminal)
- `Reject(reason string)` — qualquer não-terminal → `rejected`
- `Expire()` — `NOW() >= valid_until`

## Repository interface

```go
type IProposalRepository interface {
    Save(ctx context.Context, p *Proposal) error
    FindByID(ctx context.Context, id ProposalID) (*Proposal, error)
    FindByRequestAndCompany(ctx context.Context, rid ConnectMeRequestID, cid CompanyID) (*Proposal, error)
    ListByRequest(ctx context.Context, rid ConnectMeRequestID) ([]*Proposal, error)
    ListValidatedByRequest(ctx context.Context, rid ConnectMeRequestID) ([]*Proposal, error)    // pra votação fornecedor (≥ 2)
    ListExpiringSoon(ctx context.Context, within time.Duration) ([]*Proposal, error)            // notification
}
```

## Eventos emitidos

- `ProposalSubmitted` (E-034)
- `ProposalValidated` (E-035)
- `ProposalAccepted` / `ProposalRejected` (E-036)

## Links

- [[../bounded-contexts#4-commercial]]
- [[../invariants#commercial-inv-036-a-inv-055]]
- [[../state-machines#9-proposal]]
- [[../business-rules]] BR-024
- [[ConnectMeRequest]] · [[ClosedDeal]] · [[Company]]
