---
title: ADR-0027 — Webhook idempotency via DB UNIQUE (fonte-de-verdade) + Redis como cache L1
type: adr
tags: [adr, webhook, idempotency, stripe, mux, livekit, master-sindico, v2]
status: accepted
date: 2026-04-23
created: 2026-04-23
updated: 2026-04-23
aliases: [webhook-idempotency-db, idempotency-db-source-of-truth]
---

# ADR-0027 — Webhook idempotency via DB UNIQUE + Redis cache L1

## Status

accepted — 2026-04-23

## Contexto

Finding [[../../11-audit/audit-log/2026-04-23-pentest-specs#A-DC-PEN-016]] (🔴 M1 bloqueante) identificou **inconsistência** entre dois artefatos da spec:

- [[../api-design#9.1 Handler pattern]] codifica idempotência via **Redis `SetNX`** como primary (`webhook:<provider>:<event_id>` TTL 24h).
- `BIL-010` (billing functional) + [[../../01-domain/invariants#INV-015]] codificam **DB UNIQUE** em `webhook_events(provider, event_id)`.

**Cenário de exploração** (pentester):
1. Redis é volátil (Railway M1 sem persistência AOF por default).
2. Redis crash + restart = chave perdida → próximo retry do provider (Stripe reenvia até 3 dias) **é tratado como novo**.
3. Replay de `invoice.payment_succeeded` **dobra crédito** se handler não for idempotente no efeito.
4. Mesma janela aplica-se a `video.asset.ready` (publica vídeo não-moderado 2x) e `egress.room_finished` (grava ata duplicada).

**Impacto direto**: 🔴 fraude financeira, violação LGPD (ata duplicada = repúdio), custos operacionais em reprocessamento.

Tanto [[../../11-audit/dual-check-log/2026-04-23-security-gaps]] quanto [[../../11-audit/audit-log/2026-04-23-pentest-specs]] convergem: **DB tem que ser fonte-de-verdade**.

## Decisão

**DB UNIQUE constraint é fonte-de-verdade da idempotência**. Redis passa a ser **cache L1 opcional** (fast-path para webhooks em alta frequência); na ausência do Redis, DB sozinho cumpre o contrato.

### Schema canônico

```sql
CREATE TABLE webhook_events (
    id              BYTEA PRIMARY KEY,          -- ULID (ADR-0009)
    provider        TEXT NOT NULL,              -- 'stripe' | 'mux' | 'livekit' | 'zitadel'
    event_id        TEXT NOT NULL,              -- event.id do provider (ex: 'evt_1Jx...')
    event_type      TEXT NOT NULL,              -- 'invoice.payment_succeeded' | ...
    payload_hash    TEXT NOT NULL,              -- SHA256 do raw body (detecta replay com payload alterado)
    received_at     TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    processed_at    TIMESTAMPTZ,                -- NULL = recebido mas ainda não processado
    status          TEXT NOT NULL DEFAULT 'pending',  -- pending | processed | failed | skipped
    error_message   TEXT,
    attempts        INT NOT NULL DEFAULT 0,

    -- 🔴 constraint composta — previne colisão A-DC-PEN-018 (event_id entre providers)
    UNIQUE (provider, event_id)
);

CREATE INDEX idx_webhook_events_unprocessed
    ON webhook_events(received_at)
    WHERE processed_at IS NULL;
```

### Handler pattern canônico (substitui [[../api-design#9.1]] anterior)

```go
func (h *Webhook) Handle(c *gin.Context) {
    // 0) Body size guard (cobre A-DC-SEC-002)
    r := http.MaxBytesReader(c.Writer, c.Request.Body, 64*1024) // 64KB para webhooks
    body, err := io.ReadAll(r)
    if err != nil { c.Status(http.StatusRequestEntityTooLarge); return }

    // 1) HMAC antes de parse (INV-093)
    if !verifyHMAC(body, c.GetHeader(hmacHeader), h.secret) {
        c.Status(http.StatusBadRequest); return
    }

    // 2) Parse
    var evt ProviderEvent
    if err := json.Unmarshal(body, &evt); err != nil {
        c.Status(http.StatusBadRequest); return
    }

    payloadHash := sha256Hex(body)

    // 3) Redis fast-path (L1 cache, opcional)
    cacheKey := fmt.Sprintf("ms:webhook-dedup:%s:%s", h.provider, evt.ID)
    if cached, err := h.redis.Get(c, cacheKey).Result(); err == nil && cached == payloadHash {
        c.Status(http.StatusOK); return // hit: já processado
    }

    // 4) DB UNIQUE = fonte-de-verdade
    err := h.repo.InsertEventOrSkip(c, WebhookEvent{
        Provider:    h.provider,
        EventID:     evt.ID,
        EventType:   evt.Type,
        PayloadHash: payloadHash,
    })
    switch {
    case errors.Is(err, ErrConflict):
        // Já existe: idempotência kick in — retorna 200 sem reprocessar
        _ = h.redis.Set(c, cacheKey, payloadHash, 24*time.Hour).Err() // re-alimenta cache
        c.Status(http.StatusOK); return
    case err != nil:
        c.Status(http.StatusInternalServerError); return
    }

    // 5) Processa dentro de UoW (inclui audit + effects)
    if err := h.cmd.Handle(c, ProcessWebhookCommand{Provider: h.provider, Event: evt}); err != nil {
        h.repo.MarkFailed(c, evt.ID, err.Error())
        c.Status(http.StatusInternalServerError); return
    }

    // 6) Marca processado + popula cache
    _ = h.repo.MarkProcessed(c, evt.ID)
    _ = h.redis.Set(c, cacheKey, payloadHash, 24*time.Hour).Err()

    // 7) ACK ao provider
    c.Status(http.StatusOK)
}
```

### Regras derivadas

1. **Redis é opcional**: em degraded mode (Redis down), handler continua correto via DB. Latência sobe de ~1ms (Redis hit) para ~5-10ms (DB `ON CONFLICT` lookup). Aceitável.
2. **Payload hash**: se provider reemite `event.id` com body diferente (muito raro, mas contempla bug upstream ou tampering), handler loga + rejeita 400. Cobre [[../../11-audit/audit-log/2026-04-23-pentest-specs#A-DC-PEN-027]] (Idempotency-Key reuse).
3. **Partitioning**: `webhook_events` partitioned monthly (Goose migration) — suporta alto volume sem index bloat.
4. **Retention**: 3 anos para `stripe_*` (compliance financeiro), 1 ano para os demais.
5. **Janela timestamp HMAC** reduzida de 5 min → **2 min** (cobre [[../../11-audit/audit-log/2026-04-23-pentest-specs#A-DC-PEN-017]]).

## Consequências

### Positivas

- Fecha A-DC-PEN-016 (idempotência inconsistente Redis vs DB).
- Fecha A-DC-PEN-018 (event_id collision entre providers).
- Reconcilia [[../api-design]] com [[../../01-domain/invariants#INV-015]] e BIL-010.
- Durabilidade garantida — Redis crash é non-event.
- `payload_hash` detecta replay com payload tampering.

### Negativas

- Latência marginalmente maior sem cache (~5-10ms), aceitável dado que webhooks não são path crítico de UI.
- Migration `webhook_events` precisa ser particionada desde M1 (não refator pós-scale).
- Dev discipline: todo novo provider precisa registrar em `webhook_events` via repo dedicado (não `INSERT` ad-hoc).

## Regras

1. **Nenhum handler de webhook usa Redis como primary idempotency** — Redis é fast-path só.
2. **UNIQUE SEMPRE composta `(provider, event_id)`** — nunca só `event_id` ([[../../01-domain/invariants#INV-015]] deprecado em favor de composite).
3. **MaxBytesReader 64KB** em todos os webhooks (fecha A-DC-SEC-002 + padroniza limit).
4. **`verify-then-insert-then-process`** é ordem inegociável — jamais inverter.
5. **Alerta dedup rate**: se `(processed / received) < 0.95` por 15 min → Sentry + on-call.

## Alternativas consideradas

1. **Manter Redis-only** (spec original [[../api-design#9.1]]) — rejeitado pelo finding A-DC-PEN-016 (não-durável).
2. **Redis persistente (AOF)** — Railway M1 não oferece; mesmo que oferecesse, DB é mais robusto e tem UNIQUE nativo.
3. **Kafka/NATS consumer group dedup** — overkill em M1 (NATS só é consumer interno; webhooks são inbound HTTP).
4. **Idempotency key no header** (padrão Stripe) — providers emitem `event.id` único nativo; usar o `event.id` é equivalente e evita coupling com header.

## Referências

- Finding: [[../../11-audit/audit-log/2026-04-23-pentest-specs#A-DC-PEN-016]] (🔴 M1 bloqueante).
- Finding: [[../../11-audit/audit-log/2026-04-23-pentest-specs#A-DC-PEN-018]] (composite UNIQUE).
- Finding: [[../../11-audit/audit-log/2026-04-23-pentest-specs#A-DC-PEN-027]] (Idempotency-Key reuse DoS).
- [[../api-design#9. Webhook inbound (pattern canônico)]] — atualizado nesta ADR.
- [[../../01-domain/invariants#INV-015]] — invariante reconciliado (composite UNIQUE).
- [[../../01-domain/invariants#INV-105]] — invariante novo (DB primary).
- [[../../04-requirements/global#GR-009]] — requirement já exige DB UNIQUE; ADR formaliza.
- [[0015-uow-intra-saga-inter]] — `ProcessWebhookCommand` roda em UoW.
- Stripe idempotency: https://docs.stripe.com/webhooks/signatures + https://docs.stripe.com/api/idempotent_requests
- CWE-837 (Improper Enforcement of Behavioral Workflow): https://cwe.mitre.org/data/definitions/837.html
