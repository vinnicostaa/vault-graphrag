---
title: ADR-0004 — Stripe via IPaymentGateway
type: adr
tags: [adr, billing, stripe, master-sindico, v2]
status: accepted
date: 2026-04-23
created: 2026-04-23
updated: 2026-04-23
---

# ADR-0004 — Stripe via IPaymentGateway

## Status (atualizado 2026-04-27)

⚠️ **Atualização D-139 (2026-04-27)**: Stripe é a **implementação inicial**, mas o domínio NÃO depende de Stripe diretamente. Refactor obrigatório (M1 stretch) para resolver vazamento de domínio: domínio depende de `IPaymentGateway` (port em `internal/domain/billing/`); Stripe vira `StripeGateway` em `internal/infrastructure/providers/stripe/` implementando a interface. Mercado Pago será adicionado como segundo adapter `MercadoPagoGateway` em M2+. Cliente decidiu: **agnosticizar, não substituir**. Ver [[../../STATE#D-139]].

### Status original

accepted — 2026-04-23 (herdado de legado)

## Contexto

Billing SaaS precisa: subscriptions recorrentes, trial persona-aware (15/7/30 dias), multi-currency (BRL M1, USD futuro), webhooks confiáveis, suporte cartão brasileiro.

## Decisão

**Stripe** como payment gateway, abstraído por port `IPaymentGateway` em `application/ports/`. SDK isolado em `infrastructure/providers/stripe/`.

### Detalhes

- Stripe Subscriptions para plano mensal.
- Trial aplicado server-side via `TrialCalculator` (persona → dias); Stripe recebe trial já calculado.
- Webhooks `/webhook/stripe` — HMAC verificado via `Stripe-Signature` **antes** de parse; idempotência via `stripe_event_id` em Redis 24h.
- Circuit breaker `sony/gobreaker` com threshold 50% erro em 20 req / 30s.
- Retry 3× exponencial base 200ms com jitter em writes idempotentes (usa `Idempotency-Key`).

## Consequências

### Positivas

- Stripe BR funciona bem (cartão de crédito; Pix beta em evolução).
- Checkout hosted reduz PCI scope (SAQ A).
- Dashboard admin maduro.
- Test mode + test cards facilita QA.

### Negativas

- Custo: 3.99% + R$0,39 por transação BRL (preço 2025). Ruim em margem baixa.
- Cartão BR only em M1 — Pix integrado só quando Stripe liberar oficialmente (avaliar 2026-Q3).
- Vendor lock parcial: port `IPaymentGateway` reduz, mas webhooks schema Stripe leaks em DTOs se não filtrar.

## Alternativas consideradas

1. **Pagar.me** — BR-native, Pix mais maduro, mas integração com cartão/dispute mais simples. Avaliar em M3 se volume justifica.
2. **PagSeguro/Mercado Pago** — comissão menor, mas UX checkout pior.
3. **Próprio com bacen direto** — inviável regulatoriamente.

## Referências

- [[../patterns]] §10 (ACL), §11 (Idempotency), §12 (Circuit Breaker)
- [[../../13-research/netflix/_destilado]] §2 (resilience providers)
- [[../c4-components]] §3.2 billing
