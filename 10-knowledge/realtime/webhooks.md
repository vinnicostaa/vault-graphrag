---
title: Webhooks — outbound event callbacks
type: concept
tags:
  - realtime
  - webhooks
  - integration
  - http
  - async
doc-consulted: 2026-04-24
created: 2026-04-24
updated: 2026-04-24
aliases:
  - Webhooks
  - HTTP callbacks
  - Event callbacks
---

# Webhooks — outbound event callbacks

**O que é**: padrão de integração em que um sistema **provider** envia request HTTP POST para URL registrada pelo **consumer** quando um evento de interesse ocorre. É "server-to-server async push" — o oposto lógico de polling e fundamentalmente diferente de [[websocket|WebSocket]] (que é cliente-servidor persistente).

> **Mental model**: "provider faz HTTP para você quando algo acontece". Requer URL pública + defesa de segurança (HMAC + idempotência) + tolerância a duplicatas/retries.

## Webhook vs WebSocket vs SSE

Ver [[_moc]] para decision tree completo. Resumo:

| Dimensão | Webhook | WebSocket | SSE |
|---|---|---|---|
| **Iniciativa** | Provider → Consumer push | Bidirectional peer | Server → Client push |
| **Persistência** | 1 request por evento | Conexão longa | Conexão longa |
| **Consumer precisa** | URL pública HTTPS | Browser/TCP client | Browser/SSE client |
| **Escala** | Stateless — escala trivialmente | Stateful — broker/pub-sub | Stateless — OK |
| **Casos típicos** | Server-to-server B2B | User-facing realtime | Live dashboards |

## Fluxo canônico

```
┌──────────┐                        ┌──────────┐
│ Consumer │                        │ Provider │
└──────────┘                        └──────────┘
     │                                    │
     │ 1. Register webhook URL            │
     │    + events of interest            │
     │ ──────────────────────────────────▶│
     │                                    │
     │        (tempo depois, evento acontece)
     │                                    │
     │ 2. POST https://consumer/wh        │
     │    Header: HMAC-signature          │
     │    Body: { event_id, type, data }  │
     │ ◀──────────────────────────────────│
     │                                    │
     │ 3. Validate signature              │
     │ 4. Check idempotency (event_id)    │
     │ 5. Process                         │
     │                                    │
     │ 6. HTTP 200 (or 500 for retry)     │
     │ ──────────────────────────────────▶│
     │                                    │
     │        se 500/timeout → retry com backoff
     │                                    │
```

## 4 pilares de webhook robusto

### 1. Assinatura HMAC (OBRIGATÓRIO — antes de parsear)

Webhook sem assinatura = endpoint público aceita qualquer POST. Atacante forja eventos (activa subscription, dispara pagamento inexistente, etc.). Ver [[../runtime-antipatterns/op-023-webhook-sem-hmac|OP-023]].

**Padrão canônico** (Stripe-style):
```
Header: X-Provider-Signature: t=<timestamp>,v1=<hmac-sha256-hex>
Signed: "<timestamp>.<raw-body>"
Key: shared webhook secret (per-endpoint)
```

```ts
import crypto from 'crypto'

function verify(rawBody: string, header: string, secret: string): boolean {
  const parts = Object.fromEntries(header.split(',').map(p => p.split('=')))
  const t = parts.t, v1 = parts.v1
  const signedPayload = `${t}.${rawBody}`
  const expected = crypto.createHmac('sha256', secret).update(signedPayload).digest('hex')

  // Timing-safe compare
  if (!crypto.timingSafeEqual(Buffer.from(v1), Buffer.from(expected))) return false

  // Timestamp window (anti-replay): ±5 minutes
  if (Math.abs(Date.now()/1000 - parseInt(t)) > 300) return false

  return true
}
```

**Regras operacionais**:
- **Raw body** necessário — frameworks consomem body no bind; configurar middleware `raw` na rota do webhook.
- **Timing-safe comparison** — `crypto.timingSafeEqual` ou equivalente, nunca `===`.
- **Timestamp window** (5min) — previne replay antigo.
- **Nunca retornar detalhe do erro de assinatura** ao cliente — só "400 invalid request" + log server-side com IP.

### 2. Idempotência (OBRIGATÓRIA — antes de lógica)

Provider retenta em timeout/erro 5xx; mesmo evento chega 2-10x. Ver [[../runtime-antipatterns/op-003-webhook-sem-idempotencia|OP-003]].

**Padrão canônico**: tabela `webhook_events_processed`:
```sql
CREATE TABLE webhook_events_processed (
  event_id     TEXT PRIMARY KEY,
  source       TEXT NOT NULL,    -- 'stripe', 'github', 'mux'
  status       TEXT NOT NULL,    -- 'processed' | 'failed'
  processed_at TIMESTAMPTZ NOT NULL DEFAULT now(),
  payload_hash TEXT NOT NULL     -- detecta replay com payload alterado
);
```

```ts
async function handleWebhook(event: Event) {
  const existing = await repo.findProcessed(event.id)
  if (existing?.status === 'processed') return 200  // idempotent — OK

  try {
    await processBusinessLogic(event)
    await repo.markProcessed(event.id, 'processed', hashPayload(event))
    return 200
  } catch (err) {
    if (isRetryableError(err)) throw err                  // provider retentará
    await repo.markProcessed(event.id, 'failed', hashPayload(event))
    return 200  // não-retentável → confirma pra parar retry
  }
}
```

### 3. Retry policy do provider (entendimento)

| Provider | Política típica |
|---|---|
| **Stripe** | 3 dias, exponential backoff (minutes → hours) |
| **GitHub** | 30 retries ao longo de 72h |
| **Mux** | 24h |
| **Shopify** | 48h, 19 tentativas |
| **PayPal** | 25 tentativas em 3 dias |
| **Twilio** | custom — configurável |

**Consumer deve**: responder 2xx rápido (<10s tipicamente) → provider considera OK. Responder 5xx ou timeout → retry. Responder 4xx (não-500) → provider **para** de tentar e marca failed.

### 4. Retry do consumer (processamento async)

Se webhook chegou OK (200) mas processamento síncrono é caro, empilhar em **queue**:

```ts
app.post('/webhooks/stripe', rawBody(), async (c) => {
  const rawBody = await c.req.raw.text()
  if (!verify(rawBody, c.req.header('Stripe-Signature')!, SECRET)) {
    return c.text('bad signature', 400)
  }

  const event = JSON.parse(rawBody)

  // Empilhar em queue; processar async
  await queue.enqueue('stripe-events', event)

  return c.json({ received: true })   // < 1s response
})
```

Consumer da queue faz idempotência check + processamento + retry com backoff + DLQ. Ver [[../providers/cloudflare/_moc|Cloudflare Queues (lazy — D-vault-020)]] ou AWS SQS.

## Event ID e delivery guarantees

- **`event.id`** — UUID único POR EVENTO. Stripe: `evt_1ABC...`. Não confundir com `object.id` (o recurso que mudou).
- **Ordem**: providers **não garantem** ordem de delivery. 2 events `charge.created` e `charge.updated` podem chegar invertidos. Consumer valida via `occurred_at` + idempotência.
- **At-least-once**: padrão; receber duplicatas é a regra, não exceção.
- **Exactly-once**: raríssimo; só com queue FIFO (SQS FIFO, Kafka) + idempotência no consumer.

## HTTP codes semântica

| Código | Significado | Provider faz |
|---|---|---|
| `200`-`299` | OK | Para de retentar |
| `400`-`409` | Não-retentável (validação, conflito) | Marca failed, para |
| `410` | Endpoint permanentemente removido | Desabilita webhook |
| `429` | Rate limit | Retry com backoff maior |
| `500`-`599` | Erro temporário | Retry exponencial |
| Timeout | Indeterminado | Retry |

**Regra**: retornar **200** para eventos que você **não processará** (feature desabilitada, user deletado). 4xx é "consumer quebrado"; 200 "processei (inclusive ignorando)".

## Webhook registration

### Manual (dashboard)

Consumer cria URL + secret no painel do provider. Simples; não escala para multi-tenant SaaS.

### Programático (API)

```ts
// Stripe SDK
await stripe.webhookEndpoints.create({
  url: 'https://app.example.com/webhooks/stripe',
  enabled_events: ['invoice.payment_succeeded', 'customer.subscription.updated'],
  api_version: '2024-10-28',   // pin para evitar breaking changes
})
```

### Dynamic (per-tenant)

SaaS multi-tenant: cada cliente do SaaS tem webhook próprio no provider (ex: cada tenant conecta seu próprio Stripe). Webhook URL típica:
```
https://app.example.com/webhooks/stripe/<tenant-id>
```

Route extrai tenant, valida HMAC **com o secret daquele tenant**, processa.

## Exposing URL — requisitos

- **HTTPS** (valid cert; não self-signed).
- **Publicamente roteável** — IP público ou hostname DNS.
- **Timeout generoso** — alguns providers esperam 10-30s para response; não bloquear.
- **Firewall allow** do range de IPs do provider (alguns publicam; ver [IP egress Stripe/GitHub/etc.]).

### Ambiente local (dev)

Providers não chegam em `localhost`. Use:
- **ngrok** — tunnel público com URL `https://abcd.ngrok.io`.
- **Cloudflare Tunnel** (ver [[../providers/cloudflare/tunnel]]) — grátis + integrado a Access.
- **Tailscale Funnel** — alternativa P2P.
- **Stripe CLI** — `stripe listen --forward-to localhost:3000/webhooks/stripe` (só Stripe).

## Security checklist

- [ ] HMAC signature verification **antes** de parse body.
- [ ] Timing-safe comparison de signatures.
- [ ] Timestamp window (±5min).
- [ ] Idempotency via `event_id` table.
- [ ] Rate limit — webhooks tem burst legítimo mas atacante pode flood. Ver [[../runtime-antipatterns/op-024-rate-limit-ausente|OP-024]] (key por IP + assinatura).
- [ ] Log de eventos + payload hash (não o payload cru — PII).
- [ ] Secret rotation periódica (90d).
- [ ] Secret DIFERENTE por endpoint/tenant.
- [ ] IP allowlist do provider quando disponível (defesa extra).
- [ ] TLS 1.2+ no endpoint.
- [ ] Nunca responder 500 em erros de validação permanentes (força retry inútil).

## Anti-patterns

1. **Sem HMAC** ([[../runtime-antipatterns/op-023-webhook-sem-hmac|OP-023]]).
2. **Sem idempotência** ([[../runtime-antipatterns/op-003-webhook-sem-idempotencia|OP-003]]).
3. **Logar payload cru** ([[../runtime-antipatterns/op-022-log-com-pii|OP-022]]).
4. **Processar tudo síncrono em handler** — 30s timeout do provider; 10s recomendado. Empilhar em queue.
5. **Bind do body consumido antes da signature check** — middleware framework faz JSON.parse; HMAC falha. Config `raw` na rota.
6. **Webhook secret no código/env sem rotation** — vaza em breach.
7. **Responder 500 em validação** — retry eterno sem solução.
8. **Ignorar `api_version`** — provider atualiza schema; seu código quebra silenciosamente.

## Providers com webhook notável

| Provider | HMAC algo | Header | Retry | Versioning |
|---|---|---|---|---|
| **Stripe** | HMAC-SHA256 | `Stripe-Signature: t=...,v1=...` | 3d exp backoff | `api_version` no endpoint |
| **GitHub** | HMAC-SHA256 | `X-Hub-Signature-256: sha256=...` | 30×/72h | Versioning via Accept header |
| **Shopify** | HMAC-SHA256 | `X-Shopify-Hmac-SHA256` | 48h | Versioning semestral |
| **Twilio** | HMAC-SHA1 | `X-Twilio-Signature` | Configurável | — |
| **Slack** | HMAC-SHA256 | `X-Slack-Signature: v0=...` + `X-Slack-Request-Timestamp` | Sem retry (exigem 3s response) | — |
| **PayPal** | RSA async verification | `Paypal-Auth-Algo` + request | 25×/3d | — |

## Como grandes empresas usam

- **Stripe**: webhook é primary channel para events async (pagamentos, subscriptions). 3 dias de retry + CLI local + test mode separado.
- **GitHub Actions / Webhooks**: push/PR events → CI pipeline; Marketplace requer HMAC verified.
- **Shopify**: order fulfillment depende de webhook delivery; apps do Marketplace validados.
- **Mux**: asset encoding status via webhook; consumer deve tolerar 24h de processing.
- **Twilio**: SMS/call status callbacks em real-time.
- **Discord/Slack**: slash commands via HTTP POST com signature (semelhante a webhook).
- **Calendly / Typeform**: form submissions via webhook.

## Relações

- **Cluster realtime**: [[_moc]] decision tree
- **Alternativas**: [[websocket]] (bidirectional user-facing), [[server-sent-events|SSE]], polling
- **Segurança HMAC**: [[../security/security-principles#Webhook-Signature]]
- **Implementações reais**: [[../providers/stripe/_moc]] (webhook Stripe), [[../providers/mux/_moc]], [[../providers/cloudflare/_moc|Cloudflare Queues (lazy — D-vault-020)]] (queue consumer para async)
- **Runtime antipatterns**: [[../runtime-antipatterns/op-003-webhook-sem-idempotencia|OP-003]], [[../runtime-antipatterns/op-004-sem-retry-strategy|OP-004]], [[../runtime-antipatterns/op-022-log-com-pii|OP-022]], [[../runtime-antipatterns/op-023-webhook-sem-hmac|OP-023]], [[../runtime-antipatterns/op-024-rate-limit-ausente|OP-024]]

## Fontes

- [Stripe Webhooks best practices](https://docs.stripe.com/webhooks/best-practices) (consultada 2026-04-24)
- [GitHub Webhooks — validating deliveries](https://docs.github.com/en/webhooks/using-webhooks/validating-webhook-deliveries) (consultada 2026-04-24)
- [Shopify Webhooks verification](https://shopify.dev/docs/apps/build/webhooks/subscribe/https) (consultada 2026-04-24)
- [Slack verifying requests](https://api.slack.com/authentication/verifying-requests-from-slack) (consultada 2026-04-24)
- [OWASP Webhook Security](https://cheatsheetseries.owasp.org/cheatsheets/REST_Security_Cheat_Sheet.html) (consultada 2026-04-24)
- [websocket.org — WebSocket vs Webhook comparison](https://websocket.org/guides/websockets-vs-webhooks) (consultada 2026-04-24)

## Links

- [[_moc]]
- [[websocket]]
- [[server-sent-events]]
- [[../security/security-principles]]
- [[../providers/stripe/_moc]]
- [[../providers/mux/_moc]]
- [[../providers/cloudflare/_moc|Cloudflare Queues (lazy — D-vault-020)]] *(a destilar)*
