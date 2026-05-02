---
title: Aggregate — Unit
type: spec
tags:
  - domain
  - ddd
  - aggregate
  - institutional
  - master-sindico
  - v2
bc: institutional
source: 90-ingested/.../specs/requirements/institutional.md Req 9
created: 2026-04-23T00:00:00.000Z
updated: '2026-04-25'
---

# Aggregate — `Unit`

**BC**: institutional · **Raiz**: ✅ (modelagem pragmática — tratamos como AR próprio porque é referenciado standalone por `Vote` em `assembly`; conceitualmente é sub de `Condominium`)

Unidade habitacional/comercial do condomínio. Carrega fração ideal (peso de voto) e relação com titular + dependentes.

## Entidade raiz

```go
type Unit struct {
    id              UnitID
    condominiumID   CondominiumID
    blockID         *BlockID                // ← NOVO (D-128 2026-04-25): NULLABLE; condomínio pequeno sem blocos
    unitIdentifier  string                  // "101", "Apto 501"
    unitType        UnitType                // apartamento | casa | sala_comercial | loja | garagem | outro
    fracaoIdeal     FracaoIdeal             // NUMERIC(5,4) ∈ (0, 1.0]
    titularID       *UserID                 // Morador principal
    dependents      []UserID                // até 2
    status          UnitStatus              // active | vacant | archived
    createdAt       time.Time
    updatedAt       time.Time
}

// D-128 (2026-04-25): coluna `block *string` deprecated; `block_id *BlockID` é canônico.
// Migration expand-contract:
//   1. ALTER unit ADD COLUMN block_id (release N)
//   2. backfill via job lendo unit.block string → criar Blocks + atualizar block_id (release N+0.1)
//   3. deprecate unit.block (release N+1; read-only)
//   4. DROP COLUMN block (release N+2)
```

## Atualização D-128 (2026-04-25)

> Bloco promovido a aggregate root próprio ([[Block]]). Unit ganha FK `block_id` **OPCIONAL** — condomínios pequenos (prédio único sem blocos) mantêm `block_id = NULL` e Unit pertence direto a Condominium.

## Value Objects

- `UnitID`, `UnitType` (enum pt-BR), `UnitStatus`
- `FracaoIdeal` — NUMERIC(5,4) ∈ (0, 100], 4 casas decimais (T10)

## Invariantes

- **INV-022**: `FracaoIdeal ∈ (0, 1.0]` (NUMERIC(5,4))
- **INV-023**: coluna NUMERIC(5,4); nunca float
- **INV-029**: 1 titular + até 2 dependentes (Req 9)
- **INV-021** (participante): Σ frações **por Condominium** ≤ 1.0 (NÃO por Block — D-128)
- **INV-INS-032** (D-128 2026-04-25): `block_id` **OPCIONAL** (NULLABLE) — Unit pode pertencer direto a Condominium sem Block intermediário
- **INV-INS-033** (D-128 2026-04-25): se `block_id IS NOT NULL`, então `Block.condominium_id = Unit.condominium_id` (consistência cross-FK; DB CHECK ou trigger)
- **INV-INS-034** (D-128): "Unidade vazia → dependentes auto-removidos" (Jornada Morador regra 5; trigger DB-enforced ao mudar `status=vacant`)

## Factories

```go
func NewUnit(condoID CondominiumID, identifier string, unitType UnitType, fracao FracaoIdeal) (*Unit, error) {
    // valida: fracao ∈ (0,100], identifier não vazio
}

func ReconstructUnit(...) *Unit
```

## Métodos de domínio

- `AssignTitular(userID UserID) error` — UPSERT; invalida titular anterior (MembershipEnded)
- `AddDependent(userID UserID) error` — valida `len(dependents) < 2`
- `RemoveDependent(userID UserID) error`
- `UpdateFracaoIdeal(f FracaoIdeal) error` — raro; via Amendment (adendo); publica `UnitUpdated`
- `Archive()` — soft delete

## Repository interface

```go
type IUnitRepository interface {
    Save(ctx context.Context, u *Unit) error
    FindByID(ctx context.Context, id UnitID) (*Unit, error)
    ListByCondominium(ctx context.Context, cid CondominiumID) ([]*Unit, error)
    ListByBlock(ctx context.Context, bid BlockID) ([]*Unit, error)                          // D-128
    ListByCondominiumWithoutBlock(ctx context.Context, cid CondominiumID) ([]*Unit, error)  // D-128: Units órfãs (sem block)
    FindByTitularAndCondominium(ctx context.Context, uid UserID, cid CondominiumID) ([]*Unit, error)
    SumFracaoIdealByCondominium(ctx context.Context, cid CondominiumID) (FracaoIdeal, error)
    HasActiveUnitsInBlock(ctx context.Context, bid BlockID) (bool, error)                    // D-128: Block.Archive() check
}
```

## Eventos emitidos

- `UnitRegistered` (E-015)
- `UnitUpdated` (E-016) — via adendo

## Links

- [[../bounded-contexts#3-institutional]]
- [[../invariants#institutional-inv-021-a-inv-035]]
- [[../business-rules]] BR-008 (Σ fração)
- [[Condominium]] · [[Membership]]
