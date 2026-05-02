---
title: Aggregate — Poll
type: spec
tags: [domain, ddd, aggregate, institutional, master-sindico, v2]
bc: institutional
source: 90-ingested/.../specs/requirements/institutional.md + assembly.md Req 51 (preliminary)
created: 2026-04-23
updated: 2026-04-23
---

# Aggregate — `Poll` (Enquete Informal)

**BC**: institutional · **Raiz**: ✅

Enquete consultiva (**não** votação formal de assembleia). Sem valor jurídico; sem quórum fracional. Usada pra sondar opinião antes de assembleia ou pra decisões leves.

## Entidade raiz

```go
type Poll struct {
    id              PollID
    condominiumID   CondominiumID
    createdBy       UserID
    title           string
    description     string
    pollType        PollType                // sim_nao_nao_sei | multiple_choice | scale_1_to_5
    options         []PollOption            // pra multiple_choice
    state           PollState               // draft | open | closed
    openedAt        *time.Time
    closedAt        *time.Time
    results         []PollResponse          // agregado
    createdAt       time.Time
    updatedAt       time.Time
}
```

## Value Objects

- `PollID`, `PollType` (enum), `PollState`
- `PollOption` (id + text)
- `PollResponse` (user_id, option_id ou value, responded_at)

## Invariantes

- Uma resposta única por (poll, user) — UNIQUE parcial
- Não é vinculante (diferente de Vote em assembly)

## Factories

```go
func NewPoll(condoID CondominiumID, createdBy UserID, title, description string, typ PollType, options []PollOption) (*Poll, error) {
    // state=draft
}

func ReconstructPoll(...) *Poll
```

## Métodos de domínio

- `Open() error` — `draft → open`
- `Close() error` — `open → closed`
- `RegisterResponse(userID UserID, optionID *OptionID, value *int) error` — UPSERT
- `Results() []AggregatedResult`

## Repository interface

```go
type IPollRepository interface {
    Save(ctx context.Context, p *Poll) error
    FindByID(ctx context.Context, id PollID) (*Poll, error)
    ListByCondominium(ctx context.Context, cid CondominiumID) ([]*Poll, error)
    SaveResponse(ctx context.Context, r *PollResponse) error                // UPSERT
}
```

## Eventos emitidos

- `PollOpened` / `PollClosed` (E-022)

## Links

- [[../bounded-contexts#3-institutional]]
- [[Condominium]]
