---
title: Aggregate — Company
type: spec
tags: [domain, ddd, aggregate, commercial, master-sindico, v2]
bc: commercial
source: 90-ingested/.../specs/requirements/commercial.md + institutional.md Req 6.3
created: 2026-04-23
updated: 2026-04-23
---

# Aggregate — `Company`

**BC**: commercial · **Raiz**: ✅ · **Tenant próprio** (`tenant_type=company`)

Empresa cadastrada com CNPJ ativo. Presta serviço ao condomínio (portaria, limpeza, reforma, segurança). Distingue de `Partner` (Vizinhança — comércio local ao morador).

## Entidade raiz

```go
type Company struct {
    id                      CompanyID
    adminUserID             UserID                  // admin da empresa (1+ CompanyUser)
    cnpj                    CNPJ
    razaoSocial             string
    nomeFantasia            string
    dataAniversario         *time.Time
    representanteLegal      string
    emailComercial          Email
    telefoneComercial       Phone
    endereco                AddressBR
    categoriaPrincipalID    CategoriaID
    subcategoriaIDs         []CategoriaID           // ≤ 5
    complianceMarkers       []ComplianceMarker      // 25 autodeclarados
    possuiSeguroRespCivil   bool
    status                  CompanyStatus           // pending_verification | active | suspended | blocked
    reputationTier          ReputacaoTier           // Bronze | Prata | Ouro | Platina | Diamante
    suspendedFlag           bool                    // flag 72h harassment BR-031
    suspendedUntil          *time.Time
    logoURL                 string
    createdAt               time.Time
    updatedAt               time.Time
}
```

## Value Objects

- `CompanyID`, `CategoriaID`
- `CNPJ` (compartilhado)
- `AddressBR`
- `CompanyStatus`, `ReputacaoTier` (enum pt-BR)
- `ComplianceMarker` (enum: ISO 9001/14001/45001/37001/37301, ESG, NRs, selos)

## Invariantes

- **INV-003**: CNPJ UNIQUE (1 CNPJ = 1 Company)
- **INV-036**: Reputação é função determinística
- **INV-031** (participante): subcategorias ≤ 5; devem pertencer à categoria principal
- `possui_seguro_responsabilidade_civil` obrigatório pra Prata+
- Regras de conteúdo (Req 6.3): sem chamadas comerciais/preços/contato em conteúdo público

## Factories

```go
func NewCompany(adminUserID UserID, cnpj CNPJ, razao, fantasia string, email Email, phone Phone, endereco AddressBR, categoriaPrincipal CategoriaID) (*Company, error) {
    // valida: cnpj formato, categoria principal exists; status=pending_verification, tier=Bronze
}

func ReconstructCompany(...) *Company
```

## Métodos de domínio

- `Approve(by UserID)` — `pending_verification → active`
- `Suspend(reason string, until *time.Time)` — `active → suspended`
- `AddSubcategoria(catID CategoriaID) error` — valida limite 5
- `UpdateReputationTier(newTier ReputacaoTier, inputs ReputationInputs) error` — chamado por `ReputationCalculator.Recompute(...)`
- `ApplyHarassmentSuspension(until time.Time)` — seta `suspended=true` por 72h **sem** rebaixar tier (BR-031)
- `LiftHarassmentSuspension()` — reset flag

## Repository interface

```go
type ICompanyRepository interface {
    Save(ctx context.Context, c *Company) error
    FindByID(ctx context.Context, id CompanyID) (*Company, error)
    FindByCNPJ(ctx context.Context, cnpj CNPJ) (*Company, error)
    ListActiveByCategory(ctx context.Context, catID CategoriaID) ([]*Company, error)
    ListByAdminUser(ctx context.Context, uid UserID) ([]*Company, error)
}
```

## Eventos emitidos

- `CompanyCreated` (E-028)
- `CompanyApproved` (E-029)
- `CompanySuspended` (E-030)
- `ReputationTierChanged` (E-041)

## Links

- [[../bounded-contexts#4-commercial]]
- [[../invariants#commercial-inv-036-a-inv-055]]
- [[../state-machines#6-company-reputation-tier]]
- [[../business-rules]] BR-028..BR-032
- [[Partner]] · [[CompanyEvaluation]] · [[ConnectMeRequest]]
