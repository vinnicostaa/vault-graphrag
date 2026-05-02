---
title: "Aggregate — Plan"
type: spec
tags: [aggregate, ddd, billing, master-sindico, v2, fase-8-consolidado]
source: ADR-0032 + STATE D-066/D-067 + backend/internal/modules/billing/domain/entities/plan.go + requirements/billing.md
created: 2026-04-23
updated: 2026-04-23
status: consolidado-fase-8
dual_check: pending
---

# Aggregate — `Plan`

**BC**: billing · **Raiz**: ✅ (catálogo global Stripe-style, editável via painel admin MS)

## § Propósito

`Plan` é a **entidade global** do catálogo de assinaturas do Master Síndico. Sob ADR-0032 (**Planos Globais Stripe-style**), `Plan` deixa de ser "plano por persona" e passa a ser **catálogo universal** — qualquer `User` (independente de `role`) pode assinar qualquer `Plan` ativo. Permissões e quotas derivam do par `(user.role, plan.tier, feature)` aplicado à matriz ABAC em runtime — **não da identidade do plano**.

Inspiração direta: **Stripe Products + Prices**. O `Plan` equivale ao par (Product, Price) no Stripe: `tier` é a categoria de serviço; `name` / `label_ui` são o rótulo comercial; `stripe_price_id` faz a ponte com o provider externo.

**Decisões-raiz**:
- **D-066** (STATE Fase 8 linha 549): planos são globais, personas carregam matriz
- **D-067** (STATE Fase 8 linha 569): slugs compostos (`syndic_n1`, `enterprise_plus`, `resident_base`, ...) **abolidos** em código, banco, UI e marketing
- **ADR-0032**: enum canônico `plan_tier ∈ {trial, base, plus, pro}` — único em todo o sistema

## § Entidade raiz (campos + tipos + constraints)

```go
// internal/modules/billing/domain/entities/plan.go
type Plan struct {
    id              PlanID          // UUIDv7 PK
    tier            PlanTier        // enum: trial | base | plus | pro  (UNIVERSAL — ADR-0032)
    name            string          // identificador comercial interno, ex: "Plano Síndico Plus"
    labelUI         string          // rótulo exibido ao usuário (i18n-friendly), ex: "Plano Profissional"
    description     string          // descrição longa (marketing copy)
    features        FeatureFlags    // JSONB — feature_key → bool
    quotas          QuotaLimits     // JSONB — feature_key → {limit int64, period QuotaPeriod, unlimited bool}
    priceCents      int64           // Money — centavos BRL; trial pode ser 0 (sem cobrança)
    currency        string          // sempre "BRL" no M1
    stripePriceID   string          // price_1ABCxyz — lookup reverso no webhook; UNIQUE quando active=true
    trialDays       int             // >= 0; trial oferecido ao iniciar Subscription (persona-dependente: cálculo fica em TrialRules)
    billingCycle    BillingCycle    // enum: monthly | annual
    active          bool            // soft-disable; inativo NÃO pode ser assinado mas subscriptions existentes permanecem
    createdAt       time.Time
    updatedAt       time.Time
}
```

**Constraints SQL** (ver § Migrações):
- `id UUID PK DEFAULT gen_random_uuid_v7()`
- `tier plan_tier_enum NOT NULL` (enum PG: `CREATE TYPE plan_tier AS ENUM ('trial','base','plus','pro')`)
- `price_cents BIGINT NOT NULL CHECK (price_cents >= 0)`
- `currency CHAR(3) NOT NULL DEFAULT 'BRL'`
- `stripe_price_id TEXT` (NULL quando `active=false` recém-arquivado; UNIQUE parcial `WHERE active=true`)
- `trial_days INT NOT NULL DEFAULT 0 CHECK (trial_days >= 0)`
- `billing_cycle billing_cycle_enum NOT NULL`
- `features JSONB NOT NULL DEFAULT '{}'`
- `quotas JSONB NOT NULL DEFAULT '{}'`
- `active BOOLEAN NOT NULL DEFAULT true`

## § Value Objects

```go
// ------- PlanID -------
type PlanID string // UUIDv7 (ADR-0009)

func NewPlanID() PlanID              { return PlanID(uuidv7.New()) }
func (id PlanID) String() string     { return string(id) }
func (id PlanID) IsValid() bool      { return uuidv7.IsValid(string(id)) }

// ------- PlanTier (enum canônico — ADR-0032) -------
type PlanTier string

const (
    PlanTierTrial PlanTier = "trial"
    PlanTierBase  PlanTier = "base"
    PlanTierPlus  PlanTier = "plus"
    PlanTierPro   PlanTier = "pro"
)

func (t PlanTier) IsValid() bool {
    switch t {
    case PlanTierTrial, PlanTierBase, PlanTierPlus, PlanTierPro:
        return true
    }
    return false
}

// Hierarquia: trial < base < plus < pro (base ⊂ plus ⊂ pro em features)
func (t PlanTier) Rank() int {
    switch t {
    case PlanTierTrial: return 0
    case PlanTierBase:  return 1
    case PlanTierPlus:  return 2
    case PlanTierPro:   return 3
    }
    return -1
}

// ------- Price -------
type Price struct {
    Cents    int64
    Currency string // "BRL"
}

func NewPrice(cents int64, currency string) (Price, error) {
    if cents < 0                  { return Price{}, ErrNegativePrice }
    if currency != "BRL"          { return Price{}, ErrUnsupportedCurrency }
    return Price{Cents: cents, Currency: currency}, nil
}

func (p Price) IsZero() bool { return p.Cents == 0 }

// ------- FeatureFlags -------
type FeatureKey string

const (
    FeatKeyConnectMe                FeatureKey = "connect_me"
    FeatKeyVideoUploadInstitutional FeatureKey = "video_upload_institutional"
    FeatKeyVideoUploadCurriculum    FeatureKey = "video_upload_curriculum"
    FeatKeyLiveSession              FeatureKey = "live_session"
    FeatKeyEbookDownload            FeatureKey = "ebook_download"
    FeatKeyAssemblyCreate           FeatureKey = "assembly.create"
    FeatKeyReportsExport            FeatureKey = "reports.export"
)

type FeatureFlags map[FeatureKey]bool

func (ff FeatureFlags) Has(k FeatureKey) bool { return ff[k] }

// ------- QuotaLimits -------
type QuotaPeriod string

const (
    QuotaPeriodMonthly QuotaPeriod = "monthly"
    QuotaPeriodAnnual  QuotaPeriod = "annual"
    QuotaPeriodOnce    QuotaPeriod = "once"
    QuotaPeriodNever   QuotaPeriod = "never"
)

type QuotaRule struct {
    Limit     int64       // -1 (QuotaUnlimited) | 0 (indisponível) | N > 0
    Period    QuotaPeriod
    Unlimited bool        // redundante com Limit == -1; mantido pra clareza no JSON
}

type QuotaLimits map[FeatureKey]QuotaRule

const QuotaUnlimited int64 = -1

// ------- BillingCycle -------
type BillingCycle string

const (
    BillingCycleMonthly BillingCycle = "monthly"
    BillingCycleAnnual  BillingCycle = "annual"
)
```

## § Invariantes (testáveis)

| ID | Regra | Onde é testado |
|---|---|---|
| **INV-PLN-001** | `price_cents >= 0` (trial pode ser 0) | factory `NewPlan` + CHECK SQL |
| **INV-PLN-002** | Toda `QuotaRule.Limit != 0` respeita `>= -1` (`-1` = unlimited) | factory + PBT fuzz |
| **INV-PLN-003** | **Coerência hierárquica**: se `tier == plus` então features do `base` ⊂ features do plano; se `tier == pro` então `plus` ⊂ `pro` | test `TestPlan_TierHierarchy` |
| **INV-PLN-004** | `stripe_price_id` é UNIQUE entre planos `active=true` | migration (UNIQUE parcial) |
| **INV-PLN-005** | `tier` ∈ enum canônico — strings `"syndic_n1"`, `"enterprise_plus"`, etc. **REJEITADAS** pela factory | `PlanTier.IsValid()` + test-guard |
| **INV-PLN-006** | `trial_days >= 0`; quando `tier == trial`, `price_cents` DEVE ser 0 | factory |
| **INV-PLN-007** | Plano com `active=false` não pode ser destino de `NewTrialSubscription` (use case checa) | use case `CheckoutSubscriptionSaga` |
| **INV-PLN-008** | `currency == "BRL"` em M1 (multi-moeda fica pra M3+) | factory |
| **INV-PLN-009** | `billing_cycle` ∈ {monthly, annual} — sem biweekly/custom em M1 | enum SQL + factory |

## § Factories (New... / Reconstruct...)

```go
// NewPlan — factory oficial usada em seed admin + futura UI de catálogo.
// Valida todas invariantes INV-PLN-001 a INV-PLN-009 sincronamente.
func NewPlan(
    tier PlanTier,
    name, labelUI, description string,
    price Price,
    trialDays int,
    cycle BillingCycle,
    features FeatureFlags,
    quotas QuotaLimits,
    stripePriceID string,
) (*Plan, error) {
    if !tier.IsValid()              { return nil, ErrInvalidTier }
    if name == ""                   { return nil, ErrEmptyName }
    if labelUI == ""                { return nil, ErrEmptyLabel }
    if price.Currency != "BRL"      { return nil, ErrUnsupportedCurrency }
    if trialDays < 0                { return nil, ErrNegativeTrialDays }
    if tier == PlanTierTrial && !price.IsZero() {
        return nil, ErrTrialMustBeFree
    }
    if !cycle.IsValid()             { return nil, ErrInvalidBillingCycle }
    for k, q := range quotas {
        if q.Limit < QuotaUnlimited {
            return nil, fmt.Errorf("quota %s: limit=%d inválido (mínimo -1)", k, q.Limit)
        }
    }
    now := time.Now().UTC()
    return &Plan{
        id:            NewPlanID(),
        tier:          tier,
        name:          name,
        labelUI:       labelUI,
        description:   description,
        priceCents:    price.Cents,
        currency:      price.Currency,
        stripePriceID: stripePriceID,
        trialDays:     trialDays,
        billingCycle:  cycle,
        features:      features,
        quotas:        quotas,
        active:        true,
        createdAt:     now,
        updatedAt:     now,
    }, nil
}

// ReconstructPlan — hidrata do banco via repositório. Sem validações (dado persistido
// é confiável — se corrompeu, falha em boot, não em runtime). Usado somente em
// infrastructure/database/plan_repository_impl.go.
func ReconstructPlan(
    id PlanID,
    tier PlanTier,
    name, labelUI, description string,
    priceCents int64,
    currency string,
    stripePriceID string,
    trialDays int,
    cycle BillingCycle,
    features FeatureFlags,
    quotas QuotaLimits,
    active bool,
    createdAt, updatedAt time.Time,
) *Plan {
    return &Plan{
        id: id, tier: tier, name: name, labelUI: labelUI, description: description,
        priceCents: priceCents, currency: currency, stripePriceID: stripePriceID,
        trialDays: trialDays, billingCycle: cycle, features: features, quotas: quotas,
        active: active, createdAt: createdAt, updatedAt: updatedAt,
    }
}
```

## § Métodos de domínio

```go
// Getters (sem setters — estado privado, mutações via métodos intencionais)
func (p *Plan) ID() PlanID              { return p.id }
func (p *Plan) Tier() PlanTier          { return p.tier }
func (p *Plan) Name() string            { return p.name }
func (p *Plan) LabelUI() string         { return p.labelUI }
func (p *Plan) PriceCents() int64       { return p.priceCents }
func (p *Plan) TrialDays() int          { return p.trialDays }
func (p *Plan) StripePriceID() string   { return p.stripePriceID }
func (p *Plan) IsActive() bool          { return p.active }

// HasFeature — delega à matriz ABAC via FeatureFlags.
func (p *Plan) HasFeature(k FeatureKey) bool {
    return p.features.Has(k)
}

// QuotaFor — retorna regra de quota (ou zero-value se feature não tem quota).
func (p *Plan) QuotaFor(k FeatureKey) (QuotaRule, bool) {
    q, ok := p.quotas[k]
    return q, ok
}

// Activate / Deactivate — soft toggle (admin UI).
// Deactivate NÃO afeta Subscriptions existentes (invariante INV-PLN-007 só bloqueia criação nova).
func (p *Plan) Activate() (events.PlanActivated, error) {
    if p.active                    { return events.PlanActivated{}, ErrPlanAlreadyActive }
    if p.stripePriceID == ""       { return events.PlanActivated{}, ErrMissingStripePriceID }
    p.active = true
    p.updatedAt = time.Now().UTC()
    return events.PlanActivated{PlanID: p.id.String(), OccurredAt: p.updatedAt}, nil
}

func (p *Plan) Deactivate(reason string) events.PlanDeactivated {
    p.active = false
    p.updatedAt = time.Now().UTC()
    return events.PlanDeactivated{PlanID: p.id.String(), Reason: reason, OccurredAt: p.updatedAt}
}

// ChangePrice — mudança de preço emite evento; Stripe Dashboard deve ser atualizado antes.
func (p *Plan) ChangePrice(newPrice Price) (events.PlanPriceChanged, error) {
    if newPrice.Currency != "BRL"                  { return events.PlanPriceChanged{}, ErrUnsupportedCurrency }
    if p.tier == PlanTierTrial && !newPrice.IsZero() {
        return events.PlanPriceChanged{}, ErrTrialMustBeFree
    }
    old := p.priceCents
    p.priceCents = newPrice.Cents
    p.updatedAt = time.Now().UTC()
    return events.PlanPriceChanged{
        PlanID:        p.id.String(),
        OldPriceCents: old,
        NewPriceCents: newPrice.Cents,
        Currency:      p.currency,
        OccurredAt:    p.updatedAt,
    }, nil
}
```

## § Repository interface (Go)

```go
// internal/modules/billing/domain/repositories/i_plan_repository.go
package repositories

import (
    "context"
    "github.com/mastersindico/backend/internal/modules/billing/domain/entities"
)

type IPlanRepository interface {
    // Find — busca por PK; ErrNotFound se inexistente ou desativado há mais de retention.
    Find(ctx context.Context, id entities.PlanID) (*entities.Plan, error)

    // ListActive — catálogo para pricing page (active=true).
    // Resultados ordenados por tier.Rank() então price_cents asc.
    ListActive(ctx context.Context) ([]*entities.Plan, error)

    // ListByTier — usado em use case CheckoutSubscriptionSaga para apresentar opções ao user.
    ListByTier(ctx context.Context, tier entities.PlanTier) ([]*entities.Plan, error)

    // FindByStripePriceID — reverse lookup no webhook (customer.subscription.updated).
    // UNIQUE constraint garante 0 ou 1 resultado.
    FindByStripePriceID(ctx context.Context, stripePriceID string) (*entities.Plan, error)

    // Save — admin edit (seed, UI admin). UPSERT por id.
    Save(ctx context.Context, plan *entities.Plan) error
}
```

**Nota**: NÃO existe `FindBySlug` nem `FindByRoleAndTier` — ambos métodos vinculados a slugs compostos foram **removidos** por D-066/D-067. `FindByRoleAndTier` existia no código legado (`backend/.../plan.go` linha 38-40); a reescrita Fase 8 do backend deve eliminá-lo.

## § Domain events emitidos

```go
// internal/modules/billing/domain/events/plan_events.go

// PlanActivated — plano foi ativado (novo ou reativado). Handlers:
//   - cache redis: invalidar ms:catalog:active
//   - analytics: registrar novo plano disponível
type PlanActivated struct {
    PlanID     string
    OccurredAt time.Time
}

// PlanDeactivated — admin arquivou plano. Subscriptions ativas permanecem;
// novas não podem ser criadas. Handlers:
//   - cache: invalidar ms:catalog:active
//   - billing job: alertar ops se houver subscriptions pendentes
type PlanDeactivated struct {
    PlanID     string
    Reason     string
    OccurredAt time.Time
}

// PlanPriceChanged — preço alterado. Handlers:
//   - notifications: avisar usuários com renewal próximo (se houver diff > 10%)
//   - audit: AuditEntry com old/new price (LGPD)
type PlanPriceChanged struct {
    PlanID        string
    OldPriceCents int64
    NewPriceCents int64
    Currency      string
    OccurredAt    time.Time
}
```

## § Migrações SQL (tabelas + constraints)

```sql
-- migrations/20260423_0001_create_plans.up.sql

CREATE TYPE plan_tier AS ENUM ('trial', 'base', 'plus', 'pro');
CREATE TYPE billing_cycle AS ENUM ('monthly', 'annual');

CREATE TABLE plans (
    id                UUID        PRIMARY KEY DEFAULT gen_random_uuid_v7(),
    tier              plan_tier   NOT NULL,
    name              TEXT        NOT NULL,
    label_ui          TEXT        NOT NULL,
    description       TEXT        NOT NULL DEFAULT '',
    features          JSONB       NOT NULL DEFAULT '{}',
    quotas            JSONB       NOT NULL DEFAULT '{}',
    price_cents       BIGINT      NOT NULL CHECK (price_cents >= 0),
    currency          CHAR(3)     NOT NULL DEFAULT 'BRL'
                                  CHECK (currency = 'BRL'), -- M1
    stripe_price_id   TEXT,
    trial_days        INT         NOT NULL DEFAULT 0 CHECK (trial_days >= 0),
    billing_cycle     billing_cycle NOT NULL,
    active            BOOLEAN     NOT NULL DEFAULT true,
    created_at        TIMESTAMPTZ NOT NULL DEFAULT now(),
    updated_at        TIMESTAMPTZ NOT NULL DEFAULT now(),

    -- INV-PLN-006: trial tier obrigatoriamente gratuito
    CONSTRAINT plan_trial_is_free
        CHECK ( (tier = 'trial' AND price_cents = 0) OR tier <> 'trial' )
);

-- INV-PLN-004: stripe_price_id único entre planos ativos
CREATE UNIQUE INDEX plans_stripe_price_active_uq
    ON plans (stripe_price_id)
    WHERE active = true AND stripe_price_id IS NOT NULL;

-- Lookup frequente no catálogo
CREATE INDEX plans_active_tier_idx ON plans (tier, price_cents) WHERE active = true;

-- Touch updated_at
CREATE TRIGGER plans_set_updated_at
    BEFORE UPDATE ON plans
    FOR EACH ROW EXECUTE FUNCTION set_updated_at();
```

## § Exemplos de uso

### Seed de catálogo inicial (M1 — 2026-05-08)

```go
// seed/plans.go
func SeedPlans(ctx context.Context, repo repositories.IPlanRepository) error {
    plans := []seedEntry{
        // --- Tier TRIAL (universal — 0 cents) ---
        {
            tier:      PlanTierTrial,
            name:      "Plano Síndico Trial",
            labelUI:   "Experimente Master Síndico",
            priceCents: 0,
            trialDays: 15,
            cycle:     BillingCycleMonthly,
            features: FeatureFlags{
                FeatKeyConnectMe: true,
                FeatKeyVideoUploadInstitutional: true,
            },
            quotas: QuotaLimits{
                FeatKeyConnectMe:                {Limit: 2, Period: QuotaPeriodAnnual},
                FeatKeyVideoUploadInstitutional: {Limit: 4, Period: QuotaPeriodMonthly},
            },
            stripePriceID: "price_trial_syndic_m1",
        },
        {
            tier: PlanTierTrial,
            name: "Plano Parceiro Trial",
            labelUI: "Experimente a Vizinhança",
            priceCents: 0,
            trialDays: 30,
            cycle: BillingCycleMonthly,
            features: FeatureFlags{},
            quotas:   QuotaLimits{},
            stripePriceID: "price_trial_partner_m1",
        },

        // --- Tier BASE ---
        {
            tier: PlanTierBase,
            name: "Plano Síndico Base",
            labelUI: "Plano Essencial",
            priceCents: 4990, // R$ 49,90
            trialDays: 0,
            cycle: BillingCycleMonthly,
            features: FeatureFlags{
                FeatKeyConnectMe: true,
                FeatKeyVideoUploadInstitutional: true,
            },
            quotas: QuotaLimits{
                FeatKeyConnectMe:                {Limit: 2, Period: QuotaPeriodAnnual},
                FeatKeyVideoUploadInstitutional: {Limit: 4, Period: QuotaPeriodMonthly},
            },
            stripePriceID: "price_1ABCbase",
        },
        {
            tier: PlanTierBase,
            name: "Plano Morador Pagante",
            labelUI: "Morador Plus",
            priceCents: 1990, // R$ 19,90
            trialDays: 0,
            cycle: BillingCycleMonthly,
            features: FeatureFlags{FeatKeyConnectMe: true},
            quotas:   QuotaLimits{FeatKeyConnectMe: {Limit: 4, Period: QuotaPeriodAnnual}},
            stripePriceID: "price_1ABCresident",
        },

        // --- Tier PLUS ---
        {
            tier: PlanTierPlus,
            name: "Plano Síndico Plus",
            labelUI: "Plano Profissional",
            priceCents: 9990, // R$ 99,90
            cycle: BillingCycleMonthly,
            features: FeatureFlags{
                FeatKeyConnectMe: true,
                FeatKeyVideoUploadInstitutional: true,
                FeatKeyAssemblyCreate: true,
            },
            quotas: QuotaLimits{
                FeatKeyConnectMe:                {Limit: 4, Period: QuotaPeriodAnnual},
                FeatKeyVideoUploadInstitutional: {Limit: 8, Period: QuotaPeriodMonthly},
            },
            stripePriceID: "price_1ABCplus",
        },
        {
            tier: PlanTierPlus,
            name: "Plano Empresa Plus",
            labelUI: "Empresa Profissional",
            priceCents: 19990,
            cycle: BillingCycleMonthly,
            features: FeatureFlags{
                FeatKeyVideoUploadInstitutional: true,
                FeatKeyReportsExport: true,
            },
            quotas: QuotaLimits{
                FeatKeyVideoUploadInstitutional: {Limit: 8, Period: QuotaPeriodMonthly},
            },
            stripePriceID: "price_1ABCenterpriseplus",
        },
        {
            tier: PlanTierPlus,
            name: "Plano Parceiro Standard",
            labelUI: "Parceiro da Vizinhança",
            priceCents: 4990,
            cycle: BillingCycleMonthly,
            features: FeatureFlags{}, // permissões vêm de role=local_company × tier=plus
            quotas:   QuotaLimits{},
            stripePriceID: "price_1ABCpartnerstd",
        },

        // --- Tier PRO ---
        {
            tier: PlanTierPro,
            name: "Plano Síndico Pro",
            labelUI: "Plano Ilimitado",
            priceCents: 19990,
            cycle: BillingCycleMonthly,
            features: FeatureFlags{
                FeatKeyConnectMe: true,
                FeatKeyVideoUploadInstitutional: true,
                FeatKeyAssemblyCreate: true,
                FeatKeyReportsExport: true,
                FeatKeyLiveSession: true,
            },
            quotas: QuotaLimits{
                FeatKeyConnectMe:                {Limit: QuotaUnlimited, Period: QuotaPeriodAnnual, Unlimited: true},
                FeatKeyVideoUploadInstitutional: {Limit: 12, Period: QuotaPeriodMonthly},
            },
            stripePriceID: "price_1ABCpro",
        },
        {
            tier: PlanTierPro,
            name: "Plano Empresa Pro",
            labelUI: "Empresa Premium",
            priceCents: 39990,
            cycle: BillingCycleMonthly,
            features: FeatureFlags{
                FeatKeyVideoUploadInstitutional: true,
                FeatKeyReportsExport: true,
                FeatKeyLiveSession: true,
            },
            quotas: QuotaLimits{
                FeatKeyVideoUploadInstitutional: {Limit: 12, Period: QuotaPeriodMonthly},
            },
            stripePriceID: "price_1ABCenterprisepro",
        },
    }

    for _, e := range plans {
        p, err := NewPlan(e.tier, e.name, e.labelUI, "", Price{Cents: e.priceCents, Currency: "BRL"},
            e.trialDays, e.cycle, e.features, e.quotas, e.stripePriceID)
        if err != nil {
            return fmt.Errorf("seed %s: %w", e.name, err)
        }
        if err := repo.Save(ctx, p); err != nil {
            return err
        }
    }
    return nil
}
```

**Nota importante sobre seed**: os nomes `"Plano Síndico Base"`, `"Plano Empresa Plus"`, etc. são **identificadores comerciais** no campo `name` — NÃO são slugs. O dado de identificação universal permanece `tier ∈ {trial, base, plus, pro}`. Um user `role=enterprise` PODE assinar "Plano Síndico Plus" tecnicamente (sem bloqueio de banco); a **matriz ABAC** decide se ele tem acesso a features específicas combinando `role × tier`. Soft-guards de UI filtram o catálogo por role, mas isso é convivência, não constraint de dado.

### Consulta no use case (CheckoutSubscriptionSaga)

```go
func (uc *CheckoutSubscriptionSaga) Run(ctx context.Context, cmd CheckoutCommand) error {
    plan, err := uc.planRepo.Find(ctx, cmd.PlanID)
    if err != nil {
        return err
    }
    if !plan.IsActive() {
        return ErrPlanInactive // INV-PLN-007
    }
    // prosseguir com checkout Stripe...
}
```

### Webhook reverse-lookup

```go
func (uc *HandleStripeWebhookUseCase) onSubscriptionUpdated(ctx context.Context, evt stripe.Event) error {
    priceID := evt.Data.Object["price_id"].(string)
    plan, err := uc.planRepo.FindByStripePriceID(ctx, priceID)
    if err != nil {
        return fmt.Errorf("plano com stripe_price_id=%s desconhecido: %w", priceID, err)
    }
    // ...
}
```

## § Dependências com outros aggregates

- **[[Subscription]]** — FK `subscription.plan_id → plan.id`. `Subscription` usa `plan.trialDays`, `plan.quotas`, `plan.priceCents` (snapshot no momento do checkout).
- **[[FeatureUsage]]** — consulta `plan.QuotaFor(featureKey)` para determinar `quota_limit` ao criar instância de tracking.
- **[[User]]** — `user.role` é ortogonal a `plan.tier`. Cruzam-se na matriz ABAC.
- **[[AbacDecision]]** — engine ABAC recebe `plan.tier` como atributo; combina com `user.role` e `action` para computar allow/deny.

## § Referências cruzadas (STATE, ADRs, specs)

- **STATE**: [[../../STATE#D-066-decisao-arquitetural-critica-planos-globais-stripe-style]] · [[../../STATE#D-067-n1-n2-n3-e-outros-nomes-de-plano-por-persona-abolidos-completamente]] · [[../../STATE#D-058-morador-base-connect-me-0-ano-conforme-matriz]]
- **ADR**: [[../../02-architecture/adr/0032-global-plans-stripe-style]] · [[../../02-architecture/adr/0004-stripe-payment-gateway]] · [[../../02-architecture/adr/0009-uuidv7-ulid-pk]]
- **Requirements**: [[../../04-requirements/functional/billing]] Req 4, Req 4.1, Req 6
- **Matriz**: [[../../04-requirements/matrix-functional]] (coluna `Trial | Base | Plus | Pro` universal)
- **Sub-produtos com quota**: [[../../00-product/sub-produtos/02-connect-me]] · [[../../00-product/sub-produtos/04-conteudo-videos]] · [[../../00-product/sub-produtos/06-lms-academia]]
- **Backend real (cruzar na reescrita Fase 8 do sub-projeto backend)**:
  - `backend/internal/modules/billing/domain/entities/plan.go` — **CONTAMINADO** com campo `role` (linhas 13, 39, 87); elimina em reescrita
  - `backend/internal/modules/billing/domain/repositories/i_plan_repository.go` — `FindByRoleAndTier` deve ser trocado por `ListByTier`
  - `backend/internal/modules/billing/domain/services/quota_limits.go` — matriz `role × tier` deve virar consulta `plan.QuotaFor(feature)` puro (role vira dimensão ABAC, não de Plan)
- **Links aggregates**: [[Subscription]] · [[FeatureUsage]] · [[User]] · [[AbacDecision]]
- **Pendência**: [[../../11-audit/AUDIT#a-fase8-plan-001-contaminacao-role-em-plango]] (criar finding)
