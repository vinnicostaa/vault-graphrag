---
title: Aggregate — Announcement
type: spec
tags: [domain, ddd, aggregate, institutional, immutable, master-sindico, v2]
bc: institutional
source: 90-ingested/.../specs/requirements/institutional.md + domain-rules DR-008
created: 2026-04-23
updated: 2026-04-23
---

# Aggregate — `Announcement` (Comunicado)

**BC**: institutional · **Raiz**: ✅ · **Imutável pós-publish**

Comunicado oficial publicado pelo síndico. Tipo obrigatório, conteúdo rico (markdown + anexos). Após `publish` → imutável (INV-025); correção via Amendment.

## Entidade raiz

```go
type Announcement struct {
    id              AnnouncementID
    condominiumID   CondominiumID
    authorID        UserID
    announcementType AnnouncementType       // InformativoGeral | Aviso | Urgente | Festa
    title           string
    body            string                  // markdown
    attachments     []AttachmentRef
    state           AnnouncementState       // draft | published
    publishedAt     *time.Time
    createdAt       time.Time
    updatedAt       time.Time
}
```

## Value Objects

- `AnnouncementID`, `AnnouncementType` (enum pt-BR), `AnnouncementState`
- `AttachmentRef`

## Invariantes

- **INV-025**: imutável após `state = published` (DR-008)
- Factory `Publish()` dispara `AnnouncementPublished` + auto-cria `TimelineEntry` tipo `comunicado` (via evento)

## Factories

```go
func NewAnnouncement(condoID CondominiumID, authorID UserID, typ AnnouncementType, title, body string, attachments []AttachmentRef) (*Announcement, error) {
    // state=draft
}

func ReconstructAnnouncement(...) *Announcement
```

## Métodos de domínio

- `EditDraft(title, body string, attachments []AttachmentRef) error` — só se `state=draft`
- `Publish() error` — `draft → published`; seta `publishedAt = NOW()`; dispara evento
- (sem Unpublish, sem Edit pós-publish)

## Repository interface

```go
type IAnnouncementRepository interface {
    Save(ctx context.Context, a *Announcement) error
    FindByID(ctx context.Context, id AnnouncementID) (*Announcement, error)
    ListByCondominium(ctx context.Context, cid CondominiumID, cursor Cursor) ([]*Announcement, Cursor, error)
    ListDraftsByAuthor(ctx context.Context, aID UserID) ([]*Announcement, error)
}
```

## Eventos emitidos

- `AnnouncementPublished` (E-020) — fan-out notification via FCM/Resend; handler em institutional cria TimelineEntry

## Links

- [[../bounded-contexts#3-institutional]]
- [[../invariants#institutional-inv-021-a-inv-035]]
- [[../business-rules]] BR-009
- [[TimelineEntry]] · [[Condominium]]
