---
title: Aggregate — Assembly
type: spec
tags: [domain, ddd, aggregate, assembly, master-sindico, v2]
bc: assembly
source: 90-ingested/.../specs/requirements/assembly.md Req 48-60
created: 2026-04-23
updated: 2026-04-23
---

# Aggregate — `Assembly` (Assembleia)

**BC**: assembly · **Raiz**: ✅

Assembleia condominial. Contém pauta, check-ins, votações, fila de fala, ata. Modalidade MVP: presencial. Online/híbrida M4+.

## Entidade raiz

```go
type Assembly struct {
    id                  AssemblyID
    condominiumID       CondominiumID
    createdBy           UserID                  // síndico
    presidentID         UserID                  // síndico presidente (não vota — INV-081)
    type                AssemblyType            // Ordinaria | Extraordinaria | Ratificacao | Outra
    status              AssemblyStatus          // scheduled | published | in_progress | closed | homologated | canceled
    modality            Modality                // presencial | hibrida | online (MVP=presencial)
    scheduledDate       time.Time
    location            *string
    speechTimeMinutes   int                     // padrão 2 ou 3
    quorumType          QuorumType              // simple_majority | qualified_majority | unanimous
    agendaItems         []AgendaItem            // via aggregate AgendaItem
    attendanceRecords   []AttendanceRecord
    proxies             []Proxy
    minutesID           *MinutesID
    liveSessionID       *LiveSessionID
    editalPublishedAt   *time.Time
    agendaPublishedAt   *time.Time
    startedAt           *time.Time
    closedAt            *time.Time
    homologatedAt       *time.Time
    transparencyScore   *int                    // 0-100 pós-homologação
    createdAt           time.Time
    updatedAt           time.Time
}
```

## Value Objects

- `AssemblyID`, `MinutesID`, `LiveSessionID`
- `AssemblyType`, `AssemblyStatus`, `Modality`, `QuorumType` (enums)
- `QuorumThreshold` (derivado de `QuorumType`)

## Invariantes

- **INV-074**: não inicia sem edital publicado (A-013 legado fix)
- **INV-075**: ≤ 1 assembleia ativa por condomínio (UNIQUE parcial)
- **INV-084**: saga StartLive com compensação (A-028 legado)
- **INV-081**: presidente não vota

## Factories

```go
func NewAssembly(condoID CondominiumID, createdBy, presidentID UserID, typ AssemblyType, modality Modality, scheduledDate time.Time, location *string, speechTimeMin int, quorumType QuorumType) (*Assembly, error) {
    // status=scheduled; valida scheduledDate > NOW()+5d
}

func ReconstructAssembly(...) *Assembly
```

## Métodos de domínio

- `PublishEdital(editalPdfURL string) error`
- `PublishAgenda() error` — lock edição; `scheduled → published`
- `Start(now time.Time) error` — `published → in_progress`; requer edital + agenda published
- `Close(now time.Time) error` — `in_progress → closed`; todas votações fechadas; dispara geração ata
- `Homologate(homologationApproved bool) error` — `closed → homologated`
- `Cancel(reason string)` — até 3d pré

## Repository interface

```go
type IAssemblyRepository interface {
    Save(ctx context.Context, a *Assembly) error
    FindByID(ctx context.Context, id AssemblyID) (*Assembly, error)
    FindActiveByCondominium(ctx context.Context, cid CondominiumID) (*Assembly, error)    // INV-075
    ListByCondominium(ctx context.Context, cid CondominiumID) ([]*Assembly, error)
}
```

## Eventos emitidos

- `AssemblyScheduled` (E-062)
- `EditalPublished` (E-063)
- `AgendaPublished` (E-064)
- `AssemblyStarted` (E-066)
- `AssemblyClosed` (E-073)
- `MinutesGenerated` / `MinutesPublished` (E-074, E-075)
- `HomologationOpened` / `HomologationApproved` (E-076)
- `TransparencyScoreCalculated` (E-077)

## Entities filhas (sub-aggregates de Assembly)

> Ratificado em 2026-04-25 (Fase E + decisões D-127, D-130, D-132). Repositórios dedicados por escala/query, mas invariantes delegadas a este aggregate root.

- [[AgendaItem]] — sub-aggregate (próprio AR pra Vote referenciar)
- [[Vote]] — entity de [[AgendaItem]] (D-130, antes era AR)
- [[AssemblyAdministratorLink]] — entity nova (D-127); cargo cedido pelo síndico, auto-invalida em `Assembly.End()`
- [[Proxy]] — sub-aggregate (procuração 100% digital — ASM-010)
- [[AttendanceRecord]] — entity (check-in 3 modalidades — ASM-012)
- (futuro Fase E) `AssemblyDirectorship` — snapshot mesa diretora (presidente + secretários + administradora + auditor + fornecedores) — ASM-035

## Métodos de domínio adicionais (Fase E)

- `GrantAdministrator(grantee, capabilities) → AssemblyAdministratorLink` — D-127
- `RevokeAdministrator(linkID, reason)` — D-127
- `RegisterAuditor(user) → Membership efêmera` — D-132 (TTL = `endedAt`)
- `ActivateContingency(reason, itemID?)` — ASM-022/042 (capability check)
- `DeactivateContingency()` — quando situação normalizada
- `CascadeInvalidateOnEnd()` — chamado por `End()`; cascata em todos `AssemblyAdministratorLink` + Memberships efêmeras + Proxies (após +24h via job scheduled)

## Links

- [[../bounded-contexts#6-assembly]]
- [[../invariants#assembly-inv-071-a-inv-085]]
- [[../state-machines#3-assembly]]
- [[../business-rules]] BR-041..BR-052
- [[AgendaItem]] · [[Vote]] · [[Minutes]] · [[LiveSession]] · [[Proxy]] · [[AttendanceRecord]] · [[AssemblyAdministratorLink]]
- [[../../STATE]] — D-127, D-130, D-132, D-133
