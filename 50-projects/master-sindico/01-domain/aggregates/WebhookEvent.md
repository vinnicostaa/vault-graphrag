---
title: Aggregate — WebhookEvent
type: spec
tags: [domain, ddd, aggregate, cross-domain, webhook, idempotency, ledger, master-sindico, v2]
bc: cross-domain
source: 01-domain/invariants.md (INV-015, INV-018, INV-064, INV-093) + 04-requirements/global.md (GR-008, GR-009) + 04-requirements/functional/billing.md (BIL-010)
created: 2026-04-23
updated: 2026-04-23
---

# Aggregate — `WebhookEvent`

**BC**: cross-domain · **Raiz**: ✅ (ledger de idempotência de webhooks inbound)

Ledger persistente que garante idempotência em **todos** os webhooks inbound (Stripe, Mux, Livekit, Zitadel, etc). Primeira chegada de um `event_id` por `provider` → INSERT + processa; replays subsequentes → hit no ledger → 200 OK imediato sem re-processar (INV-015). **UNIQUE** `(provider, event_id)` no Postgres.

Criado como fechamento do finding **A-DC-QA-040**.

> **Decisão ADR-0027 proposta** (A-DC-PEN-016): **DB UNIQUE é a fonte da verdade** (durável), Redis é **cache de segundo nível** (performance). Se `api-design §9.1` dizia Redis-only, está errado — replay de `invoice.payment_succeeded` → fraude direta.

## Entidade raiz

```go
type WebhookEvent struct {
    id              WebhookEventID    // UUIDv7 (interno MS)
    provider        string            // "stripe" | "mux" | "livekit" | "zitadel" | "twilio" | ...
    eventID         string            // id do evento no provider (Stripe `evt_...`, Mux uuid, Zitadel uuid, ...)
    eventType       string            // "invoice.payment_succeeded", "video.asset.ready", "user.deleted", ...
    payloadHash     string            // SHA-256 do raw body (pós-HMAC verify) pra detectar payload drift em replays
    rawBody         []byte            // corpo completo (armazenado pra audit; TTL retention 90d then archive)
    receivedAt      time.Time         // wall-clock da chegada
    processedAt     *time.Time        // wall-clock pós-handler sucesso
    status          WebhookStatus     // received | processing | processed | failed | ignored
    attemptCount    int               // quantas vezes o provider re-entregou
    lastError       *string           // se failed
    handlerDuration *time.Duration    // latência do handler (observability)
    correlationID   string            // OTel trace_id
}
```

## Value Objects

```go
type WebhookEventID string // UUIDv7
type WebhookStatus string

const (
    WebhookReceived   WebhookStatus = "received"    // HMAC válido, inserido, vai processar
    WebhookProcessing WebhookStatus = "processing"  // handler rodando (transient)
    WebhookProcessed  WebhookStatus = "processed"   // sucesso terminal
    WebhookFailed     WebhookStatus = "failed"      // erro handler; provider vai retry
    WebhookIgnored    WebhookStatus = "ignored"     // event.type fora da whitelist; 200 OK sem processar
)
```

## Invariantes

- **INV-015**: Stripe `webhook_events.event_id` UNIQUE (idempotency ledger) — generalizado a todos os providers: UNIQUE `(provider, event_id)`
- **INV-018**: HMAC verificado **antes** de parse body (Stripe) — generalizado
- **INV-064**: Mux webhook HMAC verificado antes de parse — idem
- **INV-093**: HMAC verify antes de parse em todo webhook — pattern obrigatório
- Body size limit: `MaxBytesReader(r.Body, 64KB)` antes de qualquer ler (A-DC-SEC-002 mitigação aplicada a todos os providers, não só livekit)
- Replay detection: segundo INSERT com mesmo `(provider, event_id)` retorna `ErrConflict` → handler retorna 200 OK imediato sem re-processar
- `rawBody` mantém bytes originais pra: (a) audit cross-check; (b) re-verify HMAC em replay debug; (c) re-process manual via admin AD8 em caso de bug de handler

## Factories

```go
// NewWebhookEvent — chamado dentro do middleware após HMAC verify + body read
func NewWebhookEvent(provider, eventID, eventType string, rawBody []byte, correlationID string) (*WebhookEvent, error) {
    // valida: provider em whitelist conhecida; eventID != ""; payloadHash computado
}

func ReconstructWebhookEvent(...) *WebhookEvent
```

## Métodos de domínio

- `MarkProcessing()` — `received → processing`
- `MarkProcessed(duration time.Duration)` — `processing → processed`; seta `processedAt`
- `MarkFailed(err error)` — `processing → failed`
- `MarkIgnored(reason string)` — event_type fora whitelist; `received → ignored`
- `IsProcessed() bool` — true se `status == processed`
- `PayloadMatches(rawBody []byte) bool` — compara SHA-256 hash; detecta tampering em replay

## Repository interface (Go)

```go
// domain/shared/webhook_event_repository.go
type IWebhookEventRepository interface {
    Save(ctx context.Context, e *WebhookEvent) error                        // INSERT; ErrConflict se UNIQUE violado → replay detected
    FindByProviderEventID(ctx context.Context, provider, eventID string) (*WebhookEvent, error) // pra replay fast-path
    MarkProcessed(ctx context.Context, id WebhookEventID, at time.Time, dur time.Duration) error
    MarkFailed(ctx context.Context, id WebhookEventID, err string) error
    ListFailed(ctx context.Context, provider string, page PageArgs) ([]*WebhookEvent, error) // DLQ admin AD8
    Retry(ctx context.Context, id WebhookEventID) error                     // admin manual retry
}
```

## Pattern canônico de handler

```go
func (h *StripeWebhookHandler) Handle(ctx context.Context, w http.ResponseWriter, r *http.Request) {
    // 1. Body size limit (A-DC-SEC-002)
    r.Body = http.MaxBytesReader(w, r.Body, 64*1024)

    // 2. Read raw body (HMAC verify before parse — INV-093)
    rawBody, err := io.ReadAll(r.Body)
    if err != nil { http.Error(w, "body too large", 413); return }

    // 3. HMAC verify (SDK Stripe / constant-time compare)
    event, err := webhook.ConstructEvent(rawBody, r.Header.Get("Stripe-Signature"), h.secret)
    if err != nil { http.Error(w, "invalid signature", 400); return }

    // 4. Idempotency ledger INSERT (INV-015)
    we, err := NewWebhookEvent("stripe", event.ID, event.Type, rawBody, correlationID(ctx))
    if err != nil { http.Error(w, "internal", 500); return }
    if err := h.repo.Save(ctx, we); err != nil {
        if errors.Is(err, ErrConflict) {
            // Replay detectado — retorna 200 sem reprocessar
            w.WriteHeader(200); return
        }
        http.Error(w, "internal", 500); return
    }

    // 5. Despacha handler específico do event.type (dentro de UoW)
    start := time.Now()
    err = h.dispatch(ctx, event)
    dur := time.Since(start)
    if err != nil {
        h.repo.MarkFailed(ctx, we.id, err.Error())
        http.Error(w, "handler failed, Stripe will retry", 500); return
    }

    // 6. Marca processed
    h.repo.MarkProcessed(ctx, we.id, time.Now(), dur)
    w.WriteHeader(200)
}
```

## Dedup janela

- UNIQUE `(provider, event_id)` **permanente** no Postgres. Replays ilimitados (Stripe pode replay 72h, mas o ledger é perpétuo).
- Retention do `rawBody`: 90 dias em Postgres, depois arquivar em MinIO comprimido (cold storage).
- Metadados (id, provider, event_id, event_type, status, timestamps) ficam permanentes em hot DB pra auditoria.

## DLQ + admin recovery

- `status == failed` após N tentativas do provider → entra em dashboard "Webhook DLQ" (AD8 Billing admin).
- Admin pode: (a) inspect raw body; (b) re-run handler manual (útil se foi bug fixed); (c) marcar `ignored` definitivo (business decision).

## Eventos emitidos

Não emite domain events (é ledger técnico). Emite métricas:
- `webhook_events_received_total{provider, event_type}`
- `webhook_events_processed_total{provider, status}`
- `webhook_event_handler_duration_seconds{provider, event_type}`

## ⚠️ Pendências

- **P-WE-001**: consolidar whitelist de `event_type` por provider em config (YAML) ao invés de hardcoded — ver BIL-011 pendência.
- **P-WE-002**: HMAC `length-diff timing leak` (A-DC-PEN-011) — confirmar uso de `hmac.Equal` (constant-time) em todos providers.
- **P-WE-003**: Zitadel webhook `user.deleted` race com Stripe `invoice.payment_succeeded` (A-DC-SEC-004) — resolver via soft-flag `pending_hard_delete` de 48h (ver [[User]] e [[../../90-ingested/_consolidado-providers-fluxos#fluxos-inbound-webhooks-zitadel-a-dc-qa-050-expandido]]).

## Links

- [[../bounded-contexts#7-cross-domain]]
- [[../invariants#cross-domain-inv-086-a-inv-100]] (INV-093)
- [[../invariants#billing-inv-011-a-inv-020]] (INV-015, INV-018)
- [[../invariants#content-inv-056-a-inv-070]] (INV-064)
- [[../../04-requirements/global]] (GR-008, GR-009)
- [[../../04-requirements/functional/billing]] (BIL-009, BIL-010)
- [[../../90-ingested/_consolidado-providers-fluxos]] §todos webhook inbound
- [[OutboxEntry]] — padrão análogo pra outbound (dedup no lado consumer)
- [[DomainEvent]] — estrutura canônica
- [[AuditEntry]] — handlers sensíveis geram audit
