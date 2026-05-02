---
title: >-
  Functional Requirements — Billing BC (consolidado Fase 8, ADR-0032
  Stripe-style)
type: spec
tags:
  - requirement
  - billing
  - matrix
  - master-sindico
  - v2
  - fase-8-consolidado
  - stripe
  - abac
  - ears
source: >-
  sub-produtos/* + ADR-0032 + STATE D-056..D-069 (Fase 8) + D-070..D-076 (Fase
  7) + _chaos/requirements(6).md (2026-04-13, R61 absorvido com correção)
created: 2026-04-23T00:00:00.000Z
updated: '2026-04-25'
status: consolidado-fase-8
dual_check: pending
fase_c_absorvido: true
---

> **Banner Fase C (2026-04-25)**: ratificação canônica — `_chaos/requirements(6).md` R61 menciona Mercado Pago, **REJEITADO**. Stripe canônico (BIL-001..BIL-055 cobre 100%). PIX via Stripe BR (não Pagar.me) reavaliar M2+ via ADR. R62-R67 (integrações email, SMS, video, storage, CEP, push) cobertos em XD-015..XD-019. Cobertura Fase C: 100% no-op.

# Functional Requirements — Billing BC

Requisitos funcionais do **bounded context `billing`**. Responsabilidade: catálogo global de planos (Stripe-style), assinaturas, Stripe integration, trial persona-aware, quotas de uso, soft-block, feature flags por plano, refunds/disputes.

**Formato**: EARS ("Quando …, o sistema deve …" / "Enquanto …, o sistema deve …"). **Prioridade**: M (M1), S (M2), C (M3+), W (backlog).

> **Invalidação de versões anteriores**: toda spec anterior que mencionava slugs `syndic_n1|n2|n3`, `enterprise_plus|pro`, `resident_base|paid`, `local_company_standard`, `marketing_standard` está **invalidada** por ADR-0032 (planos globais) + D-067 (abolir N1/N2/N3 Fase 8). Este arquivo é a fonte de verdade consolidada pós-Fase 8.

## Pré-condições de modelo (ADR-0032)

- `plan_tier ∈ {trial, base, plus, pro}` — enum universal, aplica-se a TODAS as personas.
- `user.role ∈ {syndic, resident, enterprise, marketing, local_company, admin}` — ortogonal ao tier.
- Permissão efetiva = f(`user.role`, `plan.tier`, `action`, `resource`) via ABAC engine (ver matrix-functional.md + ADR-0012).
- Entidade `Plan` é global (1 row por produto no catálogo); slug composto por persona **proibido** em código, banco, UI, docs.
- Subscription preserva `plan_id` mesmo se `user.role` mudar (ex: morador eleito síndico).
- Morador Base Connect Me = 2/ano (D-094 Fase 11 revoga D-058/D-079 que ratificavam 0/ano).

---

## 1. Plan catalog (BIL-001..BIL-005)

### BIL-001 — Catálogo global de planos (Stripe-style)

**Prioridade**: M
**EARS**: *Quando* admin MS registra um plano novo, *o sistema deve* persistir em `plans {id UUID, tier, name, label_ui, description, features JSONB, quotas JSONB, price_cents BIGINT (nullable), currency CHAR(3), interval, stripe_price_id, active BOOL, created_at, updated_at}`. *Quando* frontend consulta `GET /plans?active=true` (sem filtro por persona), *o sistema deve* retornar lista completa ordenada por `tier ASC, price_cents ASC`.

**Critério de aceite**:
- `tier` é enum Postgres `plan_tier ∈ {trial, base, plus, pro}` — único enum de plano no DB.
- `price_cents` em BIGINT (centavos BRL); `currency='BRL'` validado por CHECK constraint.
- `features` e `quotas` em JSONB — validados via schema no domínio (não no DB).
- Cache Redis `plans:catalog:v1` com TTL 1h, invalidação via evento `billing.plan.updated`.
- Slug composto por persona **proibido** — lint (migration review) rejeita `*_n1`, `*_plus`, `*_paid` em `name|id|any column`.

**Fonte**: ADR-0032 §Modelo canônico adotado; Stripe Products & Prices.

---

### BIL-002 — CRUD de plano restrito a admin MS

**Prioridade**: M
**EARS**: *Quando* usuário com role ≠ `admin` chama `POST /admin/plans`, `PATCH /admin/plans/{id}`, `DELETE /admin/plans/{id}`, *o sistema deve* rejeitar com `403 FORBIDDEN` antes de qualquer processamento. *Quando* admin cria/edita plano, *o sistema deve* exigir **4-eyes approval** (D-054 legado) em mudanças que afetam `price_cents`, `tier`, `quotas` — registrando aprovador em `plan_approvals {plan_id, approver_id, approved_at, reason}`.

**Critério de aceite**:
- Middleware ABAC valida `user.role = admin` antes do handler.
- Price change requer 2 admin IDs distintos (criador ≠ aprovador).
- Audit trail `audit_trail {entity=plan, action, diff_before, diff_after, actor_id}`.

**Fonte**: D-061 Fase 8 (Admin MS role privilegiada); D-054 legado (4-eyes para finance).

---

### BIL-003 — Seed inicial do catálogo (M1)

**Prioridade**: M
**EARS**: *Quando* migration de seed M1 roda, *o sistema deve* popular o catálogo com os planos-base:

| Plano (nome comercial) | Tier | Trial days | Price cents/mês | Público-alvo (role) |
|---|---|---|---|---|
| Trial Plataforma | trial | — | 0 | todos (depende da role — ver BIL-010) |
| Plano Base | base | 0 | 0 | resident (gratuito) |
| Plano Essencial | base | 0 | 990 | resident (vira Plus da persona; substitui "Morador Pagante" histórico) |
| Plano Plus | plus | 15 | (a definir) | syndic, enterprise, local_company |
| Plano Pro | pro | 15 | (a definir) | syndic, enterprise |

**Critério de aceite**:
- Migration `00600_seed_plans.up.sql` idempotente (`ON CONFLICT (id) DO NOTHING`).
- `stripe_price_id` nullable em trial/base-gratuito; obrigatório em planos pagos.
- Pendência: preços exatos de Plus/Pro — travar com stakeholder antes de Sprint 10.

**Fonte**: BIL-001; ADR-0032 Apêndice (procedimento de instanciação).

---

### BIL-004 — Planos globais atravessam mudança de role

**Prioridade**: M
**EARS**: *Quando* `user.role` muda (ex: morador eleito síndico em assembleia), *o sistema deve* preservar `subscription.plan_id` inalterado. *Quando* nova role exige tier mínimo (ex: syndic precisa `tier ∈ {plus, pro}` pra criar condomínio), *o sistema deve* aplicar soft-block nas features não cobertas e disparar evento `billing.upgrade_required {user_id, missing_tier, missing_actions}` para UI oferecer upgrade.

**Critério de aceite**:
- Não há chamada Stripe ao mudar role; apenas registro em `role_changes {user_id, from_role, to_role, changed_at, reason}`.
- Matriz ABAC reavalia permissões na primeira request pós-mudança (cache `ms:abac:{user_id}:*` invalidado).
- Teste: morador plano `base` (grátis) → eleito síndico → backend mantém `plan_id`; síndico-actions retornam `402 UPGRADE_REQUIRED` até user comprar tier plus+.

**Fonte**: ADR-0032 §Upgrade/downgrade entre personas.

---

### BIL-005 — Nomes comerciais vs tier canônico

**Prioridade**: M
**EARS**: *Quando* UI renderiza plano, *o sistema deve* usar `plan.label_ui` (i18n). *Quando* backend toma decisão ABAC/quota, *o sistema deve* usar exclusivamente `plan.tier` — nunca `plan.name` ou `plan.label_ui`.

**Critério de aceite**:
- Code review rejeita `plan.Name == "Plus"` em ABAC/quota logic — só `plan.Tier == TierPlus` permitido.
- Teste: renomear `Plano Plus` → `Plano Profissional` na UI não quebra nenhum gate ABAC.

**Fonte**: ADR-0032 §Consequências neutras.

---

## 2. Subscription lifecycle (BIL-006..BIL-015)

### BIL-006 — Estrutura de subscription (append-only)

**Prioridade**: M
**EARS**: *Quando* user compra plano via Checkout Stripe, *o sistema deve* criar `subscriptions {id, user_id, plan_id, stripe_subscription_id, stripe_customer_id, status, trial_started_at, trial_ends_at, current_period_start, current_period_end, cancel_at_period_end, canceled_at, created_at, updated_at}`. *Quando* assinatura muda de plano, *o sistema deve* criar **novo row** preservando histórico (invariante 2).

**Critério de aceite**:
- Tabela append-only — sem `UPDATE` destrutivo (só status/period columns via webhook).
- Status enum: `trial | active | past_due | canceled | paused | expired`.
- Unique: `(user_id) WHERE status IN ('trial','active')` — 1 assinatura ativa por user.

**Fonte**: ADR-0032; invariante 2 legado.

---

### BIL-007 — Transições de status (state machine)

**Prioridade**: M
**EARS**: *Quando* evento Stripe dispara transição, *o sistema deve* aceitar apenas transições legais:

```
trial    → active (pagamento ok) | expired (trial esgotou)
active   → past_due (pagamento falhou) | canceled (user cancelou) | paused (admin)
past_due → active (retry ok) | canceled (retry esgotou)
canceled → expired (fim do período) | active (reativação dentro do período)
paused   → active (admin) | canceled
expired  → (terminal; nova subscription requer novo row)
```

**Critério de aceite**:
- Guard no domínio: `Subscription.TransitionTo(newStatus)` retorna erro se ilegal.
- Teste PBT: toda combinação (`from`, `to`) ∉ whitelist retorna `ErrIllegalTransition`.

**Fonte**: ADR-0032; Stripe subscription states.

---

### BIL-008 — Trial auto-start persona-aware

**Prioridade**: M
**EARS**: *Quando* user completa onboarding (Zitadel status = `active`), *o sistema deve* criar subscription trial com `trial_started_at=now()`, `trial_ends_at=now() + trial_days`, `status=trial`. *Quando* `user.role` é `syndic` → `trial_days=15`; `enterprise` → 7; `local_company` → 30; `resident` → **sem trial** (inicia em Plano Base grátis).

**Critério de aceite**:
- Duração de trial é **dado do Plan** (não do código) — `plans.trial_days` consultado por `user.role` → trial-plan mapping seed.
- Lookup de "qual trial-plan corresponde a qual role" via tabela `role_trial_plan_map {role, plan_id}` populada em seed.
- Teste: onboard syndic → subscription com `trial_ends_at = now() + 15d`.
- Teste: onboard resident → subscription status=active direto, plan=Base grátis (sem trial).

**Fonte**: ADR-0032; legado `billing.md` Req 4.1 (persona-aware trial); D-067 Fase 8 (abolir slug preservando duração por persona).

---

### BIL-009 — Comportamento durante trial (soft-block estratégico)

**Prioridade**: M
**EARS**: *Enquanto* `subscription.status = trial`, *o sistema deve* permitir leitura de TODAS as features visíveis à role, mas bloquear **ações estratégicas** definidas na matriz (ver matrix-functional.md). *Quando* ação bloqueada é tentada, *o sistema deve* responder `402 PAYMENT_REQUIRED` com `code=trial_feature_blocked` e mensagem EN-i18n "Ative seu plano para usar essa funcionalidade".

**Critério de aceite**:
- Middleware ABAC consulta matriz: `(role, plan.tier=trial, action)` → se `entry.trial_allowed=false` → 402.
- Resposta 402 inclui `upgrade_url` apontando Customer Portal Stripe.
- Teste: syndic trial → criar atividade Timeline → 402.
- Teste: syndic trial → ler Timeline → 200.

**Fonte**: legado `billing.md` Req 4.1 reinterpretado em ADR-0032.

---

### BIL-010 — Trial banner T-24h

**Prioridade**: M
**EARS**: *Quando* `trial_ends_at - now() ≤ 24h`, *o sistema deve* flaggear `trial_warning=true` no payload `GET /me/subscription`. *Quando* frontend detecta flag, *o sistema deve* exibir banner persistente com CTA "Ative seu plano agora".

**Critério de aceite**:
- Cálculo server-side (frontend confia no flag, não faz date math).
- Banner link → Stripe Customer Portal via `POST /billing/portal`.

**Fonte**: legado Req 4.1.

---

### BIL-011 — Trial expirou: soft-block imediato (sem grace period)

**Prioridade**: M
**EARS**: *Quando* `trial_ends_at ≤ now()` E `status = trial`, *o sistema deve* transicionar `status → expired`, invalidar cache ABAC `ms:abac:{user_id}:*`, persistir `subscriptions.expired_at=now()`. *Quando* user faz request pós-expiração, *o sistema deve* aplicar soft-block em features premium (402 `PLAN_REQUIRED`) preservando acesso de leitura básica (feed, busca geral).

**Critério de aceite**:
- **Sem grace period** — D-058 legado invalidado por Fase 8; hoje: expiração é imediata (D-067/D-069 Fase 8 confirma soft-block sem janela de cortesia).
- Detecção: middleware lazy (`current_period_end < now()` OR `trial_ends_at < now()`) OU job noturno às 3h America/Sao_Paulo.
- Dados do user permanecem intactos (aggregates, uploads, histórico) — apenas actions premium bloqueadas.

**Fonte**: ADR-0032; STATE D-067/D-069 Fase 8 (reafirma soft-block imediato).

---

### BIL-012 — Cancelamento (soft via Customer Portal)

**Prioridade**: M
**EARS**: *Quando* user cancela via Stripe Customer Portal, *o sistema deve* receber webhook `customer.subscription.updated {cancel_at_period_end=true}` e persistir. *Quando* `current_period_end` passa, *o sistema deve* transicionar `status → expired` via webhook `customer.subscription.deleted`.

**Critério de aceite**:
- Reativação dentro do período → webhook `customer.subscription.updated {cancel_at_period_end=false}` → status volta a `active`.
- Dados preservados (soft-block).

**Fonte**: ADR-0032; Stripe cancel_at_period_end pattern.

---

### BIL-013 — Past-due → retry automático

**Prioridade**: M
**EARS**: *Quando* `invoice.payment_failed`, *o sistema deve* transicionar `subscription.status → past_due` e delegar retry ao Stripe Smart Retries (3 tentativas em 7 dias). *Quando* todas tentativas falham, *o sistema deve* receber `customer.subscription.deleted` e transicionar `status → canceled`.

**Critério de aceite**:
- Stripe Dashboard: Smart Retries configurado (3x, 7 dias).
- Email automático Stripe ao user; backend só loga + atualiza status.

**Fonte**: legado `billing.md` Req 4; Stripe best practices.

---

### BIL-014 — Trial não resetável

**Prioridade**: M
**EARS**: *Quando* user com subscription prévia (trial ou paid) cancela e tenta nova subscription, *o sistema deve* criar nova subscription com `status=active` direto (sem trial), exigindo pagamento imediato.

**Critério de aceite**:
- Invariante: `trial_ends_at IS NOT NULL` só na primeira subscription do user.
- Query guard: `SELECT COUNT(*) FROM subscriptions WHERE user_id=$1` > 0 → skip trial.
- PBT: 2ª subscription mesmo user → sem trial (fuzz test).

**Fonte**: legado `billing.md` invariante 3.

---

### BIL-015 — Pause/unpause (admin-only)

**Prioridade**: C
**EARS**: *Quando* admin emite `POST /admin/subscriptions/{id}/pause`, *o sistema deve* chamar Stripe `subscription.pause_collection` + registrar `status=paused`. *Quando* admin emite `unpause`, *o sistema deve* reverter.

**Critério de aceite**:
- Role `admin` obrigatória; audit trail `billing.subscription.paused|unpaused`.
- Paused ≠ canceled — user ainda tem acesso read-only até unpause ou expire.

**Fonte**: Stripe pause_collection; caso de uso admin support.

---

## 3. Stripe integration — webhooks (BIL-016..BIL-021)

### BIL-016 — Webhook HMAC signature verify

**Prioridade**: M
**EARS**: *Quando* `POST /webhooks/stripe` chega, *o sistema deve* verificar header `Stripe-Signature` contra `STRIPE_WEBHOOK_SECRET` via `webhook.ConstructEvent` (SDK Stripe Go) **antes** de parse/unmarshal. *Quando* signature inválida ou ausente, *o sistema deve* retornar `400 INVALID_SIGNATURE` sem logar body.

**Critério de aceite**:
- Raw body via `io.ReadAll(r.Body)` antes de qualquer operação.
- Tolerance default 5min (replay protection).
- Sem log do body em caso de falha (evita vazar PII em logs).

**Fonte**: ADR-0027 (webhook idempotency DB); GR-008 legado.

---

### BIL-017 — Webhook idempotência via UNIQUE (provider, event_id)

**Prioridade**: M
**EARS**: *Quando* webhook chega, *o sistema deve* inserir `webhook_events {id, provider, event_id, event_type, payload_hash, status, received_at, processed_at, last_error}` com `INSERT ... ON CONFLICT (provider, event_id) DO NOTHING RETURNING id`. *Quando* conflict (evento já processado), *o sistema deve* retornar `200 OK` sem reprocessar. *Quando* insert OK → processar transacionalmente (handler + status=processed em mesma UoW).

**Critério de aceite**:
- UNIQUE constraint `webhook_events (provider, event_id)`.
- Transação: handler executa → marca `status=processed` atomicamente; falha → `status=failed`, `last_error` populado, retorna 500 (Stripe retry).
- Teste: 100 deliveries idênticas → 1 processamento + 99 no-ops com 200.

**Fonte**: ADR-0027 (webhook-idempotency-db).

---

### BIL-018 — Eventos Stripe suportados (whitelist)

**Prioridade**: M
**EARS**: *Quando* webhook chega com `event.type` em whitelist, *o sistema deve* rotear para handler específico:

| event.type | Handler action |
|---|---|
| `checkout.session.completed` | Upsert subscription (cria se não existe; trial→active) |
| `customer.subscription.created` | Upsert subscription (status/plan/period) |
| `customer.subscription.updated` | Atualiza status, plan_id (prorata), current_period_end, cancel_at_period_end |
| `customer.subscription.deleted` | status→expired; `canceled_at=now()` |
| `invoice.payment_succeeded` | Renova `current_period_end`; status→active |
| `invoice.payment_failed` | status→past_due; registra last_payment_error |
| `customer.updated` | Sync email/billing address (se necessário) |
| `charge.dispute.created` | Alert admin (ver BIL-029) |
| `charge.refunded` | Registra refund (ver BIL-027) |

*Quando* event fora da whitelist chega, *o sistema deve* logar `info` + retornar 200 (não reprocessa).

**Critério de aceite**:
- Listener dispatch: `map[eventType]Handler` injetado no billing service.
- Teste integração por event type com fixtures Stripe.
- Log estruturado `zerolog` sem body raw (só event_id + type).

**Fonte**: ADR-0027; Stripe events catalog 2026.

---

### BIL-019 — Invalidação de cache ABAC pós-webhook

**Prioridade**: M
**EARS**: *Quando* webhook modifica `plan_id` ou `status` de subscription ativa, *o sistema deve* invalidar `ms:abac:{user_id}:*` (Redis DEL pattern) E publicar `billing.subscription.changed {user_id, old_plan_id, new_plan_id, old_status, new_status}` via NATS JetStream.

**Critério de aceite**:
- Invalidação dentro da mesma UoW do webhook handler.
- Consumer NATS em outros BCs recebe evento (ex: `commercial` recalcula quotas de Connect Me).
- Teste: upgrade plus→pro → próxima request do user tem `plan.tier=pro` (não cache stale).

**Fonte**: GR-002 (ABAC ordenado); ADR-0012 (ABAC Redis cache).

---

### BIL-020 — Retry automático Stripe → backend

**Prioridade**: M
**EARS**: *Quando* backend retorna ≠ 2xx ao webhook, *o sistema deve* confiar no retry automático Stripe (até 72h com exponential backoff). *Quando* `webhook_events.status=failed` persiste > 24h, *o sistema deve* disparar alert Sentry com `event_id, last_error, attempts_count`.

**Critério de aceite**:
- Stripe Dashboard: auto-retry habilitado (default).
- Sentry alert rule: tag `webhook:stripe:failed` + age > 24h.
- Admin Dashboard lista webhooks em `failed` para inspect manual.

**Fonte**: ADR-0027; Stripe webhook retry docs.

---

### BIL-021 — Webhook provider-agnostic (futuro M2+)

**Prioridade**: S
**EARS**: *Quando* novo provider (ex: Pagar.me PIX) for adicionado, *o sistema deve* suportar via mesma tabela `webhook_events` com coluna `provider` discriminante. Handler-map roteia por `provider → eventType → handler`.

**Critério de aceite**:
- Interface `IPaymentGateway` (ADR-0004, D-056) absorve novo adapter sem alterar `webhook_events` schema.
- ADR novo para PIX/Pagar.me se aprovado M2.

**Fonte**: D-056 Fase 8 (agnosticismo de provedor); ADR-0004.

---

## 4. Checkout e Customer Portal (BIL-022..BIL-024)

### BIL-022 — Stripe Checkout Session

**Prioridade**: M
**EARS**: *Quando* user autenticado chama `POST /billing/checkout {plan_id}`, *o sistema deve* validar: (a) `plan.active=true`, (b) user não tem subscription ativa conflitante, (c) quota rate-limit OK; então chamar `IPaymentGateway.CreateCheckoutSession(user_id, plan_id, metadata={user_id,plan_id,tenant_id?})` e retornar `{checkout_url}`. *Quando* user completa checkout → Stripe dispara `checkout.session.completed` (ver BIL-018).

**Critério de aceite**:
- Sucesso redirect: `${FRONTEND_URL}/billing/success?session_id={CHECKOUT_SESSION_ID}`.
- Cancel redirect: `${FRONTEND_URL}/billing/cancel`.
- Rate limit: 10 checkouts/hora por user (Redis `ratelimit:checkout:{user_id}`).

**Fonte**: Stripe Checkout docs; legado Req 4.

---

### BIL-023 — Customer Portal Session

**Prioridade**: M
**EARS**: *Quando* user chama `POST /billing/portal`, *o sistema deve* chamar `IPaymentGateway.CreateBillingPortalSession(user_id, return_url)` e retornar `{portal_url}`. *Quando* `stripe_customer_id` não existe ainda, *o sistema deve* criar via `stripe.Customer.create {email=user.email, metadata={user_id}}` lazily.

**Critério de aceite**:
- Return URL: `${FRONTEND_URL}/settings/billing`.
- Portal permite: mudar cartão, baixar fatura, cancelar, upgrade/downgrade.
- TTL session default Stripe (1h).

**Fonte**: Stripe Customer Portal docs.

---

### BIL-024 — Fatura retrieval

**Prioridade**: S
**EARS**: *Quando* user chama `GET /me/invoices?limit=20&cursor=...`, *o sistema deve* chamar `IPaymentGateway.ListInvoices(user_id, cursor, limit)` e retornar `{invoices:[{id, amount_cents, status, hosted_invoice_url, invoice_pdf_url, period_start, period_end}], next_cursor}`.

**Critério de aceite**:
- Stripe é fonte da verdade — backend **não persiste** fatura (evita sync drift).
- hosted_invoice_url + invoice_pdf_url expiram conforme Stripe (24h).

**Fonte**: legado `billing.md` Req 4; Stripe Invoices API.

---

## 5. Upgrade / downgrade proporcional (BIL-025..BIL-026)

### BIL-025 — Prorata via Stripe (upgrade)

**Prioridade**: M
**EARS**: *Quando* user muda para tier superior via Customer Portal, *o sistema deve* delegar cálculo de prorata ao Stripe (`proration_behavior=create_prorations`). *Quando* webhook `customer.subscription.updated` chega com `plan_id` novo, *o sistema deve* UPSERT subscription com `plan_id=new`, **preservando histórico** via `subscription_changes {subscription_id, old_plan_id, new_plan_id, changed_at, prorated_amount_cents}`.

**Critério de aceite**:
- Stripe cobra diferença proporcional automaticamente.
- `subscription_changes` é append-only (invariante 2 estendido).
- Cache ABAC invalidado imediatamente (BIL-019).

**Fonte**: Stripe proration docs; ADR-0032.

---

### BIL-026 — Downgrade (imediato ou agendado)

**Prioridade**: M
**EARS**: *Quando* user faz downgrade via Customer Portal com `proration_behavior=create_prorations`, *o sistema deve* receber webhook imediato e bloquear features premium **sincronamente** após ABAC cache invalidate. *Quando* user escolhe `schedule_at_period_end` (Stripe feature), *o sistema deve* manter features atuais até `current_period_end`, então aplicar downgrade.

**Critério de aceite**:
- Teste: Pro → Plus imediato → tentativa de action Pro-only → 402 `PLAN_REQUIRED`.
- Teste: Pro → Plus agendado → features Pro funcionam até fim do período.

**Fonte**: Stripe subscription schedules.

---

## 6. Refunds e disputes (BIL-027..BIL-029)

### BIL-027 — Refund admin-only com 4-eyes

**Prioridade**: S
**EARS**: *Quando* admin emite `POST /admin/subscriptions/{id}/refund {amount_cents, reason}`, *o sistema deve* exigir aprovação de 2º admin via `POST /admin/refunds/{pending_id}/approve` antes de chamar `IPaymentGateway.CreateRefund`. *Quando* aprovado, *o sistema deve* chamar Stripe, registrar `refunds {id, subscription_id, amount_cents, reason, requester_id, approver_id, stripe_refund_id, status, created_at}` e emitir evento `billing.refund.issued`.

**Critério de aceite**:
- Role `admin` obrigatória em ambos endpoints.
- `requester_id ≠ approver_id` — validado.
- Audit trail completo (`audit_trail`) + LGPD log se afeta dado pessoal.

**Fonte**: D-054 legado (4-eyes finance); D-061 Fase 8 (admin role privilegiada).

---

### BIL-028 — Refund parcial e total

**Prioridade**: S
**EARS**: *Quando* `amount_cents` especificado ≤ `invoice.amount_paid`, *o sistema deve* processar refund parcial. *Quando* `amount_cents` omitido, *o sistema deve* processar refund total da última invoice.

**Critério de aceite**:
- Validação: `amount_cents > 0`; `amount_cents ≤ invoice.amount_paid`.
- Stripe API: `refund.create {payment_intent, amount?, reason}`.

**Fonte**: Stripe Refund API.

---

### BIL-029 — Dispute (chargeback) — alert admin

**Prioridade**: S
**EARS**: *Quando* webhook `charge.dispute.created` chega, *o sistema deve* persistir `disputes {id, subscription_id, stripe_dispute_id, amount_cents, reason, status, evidence_due_by, created_at}` e disparar alert prioritário (Sentry + email admin@mastersindico).

**Critério de aceite**:
- Status enum: `warning_needs_response | needs_response | under_review | won | lost | charge_refunded`.
- Admin Dashboard exibe disputes ativas com contagem regressiva para `evidence_due_by`.

**Fonte**: Stripe Disputes API.

---

## 7. Quotas (BIL-030..BIL-040)

### BIL-030 — Quota: Connect Me anual (reset 1º Jan)

**Prioridade**: M
**EARS**: *Quando* user emite Connect Me, *o sistema deve* decrementar quota via `feature_usages {user_id, feature_key, period_key, used, limit}` + cache Redis `ms:quota:{user_id}:connect_me:{year}`. *Quando* quota esgotada, *o sistema deve* retornar `403 QUOTA_EXCEEDED`. *Quando* 1º de janeiro 00:00 America/Sao_Paulo, *o sistema deve* resetar contador (TTL Redis expira; nova period_key).

**Quotas Connect Me (ADR-0032 matrix, D-058 Fase 8):**

| Role + tier | Quota Connect Me/ano |
|---|---|
| resident, base | **0** (D-058 Fase 8 — sem Connect Me) |
| resident, plus (Pagante) | 4/ano |
| syndic, trial | ⏳ 0 (bloqueado durante trial) |
| syndic, plus | 4/ano |
| syndic, pro | ∞ |
| enterprise, plus | ∞ (apenas recebe E→E) |
| enterprise, pro | ∞ (inclui iniciar E→E) |
| local_company | N/A (só recebe) |

**Critério de aceite**:
- Decremento transacional com criação do aggregate `ConnectMeRequest` (mesma UoW).
- `GET /me/quotas` retorna `{connect_me: {used, limit, reset_at}}`.
- PBT: sequência de `limit+1` requests → `limit` sucessos + 1 `403`.

**Fonte**: D-094 Fase 11 (Morador Base 2/ano — revoga D-058/D-079 que diziam 0/ano); ADR-0032 matriz de permissão.

---

### BIL-031 — Quota: vídeos institucionais mensal (reset 1º de cada mês)

**Prioridade**: M
**EARS**: *Quando* user com permissão de publicação publica vídeo institucional, *o sistema deve* decrementar quota mensal. *Quando* quota esgotada → `403 QUOTA_EXCEEDED`. *Quando* 1º do mês 00:00 America/Sao_Paulo → reset. *Quando* user sem permissão de publicação tenta publicar (ex: role=syndic + tier=base), *o sistema deve* responder `402 PLAN_REQUIRED` antes mesmo de checar quota.

**Quotas vídeos institucionais/mês:**

| Role + tier | Uploads/mês |
|---|---|
| syndic, trial | ⏳ bloqueado |
| syndic, base | 0 (sem publicação) |
| syndic, plus | 4/mês |
| syndic, pro | 12/mês |
| enterprise, plus | 8/mês |
| enterprise, pro | 12/mês |
| resident (qualquer tier) | 0 (só vídeo-currículo — BIL-033) |
| marketing | consome quota da empresa cliente via delegation grant |
| local_company | 0 (Vizinhança publica cupons, não vídeo institucional) |
| admin | ∞ |

**Critério de aceite**:
- Reset automático 1º do mês (job cron `0 0 1 * *` America/Sao_Paulo ou TTL Redis).
- Separada de Connect Me (BIL-030) — chaves distintas.
- Bloqueio antes da quota: middleware ABAC verifica feature flag `content.video.publish` por `(role, tier)`; só se liberado a quota é avaliada.

**Fonte**: ADR-0032; STATE D-067 (abolir N1/N2/N3 em quotas); sub-produtos/04-conteudo-videos.md §5.2.

---

### BIL-032 — Quota: condomínios por síndico (hard limit 15)

**Prioridade**: M
**EARS**: *Quando* syndic tenta criar 16º condomínio, *o sistema deve* responder `403 QUOTA_EXCEEDED code=max_condominiums_reached`.

**Critério de aceite**:
- Hard limit: 15 condomínios por conta de síndico (tier-independente para tier ≥ plus; tier=base → limite 1).
- Override apenas via admin manual (`POST /admin/users/{id}/quota-override`).

**Fonte**: legado Decision 11; sub-produtos/01-governanca-institucional.md §5.

---

### BIL-033 — Quota: vídeo-currículo (1 fixo, lock 90d)

**Prioridade**: M
**EARS**: *Quando* resident tier plus (Pagante) tenta novo upload de vídeo-currículo enquanto existe um publicado há < 90d, *o sistema deve* bloquear com `409 VIDEO_LOCKED`. *Quando* ≥ 90d → permite substituição (arquiva anterior).

**Critério de aceite**:
- UNIQUE por `user_id` em `curriculum_videos` (não é galeria).
- Teste: dia 89 → bloqueio; dia 91 → ok.

**Fonte**: legado `novo escopo-7.md` §2.6; sub-produtos/07-banco-de-talentos.md §6.2.

---

### BIL-034 — Quota: cupons Vizinhança (lock 4h por morador por parceiro)

**Prioridade**: M (ativo D-059 Fase 8)
**EARS**: *Quando* resident tier plus tenta gerar cupom em parceiro X enquanto já tem cupom gerado há < 4h no mesmo X, *o sistema deve* bloquear com `429 RATE_LIMITED code=coupon_cooldown` + header `Retry-After` em segundos.

**Critério de aceite**:
- Tabela `coupon_generations {coupon_id, user_id, partner_id, generated_at}`.
- Trava computada via `SELECT NOT EXISTS (… WHERE user_id=$1 AND partner_id=$2 AND generated_at > now() - interval '4 hours')`.
- Mensagem i18n: "Aguarde X horas pra gerar novo cupom desse estabelecimento".

**Fonte**: D-059 Fase 8 (Sistema Cupons ATIVO); sub-produtos/08-vizinhanca.md §10.2.

---

### BIL-035 — Quota: lives (D-097 — admin + Empresa Pro + agência delegada)

**Prioridade**: M
**EARS**: *Quando* user tenta criar live, *o sistema deve* permitir se: `role=admin` (qualquer tier) OU (`role=enterprise` AND `plan.tier=pro`) OU (`role=marketing` AND tem `delegation_grant` ativo da empresa Pro). Demais combinações → `403 FORBIDDEN code=live_not_authorized`. **D-097 Fase 11 revoga D-063/D-076** (admin-only no MVP).

**Critério de aceite**:
- ABAC shortcut: `action=live.create` → allow iff `role=admin` OR (role=enterprise AND tier=pro) OR (role=marketing AND has_active_delegation).
- Teste: syndic tier=pro → live.create → 403; enterprise tier=pro → 200; marketing com grant ativo → 200; marketing sem grant → 403.
- Teste: admin → live.create → 200.
- Pendência M2+: reavaliar se LMS abre lives para empresas (D-076 Fase 7).

**Fonte**: D-063 Fase 8; D-076 Fase 7 (Lives admin-only MVP).

---

### BIL-036 — Quota: tracking via Redis + DB (source of truth)

**Prioridade**: M
**EARS**: *Quando* ABAC avalia quota, *o sistema deve* consultar **Redis primeiro** (`ms:quota:{user_id}:{feature_key}:{period_key}`); se miss, *o sistema deve* consultar DB (`feature_usages`) e repovoar cache com TTL = tempo restante do período. *Quando* decremento, *o sistema deve* atualizar **DB (fonte da verdade) + cache** dentro da mesma UoW.

**Critério de aceite**:
- TTL Redis = `period_end - now()` em segundos.
- Consistência eventual pós-webhook: invalidação cache garante correção.
- Teste: derrubar Redis → quota consultada direto do DB → funciona.

**Fonte**: ADR-0012 (ABAC Redis cache); GR-002.

---

### BIL-037 — Quota: dashboard de uso (self-service)

**Prioridade**: S
**EARS**: *Quando* user acessa `GET /me/quotas`, *o sistema deve* retornar uso atual de todas quotas aplicáveis à sua `(role, tier)`:

```json
{
  "connect_me": {"used": 2, "limit": 4, "reset_at": "2027-01-01T03:00:00Z"},
  "videos_institutional": {"used": 3, "limit": 8, "reset_at": "2026-05-01T03:00:00Z"},
  "condominiums": {"used": 7, "limit": 15, "reset_at": null},
  "curriculum_video": {"locked_until": "2026-06-12T00:00:00Z"}
}
```

**Critério de aceite**:
- `limit = ∞` → JSON retorna `"limit": null`.
- Quotas não aplicáveis à role → não aparecem no payload (ex: resident não vê `condominiums`).

**Fonte**: legado Req 4 "SHOULD"; ADR-0032 matriz.

---

### BIL-038 — Quota: override admin manual

**Prioridade**: C
**EARS**: *Quando* admin emite `POST /admin/users/{id}/quota-override {feature_key, new_limit, reason, expires_at}`, *o sistema deve* persistir `quota_overrides {user_id, feature_key, limit_override, reason, granted_by, expires_at}` e invalidar cache. *Quando* ABAC consulta quota, *o sistema deve* preferir override se ativo e não expirado.

**Critério de aceite**:
- Admin role obrigatória; audit trail `billing.quota.overridden`.
- Override expira automaticamente (TTL via `expires_at`).

**Fonte**: D-061 Fase 8; caso de uso enterprise manual.

---

### BIL-039 — Quota: job noturno pré-cálculo

**Prioridade**: C
**EARS**: *Quando* cron diário 3h America/Sao_Paulo dispara, *o sistema deve* pré-calcular `feature_usages_agg_daily` materializando `(user_id, feature_key, date, count)` para consulta rápida em relatórios admin.

**Critério de aceite**:
- Idempotente (re-run no mesmo dia produz mesmo resultado).
- Incremental (só partições do dia anterior).

**Fonte**: legado Req 6 "SHOULD".

---

### BIL-040 — Quota: PBT para invariantes

**Prioridade**: M
**EARS**: *Quando* suite de testes roda, *o sistema deve* executar property-based test que gere sequência aleatória de requests e verifique: (a) `used ≤ limit` sempre; (b) soma de decrementos = `used`; (c) reset no 1º do período zera `used`; (d) quota_override sobrescreve limit sem mexer em `used`.

**Critério de aceite**:
- `gopter` ou `rapid` — 1000 iterações por property.
- Integrado em CI com seed determinístico.

**Fonte**: GR-017 (PBT para invariantes numéricos); Calisthenics.

---

## 8. Feature flags via plan.features JSONB (BIL-041..BIL-043)

### BIL-041 — Feature flags canônicas

**Prioridade**: M
**EARS**: *Quando* ABAC avalia ação, *o sistema deve* consultar `plan.features JSONB` combinado com matriz `(role, action) → required_flag`. *Quando* flag presente e `true` → allow; *quando* ausente ou `false` → deny.

**Feature keys canônicas (namespaced):**

```
institutional.timeline.create
institutional.announcement.create
institutional.master_plan.export_pdf
institutional.condominium.create
assembly.create
assembly.preside
assembly.vote.cast
commercial.connect_me.to_company
commercial.connect_me.to_syndic
commercial.connect_me.e2e
commercial.marketing_link.create
content.video.publish
content.video.publish.institutional
content.curriculum_video.publish
content.lms.course.create
content.lms.certificate.issue
content.live.start
reputation.badge.view
neighborhood.coupon.generate
neighborhood.coupon.create
neighborhood.seal.grant
compliance.annual_declaration
compliance.audit.view_full
admin.broadcast.send
admin.moderation.ban
admin.tenant.cross_scope
marketing.delegation.act_as
```

**Critério de aceite**:
- `features` em JSONB flat: `{"institutional.timeline.create": true, "content.video.publish": true, ...}`.
- Seed inicial populado por migration; admin pode editar via UI (BIL-002).
- Teste: remover flag `content.video.publish` do plano Plus → publish retorna 402.

**Fonte**: ADR-0032 §ABAC engine; sub-produtos/* matrizes.

---

### BIL-042 — Feature flag rollout gradual

**Prioridade**: C
**EARS**: *Quando* nova feature é lançada com flag `experimental=true`, *o sistema deve* avaliar `feature_flags_rollout {feature_key, enabled_for_percentage, enabled_for_user_ids[]}` antes de liberar. *Quando* user dentro do %, *o sistema deve* liberar.

**Critério de aceite**:
- Hash determinístico: `hash(user_id + feature_key) % 100 < percentage`.
- Default: feature desligada (fail-closed).
- Admin UI pra toggle e % rollout.

**Fonte**: legado Req 6; pattern canário.

---

### BIL-043 — ABAC consulta unificada (prevalência)

**Prioridade**: M
**EARS**: *Quando* ABAC resolve `(role, tier, action)`, *o sistema deve* seguir ordem:

1. Admin bypass (`role=admin` → allow sempre, exceto ações com `admin_scoped=false`)
2. Matriz canônica `(role, tier, action)` → deny explícito → `403`
3. `plan.features[action]` → ausente → `402 PLAN_REQUIRED`
4. Quota check (se aplicável) → esgotado → `403 QUOTA_EXCEEDED`
5. Tenant scope (se aplicável) → fora do scope → `403`
6. Rollout flag (se `experimental`) → fora do % → `404 NOT_FOUND` (feature-hidden)
7. allow

**Critério de aceite**:
- Ordem é testada em sequência (test matrix).
- Logging estruturado com `reason=(admin_bypass|matrix_deny|plan_missing|quota_exceeded|tenant_out_of_scope|experimental_hidden|allowed)`.

**Fonte**: ADR-0032; ADR-0012 (ABAC Redis cache).

---

## 9. Billing cycle, moeda e cálculo financeiro (BIL-044..BIL-047)

### BIL-044 — Billing cycle mensal/anual

**Prioridade**: M
**EARS**: *Quando* admin cria plano no Stripe, *o sistema deve* suportar `interval ∈ {month, year}`. *Quando* user escolhe anual, *o sistema deve* cobrar `price_cents * 12 * desconto_anual%` (configurável por plano, default 0% no M1).

**Critério de aceite**:
- Backend não calcula prorata — Stripe faz (BIL-025).
- `subscriptions.interval` espelhado do Stripe para cache local.

**Fonte**: Stripe pricing docs; ADR-0032.

---

### BIL-045 — Moeda BRL (MVP)

**Prioridade**: M
**EARS**: *Enquanto* plataforma opera no Brasil, *o sistema deve* usar `currency='BRL'` em Stripe e banco. *Quando* multi-moeda (M6+), *o sistema deve* exigir nova ADR.

**Critério de aceite**:
- CHECK constraint: `plans.currency = 'BRL'`.
- Valores em centavos (BIL-046).

**Fonte**: STEERING §3.

---

### BIL-046 — Money VO: int64 centavos, imutável, zero float

**Prioridade**: M
**EARS**: *Quando* código manipula valor monetário, *o sistema deve* usar `MoneyVO {amount int64, currency Currency}` imutável — com métodos `Add`, `Sub`, `Mul(factor)`, `Pct(basis_points)`. *Quando* operação retorna novo Money, *o sistema deve* retornar nova instância (sem mutação). *Quando* código usa `float32|float64` em cálculo financeiro, *o sistema deve* falhar no code review (lint custom).

**Critério de aceite**:
- `MoneyVO` em `internal/modules/shared/money/money.go`.
- Operações de divisão com basis points (1/10000) — nunca float.
- Regra de código: `grep -r "float" internal/modules/billing/` → zero ocorrências (CI check).
- Code Calisthenics #4 (no primitive obsession): nenhum `int64` raw em API billing.

**Fonte**: Code Calisthenics #4; GR-032 legado.

---

### BIL-047 — PIX via Pagar.me — backlog M2+

**Prioridade**: W
**EARS**: *Quando* decisão de habilitar PIX for tomada, *o sistema deve* adicionar adapter `PagarMeProvider implements IPaymentGateway` preservando contrato. Nova ADR obrigatória.

**Critério de aceite**:
- Interface `IPaymentGateway` absorve PIX sem alterar contratos.
- Webhook Pagar.me usa mesma tabela `webhook_events` com `provider='pagarme'`.

**Fonte**: D-056 Fase 8; backlog M2+.

---

## 10. PII em logs, LGPD, audit trail (BIL-048..BIL-052)

### BIL-048 — PII em logs proibido

**Prioridade**: M
**EARS**: *Quando* código em `billing/` loga evento, *o sistema deve* **nunca** emitir: email plain-text, número de cartão (PAN), CVV, CPF, nome completo, endereço. *Quando* necessário referenciar user → log `user_id UUID` apenas. *Quando* necessário email → HMAC-keyed hash via `HMAC_PII_KEY` (ADR-0028).

**Critério de aceite**:
- Lint rule custom: `zerolog.Event.Str("email", ...)` em billing/ → CI fail.
- Masking helper: `pii.MaskEmail("joao@x.com")` → `"j***@x.com"` — único método permitido se email necessário em UI.
- Teste: log output grep por regex `[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+` → zero matches em staging/prod.

**Fonte**: ADR-0028 (LGPD keyed HMAC); BeyondCorp §10.

---

### BIL-049 — Audit trail para mudança de plano

**Prioridade**: M
**EARS**: *Quando* subscription muda (`plan_id`, `status`), *o sistema deve* persistir em `audit_trail {id, actor_id, actor_role, entity_type, entity_id, action, diff_before JSONB, diff_after JSONB, metadata JSONB, occurred_at}` dentro da mesma UoW.

**Critério de aceite**:
- Append-only (invariante AUDIT-001).
- `actor_id` pode ser `system` (webhook Stripe) — registrado como tal.
- Retention: 10 anos (LGPD + CVM-like).

**Fonte**: GR-011 legado; ADR-0013 (timeline immutable pattern aplicado a audit).

---

### BIL-050 — LGPD log para dados pessoais

**Prioridade**: M
**EARS**: *Quando* operação billing lê/escreve/deleta dado pessoal (email, CPF, cartão), *o sistema deve* adicionalmente registrar em `lgpd_logs {id, user_id, data_subject_id, operation, purpose, legal_basis, occurred_at, retention_until}`. *Quando* user solicita portability/forget via `GET /me/data-export` ou `POST /me/data-delete`, *o sistema deve* coletar todos `lgpd_logs` relacionados e incluir no relatório.

**Critério de aceite**:
- `legal_basis ∈ {consent, contract, legal_obligation, vital_interest, public_task, legitimate_interest}`.
- `purpose` taxonomia fixa: `billing, fraud_prevention, customer_support, tax_reporting, marketing_optout`.
- Teste: refund → lgpd_logs registra `operation=refund_issued, legal_basis=contract`.

**Fonte**: ADR-0028; LGPD Art. 37.

---

### BIL-051 — Direito ao esquecimento (LGPD)

**Prioridade**: S
**EARS**: *Quando* user emite `POST /me/data-delete`, *o sistema deve* (a) cancelar subscription ativa via Stripe, (b) anonimizar PII em `subscriptions` (email, nome) substituindo por hash, (c) manter `user_id` + `subscription_id` (para integridade referencial audit), (d) registrar em `lgpd_logs` com `operation=erasure`.

**Critério de aceite**:
- **Não deleta rows** — anonimiza para preservar integridade financeira/fiscal.
- Stripe customer marcado `metadata.deleted=true` via `customer.update`.
- Export fiscal (NF-e futura) continua funcionando com `user_id` anônimo.

**Fonte**: LGPD Art. 18 VI; ADR-0028.

---

### BIL-052 — Data export portability

**Prioridade**: S
**EARS**: *Quando* user emite `GET /me/data-export`, *o sistema deve* retornar em JSON: subscriptions history, invoices (Stripe URLs), quota usage, audit_trail entries onde user é `actor`/`subject`.

**Critério de aceite**:
- Formato JSON estruturado documentado em OpenAPI.
- Rate limit 1/dia por user (prevent abuse).
- Geração assíncrona via job + link de download assinado (TTL 24h).

**Fonte**: LGPD Art. 18 V; GR-LGPD-003.

---

## 11. Relatórios financeiros admin (BIL-053..BIL-055)

### BIL-053 — MRR, ARR, churn, conversion

**Prioridade**: S
**EARS**: *Quando* admin acessa painel financeiro, *o sistema deve* exibir:

- **MRR** (Monthly Recurring Revenue) = Σ `subscriptions.active.plan.price_cents` normalizado mensal
- **ARR** = MRR × 12
- **Churn rate** = `(canceled_this_month / active_start_of_month) × 100%`
- **Active subscriptions por plano** (breakdown `tier`)
- **Trial → paid conversion rate** = `(trials_converted / trials_ended_this_month) × 100%`
- **Past-due count** (signaling problems)

**Critério de aceite**:
- Dados agregados via materialized view `billing_metrics_daily` refreshed 4h.
- Gráficos 12 meses rolling window.
- Export CSV.

**Fonte**: legado `novo escopo-7.md` §4.6.1; SaaS metrics standard.

---

### BIL-054 — Inadimplência control

**Prioridade**: S
**EARS**: *Quando* subscription em `past_due` > 7d, *o sistema deve* aparecer em "Controle de Inadimplentes" admin dashboard com ações: (a) reenviar invoice, (b) ligar para customer support manual, (c) cancelar manualmente (com 4-eyes).

**Critério de aceite**:
- Lista paginada com filtros (tier, age, amount).
- Bulk actions desabilitadas (evita erro humano massivo).

**Fonte**: D-061 Fase 8 (admin role); legado admin dashboard.

---

### BIL-055 — Export fiscal (backlog)

**Prioridade**: W
**EARS**: *Quando* integração fiscal BR for priorizada, *o sistema deve* suportar export NFS-e via provider específico. Fora de M1-M3.

**Fonte**: portfolio anti-catálogo.

---

## 12. Invariantes de billing (consolidadas)

1. **Append-only `subscriptions`**: novo plano = novo row; histórico preservado.
2. **Trial único por user**: `trial_ends_at IS NOT NULL` só na primeira subscription.
3. **`plan_tier ∈ {trial, base, plus, pro}`**: único enum de plano no banco/código (ADR-0032).
4. **Slugs compostos por persona proibidos** em código, banco, UI, docs (`syndic_n1`, `enterprise_plus`, etc.).
5. **Money em int64 centavos**: nunca float; `MoneyVO` imutável.
6. **Webhook idempotência**: UNIQUE `(provider, event_id)`; no-op em conflict.
7. **HMAC verify antes de parse** em webhooks.
8. **Soft-block imediato** (sem grace period) em expiração/cancelamento.
9. **PII em logs proibido**: email/CPF/cartão mascarados ou omitidos.
10. **Audit trail + LGPD log** para toda mudança de plano que afeta PII.
11. **Admin bypass** em ABAC curto-circuita matriz (`role=admin → allow` exceto `admin_scoped=false`).
12. **Mudança de role preserva `plan_id`** — quotas recalculadas pela matriz nova.
13. **4-eyes approval** obrigatório para refund, price change, quota override.
14. **ABAC lookup ordem fixa**: admin_bypass → matrix → plan.features → quota → tenant → rollout → allow.
15. **Morador Base Connect Me = 2/ano** (D-094 Fase 11 revoga D-058/D-079).
16. **Lives = admin + Empresa Pro + agência delegada** (D-097 Fase 11 revoga D-063/D-076 admin-only).

---

## 13. Dependências e pendências

### Dependências

- ADR-0004 (Stripe payment gateway)
- ADR-0012 (ABAC Redis cache)
- ADR-0027 (webhook idempotency DB)
- ADR-0028 (LGPD keyed HMAC)
- ADR-0032 (global plans Stripe-style)
- Interface `IPaymentGateway` (D-056, D-071 Fase 7 — agnosticismo provedor)

### Pendências

- ⚠️ **PENDÊNCIA BIL-003**: preços exatos de Plus/Pro em R$ — stakeholder pre-Sprint 10.
- ⚠️ **PENDÊNCIA BIL-018**: lista exaustiva `event.type` Stripe — dual-check vs API 2026 latest antes do deploy.
- ⚠️ **PENDÊNCIA BIL-047**: ADR PIX/Pagar.me — abrir se backlog M2 aprovar.
- ⚠️ **PENDÊNCIA BIL-013**: configuração exata Smart Retries (nº tentativas, janela) — confirmar com finance.
- ⚠️ **PENDÊNCIA BIL-035**: revisitar D-076 Fase 7 em M2 — LMS poderia abrir lives para empresas?

---

## 14. Referências

- ADR-0032 `02-architecture/adr/0032-global-plans-stripe-style.md` — planos globais (fonte canônica)
- ADR-0004 Stripe; ADR-0027 webhook idempotency; ADR-0028 LGPD HMAC; ADR-0012 ABAC cache
- [[../matrix-functional]] — matriz funcional reescrita (Trial/Base/Plus/Pro universais)
- [[../../00-product/sub-produtos/_moc]] — 11 sub-produtos com features catalogadas
- STATE D-056..D-069 (Fase 8) — D-058 (Morador Base 0/ano REVOGADA por D-094), D-063 (lives admin-only REVOGADA por D-097), D-059 (cupons ativos)
- STATE D-070..D-076 (Fase 7) — D-071 agnosticismo; D-073 agência sub-produto; D-076 lives admin REVOGADA por D-097
- STATE D-094..D-118 (Fase 11+) — D-094 Morador Base 2/ano, D-097 Lives expandidas, D-103 Score duplo, D-106 M1 = 2026-05-18
- Stripe Docs: Products, Prices, Subscriptions, Webhooks, Proration, Customer Portal
- [[../global]] — requirements globais (GR-008 HMAC, GR-009 idempotency, GR-011 audit, GR-027 trial, GR-032 money)
- [[../registration-data]] — dados pré-cadastro por persona (sem slug)
- [[_moc]]
