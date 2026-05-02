---
title: OP-003 — Webhook sem idempotência
type: antipattern
tags:
  - antipattern
  - runtime
  - operational
  - idempotency
  - webhook
id: OP-003
severity: Critical
created: 2026-04-24T00:00:00.000Z
updated: 2026-04-24T00:00:00.000Z
aliases:
  - Webhook replay duplicado
  - Non-idempotent webhook
doc-consulted: '2026-04-24'
---

# OP-003 — Webhook sem idempotência

**Severidade**: Critical
**Categoria**: Idempotência

## Sintoma

Handler de webhook processa `event.id` sem checar se já foi processado. Provider (Stripe, Mux, GitHub) reenvia o mesmo evento (timeout, retry policy) → transação financeira executa 2×, assinatura duplica, email sai 3 vezes.

## Por que é problema

- **Financeiro**: cobrança duplicada tem custo direto (chargeback + reputação).
- **Compliance**: eventos de domínio duplicados corrompem audit trail e projeções.
- **Provider é retentador por contrato**: Stripe retenta 3 dias, GitHub 30 vezes, Mux até 24h. Assumir "só vem uma vez" é incorreto.

## Exemplo (anti-pattern)

```ts
// ❌ ERRADO — mesmo evento processado N vezes
app.post('/webhooks/stripe', async (req, res) => {
  const event = stripe.webhooks.constructEvent(...)
  await processSubscription(event)   // ← sem checar duplicação
  res.sendStatus(200)
})
```

## Fix preferido — tabela de idempotência

```sql
CREATE TABLE webhook_events_processed (
  event_id     TEXT PRIMARY KEY,
  source       TEXT NOT NULL,   -- 'stripe', 'mux', 'github'
  status       TEXT NOT NULL,   -- 'processed' | 'failed'
  processed_at TIMESTAMPTZ NOT NULL DEFAULT now(),
  payload_hash TEXT NOT NULL    -- detecta replay com payload alterado
);
```

```ts
async function handleWebhook(event) {
  const existing = await repo.findProcessedEvent(event.id)
  if (existing?.status === 'processed') return 200   // idempotent replay OK
  try {
    await processBusinessLogic(event)
    await repo.markProcessed(event.id, 'processed', hashPayload(event))
    return 200
  } catch (err) {
    if (isRetryableError(err)) throw err             // provider reenvia
    await repo.markProcessed(event.id, 'failed', ...)
    return 200  // não-retentável — confirma pra provider parar
  }
}
```

## Complemento essencial

Diferenciar **erros retentáveis** (timeout, DB down → rethrow → provider retenta) de **não-retentáveis** (validação, regra de negócio quebrada → marcar `failed` + retornar 200 pra parar retry).

## Alternativa

- Cache Redis com TTL curto quando volume é altíssimo e tabela é gargalo — `SETNX event:{id} processed EX 86400`.
- Deduplicação em queue (SQS FIFO `MessageDeduplicationId`).

## Quando tolerar

Nunca em operação financeira ou de estado de negócio. Aceitável em webhook puramente observacional (ex: notificação de deploy — processar 2× não causa dano) **desde que documentado**.

## Relações

- **Patterns**: [[../patterns/transaction-patterns#saga]] (compensação quando webhook cruza BC), [[../patterns/transaction-patterns]]
- **OPs relacionados**: [[op-023-webhook-sem-hmac]] (assinatura antes de processar), [[op-004-sem-retry-strategy]], [[op-022-log-com-pii]]

## Fontes

- [[../../40-templates/AGENTS_SPEC]] §4 OP-003
- Stripe — [Best practices: Handling webhook events](https://docs.stripe.com/webhooks/best-practices)
- GitHub — [Redelivering webhooks](https://docs.github.com/en/webhooks)
- AWS SQS FIFO — [Exactly-once processing](https://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/FIFO-queues-exactly-once-processing.html)

## Links

- [[_moc]]
- [[../antipatterns/_moc]]
- [[../security/security-principles]]
- [[../providers/stripe/_moc]]
