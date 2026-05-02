---
title: Stripe — Patterns
type: pattern
tags: [provider, stripe, patterns, idempotency, webhooks, reconciliation]
source: https://docs.stripe.com
source_date: 2026-04-24
created: 2026-04-24
updated: 2026-04-24
aliases: [Stripe Patterns]
---

# Stripe — Patterns

Padrões atemporais e cross-projeto pra integrações Stripe. Contraparte em [[antipatterns]]; contexto em [[_moc]] e [[overview]].

> **Fonte oficial**: https://docs.stripe.com — consultada em 2026-04-24. Vizinhos: [[../_moc]], [[../../patterns/_moc]], [[../../security/_moc]].

---

## 1. Idempotency-Key em toda mutação

**Problema**: retry em timeout pode duplicar cobrança (double-charge) ou criar Customer duplicado.

**Padrão**: todo request `POST` / `DELETE` envia header `Idempotency-Key: <uuid>` estável por intenção de negócio. Stripe retorna **mesmo resultado** se chave repetir dentro de 24h.

```go
params := &stripe.PaymentIntentParams{ /* ... */ }
params.SetIdempotencyKey(orderID.String()) // NÃO uuid-v4 gerado no retry
pi, err := paymentintent.New(params)
```

**Chave**: algo determinístico do domínio (`order_id`, `subscription_change_id`, `command_id`), **não** `uuid.NewV4()` do retry loop. Ver [[../../patterns/_moc]] → idempotency pattern.

**Escopo**: mutações. `GET` é naturalmente idempotente.

---

## 2. Webhook signing + replay protection

**Problema**: endpoint público aceita POST de qualquer origem → atacante injeta evento falso.

**Padrão**: validar `Stripe-Signature` com **body bruto** (não parsed) e janela de tolerância:

```go
const tolerance = 5 * time.Minute
event, err := webhook.ConstructEventWithOptions(
    payloadBytes,
    r.Header.Get("Stripe-Signature"),
    webhookSecret,
    webhook.ConstructEventOptions{Tolerance: tolerance},
)
```

Validações implícitas no SDK:

- HMAC SHA256 do `timestamp.payload` bate com `v1=...`
- `timestamp` dentro da janela (default 5 min) — rejeita replay antigo

**Camada extra** (defense-in-depth): persistir `event.ID` em tabela `stripe_events_processed (event_id PK, processed_at)` e rejeitar duplicata. Protege contra replay **dentro** da janela.

```sql
INSERT INTO stripe_events_processed (event_id) VALUES ($1)
ON CONFLICT (event_id) DO NOTHING
RETURNING event_id;
-- se não retornou linha → já processado, skip
```

---

## 3. Reconciliação periódica (Stripe = source of truth)

**Problema**: webhook pode falhar (endpoint down, bug, fila travada). DB local diverge do Stripe silenciosamente.

**Padrão**: job periódico (hourly / daily) que consulta Stripe e reconcilia estado:

```
Pra cada Subscription ativa em DB local:
  stripe.Subscription.Retrieve(sub.stripe_id)
  compara status DB vs Stripe
  se diverge: atualiza DB com Stripe (truth) + emite alerta
  se Stripe 404: marca como órfão, investigar
```

**Quando**: Subscriptions (status, current_period_end), Invoices (paid/unpaid), Connect accounts (`charges_enabled`, `payouts_enabled`).

**Regra dura**: Stripe é **source of truth**. DB local cacheia. Divergência → atualizar local, nunca "corrigir" Stripe baseado em local.

---

## 4. Customer 1:1 com User via metadata

**Padrão**: cada `User` tem no máximo **um** Stripe Customer. Lookup bidirecional:

- **User → Customer**: coluna `users.stripe_customer_id` (persistida ao criar)
- **Customer → User**: `Customer.metadata.user_id` (gravado na criação)

```go
cust, _ := customer.New(&stripe.CustomerParams{
    Email: stripe.String(user.Email),
    Metadata: map[string]string{"user_id": user.ID.String()},
})
db.Exec("UPDATE users SET stripe_customer_id = $1 WHERE id = $2", cust.ID, user.ID)
```

**Recuperação em caso de perda** (DB corrompido): `Customer.Search` por `metadata['user_id']:'<id>'`.

---

## 5. PaymentIntent + fluxo 3DS (SCA)

**Problema**: cartão europeu / BR moderno exige 3D Secure (PSD2/SCA). Fluxo não é atômico.

**Padrão**: state machine no front + server:

```
[server] create PaymentIntent(automatic_payment_methods.enabled=true)
         → status: requires_payment_method
         → devolve client_secret

[client] stripe.confirmPayment({clientSecret, ...})
         se cartão sem 3DS: status → succeeded
         se cartão com 3DS: status → requires_action
                             → Stripe.js abre modal / redirect
                             → usuário autentica
                             → status → succeeded | requires_payment_method

[server webhook] payment_intent.succeeded → liberar valor ao domínio
                 payment_intent.payment_failed → notificar user, oferecer retry
```

**Nunca** liberar produto/serviço antes do webhook `payment_intent.succeeded` — status no front pode ser mentira (DevTools override).

---

## 6. Subscription lifecycle — 8 estados

Status possíveis (`subscription.status`):

| Status | Significado | Ação típica |
|---|---|---|
| `trialing` | Em trial, não cobra ainda | Liberar acesso; agendar notificação fim-de-trial |
| `active` | Pagando normalmente | Liberar acesso |
| `past_due` | Última invoice falhou, retry agendado | Avisar user; manter acesso (configurável via `cancel_at_period_end`) |
| `unpaid` | Retries esgotados, não cancelou ainda | Bloquear features premium; cobrar update de PaymentMethod |
| `canceled` | Encerrada | Revogar acesso no `current_period_end` (ou imediato se `cancel_at=now`) |
| `incomplete` | Primeira invoice não paga em 23h | Bloqueio imediato |
| `incomplete_expired` | `incomplete` expirou | Limpar do DB local |
| `paused` | Pausada (ex: collection paused) | Comportamento configurável |

**Padrão**: use-case dedicado `ApplySubscriptionStatus(sub)` com `switch` exaustivo — compilador avisa se Stripe adicionar status novo (ver [[antipatterns]] → comparar por string).

---

## 7. Cache ABAC invalidation em webhook

**Problema**: decisão de autorização (ex: "user pode criar evento?") depende de plano ativo. Cache de decisão ABAC dura minutos; plano muda em tempo real via Stripe.

**Padrão**: webhook handler invalida cache ABAC nos eventos que afetam capability:

```
customer.subscription.created   → invalidar cache(user_id)
customer.subscription.updated   → invalidar cache(user_id)  # inclui cancel agendado, upgrade, downgrade
customer.subscription.deleted   → invalidar cache(user_id)
invoice.paid                    → invalidar cache(user_id)  # sai de past_due → active
invoice.payment_failed          → invalidar cache(user_id)  # entra em past_due
customer.updated                → invalidar se metadata mudou
```

`user_id` recuperado via `Customer.metadata.user_id` (ver pattern 4). Ver [[../../security/_moc]] pra ABAC em geral.

---

## 8. Rate limit 429 → exponential backoff

**Limite Stripe**: ~100 read / 100 write por segundo em live; menor em test. Response `429 Too Many Requests` com header `Retry-After`.

**Padrão**: SDKs oficiais já retentam automaticamente com backoff. **Não desabilite**. Se tiver wrapper próprio, seguir:

- Backoff exponencial: 500ms, 1s, 2s, 4s, 8s (jitter ±20%)
- Respeitar `Retry-After` se presente
- Max 5 retries; depois falhar pra circuit breaker
- Logar `request_id` da response (header `Request-Id`) — Stripe support usa ele

Se você faz **bulk operations** (import inicial, migração), preferir `Search` + paginação em vez de loop de `Retrieve` individual. Se inevitável, throttle explícito < 80 req/s.

---

## 9. API version pinning

`STRIPE_API_VERSION=2024-06-20` no server + **mesma versão** no endpoint de webhook (dashboard). Stripe muda formato de payload entre versões. Pinar evita breaking change sem deploy.

Upgrade planejado: ler changelog (https://docs.stripe.com/upgrades), testar em staging com nova versão, só então subir em prod.

---

## Vizinhos

- [[_moc]] — índice do Stripe
- [[antipatterns]] — o que não fazer
- [[integration-guide]] — setup
- [[../_moc]] — providers raiz
- [[../../patterns/_moc]] — patterns cross-provider (idempotency, saga, retry)
- [[../../security/_moc]] — ABAC, PCI, webhook signing
