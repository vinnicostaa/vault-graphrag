---
title: "Aggregate — Subscription"
type: spec
tags: [aggregate, ddd, billing, subscription, stripe, master-sindico, v2, fase-8-consolidado]
source: ADR-0032 + STATE D-066/D-067/D-058 + backend/internal/modules/billing/domain/entities/subscription.go + requirements/billing.md
created: 2026-04-23
updated: 2026-04-23
status: consolidado-fase-8
dual_check: pending
---

# Aggregate — `Subscription`

**BC**: billing · **Raiz**: ✅ (liga `User` a `Plan`; mirror local do Stripe Subscription)

## § Propósito

`Subscription` é o **elo User ↔ Plan** no modelo Stripe-style (ADR-0032). Ela NÃO carrega features nem quotas próprios — apenas aponta para um `Plan` via `plan_id`. Permissões derivam, em runtime, do par `(user.role, plan.tier)` consultado na matriz ABAC.

Modelo append-**friendly**: diferente da v1 que era strictly append-only, v2 permite UPDATE de campos voláteis (`status`, `current_period_end`, `cancel_at_period_end`) para espelhar o Stripe sem explosão de rows, mas mantém **idempotência garantida por webhook ID** (ADR-0027) e AuditEntry para toda transição de estado.

**Decisões-raiz**:
- **D-066** / **ADR-0032**: Subscription aponta `plan_id` global, não mais `(role, tier)` ou slug composto
- **D-058** (STATE Fase 7): sem grace period — `canceled`/`expired` = soft-block imediato
- **ADR-0004**: Stripe como payment gateway (`IPaymentGateway`)
- **ADR-0027**: idempotência de webhooks via tabela `webhook_events`

## § Entidade raiz (campos + tipos + constraints)

```go
// internal/modules/billing/domain/entities/subscription.go
type Subscription struct {
    id                      SubscriptionID   // UUIDv7 PK
    userID                  UserID           // FK → users.id
    planID                  PlanID           // FK → plans.id

    status                  SubscriptionStatus
    // enum: trial | active | past_due | canceled | paused | expired | incomplete

    trialStartsAt           *time.Time       // nullable (se plano sem trial)
    trialEndsAt             *time.Time       // INVARIANTE: > trialStartsAt quando ambos presentes
    currentPeriodStart      time.Time        // período atual de cobrança
    currentPeriodEnd        time.Time        // INVARIANTE: > currentPeriodStart
    canceledAt              *time.Time
    cancelReason            string           // "user_request" | "payment_failed" | "admin" | ...
    cancelAtPeriodEnd       bool             // soft-cancel (mantém acesso até period_end)

    // --- Referências Stripe (mirror) ---
    stripeSubscriptionID    string           // UNIQUE quando != ""
    stripeCustomerID        string
    paymentMethodLast4      string           // "4242" — exibição UI apenas

    createdAt               time.Time
    updatedAt               time.Time
}
```

**Constraints SQL** (ver § Migrações):
- `id UUID PK`
- `user_id UUID NOT NULL REFERENCES users(id)`
- `plan_id UUID NOT NULL REFERENCES plans(id)`
- `status subscription_status_enum NOT NULL`
- `trial_starts_at, trial_ends_at TIMESTAMPTZ NULL`
- `current_period_start, current_period_end TIMESTAMPTZ NOT NULL`
- `stripe_subscription_id TEXT` (UNIQUE parcial `WHERE stripe_subscription_id <> ''`)
- UNIQUE parcial: `(user_id, plan_id) WHERE status IN ('trial', 'active')` — garante INV-SUB-003
- UNIQUE derivado via expressão: `(user_id, (plans.tier via join))` — enforcado no **use case**, não no banco (cross-table)

## § Value Objects

```go
// ------- SubscriptionID -------
type SubscriptionID string // UUIDv7

func NewSubscriptionID() SubscriptionID  { return SubscriptionID(uuidv7.New()) }
func (id SubscriptionID) String() string { return string(id) }

// ------- SubscriptionStatus -------
type SubscriptionStatus string

const (
    SubscriptionStatusTrial      SubscriptionStatus = "trial"
    SubscriptionStatusActive     SubscriptionStatus = "active"
    SubscriptionStatusPastDue    SubscriptionStatus = "past_due"
    SubscriptionStatusCanceled   SubscriptionStatus = "canceled"
    SubscriptionStatusPaused     SubscriptionStatus = "paused"
    SubscriptionStatusExpired    SubscriptionStatus = "expired"
    SubscriptionStatusIncomplete SubscriptionStatus = "incomplete"
)

// IsActive — concede acesso premium. Paused NÃO dá acesso (cortesia fica pra M2).
func (s SubscriptionStatus) GrantsAccess() bool {
    return s == SubscriptionStatusTrial || s == SubscriptionStatusActive
}

func (s SubscriptionStatus) IsTerminal() bool {
    return s == SubscriptionStatusCanceled || s == SubscriptionStatusExpired
}

// ------- BillingPeriod (VO) -------
type BillingPeriod struct {
    Start time.Time
    End   time.Time
}

func NewBillingPeriod(start, end time.Time) (BillingPeriod, error) {
    if !end.After(start) {
        return BillingPeriod{}, ErrInvalidBillingPeriod
    }
    return BillingPeriod{Start: start, End: end}, nil
}

func (bp BillingPeriod) Covers(t time.Time) bool {
    return !t.Before(bp.Start) && t.Before(bp.End)
}

func (bp BillingPeriod) Duration() time.Duration { return bp.End.Sub(bp.Start) }
```

## § Invariantes (testáveis)

| ID | Regra | Onde é testado |
|---|---|---|
| **INV-SUB-001** | `trial_ends_at > trial_starts_at` quando ambos presentes | factory + CHECK SQL |
| **INV-SUB-002** | `current_period_end > current_period_start` | factory + CHECK SQL |
| **INV-SUB-003** | User pode ter **no máximo 1 subscription** com `status ∈ {trial, active}` por **plan_tier** (não por plan.id). Aplicado em use case; banco garante `(user_id, plan_id)` único | UNIQUE parcial + use case |
| **INV-SUB-004** | State machine (ver § Estado): `trial → {active, canceled, expired}`; `active → {past_due, canceled, paused}`; `past_due → {active, canceled, expired}`; `canceled → ∅` (terminal) | método `transitionTo` |
| **INV-SUB-005** | `canceled → active` **NÃO existe** — reativação cria nova Subscription | enforced em `Activate()` |
| **INV-SUB-006** | Soft-block imediato pós `expired`/`canceled` — **sem grace period** (D-058 legado) | ABAC checa `status.GrantsAccess()` |
| **INV-SUB-007** | `stripe_subscription_id` é UNIQUE global quando não vazio | UNIQUE parcial SQL |
| **INV-SUB-008** | Toda transição de estado emite domain event + AuditEntry | método `transitionTo` |
| **INV-SUB-009** | `trial_days` efetivo = `plan.TrialDays()` snapshot no momento de `NewTrialSubscription` — mudanças subsequentes em `plan.trialDays` NÃO afetam subscriptions existentes | factory |
| **INV-SUB-010** | `cancel_at_period_end=true` mantém `GrantsAccess()=true` até `current_period_end`; job `sweep_expired_subscriptions` roda às 00:05 UTC e faz `CancelNow()` das expiradas | cron + test |

## § Factories (New... / Reconstruct...)

```go
// NewTrialSubscription — use case StartTrialUseCase chama após UserCreated.
// Snapshot de plan.trialDays + criação de BillingPeriod preliminar.
func NewTrialSubscription(
    userID UserID,
    plan *Plan,
    now time.Time,
) (*Subscription, events.SubscriptionStarted, error) {
    if !plan.IsActive() {
        return nil, events.SubscriptionStarted{}, ErrPlanInactive
    }
    td := plan.TrialDays()
    if td <= 0 {
        // plano sem trial — cria Subscription direto em status=incomplete
        // (aguarda webhook Stripe com payment method confirmado)
        return newIncomplete(userID, plan, now)
    }
    trialStart := now.UTC()
    trialEnd := trialStart.Add(time.Duration(td) * 24 * time.Hour)
    periodStart := trialStart
    periodEnd := trialEnd
    sub := &Subscription{
        id:                 NewSubscriptionID(),
        userID:             userID,
        planID:             plan.ID(),
        status:             SubscriptionStatusTrial,
        trialStartsAt:      &trialStart,
        trialEndsAt:        &trialEnd,
        currentPeriodStart: periodStart,
        currentPeriodEnd:   periodEnd,
        createdAt:          trialStart,
        updatedAt:          trialStart,
    }
    return sub, events.SubscriptionStarted{
        SubscriptionID: sub.id.String(),
        UserID:         userID.String(),
        PlanID:         plan.ID().String(),
        Status:         string(sub.status),
        TrialEndsAt:    sub.trialEndsAt,
        OccurredAt:     trialStart,
    }, nil
}

// ReconstructSubscription — hidratação pelo repositório.
func ReconstructSubscription(
    id SubscriptionID,
    userID UserID,
    planID PlanID,
    status SubscriptionStatus,
    trialStartsAt, trialEndsAt *time.Time,
    currentPeriodStart, currentPeriodEnd time.Time,
    canceledAt *time.Time,
    cancelReason string,
    cancelAtPeriodEnd bool,
    stripeSubscriptionID, stripeCustomerID, paymentMethodLast4 string,
    createdAt, updatedAt time.Time,
) *Subscription {
    return &Subscription{
        id: id, userID: userID, planID: planID, status: status,
        trialStartsAt: trialStartsAt, trialEndsAt: trialEndsAt,
        currentPeriodStart: currentPeriodStart, currentPeriodEnd: currentPeriodEnd,
        canceledAt: canceledAt, cancelReason: cancelReason,
        cancelAtPeriodEnd: cancelAtPeriodEnd,
        stripeSubscriptionID: stripeSubscriptionID,
        stripeCustomerID: stripeCustomerID,
        paymentMethodLast4: paymentMethodLast4,
        createdAt: createdAt, updatedAt: updatedAt,
    }
}
```

## § Métodos de domínio

```go
// Getters — estado privado
func (s *Subscription) ID() SubscriptionID             { return s.id }
func (s *Subscription) UserID() UserID                 { return s.userID }
func (s *Subscription) PlanID() PlanID                 { return s.planID }
func (s *Subscription) Status() SubscriptionStatus     { return s.status }
func (s *Subscription) TrialEndsAt() *time.Time        { return s.trialEndsAt }
func (s *Subscription) CurrentPeriodEnd() time.Time    { return s.currentPeriodEnd }
func (s *Subscription) IsCancelScheduled() bool        { return s.cancelAtPeriodEnd }
func (s *Subscription) StripeSubscriptionID() string   { return s.stripeSubscriptionID }

// GrantsAccess — consulta canônica pelo ABAC / middleware paywall.
// Combina status.GrantsAccess() com a checagem de trial ainda válido.
func (s *Subscription) GrantsAccess() bool {
    if s.status == SubscriptionStatusTrial {
        if s.trialEndsAt == nil            { return false }
        return s.trialEndsAt.After(time.Now().UTC())
    }
    return s.status.GrantsAccess()
}

// IsTrialExpiringSoon — helper pra job TrialReminders (D-3 / D-1 / D+0).
func (s *Subscription) IsTrialExpiringSoon(within time.Duration, now time.Time) bool {
    if s.status != SubscriptionStatusTrial || s.trialEndsAt == nil { return false }
    return s.trialEndsAt.Sub(now) <= within && s.trialEndsAt.After(now)
}

// --- Transições de estado (INV-SUB-004, INV-SUB-008) ---

// Activate — converte trial → active (pós webhook payment_intent.succeeded).
// REJEITA canceled → active (INV-SUB-005).
func (s *Subscription) Activate(period BillingPeriod, stripeSubID, stripeCustomerID string) (events.SubscriptionActivated, error) {
    switch s.status {
    case SubscriptionStatusTrial, SubscriptionStatusPastDue, SubscriptionStatusIncomplete:
        // ok
    case SubscriptionStatusCanceled, SubscriptionStatusExpired:
        return events.SubscriptionActivated{}, ErrCannotReactivate // INV-SUB-005
    case SubscriptionStatusActive:
        return events.SubscriptionActivated{}, ErrAlreadyActive
    default:
        return events.SubscriptionActivated{}, ErrInvalidTransition
    }
    s.status = SubscriptionStatusActive
    s.currentPeriodStart = period.Start
    s.currentPeriodEnd = period.End
    s.stripeSubscriptionID = stripeSubID
    s.stripeCustomerID = stripeCustomerID
    s.updatedAt = time.Now().UTC()
    return events.SubscriptionActivated{
        SubscriptionID: s.id.String(),
        UserID:         s.userID.String(),
        PlanID:         s.planID.String(),
        OccurredAt:     s.updatedAt,
    }, nil
}

// MarkPastDue — webhook invoice.payment_failed. active → past_due.
func (s *Subscription) MarkPastDue() (events.SubscriptionPastDue, error) {
    if s.status != SubscriptionStatusActive {
        return events.SubscriptionPastDue{}, ErrInvalidTransition
    }
    s.status = SubscriptionStatusPastDue
    s.updatedAt = time.Now().UTC()
    return events.SubscriptionPastDue{
        SubscriptionID: s.id.String(),
        UserID:         s.userID.String(),
        OccurredAt:     s.updatedAt,
    }, nil
}

// Cancel — cancelamento imediato (hard). Usuário perde acesso NA HORA (D-058: sem grace).
func (s *Subscription) Cancel(reason string, now time.Time) (events.SubscriptionCanceled, error) {
    if s.status.IsTerminal() {
        return events.SubscriptionCanceled{}, ErrAlreadyCanceled
    }
    s.status = SubscriptionStatusCanceled
    s.canceledAt = &now
    s.cancelReason = reason
    s.updatedAt = now
    return events.SubscriptionCanceled{
        SubscriptionID: s.id.String(),
        UserID:         s.userID.String(),
        PlanID:         s.planID.String(),
        Reason:         reason,
        OccurredAt:     now,
    }, nil
}

// CancelAtPeriodEnd — soft-cancel. User continua acesso até current_period_end.
// Cron job sweep executa Cancel() após expirar.
func (s *Subscription) CancelAtPeriodEnd(reason string) error {
    if !s.status.GrantsAccess() { return ErrInvalidTransition }
    s.cancelAtPeriodEnd = true
    s.cancelReason = reason
    s.updatedAt = time.Now().UTC()
    return nil
}

// Pause — status=paused (Stripe customer-initiated). Não concede acesso (M1).
func (s *Subscription) Pause() (events.SubscriptionPaused, error) {
    if s.status != SubscriptionStatusActive {
        return events.SubscriptionPaused{}, ErrInvalidTransition
    }
    s.status = SubscriptionStatusPaused
    s.updatedAt = time.Now().UTC()
    return events.SubscriptionPaused{
        SubscriptionID: s.id.String(),
        UserID:         s.userID.String(),
        OccurredAt:     s.updatedAt,
    }, nil
}

// Resume — paused → active.
func (s *Subscription) Resume(period BillingPeriod) (events.SubscriptionResumed, error) {
    if s.status != SubscriptionStatusPaused {
        return events.SubscriptionResumed{}, ErrInvalidTransition
    }
    s.status = SubscriptionStatusActive
    s.currentPeriodStart = period.Start
    s.currentPeriodEnd = period.End
    s.updatedAt = time.Now().UTC()
    return events.SubscriptionResumed{
        SubscriptionID: s.id.String(),
        UserID:         s.userID.String(),
        OccurredAt:     s.updatedAt,
    }, nil
}

// Expire — trial esgotado sem conversão OR past_due esgotou retry Stripe.
// Soft-block (D-058): dados permanecem, acesso bloqueado.
func (s *Subscription) Expire(now time.Time) (events.SubscriptionExpired, error) {
    switch s.status {
    case SubscriptionStatusTrial, SubscriptionStatusPastDue:
        s.status = SubscriptionStatusExpired
        s.updatedAt = now
        return events.SubscriptionExpired{
            SubscriptionID: s.id.String(),
            UserID:         s.userID.String(),
            PlanID:         s.planID.String(),
            ExpiredFrom:    "trial_or_past_due",
            OccurredAt:     now,
        }, nil
    }
    return events.SubscriptionExpired{}, ErrInvalidTransition
}

// UpdateFromStripe — sincroniza campos voláteis do webhook customer.subscription.updated.
// NÃO muda status (estado é driven por eventos tipados acima); só campos mirror.
func (s *Subscription) UpdateFromStripe(period BillingPeriod, last4 string) {
    s.currentPeriodStart = period.Start
    s.currentPeriodEnd = period.End
    if last4 != "" { s.paymentMethodLast4 = last4 }
    s.updatedAt = time.Now().UTC()
}
```

### State machine (visualizado)

```
                ┌──────────────┐
                │  incomplete  │  (plano sem trial aguardando pagamento)
                └──────┬───────┘
                       │ Activate
                       ▼
┌─────────┐  Activate  ┌────────┐  MarkPastDue   ┌──────────┐
│  trial  │──────────► │ active │ ─────────────► │ past_due │
└────┬────┘            └───┬────┘                └────┬─────┘
     │ Expire               │ Cancel                  │ Activate  (Stripe retry sucesso)
     │                       │                        ▼
     ▼                       ▼                     ┌────────┐
  ┌─────────┐            ┌──────────┐              │ active │
  │ expired │            │ canceled │ (terminal)   └────────┘
  └─────────┘            └──────────┘
     ▲
     │ Expire (past_due esgotou retry)
     │
  ┌──────────┐ Pause       ┌────────┐
  │  active  │────────────►│ paused │
  └──────────┘◄────────────└────────┘
                 Resume

TERMINAL: expired, canceled  ---- canceled → active NUNCA (INV-SUB-005)
```

## § Repository interface (Go)

```go
// internal/modules/billing/domain/repositories/i_subscription_repository.go
package repositories

import (
    "context"
    "github.com/mastersindico/backend/internal/modules/billing/domain/entities"
)

type ISubscriptionRepository interface {
    // Find — por PK.
    Find(ctx context.Context, id entities.SubscriptionID) (*entities.Subscription, error)

    // FindActiveByUser — retorna subscription atual que concede acesso.
    // Dado INV-SUB-003: user tem no máximo 1 active/trial por plan_tier; esta query
    // retorna a "principal" (tier mais alto). Retorna ErrNotFound se usuário não tem.
    FindActiveByUser(ctx context.Context, userID entities.UserID) (*entities.Subscription, error)

    // ListByUser — todas as subscriptions (ativas + canceled + expired) do user. Histórico.
    ListByUser(ctx context.Context, userID entities.UserID) ([]*entities.Subscription, error)

    // FindByStripeID — reverse lookup no webhook.
    FindByStripeID(ctx context.Context, stripeSubscriptionID string) (*entities.Subscription, error)

    // Save — UPSERT por id.
    Save(ctx context.Context, sub *entities.Subscription) error

    // CancelAtPeriodEnd — método de conveniência; valida estado antes de flipar flag.
    CancelAtPeriodEnd(ctx context.Context, id entities.SubscriptionID, reason string) error

    // FindExpiringTrialsBetween — usado por cron TrialReminders.
    FindExpiringTrialsBetween(ctx context.Context, from, to time.Time) ([]*entities.Subscription, error)

    // FindDueForExpiry — subscriptions com cancel_at_period_end=true cujo period_end já passou.
    FindDueForExpiry(ctx context.Context, now time.Time) ([]*entities.Subscription, error)
}
```

## § Domain events emitidos

```go
// internal/modules/billing/domain/events/subscription_events.go

type SubscriptionStarted struct {   // trial OU incomplete
    SubscriptionID string
    UserID         string
    PlanID         string
    Status         string // "trial" | "incomplete"
    TrialEndsAt    *time.Time
    OccurredAt     time.Time
}

type SubscriptionActivated struct { // pagamento confirmado
    SubscriptionID string
    UserID         string
    PlanID         string
    OccurredAt     time.Time
}

type SubscriptionPastDue struct {
    SubscriptionID string
    UserID         string
    OccurredAt     time.Time
}

type SubscriptionExpired struct {
    SubscriptionID string
    UserID         string
    PlanID         string
    ExpiredFrom    string // "trial" | "past_due"
    OccurredAt     time.Time
}

type SubscriptionCanceled struct {
    SubscriptionID string
    UserID         string
    PlanID         string
    Reason         string
    OccurredAt     time.Time
}

type SubscriptionPaused struct {
    SubscriptionID string
    UserID         string
    OccurredAt     time.Time
}

type SubscriptionResumed struct {
    SubscriptionID string
    UserID         string
    OccurredAt     time.Time
}

// Cron-emitted (via job TrialReminders rodando 00:00 UTC diariamente)
type TrialExpiringSoon struct {
    SubscriptionID string
    UserID         string
    DaysLeft       int    // 3, 1, 0
    TrialEndsAt    time.Time
    OccurredAt     time.Time
}

type TrialExpired struct {
    SubscriptionID string
    UserID         string
    PlanID         string
    ExpiredAt      time.Time
    OccurredAt     time.Time
}
```

**Handlers canônicos**:
- `SubscriptionStarted` → notifications (welcome email) + analytics (mixpanel/ph event)
- `SubscriptionActivated` → ABAC cache invalidate + notifications (payment confirmed)
- `SubscriptionPastDue` → notifications (update payment method) + ABAC **não invalida** (continua GrantsAccess=false desde status mudou)
- `SubscriptionExpired` → ABAC cache invalidate + feature_usages **não apaga** (histórico)
- `SubscriptionCanceled` → ABAC cache invalidate + notifications
- `TrialExpiringSoon` → notifications (email + in-app banner)
- `TrialExpired` → dispara `Subscription.Expire()` via command handler, que emite `SubscriptionExpired`

## § Migrações SQL (tabelas + constraints)

```sql
-- migrations/20260423_0002_create_subscriptions.up.sql

CREATE TYPE subscription_status AS ENUM (
    'trial', 'active', 'past_due', 'canceled', 'paused', 'expired', 'incomplete'
);

CREATE TABLE subscriptions (
    id                      UUID         PRIMARY KEY DEFAULT gen_random_uuid_v7(),
    user_id                 UUID         NOT NULL REFERENCES users(id) ON DELETE RESTRICT,
    plan_id                 UUID         NOT NULL REFERENCES plans(id) ON DELETE RESTRICT,
    status                  subscription_status NOT NULL,

    trial_starts_at         TIMESTAMPTZ,
    trial_ends_at           TIMESTAMPTZ,
    current_period_start    TIMESTAMPTZ  NOT NULL,
    current_period_end      TIMESTAMPTZ  NOT NULL,

    canceled_at             TIMESTAMPTZ,
    cancel_reason           TEXT         NOT NULL DEFAULT '',
    cancel_at_period_end    BOOLEAN      NOT NULL DEFAULT false,

    stripe_subscription_id  TEXT         NOT NULL DEFAULT '',
    stripe_customer_id      TEXT         NOT NULL DEFAULT '',
    payment_method_last4    TEXT         NOT NULL DEFAULT '',

    created_at              TIMESTAMPTZ  NOT NULL DEFAULT now(),
    updated_at              TIMESTAMPTZ  NOT NULL DEFAULT now(),

    -- INV-SUB-001
    CONSTRAINT sub_trial_window_valid
        CHECK ( (trial_starts_at IS NULL AND trial_ends_at IS NULL)
             OR (trial_starts_at IS NOT NULL AND trial_ends_at IS NOT NULL AND trial_ends_at > trial_starts_at) ),

    -- INV-SUB-002
    CONSTRAINT sub_period_valid
        CHECK (current_period_end > current_period_start)
);

-- INV-SUB-003: unicidade de subscription ativa por (user, plan)
CREATE UNIQUE INDEX subscriptions_active_user_plan_uq
    ON subscriptions (user_id, plan_id)
    WHERE status IN ('trial', 'active');

-- INV-SUB-007
CREATE UNIQUE INDEX subscriptions_stripe_id_uq
    ON subscriptions (stripe_subscription_id)
    WHERE stripe_subscription_id <> '';

-- Queries frequentes
CREATE INDEX subscriptions_user_status_idx    ON subscriptions (user_id, status);
CREATE INDEX subscriptions_trial_ends_at_idx  ON subscriptions (trial_ends_at)
    WHERE status = 'trial';
CREATE INDEX subscriptions_period_end_idx     ON subscriptions (current_period_end)
    WHERE cancel_at_period_end = true AND status IN ('active', 'past_due');

CREATE TRIGGER subscriptions_set_updated_at
    BEFORE UPDATE ON subscriptions
    FOR EACH ROW EXECUTE FUNCTION set_updated_at();
```

## § Exemplos de uso

### Trial start após UserCreated

```go
// application/use_cases/start_trial_use_case.go
func (uc *StartTrialUseCase) Run(ctx context.Context, cmd StartTrialCommand) error {
    return uc.uow.Run(ctx, func(ctx context.Context) error {
        plan, err := uc.planRepo.Find(ctx, cmd.PlanID)
        if err != nil { return err }

        sub, evt, err := entities.NewTrialSubscription(cmd.UserID, plan, time.Now().UTC())
        if err != nil { return err }

        if err := uc.subRepo.Save(ctx, sub); err != nil { return err }
        return uc.publisher.Publish(ctx, evt)
    })
}
```

### Webhook customer.subscription.updated

```go
func (uc *HandleStripeWebhookUseCase) onSubscriptionUpdated(ctx context.Context, evt stripe.Event) error {
    return uc.uow.Run(ctx, func(ctx context.Context) error {
        stripeSub := evt.Data.Object.(*stripe.Subscription)
        sub, err := uc.subRepo.FindByStripeID(ctx, stripeSub.ID)
        if err != nil { return err }

        period, _ := entities.NewBillingPeriod(
            time.Unix(stripeSub.CurrentPeriodStart, 0).UTC(),
            time.Unix(stripeSub.CurrentPeriodEnd, 0).UTC(),
        )

        switch stripeSub.Status {
        case stripe.SubscriptionStatusActive:
            evtOut, err := sub.Activate(period, stripeSub.ID, stripeSub.Customer.ID)
            if err != nil { return err }
            if err := uc.subRepo.Save(ctx, sub); err != nil { return err }
            return uc.publisher.Publish(ctx, evtOut)
        case stripe.SubscriptionStatusPastDue:
            evtOut, err := sub.MarkPastDue()
            if err != nil { return err }
            _ = uc.subRepo.Save(ctx, sub)
            return uc.publisher.Publish(ctx, evtOut)
        case stripe.SubscriptionStatusCanceled:
            evtOut, err := sub.Cancel("stripe_event", time.Now().UTC())
            if err != nil { return err }
            _ = uc.subRepo.Save(ctx, sub)
            return uc.publisher.Publish(ctx, evtOut)
        }
        // fallback — só atualiza período sem mudar status
        sub.UpdateFromStripe(period, "")
        return uc.subRepo.Save(ctx, sub)
    })
}
```

### Cron sweep — expira trials vencidos (D+0 00:05 UTC)

```go
func (j *TrialSweepJob) Run(ctx context.Context) error {
    now := time.Now().UTC()
    // trials que expiraram nas últimas 24h
    subs, err := j.subRepo.FindExpiringTrialsBetween(ctx, now.Add(-24*time.Hour), now)
    if err != nil { return err }

    for _, sub := range subs {
        if sub.Status() != entities.SubscriptionStatusTrial { continue }
        if sub.TrialEndsAt().After(now)                      { continue }

        evt, err := sub.Expire(now)
        if err != nil {
            j.logger.Warn().Err(err).Str("sub_id", sub.ID().String()).Msg("skip expire")
            continue
        }
        _ = j.subRepo.Save(ctx, sub)
        _ = j.publisher.Publish(ctx, evt)
    }
    return nil
}
```

## § Dependências com outros aggregates

- **[[Plan]]** — `plan_id` FK; snapshot de `plan.TrialDays()` e `plan.quotas` no momento do checkout
- **[[User]]** — `user_id` FK; `user.role` é ortogonal a `plan.tier`, mas consulta combina os dois via ABAC
- **[[FeatureUsage]]** — FK inversa: `feature_usages.subscription_id → subscriptions.id` permite reset de quotas no cancelamento
- **[[WebhookEvent]]** — idempotência de webhooks Stripe (ADR-0027)
- **[[AuditEntry]]** — toda transição de estado gera AuditEntry (INV-SUB-008)
- **[[OutboxEntry]]** — eventos publicados via outbox transacional para garantir at-least-once

## § Referências cruzadas (STATE, ADRs, specs)

- **STATE**: [[../../STATE#D-066-decisao-arquitetural-critica-planos-globais-stripe-style]] · [[../../STATE#D-067-n1-n2-n3-e-outros-nomes-de-plano-por-persona-abolidos-completamente]] · [[../../STATE#D-058-painel-admin-ms-nao-e-app-separada-e-role-privilegiada-transversal]] · [[../../STATE#D-058-morador-base-connect-me-0-ano-conforme-matriz]] (ambos D-058 são distintos — ver DT-008)
- **ADR**: [[../../02-architecture/adr/0032-global-plans-stripe-style]] · [[../../02-architecture/adr/0004-stripe-payment-gateway]] · [[../../02-architecture/adr/0027-webhook-idempotency-db]] · [[../../02-architecture/adr/0030-saga-orchestration-default]]
- **Requirements**: [[../../04-requirements/functional/billing]] Req 4, Req 4.1
- **Backend real (cruzar na reescrita Fase 8 do sub-projeto backend)**:
  - `backend/internal/modules/billing/domain/entities/subscription.go` — estado atual (status `trialing` em vez de `trial`; LinkStripe separado de Activate — **revisar alinhamento com v2 state machine**)
  - `backend/internal/modules/billing/domain/repositories/i_subscription_repository.go` — adicionar `FindExpiringTrialsBetween` e `FindDueForExpiry`
  - `backend/internal/modules/billing/application/use_cases/handle_stripe_webhook_use_case.go` — refatorar pra usar transições tipadas do aggregate v2
- **Links aggregates**: [[Plan]] · [[FeatureUsage]] · [[User]] · [[WebhookEvent]] · [[AuditEntry]] · [[OutboxEntry]]
- **Pendência**: [[../../11-audit/AUDIT#a-fase8-sub-001-reconciliar-status-trialing-vs-trial]] (criar finding: backend usa `trialing`, v2 usa `trial`)
