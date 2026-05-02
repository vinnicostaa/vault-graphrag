---
title: Aggregate — LiveSession
type: spec
tags: [domain, ddd, aggregate, assembly, livekit, master-sindico, v2]
bc: assembly
source: 90-ingested/.../specs/requirements/{assembly,cross-domain}.md Req 68
created: 2026-04-23
updated: 2026-04-23
---

# Aggregate — `LiveSession`

**BC**: assembly · **Raiz**: ✅

Sessão Livekit associada a uma `Assembly` (modalidade híbrida/online — M4+). Rooms: `assembly:{id}`. Recording automático em MinIO. API pra corte de áudio (timeout fila de fala — INV-084 saga).

## Entidade raiz

```go
type LiveSession struct {
    id                  LiveSessionID
    assemblyID          AssemblyID
    livekitRoomName     string                  // assembly:{id}
    livekitServerURL    string
    state               LiveSessionState        // scheduled | active | ended | errored
    startedAt           *time.Time
    endedAt             *time.Time
    recordingURL        *string
    participantCount    int
    createdAt           time.Time
}
```

## Value Objects

- `LiveSessionID`, `LiveSessionState` (enum)

## Invariantes

- **INV-084**: saga com compensação — falha em criar Assembly após `StartRoom` → `EndRoom` no Livekit (A-028 legado)
- 1 LiveSession ativa por Assembly em dado momento
- Moderator role no Livekit = presidente assembleia

## Factories

```go
func NewLiveSession(assemblyID AssemblyID, roomName, serverURL string) *LiveSession {
    // state=scheduled
}

func ReconstructLiveSession(...) *LiveSession
```

## Métodos de domínio

- `Start(now time.Time)` — `scheduled → active`
- `End(now time.Time, recordingURL *string)` — `active → ended`
- `MarkErrored(reason string)` — `* → errored` (saga dispara compensation)
- `UpdateParticipantCount(n int)`

## Repository interface

```go
type ILiveSessionRepository interface {
    Save(ctx context.Context, s *LiveSession) error
    FindByID(ctx context.Context, id LiveSessionID) (*LiveSession, error)
    FindByAssemblyID(ctx context.Context, aid AssemblyID) (*LiveSession, error)
    FindActiveByAssembly(ctx context.Context, aid AssemblyID) (*LiveSession, error)
}
```

## Eventos emitidos

- `LiveSessionStarted` (via `AssemblyStarted` E-066 acoplado)
- `LiveSessionEnded` (via `AssemblyClosed` E-073 acoplado)

## Provider externo

- Livekit via `ILiveProvider` — `StartRoom(name, moderator) → RoomInfo`, `EndRoom(name)`, `CutParticipantAudio(room, participant)`.

## Links

- [[../bounded-contexts#6-assembly]]
- [[../invariants#assembly-inv-071-a-inv-085]]
- [[Assembly]] · `SpeechQueue` *(aggregate filho a destilar — fila de fala da assembleia)* *(não detalhado aqui; ver state-machine assembly)*
