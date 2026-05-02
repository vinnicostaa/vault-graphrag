---
title: Stripe — usage across projects
type: note
tags: [provider, stripe, cross-project]
created: 2026-04-22
updated: 2026-04-23
---

# Stripe — usage-in-projects

Tabela de projetos do vault que usam Stripe + link pra aplicação concreta.

| Projeto | Uso | Aplicado em |
|---|---|---|
| [[../../../50-projects/master-sindico/_moc\|Master Síndico]] | Stripe Connect (plataforma com síndicos como sellers) + Subscriptions (planos Base/Pagante do condomínio) + Webhook handler (gov. de plano → invalidação de cache ABAC) | [[../../../50-projects/master-sindico/08-security/BEYOND_CORP]] (ABAC × plano), [[../../../50-projects/master-sindico/STATE]] (D-### relacionados), `_source-material/.../contextos/stripe-context.md` (101K contexto — fora do vault) |

## Padrão de integração observado

- Assinar Connect Account como síndico; plataforma recebe % sobre cobranças condominiais
- Webhook signing verificado em todo endpoint (HMAC SHA256)
- Idempotency key em operações de mutação (evita duplo-charge em retry)
- Cache ABAC invalida em eventos: `customer.subscription.updated`, `customer.subscription.deleted`, `invoice.paid`, `invoice.payment_failed`

## Quando criar novo projeto com Stripe

1. Ler [[_moc]] primeiro (visão geral)
2. Ler [[integration-guide]] (setup inicial)
3. Ver padrão do Master Síndico em `_source-material/2026-04-23-raw-source/from-master-sindico/contextos/stripe-context.md` (fonte bruta fora do vault, enquanto não destilado)
4. Se usar Connect, ler [Stripe Connect docs](https://docs.stripe.com/connect) — é subconjunto específico
