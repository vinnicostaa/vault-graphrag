---
title: Stripe — Antipatterns
type: antipattern
tags: [provider, stripe, antipatterns, pitfalls]
source: https://docs.stripe.com
source_date: 2026-04-24
created: 2026-04-24
updated: 2026-04-24
aliases: [Stripe Antipatterns, Stripe Pitfalls]
---

# Stripe — Antipatterns

Erros comuns de integração Stripe e a correção idiomática. Contrapartida positiva em [[patterns]]. Vizinhos: [[_moc]], [[../_moc]], [[../../antipatterns/_moc]], [[../../security/_moc]].

> **Fonte oficial**: https://docs.stripe.com — consultada em 2026-04-24.

---

## ❌ AP-1 · Criar Customer a cada login

**Sintoma**: código chama `customer.New(...)` em todo fluxo de billing sem olhar o DB.

**Consequência**: N Customers fantasma no Stripe por User, Subscriptions associadas à conta errada, dashboard poluído, lookup reverso impossível.

**Correção**: Customer é **1:1** com User. Criar uma vez, persistir `stripe_customer_id` no DB. Recuperar via query antes de criar. Ver [[patterns]] → pattern 4.

```go
// ❌ errado
cust, _ := customer.New(&stripe.CustomerParams{Email: &user.Email})

// ✅ certo
if user.StripeCustomerID == "" {
    cust, _ := customer.New(...)
    user.StripeCustomerID = cust.ID
    db.Update(user)
}
```

---

## ❌ AP-2 · Comparar `subscription.status` por string sem exaustividade

**Sintoma**:

```go
if sub.Status == "active" {
    grantAccess()
} // e o resto? trialing? past_due?
```

**Consequência**: usuário em `trialing` perde acesso; usuário em `past_due` recebe bloqueio imediato quando política é grace period. Stripe adiciona status novo → silenciosamente cai no `else`.

**Correção**: `switch` exaustivo sobre `SubscriptionStatus` com `default` que loga "status desconhecido" e alerta. Os 8 estados estão em [[patterns]] → pattern 6.

```go
switch sub.Status {
case stripe.SubscriptionStatusTrialing, stripe.SubscriptionStatusActive:
    grantAccess()
case stripe.SubscriptionStatusPastDue:
    grantAccessWithGrace()
case stripe.SubscriptionStatusUnpaid, stripe.SubscriptionStatusIncomplete,
     stripe.SubscriptionStatusIncompleteExpired, stripe.SubscriptionStatusCanceled,
     stripe.SubscriptionStatusPaused:
    revokeAccess()
default:
    log.Error("unknown subscription status", "status", sub.Status)
    alert()
}
```

---

## ❌ AP-3 · Confiar no payload do webhook sem verificar signing

**Sintoma**: handler parseia JSON do body e atua (`grantAccess`, `creditBalance`) sem validar `Stripe-Signature`.

**Consequência**: endpoint público aceita POST falso de qualquer origem. Atacante fabrica `invoice.paid` e libera plano premium sem pagar. **CVSS alto**.

**Correção**: usar `webhook.ConstructEvent(rawBody, signature, secret)` do SDK **antes** de qualquer ação. Ler body bruto (bytes), não JSON decoded — Stripe assina bytes. Ver [[patterns]] → pattern 2, [[../../security/_moc]] → webhook signing.

---

## ❌ AP-4 · Armazenar PAN (Primary Account Number) no seu DB

**Sintoma**: tabela `card_details (number, cvv, holder)` ou similar. Log com `card.number` redigido "por conveniência".

**Consequência**: violação **PCI DSS** (níveis SAQ A → SAQ D; custos/auditoria explodem). Breach → multa massiva + notificação compulsória. Stripe nem expõe PAN após tokenização — se você tem, é porque burlou.

**Correção**: nunca tocar PAN no backend. Usar Stripe.js / Elements / PaymentSheet no cliente → PaymentMethod token volta ao servidor (`pm_...`). Armazenar **só o token** + last4/brand pra UI.

```
front (Stripe.js) ──PAN──► Stripe ──token pm_xxx──► seu server ──salva pm_xxx──► DB
```

Ver [[../../security/_moc]] → PCI scope minimization.

---

## ❌ AP-5 · Usar `charge.id` em vez de `payment_intent.id`

**Sintoma**: código referencia `ch_...` (Charge) como identificador primário de transação.

**Consequência**: Charge é **API legada**. Não suporta 3DS/SCA corretamente, não cobre fluxo `requires_action`, vai ser descontinuado. Refunds/disputes via Charge não batem com relatórios modernos.

**Correção**: PaymentIntent (`pi_...`) é o modelo atual. Sempre criar via `paymentintent.New`. Charge existe como **filho** do PaymentIntent (auto-gerado) — usar só pra debug / inspection, nunca como chave primária no seu DB.

```go
// ❌ errado
txn.StripeChargeID = event.Data.Object["id"] // "ch_..."

// ✅ certo
txn.StripePaymentIntentID = pi.ID // "pi_..."
```

---

## ❌ AP-6 · Skip idempotency key em retry

**Sintoma**: retry loop sem `Idempotency-Key`, ou chave é `uuid.NewV4()` gerado dentro do retry (diferente a cada tentativa).

**Consequência**: timeout no 1o request + sucesso no 2o → **duplo-charge**. Cliente vê 2 cobranças, suporte é acionado, chargeback vira realidade.

**Correção**: Idempotency-Key **determinística por intenção** — `order_id`, `command_id`, `subscription_change_id`. Estável entre retries. Stripe devolve mesmo resultado em 24h pra mesma chave.

```go
// ❌ errado
for i := 0; i < 3; i++ {
    p := &stripe.PaymentIntentParams{...}
    p.SetIdempotencyKey(uuid.New().String()) // muda a cada retry!
    pi, err := paymentintent.New(p)
}

// ✅ certo
key := orderID.String() // estável
p := &stripe.PaymentIntentParams{...}
p.SetIdempotencyKey(key)
// SDK oficial já retenta internamente com mesma key
pi, err := paymentintent.New(p)
```

Ver [[patterns]] → pattern 1.

---

## ❌ AP-7 · Hardcode `price_id` / `product_id` no código

**Sintoma**:

```go
const PlanPremiumPriceID = "price_1NxYz2AbCdEf"
```

**Consequência**: mudança de plano (ajuste de valor, nova moeda, novo trial) exige **deploy** de código. Ambiente test vs live têm IDs diferentes — if-else por env vira spaghetti. Refactor perde histórico.

**Correção**: `price_id` vive em **config/DB**, indexado por chave de negócio (`plan_slug`, `tier`). Seed por env. UI admin pra swap sem deploy.

```sql
CREATE TABLE billing_plans (
  slug text PRIMARY KEY,
  stripe_price_id text NOT NULL,
  display_name text NOT NULL,
  active boolean NOT NULL DEFAULT true
);
```

Código consulta `billing_plans WHERE slug = 'premium_monthly'` → pega `stripe_price_id` dinamicamente.

---

## ❌ AP-8 · Logar dados de cartão (mesmo tentando)

**Sintoma**: `log.Info("payment method", "pm", paymentMethod)` em dev — struct pode conter `card.last4`, e alguns devs tentam logar `card.number` "só em debug".

**Consequência**: Stripe nunca expõe PAN completo após tokenização, mas **tentar** logar estrutura completa cai em violação PCI. Logs centralizados (Datadog, CloudWatch) viram out-of-scope pra auditoria. Breach nos logs → incidente de dados.

**Correção**:

- **Nunca** logar objeto PaymentMethod / Card completo
- Redact list explícita no logger: `card.number`, `card.cvc`, `card.exp_*`, `bank_account.*`
- Logar só: `pm.ID`, `card.Last4`, `card.Brand`, `card.ExpMonth/Year` (esses podem)
- Em dev: mesma regra — hábito é tudo

```go
// ❌ errado
log.Info("pm", "raw", pm) // pode ter campos sensíveis se Stripe mudar schema

// ✅ certo
log.Info("pm received", "id", pm.ID, "brand", pm.Card.Brand, "last4", pm.Card.Last4)
```

Ver [[../../security/_moc]] → PII / secrets in logs.

---

## Vizinhos

- [[_moc]] — índice do Stripe
- [[patterns]] — contraparte positiva
- [[overview]] — arquitetura
- [[integration-guide]] — setup
- [[../_moc]] — providers raiz
- [[../../antipatterns/_moc]] — catálogo AP-### geral
- [[../../security/_moc]] — PCI, signing, logs
