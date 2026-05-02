---
title: Aggregate — SupplierVoteSession
type: spec
tags: [domain, ddd, aggregate, commercial, supplier-vote, master-sindico, v2]
bc: commercial
source: 90-ingested/.../specs/requirements/commercial.md Req 21 + assembly.md Req 57
created: 2026-04-23
updated: 2026-04-23
---

# Aggregate — `SupplierVoteSession`

**BC**: commercial (também aparece em assembly como tipo de `votacao_fornecedor`)

Votação colegiada do conselho (ou assembleia completa) pra escolher entre ≥ 2 `Proposal` validadas. Fração ideal usada como peso.

## Entidade raiz

```go
type SupplierVoteSession struct {
    id                  SessionID
    condominiumID       CondominiumID
    connectMeRequestID  ConnectMeRequestID
    assemblyID          *AssemblyID             // se é parte de assembleia formal
    agendaItemID        *AgendaItemID
    validatedProposalIDs []ProposalID           // ≥ 2
    quorumType          QuorumType              // simple_majority | qualified_majority | unanimous
    state               VotingStatus            // open | vote_count | closed | tallied
    openedAt            time.Time
    closedAt            *time.Time
    casts               []SupplierVoteCast      // sub-entidades
    result              *VotingResult
    createdAt           time.Time
}

type SupplierVoteCast struct {
    id                  CastID
    sessionID           SessionID
    voterUserID         UserID
    voterFracaoIdeal    FracaoIdeal             // snapshot
    chosenProposalID    ProposalID
    castAt              time.Time
}
```

## Value Objects

- `SessionID`, `CastID`, `AssemblyID`, `AgendaItemID`
- `QuorumType`, `VotingStatus` (enums)
- `VotingResult` (vencedor + counts + quorum_reached)

## Invariantes

- **INV-042**: `SupplierVoteCast` UNIQUE(session_id, voter_id) (DR-005; A-026 legado fix)
- **INV-083** (compartilhado com assembly): `voterFracaoIdeal` snapshot no momento do voto
- Mínimo 2 `validated_proposal_ids` (pré-req abertura)
- Resultado imutável após `closed`

## Factories

```go
func NewSupplierVoteSession(condoID CondominiumID, requestID ConnectMeRequestID, proposals []ProposalID, quorumType QuorumType) (*SupplierVoteSession, error) {
    // valida: len(proposals) >= 2; todas validated
}

func ReconstructSupplierVoteSession(...) *SupplierVoteSession
```

## Métodos de domínio

- `Open()` — `— → open`
- `RegisterCast(cast SupplierVoteCast) error` — UNIQUE per voter; dentro UoW + SELECT FOR UPDATE (A-030 fix)
- `Close()` — `open → vote_count → closed`
- `Tally()` — calcula resultado por peso de fração; seta `result`; transition `closed → tallied`

## Repository interface

```go
type ISupplierVoteSessionRepository interface {
    Save(ctx context.Context, s *SupplierVoteSession) error
    FindByID(ctx context.Context, id SessionID) (*SupplierVoteSession, error)
    FindByRequestID(ctx context.Context, rid ConnectMeRequestID) (*SupplierVoteSession, error)
    SaveCast(ctx context.Context, c *SupplierVoteCast) error                          // com ErrConflict (UNIQUE)
    CountCastsBySessionForUpdate(ctx context.Context, sid SessionID) (int, error)     // SELECT FOR UPDATE
}
```

## Eventos emitidos

- `SupplierVoteSessionOpened` / `SupplierVoteClosed` (E-042)
- `SupplierVoteCastRegistered` (E-043)

## Links

- [[../bounded-contexts#4-commercial]]
- [[../invariants#commercial-inv-036-a-inv-055]]
- [[ConnectMeRequest]] · [[Proposal]] · [[../aggregates/Assembly]] · [[../aggregates/AgendaItem]]
