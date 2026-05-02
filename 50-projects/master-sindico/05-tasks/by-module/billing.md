---
title: Tasks — BC billing
type: spec
tags: [tasks, by-module, billing, master-sindico, v2]
source: 04-requirements/functional/billing.md + backlog.md
created: 2026-04-23
updated: 2026-04-23
---

# Tasks — BC `billing`

Tasks do bounded context **billing** (Stripe + planos + trial persona-aware + quotas + soft-block + webhooks idempotentes).

Alinhamento: [[../../04-requirements/functional/billing]] (BIL-001..BIL-014).

---

## M1

| ID | Título | Sprint | Status |
|---|---|---|---|
| T-006 | Consent registry versionado (LGPD base) + endpoint GET/DELETE `/me` | 10 | ⏳ |
| T-043 | Stripe live mode + webhook prod configurado | 10 | ⏳ |
| T-610 | Customer Portal Stripe link UI (BIL-010) | 10 | ⏳ |
| T-611 | Quota exhausted UI feedback (paywall elegante) | 10 | ⏳ |

## M2

| ID | Título | Sprint | Status |
|---|---|---|---|
| T-106 | Reset quota Connect Me 1º-jan (cron) (BIL-009) | 11 | ⏳ |
| T-612 | Reset quota vídeos mensal (cron 1º de cada mês) | 11 | ⏳ |
| T-613 | Upgrade/downgrade proporcional Stripe pro-rata (BIL-006) | 12 | ⏳ |

---

## Invariantes cobertos

I-006..I-012 de [[../../01-domain/bounded-contexts#billing]]:
- I-006 `users.plan_id` → Subscription ativa ou NULL (soft-block não apaga)
- I-007 `subscriptions` append-only
- I-008 Trial não volta após conversão paid
- I-009 `Money` int64 centavos
- I-010 Webhook `event.id` único no ledger
- I-011 `used ≤ limit` quando `!unlimited`
- I-012 Trial persona-aware (síndico 15d, empresa 7d, parceiro 30d)

---

## Links

- [[_moc]]
- [[../../04-requirements/functional/billing]]
- [[../../01-domain/bounded-contexts#billing]]
- [[../backlog]]
