---
title: Sprint 1 — Identity & Billing
version: 1.0
status: in_progress
sprint: 1
period: "2026-04-07 to 2026-04-20"
tags: [sprint-1, identity, billing, abac, stripe]
---

# Sprint 1 — Identity & Billing

**Período**: 2026-04-07 → 2026-04-20
**Foco principal**: ABAC engine, Stripe adapter, Plans+Subscriptions, Trial logic, Quotas
**Objetivo de sprint**: Backend protegido por ABAC real + billing funcional end-to-end em sandbox Stripe.

## Status geral

| Seção | Status |
|---|---|
| Correções técnicas (1.0.x) | ✅ concluídas (12 itens) |
| BeyondCorp Middleware (1.1) | ✅ concluído — OIDC PKCE + introspection + sessions + 1-device + sync |
| ABAC engine (1.2) | ✅ concluída (2026-04-21) |
| Billing Stripe (1.3-1.6) | ✅ todas concluídas (2026-04-21) |
| Frontend Onboarding (1.7-1.13) | ⏳ pending (Dev2) |
| Mobile Flutter (1.14) | ⏳ pending (Dev2) |

---

## Task 1.2 — ABAC Engine

**Estado**: `pending`
**Sprint**: 1
**Depende de**: 1.1 (concluída)
**Estima**: 8-12h
**Responsável**: Dev1

### Discuss

- **Requisitos envolvidos**: Req 2 (Autorização ABAC) — ver `requirements.md` linhas 94-110 (ou `requirements/identity.md` quando migrado)
- **Design envolvido**: `design.md` seção "Middleware Stack (BeyondCorp)" linhas 146-169
- **Decisão em aberto**: D-001 em `.kiro/STATE.md` (cache TTL de introspection — não bloqueia esta task)
- **Lições do legacy aplicáveis**:
  - **A1** — ABAC não pode ser opcional; falhar no startup se matriz vazia
  - **A6** — resultado da avaliação deve ter `Reason` + logs estruturados de denial
  - **A4** — autorização no middleware, nunca no use case

### Plan

**Criar**:
- `internal/shared/middleware/abac_engine.go` — struct `ABACEngine` + método `Evaluate(ctx, action, resource)`
- `internal/shared/middleware/abac_rules.go` — matriz declarativa de regras (role × plan_tier × action)
- `internal/shared/middleware/require_permission.go` — factory `RequirePermission(action, resource) HandlerFunc`
- `internal/shared/types/abac_context.go` — struct `ABACContext` + `ABACResult`
- `internal/shared/middleware/abac_engine_test.go` — unit tests (Red-Green-Refactor)
- `internal/shared/middleware/abac_engine_pbt_test.go` — property-based tests (rapid)

**Modificar**:
- `internal/server/routes.go` — aplicar `RequirePermission` em rotas protegidas
- `internal/shared/config/config.go` — adicionar `ABACMatrixPath` (opcional) ou carregar inline
- `cmd/api/main.go` — instanciar `ABACEngine` e validar `config.Validate()` falha se matriz vazia

**Não tocar**:
- `modules/identity/` — ABAC vive em `shared/middleware/`
- `contracts/*` — ABAC é implementação, não contrato (a menos que a interface de `Context` precise de novo método)

**Docs oficiais a consultar via Context7**:
- `gin-gonic/gin` — topic: "middleware chaining + context.Set"
- `rs/zerolog` — topic: "structured logging with context"

**Subagentes?**: Não — task contida em 1 camada. Não precisa split.

### `<verify>`

Escrever testes **antes** do código (TDD):

- [ ] `TestABACEngine_Evaluate_GrantsWhenRoleAndPlanAllow` — síndico N3 pode criar comunicado
- [ ] `TestABACEngine_Evaluate_DeniesWhenRoleWrong` — morador **não** pode criar comunicado → `Granted=false, Reason="role_not_allowed"`
- [ ] `TestABACEngine_Evaluate_DeniesWhenPlanBelowRequired` — síndico N1 **não** pode publicar vídeos → `Reason="plan_below_required"`
- [ ] `TestABACEngine_Evaluate_DeniesWhenTenantMismatch` — síndico do condo A **não** pode editar timeline do condo B → `Reason="tenant_mismatch"`
- [ ] `TestABACEngine_FailsStartupIfMatrixEmpty` — `NewABACEngine(nil)` ou matriz vazia retorna erro
- [ ] **PBT**: `TestABACNeverGrantsAdminActionToNonAdmin` (rapid — 500 casos) — nenhum role não-admin consegue ação admin
- [ ] **PBT**: `TestABACTenantIsolationAlways` — 1000 casos, nunca concede cross-tenant sem grant explícito
- [ ] Coverage em `internal/shared/middleware/`: **≥ 90%**
- [ ] `go build ./...` limpo
- [ ] `go vet ./...` sem warning
- [ ] `gosec -severity high ./internal/shared/middleware/` sem findings
- [ ] Logs estruturados em cada denial (`user_id`, `tenant_id`, `role`, `plan_tier`, `action`, `resource`, `reason`)
- [ ] Response ao cliente: 403 com body `{"error":"FORBIDDEN","reason":"INSUFFICIENT_PERMISSION"}` — sem revelar detalhes internos

### Execute — ordem

1. **RED**: escrever `abac_engine_test.go` com os 5 casos unit + 2 PBT
2. Criar `abac_context.go` + `abac_rules.go` com matriz vazia
3. **GREEN**: implementar `ABACEngine.Evaluate` até todos os testes passarem
4. Popular matriz inicial em `abac_rules.go` com regras dos Reqs:
   - `timeline.create` → role=syndic (qualquer role_type exceto assistant) + plan_tier>=base
   - `timeline.read` → qualquer role do tenant
   - `subscription.cancel` → role=syndic (role_type=principal) OR role=admin
   - (demais conforme matriz funcional)
5. **REFACTOR**: extrair helpers, simplificar matriz se houver repetição
6. Criar `require_permission.go` — middleware factory que recebe `(action, resource)` e retorna `HandlerFunc`
7. Teste de integração — aplicar em rota real em `routes.go`, rodar request como diferentes roles
8. Validar logs estruturados de denial aparecem no stderr (`jq` para parsear JSON)

### Ship

- [ ] Todos os gates verdes
- [ ] PR com título: `identity: Add ABAC engine with denial logging`
- [ ] Release Notes inclui: `- Added ABAC engine enforcing role × plan_tier × action matrix; denials logged with user_id, tenant_id, reason`
- [ ] Se qualquer decisão nova surgir (ex: matriz incompleta): documentar em `.kiro/STATE.md`
- [ ] Marcar task `completed` neste arquivo

---

## Task 1.3 — Stripe Adapter

**Estado**: `pending`
**Sprint**: 1
**Depende de**: 1.2
**Estima**: 8h
**Responsável**: Dev1

### Discuss

- **Requisitos**: Req 4 (Billing), Req 4.1 (Trial por persona)
- **Design envolvido**: `design.md` seção "Adapter Layer" + novo `design/adapter-layer.md`
- **Antipatterns**: A11 (Drizzle beta instável — usar estável stripe-go v82+)

### Plan

**Criar**:
- `internal/modules/billing/domain/providers/i_payment_gateway.go` — interface com 6 métodos: `CreateCustomer`, `CreateSubscription`, `CancelSubscription`, `UpdateSubscription`, `VerifyWebhookSignature`, `ParseWebhookEvent`
- `internal/modules/billing/infrastructure/providers/stripe_gateway.go` — implementação com `stripe.Client`
- `internal/modules/billing/infrastructure/providers/stripe_gateway_test.go` — com mock do stripe SDK
- `internal/shared/config/config.go` — adicionar `StripeConfig` com `SecretKey` + `WebhookSecret`

**Modificar**:
- `cmd/api/main.go` — wire-up Stripe gateway

**Docs oficiais via Context7**:
- `stripe/stripe-go` (trust 8.9) — topic: "subscription creation with trial", "webhook ConstructEvent", "idempotency keys"

**MCP Stripe**: instalar plugin/MCP (descomentar em `.mcp.json`) no início desta task — útil para `search_documentation` e validar sandbox.

### `<verify>`

- [ ] `TestStripeGateway_CreateSubscription_WithTrial` — cria subscription com `trial_period_days` correto
- [ ] `TestStripeGateway_VerifyWebhookSignature_ValidSignature` — passa
- [ ] `TestStripeGateway_VerifyWebhookSignature_InvalidSignature` — retorna erro, **não** processa payload
- [ ] `TestStripeGateway_MapErrorsCorrectly` — `*stripe.CardError` → `apperrors.ErrValidation`, `*stripe.RateLimitError` → `apperrors.ErrRateLimited`
- [ ] Idempotency key gerada em toda escrita (`customer:{id}:{timestamp}`, etc.)
- [ ] Integration test com **Stripe sandbox** (sk_test_*): cria customer, cria subscription trial 7d, cancela — asserts no Stripe Dashboard
- [ ] Coverage em `infrastructure/providers/`: ≥ 85%
- [ ] `gosec` sem findings (especial atenção a G101 hardcoded secrets)

### Ship

- [ ] Gates verdes
- [ ] PR: `billing: Add Stripe gateway with webhook signature validation`
- [ ] Release Notes: `- Added Stripe payment gateway (sandbox only) with webhook signature verification, idempotency keys, and error mapping`

---

## Task 1.4 — Plans + Subscriptions + Webhook Handler

**Estado**: `pending`
**Sprint**: 1
**Depende de**: 1.3
**Estima**: 12h

### Discuss

- **Requisitos**: Req 4 (Billing), Req 4.1 (Trial), Req 6 (Acesso por plano)
- **Antipatterns críticos**: A8 (race em quota — usar `SELECT FOR UPDATE`), A7 (domain events para invalidar cache ABAC)
- **Transaction pattern**: Saga orquestrada para `Checkout` (Stripe + DB + cache invalidate) — ver `steering/transaction-patterns.md`

### Plan

**Criar**:
- Migration `00101_update_plans_seed.sql` — seed das linhas de planos com price_ids reais (via MCP Stripe para pegar os IDs no sandbox)
- `internal/modules/billing/domain/entities/subscription.go` — entidade com métodos `Activate`, `Cancel`, `MarkPastDue`, `LinkStripe`
- `internal/modules/billing/domain/events/subscription_activated.go`, `subscription_canceled.go`
- `internal/modules/billing/application/use_cases/checkout_subscription_saga.go` — Saga orquestrada
- `internal/modules/billing/application/use_cases/handle_stripe_webhook_use_case.go` — processa `customer.subscription.created`, `updated`, `deleted`, `invoice.paid`, `invoice.payment_failed`
- `internal/modules/billing/infrastructure/database/subscription_repository.go` — queries sqlc
- `internal/modules/billing/infrastructure/database/queries/subscription.sql`
- `internal/modules/billing/infrastructure/http/routes/subscription_handler.go`
- `internal/modules/billing/infrastructure/http/routes/webhook_handler.go`
- `internal/modules/billing/module.go`

**Modificar**:
- `cmd/api/main.go` — registrar módulo billing

### `<verify>`

- [ ] TDD: escrever testes primeiro para Saga (happy path + compensação)
- [ ] `TestCheckoutSubscriptionSaga_Success` — Stripe create + DB save + cache invalidate
- [ ] `TestCheckoutSubscriptionSaga_CompensateWhenDBFails` — se DB falha, Stripe subscription é cancelada
- [ ] `TestHandleStripeWebhook_InvalidSignature_Returns400` — **antes** de qualquer parse
- [ ] `TestHandleStripeWebhook_ValidEvent_UpdatesSubscription`
- [ ] Integration test com sandbox: fluxo completo checkout → webhook recebido → DB atualizado → `users.plan_id` atualizado
- [ ] Coverage ≥ 90% em `application/`, ≥ 85% em `infrastructure/`
- [ ] Cache Redis invalidado após update (teste verifica que `ms:auth:*` é deletado)

### Ship

- [ ] Gates verdes
- [ ] PR: `billing: Implement subscription checkout saga + Stripe webhook handler`
- [ ] Task bloqueia 1.5 e 1.6

---

## Task 1.5 — Trial Logic por Persona

**Estado**: `completed` (2026-04-21)
**Sprint**: 1
**Depende de**: 1.4
**Estima**: 6h

### Discuss

- **Requisitos**: Req 4.1 (Trial por Persona), Req 6 (Matriz Funcional)
- **Comportamento**: 15d síndico, 7d empresa, 30d parceiro. Durante trial: tudo visível, ações estratégicas bloqueadas com mensagem padrão.

### Plan

**Criar**:
- `internal/modules/billing/application/use_cases/check_feature_access_use_case.go` — query: `(userID, featureKey) → CanAccess`
- `internal/modules/billing/domain/services/trial_rules.go` — regras de bloqueio por persona durante trial
- Tests unit + PBT

**Modificar**:
- `abac_engine.go` (Task 1.2) — chamar `CheckFeatureAccess` para ações gated por plano/trial

### `<verify>`

- [ ] `TestTrialRules_SyndicTrial_BlocksCreateTimeline` — 15 dias → 16º dia bloqueia
- [ ] **PBT**: `TestTrialNeverAllowsAccessAfterExpiration` — 1000 datas aleatórias; após `trial_ends_at`, `CanAccess=false` sempre
- [ ] Integration test: criar user trial → avançar data no banco → assert feature bloqueada
- [ ] Coverage ≥ 90%

---

## Task 1.6 — Quotas + Feature Usage

**Estado**: `completed` (2026-04-21)
**Sprint**: 1
**Depende de**: 1.5
**Estima**: 8h

### Discuss

- **Requisitos**: Req 6 (Quotas Connect Me, uploads)
- **Antipattern crítico**: **A8** — race condition em `incrementUsage`. **Obrigatório** usar `SELECT ... FOR UPDATE` dentro de UnitOfWork.
- **Reset**: anual para Connect Me (1 de janeiro), mensal para uploads (1 do mês).

### Plan

**Criar**:
- Migration `00102_create_feature_usages.sql` — tabela com `user_id, feature_key, period_start, period_end, used, limit`
- `internal/modules/billing/domain/entities/feature_usage.go` — com métodos `Consume()`, `HasRemaining()`
- `internal/modules/billing/infrastructure/database/feature_usage_repository.go` — query `FindByUserIDForUpdate` com `FOR UPDATE`
- `internal/modules/billing/application/use_cases/consume_quota_use_case.go` — dentro de UoW
- Redis cache `ms:quota:{userID}:{feature}:{period}` para reads rápidos; source of truth no Postgres

### `<verify>`

- [ ] `TestConsumeQuota_DoesNotExceedLimit` — quando `used == limit`, retorna `ErrBusiness`
- [ ] `TestConsumeQuota_ConcurrencyControl` — 10 goroutines tentando consumir no limite máximo; exatamente `limit` sucessos, resto falhas
- [ ] **PBT**: `TestQuotaNeverGoesNegative` — nunca `used > limit`; nunca `used < 0`
- [ ] Integration test: comportamento real com Postgres `FOR UPDATE` em múltiplas transações concorrentes
- [ ] Reset automático via cron job noturno: em 1 de janeiro `period=2026` zera → `period=2027`
- [ ] Coverage ≥ 90%

---

## Frontend + Mobile (Dev2) — tasks 1.7 a 1.14

Ver `tasks.md` monolítico (ainda canônico para frontend até migração). Migrará para `tasks/sprint-1-frontend.md` quando apropriado.

## [Background] 1.15 — OpenSearch sync job

Deferred para Sprint 5 junto com setup OpenSearch completo (Req 31).

---

## Notas pós-sprint (preencher ao fechar)

- **O que deu certo**:
- **O que falhou**:
- **Bloqueadores encontrados**:
- **Débitos criados** (registrados em `.kiro/STATE.md`):
- **Próximos passos** (Sprint 2):
