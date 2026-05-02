---
title: "Aggregate — FeatureUsage"
type: spec
tags: [aggregate, ddd, billing, quota, master-sindico, v2, fase-8-consolidado]
source: ADR-0032 + STATE D-066/D-067 + backend/internal/modules/billing/domain/entities/feature_usage.go + requirements/billing.md (Req 6)
created: 2026-04-23
updated: 2026-04-23
status: consolidado-fase-8
dual_check: pending
---

# Aggregate — `FeatureUsage`

**BC**: billing · **Raiz**: ✅ (contador vs quota por User + feature + período)

## § Propósito

`FeatureUsage` rastreia **consumo atômico de features com quota** (Connect Me, uploads de vídeo, downloads de ebook, minutos de live session). Uma linha = `(user_id, feature_key, period_key)`. A coluna `quota_limit` é **snapshot** do limite derivado de `plan.QuotaFor(feature)` no momento da criação — muda **apenas em reset** (período novo) ou troca de plano (`ChangePlan` emite recalc).

Protegido contra TOCTOU via `SELECT ... FOR UPDATE` dentro de `UnitOfWork` (antipattern A8 do legacy — fix canônico).

**Decisões-raiz**:
- **ADR-0032** (D-066/D-067): `quota_limit` é derivado de `plan.tier` + matriz ABAC (não mais de `(role, tier)` como no legado `quota_limits.go`)
- **D-094 Fase 11** (revoga D-058/D-079): Morador Base Connect Me = **2/ano** (usage criada; CTA upgrade quando saldo zerar)
- **INV-045** (legado) mantido: quota ConnectMe decrementa no `publish` (não no `draft`)
- **INV-046** (legado) mantido: reset anual Connect Me em 1º/jan UTC; mensal em 1º do mês UTC

## § Entidade raiz (campos + tipos + constraints)

```go
// internal/modules/billing/domain/entities/feature_usage.go
type FeatureUsage struct {
    id             FeatureUsageID  // UUIDv7 PK
    userID         UserID          // FK → users.id
    subscriptionID SubscriptionID  // FK → subscriptions.id (snapshot do plano ativo)
    featureKey     FeatureKey      // enum: connect_me | video_upload_institutional | ...
    periodKey      PeriodKey       // "2026" | "2026-05" | "once" | "never"
    periodStart    time.Time       // início da janela [start, end)
    periodEnd      time.Time       // fim da janela (exclusivo)
    quotaLimit     int64           // snapshot de plan.QuotaFor(feature).Limit; -1 = unlimited
    quotaUsed      int64           // contador; invariante: 0 ≤ used ≤ limit+N (com N = margem audit)
    resetAt        *time.Time      // quando o contador será zerado pelo próximo job
    createdAt      time.Time
    updatedAt      time.Time
}
```

**Constraints SQL** (ver § Migrações):
- `id UUID PK`
- `user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE`
- `subscription_id UUID NOT NULL REFERENCES subscriptions(id) ON DELETE RESTRICT`
- `feature_key TEXT NOT NULL`
- `period_key TEXT NOT NULL`
- `quota_limit BIGINT NOT NULL CHECK (quota_limit >= -1)`
- `quota_used BIGINT NOT NULL CHECK (quota_used >= 0)`
- `period_end > period_start`
- **UNIQUE** `(user_id, feature_key, period_key)` — INV-FU-003
- `CHECK (quota_used <= quota_limit + overrun_margin)` — overrun auditável (ver INV-FU-005)

## § Value Objects

```go
// ------- FeatureUsageID -------
type FeatureUsageID string

func NewFeatureUsageID() FeatureUsageID  { return FeatureUsageID(uuidv7.New()) }

// ------- FeatureKey (reutilizado de Plan.md — mesma enum) -------
// Reiterado aqui para autocontenção:
type FeatureKey string

const (
    FeatKeyConnectMe                FeatureKey = "connect_me"
    FeatKeyVideoUploadInstitutional FeatureKey = "video_upload_institutional"
    FeatKeyVideoUploadCurriculum    FeatureKey = "video_upload_curriculum"
    FeatKeyLiveSessionMinutes       FeatureKey = "live_session_minutes"
    FeatKeyEbookDownload            FeatureKey = "ebook_download"
    FeatKeyAssemblyCreate           FeatureKey = "assembly.create"
    FeatKeyReportsExport            FeatureKey = "reports.export"
)

func (k FeatureKey) IsValid() bool {
    switch k {
    case FeatKeyConnectMe, FeatKeyVideoUploadInstitutional, FeatKeyVideoUploadCurriculum,
         FeatKeyLiveSessionMinutes, FeatKeyEbookDownload,
         FeatKeyAssemblyCreate, FeatKeyReportsExport:
        return true
    }
    return false
}

// ------- QuotaPeriod (compartilhado com Plan.md) -------
type QuotaPeriod string

const (
    QuotaPeriodMonthly QuotaPeriod = "monthly"
    QuotaPeriodAnnual  QuotaPeriod = "annual"
    QuotaPeriodOnce    QuotaPeriod = "once"    // 1-shot vitalício (ex: bônus signup)
    QuotaPeriodNever   QuotaPeriod = "never"   // feature sem cota (placeholder)
)

// ------- PeriodKey — representação human-readable do período -------
// Usado no UNIQUE(user_id, feature_key, period_key).
type PeriodKey string

func PeriodKeyAnnual(year int) PeriodKey           { return PeriodKey(fmt.Sprintf("%04d", year)) }
func PeriodKeyMonthly(year int, month time.Month) PeriodKey {
    return PeriodKey(fmt.Sprintf("%04d-%02d", year, int(month)))
}
func PeriodKeyOnce() PeriodKey  { return PeriodKey("once") }
func PeriodKeyNever() PeriodKey { return PeriodKey("never") }

// ------- QuotaKey (VO composto — chave lógica) -------
type QuotaKey struct {
    UserID     UserID
    FeatureKey FeatureKey
    Period     PeriodKey
}

func (q QuotaKey) String() string {
    return fmt.Sprintf("%s:%s:%s", q.UserID, q.FeatureKey, q.Period)
}

// CacheKey — Redis `ms:quota:{userID}:{feature}:{period}` (INV-046).
func (q QuotaKey) CacheKey() string {
    return fmt.Sprintf("ms:quota:%s:%s:%s", q.UserID, q.FeatureKey, q.Period)
}
```

## § Invariantes (testáveis)

| ID | Regra | Onde é testado |
|---|---|---|
| **INV-FU-001** | `quota_used >= 0` | CHECK SQL + factory + `Consume` |
| **INV-FU-002** | `quota_used <= quota_limit` quando `quota_limit != -1` (unlimited) — salvo overrun auditável (INV-FU-005) | factory + `Consume` |
| **INV-FU-003** | UNIQUE `(user_id, feature_key, period_key)` — uma linha por janela | UNIQUE SQL |
| **INV-FU-004** | Factory `NewFeatureUsage` **deriva `quota_limit`** de `plan.QuotaFor(feature).Limit` no momento de criar; mudança de `plan.quotas` NÃO altera retroativamente (INV-SUB-009 simétrico) | factory + test |
| **INV-FU-005** | Overrun auditável: `quota_used <= quota_limit + overrun_margin` (default `overrun_margin = 0`; configurável por feature em casos especiais). Overrun exige `AuditEntry` | `Consume(n)` retorna `ErrQuotaExceeded` se `n` excede |
| **INV-FU-006** | `period_end > period_start` | CHECK SQL + factory |
| **INV-FU-007** | Reset **idempotente**: chamar `Reset(newPeriod)` em FeatureUsage já resetado no mesmo período é no-op | test |
| **INV-FU-008** | Quando `plan.QuotaFor(feature).Limit == 0` (feature indisponível), **não cria** `FeatureUsage` — use case rejeita antes | use case CheckFeatureAccess |
| **INV-FU-009** | `Consume(n)` DEVE ser chamado dentro de `UnitOfWork.Run` com `FindByUserForUpdate` — TOCTOU (A8 fix) | contrato no repo |
| **INV-FU-010** | Quando `quota_limit == -1` (unlimited), `Consume(n)` sempre incrementa (tracking analítico) mas nunca retorna `ErrQuotaExceeded` | `Consume` |

## § Factories (New... / Reconstruct...)

```go
// NewFeatureUsage — factory canônica. Recupera limite do Plan associado à Subscription.
// O período (start/end + key) é computado por `CurrentPeriod(feature, now)`.
func NewFeatureUsage(
    userID UserID,
    sub *Subscription,
    plan *Plan,
    feature FeatureKey,
    now time.Time,
) (*FeatureUsage, error) {
    if !feature.IsValid() {
        return nil, ErrInvalidFeatureKey
    }
    rule, ok := plan.QuotaFor(feature)
    if !ok {
        return nil, ErrFeatureNotInPlan // INV-FU-008 guard
    }
    if rule.Limit == 0 {
        return nil, ErrFeatureDisabled // INV-FU-008: 0 = indisponível
    }

    start, end, periodKey := CurrentPeriod(feature, rule.Period, now)

    if !end.After(start) {
        return nil, ErrInvalidPeriodWindow // INV-FU-006
    }

    return &FeatureUsage{
        id:             NewFeatureUsageID(),
        userID:         userID,
        subscriptionID: sub.ID(),
        featureKey:     feature,
        periodKey:      periodKey,
        periodStart:    start,
        periodEnd:      end,
        quotaLimit:     rule.Limit,
        quotaUsed:      0,
        resetAt:        &end,
        createdAt:      now,
        updatedAt:      now,
    }, nil
}

// ReconstructFeatureUsage — hidratação do banco.
func ReconstructFeatureUsage(
    id FeatureUsageID,
    userID UserID,
    subscriptionID SubscriptionID,
    feature FeatureKey,
    periodKey PeriodKey,
    periodStart, periodEnd time.Time,
    quotaLimit, quotaUsed int64,
    resetAt *time.Time,
    createdAt, updatedAt time.Time,
) *FeatureUsage {
    return &FeatureUsage{
        id: id, userID: userID, subscriptionID: subscriptionID,
        featureKey: feature, periodKey: periodKey,
        periodStart: periodStart, periodEnd: periodEnd,
        quotaLimit: quotaLimit, quotaUsed: quotaUsed, resetAt: resetAt,
        createdAt: createdAt, updatedAt: updatedAt,
    }
}

// CurrentPeriod — dado a feature + período canônico + now, devolve [start, end) + key.
// Timezone UTC (evita ambiguidade de fuso horário na borda de ano/mês).
func CurrentPeriod(feature FeatureKey, period QuotaPeriod, now time.Time) (start, end time.Time, key PeriodKey) {
    u := now.UTC()
    switch period {
    case QuotaPeriodAnnual:
        year := u.Year()
        start = time.Date(year, 1, 1, 0, 0, 0, 0, time.UTC)
        end = start.AddDate(1, 0, 0)
        key = PeriodKeyAnnual(year)
    case QuotaPeriodMonthly:
        start = time.Date(u.Year(), u.Month(), 1, 0, 0, 0, 0, time.UTC)
        end = start.AddDate(0, 1, 0)
        key = PeriodKeyMonthly(u.Year(), u.Month())
    case QuotaPeriodOnce:
        start = time.Unix(0, 0).UTC()
        end = time.Date(9999, 12, 31, 23, 59, 59, 0, time.UTC)
        key = PeriodKeyOnce()
    case QuotaPeriodNever:
        start, end = u, u
        key = PeriodKeyNever()
    }
    return
}
```

## § Métodos de domínio

```go
// Getters
func (f *FeatureUsage) ID() FeatureUsageID            { return f.id }
func (f *FeatureUsage) UserID() UserID                 { return f.userID }
func (f *FeatureUsage) SubscriptionID() SubscriptionID { return f.subscriptionID }
func (f *FeatureUsage) FeatureKey() FeatureKey         { return f.featureKey }
func (f *FeatureUsage) PeriodKey() PeriodKey           { return f.periodKey }
func (f *FeatureUsage) QuotaLimit() int64              { return f.quotaLimit }
func (f *FeatureUsage) QuotaUsed() int64               { return f.quotaUsed }

// IsUnlimited — helper
func (f *FeatureUsage) IsUnlimited() bool { return f.quotaLimit == QuotaUnlimited }

// Remaining — saldo restante. -1 se unlimited.
func (f *FeatureUsage) Remaining() int64 {
    if f.IsUnlimited() { return QuotaUnlimited }
    r := f.quotaLimit - f.quotaUsed
    if r < 0 { return 0 }
    return r
}

// IsOverQuota — retorna true se adicionar N estoura a quota.
// Unlimited sempre retorna false.
func (f *FeatureUsage) IsOverQuota(n int64) bool {
    if f.IsUnlimited() { return false }
    return f.quotaUsed+n > f.quotaLimit
}

// Consume — incrementa `quota_used` em N se houver espaço.
// Unlimited: sempre incrementa (tracking analítico, retorna nil).
// Overrun: retorna ErrQuotaExceeded + NÃO muta state (chamador aborta tx).
// Requisito: chamador DEVE ter bloqueado a linha com FindByUserForUpdate (INV-FU-009).
func (f *FeatureUsage) Consume(n int64) (events.QuotaConsumed, error) {
    if n <= 0 {
        return events.QuotaConsumed{}, ErrInvalidConsumeAmount
    }
    if f.IsOverQuota(n) {
        return events.QuotaConsumed{}, ErrQuotaExceeded
    }
    f.quotaUsed += n
    f.updatedAt = time.Now().UTC()
    return events.QuotaConsumed{
        UserID:     f.userID.String(),
        FeatureKey: string(f.featureKey),
        PeriodKey:  string(f.periodKey),
        Delta:      n,
        NewUsed:    f.quotaUsed,
        Limit:      f.quotaLimit,
        OccurredAt: f.updatedAt,
    }, nil
}

// Reset — zera contador + move janela de período. Idempotente (INV-FU-007).
// Chamado por cron `quota_reset_job` em 1º/jan (annual) ou 1º do mês (monthly).
func (f *FeatureUsage) Reset(newStart, newEnd time.Time, newKey PeriodKey) (events.QuotaReset, bool) {
    if f.periodKey == newKey && f.periodStart.Equal(newStart) {
        return events.QuotaReset{}, false // idempotente
    }
    f.quotaUsed = 0
    f.periodStart = newStart
    f.periodEnd = newEnd
    f.periodKey = newKey
    f.resetAt = &newEnd
    f.updatedAt = time.Now().UTC()
    return events.QuotaReset{
        UserID:     f.userID.String(),
        FeatureKey: string(f.featureKey),
        PeriodKey:  string(newKey),
        ResetAt:    f.updatedAt,
    }, true
}

// MarkExceeded — emitido separadamente quando Consume falha (use case captura erro e emite).
// Mantido como helper caso outro caminho precise registrar overrun sem mutação.
func (f *FeatureUsage) BuildExceededEvent(attemptedDelta int64) events.QuotaExceeded {
    return events.QuotaExceeded{
        UserID:          f.userID.String(),
        FeatureKey:      string(f.featureKey),
        PeriodKey:       string(f.periodKey),
        AttemptedDelta:  attemptedDelta,
        CurrentUsed:     f.quotaUsed,
        Limit:           f.quotaLimit,
        OccurredAt:      time.Now().UTC(),
    }
}
```

## § Repository interface (Go)

```go
// internal/modules/billing/domain/repositories/i_feature_usage_repository.go
package repositories

import (
    "context"
    "time"

    "github.com/mastersindico/backend/internal/modules/billing/domain/entities"
)

type IFeatureUsageRepository interface {
    // GetCurrent — busca FeatureUsage da janela atual sem lock. Use em reads/dashboards.
    // Se não existe, retorna ErrNotFound (não cria).
    GetCurrent(ctx context.Context, userID entities.UserID, feature entities.FeatureKey) (*entities.FeatureUsage, error)

    // FindByUserForUpdate — SELECT ... FOR UPDATE. Usado DENTRO de UnitOfWork.Run antes de Consume.
    // Cria linha atomicamente se não existe (UPSERT).
    // INV-FU-009: chamar fora de tx é bug — implementação detecta via context cancel.
    FindByUserForUpdate(
        ctx context.Context,
        userID entities.UserID,
        feature entities.FeatureKey,
        periodKey entities.PeriodKey,
    ) (*entities.FeatureUsage, error)

    // IncrementUsed — short-circuit para Consume simples (sem condicionais).
    // INSERT ... ON CONFLICT DO UPDATE SET quota_used = quota_used + n.
    // Retorna erro se resultaria em quota_used > quota_limit (atomic via CHECK constraint).
    IncrementUsed(ctx context.Context, id entities.FeatureUsageID, delta int64) error

    // Save — UPSERT completo por id.
    Save(ctx context.Context, fu *entities.FeatureUsage) error

    // ResetByPeriod — batch reset para job cron.
    // Cria/atualiza linhas zeradas para todos os users que tinham FeatureUsage naquela feature.
    // Idempotente: linhas já no período novo são ignoradas.
    ResetByPeriod(
        ctx context.Context,
        feature entities.FeatureKey,
        newStart, newEnd time.Time,
        newKey entities.PeriodKey,
    ) (int64, error)

    // ListByUser — dashboard do usuário mostrando quotas atuais.
    ListByUser(ctx context.Context, userID entities.UserID) ([]*entities.FeatureUsage, error)
}
```

**Uso crítico — fluxo Consume**:

```go
// application/use_cases/consume_quota_use_case.go
func (uc *ConsumeQuotaUseCase) Run(ctx context.Context, cmd ConsumeQuotaCommand) error {
    return uc.uow.Run(ctx, func(ctx context.Context) error {
        periodKey := entities.CurrentPeriodKey(cmd.Feature, cmd.Period, time.Now().UTC())

        fu, err := uc.usageRepo.FindByUserForUpdate(ctx, cmd.UserID, cmd.Feature, periodKey)
        if err != nil { return err }

        evt, err := fu.Consume(cmd.Delta)
        if errors.Is(err, entities.ErrQuotaExceeded) {
            exceededEvt := fu.BuildExceededEvent(cmd.Delta)
            _ = uc.publisher.Publish(ctx, exceededEvt)
            return err
        }
        if err != nil { return err }

        if err := uc.usageRepo.Save(ctx, fu); err != nil { return err }
        return uc.publisher.Publish(ctx, evt)
    })
}
```

## § Domain events emitidos

```go
// internal/modules/billing/domain/events/quota_events.go

type QuotaConsumed struct {
    UserID     string
    FeatureKey string
    PeriodKey  string
    Delta      int64
    NewUsed    int64
    Limit      int64
    OccurredAt time.Time
}

type QuotaExceeded struct {
    UserID          string
    FeatureKey      string
    PeriodKey       string
    AttemptedDelta  int64
    CurrentUsed     int64
    Limit           int64
    OccurredAt      time.Time
}

type QuotaReset struct {
    UserID     string
    FeatureKey string
    PeriodKey  string  // novo período (após reset)
    ResetAt    time.Time
}
```

**Handlers canônicos**:
- `QuotaConsumed` → Redis cache invalidate (`ms:quota:{userID}:{feature}:{period}`) + analytics
- `QuotaExceeded` → notifications (email + in-app banner "quota esgotada — upgrade")  + AuditEntry (LGPD)
- `QuotaReset` → Redis cache delete (`ms:quota:{userID}:{feature}:*`) + notifications (opcional, "quota renovada")

## § Migrações SQL (tabelas + constraints)

```sql
-- migrations/20260423_0003_create_feature_usages.up.sql

CREATE TABLE feature_usages (
    id               UUID         PRIMARY KEY DEFAULT gen_random_uuid_v7(),
    user_id          UUID         NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    subscription_id  UUID         NOT NULL REFERENCES subscriptions(id) ON DELETE RESTRICT,
    feature_key      TEXT         NOT NULL,
    period_key       TEXT         NOT NULL,
    period_start     TIMESTAMPTZ  NOT NULL,
    period_end       TIMESTAMPTZ  NOT NULL,
    quota_limit      BIGINT       NOT NULL CHECK (quota_limit >= -1),
    quota_used       BIGINT       NOT NULL DEFAULT 0 CHECK (quota_used >= 0),
    reset_at         TIMESTAMPTZ,
    created_at       TIMESTAMPTZ  NOT NULL DEFAULT now(),
    updated_at       TIMESTAMPTZ  NOT NULL DEFAULT now(),

    -- INV-FU-006
    CONSTRAINT fu_period_valid CHECK (period_end > period_start),

    -- INV-FU-002 + INV-FU-005 (overrun_margin = 0 no schema; overrides em migrations específicas)
    CONSTRAINT fu_used_le_limit
        CHECK (quota_limit = -1 OR quota_used <= quota_limit)
);

-- INV-FU-003
CREATE UNIQUE INDEX feature_usages_user_feature_period_uq
    ON feature_usages (user_id, feature_key, period_key);

-- Queries frequentes
CREATE INDEX feature_usages_user_idx       ON feature_usages (user_id);
CREATE INDEX feature_usages_feature_period ON feature_usages (feature_key, period_key);
CREATE INDEX feature_usages_reset_due      ON feature_usages (reset_at)
    WHERE reset_at IS NOT NULL;

CREATE TRIGGER feature_usages_set_updated_at
    BEFORE UPDATE ON feature_usages
    FOR EACH ROW EXECUTE FUNCTION set_updated_at();
```

## § Exemplos de uso

### Consumo de Connect Me ao publicar request

```go
// application/use_cases/publish_connect_me_use_case.go
func (uc *PublishConnectMeUseCase) Run(ctx context.Context, cmd PublishCommand) error {
    return uc.uow.Run(ctx, func(ctx context.Context) error {
        // 1. resolver usage (ou criar)
        periodKey := entities.PeriodKeyAnnual(time.Now().UTC().Year())
        fu, err := uc.usageRepo.FindByUserForUpdate(ctx, cmd.UserID, entities.FeatKeyConnectMe, periodKey)
        if errors.Is(err, repoerr.ErrNotFound) {
            // primeiro uso do ano — cria
            sub, _ := uc.subRepo.FindActiveByUser(ctx, cmd.UserID)
            plan, _ := uc.planRepo.Find(ctx, sub.PlanID())
            fu, err = entities.NewFeatureUsage(cmd.UserID, sub, plan, entities.FeatKeyConnectMe, time.Now().UTC())
            if err != nil { return err }
            if err := uc.usageRepo.Save(ctx, fu); err != nil { return err }
        } else if err != nil {
            return err
        }

        // 2. consumir 1 unidade (INV-045: decrementa no publish, não no draft)
        evt, err := fu.Consume(1)
        if err != nil { return err }
        if err := uc.usageRepo.Save(ctx, fu); err != nil { return err }

        // 3. publicar domain events
        return uc.publisher.Publish(ctx, evt)
    })
}
```

### Reset anual via cron (1º/jan 00:05 UTC)

```go
// application/jobs/quota_reset_annual_job.go
func (j *QuotaResetAnnualJob) Run(ctx context.Context) error {
    now := time.Now().UTC()
    if now.Month() != time.January || now.Day() != 1 {
        return nil // safety check
    }
    start, end, key := entities.CurrentPeriod(entities.FeatKeyConnectMe, entities.QuotaPeriodAnnual, now)

    affected, err := j.usageRepo.ResetByPeriod(ctx, entities.FeatKeyConnectMe, start, end, key)
    if err != nil { return err }

    j.logger.Info().Int64("affected", affected).Msg("connect_me quotas reset for new year")

    return j.publisher.Publish(ctx, events.QuotaReset{
        FeatureKey: string(entities.FeatKeyConnectMe),
        PeriodKey:  string(key),
        ResetAt:    now,
    })
}
```

### Dashboard do usuário

```go
func (uc *GetMyUsageUseCase) Run(ctx context.Context, userID UserID) ([]UsageSnapshot, error) {
    usages, err := uc.usageRepo.ListByUser(ctx, userID)
    if err != nil { return nil, err }

    out := make([]UsageSnapshot, 0, len(usages))
    for _, fu := range usages {
        out = append(out, UsageSnapshot{
            Feature:   string(fu.FeatureKey()),
            Period:    string(fu.PeriodKey()),
            Used:      fu.QuotaUsed(),
            Limit:     fu.QuotaLimit(),
            Remaining: fu.Remaining(),
            Unlimited: fu.IsUnlimited(),
        })
    }
    return out, nil
}
```

## § Dependências com outros aggregates

- **[[Plan]]** — `plan.QuotaFor(feature)` é lido pela factory `NewFeatureUsage` para inicializar `quota_limit`. INV-FU-004: snapshot, não referência viva.
- **[[Subscription]]** — FK `subscription_id` permite rastrear qual período-de-plano gerou a usage (histórico). Também: troca de plano (`ChangePlan`) dispara recálculo via novo `NewFeatureUsage` (linha nova com `period_key` igual e `subscription_id` novo — conflito UNIQUE gerenciado por política "last wins" em `SaveChangePlan`).
- **[[User]]** — FK `user_id`; cascade on DELETE (LGPD: deletar user remove usages).
- **[[AuditEntry]]** — overrun (`QuotaExceeded`) registra `AuditEntry{action:'quota.exceeded', ...}` pra dashboard admin.

## § Referências cruzadas (STATE, ADRs, specs)

- **STATE**: [[../../STATE#D-066-decisao-arquitetural-critica-planos-globais-stripe-style]] · [[../../STATE#D-067-n1-n2-n3-e-outros-nomes-de-plano-por-persona-abolidos-completamente]] · [[../../STATE#D-058-morador-base-connect-me-0-ano-conforme-matriz]] · [[../../STATE#D-065-morador-base-connect-me-0-ano-conforme-matriz-funcional]]
- **ADR**: [[../../02-architecture/adr/0032-global-plans-stripe-style]] · [[../../02-architecture/adr/0015-uow-intra-saga-inter]] (UoW obrigatório pra Consume)
- **Requirements**: [[../../04-requirements/functional/billing]] Req 6 (quotas) — ConnectMe anual; vídeos mensal
- **Sub-produtos com quota**:
  - [[../../00-product/sub-produtos/02-connect-me]] — anual, reset 1º/jan
  - [[../../00-product/sub-produtos/04-conteudo-videos]] — mensal, reset 1º do mês
  - [[../../00-product/sub-produtos/06-lms-academia]] — download de ebook (QuotaPeriodOnce), minutos de live session (QuotaPeriodMonthly)
- **Invariantes legado (mantidos)**: INV-016 (saldo ∈ [0,∞]), INV-045 (publish decrementa, não draft), INV-046 (reset 1º/jan ou 1º/mês)
- **Backend real (cruzar na reescrita Fase 8 do sub-projeto backend)**:
  - `backend/internal/modules/billing/domain/entities/feature_usage.go` — **não tem** `subscription_id` nem `quota_limit` snapshot (busca do service em runtime); v2 adota snapshot pra auditabilidade
  - `backend/internal/modules/billing/domain/services/quota_limits.go` — matriz `role × tier` **deve ser eliminada** (era o acoplamento que D-066 rejeitou); substituir por consulta `plan.QuotaFor(feature)` direta
  - `backend/internal/modules/billing/domain/repositories/i_feature_usage_repository.go` — adicionar `IncrementUsed` e `ResetByPeriod`
  - `backend/internal/modules/billing/application/use_cases/consume_quota_use_case.go` — revisar para usar snapshot de `quota_limit` na entidade em vez de consultar `QuotaLimits` service
- **Links aggregates**: [[Plan]] · [[Subscription]] · [[User]] · [[AuditEntry]]
- **Pendência**: [[../../11-audit/AUDIT#a-fase8-fu-001-snapshot-quota-limit-vs-service-query]] (criar finding: v2 faz snapshot; backend faz query em runtime — decidir)
