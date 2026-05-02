---
title: Runbook — Reprocessar webhooks (Stripe / Mux / Livekit)
type: guide
tags: [runbook, webhooks, dlq, stripe, mux, livekit, master-sindico, v2]
source: 90-ingested/_consolidado-providers-fluxos.md + 06-execution/incident.md
created: 2026-04-23
updated: 2026-04-23
---

# Runbook — Reprocessar webhooks

Quando webhooks de providers (Stripe, Mux, Livekit, Twilio inbound) falham em processar — ou provider passou por outage e fila backlog — precisamos reprocessar.

**Dados críticos**: Stripe (pagamento) > Mux (asset ready) > Livekit (room events) > Twilio (inbound SMS).

---

## Quando rodar

### Gatilhos

- Alert Grafana: `webhook.<provider>.fail_rate > 5%` por 1h
- Alert Sentry: pico de errors em webhook handler
- DLQ (Dead Letter Queue) crescendo: `SELECT COUNT(*) FROM webhook_dlq WHERE processed = false > 0`
- Provider status page reporta outage (tópico depois reabre)
- Cliente reclama: "paguei mas não apareceu assinatura" (Stripe)
- Cliente reclama: "vídeo subido mas está em processing há horas" (Mux)
- Assembleia live reporta: "participants não entram" (Livekit)

---

## Arquitetura do webhook handling

Todo webhook handler segue pattern padrão:

```
1. Receber POST
2. Read raw body (ANTES de parse)
3. HMAC verify (I-053)
4. Parse event
5. Idempotency check (event.id no ledger)
6. Execute use case
7. If use case fails:
     - Log error
     - Insert em webhook_dlq (table append-only)
     - Return 5xx (provider retry)
8. If success:
     - Mark event.id como processed no ledger
     - Return 200
```

Tabela `webhook_dlq`:
```sql
CREATE TABLE webhook_dlq (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  provider VARCHAR(32) NOT NULL,  -- 'stripe', 'mux', 'livekit', 'twilio'
  event_id VARCHAR(255) NOT NULL, -- provider-specific
  event_type VARCHAR(128) NOT NULL,
  raw_payload JSONB NOT NULL,
  received_at TIMESTAMPTZ NOT NULL,
  last_attempt_at TIMESTAMPTZ,
  attempt_count INT NOT NULL DEFAULT 0,
  last_error TEXT,
  processed BOOLEAN NOT NULL DEFAULT false,
  processed_at TIMESTAMPTZ,
  UNIQUE(provider, event_id)
);
```

Cron `webhook-dlq-retry` (a cada 5min):
```sql
SELECT * FROM webhook_dlq 
WHERE processed = false 
  AND (last_attempt_at IS NULL OR last_attempt_at < NOW() - INTERVAL '5 minutes')
  AND attempt_count < 10
ORDER BY received_at;
```

Após 10 tentativas: alerta pra intervenção manual.

---

## Cenários

### Cenário 1: Stripe webhook falhou (um evento específico)

**Sintoma**: pagamento aprovado na Stripe mas Subscription não ativada localmente.

**Debug**:
```sql
-- Ver status do evento no ledger
SELECT * FROM stripe_webhook_events WHERE event_id = 'evt_XXX';

-- Ver DLQ
SELECT * FROM webhook_dlq 
WHERE provider = 'stripe' AND event_id = 'evt_XXX';

-- Ver audit trail
SELECT * FROM audit_trail 
WHERE action LIKE 'stripe:%' AND metadata->>'event_id' = 'evt_XXX';
```

**Reprocess**:
```bash
# Triggar retry manual via endpoint admin
curl -X POST https://api.mastersindico.com.br/admin/webhooks/reprocess \
  -H "Authorization: Bearer $ADMIN_TOKEN" \
  -d '{"provider": "stripe", "event_id": "evt_XXX"}'

# OU re-entrega via Stripe dashboard
# https://dashboard.stripe.com/webhooks
# → select endpoint → Recent deliveries → failed event → "Resend"
```

**Validar**: `subscriptions.status = 'active'` + audit entry nova.

---

### Cenário 2: Mux asset stuck em processing

**Sintoma**: vídeo uploaded, mas status local = `processing` há > 30min.

**Debug**:
```bash
# Ver status direto no Mux
curl -u "$MUX_TOKEN:$MUX_SECRET" \
  https://api.mux.com/video/v1/assets/<asset_id>

# Se Mux diz "ready" mas nosso DB diz "processing":
# webhook asset.ready não chegou ou falhou
```

```sql
-- Local state
SELECT status, mux_asset_id, mux_playback_id 
FROM videos WHERE mux_asset_id = '<asset_id>';

-- DLQ
SELECT * FROM webhook_dlq 
WHERE provider = 'mux' 
  AND raw_payload->'data'->>'id' = '<asset_id>';
```

**Reprocess**:
```bash
# Resync status manual (via admin endpoint)
curl -X POST https://api.mastersindico.com.br/admin/videos/<video_id>/resync \
  -H "Authorization: Bearer $ADMIN_TOKEN"
```

Alternativa: re-entrega via Mux dashboard → Webhooks → select → Resend event.

---

### Cenário 3: Livekit room events perdidos

**Sintoma**: assembleia terminou, mas `attendance_records` não atualizou pra alguns participants.

**Debug**:
```bash
# Livekit API: ver gravação + participants
livekit-cli list-room-participants --identity assembly:<assembly_id>

# Comparar com DB local
```

```sql
SELECT voter_id, check_in_time 
FROM attendance_records 
WHERE assembly_id = '<id>';
```

**Reprocess**:

Livekit webhooks (`room_started`, `participant_joined`, etc.) são efêmeros — perdidos não voltam via resend. Mitigação:
- Pull final state via Livekit API no `AssemblyClosed` event handler (defensivo)
- Se perdeu: `UPDATE attendance_records` manualmente via admin (audit log obrigatório)

Task de melhoria: implementar **pull periódico** durante room active (Sprint 14+).

---

### Cenário 4: Provider outage — lote grande de webhooks em DLQ

**Sintoma**: Stripe anunciou 30min outage; 50+ eventos na DLQ pós-restore.

**Reprocess em batch**:

```bash
# Admin endpoint pra processar tudo
curl -X POST https://api.mastersindico.com.br/admin/webhooks/reprocess-batch \
  -H "Authorization: Bearer $ADMIN_TOKEN" \
  -d '{
    "provider": "stripe",
    "since": "2026-05-15T14:00:00Z",
    "until": "2026-05-15T14:30:00Z"
  }'
```

Ou rodar cron manualmente:
```bash
# SSH Railway
railway ssh
# Dentro do container:
./bin/dlq-replay --provider=stripe --since="1h ago"
```

Monitor via Grafana `webhook.<provider>.reprocessed_count`.

---

## Endpoints admin

### Reprocessar evento único
```
POST /admin/webhooks/reprocess
Body: { "provider": "stripe", "event_id": "evt_XXX" }
Auth: Bearer admin token
```

### Reprocessar batch
```
POST /admin/webhooks/reprocess-batch
Body: { 
  "provider": "stripe",
  "since": "ISO8601",
  "until": "ISO8601"
}
```

### Ver DLQ
```
GET /admin/webhooks/dlq?provider=stripe&processed=false
```

### Resync Mux video
```
POST /admin/videos/:id/resync
```

---

## Segurança do admin

- `Authorization: Bearer <token>` — token ABAC role `admin`
- Endpoints **não expostos** em web/mobile UI (só API admin)
- Audit trail obrigatório (toda chamada admin registrada)
- Rate limit agressivo (10 req/min)

---

## Checklist pós-reprocessamento

- [ ] DLQ drenando (`SELECT COUNT(*) FROM webhook_dlq WHERE processed = false` reduziu)
- [ ] Métricas Grafana: `webhook.<provider>.success_rate` voltando
- [ ] Clientes afetados: subscription/video/asset em status esperado
- [ ] Audit trail tem entries das reprocessamentos
- [ ] Postmortem se outage > 30min afetou mais de 10 clientes

---

## Prevenção

- Idempotency obrigatória (`event.id` unique no ledger) — OK
- Retry com backoff (provider cuida) + DLQ (nosso) — OK
- Alertas Grafana em fail_rate — OK
- Pull periódico pra estados críticos (assembleia live) — pendente T-###

---

## Links

- [[_moc]]
- [[rollback-deploy]]
- [[incident-lgpd-breach]]
- [[../../06-execution/incident]]
- [[../../90-ingested/_consolidado-providers-fluxos]] — catálogo 27 webhooks
- Stripe webhooks: https://stripe.com/docs/webhooks
- Mux webhooks: https://docs.mux.com/guides/system/listen-for-webhooks
- Livekit webhooks: https://docs.livekit.io/realtime/server/webhooks/
