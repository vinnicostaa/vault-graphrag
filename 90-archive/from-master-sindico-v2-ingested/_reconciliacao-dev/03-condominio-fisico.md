---
title: Reconciliação — Condomínio físico (tipos, blocos, unidades, fração ideal)
type: note
tags: [reconciliation, institutional, condominio, blocos, unidades, master-sindico, v2, fase-7]
source: Development/backend/internal/modules/institutional/* + backend/migrate/migrations/* + Content/04-Listas-Mestre/Enums-Consolidados
created: 2026-04-23
updated: 2026-04-23
---

# Reconciliação — Modelagem Física do Condomínio

> **Origem**: Agent Explore (Fase 7d) leu código real em `backend/internal/modules/institutional/domain/entities/{condominium.go,unit.go,membership.go}`, migrations SQL, e `Enums-Consolidados.md`. Escopo: tipos de condomínio, blocos/torres, unidades, fração ideal.

---

## § 1. Tipos de Condomínio — 6 canônicos (enum)

**Código**: `backend/internal/modules/institutional/domain/enums/enums.go` + `00200_create_condominium_enums.sql`

```go
type CondominiumType string

const (
    CondominiumTypeResidencial       = "residencial"
    CondominiumTypeComercial         = "comercial"
    CondominiumTypeMisto             = "misto"
    CondominiumTypeShoppingCenter    = "shopping_center"
    CondominiumTypeGaleriaComercial  = "galeria_comercial"
    CondominiumTypeEdificioGaragem   = "edificio_garagem"
)

func (t CondominiumType) IsValid() bool { /* switch 6 casos */ }
```

**Implementação**:
- PostgreSQL ENUM nativo: `CREATE TYPE condominium_type AS ENUM (...)`
- CHECK constraint na tabela
- **6 tipos fechados** — nenhum outro tipo permitido

**Implicações operacionais**:
- Tipo define padrões de unidade (residencial = apto/casa; comercial = sala/loja; edificio_garagem = vaga)
- Tipo impacta regras de votação em Assembleia (misto tem pauta dupla — Req 48-60)
- Tipo restringe categorias de empresa contratável

**v2 status**: ✅ Cobertura completa.

---

## § 2. Estrutura física — Condominium (Aggregate raiz)

### Campos de entidade:

```go
type Condominium struct {
    id              string       // UUID primário
    publicID        string       // 8 chars [A-Z0-9] UNIQUE (identificador human-readable)
    name            string       // 2-200 chars
    condoType       enums.CondominiumType
    address         value_objects.Address
    totalUnits      int          // 1-10000
    blocksOrTowers  int          // 0-500 (0 = sem estrutura em blocos)
    createdBy       string       // user_id do síndico fundador
    createdAt       time.Time
    updatedAt       time.Time
    deletedAt       *time.Time   // soft delete
}
```

### Value Object: `Address`

```go
type Address struct {
    cep          string   // 8 dígitos, validado ^[0-9]{8}$
    street       string   // 2-200 chars
    number       string   // 1-20 chars
    complement   string   // ≤100 chars (opcional)
    neighborhood string   // 2-100 chars
    city         string   // 2-100 chars
    state        string   // 2 letras maiúsculas (UF BR)
}
```

### Invariantes ✅:
- `INV-030`: publicID UNIQUE 8 chars
- `INV-027`: ≤ 15 condomínios por síndico (Decision 11)
- `INV-033`: Audit trail retenção 5 anos (planejado)

### Factories: `NewCondominium`, `ReconstructCondominium`

Métodos de domínio: apenas getters. Mudança via Timeline + Amendments (Req 25).

**v2 status**: ✅ Alinhado com código.

---

## § 3. Bloco / Torre / Edifício

**Achado crítico**: **NÃO há entidade separada para Bloco/Torre.**

**Implementação atual**:
- Campo `block string` em Unit (nullable, max 20 chars)
- Valor **livre** — cada condomínio define convenção ("A", "Bloco A", "Torre 1", "Ala Sul")
- Sem validação/enum de tipos de bloco
- Campo `blocks_or_towers int` em Condominium é **apenas count** (denormalizado)

**Diferença v2**: v2 spec propõe entidade `Block` separada (BlockID, tipo enum, units[], fração ideal agregada).

**Recomendação**: manter simples no MVP (sem entidade separada); adicionar entidade Block só se UI precisar filtrar/reportar por bloco físico específico (M3+).

---

## § 4. Unidade — ⚠️ DIVERGÊNCIA CRÍTICA M1

### Campos de entidade (código real):

```go
type Unit struct {
    id            string
    condominiumID string       // FK Condominium
    unitNumber    string       // 1-20 chars
    block         string       // nullable, livre text
    floor         *int         // nullable, ≥ 0
    active        bool         // soft delete via false
    createdAt     time.Time
    updatedAt     time.Time
}
```

**Constraint**: `UNIQUE(condominium_id, unit_number, block)`.

### ❌ FALHA CRÍTICA 1: Tipo de Unidade AUSENTE

**Canônico** (Enums-Consolidados.md §3, Req 9):
```
UnitType: apartamento | casa | sala_comercial | loja | garagem | outro
```

**Código atual**:
- ❌ Não há campo `unitType` em `Unit struct`
- ❌ Não há enum em `enums.go`
- ❌ Não há coluna `unit_type` em migration `00202_create_units_memberships.sql`

**v2 precisa adicionar**:
```go
type Unit struct {
    // ... campos existentes ...
    unitType    UnitType  // NEW
}

type UnitType string
const (
    UnitTypeApartamento     = "apartamento"
    UnitTypeCasa            = "casa"
    UnitTypeSalaComercial   = "sala_comercial"
    UnitTypeLoja            = "loja"
    UnitTypeGaragem         = "garagem"
    UnitTypeOutro           = "outro"
)
```

**Mapeamento com CondominiumType**:
- `residencial` → {apartamento, casa, outro}
- `comercial` → {sala_comercial, loja, outro}
- `shopping_center` → {loja, sala_comercial, outro}
- `edificio_garagem` → {garagem} obrigatoriamente

### ❌ FALHA CRÍTICA 2: Fração Ideal AUSENTE

**Canônico** (Req 9, T10):
```
fracao_ideal: NUMERIC(5,4) ∈ (0, 1.0000]
Σ frações por condomínio ≤ 1.0000
```

**Código atual**: zero menção a `fracao_ideal` em `unit.go` ou migrations.

**Impacto**: **BLOCKER M1** — votação em Assembleia usa fração ideal como peso. Sem o campo, peso é implicitamente igual (1/n). **Viola Lei do Condomínio BR**.

**v2 precisa**:
```go
import "github.com/shopspring/decimal"

type Unit struct {
    // ...
    fracaoIdeal decimal.Decimal  // NUMERIC(5,4) no PG
}
```

**Validação**:
- `fracaoIdeal ∈ (0, 1.0000]`
- Query: `SELECT SUM(fracao_ideal) FROM units WHERE condominium_id = ? AND active = true` ≤ 1.0000

**Trigger SQL**:
```sql
CREATE CONSTRAINT TRIGGER check_fracao_sum
AFTER INSERT OR UPDATE ON units
FOR EACH ROW
EXECUTE FUNCTION validate_fracao_ideal_sum();
```

---

## § 5. Membership (User ↔ Condominium ↔ Unit)

### Enum `membership_status`:

```go
const (
    MembershipStatusPendingConfirmation = "pending_confirmation"
    MembershipStatusActive              = "active"
    MembershipStatusEnded               = "ended"
)
```

### Enum `membership_end_reason`:

```go
const (
    MembershipEndMovedOut        = "moved_out"
    MembershipEndSoldUnit        = "sold_unit"
    MembershipEndContractEnded   = "contract_ended"
    MembershipEndRemovedBySyndic = "removed_by_syndic"
)
```

### Invariantes:
- `INV-029`: 1 titular + ≤ 2 dependentes por unidade (UNIQUE parcial + app logic)
- `INV-026`: 1 membership ativa por (user_id, condominium_id)
- `INV-107` (v2): DTO stricto — user_id/condominium_id nunca em body PATCH (corrigido na Fase 5 fix)

### Histórico:
- Soft delete: `status='ended'` + campos ended_at, end_reason, end_note
- Sem hard delete (auditoria 5 anos — Req 33)
- Transição: `pending_confirmation → active` (após assinar termo)

**v2 status**: ✅ Alinhado.

---

## § 6. Tipos de Unidade — spec vs v2

### Canônico consolidado (Enums-Consolidados.md §3, 3 primários):
- residencial | comercial | vaga_garagem

### Spec funcional Req 9 (6 tipos específicos):
- apartamento | casa | sala_comercial | loja | garagem | outro

### v2 expandido (6):
- apartamento | casa | sala_comercial | loja | garagem | outro

**Recomendação**: v2 adota 6 tipos (mais granular). Consolidado de 3 será deprecado.

---

## § 7. Fração Ideal — spec completa

### Tipo de dado:
```sql
fracao_ideal NUMERIC(5,4)
```

- 5 dígitos totais, 4 casas decimais
- Intervalo: (0, 99.9999]
- Exemplo: 0.2500 = 25%; 1.0000 = 100%
- **NUNCA float** (Calisthenics #4)

### Regra INV-021:
```
Σ fracao_ideal de todas as Units ativas de um Condomínio ≤ 1.0000
```

### Precisão em condomínios 1000+ unidades:
- NUMERIC PostgreSQL oferece precisão arbitrária
- 1000 units com fracao_ideal=0.0010 cada = exatamente 1.0000
- Arredondamento em display (UI): 4 casas decimais nativo

### Ausência no código:
**CRÍTICO**: Campo não existe em `unit.go` nem em migration. **v2 deve adicionar IMEDIATAMENTE em M1**.

---

## § 8. Áreas Comuns (Common Areas)

**Status**: **NÃO modelado como entidade separada.**

Spec canônica (Req 12, Enums-Consolidados §4): 26 áreas impactadas (estrutura_predial, sistema_hidraulico, ..., outros).

**Implementação atual**:
- Sem enum para área/common_area
- Sem tabela separada
- Modeladas como **categorias de atividade no Master Plan** (Req 12)

**v2 vs atual**: sem mudança necessária. Modelo atual via Master Plan é suficiente se UI não precisar CRUD específico.

---

## § 9. Vagas de Garagem

**Modelagem**: como Unit comum, não entidade separada.

```go
unit := Unit{
    unitNumber: "G-101",
    block: "Garagem A",
    floor: nil,
    // unit_type: "garagem" (quando implementado)
}
```

**Características**:
- `unit_type = "garagem"` (quando adicionado)
- Fração ideal geralmente pequena (0.0001 a 0.0050)
- **Sem vínculo explícito com Unit residencial** no código atual

**v2 proposta**: adicionar `primary_unit_id` (nullable) em Unit para vincular vaga a apartamento (M2+).

---

## § 10. Migrations SQL reais

### 00200: Enums institucionais
```sql
CREATE TYPE condominium_type AS ENUM (
    'residencial', 'comercial', 'misto',
    'shopping_center', 'galeria_comercial', 'edificio_garagem'
);

CREATE TYPE mandate_type AS ENUM (
    'sindico_eleito', 'sindico_profissional', 'sindico_interino'
);

CREATE TYPE mandate_status AS ENUM ('scheduled', 'active', 'ended');
```

### 00201: Condominiums + SyndicMandates
```sql
CREATE TABLE condominiums (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    public_id TEXT NOT NULL UNIQUE
        CHECK (length(public_id) = 8 AND public_id ~ '^[A-Z0-9]{8}$'),
    name TEXT NOT NULL CHECK (length(trim(name)) BETWEEN 2 AND 200),
    condominium_type condominium_type NOT NULL,
    cep TEXT NOT NULL CHECK (cep ~ '^[0-9]{8}$'),
    street TEXT NOT NULL CHECK (length(trim(street)) BETWEEN 2 AND 200),
    -- ... demais campos address ...
    total_units INTEGER NOT NULL CHECK (total_units > 0 AND total_units <= 10000),
    blocks_or_towers INTEGER NOT NULL CHECK (blocks_or_towers >= 0 AND blocks_or_towers <= 500),
    created_by UUID NOT NULL REFERENCES users(id),
    created_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW(),
    updated_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW(),
    deleted_at TIMESTAMP WITH TIME ZONE
);

CREATE UNIQUE INDEX uq_syndic_mandates_active_principal
    ON syndic_mandates(condominium_id)
    WHERE status = 'active' AND role_type = 'principal';
```

### 00202: Units + Memberships
```sql
CREATE TABLE units (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    condominium_id UUID NOT NULL REFERENCES condominiums(id) ON DELETE CASCADE,
    unit_number TEXT NOT NULL CHECK (length(trim(unit_number)) BETWEEN 1 AND 20),
    block TEXT CHECK (block IS NULL OR length(trim(block)) BETWEEN 1 AND 20),
    floor INTEGER CHECK (floor IS NULL OR floor >= 0),
    -- ⚠️ FALTA: unit_type + fracao_ideal
    active BOOLEAN NOT NULL DEFAULT true,
    created_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW(),
    updated_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW(),
    UNIQUE (condominium_id, unit_number, block)
);

CREATE TABLE memberships (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    unit_id UUID NOT NULL REFERENCES units(id) ON DELETE CASCADE,
    user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    role_type user_role_type NOT NULL CHECK (role_type IN ('titular', 'dependent')),
    status membership_status NOT NULL DEFAULT 'pending_confirmation',
    consent_signed_at TIMESTAMP WITH TIME ZONE,
    ended_at TIMESTAMP WITH TIME ZONE,
    end_reason membership_end_reason,
    end_note TEXT,
    created_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW(),
    updated_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW()
);

CREATE UNIQUE INDEX uq_memberships_active_titular
    ON memberships(unit_id)
    WHERE status IN ('pending_confirmation', 'active')
    AND role_type = 'titular';
```

---

## § 11. Divergências v2 vs código — Checklist

### ✅ Implementado corretamente (6):

1. 6 tipos de condomínio
2. Endereço (Value Object com validação CEP/UF)
3. Condominium base (todos campos)
4. Unit básico (unitNumber, block, floor)
5. Membership (estados, tipos, motivos encerramento)
6. Soft delete + histórico (sem hard delete, 5 anos audit)

### ❌ Ausências críticas (v2 precisa adicionar):

| # | Gap | Local | Impacto | Prioridade |
|---|---|---|---|---|
| 1 | `unit_type` enum em Unit | entity + migration | Sem info apto/casa/sala/loja/vaga | **M1** |
| 2 | `fracao_ideal` NUMERIC(5,4) | entity + migration | **Votação Assembleia quebrada** | **M1** |
| 3 | Validação Σ fracao ≤ 100% | migration trigger | Sem garantia de consistência | **M1** |
| 4 | Tipos de bloco enum | — | Só string livre; sem validação | M3 |
| 5 | Common areas entidade | — | Apenas via Master Plan | M3 |
| 6 | Vínculo vaga → unidade | Unit entity | Vagas soltas | M2 |

---

## § 12. Recomendações imediatas para M1

```go
// em domain/entities/unit.go
type Unit struct {
    id            string
    condominiumID string
    unitNumber    string
    block         string
    floor         *int
    unitType      UnitType          // NEW
    fracaoIdeal   decimal.Decimal   // NEW — NUMERIC(5,4)
    primaryUnitID *string           // OPTIONAL — vagas vinculadas (M2+)
    active        bool
    createdAt     time.Time
    updatedAt     time.Time
}

// em domain/enums/enums.go
type UnitType string

const (
    UnitTypeApartamento     UnitType = "apartamento"
    UnitTypeCasa            UnitType = "casa"
    UnitTypeSalaComercial   UnitType = "sala_comercial"
    UnitTypeLoja            UnitType = "loja"
    UnitTypeGaragem         UnitType = "garagem"
    UnitTypeOutro           UnitType = "outro"
)

func (u UnitType) IsValid() bool { /* switch */ }

// em NewUnit
func NewUnit(
    condoID string,
    identifier string,
    unitType UnitType,
    fracaoIdeal decimal.Decimal,
    // ...
) (*Unit, error) {
    // Valida: unitType válido p/ condoType
    // Valida: fracaoIdeal ∈ (0, 1.0000]
}
```

### Migration nova:
```sql
-- 00203_add_unit_type_fracao_ideal.sql
CREATE TYPE unit_type AS ENUM (
    'apartamento', 'casa', 'sala_comercial',
    'loja', 'garagem', 'outro'
);

ALTER TABLE units
    ADD COLUMN unit_type unit_type NOT NULL DEFAULT 'apartamento',
    ADD COLUMN fracao_ideal NUMERIC(5,4) NOT NULL DEFAULT 0.0001
        CHECK (fracao_ideal > 0 AND fracao_ideal <= 1.0000);

-- Backfill: recalcular fracao_ideal proporcional pra condomínios existentes
-- (operação manual por condomínio, com aprovação do síndico)

CREATE OR REPLACE FUNCTION validate_fracao_ideal_sum()
RETURNS TRIGGER AS $$
DECLARE total NUMERIC;
BEGIN
    SELECT COALESCE(SUM(fracao_ideal), 0) INTO total
    FROM units
    WHERE condominium_id = NEW.condominium_id AND active = true;

    IF total > 1.0000 THEN
        RAISE EXCEPTION 'Fração ideal total excede 100%% no condomínio %', NEW.condominium_id;
    END IF;
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE CONSTRAINT TRIGGER check_fracao_ideal_sum
AFTER INSERT OR UPDATE ON units
FOR EACH ROW
EXECUTE FUNCTION validate_fracao_ideal_sum();
```

---

## § 13. Conclusão

**Código atual modela bem estrutura estática** (condomínio, unidades básicas, memberships) **mas falha na diferenciação de tipos e dados de votação**:

1. ❌ **Tipos de unidade ausentes** → impossível validar que apartamento só tem moradores, loja só comerciantes
2. ❌ **Fração ideal ausente** → **votação Assembleia quebrada** (peso padrão implícito 1/n)
3. ❌ **Validação de soma de frações ausente** → sem garantia Σ ≤ 100%

**v2 deve corrigir IMEDIATAMENTE em M1** antes de qualquer feature de Assembleia ou votação ir production-ready.
