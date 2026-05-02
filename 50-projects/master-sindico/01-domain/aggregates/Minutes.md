---
title: Aggregate — Minutes (Ata)
type: spec
tags: [domain, ddd, aggregate, assembly, immutable, master-sindico, v2]
bc: assembly
source: 90-ingested/.../specs/requirements/assembly.md Req 58 + 60.6
created: 2026-04-23
updated: 2026-04-23
---

# Aggregate — `Minutes` (Ata)

**BC**: assembly · **Raiz**: ✅ · **Imutável pós-publish + homologation**

Ata formal da assembleia. Gerada automaticamente pós `AssemblyClosed`. Assinada digitalmente (MP 2.200-2 / Lei 14.063) pelo síndico + secretário. Homologação obrigatória via votação (Req 60.6) — após: imutável absoluto.

## Entidade raiz

```go
type Minutes struct {
    id                  MinutesID
    assemblyID          AssemblyID
    condominiumID       CondominiumID
    pdfURL              string                  // MinIO/R2
    content             MinutesContent          // estruturado
    publishedAt         *time.Time
    homologatedAt       *time.Time              // após votação homologação
    sindicoSignedAt     *time.Time
    secretarySignedAt   *time.Time
    contentHashSHA256   string                  // hash do conteúdo (Lei 14.063 — Fase E)
    timestampCarimbo    *time.Time              // ICP-Brasil RFC 3161 TSA (M4+)
    transparencyScore   *int
    timelineEntryID     *TimelineEntryID        // auto-criada em institutional
    createdAt           time.Time
}

type MinutesContent struct {
    date                time.Time
    location            string
    presenceList        []PresenceEntry         // anonimizada (só unit + fração)
    proxies             []ProxySummary
    agendaResults       []AgendaResult          // resultado por item
    speechSummaries     []SpeechSummary
    quorumReached       bool
}
```

## Value Objects

- `MinutesID`, `TimelineEntryID`
- `PresenceEntry` (unit_id, fracao_ideal)
- `AgendaResult` (item_id, choice, weight_approve/reject/abstain)

## Invariantes

- **INV-072**: imutável após `published`; absolutamente imutável após `homologated`
- Correção pós-publish/homologação: nova `TimelineEntry` em institutional (nunca edita Minutes)
- **INV-### (Fase E 2026-04-25)**: ata homologada exibe `hash_sha256` do conteúdo + `timestamp_carimbo` (Lei 14.063 ICP-Brasil) — para integridade verificável publicamente. M3: stub HMAC + auditoria; M4+: integração ICP-Brasil (RFC 3161 TSA)

## Factories

```go
func NewMinutes(assemblyID AssemblyID, condoID CondominiumID, content MinutesContent, pdfURL string) (*Minutes, error)
func ReconstructMinutes(...) *Minutes
```

## Métodos de domínio

- `Publish(now time.Time) error` — seta `publishedAt`; dispara `MinutesPublished` → handler institucional cria TimelineEntry
- `SignBySindico(now time.Time)` / `SignBySecretary(now time.Time)` — assinaturas
- `Homologate(now time.Time) error` — requer votação homologação aprovada; seta `homologatedAt` → imutável absoluto
- `SetTransparencyScore(score int)` — pós-homologação

## Repository interface

```go
type IMinutesRepository interface {
    Save(ctx context.Context, m *Minutes) error
    FindByID(ctx context.Context, id MinutesID) (*Minutes, error)
    FindByAssemblyID(ctx context.Context, aid AssemblyID) (*Minutes, error)
    ListByCondominium(ctx context.Context, cid CondominiumID) ([]*Minutes, error)
}
```

## Eventos emitidos

- `MinutesGenerated` (E-074)
- `MinutesPublished` (E-075)
- (homologação via `HomologationApproved` E-076)

## Links

- [[../bounded-contexts#6-assembly]]
- [[../invariants#assembly-inv-071-a-inv-085]]
- [[../business-rules]] BR-049
- [[Assembly]] · [[TimelineEntry]]
