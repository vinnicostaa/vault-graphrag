---
title: Aggregate — Video
type: spec
tags: [domain, ddd, aggregate, content, mux, master-sindico, v2]
bc: content
source: 90-ingested/.../specs/requirements/content.md Req 29 + contextos/MUX_VIDEO_CONTEXT.md
created: 2026-04-23
updated: 2026-04-23
---

# Aggregate — `Video`

**BC**: content · **Raiz**: ✅

Vídeo institucional (empresa/síndico com `plan_tier ∈ {plus, pro}` — D-067/D-081/D-096) ou vídeo-currículo (morador via addon Banco de Talentos opt-in — D-099 5 vínculos M3). Backed por Mux (upload presigned, HLS, signed playback). **Lock 90d pós-publish** (BR-033).

## Entidade raiz

```go
type Video struct {
    id                          VideoID
    ownerID                     UserID                  // user owner
    tenantType                  TenantType              // company | syndic | resident
    tenantID                    UUID                    // company_id ou condominium_id
    title                       string
    description                 string
    videoType                   VideoType               // 12 tipos
    muxAssetID                  string
    muxPlaybackID               string
    durationSeconds             int
    state                       VideoState              // uploading | processing | moderation | approved | published | locked | unlocked | removed | rejected
    moderationID                *ModerationID
    publishedAt                 *time.Time
    lockedUntil                 *time.Time              // published_at + 90d
    autorizarExibicaoVotacao    bool                    // Rule 4 — empresa autoriza morador ver durante votação
    replacesVideoID             *VideoID                // trava trimestral link
    createdAt                   time.Time
    updatedAt                   time.Time
}
```

## Value Objects

- `VideoID`, `ModerationID`
- `VideoState`, `VideoType` (enum: 12 tipos), `TenantType`
- `LockedUntil` (timestamp)
- `MuxAssetID`, `MuxPlaybackID`

## Invariantes

- **INV-056**: `published` → `locked_until = published_at + 90d`
- **INV-057**: remove pre-90d requer `user.is_admin=true` + audit log
- **INV-058**: passa por `moderation → approved → published`
- **INV-059**: substituir mesmo vídeo só após 90d (via `replaces_video_id`)
- **INV-060**: métricas privadas por tenant
- **INV-061, INV-062**: morador vê empresa só via `VideoVisibilityGrant`
- **INV-070**: signed playback URL TTL curto (≤ 24h)

## Factories

```go
func NewVideo(ownerID UserID, tenantType TenantType, tenantID UUID, title, description string, videoType VideoType, muxAssetID string) (*Video, error) {
    // state=uploading; valida persona pode publicar este videoType
}

// NewReplacement valida trava trimestral 90d
func NewReplacement(original *Video, title, description string, muxAssetID string, now time.Time) (*Video, error) {
    // valida NOW() >= original.published_at + 90d
}

func ReconstructVideo(...) *Video
```

## Métodos de domínio

- `MarkProcessing()` — `uploading → processing` (webhook Mux `video.ready`)
- `RequestModeration()` — `processing → moderation`
- `ApproveModeration(by UserID)` — `moderation → approved`
- `RejectModeration(by UserID, reason string)` — `moderation → rejected`
- `Publish(now time.Time) error` — `approved → published`; seta `lockedUntil = now + 90d`
- `UpdateLockState(now time.Time)` — job: `locked ↔ unlocked`
- `Remove(by UserID, isAdmin bool, reason string) error` — valida lock se !admin
- `AuthorizeExibicaoVotacao(flag bool)` — empresa altera (só Company owners)

## Repository interface

```go
type IVideoRepository interface {
    Save(ctx context.Context, v *Video) error
    FindByID(ctx context.Context, id VideoID) (*Video, error)
    FindByMuxAssetID(ctx context.Context, aid string) (*Video, error)
    ListPublishedByTenant(ctx context.Context, tenantType TenantType, tenantID UUID) ([]*Video, error)
    ListLockedWithExpiredLock(ctx context.Context, now time.Time) ([]*Video, error)         // job unlock
    FindLastByOwnerAndType(ctx context.Context, oid UserID, typ VideoType) (*Video, error)  // trava trimestral
}
```

## Eventos emitidos

- `VideoUploaded` (E-048)
- `VideoModerationRequested` (E-049)
- `VideoApproved` / `VideoRejected` (E-050, E-051)
- `VideoPublished` (E-052)
- `VideoLocked` / `VideoLockExpired` (E-053)
- `VideoRemoved` (E-054)

## Links

- [[../bounded-contexts#5-content]]
- [[../invariants#content-inv-056-a-inv-070]]
- [[../state-machines#2-videomoderation]]
- [[../business-rules]] BR-033..BR-036
- [[VideoModeration]]
