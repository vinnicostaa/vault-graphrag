---
title: Aggregate — AgendaItem
type: spec
tags: [domain, ddd, aggregate, assembly, master-sindico, v2]
bc: assembly
source: 90-ingested/.../specs/requirements/assembly.md Req 49
created: 2026-04-23
updated: 2026-04-23
---

# Aggregate — `AgendaItem`

**BC**: assembly · **Raiz**: ✅ (tratado como AR próprio porque é referenciado standalone por `Vote`)

Item de pauta de uma `Assembly`. 5 tipos. Pauta é lockada após `AgendaPublished`.

## Entidade raiz

```go
type AgendaItem struct {
    id                  AgendaItemID
    assemblyID          AssemblyID
    order               int
    title               string
    description         string
    itemType            AgendaItemType          // informativo | consulta_moradores | votacao_simples | votacao_fracao_ideal | votacao_fornecedor
    attachmentURL       *string
    requiresVoting      bool
    state               AgendaItemState         // Pending | InDebate | Voting | Passed | Rejected | Withdrawn
    votingID            *VotingID               // quando tem votação ativa
    supplierVoteSessionID *SessionID            // pra votacao_fornecedor (link com commercial)
    publishedAt         *time.Time
    createdAt           time.Time
    updatedAt           time.Time
}
```

## Value Objects

- `AgendaItemID`, `VotingID`
- `AgendaItemType`, `AgendaItemState` (enums)

## Invariantes

- **INV-080**: item lockado pós `AgendaPublished` (assembly.agenda_published_at != NULL)
- `order` único por assembly
- `item_type=votacao_fornecedor` requer `supplier_vote_session_id != NULL` e ≥ 2 propostas validadas

## Factories

```go
func NewAgendaItem(assemblyID AssemblyID, order int, title, description string, itemType AgendaItemType, attachmentURL *string, requiresVoting bool) (*AgendaItem, error)
func ReconstructAgendaItem(...) *AgendaItem
```

## Métodos de domínio

- `EditDraft(...) error` — só se `assembly.agenda_published_at == NULL`
- `MoveToDebate()` — `Pending → InDebate`
- `OpenVoting(votingID VotingID)` — `Pending|InDebate → Voting`
- `CloseVoting(result VotingResult)` — `Voting → Passed|Rejected`
- `Withdraw(reason string)` — qualquer não-terminal → `Withdrawn`

## Repository interface

```go
type IAgendaItemRepository interface {
    Save(ctx context.Context, a *AgendaItem) error
    FindByID(ctx context.Context, id AgendaItemID) (*AgendaItem, error)
    ListByAssembly(ctx context.Context, aid AssemblyID) ([]*AgendaItem, error)
}
```

## Eventos emitidos

- (implícitos via `Vote` — `VotingOpened`, `VotingClosed`)

## Entities filhas

> Ratificado em 2026-04-25 (Fase E + D-130 dual-check formal).

- [[Vote]] — entity de AgendaItem (D-130, antes era AR). Métodos críticos via `agendaItem.CastVote()` / `agendaItem.Homologate()`. UNIQUE `(agenda_item_id, voter_id)` enforça INV-071 no DB.

## Status do item de pauta — 4 estados terminais (ASM-038 — Fase E)

- `RESOLVIDO` — votação encerrada com resultado válido (aprovado/rejeitado/escolha).
- `ADIADO` — síndico decide adiar para próxima assembleia.
- `NÃO_RESOLVIDO` — quórum não atingido / votação inconclusiva.
- `PREJUDICADO` — perdeu sentido (ex: já resolvido em outro item).

Cada item registra: `hora_abertura`, `hora_entrada_votacao`, `hora_encerramento`, `duracao_total_discussao`.

## Links

- [[../bounded-contexts#6-assembly]]
- [[../invariants#assembly-inv-071-a-inv-085]]
- [[Assembly]] · [[Vote]] · [[SupplierVoteSession]]
- [[../../STATE]] — D-130
