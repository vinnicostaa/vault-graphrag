---
title: Stripe — Overview
type: note
tags: [provider, stripe, payments, overview]
source: https://docs.stripe.com
source_date: 2026-04-24
created: 2026-04-24
updated: 2026-04-24
aliases: [Stripe Overview]
---

# Stripe — Overview

Visão geral destilada da plataforma Stripe: arquitetura de objetos, Connect (marketplace), Webhooks, modelo de preço e quando Stripe vence (ou perde) pros concorrentes.

> **Fonte oficial**: https://docs.stripe.com — consultada em 2026-04-24. Links: [[_moc]], [[../_moc]], [[../../security/_moc]].

---

## Arquitetura de objetos (núcleo)

Stripe modela pagamento como grafo de recursos versionados. Os 6 objetos centrais:

| Objeto | Papel | Observação |
|---|---|---|
| **Customer** | Entidade de billing (≠ `User` do seu sistema) | Agrega `PaymentMethod`, `Subscription`, `Invoice`. Relação 1:1 com `User` via `metadata.user_id` |
| **Product** | Item do catálogo (plano, serviço, SKU) | Imutável na essência; mudou → novo Product |
| **Price** | Preço de um `Product` (valor + moeda + recorrência) | Um Product pode ter N Prices (mensal, anual, por quantidade) |
| **Subscription** | Instância de cobrança recorrente ligada a `Customer` + `Price` | Tem ciclo de vida com 8 status (trialing, active, past_due, unpaid, canceled, incomplete, incomplete_expired, paused) |
| **Invoice** | Documento de cobrança (gerado auto por Subscription ou manual) | Coleta via `charge_automatically` ou `send_invoice` |
| **PaymentIntent** | Intent transacional com suporte 3DS / SCA | Substituiu `Charge` legado. Fluxo `requires_payment_method` → `requires_confirmation` → `requires_action` → `succeeded` |

**Relação funcional**:

```
User (seu DB) ──1:1── Customer (Stripe)
                       ├── PaymentMethod[]  (cartões tokenizados)
                       ├── Subscription[]   ── Price ── Product
                       └── Invoice[]        ── PaymentIntent ── Charge (legado, auto-gerado)
```

Ver [[patterns]] pra padrões de lookup Customer via `metadata.user_id`.

---

## Stripe Connect (plataforma / marketplace)

Connect viabiliza **plataforma com múltiplos sellers** — você intermedia pagamentos entre comprador final e terceiros, recebendo `application_fee`. Três tipos de conta:

| Tipo | Onboarding | UI Dashboard | Responsabilidade compliance | Quando usar |
|---|---|---|---|---|
| **Standard** | Conta Stripe do próprio seller (pode ser pré-existente) | Stripe hostado | Seller (KYC, dispute, taxes) | Seller é negócio estabelecido com conta Stripe própria |
| **Express** | Onboarding Stripe-hostado simplificado | Stripe hostado (limitado) | Dividida com plataforma | Marketplace comum (SaaS B2B2C, ride-sharing) |
| **Custom** | Totalmente customizado via API | Plataforma constrói UI | Plataforma (risco total) | Precisa controle total da experiência; maior esforço de compliance |

**Fee model** na plataforma: `application_fee_amount` em cada `PaymentIntent` ou `transfer_data.amount` redireciona saldo. Separação contábil automática.

---

## Webhooks

Stripe emite **eventos assíncronos** via HTTP POST no endpoint registrado. Pontos críticos:

- **Versão de API**: cada endpoint fixa versão (`2024-06-20` etc.) — evita breaking change automático
- **Assinatura HMAC SHA256** via header `Stripe-Signature` (timestamp + payload bruto)
- **Tolerance janela** de 5 min pra replay protection (ver [[patterns]])
- **Eventos core**: `customer.subscription.*`, `invoice.*`, `payment_intent.*`, `charge.dispute.created`, `account.updated` (Connect)
- **At-least-once delivery** — idempotência do handler é obrigatória (ver [[../../patterns/_moc]] → idempotency)
- **Retry com backoff exponencial** até 3 dias se endpoint não devolver 2xx

---

## Modelo de preço

Sem mensalidade fixa; cobra por transação bem-sucedida. Resumo (valores podem mudar; verificar em https://stripe.com/pricing):

| Região | One-time / Subscription | Observação |
|---|---|---|
| **US** | 2.9% + US$ 0.30 | Cartão doméstico |
| **US internacional** | +1.5% | Cartão emitido fora |
| **BR (cartão)** | ~3.99% + R$ 0.39 | Varia com volume negociado |
| **BR (Pix via Stripe)** | Suporte limitado comparado a gateway local | Confirmar disponibilidade no dashboard |
| **Connect** | +0.25% + US$ 0.25 por transferência (Express/Custom) | Standard não cobra extra |
| **Billing (Subscriptions)** | +0.5% no plano Starter | Até volume X; depois escalona |
| **Radar (fraud)** | Incluso básico; advanced +US$ 0.05/tx | Opt-in |

---

## Concorrentes — quando cada um vence

| Cenário | Vencedor provável | Rationale |
|---|---|---|
| Assinatura internacional multi-moeda com trials/proration | **Stripe** | Granularidade de billing é classe-mundial |
| Marketplace B2B2C com sellers em múltiplos países | **Stripe Connect** | Compliance multi-país incluso |
| Pequeno negócio BR com ticket médio baixo e Pix central | **Pagar.me / Iugu** | Pix nativo, NF integrada, custo menor em volume baixo |
| SaaS simples com um plano único e sem trial | **Paddle / LemonSqueezy** | Merchant-of-record (lida com sales tax por você) |
| Cobrança recorrente BR com boleto + Pix + recorrência via cartão | **Iugu / Asaas** | Suporte boleto nativo e régua de cobrança BR |
| Plataforma precisando de 3DS / SCA na Europa | **Stripe** | PaymentIntent desenhado pra PSD2 |

**Decisão típica**: se projeto tem qualquer um de {multi-country, marketplace, complex billing, SaaS global} → Stripe. Se é BR-only com ticket baixo e Pix central → avaliar gateway local antes.

---

## Próximos passos

- Setup prático: [[integration-guide]]
- Padrões atemporais: [[patterns]]
- Erros comuns: [[antipatterns]]
- Aplicações concretas: [[usage-in-projects]]
- Vizinhos: [[../_moc]], [[../../security/_moc]], [[../../patterns/_moc]]
