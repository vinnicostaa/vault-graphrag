# Sprint 1 — Gap Analysis (Atualizado 18/02/2026)

## Visão Geral

| Módulo | Status | Progresso |
|--------|--------|-----------|
| Auth | ✅ Completo | 13 use-cases, rotas, DI |
| Onboarding | ✅ Completo | `CreateOnboardingUseCase`, rota POST |
| Profile | ✅ Completo | `GetProfileUseCase` + `UpdateProfileUseCase`, rotas GET/PATCH, entidades com `update()` |
| User | ✅ Completo | `GetProfileUseCase` (user), rota |
| Billing (Infra) | 🟡 Parcial | 9 repos, Stripe provider, webhook handler — **zero use-cases** |
| Permissões ABAC | 🔴 Ausente | `PermissionService` comentado, `plan-limits.config.ts` 100% comentado |
| Motor de Vídeo | 🔴 Ausente | Schemas de DB existem (`videos`, `video-comments`, `video-likes`, `video-views`), módulo não existe |
| Motor de Busca | 🔴 Ausente | Schemas de DB existem (`buildings`, `categories`), módulo não existe |
| Connect Me | 🔴 Ausente | Nada existe |

---

## O que foi Concluído Nesta Sessão

### Profile Update (Feature Completa)
- 5 entidades com método `update()` usando `!== undefined` (Syndic, Enterprise, LocalCompany, Marketing, Resident com `updateLinked`/`updateIndependent`)
- 5 schemas Zod de update com campos imutáveis no `omit` (`birthDate`, `foundationDate`, `biometricId`, `document`)
- `UpdateProfileUseCase` com validação de role mismatch (JWT vs body)
- Rota `PATCH /v1/profiles` com `discriminatedUnion` no body
- DI registrado: `awilix.d.ts`, `profile.module.ts`

---

## Gaps Detalhados — Sprint 1

### 1. 🔴 Permissões ABAC (Alta Prioridade)

**Infra existente:**
- Schemas de DB: `permissions`, `plan-permissions`, `plan-quotas`, `feature-usages`
- Entidades: `Permission`, `PlanPermission`, `PlanQuota`, `FeatureUsage`
- Repositórios: 6 repos de billing (incluindo os de permissão)
- Schemas package: `permissions/functional-matrix.ts`, `permission-keys.ts`, `permission-types.ts`, `quota-limits.ts`

**O que falta:**
- [ ] Descomentar e ativar `PermissionService` no `billing.module.ts`
- [ ] Descomentar e ativar `plan-limits.config.ts`
- [ ] Criar middleware de autorização (`authorize.hook.ts`) que consulte `PermissionService`
- [ ] Script de seed para popular `plans`, `permissions`, `plan_permissions` e `plan_quotas`
- [ ] Integrar middleware nas rotas que exigem verificação de plano

---

### 2. 🔴 Billing Use Cases (Média-Alta Prioridade)

**Infra existente:**
- 9 repositórios implementados e registrados no DI
- `StripeProvider` (SINGLETON)
- `WebhookDispatcher` (SINGLETON)
- Rota de webhook registrada

**O que falta:**
- [ ] `ListPlansUseCase` + rota `GET /v1/plans`
- [ ] `CreateSubscriptionUseCase` + rota `POST /v1/subscriptions`
- [ ] `CancelSubscriptionUseCase` + rota `DELETE /v1/subscriptions/:id`
- [ ] `GetSubscriptionUseCase` + rota `GET /v1/subscriptions`
- [ ] `ListInvoicesUseCase` + rota `GET /v1/invoices`
- [ ] `AddPaymentMethodUseCase` + rota `POST /v1/payment-methods`
- [ ] `ListPaymentMethodsUseCase` + rota `GET /v1/payment-methods`
- [ ] Registrar todos no DI do `billing.module.ts`
- [ ] Criar `billing.routes.ts`

---

### 3. 🔴 Motor de Vídeo (Média Prioridade)

**Infra existente:**
- Schemas de DB: `videos`, `video-comments`, `video-likes`, `video-views`

**O que falta:**
- [ ] Provedor de streaming (Mux ou Cloudflare Stream)
- [ ] Módulo `VideoModule` com DI
- [ ] Entidades: `Video`, `VideoComment`, `VideoLike`, `VideoView`
- [ ] Repositórios + implementações Drizzle
- [ ] `UploadVideoUseCase` (com trava de cota trimestral via `FeatureUsage`)
- [ ] `ListVideosUseCase`
- [ ] `GetVideoUseCase` (paywall baseado em plano)
- [ ] `RecordVideoViewUseCase` (contador de views)
- [ ] `LikeVideoUseCase`, `CommentVideoUseCase`
- [ ] Rotas HTTP
- [ ] Schemas Zod no package `mastersindico-schemas`

---

### 4. 🔴 Motor de Busca (Pode Esperar)

**Infra existente:**
- Schemas de DB: `buildings`, `enterprise-categories`, `service-categories`

**O que falta:**
- [ ] Módulo `SearchModule`
- [ ] `SearchProfilesUseCase` (busca geral por categoria, CEP, tipo)
- [ ] `ListCategoriesUseCase`
- [ ] Middleware de visibilidade por plano
- [ ] Rotas HTTP + schemas

---

### 5. 🔴 Connect Me (Pode Esperar)

**O que falta:**
- [ ] Schema de DB para mensagens de contato
- [ ] Módulo `ConnectMeModule`
- [ ] `SendConnectMeUseCase` (com validação de cota anual via `FeatureUsage`)
- [ ] Integração com `EmailProvider` para notificação
- [ ] Rotas HTTP + schemas

---

## Priorização Recomendada

```
Sprint 1 - Ordem de execução sugerida:
┌──────────────────────────────────────┐
│ 1. Permissões ABAC                   │ ← Fundação para tudo
│ 2. Billing Use Cases                 │ ← Depende de ABAC para funcionar
│ 3. Motor de Vídeo                    │ ← Feature principal, depende de billing
│ 4. Motor de Busca                    │ ← Feature de descoberta
│ 5. Connect Me                        │ ← Feature social
└──────────────────────────────────────┘
```

> [!IMPORTANT]
> As permissões ABAC são a **fundação** para billing, vídeo, busca e connect me.
> Sem elas, não há como impor limites de plano nas features.

---

## Resumo Quantitativo

| Métrica | Valor |
|---------|-------|
| Use-cases implementados | **17** (13 auth + 2 profile + 1 onboarding + 1 user) |
| Use-cases faltando (estimativa) | **~15-20** |
| Módulos registrados | **5** feature + **3** infra |
| Módulos faltando | **~3** (video, search, connect-me) |
| Entidades implementadas | **21** |
| Rotas HTTP | **3** arquivos (auth, profile, user) + 1 webhook |
| Rotas faltando | **~3-4** arquivos (billing, video, search, connect-me) |
