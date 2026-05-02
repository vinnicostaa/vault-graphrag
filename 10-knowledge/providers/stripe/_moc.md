---
title: MOC — Stripe (Pagamentos / Connect / Subscriptions)
type: moc
tags: [moc, provider, stripe, payments, subscriptions, connect, webhooks]
category: payments
mcp: https://mcp.stripe.com
sdk-official-go: stripe-go v78+ (github.com/stripe/stripe-go/v78)
sdk-official-node: stripe v17+ (@stripe/stripe-node)
sdk-official-dart: stripe_dart (community) / flutter_stripe (Flutter)
doc-oficial: https://docs.stripe.com
doc-consulted: 2026-04-23
created: 2026-04-22
updated: 2026-04-23
---

# Stripe

**Categoria**: Pagamentos (one-time + subscriptions) · Stripe Connect (marketplace/plataforma) · Webhooks · Billing.

**MCP disponível**: ✅ [mcp.stripe.com](https://mcp.stripe.com) — priorizar este sobre WebFetch pra consultas dinâmicas.

**SDKs oficiais**:
- Go: `github.com/stripe/stripe-go/v78` (ou v79+ conforme release atual)
- Node: `stripe` npm package
- Flutter: `flutter_stripe` (plugin oficial para mobile)
- Python: `stripe`

**Doc oficial**: https://docs.stripe.com — consultada em 2026-04-23.

**Sandbox**: `sk_test_...` / `pk_test_...` (modo test) — objetos criados em test não refletem em live. Dashboard tem toggle Live/Test.

**Modelo de preço**: 2.9% + $0.30 por transação (US) | 3.99% + R$0.39 (BR Cartão) · Connect platform fee configurável · No monthly fee.

---

## Quando usar Stripe

1. **Pagamento recorrente complexo** (trials, proration, múltiplos planos, add-ons) — melhor que competitors (Pagar.me, Iugu) em granularidade
2. **Marketplace / plataforma multi-seller** — Stripe Connect é padrão-ouro (Standard, Express, Custom accounts)
3. **Pagamento internacional** (multi-currency, 135+ moedas)
4. **Compliance PCI DSS sem tocar PAN** — Stripe.js/Elements/PaymentSheet mantém PAN fora do servidor
5. **Webhook robusto** (signing, replay, idempotency key)

## Quando NÃO usar Stripe

- **Brasil-only small business com baixo volume** → Pagar.me/Iugu às vezes vencem em ticket baixo, Pix nativo, NF integrada
- **Só Pix** → Gateway local (Asaas, Gerencianet) mais simples
- **Assinatura super simples sem trial/proration** → Paddle/LemonSqueezy mais leve

---

## Sub-docs do Stripe

- [[overview]] — visão geral: produtos, customers, subscriptions, invoices, Connect
- [[integration-guide]] — setup: env vars, webhooks, teste local, checkout
- [[patterns]] — idempotência, reconciliação, webhook signing, cache invalidation, error handling
- [[antipatterns]] — o que NÃO fazer (ex: criar Customer a cada login; comparar status por string)
- [[usage-in-projects]] — quais projetos usam e como

## Glossário interno do Stripe (confusões comuns)

- **Customer** (Stripe) ≠ seu `User`. É uma entidade de billing, pode ter múltiplos PaymentMethods e Subscriptions.
- **Subscription** ≠ Plan. Subscription é instância; Product+Price são o catálogo.
- **Price** ≠ Amount. Price é o preço cadastrado no catálogo; Amount é o valor de uma transação.
- **Invoice** é gerada automaticamente por Subscriptions; pode ser manual (`send_invoice` collection method).
- **PaymentIntent** ≠ Charge. PaymentIntent é o novo modelo (3DS ready); Charge é legado.
- **Connect Account** (marketplace) tem tipos: `standard` (Stripe UI), `express` (onboarding Stripe-hosted), `custom` (você UI).

## Fontes brutas (pendente destilar — fora do vault)

Material-fonte bruto preservado em filesystem (`_source-material/`, não indexado pelo Obsidian):

- `_source-material/2026-04-23-raw-source/from-master-sindico/contextos/stripe-context.md` — 101K de contexto do projeto Master Síndico (Connect + Subscriptions + pricing BR)
- `_source-material/2026-04-23-raw-source/from-master-sindico/contextos/stripe-helper.md` — 19K de helpers de código

Path absoluto: `/home/vinni/Documentos/Projetos/Privados/automation-agents/_source-material/2026-04-23-raw-source/from-master-sindico/contextos/`

> **Estado**: destilação pendente em `overview.md`, `integration-guide.md`, `patterns.md`, `antipatterns.md`. Agente acessa fontes via Read/Bash no filesystem, não MCP Obsidian (fora do vault). Aplicação concreta ao projeto: [[../../../50-projects/master-sindico/_moc|Master Síndico]].

## Vizinhos no grafo

- [[../_moc]] — providers raiz
- [[../../security/_moc]] — webhook signing + PCI
- [[../../antipatterns/_moc]] — antipatterns gerais (não só Stripe)
- [[usage-in-projects]] — projetos consumindo Stripe
