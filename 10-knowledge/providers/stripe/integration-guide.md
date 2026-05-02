---
title: Stripe — Integration Guide
type: guide
tags: [provider, stripe, integration, setup, webhooks]
source: https://docs.stripe.com
source_date: 2026-04-24
created: 2026-04-24
updated: 2026-04-24
aliases: [Stripe Setup, Stripe Integration]
---

# Stripe — Integration Guide

Setup passo-a-passo de test → produção. Complementa [[_moc]] e [[overview]]. Padrões atemporais em [[patterns]], erros a evitar em [[antipatterns]].

> **Fonte oficial**: https://docs.stripe.com/development — consultada em 2026-04-24.

---

## 1. Conta + chaves

1. Criar conta em https://dashboard.stripe.com/register
2. No dashboard, toggle **Test mode** (canto superior direito) fica ativo por default
3. Em `Developers → API keys` aparecem 2 pares de chaves:
   - **Test**: `pk_test_...` (publishable, expõe no front) + `sk_test_...` (secret, só server)
   - **Live**: `pk_live_...` + `sk_live_...` (só após ativar a conta: KYC, bank account)
4. Toggle test/live **nunca mistura dados** — objetos criados em test não existem em live e vice-versa.

**Regra**: nunca commitar `sk_*` nem `whsec_*`. Vault/KMS em produção, `.env.local` git-ignorado em dev.

---

## 2. Variáveis de ambiente canônicas

```env
# Server-side
STRIPE_SECRET_KEY=sk_test_...         # ou sk_live_... em prod
STRIPE_WEBHOOK_SECRET=whsec_...       # um por endpoint registrado
STRIPE_API_VERSION=2024-06-20         # pinned — evita breaking change auto

# Client-side (pode expor)
STRIPE_PUBLISHABLE_KEY=pk_test_...    # ou pk_live_...

# Connect (se aplicável)
STRIPE_CONNECT_CLIENT_ID=ca_...       # OAuth onboarding Standard
```

Pinar `STRIPE_API_VERSION` é **obrigatório** — sem isso, atualizações quebram seu handler sem aviso. Ver [[antipatterns]].

---

## 3. SDK install por stack

### Go

```bash
go get github.com/stripe/stripe-go/v78
```

```go
import "github.com/stripe/stripe-go/v78"
stripe.Key = os.Getenv("STRIPE_SECRET_KEY")
```

### Node/TypeScript

```bash
npm install stripe
```

```ts
import Stripe from "stripe";
const stripe = new Stripe(process.env.STRIPE_SECRET_KEY!, {
  apiVersion: "2024-06-20",
});
```

### Flutter (mobile)

```yaml
dependencies:
  flutter_stripe: ^11.0.0
```

```dart
Stripe.publishableKey = const String.fromEnvironment('STRIPE_PUBLISHABLE_KEY');
await Stripe.instance.applySettings();
```

Flutter usa **PaymentSheet** (UI nativa Stripe) — PAN nunca toca seu backend. Ver [[../../security/_moc]] → PCI scope.

---

## 4. Primeira chamada: Customer + PaymentIntent

Fluxo mínimo pra cobrar US$ 10,00 de um usuário:

### 4.1 Criar Customer (uma vez por User)

```go
params := &stripe.CustomerParams{
    Email: stripe.String(user.Email),
    Metadata: map[string]string{
        "user_id": user.ID.String(), // linkagem reversa
    },
}
cust, err := customer.New(params)
// persistir cust.ID no seu User (ex: users.stripe_customer_id)
```

**Nunca** criar Customer a cada login — é 1:1 com User, armazena no DB (ver [[antipatterns]]).

### 4.2 Criar PaymentIntent

```go
piParams := &stripe.PaymentIntentParams{
    Amount:   stripe.Int64(1000),       // centavos
    Currency: stripe.String("usd"),
    Customer: stripe.String(cust.ID),
    AutomaticPaymentMethods: &stripe.PaymentIntentAutomaticPaymentMethodsParams{
        Enabled: stripe.Bool(true),
    },
}
piParams.SetIdempotencyKey(requestID) // retry-safe
pi, err := paymentintent.New(piParams)
// devolve pi.ClientSecret para o front confirmar (Stripe.js / PaymentSheet)
```

Front recebe `client_secret` e chama `stripe.confirmPayment()` ou `PaymentSheet`. Padrão 3DS: ver [[patterns]] → fluxo `requires_action`.

---

## 5. Webhook setup

### 5.1 Endpoint HTTP (server)

Endpoint deve:

1. Ler **body bruto** (não parseado) — Stripe assina bytes, não JSON deserializado
2. Validar `Stripe-Signature` via SDK
3. Idempotência no handler (ver [[patterns]])
4. Devolver `2xx` rápido (< 5s); processar async se demorado

```go
func StripeWebhookHandler(w http.ResponseWriter, r *http.Request) {
    payload, _ := io.ReadAll(r.Body)
    sig := r.Header.Get("Stripe-Signature")

    event, err := webhook.ConstructEvent(payload, sig, os.Getenv("STRIPE_WEBHOOK_SECRET"))
    if err != nil {
        http.Error(w, "bad signature", http.StatusBadRequest)
        return
    }

    // enfileirar evento.ID + evento.Type + payload pra worker async
    if err := eventQueue.Enqueue(event); err != nil {
        http.Error(w, "queue error", http.StatusInternalServerError)
        return
    }
    w.WriteHeader(http.StatusOK)
}
```

### 5.2 Registrar endpoint no Stripe

Dashboard → `Developers → Webhooks → Add endpoint`:

- URL: `https://api.seudominio.com/webhooks/stripe`
- Eventos a escutar (subscribe apenas o necessário — não `*`):
  - `customer.subscription.created|updated|deleted`
  - `invoice.paid|payment_failed|finalized`
  - `payment_intent.succeeded|payment_failed`
  - `charge.dispute.created` (se usa disputes)
  - `account.updated` (Connect)
- API version: pinar igual ao `STRIPE_API_VERSION` do server
- Copiar `whsec_...` exibido → `STRIPE_WEBHOOK_SECRET`

---

## 6. Teste local com Stripe CLI

Stripe CLI faz tunnel dos webhooks de test pra `localhost` sem ngrok:

```bash
# Instalar (macOS)
brew install stripe/stripe-cli/stripe

# Login (abre browser, autoriza conta test)
stripe login

# Forward eventos pro endpoint local
stripe listen --forward-to localhost:8080/webhooks/stripe

# Disparar evento específico manualmente
stripe trigger payment_intent.succeeded
stripe trigger invoice.paid
stripe trigger customer.subscription.updated
```

O comando `listen` imprime um `whsec_...` temporário — usar nas env vars locais **em vez do** `whsec` da dashboard. Evita poluição do log de entrega remoto.

---

## 7. Go-live checklist

Antes de flipar `pk_live` / `sk_live`:

- [ ] Conta Stripe ativada (KYC submetido + conta bancária verificada)
- [ ] Endpoint de webhook **HTTPS** (HTTP não aceito em live)
- [ ] `STRIPE_WEBHOOK_SECRET` de **produção** nas env vars (diferente do test)
- [ ] Endpoint registrado no dashboard **em modo Live** (toggle)
- [ ] `STRIPE_API_VERSION` pinado igual em server + endpoint dashboard
- [ ] Timeout do handler < 5s (enfileirar async se pesado)
- [ ] Idempotência no handler validada (replay teste: CLI `stripe events resend <id>`)
- [ ] Logs **sem PAN, CVV, full card number** (ver [[antipatterns]])
- [ ] Dashboard → `Radar rules` revisadas (fraude)
- [ ] `Tax` configurado se cobrança internacional (Stripe Tax ou alternativa)
- [ ] Webhooks de `charge.dispute.*` tratados (chargeback)
- [ ] Reconciliação periódica agendada (ver [[patterns]] → source-of-truth sync)
- [ ] Monitoramento: alerta se `webhook_events_failed > N / hora`
- [ ] Runbook de incident: endpoint fora → eventos se acumulam (retry 3 dias); procedimento de replay documentado

---

## Vizinhos

- [[_moc]] — índice do Stripe
- [[overview]] — arquitetura de objetos
- [[patterns]] — idempotência, reconciliação, rate limit
- [[antipatterns]] — erros comuns
- [[../../security/_moc]] — PCI DSS, webhook signing
- [[../../testing/_moc]] — estratégia de teste com stubs/fakes
