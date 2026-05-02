---
title: Aggregate — VideoModeration
type: spec
tags: [domain, ddd, aggregate, content, moderation, master-sindico, v2]
bc: content
source: 90-ingested/.../specs/requirements/content.md Req 29
created: 2026-04-23
updated: 2026-04-23
---

# Aggregate — `VideoModeration`

**BC**: content · **Raiz**: ✅ (ciclo de vida próprio + decisão de admin)

Rastreia processo de moderação de um `Video`. SLA 48h. MVP: moderação manual; ML pipeline M4+ (lista negra de keywords em descrição + análise de vídeo).

## Entidade raiz

```go
type VideoModeration struct {
    id                  ModerationID
    videoID             VideoID
    status              ModerationStatus        // pending | approved | rejected
    reviewedBy          *UserID                 // admin
    reviewedAt          *time.Time
    rejectionReason     *string
    keywordFlags        []string                // palavras-chave detectadas (ML ou manual)
    requestedAt         time.Time
    createdAt           time.Time
    updatedAt           time.Time
}
```

## Value Objects

- `ModerationID`, `ModerationStatus` (enum)

## Invariantes

- **INV-058**: vídeo não chega a `published` sem passar por moderation aprovada
- SLA 48h (monitoring alert se `NOW() - requestedAt > 48h` e `status=pending`)

## Factories

```go
func NewVideoModeration(videoID VideoID, requestedAt time.Time) *VideoModeration {
    // status=pending
}

func ReconstructVideoModeration(...) *VideoModeration
```

## Métodos de domínio

- `Approve(by UserID) error` — `pending → approved`; seta `reviewedBy`, `reviewedAt`
- `Reject(by UserID, reason string) error` — `pending → rejected`

## Repository interface

```go
type IVideoModerationRepository interface {
    Save(ctx context.Context, m *VideoModeration) error
    FindByID(ctx context.Context, id ModerationID) (*VideoModeration, error)
    FindByVideoID(ctx context.Context, vid VideoID) (*VideoModeration, error)
    ListPending(ctx context.Context) ([]*VideoModeration, error)                        // fila admin
    ListPendingSLAViolations(ctx context.Context, now time.Time) ([]*VideoModeration, error)  // alert > 48h
}
```

## Eventos emitidos

- (disparados pelo `Video` aggregate correspondente — eventos `VideoApproved` / `VideoRejected`)

## Links

- [[../bounded-contexts#5-content]]
- [[../invariants#content-inv-056-a-inv-070]]
- [[../state-machines#2-videomoderation]]
- [[Video]]
