---
title: Aggregate — Condominium
type: spec
tags: [domain, ddd, aggregate, institutional, master-sindico, v2]
bc: institutional
source: 90-ingested/.../specs/requirements/institutional.md + legacy design.md
created: 2026-04-23
updated: 2026-04-23
---

# Aggregate — `Condominium`

**BC**: institutional · **Raiz**: ✅ (**tenant primário** do sistema)

Conjunto de unidades + pessoas + timeline. Toda query escopada em `institutional`/`commercial`/`assembly` tem `WHERE condominium_id = $id` (INV-089 tenant isolation).

## Entidade raiz

```go
type Condominium struct {
    id              CondominiumID
    createdBy       UserID
    name            string
    cnpj            *CNPJ                   // opcional
    address         AddressBR
    condoType       CondominiumType         // residencial | comercial | misto | shopping_center | galeria_comercial | edificio_garagem
    totalUnits      int
    publicID        PublicID                // 8 chars alfanumérico UNIQUE
    qrCodeURL       string
    units           []Unit                  // loaded separadamente ou via join
    createdAt       time.Time
    updatedAt       time.Time
    deletedAt       *time.Time              // soft delete
}
```

## Value Objects

- `CondominiumID`, `PublicID`, `CondominiumType` (enum pt-BR)
- `AddressBR` (cep, logradouro, numero, complemento, bairro, cidade, uf)
- `CNPJ` (opcional)

## Invariantes

- **INV-021**: Σ `FracaoIdeal` de Units ≤ 100% (DR-001; CHECK + PBT)
- **INV-027**: ≤ 15 condomínios por síndico (Decision 11)
- **INV-030**: `public_id` UNIQUE
- **INV-033**: audit trail append-only retenção 5 anos

## Factories

```go
func NewCondominium(createdBy UserID, name string, address AddressBR, condoType CondominiumType, totalUnits int) (*Condominium, error) {
    // valida CEP, gera publicID 8 chars + QR; rejeita se createdBy já tem 15 condos
}

func ReconstructCondominium(...) *Condominium
```

## Métodos de domínio

- `AddUnit(u Unit) error` — valida `Σ fração <= 100%`
- `RemoveUnit(unitID UnitID) error` — raramente permitido; requer zero memberships
- `UpdateTotalUnits(n int) error`
- `TransferOwnership(newCreatedBy UserID) error`

## Repository interface

```go
type ICondominiumRepository interface {
    Save(ctx context.Context, c *Condominium) error
    FindByID(ctx context.Context, id CondominiumID) (*Condominium, error)
    FindByPublicID(ctx context.Context, pid PublicID) (*Condominium, error)
    ListBySyndic(ctx context.Context, syndicID UserID) ([]*Condominium, error)          // limite 15
    CountBySyndic(ctx context.Context, syndicID UserID) (int, error)                    // guard INV-027
    SumFracaoIdeal(ctx context.Context, id CondominiumID) (FracaoIdeal, error)          // validação INV-021
}
```

## Eventos emitidos

- `CondominiumCreated` (E-014)
- `UnitRegistered` / `UnitUpdated` (E-015, E-016)
- (indireto via Unit/Timeline/MasterPlan)

## Links

- [[../bounded-contexts#3-institutional]]
- [[../invariants#institutional-inv-021-a-inv-035]]
- [[../business-rules]] BR-006..BR-012
- [[Unit]] · [[Membership]] · [[TimelineEntry]] · [[MasterPlan]]
