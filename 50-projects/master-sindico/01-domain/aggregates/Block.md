---
title: Aggregate — Block
type: spec
tags: [domain, ddd, aggregate, institutional, master-sindico, v2, new-2026-04-25]
bc: institutional
source: STATE#D-128 (2026-04-25)
created: 2026-04-25
updated: 2026-04-25
status: proposed (M2 implementation)
---

# Aggregate — `Block`

**BC**: institutional · **Raiz**: ✅ (aggregate root próprio)

Bloco do condomínio. Conjunto de unidades agrupadas (ex: Bloco A, Torre 1, Bloco "Floricultura"). **OPCIONAL** — condomínios pequenos podem não ter blocos (Unit pertence direto a Condominium).

> Criado em 2026-04-25 via [[../../STATE#D-128]]. Antes era apenas `Unit.block *string` (campo opcional). Promovido a aggregate root próprio para suportar metadata variável e lifecycle independente (arquivar bloco mantendo resto do condomínio).

## Entidade raiz

```go
type Block struct {
    id              BlockID
    condominiumID   CondominiumID         // FK obrigatória
    identifier      string                // "A", "Torre 1", "Floricultura", "5"
    identifierType  IdentifierType        // letter | number | named
    description     *string               // opcional: "Frente do mar", "Lateral"
    floors          *int                  // opcional: número de andares
    yearBuilt       *int                  // opcional
    status          BlockStatus           // active | archived
    createdAt       time.Time
    updatedAt       time.Time
}

type IdentifierType string
const (
    Letter IdentifierType = "letter"  // "A", "B", "C"
    Number IdentifierType = "number"  // "1", "2", "Bloco 5"
    Named  IdentifierType = "named"   // "Floricultura", "Vista Mar"
)

type BlockStatus string
const (
    Active   BlockStatus = "active"
    Archived BlockStatus = "archived"
)
```

## Value Objects

- `BlockID` (UUIDv7)
- `IdentifierType` (enum 3 valores)
- `BlockStatus` (enum 2 valores)

## Invariantes

- **INV-INS-031** (nova): `identifier` não pode ser vazio nem duplicado dentro do mesmo `condominium_id` (UNIQUE `(condominium_id, identifier)`)
- **INV-INS-032** (nova): Block **OPCIONAL** — Unit pode pertencer direto a Condominium sem Block (`Unit.block_id NULL` permitido)
- **INV-INS-033** (nova): Block não pode ser arquivado com Unit ativa (`status='active'` bloqueia archive); DB-enforced via trigger
- **INV-INS-024** (existente, REFORÇADA): Σ FracaoIdeal **por Condominium** ≤ 100% (NÃO por Block — bloco não tem restrição de soma)

## Factories

```go
func NewBlock(condoID CondominiumID, identifier string, idType IdentifierType) (*Block, error) {
    // valida: identifier não vazio + UNIQUE no condomínio
}

func ReconstructBlock(...) *Block
```

## Métodos de domínio

- `UpdateMetadata(description *string, floors *int) error` — soft updates de metadata
- `Archive() error` — soft delete; falha se houver Unit `status=active` no bloco; emite `BlockArchived` (timeline imutável)
- `Restore() error` — reverter archive (raro, apenas admin)

## Repository interface

```go
type IBlockRepository interface {
    Save(ctx context.Context, b *Block) error
    FindByID(ctx context.Context, id BlockID) (*Block, error)
    ListByCondominium(ctx context.Context, cid CondominiumID) ([]*Block, error)
    FindByIdentifier(ctx context.Context, cid CondominiumID, identifier string) (*Block, error)
    HasActiveUnits(ctx context.Context, id BlockID) (bool, error)  // pra Archive() check
}
```

## Eventos emitidos

- `BlockCreated` (E-### a numerar)
- `BlockArchived` (E-### a numerar)
- `BlockMetadataUpdated` (E-### a numerar)

## Migrations (expand-contract — STEERING §41 + CLAUDE.md §12)

1. **Release N**: `CREATE TABLE blocks` + `ALTER unit ADD COLUMN block_id BlockID NULL`
2. **Release N+0.1**: backfill job lendo `unit.block` string → criar Blocks + atualizar `block_id`
3. **Release N+1**: deprecate `unit.block` (read-only no código; ainda no schema)
4. **Release N+2**: `ALTER unit DROP COLUMN block` (após auditoria 0 referências)

## Links

- [[../bounded-contexts#3-institutional]]
- [[../invariants#INV-INS-031]] (a criar) · [[../invariants#INV-INS-032]] · [[../invariants#INV-INS-033]]
- [[Condominium]] · [[Unit]] · [[Membership]]
- [[../../STATE#D-128]] (decisão original)
