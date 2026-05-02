---
title: ADR-0015 — UoW intra-módulo, Saga inter-módulo/provider externo
type: adr
tags: [adr, architecture, uow, saga, master-sindico, v2]
status: accepted
date: 2026-04-23
created: 2026-04-23
updated: 2026-04-23
---

# ADR-0015 — UoW intra-módulo, Saga inter-módulo

## Status

accepted — 2026-04-23 (herdado de legado A-027, A-028) — refinado em [[0030-saga-orchestration-default]] (2026-04-23 update abaixo).

## 2026-04-23 update — Saga orchestration explícita como default

Pós-[[../../11-audit/audit-log/2026-04-23-pentest-specs#A-DC-PEN-015]] (🟠 high — cross-BC consumer sem revalidar tenant) e [[../../11-audit/dual-check-log/2026-04-23-decisoes-arquiteturais]] (ressalva sobre ausência de explicitação), **orchestration passa a ser o modelo canônico default** para sagas.

**Mudança**:

- **Orchestration** (coordinator explícito em `application/sagas/`) é o default — ver [[0030-saga-orchestration-default]] para regras detalhadas + checklist.
- **Choreography** só é permitido em 3 cenários específicos (broadcast analítico, cache invalidation, notification fan-out) com justificativa no ADR de feature.
- **Consumer** (mesmo em choreography) **sempre** valida: `event.tenant_id` contra dados do próprio evento, HMAC interno do outbox, idempotência via `UNIQUE(event_id)`.
- Nova saga canônica adicionada M1: **`AccountDeletionSaga`** (cobre [[../../11-audit/dual-check-log/2026-04-23-security-gaps#A-DC-SEC-004]] — race Stripe pós-hard-delete).

**Findings cobertos**: A-DC-PEN-015 (tenant revalidation em consumer), A-DC-SEC-004 (LGPD hard-delete race).

## Contexto

Transações atômicas Postgres (ACID) cobrem writes em 1 BC. Mas:

- Use case atravessa provider externo (Stripe/Mux/LiveKit) — não há rollback distribuído.
- Use case cruza BCs — in-process ainda cabe em 1 tx, mas semântica de saga (compensação explícita) é melhor para debug e auditoria.

## Decisão

### UoW intra-módulo (1 BC)

`UnitOfWork.Run(ctx, fn)` envolve operações multi-aggregate do mesmo BC numa tx Postgres. Inclui INSERT em `outbox_events`.

```go
// application/ports/unit_of_work.go
type UnitOfWork interface {
    Run(ctx context.Context, fn func(ctx context.Context) error) error
}

// infrastructure/database/uow_impl.go
type pgUoW struct{ pool *pgxpool.Pool }
func (u *pgUoW) Run(ctx, fn) error {
    tx, err := u.pool.Begin(ctx); if err != nil { return err }
    ctx = context.WithValue(ctx, txKey{}, tx)
    if err := fn(ctx); err != nil { _ = tx.Rollback(ctx); return err }
    return tx.Commit(ctx)
}
```

### Saga inter-módulo / provider externo

Quando atravessa BC ou chama provider, usar **Saga com compensação explícita** (ver [[../patterns]] §7).

Estado em `saga_instances`:

```sql
CREATE TABLE shared.saga_instances (
    id            BYTEA PRIMARY KEY,
    tenant_id     BYTEA NOT NULL,
    saga_type     TEXT NOT NULL,
    aggregate_id  BYTEA NOT NULL,
    state         TEXT NOT NULL,
    context       JSONB NOT NULL,
    created_at    TIMESTAMPTZ NOT NULL DEFAULT now(),
    updated_at    TIMESTAMPTZ NOT NULL DEFAULT now(),
    completed_at  TIMESTAMPTZ,
    failed_at     TIMESTAMPTZ,
    last_error    TEXT
);
```

Sagas canônicas M1/M3:

- `ContractSaga` — RFP → Proposal → Contract → Milestones (inter-BC commercial↔institutional).
- `VideoPublishSaga` — upload → transcode → moderate → publish → timeline (content + cross-domain).
- `AssemblySaga` — schedule → CreateRoom → Egress → record → publish Minutes (assembly + LiveKit + content).
- `TrialExpirySaga` — quando trial expira: update billing + downgrade features + email user (billing + identity + content).

## Consequências

### Positivas

- ACID mantido onde PG pode oferecer; saga onde só compensação semântica é possível.
- Estado da saga persistido = debuggable.
- Evita "distributed transaction" que Stripe/Mux não suportam.
- Compensação explícita força dev a pensar em rollback.

### Negativas

- Escrever compensação é trabalho extra por step.
- Saga que trava (webhook que nunca vem) requer timeout + intervenção manual.
- Consistency is eventual — UI precisa mostrar "processing" quando saga ativa.

## Regras

1. **UoW só dentro do BC** — se command toca 2 BCs, vira saga.
2. **Provider externo = saga** — mesmo se chama 1 só, documenta compensação.
3. **Saga persistida antes de 1º step** — worker retoma de estado após crash.
4. **Timeout por step** — se LiveKit Egress não retorna em 2h, saga entra em `Compensating_Manual`.

## Alternativas consideradas

1. **2PC (two-phase commit)** — Stripe/Mux não suportam.
2. **Outbox pattern sozinho** — ok para side-effects; não coordena fluxo multi-step com compensação.
3. **Event choreography pura** — eventos emitem eventos; difícil debug em fluxos complexos. Escolhemos orquestração (saga coordinator).
4. **Temporal.io** — ótimo, mas custo adicional em M1; reavaliar M3+ se volume de sagas justificar.

## Referências

- [[../patterns]] §6 (UoW), §7 (Saga)
- [[../event-driven]] §10
- [[0030-saga-orchestration-default]] — orchestration default + regras de choreography.
- [[0027-webhook-idempotency-db]] — idempotency dentro de sagas com webhook provider.
- [[../../11-audit/audit-log/2026-04-23-pentest-specs#A-DC-PEN-015]] — finding cobrindo consumer revalidation.
- [[../../11-audit/dual-check-log/2026-04-23-security-gaps#A-DC-SEC-004]] — AccountDeletionSaga requirement.
- Histórico legado: A-027, A-028.
