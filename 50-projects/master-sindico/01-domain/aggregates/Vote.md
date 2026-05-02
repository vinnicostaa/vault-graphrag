---
title: Aggregate — Vote
type: spec
tags:
  - domain
  - ddd
  - entity
  - assembly
  - immutable
  - master-sindico
  - v2
bc: assembly
source: 90-ingested/.../specs/requirements/assembly.md Req 57 + domain-rules DR-004
created: 2026-04-23T00:00:00.000Z
updated: '2026-04-25'
---

# Entity — `Vote`

**BC**: assembly · **Raiz**: ❌ (entity de [[AgendaItem]] aggregate)

Voto individual em `AgendaItem`. Único por (agenda_item, voter). Imutável. Peso = fração ideal (snapshot no momento do voto).

> **Atualização 2026-04-25 ([[../../STATE#D-130]])**: Vote rebatizado de aggregate root → **entity de AgendaItem** após dual-check formal. Pragmaticamente mantém `IVoteRepository` dedicado por escala (10k+ votos por assembleia grande), mas **invariantes delegadas a [[AgendaItem]]**. Métodos críticos sempre via `AgendaItem.CastVote()` / `AgendaItem.Homologate()` — não via VoteRepo direto. UNIQUE `(agenda_item_id, voter_id)` enforça relação no DB.

## Entidade raiz

```go
type Vote struct {
    id                  VoteID
    assemblyID          AssemblyID
    agendaItemID        AgendaItemID
    voterUserID         UserID
    unitID              UnitID
    choice              VoteChoice              // Aprovar | Rejeitar | Abster | Absent
    fracaoIdealWeight   FracaoIdeal             // snapshot
    isProxy             bool                    // true se é proxy
    isPresencialAssistido bool                  // Req 57
    proxyID             *ProxyID
    votedAt             time.Time
    ipAddress           string
    deviceFingerprint   DeviceFingerprint
    isHomologated       bool                    // ata homologada
    createdAt           time.Time
}
```

## Value Objects

- `VoteID`
- `VoteChoice` (enum pt-BR)
- `FracaoIdeal` (snapshot)
- `DeviceFingerprint`

## Invariantes

- **INV-071**: UNIQUE(agenda_item_id, voter_id) — double-voting impossível (DR-004, A-025 legado fix); tratamento `ErrConflict`
- **INV-082**: `fracao_ideal_weight` NUMERIC(5,4)
- **INV-083**: snapshot no momento do voto (não referência)
- **INV-076**: voto `Absent` após 15min de timeout idempotente
- **INV-077**: proxy não vota diferente do titular
- **INV-081**: presidente da assembleia não vota

## Factories

```go
func NewVote(assemblyID AssemblyID, agendaItemID AgendaItemID, voterID UserID, unitID UnitID, choice VoteChoice, fracao FracaoIdeal, isProxy bool, proxyID *ProxyID, isPresencialAssistido bool, ip string, fp DeviceFingerprint, now time.Time) (*Vote, error) {
    // valida: choice não vazia; fracao > 0; se isProxy então proxyID != nil; ip não vazio
}

func NewAbsentVote(assemblyID AssemblyID, agendaItemID AgendaItemID, voterID UserID, unitID UnitID, fracao FracaoIdeal, now time.Time) (*Vote, error) {
    // choice=Absent (timeout job)
}

func ReconstructVote(...) *Vote
```

## Métodos de domínio

> Métodos invocados **via `AgendaItem` aggregate root**, não direto na entity:
> - `agendaItem.CastVote(voter, choice, ...)` cria entity Vote
> - `agendaItem.Homologate()` itera votes e marca `isHomologated=true` em todos

- `markHomologated()` (private) — chamado por `AgendaItem.Homologate()` durante homologação da ata

## Repository interface

```go
type IVoteRepository interface {
    Save(ctx context.Context, v *Vote) error                                    // com ErrConflict em UNIQUE
    FindByID(ctx context.Context, id VoteID) (*Vote, error)
    FindByAgendaItemAndVoter(ctx context.Context, aid AgendaItemID, vid UserID) (*Vote, error)
    ListByAgendaItem(ctx context.Context, aid AgendaItemID) ([]*Vote, error)
    CountByAgendaItem(ctx context.Context, aid AgendaItemID) (approve, reject, abstain, absent int, weightApprove, weightReject, weightAbstain, weightAbsent FracaoIdeal, err error)
}
```

## Eventos emitidos

- `VoteCast` (E-070) — WebSocket broadcast contagem

## Links

- [[../bounded-contexts#6-assembly]]
- [[../invariants#assembly-inv-071-a-inv-085]]
- [[../state-machines#5-votesession]]
- [[../business-rules]] BR-045..BR-048
- [[Assembly]] · [[AgendaItem]] · [[Proxy]]
