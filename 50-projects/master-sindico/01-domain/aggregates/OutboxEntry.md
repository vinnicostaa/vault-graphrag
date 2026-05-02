---
title: Aggregate — OutboxEntry
type: spec
tags: [domain, ddd, aggregate, cross-domain, outbox, nats, at-least-once, master-sindico, v2]
bc: cross-domain
source: 01-domain/invariants.md (INV-095) + 04-requirements/functional/cross-domain.md (XD-002, XD-031) + 02-architecture/patterns.md
created: 2026-04-23
updated: 2026-04-23
---

# Aggregate — `OutboxEntry`

**BC**: cross-domain · **Raiz**: ✅ (registro do outbox pattern — durabilidade at-least-once)

Registro persistente na tabela `domain_events_outbox` pra implementar o **transactional outbox pattern**: evento gravado na mesma tx que a mudança de estado do aggregate, drenado async por worker NATS JetStream. Garante at-least-once delivery sem two-phase-commit.

Criado como fechamento do finding **A-DC-QA-040**. Análogo a [[DomainEvent]] mas focado no **estado de publicação** (pending/published/failed/dlq) vs. o payload/identidade do evento.

## Entidade raiz

```go
type OutboxEntry struct {
    id                OutboxEntryID    // UUIDv7 (mesmo do DomainEvent.id quando 1:1)
    eventID           string           // FK lógica para DomainEvent.id
    eventType         string           // denormalizado pra roteamento (NATS subject)
    natsSubject       string           // "ms.identity.user.created", "ms.billing.subscription.activated", ...
    payload           []byte           // JSON snapshot do DomainEvent.payload (imutável)
    status            OutboxStatus     // pending | published | failed | dlq
    attemptCount      int
    lastAttemptAt     *time.Time
    lastError         *string          // stack trace truncado (sem PII — INV-092)
    publishedAt       *time.Time       // quando NATS ack
    dlqAt             *time.Time       // quando moveu pra DLQ após maxAttempts
    createdAt         time.Time
    scheduledAt       time.Time        // usually == createdAt; pode ser delayed (ex: retry com backoff)
    correlationID     string           // propagado pro OTel trace
}
```

## Value Objects

```go
type OutboxEntryID string // UUIDv7
type OutboxStatus string

const (
    OutboxPending   OutboxStatus = "pending"
    OutboxPublished OutboxStatus = "published"
    OutboxFailed    OutboxStatus = "failed"  // transient, retry
    OutboxDLQ       OutboxStatus = "dlq"     // terminal, manual intervention
)
```

## Invariantes

- **INV-095**: OutboxEntry gravado **dentro** da mesma tx que o aggregate estado (atomicidade) — violar quebra garantia at-least-once
- `status` progride: `pending → published` (happy) OU `pending → failed → ... → dlq` (após N retries — default 10)
- `publishedAt` NULL enquanto `status in {pending, failed, dlq}`
- `attemptCount` monotônico ≥ 0; resetado nunca (mesmo em manual retry de DLQ → novo entry)
- Drain worker NATS é **idempotente** — replay de `pending` não duplica no consumer (consumer faz dedup via `webhook_events` ledger no downstream — INV-015)
- NATS subject determinístico a partir de `eventType`: `ms.<bc>.<aggregate>.<action>` (ex: `ms.billing.subscription.activated`)

## Factories

```go
// NewOutboxEntry — chamado quando UoW está commitando o aggregate
func NewOutboxEntry(eventID string, eventType, natsSubject string, payload []byte, corID string) (*OutboxEntry, error) {
    // valida: eventID não vazio (deve ser UUIDv7), eventType mapeado pra subject conhecido
}

func ReconstructOutboxEntry(...) *OutboxEntry
```

## Métodos de domínio

- `MarkPublished(at time.Time)` — `pending → published` após NATS ack
- `MarkFailed(err error)` — incrementa `attemptCount`; seta `status=failed` se < maxAttempts, senão → `dlq`
- `ShouldRetry(now time.Time, maxAttempts int, backoff BackoffFn) bool` — drain worker consulta antes de tentar
- `NextAttemptAt(now time.Time, backoff BackoffFn) time.Time` — exponential backoff 1s, 2s, 4s, ..., max 5min

## Repository interface (Go)

```go
// domain/shared/outbox_repository.go
type IOutboxRepository interface {
    Save(ctx context.Context, e *OutboxEntry) error                              // INSERT dentro da tx
    FindPending(ctx context.Context, limit int, now time.Time) ([]*OutboxEntry, error) // drain worker poll; query: `WHERE status='pending' AND scheduled_at <= :now ORDER BY created_at LIMIT :limit FOR UPDATE SKIP LOCKED`
    FindFailedReady(ctx context.Context, limit int, now time.Time) ([]*OutboxEntry, error) // status=failed & retry time chegou
    MarkPublished(ctx context.Context, id OutboxEntryID, at time.Time) error
    MarkFailed(ctx context.Context, id OutboxEntryID, err string, nextAt time.Time) error
    MarkDLQ(ctx context.Context, id OutboxEntryID, at time.Time, lastErr string) error
    CountByStatus(ctx context.Context) (map[OutboxStatus]int, error)            // dashboard alerta DLQ size
}
```

**Query-chave** `FindPending` usa `FOR UPDATE SKIP LOCKED` (Postgres 18) pra permitir N drain workers concorrentes sem duplicação.

## Drain worker (arquitetura)

```
┌──────────────┐   poll    ┌─────────────┐   publish   ┌───────────┐
│ API/Handler  │─tx─▶ DB ◀─────────────│ DrainWorker │────────────▶│ NATS JetS │
└──────────────┘           └─────────────┘    ack       └───────────┘
                                                              │
                                                              ▼
                                                        ┌───────────┐
                                                        │ Consumers │
                                                        │ (N BCs)   │
                                                        └───────────┘
```

- **Poll interval**: 100ms (tunable).
- **Batch size**: 50.
- **Workers**: N replicas (Kubernetes deployment) com `SKIP LOCKED` garantindo not-double-processing.
- **Observability**: Prometheus `outbox_pending_count`, `outbox_publish_latency_seconds`, `outbox_dlq_size`. Alerta quando `pending > 1000` ou `dlq > 0`.

## DLQ (Dead Letter Queue)

- Entries que falharam `maxAttempts` vezes (default 10).
- Operador manual: console AD8 Billing admin tem aba "Webhook DLQ" pra inspect + retry manual.
- Após análise: re-insere nova OutboxEntry (novo id) com payload original corrigido (se bug de schema) ou descarta.

## Eventos relacionados (meta)

Este aggregate **não emite** eventos (recursão). Gera métricas:
- `outbox_entries_created_total{event_type}`
- `outbox_entries_published_total{event_type}`
- `outbox_entries_dlq_total{event_type}`

## Retention

- `published`: manter 7d pra debug, depois arquivar em MinIO.
- `dlq`: manter indefinidamente até operador resolver (auditável).
- `pending`/`failed`: sem retention (sempre ativos).

## ⚠️ Pendências

- **P-OB-001**: `maxAttempts` por tipo de evento? Alguns críticos (billing) deveriam ter mais retries; outros (analytics) menos. Proposta: config por `eventType`.
- **P-OB-002**: ordering garantia — NATS JetStream preserva por `stream` mas não entre streams. Aggregates que precisam ordering global (raro — ex: audit) devem usar stream dedicado.
- **P-OB-003**: `scheduled_at` delay para eventos "futuros" (ex: trial_ending T-24h) → atualmente cron separado; avaliar se outbox scheduled funciona.

## Links

- [[../bounded-contexts#7-cross-domain]]
- [[../invariants#cross-domain-inv-086-a-inv-100]] (INV-095)
- [[../../04-requirements/functional/cross-domain]] (XD-002, XD-031)
- [[DomainEvent]] — evento canônico que vira outbox entry
- [[WebhookEvent]] — dedup no lado consumer (idempotência INV-015)
- [[../../02-architecture/patterns]] — transactional outbox pattern
