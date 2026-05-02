---
title: Aggregate — CompanyEvaluation
type: spec
tags: [domain, ddd, aggregate, commercial, reputation, master-sindico, v2]
bc: commercial
source: 90-ingested/.../specs/requirements/commercial.md Req 26
created: 2026-04-23
updated: 2026-04-23
---

# Aggregate — `CompanyEvaluation`

**BC**: commercial · **Raiz**: ✅

Avaliação formal pós-contrato. **Anônima ao avaliado** (empresa nunca vê `evaluator_user_id` — INV-044). Alimenta fórmula de Reputação (INV-036).

5 critérios escala 1-10 (Req 26):

1. Cumprimento de prazos
2. Qualidade do trabalho
3. Comunicação
4. Profissionalismo
5. Adequação de custos

## Entidade raiz

```go
type CompanyEvaluation struct {
    id                  EvaluationID
    companyID           CompanyID
    evaluatorUserID     UserID                  // síndico (nunca exposto pra empresa)
    condominiumID       CondominiumID
    dealID              DealID
    criteria            EvaluationCriteria      // 5 notas 1-10
    comment             string
    flags               []EvaluationFlag        // pontualidade, preço-justo, comunicação
    submittedAt         time.Time
    createdAt           time.Time
}

type EvaluationCriteria struct {
    cumprimentoPrazos     int  // 1-10
    qualidadeTrabalho     int
    comunicacao           int
    profissionalismo      int
    adequacaoCustos       int
}
```

## Value Objects

- `EvaluationID`, `EvaluationFlag` (enum)
- `EvaluationCriteria` (VO com validação cada campo 1-10)

## Invariantes

- **INV-027**: UNIQUE(company_id, evaluator_user_id, condominium_id) — 1 avaliação única por trio (DR-006; A-029 legado fix)
- **INV-044**: anônima — API response mapper filtra `evaluator_user_id` pra empresa
- Cada critério ∈ [1, 10] (CHECK)
- Só disponível após `deal.state = completed` (BR-027)

## Factories

```go
func NewCompanyEvaluation(companyID CompanyID, evaluatorUID UserID, condoID CondominiumID, dealID DealID, criteria EvaluationCriteria, comment string, flags []EvaluationFlag) (*CompanyEvaluation, error) {
    // valida: cada critério 1-10; deal pertence ao condo; deal.state=completed
}

func ReconstructCompanyEvaluation(...) *CompanyEvaluation
```

## Métodos de domínio

- (sem mutação pós-criação — imutável)
- `AverageScore() float64` — média dos 5 critérios; input pra Reputação

## Repository interface

```go
type ICompanyEvaluationRepository interface {
    Save(ctx context.Context, e *CompanyEvaluation) error                   // com ErrConflict em dup
    FindByID(ctx context.Context, id EvaluationID) (*CompanyEvaluation, error)
    ListByCompany(ctx context.Context, cid CompanyID) ([]*CompanyEvaluation, error)
    ListByEvaluator(ctx context.Context, uid UserID) ([]*CompanyEvaluation, error)
    AverageByCompany(ctx context.Context, cid CompanyID) (float64, int, error)    // (avg, count) — input Reputação
}
```

## Eventos emitidos

- `CompanyEvaluationSubmitted` (E-040) — dispara `ReputationCalculator.Recompute(companyID)` dentro UoW

## Links

- [[../bounded-contexts#4-commercial]]
- [[../invariants#commercial-inv-036-a-inv-055]]
- [[../business-rules]] BR-027, BR-028
- [[Company]] · [[ClosedDeal]]
