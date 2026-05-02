---
title: Billing — Backend Requirements (Go)
type: requirement
tags: [master-sindico, backend, go, billing, stripe, subscription, quotas, saga, feature-req]
project: master-sindico
stack: backend
bounded_context: billing
milestone: m1
status: delivered
created: 2026-04-24
---

# Billing — Backend Requirements (Go)

## Escopo do bounded context

Módulo Go `internal/modules/billing/` responsável por:

- **Catálogo global de planos** Stripe-style (ADR-0032): `plan_tier ∈ {trial, base, plus, pro}` universal, ortogonal à role.
- **Subscription lifecycle** append-only com state machine (trial → active → past_due → canceled → expired).
- **Trial auto-start persona-aware** via `ITrialChecker` consumido por identity.
- **CheckoutSubscriptionSaga** (Stripe + DB + cache invalidation, com compensação).
- **Webhook Stripe** HMAC verify + UNIQUE `(provider, event_id)` idempotência.
- **Quotas** (Connect Me, vídeos, condomínios, cupons) em DB (source of truth) + Redis cache.
- **Feature flags** via `plan.features JSONB` canônicos namespaced.
- Refunds 4-eyes, disputes, Customer Portal proxy.
- **Money VO** `int64 centavos` imutável; zero float em billing/.
- LGPD: PII nunca em logs; lgpd_logs para purpose=billing/fraud_prevention/tax_reporting.

**Não é responsabilidade**: ABAC engine em si (identity expõe); notificações (usa `IEmailProvider` via XD-015); pagamento PIX (backlog BIL-047).

## Status atual (vault canônico)

- **Sprint 3 + Fase 8 legado** entregou: catálogo global (ADR-0032), Stripe integration, webhook HMAC + idempotency (ADR-0027), quotas (Connect Me, vídeos, condomínios, cupons), feature flags, Money VO.
- **A-###** fechados: A-008..A-015 (Stripe base), A-DC-PEN-016 (webhook HMAC antes de parse), A-025 fix UNIQUE (aplicado a vote mas padrão nasceu aqui).
- **A-FASE8-FU-001 aberto**: `FeatureUsage` com quota snapshot no momento do decremento (audit + replay).
- **D-###**: ADR-0004 (Stripe gateway), ADR-0012 (ABAC Redis cache), ADR-0027 (webhook idempotency), ADR-0032 (global plans Stripe-style), D-056..D-069 (Fase 8 canônica), D-070..D-082 (Fase 7 renumerada).

## Agregados (domain)

- [[../../../01-domain/aggregates/Plan]] — catálogo global; `tier` enum; `features`, `quotas` JSONB; `stripe_price_id`.
- [[../../../01-domain/aggregates/Subscription]] — state machine; append-only; `stripe_subscription_id`; UNIQUE `(user_id) WHERE status IN ('trial','active')`.
- [[../../../01-domain/aggregates/FeatureUsage]] — `{user_id, feature_key, period_key, used, limit, snapshot_at}` (A-FASE8-FU-001).
- `Refund` — 4-eyes `{requester_id, approver_id, stripe_refund_id}`.
- `Dispute` — mirror Stripe `charge.dispute.*`.
- `WebhookEvent` — `{provider, event_id, event_type, payload_hash, status, last_error}` UNIQUE.
- `SubscriptionChange` — append-only, `{old_plan_id, new_plan_id, changed_at, prorated_amount_cents}`.
- `PlanApproval` — 4-eyes price change.
- `QuotaOverride` — admin manual.

## Entidades e Value Objects

| Tipo | Elemento | Invariante chave |
|---|---|---|
| Entity | `Plan` | `tier` enum; `currency='BRL'` CHECK; slug composto por persona proibido (lint) |
| Entity | `Subscription` | append-only (novo plano = novo row); transições válidas via `Subscription.TransitionTo` |
| Entity | `FeatureUsage` | decremento transacional (mesma UoW do aggregate consumidor) |
| VO | `MoneyVO` | `{amount int64, currency Currency}` imutável; `Add/Sub/Mul/Pct`; zero float |
| VO | `PlanTier` | enum `{trial, base, plus, pro}` — único enum plano |
| VO | `FeatureKey` | string namespaced (ex: `content.video.publish`) |
| VO | `PeriodKey` | granular: `yearly`, `monthly_2026_05`, `daily_...` |

## Invariantes (INV-BIL-###)

1. **INV-BIL-001**: `subscriptions` append-only — novo plano = novo row.
2. **INV-BIL-002**: Trial único por user — `trial_ends_at IS NOT NULL` só na primeira subscription.
3. **INV-BIL-003**: `plan_tier ∈ {trial, base, plus, pro}` único enum (ADR-0032).
4. **INV-BIL-004**: Slugs compostos por persona proibidos em código/DB/UI (lint).
5. **INV-BIL-005**: Money em `int64` centavos; nunca float.
6. **INV-BIL-006**: Webhook idempotência — UNIQUE `(provider, event_id)`.
7. **INV-BIL-007**: HMAC verify ANTES de parse em webhooks (evita vazar body em logs).
8. **INV-BIL-008**: Soft-block imediato (sem grace period) em expiração/cancelamento (D-067/D-069 Fase 8).
9. **INV-BIL-009**: PII (email/CPF/cartão) nunca em logs de billing/.
10. **INV-BIL-010**: Audit trail + LGPD log para toda mudança que afeta PII.
11. **INV-BIL-011**: Admin bypass em ABAC curto-circuita matriz (exceto `admin_scoped=false`).
12. **INV-BIL-012**: Mudança de role preserva `plan_id` (BIL-004).
13. **INV-BIL-013**: 4-eyes approval para refund, price change, quota override.
14. **INV-BIL-014**: Morador Base Connect Me = 2/ano (D-094 Fase 11 revoga D-058/D-079).
15. **INV-BIL-015**: Lives = admin + Empresa Pro + agência delegada (D-097 Fase 11 revoga D-063/D-076 admin-only).

Referência: [[../../../04-requirements/functional/billing#12-invariantes-de-billing-consolidadas]].

## Eventos de domínio

Publica:

- `billing.plan.created|updated` — invalida cache Redis `plans:catalog:v1`.
- `billing.subscription.created` → identity atualiza onboarding flow.
- `billing.subscription.changed {user_id, old_plan_id, new_plan_id, old_status, new_status}` → identity invalida ABAC cache; commercial recalcula quotas.
- `billing.upgrade_required {user_id, missing_tier, missing_actions}` (BIL-004) → UI renderiza CTA.
- `billing.refund.issued` — audit + lgpd_logs.
- `billing.quota.overridden` — admin manual.
- `billing.subscription.paused|unpaused`.
- `billing.trial.expiring_soon {user_id, hours_left}` (cron T-24h).

Consome:

- `identity.user.created` → `ITrialChecker.StartTrial(user_id, role)` auto-start.
- `identity.user.role_changed` → registrar `role_changes` + emitir `upgrade_required` se aplicável (sem Stripe call).
- `identity.deletion.requested` → pre-cancelar Stripe subscription; aguardar `customer.subscription.deleted`.

## Requirements funcionais (FR-BE-BIL-###)

- **FR-BE-BIL-001** (BIL-001) — *Quando* `POST /admin/plans`, *o backend deve* validar `tier ∈ {trial, base, plus, pro}`, `currency='BRL'`, persistir em `plans` JSONB features/quotas; cache Redis `plans:catalog:v1` TTL 1h.
- **FR-BE-BIL-002** (BIL-002) — *Quando* `PATCH /admin/plans/{id}` afeta `price_cents|tier|quotas`, *o backend deve* exigir **4-eyes** via `plan_approvals` (requester_id ≠ approver_id); audit trail `audit_trail {diff_before, diff_after}`.
- **FR-BE-BIL-003** (BIL-003) — *Quando* migration `00600_seed_plans.up.sql` roda, *o backend deve* popular idempotente (ON CONFLICT DO NOTHING) os planos-base: Trial, Base (resident 0), Essencial (resident 990), Plus, Pro; `stripe_price_id` nullable em trial/base-gratuito.
- **FR-BE-BIL-004** (BIL-004) — *Quando* `identity.user.role_changed`, *o backend deve* preservar `subscription.plan_id`, registrar `role_changes`, emitir `billing.upgrade_required` se role nova exige tier não coberto.
- **FR-BE-BIL-005** (BIL-006 + BIL-007) — *Quando* webhook Stripe triggera transição, *o state machine `Subscription.TransitionTo(newStatus)` deve* aceitar apenas whitelist (trial→active|expired, active→past_due|canceled|paused, etc.); ilegal → `ErrIllegalTransition`.
- **FR-BE-BIL-006** (BIL-008) — *Quando* `identity.user.created` chega, *o listener deve* chamar `ITrialChecker.StartTrial(user_id, role)`: syndic→15d, enterprise→7d, local_company→30d, resident→sem trial (Plano Base direto).
- **FR-BE-BIL-007** (BIL-009) — *Enquanto* `subscription.status=trial`, *o middleware deve* consultar matriz `(role, trial, action)`; se `trial_allowed=false` → `402 PAYMENT_REQUIRED code=trial_feature_blocked` com `upgrade_url`.
- **FR-BE-BIL-008** (BIL-010 + BIL-011) — *Quando* cron horário roda, *o job deve* (a) flaggear `trial_warning=true` se `trial_ends_at - now() ≤ 24h`, (b) transicionar `trial → expired` se expirou, (c) invalidar `ms:abac:{user_id}:*`.
- **FR-BE-BIL-009** (BIL-014) — *Quando* user com subscription prévia tenta novo checkout, *o backend deve* skip trial (`SELECT COUNT(*) FROM subscriptions WHERE user_id=$1 > 0`); PBT fuzz confirma.
- **FR-BE-BIL-010** (BIL-016 + INV-BIL-007) — *Quando* `POST /webhooks/stripe`, *o handler deve* ler `io.ReadAll(r.Body)`, chamar `webhook.ConstructEvent` com `STRIPE_WEBHOOK_SECRET` **antes** de qualquer parse; falha → `400 INVALID_SIGNATURE` sem log do body; tolerance 5min.
- **FR-BE-BIL-011** (BIL-017) — *Quando* event passa HMAC, *o backend deve* `INSERT INTO webhook_events ON CONFLICT (provider, event_id) DO NOTHING RETURNING id`; conflict → `200 OK` no-op; insert OK → processar dentro da mesma UoW marcando `status=processed`.
- **FR-BE-BIL-012** (BIL-018) — *Quando* event in whitelist (checkout.session.completed, customer.subscription.created|updated|deleted, invoice.payment_succeeded|failed, charge.dispute.created, charge.refunded, customer.updated), *o dispatcher deve* rotear via `map[eventType]Handler`; outros → log info + 200.
- **FR-BE-BIL-013** (BIL-019) — *Quando* webhook modifica `plan_id|status`, *o handler deve* (mesma UoW) invalidar `ms:abac:{user_id}:*` + publicar `billing.subscription.changed` via `IEventBus`.
- **FR-BE-BIL-014** (BIL-022) — *Quando* `POST /billing/checkout {plan_id}`, *a CheckoutSubscriptionSaga deve* executar: (1) validate(plan.active, no conflict sub, rate limit 10/h), (2) `IPaymentGateway.CreateCheckoutSession(user_id, plan_id, metadata)`, (3) retornar `checkout_url`; compensação: cleanup Stripe session se falha downstream.
- **FR-BE-BIL-015** (BIL-023) — *Quando* `POST /billing/portal`, *o backend deve* chamar `IPaymentGateway.CreateBillingPortalSession(user_id, return_url)`; lazy-create `stripe_customer_id` se ausente.
- **FR-BE-BIL-016** (BIL-024) — *Quando* `GET /me/invoices?cursor&limit=20`, *o backend deve* chamar `IPaymentGateway.ListInvoices` sem persistir fatura local (Stripe = source of truth).
- **FR-BE-BIL-017** (BIL-025 + BIL-026) — *Quando* user muda tier via Customer Portal (`proration_behavior=create_prorations`), *o webhook handler deve* UPSERT subscription preservando histórico em `subscription_changes` append-only; ABAC cache invalidado imediatamente.
- **FR-BE-BIL-018** (BIL-027 + BIL-028) — *Quando* `POST /admin/subscriptions/{id}/refund`, *o backend deve* criar pending refund; `POST /admin/refunds/{pending}/approve` com `approver_id != requester_id` → chamar `IPaymentGateway.CreateRefund`; audit + `lgpd_logs {operation=refund_issued, legal_basis=contract}`.
- **FR-BE-BIL-019** (BIL-029) — *Quando* webhook `charge.dispute.created`, *o handler deve* persistir `disputes {evidence_due_by}` + disparar Sentry alert tag `dispute:stripe` + email admin@.
- **FR-BE-BIL-020** (BIL-030) — *Quando* Connect Me emit (COM-006), *o commercial deve* chamar `IQuotaConsumer.Consume(ctx, user_id, "commercial.connect_me", period_yearly)`; billing decrementa DB + Redis (mesma UoW do aggregate ConnectMeRequest); esgotado → `ErrQuotaExceeded` → `403`.
- **FR-BE-BIL-021** (BIL-031) — *Quando* content publica vídeo (CNT-001), *o content deve* chamar `IQuotaConsumer.Consume(ctx, user_id, "content.video.publish", period_monthly)`; matrix `(role, tier)` → limite; marketing consome da empresa cliente via delegation.
- **FR-BE-BIL-022** (BIL-032) — *Quando* syndic tenta 16º condomínio, *o institutional deve* chamar `IQuotaCheck.Check(user_id, "institutional.condominium", hard_limit=15 se tier≥plus, 1 se tier=base)`; esgotado → `403 QUOTA_EXCEEDED code=max_condominiums_reached`.
- **FR-BE-BIL-023** (BIL-033) — *Quando* morador Plus tenta vídeo-currículo, *o backend deve* `UNIQUE (user_id) WHERE type='curriculum'`; se existe há < 90d → `409 VIDEO_LOCKED`.
- **FR-BE-BIL-024** (BIL-034) — *Quando* morador Plus tenta cupom em parceiro X, *o backend deve* `SELECT NOT EXISTS (FROM coupon_generations WHERE user_id=$1 AND partner_id=$2 AND generated_at > now() - interval '4 hours')`; falha → `429 RATE_LIMITED code=coupon_cooldown` + `Retry-After`.
- **FR-BE-BIL-025** (BIL-035) — *Quando* user tenta `action=live.create`, *o ABAC deve* permitir se: `role=admin` (qualquer tier) OU (`role=enterprise` AND `plan.tier=pro`) OU (`role=marketing` AND tem `delegation_grant` ativo da empresa Pro). Demais combinações → `403 FORBIDDEN code=live_not_authorized`. **D-097 Fase 11 revoga D-063/D-076** (admin-only no MVP).
- **FR-BE-BIL-026** (BIL-036) — *Quando* ABAC quota check, *o backend deve* consultar Redis `ms:quota:{user_id}:{feature_key}:{period_key}`; miss → DB `feature_usages` + repovoa cache com TTL = `period_end - now()`.
- **FR-BE-BIL-027** (BIL-037) — *Quando* `GET /me/quotas`, *o backend deve* retornar todas quotas aplicáveis à `(role, tier)` do user; `limit=∞` → `"limit": null`.
- **FR-BE-BIL-028** (BIL-038) — *Quando* admin `POST /admin/users/{id}/quota-override`, *o backend deve* persistir `quota_overrides {limit_override, expires_at}`; ABAC prefere override se ativo; audit.
- **FR-BE-BIL-029** (BIL-041) — *Quando* ABAC resolve `action`, *o engine deve* consultar `plan.features[action_namespaced]`; ausente/false → `402 PLAN_REQUIRED`.
- **FR-BE-BIL-030** (BIL-043) — *Quando* ABAC avalia, *o backend deve* seguir ordem fixa: admin_bypass → matrix → plan.features → quota → tenant → rollout → allow; log reason estruturado.
- **FR-BE-BIL-031** (BIL-046) — *Quando* código em `billing/` opera valor, *o backend deve* usar `MoneyVO`; CI grep `float` em `internal/modules/billing/` → zero (fail pipe).
- **FR-BE-BIL-032** (BIL-048 + BIL-049 + BIL-050) — *Quando* billing loga, *o logger deve* nunca emitir email/CPF/PAN/CVV; lint custom em CI rejeita `zerolog.Event.Str("email", ...)`; `audit_trail` + `lgpd_logs` append-only retidos 10 anos.

## Endpoints REST expostos

| Método | Path | Auth | Descrição | Status |
|---|---|---|---|---|
| GET | /api/v1/plans | Bearer opcional | Catálogo global (cache) | ✅ |
| POST | /api/v1/billing/checkout | Bearer + ABAC | Stripe Checkout session | ✅ |
| POST | /api/v1/billing/portal | Bearer | Customer Portal session | ✅ |
| GET | /api/v1/me/subscription | Bearer | Detalhe com `trial_warning` flag | ✅ |
| GET | /api/v1/me/invoices | Bearer | Lista via Stripe ListInvoices | 🟡 |
| GET | /api/v1/me/quotas | Bearer | Quotas aplicáveis | ✅ |
| POST | /webhooks/stripe | público + HMAC | Webhook handler | ✅ |
| POST | /admin/v1/plans | Bearer + admin | Criar plano (4-eyes na price change) | ✅ |
| PATCH | /admin/v1/plans/:id | Bearer + admin + 4-eyes | Editar | ✅ |
| DELETE | /admin/v1/plans/:id | Bearer + admin | Desativar | 🟡 |
| POST | /admin/v1/subscriptions/:id/pause | Bearer + admin | Pause (BIL-015) | ⏳ |
| POST | /admin/v1/subscriptions/:id/refund | Bearer + admin | Request refund (pending) | ✅ |
| POST | /admin/v1/refunds/:pending_id/approve | Bearer + admin (different) | 4-eyes approve | ✅ |
| POST | /admin/v1/users/:id/quota-override | Bearer + admin | Override quota | ⏳ |
| GET | /admin/v1/disputes | Bearer + admin | Lista disputes ativas | 🟡 |
| GET | /admin/v1/metrics/financial | Bearer + admin | MRR/ARR/churn (BIL-053) | ⏳ |

## Integração com outros BCs

**Expõe** (via `internal/shared/types`):

- `ITrialChecker` — `StartTrial(ctx, user_id, role) error`, `IsTrialActive(ctx, user_id) bool`.
- `IQuotaConsumer` — `Consume(ctx, user_id, feature_key, period_key) error`.
- `IQuotaCheck` — `Check(ctx, user_id, feature_key) QuotaInfo`.
- `IFeatureFlagChecker` — `IsEnabled(ctx, user_id, feature_key) bool`.
- `IPaymentGateway` (XD-012) — interface infrastructure.

**Consome**:

- `IEventBus` (cross-domain).
- `IAuditLogger` (DT-010).
- `IEmailProvider` (XD-015) — não direto (Stripe faz emails de invoice); só para notificações admin de dispute.

**Publica eventos**: `billing.plan.*`, `billing.subscription.*`, `billing.refund.*`, `billing.quota.*`, `billing.trial.*`, `billing.upgrade_required`.

**Consome eventos**: `identity.user.created`, `identity.user.role_changed`, `identity.deletion.requested`.

## Persistência

**Tabelas principais**:

- `plans` — JSONB `features`, `quotas`; CHECK currency='BRL'.
- `subscriptions` — append-only; UNIQUE `(user_id) WHERE status IN ('trial','active')`; FK `plan_id`.
- `subscription_changes` — append-only histórico prorata.
- `feature_usages` — `(user_id, feature_key, period_key) PK`; `snapshot_at` (A-FASE8-FU-001).
- `quota_overrides` — admin manual + `expires_at`.
- `webhook_events` — UNIQUE `(provider, event_id)`; status enum.
- `refunds` — 4-eyes fields.
- `disputes` — mirror Stripe.
- `plan_approvals` — 4-eyes price change.
- `role_changes` — histórico role user.
- `coupon_generations` — lookup 4h cooldown (BIL-034).
- `lgpd_logs` — retenção 5 anos; purpose taxonomia fixa.
- `audit_trail` — append-only; 10 anos.
- `billing_metrics_daily` — materialized view (BIL-053).

**Migrations**: `00100..00299` billing.

**RLS default-on**: `subscriptions`, `feature_usages`, `quota_overrides`, `refunds` — `USING (user_id = current_setting('app.current_user_id')::uuid)`. `plans` sem RLS (público).

## Providers externos

- **Stripe** (`IPaymentGateway` — ADR-0004) — Checkout, Customer Portal, Subscriptions, Webhooks, Proration, Refunds, Invoices.
- **Pagar.me/PIX** (backlog W) — nova ADR se aprovado (BIL-047).
- **Resend/SES** (via `IEmailProvider`) — alerts admin dispute.

**Sagas**:

- `CheckoutSubscriptionSaga` (código `checkout_subscription_saga.go`) — steps: (1) validate, (2) `IPaymentGateway.CreateCheckoutSession`, (3) persist intent, (4) retornar URL. Compensação: cleanup Stripe session + delete intent.
- `AccountDeletionSaga` (ADR-0030) — billing participa cancelando Stripe sub pre-deletion.
- `RefundSaga` — 4-eyes request → approve → Stripe call → audit.

## Testes

**Coverage target**: Domain 90%, Application 80%, Infra 60%.

**PBT**:

- BIL-040: `∀ sequência de decrementos → used ≤ limit`; soma = used; reset no 1º do período; override sobrescreve limit sem mexer em used.
- State machine `Subscription.TransitionTo`: toda combinação (from,to) ∉ whitelist → `ErrIllegalTransition`.
- Money: `∀ a,b > 0 → Add(a,b) == Add(b,a)`; zero overflow int64.
- Webhook idempotency: 100 deliveries idênticas → 1 processamento.

**Integration** (testcontainers):

- Stripe webhook fixtures (Go `stripe-mock`) + Postgres + Redis.
- CheckoutSubscriptionSaga E2E com compensação induzida.
- Trial expiration cron rod rolling time.

## Segurança específica do BC

**LGPD**:

- `lgpd_logs` para toda operação que lê/escreve/deleta PII; `purpose ∈ {billing, fraud_prevention, customer_support, tax_reporting, marketing_optout}`.
- `legal_basis ∈ {consent, contract, legal_obligation, vital_interest, public_task, legitimate_interest}`.
- Direito ao esquecimento: anonimiza (não deleta) `subscriptions.email/name` → hash; `user_id` preservado pra integridade fiscal.

**ABAC actions canônicos**:

- `billing.plan.create|update|delete` (admin).
- `billing.checkout.initiate` (todos autenticados).
- `billing.refund.request|approve` (admin + 4-eyes).
- `billing.quota.override` (admin).
- `billing.admin.metrics.read` (admin).

**Rate limits**:

- `/api/v1/billing/checkout` — 10/h por user.
- `/api/v1/me/data-export` — 1/24h.
- `/webhooks/stripe` — sem rate limit (Stripe controla); mas Sentry alert se > 10k/min.

## Decisões e dívidas

**D-###**:

- ADR-0004 (Stripe gateway) — fechado.
- ADR-0012 (ABAC Redis cache) — fechado.
- ADR-0027 (webhook idempotency DB) — fechado.
- ADR-0028 (LGPD keyed HMAC) — fechado.
- ADR-0032 (global plans Stripe-style) — fechado; **invalida** todo slug composto por persona.
- D-056 (agnosticismo provider) — fechado.
- D-058 (Morador Base Connect Me 0/ano) — **REVOGADA** por D-094 Fase 11 (= 2/ano).
- D-059 (Sistema Cupons ATIVO) — fechado Fase 8.
- D-063 (Lives admin-only) — fechado Fase 8.
- D-067 (abolir slugs N1/N2/N3 preservando trial duration) — fechado.

**DT-###**:

- **A-FASE8-FU-001 aberto**: FeatureUsage precisa `snapshot_at` + `quota_snapshot` para auditoria + replay de quota alteration history.
- Pendências BIL-003 (preços Plus/Pro R$), BIL-013 (Smart Retries config), BIL-035 (lives expandidas via D-097 — implementar matriz admin/Empresa Pro/agência delegada), BIL-047 (ADR migration Stripe → Mercado Pago — em avaliação 2026-04-27).

## Gaps e bloqueadores Sprint 10 (M1)

1. **FeatureUsage snapshot** (A-FASE8-FU-001) — audit/replay de histórico de limites alterados.
2. **Preços finais Plus/Pro** (BIL-003) — stakeholder bloqueia seed definitivo.
3. **Quota override UI/endpoint** (BIL-038) — ⏳ endpoint básico; falta audit trail UI admin.
4. **Financial metrics dashboard** (BIL-053) — ⏳ materialized view não implementada; KPIs manuais até então.
5. **Disputes management admin** (BIL-029) — 🟡 webhook persiste; UI admin e escalation rules pendentes.

## Ligações

- [[../../../04-requirements/functional/billing]] — requirement canônico global
- [[../../../01-domain/aggregates/Plan]]
- [[../../../01-domain/aggregates/Subscription]]
- [[../../../01-domain/aggregates/FeatureUsage]]
- [[../../../01-domain/aggregates/_moc]]
- [[../../../02-architecture/_moc]]
- [[../../../11-audit/AUDIT]]
- [[../architecture]]
- [[../patterns]] — Saga pattern aplicado a CheckoutSubscriptionSaga
- [[../security]]
- [[../tasks]] — Sprint 3 + Fase 8
- Correspondente web: [[../../web/requirements/cms]] (billing UI)
- Correspondente mobile: a definir

## Links
- [[../../../_moc]]
- [[../../../CLAUDE]]
