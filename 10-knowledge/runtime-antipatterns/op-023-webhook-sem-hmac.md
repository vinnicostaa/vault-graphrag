---
title: OP-023 — Webhook sem validação de assinatura HMAC
type: antipattern
tags:
  - antipattern
  - runtime
  - operational
  - security
  - webhook
  - hmac
id: OP-023
severity: Critical
created: 2026-04-24
updated: 2026-04-24
doc-consulted: 2026-04-24
aliases:
  - Unverified webhook
  - HMAC missing
---

# OP-023 — Webhook sem validação de assinatura HMAC

**Severidade**: Critical
**Categoria**: Segurança

## Sintoma

Handler `/webhooks/*` aceita qualquer POST como legítimo. Atacante que descobre a URL pode **forjar eventos** — disparar transações financeiras, ativar assinaturas, publicar mensagens, deletar recursos.

## Por que é problema

- **Spoofing trivial**: URL pública + body JSON = ataque de 1 linha em `curl`.
- **Estado corrompido**: eventos forjados alteram banco de dados.
- **Perda financeira direta**: webhook Stripe forjado pode confirmar pagamento inexistente.

## Exemplo (anti-pattern)

```go
// ❌ ERRADO — confia no body cru
func (h *StripeWebhookHandler) Handle(ctx context.Context, body []byte) error {
    var event stripe.Event
    json.Unmarshal(body, &event)
    return h.processor.Process(ctx, event)
}
```

## Fix preferido — verificar HMAC **antes** de parsear

```go
func (h *StripeWebhookHandler) Handle(c gin.Context) {
    // 1. Body CRU — middleware não pode ter consumido via bind
    body, err := io.ReadAll(c.Request.Body)
    if err != nil {
        c.JSON(400, gin.H{"error": "invalid body"})
        return
    }

    // 2. Verificação PRIMEIRO
    sig := c.GetHeader("Stripe-Signature")
    event, err := webhook.ConstructEvent(body, sig, h.webhookSecret)
    if err != nil {
        // Log server-side com IP; resposta genérica pro cliente
        h.logger.Warn("invalid webhook signature",
            "ip", c.ClientIP(), "err", err)
        c.JSON(400, gin.H{"error": "invalid signature"})
        return
    }

    // 3. Só agora processa — idempotente (OP-003)
    if err := h.processor.Process(c.Request.Context(), event); err != nil {
        c.JSON(500, gin.H{"error": "processing failed"})
        return
    }
    c.JSON(200, gin.H{"received": true})
}
```

## Regras obrigatórias

- **Raw body necessário**: frameworks HTTP tipicamente consomem body no bind; configurar middleware `RawBody` na rota do webhook.
- **HMAC-SHA256 com validação de timestamp** (janela 5min) — previne replay.
- **Timing-safe comparison** (`hmac.Equal` em Go, `crypto.timingSafeEqual` em Node) — previne timing attack.
- **Never** retornar detalhes do erro de assinatura ao cliente; log server-side.

## Provedores que suportam nativamente

| Provider | Header | Algoritmo | Lib oficial |
|---|---|---|---|
| Stripe | `Stripe-Signature` | HMAC-SHA256 + timestamp | `stripe.Webhook.constructEvent` |
| GitHub | `X-Hub-Signature-256` | HMAC-SHA256 | manual ou octokit |
| Shopify | `X-Shopify-Hmac-SHA256` | HMAC-SHA256 | manual |
| PayPal | `Paypal-Auth-Algo` + verification request | async RSA | PayPal SDK |
| Twilio | `X-Twilio-Signature` | HMAC-SHA1 | Twilio SDK |

## Alternativa

- **Mutual TLS** (mTLS) em integrações B2B com provider confiável.
- **Webhook secret rotacionado** periodicamente (zero downtime via 2-secret window).
- **Whitelist de IP de origem** como camada extra — **nunca** como única defesa; IPs de provider mudam.

## Quando tolerar

Nunca. Mesmo endpoint interno que recebe webhook de sistema interno: assinar com HMAC + secret compartilhado (rotacionável).

## Relações

- **OPs relacionados**: [[op-003-webhook-sem-idempotencia]] (par canônico — assinar + idempotência), [[op-024-rate-limit-ausente]]
- **Patterns**: [[../security/security-principles]], [[../providers/stripe/_moc]]

## Fontes

- [[../../40-templates/AGENTS_SPEC]] §4 OP-023, §3.20
- Stripe — [Webhook signatures](https://docs.stripe.com/webhooks/signatures) (consultada 2026-04-24)
- GitHub — [Webhook signatures](https://docs.github.com/en/webhooks/using-webhooks/validating-webhook-deliveries) (consultada 2026-04-24)
- OWASP — [Webhook security](https://owasp.org/www-project-api-security/) (consultada 2026-04-24)

## Links

- [[_moc]]
- [[../security/security-principles]]
- [[op-003-webhook-sem-idempotencia]]
- [[../providers/stripe/_moc]]
