---
title: Aggregate — Partner (Vizinhança)
type: spec
tags: [domain, ddd, aggregate, commercial, neighborhood, master-sindico, v2]
bc: commercial
source: 90-ingested/.../specs/requirements/commercial.md Req 27 + institutional.md Req 6.4
created: 2026-04-23
updated: 2026-04-23
---

# Aggregate — `Partner` (Vizinhança)

**BC**: commercial · **Raiz**: ✅ · **Tenant próprio** (`tenant_type=company, is_neighborhood=true`)

Comércio local (pizzaria, petshop, farmácia, academia). Vende **ao morador como consumidor final**. Não confunde com `Company` (presta serviço ao condomínio).

CNPJ opcional — aceita CPF (MEI sem CNPJ). Cadastro simplificado (Req 6.4).

## Entidade raiz

```go
type Partner struct {
    id                  PartnerID
    userID              UserID                  // admin do Partner
    document            Document                // CPF OR CNPJ (union VO)
    razaoSocialNome     string
    nomeFantasia        string
    category            PartnerCategory         // enum pt-BR: pizzaria, petshop, farmacia, academia, ... (lista ampla)
    neighborhood        string
    city                string
    state               string                  // UF
    cep                 string
    address             *AddressBR
    radiusKm            float64                 // raio atendimento default 2km
    phone               Phone
    whatsapp            *string
    instagram           *string
    photos              []string                // máx 3
    descricao           string
    sealSindicoApproved bool                    // concedido por síndicos com plan_tier ∈ {plus, pro} (D-067/D-081/D-096)
    sealGrantedBy       []UserID                // quais síndicos concederam
    status              PartnerStatus           // pending_verification | active | suspended
    createdAt           time.Time
    updatedAt           time.Time
}
```

## Value Objects

- `PartnerID`, `PartnerCategory`, `PartnerStatus` (enums)
- `Document` (CPF OR CNPJ union)
- `AddressBR`

## Invariantes

- Máximo 3 fotos na galeria (CHECK `array_length(photos) <= 3`)
- Se document tipo CNPJ → UNIQUE; se CPF → UNIQUE
- Raio ∈ [0.1, 20] km

## Factories

```go
func NewPartner(userID UserID, doc Document, razao, fantasia string, category PartnerCategory, city, state, cep string, phone Phone) (*Partner, error) {
    // status=pending_verification, sealSindicoApproved=false
}

func ReconstructPartner(...) *Partner
```

## Métodos de domínio

- `Approve()` — `pending_verification → active`
- `Suspend(reason string)` — `active → suspended`
- `AddPhoto(url string) error` — valida ≤ 3
- `RemovePhoto(url string) error`
- `GrantSealBySindico(syndicID UserID) error` — adiciona em `sealGrantedBy`; se primeira → `sealSindicoApproved=true`
- `RevokeSealBySindico(syndicID UserID) error` — remove; se vazio → `sealSindicoApproved=false`

## Repository interface

```go
type IPartnerRepository interface {
    Save(ctx context.Context, p *Partner) error
    FindByID(ctx context.Context, id PartnerID) (*Partner, error)
    FindByUserID(ctx context.Context, uid UserID) (*Partner, error)
    ListByNeighborhood(ctx context.Context, neighborhood string, city string) ([]*Partner, error)
    ListByCEPRadius(ctx context.Context, cep string, radiusKm float64) ([]*Partner, error)   // busca geo
    ListBySeal(ctx context.Context, seal bool) ([]*Partner, error)
}
```

## Eventos emitidos

- (Partner não emite eventos importantes cross-BC; cupons via `NeighborhoodBenefit` aggregate separado — não modelado aqui pra manter minúcia; ver domain-events E-046, E-047)

## Links

- [[../bounded-contexts#4-commercial]]
- [[../business-rules]] BR-064..BR-068
- [[Company]] · [[../aggregates/User]]
