---
title: Aggregate — TimelineEntry
type: spec
tags: [domain, ddd, aggregate, institutional, immutable, master-sindico, v2]
bc: institutional
source: 90-ingested/.../specs/requirements/institutional.md Req 11 + domain-rules DR-007
created: 2026-04-23
updated: 2026-04-23
---

# Aggregate — `TimelineEntry`

**BC**: institutional · **Raiz**: ✅ · **Imutabilidade total**

Memória institucional **append-only imutável**. Sem UPDATE, sem DELETE, sem `deleted_at`. Correção = nova entry com `corrects_entry_id` + motivo.

## Entidade raiz

```go
type TimelineEntry struct {
    id                  TimelineEntryID         // UUIDv7 ordenável
    condominiumID       CondominiumID
    authorID            UserID                  // síndico ou sistema
    entryType           TimelineEntryType       // comunicado | evento | assembleia_result | registro_execucao | mandato_iniciado | mandato_encerrado | adendo | outro
    title               string
    description         string
    payload             json.RawMessage         // dados específicos do tipo
    attachments         []AttachmentRef
    isAmended           bool                    // true se é adendo
    originalEntryID     *TimelineEntryID        // se isAmended=true
    publishedAt         time.Time               // único timestamp; nunca muda
    // SEM updated_at, SEM deleted_at
}
```

## Value Objects

- `TimelineEntryID`, `TimelineEntryType` (enum pt-BR: 7+ tipos)
- `AttachmentRef` (`{url, mime_type, filename}`)

## Invariantes

- **INV-024**: imutável — sem UPDATE, sem DELETE, sem `deleted_at` (DR-007; CI grep bloqueia)
- **INV-032**: correção = nova entry com `corrects_entry_id` (nunca edita original)
- Eventos originadores: `AnnouncementPublished`, `AssemblyClosed`+`MinutesPublished` (auto), `ClosedDealSigned` (auto), `MandateStarted`/`MandateEnded`, `AmendmentSigned`.

## Factories

```go
func NewTimelineEntry(condoID CondominiumID, authorID UserID, entryType TimelineEntryType, title, description string, payload json.RawMessage, attachments []AttachmentRef) (*TimelineEntry, error) {
    // publishedAt = NOW() UTC; ID UUIDv7
}

// Correct cria entry de correção (adendo) linkando ao original
func (t *TimelineEntry) Correct(authorID UserID, reason string, newPayload json.RawMessage) (*TimelineEntry, error) {
    // isAmended=true; originalEntryID = t.id
}

func ReconstructTimelineEntry(...) *TimelineEntry
```

## Métodos de domínio

- **Nenhum** método de mutação. Factory + queries apenas.

## Repository interface

```go
type ITimelineEntryRepository interface {
    Save(ctx context.Context, e *TimelineEntry) error                              // INSERT-only
    FindByID(ctx context.Context, id TimelineEntryID) (*TimelineEntry, error)
    ListByCondominium(ctx context.Context, cid CondominiumID, filter TimelineFilter, cursor Cursor) ([]*TimelineEntry, Cursor, error)
    ListAmendmentsOf(ctx context.Context, originalID TimelineEntryID) ([]*TimelineEntry, error)
    // SEM Update, SEM Delete
}
```

## Eventos emitidos

- `TimelineEntryAdded` (E-019) — fan-out a `content` (indexa busca) e `cross-domain` (audit)

## Links

- [[../bounded-contexts#3-institutional]]
- [[../invariants#institutional-inv-021-a-inv-035]]
- [[../business-rules]] BR-006 (imutabilidade)
- [[Condominium]] · [[../aggregates/Announcement]]
