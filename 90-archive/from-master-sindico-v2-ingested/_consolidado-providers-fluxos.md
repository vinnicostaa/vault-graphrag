---
title: Consolidado — Providers, Fluxos de Integração, Webhooks, Crons e Eventos de Domínio
type: note
tags: [consolidation, providers, flows, webhooks, events, crons, integrations, master-sindico, v2]
source: relatório Fase 1b (toolu_01XSALYmMs5xjrsWkhNWwDEg.json) destilado de specs/requirements + contextos + legado TS
created: 2026-04-23
updated: 2026-04-23
aliases: [providers-fluxos, integrations-catalog]
---

# Consolidado — Providers, Fluxos de Integração, Webhooks e Eventos de Domínio

> **Origem**: relatório "providers e fluxos de integração" persistido em
> `/home/vinni/.claude/projects/-home-vinni-Documentos-Projetos-Privados-automation-agents/1d72f407-cd66-4ed6-8ee6-c838b924ac35/tool-results/toolu_01XSALYmMs5xjrsWkhNWwDEg.json`
> (91.7 KB, 2528 linhas no texto extraído via `jq -r '.[0].text'`).
>
> **Objetivo**: destilado canônico e exaustivo de todos providers, fluxos outbound/inbound, webhooks, crons, domain events cross-BC e pipelines críticos. **Copiar com fidelidade** — não resumir.
>
> **Data do relatório**: 2026-04-23.
> **Status**: Completo e exaustivo (assinatura do próprio relatório).
>
> Fontes referenciadas no material original (todas mantidas absolutas):
> - `vault/50-projects/master-sindico/specs/requirements/*` (identity, billing, institutional, commercial, content, assembly)
> - `vault/50-projects/master-sindico/specs/design.md`
> - `vault/50-projects/master-sindico/contextos/` (MUX_VIDEO_CONTEXT, stripe-context, etc.)
> - `vault/50-projects/master-sindico/01-domain/{bounded-contexts,business-rules}.md`
> - `vault/50-projects/master-sindico/04-requirements/global.md`

---

## Sumário

1. [Parte I — Catálogo detalhado de providers](#parte-i--catálogo-detalhado-de-providers)
   1. [Stripe](#1-stripe-billing-pagamentos-webhooks)
   2. [Zitadel](#2-zitadel-identity-oidc-mfa-webhooks)
   3. [Mux](#3-mux-video-upload-hls-webhooks-trava-90d)
   4. [Livekit](#4-livekit-room-creation-tokens-webhooks-recording)
   5. [Twilio](#5-twilio-sms-otp-templates-status-callback)
   6. [AWS SES / Resend](#6-aws-ses--resend-email-transacional-templates-bounce)
   7. [FCM (Firebase Cloud Messaging)](#7-fcm-firebase-cloud-messaging-push-notifications)
   8. [OpenSearch / PostgreSQL tsvector](#8-opensearch--postgresql-tsvector-indexing-busca-full-text)
   9. [NATS JetStream](#9-nats-jetstream-mensageria-assíncrona-durable-subscriptions)
   10. [Infra adjacente — S3/MinIO, CloudFront, Route53, CloudWatch, Grafana, Sentry, OpenTelemetry, Railway, AWS ECS](#10-infra-adjacente-storage-cdn-dns-observability-deploy)
2. [Parte II — Catálogo de webhooks externos (entrada)](#parte-ii--catálogo-de-webhooks-externos-entrada)
3. [Parte III — Catálogo de crons / background jobs](#parte-iii--catálogo-de-crons--background-jobs)
4. [Parte IV — Catálogo de domain events (cross-BC)](#parte-iv--catálogo-de-domain-events-cross-bc)
5. [Parte V — Pipelines críticos end-to-end](#parte-v--pipelines-críticos-end-to-end)
6. [Parte VI — Secrets e variáveis de ambiente (consolidado)](#parte-vi--secrets-e-variáveis-de-ambiente-consolidado)
7. [Conclusão e estatísticas](#conclusão-e-estatísticas)

---

## Parte I — Catálogo detalhado de providers

### 1. Stripe (Billing, Pagamentos, Webhooks)

**Papel no sistema**:
- Gerenciador central de subscrições, planos, billing.
- Fonte de verdade para estado de assinatura (trial, active, canceled, expired).
- Responsável por cobrança mensal/anual, upgrade/downgrade, soft-block em expiração.

**Alternativas consideradas**: Mercado Pago (descartada — menos maduro em trial/quotas), PagSeguro (descartada).

**Interface abstrata**:
```go
// infrastructure/providers/payment_gateway.go
type IPaymentGateway interface {
  // Customers
  CreateCustomer(ctx context.Context, email, cpf string) (*Customer, error)
  GetCustomer(ctx context.Context, customerId string) (*Customer, error)

  // Subscriptions
  CreateSubscription(ctx context.Context, customerId, priceId string, trialDays int) (*Subscription, error)
  CancelSubscription(ctx context.Context, subscriptionId string) error
  UpdateSubscription(ctx context.Context, subscriptionId, newPriceId string) (*Subscription, error)

  // Quotas de uso (Metering)
  RecordUsage(ctx context.Context, subscriptionId, metricId string, quantity int) error

  // URLs assinadas
  CreateBillingPortalSession(ctx context.Context, customerId, returnUrl string) (*Session, error)
}

type Subscription struct {
  ID              string       // stripe_subscription_id
  CustomerId      string
  Status          string       // active, past_due, canceled, expired
  CurrentPeriodEnd time.Time
  TrialEndsAt     *time.Time
  PriceId         string
  CreatedAt       time.Time
}
```

#### Fluxos outbound (App → Stripe)

**F1 — Criar Assinatura (Onboarding)**

```
1. UseCase: onboarding/create-subscription
2. Payload:
   - email (validado)
   - cpf/cnpj (existência garantida em User)
   - plan_tier (Enum: Trial/Basic/Pro)
   - trial_days (persona-specific: Síndico=15, Empresa=7, Parceiro=30)

3. Fluxo:
   a. IPaymentGateway.CreateCustomer(email, cpf)
      → idempotency key = hash(email+cpf)
      → Stripe retorna customer_id

   b. IPaymentGateway.CreateSubscription(customer_id, price_id_trial, trial_days)
      → trial_settings.trial_period_days = 15
      → metadata: {"master_sindico_user_id": "uuid", "persona": "syndic"}
      → Stripe retorna subscription_id

   c. Domain event: SubscriptionCreated(userId, subscriptionId, trialEndsAt)

   d. Repository: INSERT subscriptions row
      - subscription_id (PK = Stripe ID)
      - user_id (FK)
      - stripe_customer_id
      - status = 'trial'
      - trial_started_at = NOW()
      - trial_ends_at = NOW() + 15 days

   e. Response: { subscriptionId, trialEndsAt, ... }

4. Erro: stripeErr.Code == "resource_missing" → Criar customer novo e retry

5. Idempotência: idempotency_key + exponencial backoff (max 3 retries)
```

**F2 — Upgrade/Downgrade Plano**

```
1. UseCase: billing/change-plan
2. Payload:
   - user_id
   - new_plan_tier (Pro → Enterprise)
   - effective_date (prorated)

3. Fluxo:
   a. Validar quota: usuário já está neste plano? → erro 409 CONFLICT
   b. Retrieve subscription via user_id (FK)
   c. IPaymentGateway.UpdateSubscription(subscriptionId, newPriceId)
      → Stripe gera fatura prorada automaticamente
      → status permanece 'active' (ou 'past_due' se cobrança pendente)

   d. Domain event: SubscriptionUpdated(userId, oldPlan, newPlan, effectiveDate)
   e. Repository: UPDATE subscriptions SET plan_id = newPlan, updated_at = NOW()
   f. Response: { newPlan, prorationAmount, nextChargeDate, ... }

4. Retry: exponencial com deadline 30s
```

**F3 — Cancelamento (user-initiated)**

```
1. UseCase: billing/cancel-subscription
2. Payload:
   - user_id
   - reason (opcional)
   - refund_unused (boolean)

3. Fluxo:
   a. Validar: subscription status != 'canceled'
   b. IPaymentGateway.CancelSubscription(subscriptionId)
      → Stripe marca subscription como `canceled`
      → Se refund_unused=true, Stripe gera crédito automaticamente

   c. Domain event: SubscriptionCanceled(userId, reason, refundAmount)
   d. Repository: UPDATE subscriptions SET status = 'canceled', canceled_at = NOW()
   e. Soft-block: próximas requisições serão interceptadas por middleware (features premium bloqueadas)
   f. Response: { status = 'canceled', cancellationEffectiveDate, ... }

4. Erro: Stripe retorna subscription já cancelada → idempotência, retorna OK
```

**F4 — Verificar Quota Consumida**

```
1. UseCase: billing/check-quota
2. Payload:
   - user_id
   - quota_type (enum: ConnectMe, VideoUploads)

3. Fluxo:
   a. Retrieve subscription → plan_tier
   b. Query feature_usages table: SUM(used) WHERE user_id AND quota_type AND period
   c. Calcular: remaining = quota_limit - SUM(used)
   d. Response: { used, limit, remaining, percentageUsed, exceededAt? }

4. Cache: Redis `ms:quota:{userId}:{quotaType}:{year}` com TTL = até 1º janeiro

5. Decremento atômico:
   BEGIN;
   SELECT * FROM feature_usages WHERE user_id = $1 FOR UPDATE;
   -- Validar remaining > 0
   INSERT INTO feature_usages (user_id, quota_type, used, period) VALUES (...);
   COMMIT;
```

#### Fluxos inbound / webhooks Stripe

**Endpoint**: `POST /webhooks/stripe` (publicamente acessível, sem middleware Authenticate).

**Eventos críticos**:

| Evento | Ação | Idempotência |
|--------|------|--------------|
| `customer.created` | UPSERT customer local (metadata) | `event.id` unique |
| `payment_intent.succeeded` | Marcar Trial como validado, ativar features | `event.id` + idempotency ledger |
| `subscription.trial_will_end` | Enviar email aviso (24h antes) | `event.id` |
| `subscription.updated` | Sincronizar status (plan changed, paused, etc.) | `event.id` + `subscription.id` |
| `subscription.deleted` | Status = 'canceled', soft-block ativa | `event.id` |
| `customer.subscription.deleted` | Cleanup no Redis (cache ABAC) | `event.id` |
| `invoice.payment_failed` | Notificar via email, marcar status = 'payment_due' | `event.id` |
| `invoice.paid` | Registrar em accounting (futuro), liberar acesso | `event.id` |

**Handler exemplo**:
```go
func (h *StripeWebhookHandler) HandlePaymentIntentSucceeded(ctx context.Context, event StripeEvent) error {
  // 1. Idempotency check
  exists, err := h.repo.GetStripeWebhookEvent(ctx, event.ID)
  if exists {
    return nil  // Já processado
  }

  // 2. Validar HMAC signature (feito antes dessa function)

  // 3. Parse payload
  intent := PaymentIntentPayload{}
  json.Unmarshal(event.Data.Object, &intent)

  // 4. Lógica de negócio
  sub, err := h.repo.GetSubscriptionByStripeId(ctx, intent.Metadata["stripe_subscription_id"])
  if sub.Status == "trialing" {
    // Cobrança validada, ativar trial
    domain.Event -> SubscriptionValidated
  }

  // 5. Registrar processamento
  h.repo.SaveStripeWebhookEvent(ctx, event.ID, "processed")

  return nil
}
```

#### Retry policy Stripe

- **Webhook re-entrega**: Stripe retenta por 3 dias com backoff exponencial; aceitamos re-entrega idempotente.
- **Outbound requests**: exponencial backoff (1s, 2s, 4s, 8s), deadline 30s.
- **Circuito aberto**: se Stripe indisponível por 5 min, falhar-fast com status 503 (não bloquear signup).

#### Saga vs UoW Stripe

- **CreateSubscription**: Saga — se Stripe falha, local já tem transaction aberta. Compensação: `DELETE` subscription row.
  ```
  UoW.Run(ctx, func(tx UoW) error {
    // 1. Create no Stripe
    stripeId, err := paymentGateway.CreateSubscription(...)

    // 2. If erro, compensate automatically (defer)
    if err != nil { return err }

    // 3. Create local
    repo.SaveSubscription(tx, subscriptionId, stripeId)

    // 4. Commit transaction
    return nil
  })
  ```

#### Testes Stripe

- **Unit**: stub `IPaymentGateway` retorna fixtures (subscription, error conditions).
- **Integration**: usar stripe-mock (Go) — `github.com/stripe/stripe-mock`:
  ```bash
  stripe-mock & && go test -tags=integration ./...
  ```
- **Contract**: OpenAPI spec valida requests/responses contra Stripe API docs.

#### Falhas conhecidas Stripe

- **402 Payment Required**: cobrança falhou → soft-block features premium.
- **429 Rate Limited**: backoff exponencial + circuit breaker.
- **401 Unauthorized**: chave Stripe expirada/inválida → ALERT opex.
- **Stripe unavailable (5XX)**: circuit breaker, fail-fast para client.

#### Secrets e config Stripe

```env
STRIPE_SECRET_KEY=sk_live_...            # Produção
STRIPE_PUBLISHABLE_KEY=pk_live_...       # Frontend
STRIPE_WEBHOOK_SECRET=whsec_...          # Validação HMAC
STRIPE_IDEMPOTENCY_TTL_HOURS=24          # Reuso de idempotency keys
MAX_STRIPE_RETRIES=3                     # Retries de outbound
```

#### Antipatterns mitigados (Stripe)

- Antigo (TS): salvar plaintext token Stripe em storage local → **Agora**: middleware cache + introspection.
- Antigo: confiar em client-side para estado de plan → **Agora**: source-of-truth = Stripe + local cache invalidado por webhook.
- Antigo: ignorar webhook re-entregas → **Agora**: idempotency ledger obrigatório.

#### Fontes Stripe

- `specs/requirements/billing.md` — Req 4, 4.1, 6
- `specs/requirements/identity.md` — Req 2 (ABAC soft-block)
- `01-domain/business-rules.md` — BR-018 (quotas, soft-block)

---

### 2. Zitadel (Identity, OIDC, MFA, Webhooks)

**Papel no sistema**:
- Gerenciador de identidade central (CPF/CNPJ → User único).
- Authorization Code Flow OIDC com PKCE.
- Múltiplos contextos: síndico, morador, empresa (mesmo CPF/CNPJ pode ter múltiplos roles).
- MFA (futuro Sprint 5), introspection para validação de token.

**Alternativas consideradas**: Auth0 (mais caro), AWS Cognito (menos flexível pra multi-tenancy Brasil).

**Interface abstrata**:
```go
type IAuthProvider interface {
  // Authorization
  AuthorizeURL(clientId, redirectUri, state, codeChallenge string) string
  ExchangeCode(ctx context.Context, code, codeVerifier string) (*TokenResponse, error)

  // Token
  IntrospectToken(ctx context.Context, accessToken string) (*TokenClaims, error)
  RefreshToken(ctx context.Context, refreshToken string) (*TokenResponse, error)

  // User management
  GetUserInfo(ctx context.Context, accessToken string) (*UserInfo, error)
  CreateUser(ctx context.Context, email, name, role string) (*CreatedUser, error)
  UpdateUser(ctx context.Context, userId, email, name string) error
  SetUserRole(ctx context.Context, userId, role string) error
  InvalidateUserSessions(ctx context.Context, userId string) error
  DeleteUser(ctx context.Context, userId string) error

  // MFA (futuro)
  EnrollMFA(ctx context.Context, userId string) (*MFAEnrollment, error)
  VerifyMFACode(ctx context.Context, userId, code string) (bool, error)
}

type TokenClaims struct {
  Sub              string   // zitadel_user_id
  Email            string
  EmailVerified    bool
  Name             string
  Roles            []string // "syndic", "resident", "enterprise"
  Aud              string   // client_id
  Iss              string   // issuer
  Iat              time.Time
  Exp              time.Time
  AuthTime         time.Time
  Nonce            string   // CSRF protection
}
```

#### Fluxos outbound Zitadel

**F1 — Login via OIDC (Authorization Code Flow + PKCE)**

```
1. Frontend: GET /auth/login?redirect_uri=...&state=random&code_challenge=sha256(verifier)
   - Gera code_verifier, calcula SHA256 → code_challenge
   - Salva em secure cookie (apenas verifier, nunca SHA)

2. Frontend: Redireciona para Zitadel:
   GET https://zitadel.mastersindico.com.br/oauth/v2/authorize
     ?client_id={clientId}
     &redirect_uri=https://mastersindico.com.br/auth/callback
     &response_type=code
     &scope=openid+profile+email
     &state={random}
     &code_challenge={sha256(verifier)}
     &code_challenge_method=S256

3. Zitadel: User faz login, aprova scopes

4. Backend: GET /auth/callback?code={code}&state={state}
   a. Validar state (CSRF)
   b. Recuperar code_verifier da cookie
   c. POST https://zitadel/.../oauth/v2/token
      - code, code_verifier, client_id, grant_type=authorization_code
      → Zitadel retorna access_token, refresh_token, id_token

   d. Decode id_token (JWT sem validação de signature no momento, futuro)
   e. Extract claims: sub (zitadel_user_id), email, name, roles

   f. UPSERT users table:
      INSERT INTO users (zitadel_id, email, name, role, ...)
      ON CONFLICT(zitadel_id) DO UPDATE SET email=EXCLUDED.email, ...

   g. CREATE sessions row:
      - user_id (FK)
      - device_fingerprint
      - ip_address
      - zitadel_session_id
      - access_token_expires_at
      - refresh_token_expires_at
      - created_at

   h. Invalidar outras sessions (1-device policy):
      DELETE FROM sessions
      WHERE user_id = $1 AND device_fingerprint != $2

   i. Set httpOnly cookie: `session={sessionId}; Domain=.mastersindico.com.br; Secure; SameSite=Lax`

   j. Redirect: /dashboard

5. Retry: exponencial se Zitadel indisponível
```

**F2 — Renovar Token (Access Token expirado)**

```
1. Middleware: access_token expirou
2. Recuperar refresh_token da sessão
3. IAuthProvider.RefreshToken(refreshToken)
   POST https://zitadel/.../oauth/v2/token
     - refresh_token, client_id, grant_type=refresh_token
   → Retorna novo access_token
4. Atualizar cache Redis (TTL 5 min)
5. Continuar request original
```

**F3 — Logout**

```
1. Frontend: GET /auth/logout
2. Backend:
   a. Recuperar user_id da sessão
   b. DELETE sessions row
   c. Chamar IAuthProvider.InvalidateUserSessions(userId)
      - POST https://zitadel/.../oidc/logout?id_token_hint={idToken}
   d. Clear cookie: Set-Cookie: session=; Max-Age=0
   e. Redirect: /login
```

**F4 — Criar Novo User (Registro)**

```
1. Frontend: POST /register
   - email, password, name, role (syndic|resident|enterprise|local_company)

2. Backend:
   a. Validar email não existe em users table
   b. IAuthProvider.CreateUser(email, name, role)
      - Zitadel gera userId único
      - Envia email de confirmação (responsabilidade Zitadel)

   c. UPSERT users:
      INSERT INTO users (zitadel_id, email, name, role, status)
      VALUES ($1, $2, $3, $4, 'pending_verification')

   d. Evento: UserCreated(userId, email, role)
      - Trigger: billing module cria Subscription em trial
      - Trigger: onboarding módulo abre fluxo conforme persona

   e. Response: { userId, redirectTo: '/onboarding' }

3. Email de confirmação:
   - Zitadel: envia link com token
   - User clica → Zitadel marca email_verified = true
   - Próximo login: claims terão email_verified = true
   - Middleware valida: se role = 'enterprise' e !email_verified → bloqueado
```

#### Fluxos inbound / webhooks Zitadel (A-DC-QA-050 expandido)

**Endpoint**: `POST /webhooks/zitadel` (publicamente acessível).

**Integração**: Zitadel Actions v2 — triggers configurados em `Actions → Flows` no console Zitadel disparam POSTs assinados para o endpoint acima.

**Verificação**:
- HMAC-SHA256 com `ZITADEL_WEBHOOK_SECRET` validada **antes** de parse (INV-093).
- Header `X-Zitadel-Signature: sha256=...` + comparação constant-time.
- Corpo JSON com `MaxBytesReader(64KB)` (A-DC-SEC-002 mitigação).

**Idempotência**:
- `webhook_events (provider='zitadel', event_id, payload_hash, processed_at)` UNIQUE (provider, event_id) — INV-015.
- Primeira chegada: INSERT + processa; replays: ledger hit → 200 OK sem reprocessar.

**Handler location (proposto)**: `infrastructure/providers/zitadel/webhooks.go` (pasta dedicada; segue pattern `infrastructure/providers/<provider>/` já adotado em `stripe`, `mux`, `livekit`).

**Eventos críticos (catálogo expandido)**:

| Evento | Semântica | Ação no MS | Handler (proposto) | Cross-BC downstream |
|---|---|---|---|---|
| `user.created` | User criado diretamente no Zitadel (bypass onboarding app) | UPSERT local (trigger FluxoF4) | `HandleUserCreated` | — |
| `user.updated` | Atributos alterados no Zitadel | UPSERT em `users` table | `HandleUserUpdated` | Emite `UserUpdated` event |
| `user.deleted` | **Hard delete no Zitadel** | 1) `users.deleted_at = NOW()` (soft); 2) `DELETE sessions WHERE user_id=:id`; 3) Redis `DEL ms:abac:{userID}:*`; 4) publish `user.deleted` event | `HandleUserDeleted` | → billing: cancela subscription pendente (saga compensate se race com webhook Stripe — A-DC-SEC-004); → commercial: invalida ConnectMeRequests do user; → content: marca vídeos como "dono-deletado" |
| `user.email.verified` | User confirmou email via Zitadel | `users.email_verified = true`; libera features (role=enterprise passa a publicar) | `HandleEmailVerified` | → re-avalia `visibilityReady` no perfil (INS-???); libera feature flags `content.video.publish`/`commercial.connect_me.*` |
| `user.mfa.enabled` | User ativou TOTP/Passkey/WebAuthn | Atualiza `risk_profile.mfa_active=true`; diminui risk score | `HandleMFAEnabled` | → ABAC pode relaxar step-up requirement em operações menos críticas; audit `mfa.enabled` |
| `user.mfa.disabled` | User removeu MFA | `risk_profile.mfa_active=false`; aumenta risk; força step-up obrigatório na próxima operação sensível (ADR-0026 proposto) | `HandleMFADisabled` | → Session pode ser forçada a re-autenticar; audit `mfa.disabled` (high-severity) |
| `user.grant.added` | Zitadel atribuiu novo role/scope | Sync `users.role` + invalida cache ABAC | `HandleGrantAdded` | → Redis `DEL ms:abac:{userID}:*`; publish `user.role.changed` |
| `user.grant.removed` | Zitadel removeu role/scope | Sync + invalida cache ABAC | `HandleGrantRemoved` | → idem; se role foi admin, audit high-severity |
| `user.locked` | Admin Zitadel bloqueou | `users.banned=true`; invalida sessions | `HandleUserLocked` | — |
| `user.suspended` | Conta suspensa temporariamente | `users.status='suspended'` | `HandleUserSuspended` | — |
| `session.terminated` | Sessão Zitadel encerrada externamente | `DELETE sessions WHERE zitadel_session_id=:sid` | `HandleSessionTerminated` | → audit `session.terminated_zitadel` |
| `organization.member.added` | User adicionado à org Zitadel | UPSERT `users_roles` local | `HandleOrgMemberAdded` | — |
| `organization.member.removed` | User removido da org Zitadel | DELETE `users_roles`; se last org → trigger `user.deleted` handler | `HandleOrgMemberRemoved` | → idem `user.deleted` se for last org |

**Ação crítica comum a (`user.grant.added`, `user.grant.removed`, `user.role.changed`)**:

```go
// Pattern: invalidação ABAC cache por user
func (h *ZitadelWebhookHandler) invalidateABACCache(ctx context.Context, userID string) error {
    pattern := fmt.Sprintf("ms:abac:%s:*", userID)
    return h.cache.DeletePattern(ctx, pattern)  // Redis SCAN + DEL
}
```

**Ação crítica `user.deleted`** (resolve parte de A-DC-SEC-004):

```go
// Pattern pseudocode — evita race com webhook Stripe 200ms depois
func (h *ZitadelWebhookHandler) HandleUserDeleted(ctx context.Context, event ZitadelEvent) error {
    return h.uow.Run(ctx, func(tx UnitOfWork) error {
        // 1) Soft-flag pending_hard_delete (janela 48h — A-DC-SEC-004)
        if err := tx.Users().MarkPendingHardDelete(ctx, event.UserID); err != nil {
            return err
        }
        // 2) Invalida sessions
        if _, err := tx.Sessions().InvalidateByUserID(ctx, event.UserID, "zitadel.user.deleted"); err != nil {
            return err
        }
        // 3) Invalida ABAC cache
        if err := h.invalidateABACCache(ctx, event.UserID); err != nil {
            return err
        }
        // 4) Publish event via outbox (INV-095) — billing/commercial/content consumirão async
        return tx.Outbox().Publish(ctx, events.UserDeleted{UserID: event.UserID, At: time.Now()})
    })
}
```

**Ação crítica `user.email.verified`** (libera visibility):

```go
func (h *ZitadelWebhookHandler) HandleEmailVerified(ctx context.Context, event ZitadelEvent) error {
    return h.uow.Run(ctx, func(tx UnitOfWork) error {
        if err := tx.Users().SetEmailVerified(ctx, event.UserID, true); err != nil {
            return err
        }
        // Re-avalia visibilityReady (IDN-020 libera publicação vídeo pra empresa)
        if err := h.visibilityEvaluator.Recompute(ctx, event.UserID); err != nil {
            return err
        }
        return tx.Outbox().Publish(ctx, events.UserEmailVerified{UserID: event.UserID})
    })
}
```

**Eventos críticos (versão original mantida — referência rápida)**:

| Evento | Ação | Handler |
|--------|------|---------|
| `user.created` | Já tratado no FluxoF4 (trigger local) | N/A |
| `user.updated` | Sincronizar email/name em users table | UPSERT users |
| `user.deleted` | Soft-delete + invalida sessions + ABAC + event downstream | Ver acima |
| `user.locked` | Marcar banned: `users.banned = true` | UPDATE users |
| `user.suspended` | `users.status = 'suspended'` | UPDATE users |
| `session.terminated` | Invalidar sessão local | DELETE sessions |
| `organization.member.added` | Sincronizar role/grant | UPSERT users_roles |

**Handler exemplo**:
```go
func (h *ZitadelWebhookHandler) HandleUserUpdated(ctx context.Context, event ZitadelEvent) error {
  var userEvent UserUpdatedEvent
  json.Unmarshal(event.Data, &userEvent)

  // Sincronizar
  _, err := h.repo.UpsertUser(ctx, &User{
    ZitadelID: userEvent.UserID,
    Email:     userEvent.Email,
    Name:      userEvent.DisplayName,
    UpdatedAt: time.Now(),
  })

  // Invalidar cache ABAC
  h.cache.Delete(ctx, fmt.Sprintf("abac:%s", userEvent.UserID))

  return err
}
```

**Introspection em middleware**:
```go
func (m *Authenticate) Handle(ctx contracts.Context, next contracts.Next) error {
  token := extractToken(ctx)  // Cookie ou Authorization header

  // 1. Cache check (5 min)
  claims, found := m.cache.Get(token)
  if found && claims.Exp.After(time.Now()) {
    ctx.Set("claims", claims)
    return next()
  }

  // 2. Introspect no Zitadel
  claims, err := m.authProvider.IntrospectToken(ctx, token)
  if err != nil {
    // Refresh token?
    if refreshToken, ok := extractRefreshToken(ctx); ok {
      newToken, err := m.authProvider.RefreshToken(ctx, refreshToken)
      // ... re-try introspect
    }
    return ErrorUnauthorized
  }

  // 3. Validar claims obrigatórios
  if claims.Sub == "" || len(claims.Roles) == 0 {
    return ErrorNoRoleAssigned
  }

  // 4. Cache
  m.cache.Set(token, claims, 5*time.Minute)

  // 5. Próximo middleware
  return next()
}
```

#### Retry policy Zitadel

- **Outbound**: exponencial (1s, 2s, 4s), deadline 10s.
- **Token refresh**: até 3 tentativas.
- **Introspection**: cache 5 min, fallback fail-closed.

#### Saga vs UoW Zitadel

- **CreateUser**: transação local (ACID):
  ```
  UoW.Run(ctx, func(tx UoW) error {
    // Zitadel cria externo (eventual consistency)
    userId, err := authProvider.CreateUser(...)
    if err != nil { return err }

    // Local persiste
    repo.SaveUser(tx, userId, ...)

    return nil
  })
  ```

#### Testes Zitadel

- **Unit**: mock `IAuthProvider`.
- **Integration**: Zitadel testserver (Docker).
- **Contract**: JWT token validation.

#### Falhas conhecidas Zitadel

- **401 token expired**: refrescar automaticamente.
- **403 insufficient scope**: re-autorizar com scopes corretos.
- **Zitadel indisponível**: fallback cache 5 min + fail-closed (403).
- **Invalid PKCE**: erro de geração state/verifier no frontend → detectar log.

#### Secrets e config Zitadel

```env
ZITADEL_ISSUER=https://zitadel.mastersindico.com.br
ZITADEL_CLIENT_ID=...
ZITADEL_CLIENT_SECRET=...
ZITADEL_WEBHOOK_SECRET=...
ZITADEL_ORG_ID=...
MAX_ACTIVE_SESSIONS=1  # 1-device policy
INTROSPECTION_CACHE_TTL_SECONDS=300
```

#### Antipatterns mitigados (Zitadel)

- Armazenar token Zitadel em cache sem expiração → **Agora**: TTL 5 min, refresh automático.
- PKCE desabilitado → **Agora**: obrigatório.
- Sem invalidação de sessões em logout → **Agora**: DELETE local + `InvalidateUserSessions` Zitadel.
- State/nonce sem CSRF check → **Agora**: validação de state obrigatória.

#### Fontes Zitadel

- `specs/requirements/identity.md` — Req 1, 2, 3
- `specs/design/README.md` — implementação OIDC
- `01-domain/bounded-contexts.md` — identity context

---

### 3. Mux (Video Upload, HLS, Webhooks, Trava 90d)

**Papel no sistema**:
- Codificação de vídeo automática (HLS).
- Armazenamento distribuído via CDN.
- Playback com controle de qualidade adaptativa.
- Análise de métricas (views, watch time, completion rate, retention curve).
- Trava de 90 dias após publicação (política anti-churn).

**Alternativas consideradas**: Cloudflare Stream (mais caro), AWS MediaConvert (menos gerenciado).

**Interface abstrata**:
```go
type IVideoProvider interface {
  // Upload
  CreateAsset(ctx context.Context, sourceUrl string, opts UploadOptions) (*Asset, error)
  GetAssetStatus(ctx context.Context, assetId string) (*AssetStatus, error)

  // Direct upload
  CreateDirectUploadUrl(ctx context.Context, opts DirectUploadOptions) (*DirectUploadUrl, error)

  // Playback
  GetPlaybackUrl(ctx context.Context, assetId, playbackId string, signed bool) (string, error)
  GenerateSignedUrl(ctx context.Context, playbackId string, ttl time.Duration) (string, error)

  // Analytics
  GetVideoMetrics(ctx context.Context, assetId string) (*VideoMetrics, error)

  // Deletion
  DeleteAsset(ctx context.Context, assetId string) error
}

type Asset struct {
  ID             string
  PlaybackID     string
  Status         string  // "uploading", "processing", "errored", "ready"
  Duration       int     // segundos
  ThumbnailUrl   string
  PreparedSources []PlaybackPolicy
  CreatedAt      time.Time
}

type VideoMetrics struct {
  Views           int
  AvgWatchTime    int     // segundos
  CompletionRate  float64 // 0-1
  RetentionCurve  []float64 // por bucket de 10%
}
```

#### Fluxos outbound Mux

**F1 — Obter URL para upload direto (presigned upload)**

```
1. UseCase: content/request-video-upload
2. Payload:
   - user_id
   - company_id
   - video_type (enum: presentation, tutorial, demo, etc)
   - duration_estimate (segundos)

3. Validações (local):
   a. User tem permissão? (role=enterprise, plan=Plus/Pro)
   b. Quota mensal? (ex: Empresa Plus = 4/mês)
      - SELECT COUNT(*) FROM videos WHERE company_id = $1 AND created_at > NOW() - interval '1 month'
      - Se >= 4 → erro 403 QUOTA_EXCEEDED

   c. Trava trimestral? (se atualizando vídeo existente)
      - SELECT last_updated FROM videos WHERE id = $2
      - Se < 90 dias atrás → erro 403 LOCKED

4. Request Mux:
   POST https://api.mux.com/video/v1/uploads
   {
     "cors_origin": "https://app.mastersindico.com.br",
     "new_asset_settings": {
       "playback_policies": ["public"],
       "encoding_tier": "baseline"
     }
   }

5. Resposta:
   {
     "id": "upload_id",
     "url": "https://upload.mux.com/...",
     "expires_at": "2026-04-30T..."
   }

6. Response ao client:
   {
     uploadUrl: "https://upload.mux.com/...",
     uploadId: "upload_id",
     expiresIn: 3600
   }

7. Frontend: JavaScript SDK Mux para fazer upload direto (bypass backend)
   - Chunked upload
   - Retry automático
   - Progress tracking

8. Após upload completo: Mux webhook `video.upload.asset_created`
```

**F2 — Verificar Status de Processamento**

```
1. Frontend poll (a cada 5s): GET /api/v1/videos/{id}/status
2. Backend:
   a. Busca video no DB
   b. Se status == 'processing' → chama IVideoProvider.GetAssetStatus(assetId)
   c. Atualiza DB se status mudou (processing → ready ou errored)
   d. Response: { status, progress?, errorMessage? }
```

**F3 — Publicar Vídeo (transição ready → published + lock)**

```
1. UseCase: content/publish-video
2. Payload:
   - video_id
   - title, description (com validação de keywords proibidas)

3. Fluxo:
   a. Retrieve video, validar status == 'ready'
   b. Validar descrição: nenhuma palavra-chave proibida (list em config)
   c. Domain event: VideoPublished(videoId, publishedAt)
   d. Repository:
      UPDATE videos SET
        status = 'published',
        published_at = NOW(),
        locked_until = NOW() + interval '90 days'  -- TRAVA
      WHERE id = $1

   e. Response: { status = 'published', lockedUntil = '2026-07-23', ... }

4. Trava de 90 dias: qualquer tentativa de UPDATE/DELETE dentro desse período resulta em erro
   SELECT * FROM videos WHERE id = $1
   IF published_at IS NOT NULL AND NOW() < locked_until:
     RETURN ERROR 403 LOCKED
```

**F4 — Obter URL de playback (com ou sem assinatura)**

```
1. Frontend: GET /api/v1/videos/{id}/playback
2. Backend:
   a. Retrieve video
   b. Validar acesso (ABAC):
      - Morador Base → erro 403 (preview até 25%)
      - Morador Pagante → acesso completo

   c. IVideoProvider.GetPlaybackUrl(assetId, playbackId, signed=false)
      → Retorna https://stream.mux.com/{playbackId}.m3u8

   d. Se video.requiresAuth:
      - IVideoProvider.GenerateSignedUrl(playbackId, ttl=1h)
      → Retorna https://stream.mux.com/{playbackId}.m3u8?token=...&exp=...

   e. Response:
      {
        playbackUrl: "https://stream.mux.com/...",
        thumbnail: "https://image.mux.com/{playbackId}/thumbnail.jpg",
        duration: 240,
        isLocked: false
      }

3. Frontend: reproduz via Mux Player (web component)
```

#### Fluxos inbound / webhooks Mux

**Endpoint**: `POST /webhooks/mux` (publicamente acessível).

**Eventos críticos**:

| Evento | Ação | Handler |
|--------|------|---------|
| `video.upload.asset_created` | Criar Asset local, `status=processing` | UPSERT videos |
| `video.asset.ready` | `status=ready`, playback_id disponível | UPDATE videos |
| `video.asset.errored` | `status=errored`, error message log | UPDATE videos, notify user |
| `video.asset.deleted` | `status=deleted`, cleanup local | UPDATE videos |
| `video.asset.static_renditions.ready` | MP4s prontos (futuro) | UPDATE videos |

**Handler exemplo**:
```go
func (h *MuxWebhookHandler) HandleAssetReady(ctx context.Context, event MuxEvent) error {
  // 1. Validar HMAC (feito antes)

  // 2. Idempotency
  if h.repo.MuxEventProcessed(ctx, event.ID) {
    return nil
  }

  // 3. Parse
  assetData := event.Data.Object

  // 4. Atualizar vídeo
  _, err := h.repo.UpdateVideo(ctx, &Video{
    MuxAssetID:  assetData.ID,
    MuxPlaybackID: assetData.PlaybackIDs[0].ID,
    Status:      "ready",
    UpdatedAt:   time.Now(),
  })
  if err != nil {
    return err
  }

  // 5. Domain event
  h.eventBus.Publish(VideoReady{VideoID: video.ID, PlaybackID: assetData.PlaybackIDs[0].ID})

  // 6. Marcar processado
  h.repo.SaveMuxWebhookEvent(ctx, event.ID)

  return nil
}
```

#### Retry policy Mux

- **Webhook**: Mux retenta por 5 dias; aceita re-entrega (idempotência).
- **Outbound**: exponencial + deadline 30s.

#### Saga vs UoW Mux

- **CreateAsset + Local save**: Saga.
  ```
  1. Obter directUploadUrl no Mux
  2. Se falha → fail-fast (erro ao usuário)
  3. Se sucesso → UPSERT videos (status=uploading)
  4. Webhook Mux: asset.ready → UPDATE videos (status=ready)
  5. Se webhook nunca chega (Mux down): job noturno verifica status via GetAssetStatus
  ```

#### Testes Mux

- **Unit**: mock `IVideoProvider`.
- **Integration**: faker Mux responses, test webhook handler.
- **Contract**: validar payload de webhook contra Mux docs.

#### Falhas conhecidas Mux

- **429 Rate limited**: backoff, máx 3 retries.
- **Asset processing erro**: webhook `asset.errored` + notificar user.
- **Direct upload timeout**: frontend retry.
- **90-day lock bypass**: validação em DB constraint + middleware.

#### Secrets e config Mux

```env
MUX_TOKEN_ID=...
MUX_TOKEN_SECRET=...
MUX_WEBHOOK_SECRET=...
MUX_VIDEO_MAX_SIZE_MB=5000
MUX_ENCODING_TIER=baseline
VIDEO_LOCK_DAYS=90
```

#### Antipatterns mitigados (Mux)

- Confiar em client status (Mux processing) → **Agora**: sempre checar DB.
- Sem webhook signature verification → **Agora**: HMAC obrigatório.
- Permitir edit video dentro de 90d → **Agora**: constraint em DB + erro 403.

#### Fontes Mux

- `specs/requirements/content.md` — Req 29
- `contextos/MUX_VIDEO_CONTEXT.md`
- `01-domain/bounded-contexts.md` — content context

---

### 4. Livekit (Room Creation, Tokens, Webhooks, Recording)

**Papel no sistema**:
- Gerenciar salas de assembleia ao vivo (SFU — Selective Forwarding Unit).
- Gerar tokens JWT de acesso à sala.
- WebRTC real-time (latência < 200ms crítico).
- Recording automático de assembleia.
- Corte de microfone (fila de fala timeout).

**Alternativas consideradas**: Zoom (mais caro, menos customizável), Jitsi (menos confiável em escala).

**Interface abstrata**:
```go
type ILiveProvider interface {
  // Rooms
  CreateRoom(ctx context.Context, metadata RoomMetadata) (*Room, error)
  DeleteRoom(ctx context.Context, roomName string) error
  ListRooms(ctx context.Context, filter string) ([]*Room, error)

  // Tokens
  GenerateAccessToken(ctx context.Context, roomName, identity string, grants *Grants) (string, error)

  // Room control
  MutePublisher(ctx context.Context, roomName, participantId string) error
  RemoveParticipant(ctx context.Context, roomName, participantId string) error

  // Recording
  StartRecording(ctx context.Context, roomName string, opts RecordingOptions) error
  ListRecordings(ctx context.Context, roomName string) ([]*Recording, error)

  // Webhooks
  GetWebhookSecret() string
}

type Room struct {
  Name              string
  MaxParticipants   int
  CreatedAt         time.Time
  EmptyTimeout      int  // segundos até auto-delete
  CurrentNumParticipants int
  Metadata          string  // JSON
}

type Grants struct {
  Identity   string
  Video      VideoGrants
  Audio      AudioGrants
  CanPublish bool
  CanPublishData bool
}
```

#### Fluxos outbound Livekit

**F1 — Criar Sala de Assembleia (admin/síndico)**

```
1. UseCase: assembly/create-live-session
2. Payload:
   - assembly_id
   - condominium_id
   - recording = true/false

3. Validações:
   a. Síndico tem permissão? (role=syndic, plan=N2/N3)
   b. Assembleia status == 'published'? (edital publicado)

4. Request Livekit:
   POST https://livekit.mastersindico.com.br/twirp/livekit.RoomService/CreateRoom
   {
     "room": {
       "name": "assembly_{assembly_id}",
       "max_participants": 300,
       "empty_timeout": 300,
       "metadata": {
         "assembly_id": "uuid",
         "condominium_id": "uuid",
         "created_at": "ISO8601"
       }
     }
   }

5. Resposta:
   {
     "room": {
       "sid": "room_sid",
       "name": "assembly_...",
       "num_participants": 0
     }
   }

6. Repository: INSERT live_sessions
   - assembly_id (FK)
   - livekit_room_name
   - livekit_room_sid
   - status = 'created'
   - recording_enabled = true/false
   - created_at

7. Domain event: LiveSessionCreated(assemblyId, roomName)

8. Response: { roomName, roomSid, ... }
```

**F2 — Gerar Token JWT para morador (join room)**

```
1. Frontend: GET /api/v1/assemblies/{assembly_id}/live-token
2. Backend:
   a. Validar: user em `memberships` desse condomínio?
   b. Retrieve live_session para assembly
   c. ILiveProvider.GenerateAccessToken
      - roomName: "assembly_{assembly_id}"
      - identity: "{user_id}"
      - grants:
        * CanPublish: false (moradores não transmitem)
        * Video: { can_subscribe: true, can_publish: false }
        * Audio: { can_subscribe: true, can_publish: false }
      → Livekit retorna JWT (válido 24h)

   d. Response:
      {
        "token": "eyJ...",
        "url": "wss://livekit.mastersindico.com.br?access_token=eyJ...",
        "roomName": "assembly_..."
      }

3. Frontend: SDK Livekit JS conecta ao WebSocket
   const room = new Room();
   room.connect(url);
```

**F3 — Cortar microfone do morador (timeout de fala)**

```
1. Timeout dispara (fila de fala, 2 min esgotados)
2. Backend:
   a. ILiveProvider.MutePublisher(roomName, participantId)
      POST https://livekit/.../RemoveParticipant
      {
        "room": "assembly_...",
        "identity": "{user_id}"
      }

   b. Livekit desconecta participant (ou muta apenas áudio)

   c. Domain event: ParticipantMuted(userId, reason="speech_timeout")

   d. Response ao presidente: "Morador XXX foi silenciado (timeout)"

3. Frontend morador: recebe notificação, pode voltar e se registrar novamente na fila
```

**F4 — Iniciar gravação (quando assembleia começa)**

```
1. UseCase: assembly/start-recording
2. Payload:
   - assembly_id

3. Request Livekit:
   POST https://livekit/.../StartRecording
   {
     "room_name": "assembly_{assembly_id}",
     "output": {
       "file": {
         "filepath": "s3://bucket/assemblies/{assembly_id}/{timestamp}.mp4"
       }
     }
   }

4. Livekit inicia gravação automática no S3

5. Repository: UPDATE live_sessions
   - recording_started_at = NOW()
   - status = 'recording'

6. Domain event: RecordingStarted(assemblyId)
```

**F5 — Finalizar gravação e processar arquivo**

```
1. Assembleia encerra (síndico clica "Encerrar")
2. Backend:
   a. ILiveProvider.StopRecording(roomName)  [futuro — Livekit faz auto-stop]
   b. Webhook Livekit: `recording_finished`
      - Arquivo disponível em S3
      - Transcodificar para HLS? [futuro]
      - Thumbnail extraído? [futuro]

   c. Domain event: RecordingFinished(assemblyId, s3Key)
   d. Repository: UPDATE live_sessions
      - recording_finished_at = NOW()
      - recording_s3_key = "s3://..."
      - status = 'closed'

   e. Publicar na Timeline:
      timeline.AppendEntry(type='assembly_recording', reference=assemblyId, s3Key=...)
```

#### Fluxos inbound / webhooks Livekit

**Endpoint**: `POST /webhooks/livekit` (publicamente acessível).

**Eventos críticos**:

| Evento | Ação | Handler |
|--------|------|---------|
| `room_started` | Log apenas (já tracked) | UPDATE live_sessions |
| `room_finished` | Room desabitada, cleanup | DELETE live_sessions ou mark closed |
| `participant_joined` | Atualizar attendance em tempo real | INSERT assembly_check_ins |
| `participant_left` | Remover de presença | UPDATE assembly_check_ins |
| `track_published` | Log (morador habilitou câmera) | UPDATE participant_state |
| `track_unpublished` | Log (desabilitou) | UPDATE participant_state |
| `recording_finished` | Arquivo pronto em S3 | UPDATE live_sessions, publish TimelineEntry |
| `egress_finished` | HLS recording pronto [futuro] | UPDATE live_sessions |

**Handler exemplo**:
```go
func (h *LivekitWebhookHandler) HandleParticipantJoined(ctx context.Context, event LivekitEvent) error {
  participantData := event.Event.Participant

  // 1. Retrieve assembly_id via room_name
  roomName := event.Event.Room.Name
  assembly, err := h.repo.GetAssemblyByLiveroomName(ctx, roomName)

  // 2. Criar check-in
  checkin := &AssemblyCheckIn{
    AssemblyID:    assembly.ID,
    ParticipantID: participantData.Identity,
    CheckedInAt:   time.Now(),
  }
  err = h.repo.SaveCheckIn(ctx, checkin)

  // 3. Evento para WebSocket (painel do presidente)
  h.eventBus.Publish(ParticipantJoined{
    AssemblyID:    assembly.ID,
    ParticipantID: participantData.Identity,
    Name:          participantData.Name,
  })

  return nil
}
```

#### Retry policy Livekit

- **Webhook**: Livekit retenta; aceita idempotência (id de evento).
- **Outbound**: exponencial + deadline 10s.

#### Saga vs UoW Livekit

- **CreateRoom + local save**: UoW.
  ```
  UoW.Run(ctx, func(tx UoW) error {
    room, err := liveProvider.CreateRoom(...)
    if err != nil { return err }

    repo.SaveLiveSession(tx, ...)
    return nil
  })
  ```
- **Compensação (A-028)**: se `room.Create` sucede mas `SaveLiveSession` falha → `DeleteRoom` via job async.

#### Testes Livekit

- **Unit**: mock `ILiveProvider`.
- **Integration**: Livekit testserver (Docker).
- **Contract**: JWT token validation.

#### Falhas conhecidas Livekit

- **Room já existe**: idempotência (retorna existing room).
- **Participant mute fails**: retry ou notificar admin.
- **Recording não inicia**: fallback sem gravação (assembly continua).
- **WebSocket latência > 200ms**: alertar admin.

#### Secrets e config Livekit

```env
LIVEKIT_URL=https://livekit.mastersindico.com.br
LIVEKIT_API_KEY=...
LIVEKIT_API_SECRET=...
LIVEKIT_WEBHOOK_SECRET=...
LIVEKIT_WEBHOOK_API_KEY=...
LIVEKIT_MAX_PARTICIPANTS=300
ASSEMBLY_RECORDING_ENABLED=true
```

#### Antipatterns mitigados (Livekit)

- Permitir múltiplos joins do mesmo user → **Agora**: `identity = user_id` (Livekit rejeita duplicata).
- Sem controle de latência WebSocket → **Agora**: métrica P95 < 200ms em observability.
- Recording sem S3 → **Agora**: obrigatório salvar em S3.

#### Fontes Livekit

- `specs/requirements/assembly.md` — Req 56, 57, 60.2, 60.5, 60.10, 68
- `01-domain/bounded-contexts.md` — assembly context

---

### 5. Twilio (SMS OTP, Templates, Status Callback)

**Papel no sistema**:
- SMS para OTP (futuro, Sprint 5+).
- Notificações críticas (ex: "Assembleia começa em 30 min").
- Status callback para tracking de entrega.

**Alternativas consideradas**: AWS SNS (menos template-friendly), Vonage (similar custo).

**Interface abstrata**:
```go
type ISMSProvider interface {
  // Básico
  SendSMS(ctx context.Context, phoneNumber, message string, opts ...Option) (*SmsResponse, error)
  SendOTP(ctx context.Context, phoneNumber string) (*OtpResponse, error)
  VerifyOTP(ctx context.Context, phoneNumber, code string) (bool, error)

  // Template
  SendTemplated(ctx context.Context, phoneNumber, templateId string, vars map[string]string) (*SmsResponse, error)

  // Status
  GetMessageStatus(ctx context.Context, messageSid string) (*MessageStatus, error)
}

type SmsResponse struct {
  SID     string // message SID
  Status  string // "queued", "sending", "sent", "failed"
  To      string
  Body    string
  SentAt  time.Time
}
```

#### Fluxos outbound Twilio

**F1 — Enviar SMS de notificação (assembleia em 30 min)**

```
1. Cron job: a cada 30 min, busca assembleias começando em 30 min
   SELECT * FROM assemblies
   WHERE scheduled_date = TODAY
     AND scheduled_time = NOW + 30 min
     AND notification_30min_sent = false

2. UseCase: assembly/send-notification-sms
3. Para cada morador da memberships:
   a. Validar: phone no perfil?
   b. ISMSProvider.SendSMS(phone, "Assembleia começa em 30 min. Acesse: ...")
   c. Repository: INSERT sms_logs (phone, template, status, sent_at)
   d. Update assemblies.notification_30min_sent = true

4. Response: sent_count, failed_count
```

**F2 — Enviar OTP por SMS (verificação de cadastro)**

```
1. Frontend: user clica "Enviar via SMS" em 2FA
2. Backend:
   a. Validar phone exists em user profile
   b. ISMSProvider.SendOTP(phone)
      - Twilio gera código 6-dígitos aleatório
      - Envia: "Seu código de verificação é: 123456"

   c. Repository: INSERT otp_requests
      - phone (masked para log)
      - otp_sid (do Twilio)
      - created_at
      - expires_at = NOW + 10 min

   d. Response: { status = 'sent', expiresIn = 600 }

3. Frontend: user recebe SMS, insere código
4. Backend: POST /verify-otp
   a. ISMSProvider.VerifyOTP(phone, code)
   b. Se válido: delete otp_requests, continue login
   c. Se inválido: retry limit 3x (após 3x bloqueado por 1h)
```

#### Fluxos inbound / webhooks Twilio

**Endpoint**: `POST /webhooks/twilio/sms-status`.

| Evento | Ação |
|--------|------|
| `MessageStatus: sent` | UPDATE sms_logs status='sent' |
| `MessageStatus: failed` | UPDATE sms_logs status='failed', alert admin |
| `MessageStatus: delivered` | UPDATE sms_logs status='delivered' (confirmação leitura) |

#### Retry policy Twilio

- **SendSMS**: exponencial + deadline 30s.
- **Webhook**: Twilio retenta 24h.

#### Testes Twilio

- **Unit**: mock `ISMSProvider`.
- **Integration**: Twilio testbed.

#### Falhas conhecidas Twilio

- **429 Rate limit**: backoff.
- **Invalid phone**: validar E.164 formato.

#### Secrets e config Twilio

```env
TWILIO_ACCOUNT_SID=...
TWILIO_AUTH_TOKEN=...
TWILIO_PHONE_NUMBER=+5511999...
TWILIO_WEBHOOK_SECRET=...
SMS_OTP_ENABLED=false  # Sprint 5+
SMS_OTP_TTL_SECONDS=600
```

#### Fontes Twilio

- `specs/requirements/assembly.md` — Req 50 (notificações SMS opt-in)

---

### 6. AWS SES / Resend (Email Transacional, Templates, Bounce)

**Papel no sistema**:
- Email de verificação.
- Notificações de assembleia (3 momentos: T-3d, T-1d, T-3h).
- Trial expirando em 24h.
- Relatórios (export mensais).
- Recuperação de senha.

**Alternativas consideradas**: SendGrid (mais caro), Mailgun (similar).

**Decisão**: SES (mais barato) com fallback Resend (melhor deliverability — D-026 aberto).

**Interface abstrata**:
```go
type IEmailProvider interface {
  // Simples
  SendEmail(ctx context.Context, to, subject, htmlBody string) (*EmailResponse, error)
  SendEmailTemplate(ctx context.Context, to, templateId string, vars map[string]string) (*EmailResponse, error)

  // Batch
  SendBatch(ctx context.Context, recipients []EmailRecipient) ([]*EmailResponse, error)

  // Status
  GetEmailStatus(ctx context.Context, messageId string) (*EmailStatus, error)
}

type EmailResponse struct {
  MessageID string
  Status    string  // "sent", "bounce", "complaint"
  SentAt    time.Time
}
```

#### Fluxos outbound SES/Resend

**F1 — Email de verificação (signup)**

```
1. User registra → Zitadel envia email (responsabilidade Zitadel)
   Ou fallback: Master Síndico envia:

2. UseCase: identity/send-verification-email
3. Request:
   a. Gerar token de verificação (JWT com exp=24h):
      claims = { sub: user_id, type: "email_verification" }
      token = jwt.Sign(claims, secret)

   b. Recuperar template do ES: "email-verification"
   c. IEmailProvider.SendEmailTemplate(
      to: user.email,
      templateId: "email-verification",
      vars: {
        "name": user.name,
        "verification_link": "https://mastersindico.com.br/verify-email?token=" + token
      }
    )

   d. Repository: INSERT email_logs
      - user_id, recipient, template, status, sent_at

   e. Response: { status = 'sent' }

4. User clica link → valida token → marca email_verified = true no Zitadel
```

**F2 — Notificação de assembleia (T-3 dias)**

```
1. Cron job: `0 9 * * *` (9h da manhã)
   SELECT * FROM assemblies
   WHERE scheduled_date = TODAY + 3 days
     AND notification_3d_sent = false

2. Para cada membro do condomínio:
   a. IEmailProvider.SendEmailTemplate(
      to: user.email,
      templateId: "assembly-notification-3d",
      vars: {
        "assembly_title": assembly.title,
        "scheduled_date": assembly.scheduled_date,
        "link": "https://mastersindico.com.br/assemblies/{id}?utm_source=email"
      }
    )

   b. INSERT email_logs
   c. UPDATE assemblies.notification_3d_sent = true
   d. Emit event: AssemblyNotificationSent(assemblyId, "3d", recipientCount)

3. Tracking: link inclui utm_source + utm_campaign para analytics
```

**F3 — Trial expirando em 24h**

```
1. Cron job: `0 8 * * *` (8h da manhã)
   SELECT * FROM subscriptions
   WHERE status = 'trialing'
     AND trial_ends_at BETWEEN NOW AND NOW + 24h
     AND trial_expiring_24h_sent = false

2. Para cada subscription:
   a. Retrieve plan pricing
   b. IEmailProvider.SendEmailTemplate(
      to: user.email,
      templateId: "trial-expiring-24h",
      vars: {
        "plan_name": plan.name,
        "plan_price": fmt.Sprintf("R$ %.2f", plan.monthly_price),
        "upgrade_link": "https://mastersindico.com.br/billing/upgrade?plan={plan_id}"
      }
    )

   c. UPDATE subscriptions.trial_expiring_24h_sent = true
```

#### Fluxos inbound / webhooks SES/Resend

**Endpoint**: `POST /webhooks/ses/bounce` ou `POST /webhooks/resend/events`.

| Evento | Ação |
|--------|------|
| `Bounce: Permanent` | Mark user email invalid; soft-disable notifications |
| `Complaint: abuse` | Alert admin, possible spam report |
| `Send: success` | UPDATE email_logs status='sent' |
| `Open: event` | Track engagement (analytics) |
| `Click: event` | Track CTR (analytics) |

#### Retry policy SES/Resend

- **Transient failure (5XX)**: exponencial + deadline 60s.
- **Permanent bounce**: não retry, mark invalid.
- **Webhook**: SES/Resend retenta 24h.

#### Batch sending

```go
// Enviar email para 1000+ usuários (cron job noturno)
IEmailProvider.SendBatch(ctx, []*EmailRecipient{
  {Email: "user1@...", TemplateVars: {...}},
  {Email: "user2@...", TemplateVars: {...}},
  // ...
})
// Retorna array de responses com IDs para tracking
```

#### Testes SES/Resend

- **Unit**: mock `IEmailProvider`.
- **Integration**: SES testbed (com fake domain de teste).

#### Falhas conhecidas SES/Resend

- **429 Rate limit**: 14 emails/s em SES (backoff).
- **Bounce rate > 2%**: AWS suspende account (vigilância).
- **Complaint rate > 0.1%**: alerta (possível spam).

#### Secrets e config SES/Resend

```env
SES_REGION=us-east-1
SES_FROM_ADDRESS=noreply@mastersindico.com.br
SES_CONFIGURATION_SET=master-sindico  # para tracking bounce/complaint
RESEND_API_KEY=...  # Fallback
EMAIL_PROVIDER=ses  # ou resend
EMAIL_BATCH_SIZE=100
EMAIL_BATCH_DELAY_MS=100
```

#### Antipatterns mitigados (SES/Resend)

- Enviar email sem template → **Agora**: templates versionados em SES.
- Sem bounce tracking → **Agora**: webhook + soft-disable automático.
- PII em subject line → **Agora**: subject genérico, dados em body/links assinados.

#### Fontes SES/Resend

- `specs/requirements/assembly.md` — Req 50 (email notifications)
- `specs/requirements/billing.md` — trial expiring email
- `04-requirements/global.md` — GR-007 (PII em logs)

---

### 7. FCM (Firebase Cloud Messaging, Push Notifications)

**Papel no sistema**:
- Push notifications para mobile (Flutter).
- Token management per device.
- Delivery tracking.

**Alternativas consideradas**: AWS SNS, OneSignal (menos customizável).

**Interface abstrata**:
```go
type IPushProvider interface {
  // Token management
  RegisterToken(ctx context.Context, userId, deviceToken string) error
  UnregisterToken(ctx context.Context, deviceToken string) error

  // Sending
  SendNotification(ctx context.Context, deviceToken, title, body string, data map[string]string) (*PushResponse, error)
  SendMulticast(ctx context.Context, deviceTokens []string, notification *Notification) (*MulticastResult, error)
  SendTopic(ctx context.Context, topic, title, body string) (*PushResponse, error)
}

type Notification struct {
  Title string
  Body  string
  Data  map[string]string
  Badge string
  Sound string
}
```

#### Fluxos outbound FCM

**F1 — Registrar token de push (app inicia)**

```
1. Frontend (Flutter): app inicia
   a. Solicita permission: "Permitir notificações?"
   b. Se permitido: FCM.getToken() → obter device_token
   c. POST /api/v1/user/push-token
      {
        "device_token": "token_from_fcm",
        "platform": "android" | "ios",
        "device_name": "Samsung Galaxy S21"
      }

2. Backend:
   a. IPushProvider.RegisterToken(userId, deviceToken)
      - Associar token ao userId em FCM

   b. Repository: UPSERT push_tokens
      - user_id (FK)
      - device_token (PK)
      - platform
      - device_fingerprint
      - created_at
      - last_used_at

   c. Response: { status = 'registered' }

3. User logout:
   a. POST /api/v1/user/push-token/unregister
   b. IPushProvider.UnregisterToken(deviceToken)
   c. Repository: UPDATE push_tokens.active = false
```

**F2 — Enviar notificação (assembleia em 3h)**

```
1. Cron job: `0 * * * *` (a cada hora)
   SELECT * FROM assemblies
   WHERE scheduled_date = TODAY
     AND scheduled_time = NOW + 3h
     AND notification_3h_sent = false

2. Obter tokens para aquele condomínio:
   SELECT device_token FROM push_tokens pt
   JOIN users u ON pt.user_id = u.id
   JOIN memberships m ON u.id = m.user_id
   WHERE m.condominium_id = assembly.condominium_id
     AND pt.active = true

3. IPushProvider.SendMulticast(
      tokens: [token1, token2, ...],
      notification: {
        title: "Assembleia começando",
        body: "Apartamento 101 - Bloco A",
        data: {
          "assembly_id": assembly.id,
          "link": "mastersindico://assembly/{id}"
        }
      }
    )

4. Repository: INSERT push_notification_logs
   - assembly_id
   - sent_count
   - failure_count
   - sent_at

5. Update: UPDATE assemblies.notification_3h_sent = true
```

**F3 — Enviar por tópico (broadcast global)**

```
1. Admin dispara comunicado global
2. IPushProvider.SendTopic(
   topic: "all_users",
   title: "Manutenção planejada",
   body: "Plataforma offline de 22h a 23h"
 )

3. FCM automaticamente envia para todos subscritos ao tópico "all_users"
```

#### Fluxos inbound / webhooks FCM (feedback de delivery)

**Endpoint**: `POST /webhooks/fcm/feedback`.

| Evento | Ação |
|--------|------|
| `delivery_success` | UPDATE push_notification_logs success=true |
| `delivery_failure` | UPDATE push_notification_logs failure=true; possivelmente InvalidateToken |
| `user_uninstall` | AUTO cleanup token (FCM envia automaticamente) |

#### Retry policy FCM

- **Multicast**: FCM retenta automaticamente.
- **Topic**: broadcast sem retry (fire-and-forget).

#### Testes FCM

- **Unit**: mock `IPushProvider`.
- **Integration**: FCM emulator (Firebase CLI).

#### Secrets e config FCM

```env
FCM_PROJECT_ID=...
FCM_SERVICE_ACCOUNT_JSON=...  # Google Cloud service account
PUSH_NOTIFICATION_ENABLED=true
PUSH_MULTICAST_BATCH_SIZE=500
```

#### Antipatterns mitigados (FCM)

- Notificações sem opt-in → **Agora**: explicit permission request.
- Token expirado sem cleanup → **Agora**: webhook feedback automático.
- Notificação enviada sem device_token → **Agora**: validação antes de send.

#### Fontes FCM

- `specs/requirements/assembly.md` — Req 50 (push notifications)

---

### 8. OpenSearch / PostgreSQL tsvector (Indexing, Busca Full-Text)

**Papel no sistema**:
- Busca rápida de vídeos, empresas, síndicos.
- Filtros facetados.
- Autocomplete.
- Geolocalização (futuro).

**Decisão**: OpenSearch em produção, PostgreSQL tsvector em desenvolvimento (D-024+).

**Interface abstrata**:
```go
type ISearchProvider interface {
  // Indexação
  IndexDocument(ctx context.Context, doc *SearchDocument) error
  DeleteDocument(ctx context.Context, documentId string) error
  Reindex(ctx context.Context, indexName string) error

  // Busca
  Search(ctx context.Context, query string, filters SearchFilters) (*SearchResults, error)
  Autocomplete(ctx context.Context, query string, field string) ([]string, error)
  GetDocumentById(ctx context.Context, documentId string) (*SearchDocument, error)
}

type SearchDocument struct {
  ID          string                 // video_id ou company_id
  Type        string                 // "video", "company", "syndic"
  Title       string
  Description string
  Tags        []string
  Category    string
  Location    map[string]interface{} // geo point
  Rating      float64
  CreatedAt   time.Time
  Metadata    map[string]interface{}
}

type SearchFilters struct {
  Category      string
  MinRating     float64
  Location      *GeoDistance
  DateRange     *DateRange
  Status        string
}
```

#### Fluxos outbound OpenSearch

**F1 — Indexar vídeo (após publicação)**

```
1. Domain event: VideoPublished(videoId, ...)
2. Event subscriber (content module):
   a. Retrieve video + metadata
   b. ISearchProvider.IndexDocument(ctx, &SearchDocument{
      ID: video.ID,
      Type: "video",
      Title: video.title,
      Description: video.description,
      Tags: video.tags,
      Category: video.category,
      Rating: video.rating,
      CreatedAt: video.published_at,
      Metadata: {
        "company_id": video.company_id,
        "thumbnail": video.thumbnail_url,
        "duration": video.duration_seconds,
      }
    })

   c. Erro não bloqueia: log + alert

3. OpenSearch:
   - Cria/atualiza índice `master_sindico_videos`
   - Mapping: title (analyzer: pt_br), description, tags, category, rating
```

**F2 — Busca full-text (usuário digita)**

```
1. Frontend: GET /api/v1/search?q=elevador&category=manutencao&limit=20

2. Backend:
   a. ISearchProvider.Search(ctx, "elevador", SearchFilters{
      Category: "manutencao",
    })

   b. OpenSearch query (Elasticsearch Query DSL):
      {
        "query": {
          "bool": {
            "must": [
              { "multi_match": {
                  "query": "elevador",
                  "fields": ["title^2", "description", "tags"]
                }
              }
            ],
            "filter": [
              { "term": { "category": "manutencao" } }
            ]
          }
        },
        "size": 20,
        "track_scores": true
      }

   c. Resultado:
      {
        "hits": [
          {
            "_id": "video_123",
            "_score": 9.5,
            "_source": {...}
          }
        ]
      }

   d. Mapper: SearchDocument → VideoDTO

   e. Response:
      {
        "results": [
          {
            "id": "video_123",
            "title": "Como substituir elevador",
            "snippet": "Lorem ipsum...",
            "category": "manutencao"
          }
        ],
        "total": 42,
        "nextCursor": "..."
      }

3. Fallback (OpenSearch down):
   - Query PostgreSQL com tsvector:
     SELECT * FROM videos
     WHERE search_vector @@ plainto_tsquery('pt', 'elevador')
     LIMIT 20
```

**F3 — Autocomplete (enquanto digita)**

```
1. Frontend: GET /api/v1/search/autocomplete?q=ele

2. Backend:
   a. ISearchProvider.Autocomplete(ctx, "ele", "title")
   b. OpenSearch: match_phrase_prefix query
      {
        "query": {
          "match_phrase_prefix": {
            "title": "ele"
          }
        }
      }

   c. Response: ["elevador", "eletricista", "eletrônico"]

3. Cache: Redis `search:autocomplete:{field}:{query}` (TTL 1h)
```

#### Fluxos inbound / eventos → reindex

Domain events consumidos pelo search module:
- `VideoPublished` → IndexDocument
- `VideoRemoved` → DeleteDocument
- `CompanyApproved` → IndexDocument
- `CompanyUpdated` → UpdateDocument
- `SyndicProfileUpdated` → UpdateDocument

#### Retry policy OpenSearch

- **Index falha**: log + alertar (não bloqueia operação).
- **Search timeout**: fallback PostgreSQL.

#### Reindexação periódica

```go
// Job noturno (cron `0 2 * * *`):
func ReindexJob(ctx context.Context) error {
  // 1. Criar índice novo com suffix "_new"
  // 2. Copiar docs do índice antigo
  // 3. Adicionar docs novos desde última reindex
  // 4. Testar com queries do monitoring
  // 5. Swap alias: master_sindico_videos → novo índice
  // 6. Deletar índice antigo
  return nil
}
```

#### Testes OpenSearch

- **Unit**: mock `ISearchProvider`.
- **Integration**: OpenSearch testcontainer (Docker).

#### Falhas conhecidas OpenSearch

- **Search timeout > 5s**: retry com query simplificada.
- **Índice desincronizado**: job noturno reindex.
- **Full-text não funciona**: validar analyzer pt_br.

#### Secrets e config OpenSearch

```env
SEARCH_PROVIDER=opensearch  # dev=postgresql
OPENSEARCH_HOST=https://opensearch.mastersindico.com.br
OPENSEARCH_USERNAME=...
OPENSEARCH_PASSWORD=...
OPENSEARCH_INDEX_PREFIX=master_sindico
SEARCH_TIMEOUT_MS=5000
```

#### Antipatterns mitigados (OpenSearch)

- Busca sem fallback → **Agora**: fallback PostgreSQL tsvector.
- Índice desatualizado → **Agora**: event-driven sync + cron reindex.

#### Fontes OpenSearch

- `specs/requirements/content.md` — Req 31

---

### 9. NATS JetStream (Mensageria Assíncrona, Durable Subscriptions)

**Papel no sistema**:
- Event sourcing de domínio.
- Processamento assíncrono (jobs background).
- Garantia de entrega (at-least-once).
- Quando usar vs UoW: cross-context ou latência tolerante.

**Quando usar vs UoW**:
- **UoW** (Unit of Work): tudo cabe na mesma transação de banco (99% dos casos).
- **NATS** (Async queue): quando eventos cruzam contextos ou precisam retry com backoff.
  - Ex: `VideoPublished` → reindex OpenSearch (tolerante latência 5min).
  - Ex: `AssemblyCreated` → enviar notificações via email (retry indefinido).

**Interface abstrata**:
```go
type IEventBus interface {
  // Publish
  Publish(ctx context.Context, event DomainEvent) error

  // Subscribe
  Subscribe(ctx context.Context, subject string, handler Handler) (subscription Subscription, err error)

  // Types
  GetStreamInfo(ctx context.Context, streamName string) (*StreamInfo, error)
}

type DomainEvent interface {
  AggregateID() string
  EventType() string
  Timestamp() time.Time
  Version() int
  Payload() interface{}
}

type Handler = func(ctx context.Context, event DomainEvent) error
```

#### Fluxos NATS

**F1 — Publicar evento dentro de transação (UoW)**

```go
func (u *CreateVideoUseCase) Execute(ctx contracts.Context, cmd *CreateVideoCommand) error {
  return u.uow.Run(ctx, func(tx repositories.UoW) error {
    // 1. Validações
    // 2. Criar entidade
    video := domain.NewVideo(...)

    // 3. Persistir
    err := u.videoRepo.Save(tx, video)
    if err != nil { return err }

    // 4. Publicar evento (registrado mas não enviado até commit)
    u.eventBus.Publish(ctx, &VideoCreated{
      VideoID:   video.ID,
      CompanyID: video.CompanyID,
      CreatedAt: time.Now(),
    })

    // 5. Ao commitar: evento é enviado via NATS
    return nil
  })
}
```

**F2 — Subscriber processando evento (NATS consumer)**

```
Evento publicado: VideoPublished

Subscribers:
1. Content Module: reindex OpenSearch
   - Subject: "domain.content.video.published"
   - Consumer: "search-indexer"
   - Durable: sim
   - Max retry: 5
   - Backoff: exponencial

2. Notification Module: enviar email "Novo vídeo"
   - Subject: "domain.content.video.published"
   - Consumer: "email-notifier"
   - Durable: sim
   - Max retry: indefinido

Fluxo:
1. Evento chega em NATS JetStream
2. Ambos subscribers recebem cópia
3. Cada um processa independentemente
4. Se falha: retry conforme config
5. Se max retry exceeded: move para dead-letter queue
```

**F3 — Consumer configuration**

```
stream: "master_sindico_events"
subjects: [
  "domain.identity.*",
  "domain.billing.*",
  "domain.content.*",
  "domain.assembly.*",
  "domain.commercial.*"
]
retention: "30d"
max_age: "720h"

consumer: "search-indexer"
filter_subject: "domain.content.video.published"
durable_name: "search-indexer"
deliver_policy: "all"  # desde início para catch-up
ack_policy: "explicit"  # manual ACK
max_ack_pending: 100
backoff: [100ms, 1s, 10s, 30s, 60s, ...]
max_deliver: 5
```

#### Retry policy NATS

- **Handler error**: retry com exponencial backoff.
- **Timeout handler**: deadline 30s.
- **Max retries exceeded**: send to DLQ (dead-letter queue).
- **Consumer lag**: monitor via observability.

#### Testes NATS

- **Unit**: mock `IEventBus`.
- **Integration**: NATS testcontainer.

#### Falhas conhecidas NATS

- **Consumer lag > 1h**: alertar ops.
- **Dead-letter queue growing**: investigar handlers com bug.

#### Secrets e config NATS

```env
NATS_JETSTREAM_ENABLED=false  # Sprint 5+ (MVP com in-process event bus)
NATS_URL=nats://nats.mastersindico.com.br:4222
NATS_EVENT_STREAM=master_sindico_events
NATS_CONSUMER_BACKOFF_INITIAL_MS=100
NATS_CONSUMER_BACKOFF_MAX_MS=60000
```

**Quando ativar**:
- Atual (MVP): in-process event bus (simples, fica em memória).
- Sprint 5+: NATS JetStream (quando escalar, múltiplos workers).

#### Fontes NATS

- `01-domain/bounded-contexts.md` — cross-domain

---

### 10. Infra adjacente (Storage, CDN, DNS, Observability, Deploy)

O relatório original cita os seguintes providers de infraestrutura de apoio; descrições compactas (referências a secrets consolidados em [Parte VI](#parte-vi--secrets-e-variáveis-de-ambiente-consolidado)):

- **S3 / MinIO** — armazenamento de recordings Livekit (`s3://bucket/assemblies/{id}/...`), uploads intermediários, atas PDF, dossiês, documentos. MinIO local via docker-compose (Sprint 9.12–9.15). Buckets auto-created: `docs`, `images`, `assembly`. Env: `S3_BUCKET`, `S3_REGION`, `S3_ENDPOINT`, `S3_ACCESS_KEY_ID`, `S3_SECRET_ACCESS_KEY`.
- **CloudFront** — CDN de assets estáticos (thumbnails, PDFs assinados). Env: `CLOUDFRONT_DOMAIN=d123.cloudfront.net`.
- **Route53** — DNS produção (`mastersindico.com.br`, subdomínios `app/admin/zitadel/livekit/opensearch/nats/grafana`).
- **CloudWatch** — métricas AWS (SES bounce, Lambda se aplicável). Integração com observability.
- **Grafana** — dashboard métrica de produção (SLOs, latência p95/p99, error rate, consumer lag NATS). Env: `GRAFANA_URL=https://grafana.mastersindico.com.br`, `PROMETHEUS_PORT=9090`.
- **Sentry** — captura de erros estruturados; agrupamento por release. Env: `SENTRY_DSN=...`.
- **OpenTelemetry (OTel)** — traces distribuídos, spans de cada DB query, Stripe call, Mux call. Env: `OTEL_EXPORTER_OTLP_ENDPOINT=http://otel-collector:4317`. Habilitado em Sprint 7.9.
- **Railway** — deploy gerenciado (hoje); Dockerfile multi-stage + `railway.toml` + GitHub Actions CI (Sprints 9.16–9.18). Env: `RAILWAY_TOKEN`.
- **AWS ECS (Fargate)** — decisão D-002 (migração futura managed → ECS em produção). Env: `AWS_ACCOUNT_ID`, `AWS_REGION`, `ECS_CLUSTER_NAME=master-sindico-prod`, `ECS_SERVICE_NAME=api-server`.

> Papéis detalhados de cada camada (auth/CDN/DNS/observability/deploy) são tratados em decisões arquiteturais D-002 (Railway vs ECS), D-007 (OpenSearch vs PG tsvector), D-008 (Zitadel cloud), e tasks Sprint 0/7/9.

---

## Parte II — Catálogo de webhooks externos (entrada)

Todos os webhooks seguem o padrão:

1. **Validação de signature** antes de parse JSON.
2. **Idempotency ledger** para re-entregas.
3. **Async processing** (não bloqueia resposta).

| Provider | Endpoint | Auth | Idempotency | Eventos | Handler |
|----------|----------|------|-------------|---------|---------|
| **Stripe** | `POST /webhooks/stripe` | HMAC-SHA256 | `event.id` + ledger | customer.created, payment_intent.succeeded, subscription.updated, invoice.paid | StripeWebhookHandler |
| **Zitadel** | `POST /webhooks/zitadel` | HMAC-SHA256 | `event.id` | user.updated, user.deleted, session.terminated | ZitadelWebhookHandler |
| **Mux** | `POST /webhooks/mux` | request_id + object.id | ledger | video.upload.asset_created, video.asset.ready, video.asset.errored | MuxWebhookHandler |
| **Livekit** | `POST /webhooks/livekit` | JWT signature | event_id | room_started, room_finished, participant_joined, recording_finished | LivekitWebhookHandler |
| **SES/Resend** | `POST /webhooks/ses` | AWS SNS subscription confirm | message_id | send, bounce, complaint, open, click | SESWebhookHandler |
| **Twilio** | `POST /webhooks/twilio/sms-status` | request signature | SID | MessageStatus | TwilioWebhookHandler |
| **FCM** | `POST /webhooks/fcm/feedback` | token (assinado pelo FCM) | delivery_id | delivery_success, delivery_failure, user_uninstall | FCMWebhookHandler |

---

## Parte III — Catálogo de crons / background jobs

| Job | Schedule | Módulo | O que faz | Failure Policy |
|-----|----------|--------|-----------|-----------------|
| **Trial expiring 24h email** | `0 8 * * *` | billing | Query subs com trial_ends_at ≈ NOW+24h, enviar email | Log + retry 1x amanhã |
| **Trial expiring 3h check-in** | `0 * * * *` | assembly | Query assembleias em T-3h, enviar SMS/push | Log + retry próxima hora |
| **Trial expiring 1d check-in** | `0 9 * * *` | assembly | Query assembleias em T-1d, enviar push | Log |
| **Trial expiring 3d check-in** | `0 10 * * 0-5` | assembly | Query assembleias em T-3d, enviar push | Log |
| **Reset quotas (mensais)** | `0 0 1 * *` | billing | Reset de `video_uploads` quota para plano mensal | Idempotente |
| **Reset quotas (anuais)** | `0 0 1 1 *` | billing | Reset de ConnectMe quota | Idempotente |
| **Assembleia expirada → auto-close** | `0 * * * *` | assembly | Query assembleias com `close_at < NOW`, mudar status → closed | Idempotente |
| **Reindex OpenSearch** | `0 2 * * *` | content | Recriar índice, copiar docs | Log; fallback PostgreSQL OK |
| **Sincronizar usuários Zitadel** | `0 3 * * *` | identity | Query users deletados em Zitadel, soft-delete local | Log |
| **Limpeza sessões expiradas** | `0 * * * *` | identity | DELETE sessions expiradas | Idempotente |
| **Limpeza de cache Redis** | `0 5 * * *` | shared | Trigger FLUSHDB (se Redis config permitir) | Não-crítico |
| **Reconhecer síndico (Bronze→Diamante)** | `0 4 * * *` | commercial | Recalcular reputação para cada empresa | Idempotente |
| **Gerar certificados pendentes** | `0 6 * * *` | content | Query aulas_concluidas com cert pendente, gerar PDF+QR | Log |
| **Limpar upload direto expirado** | `0 * * * *` | content | DELETE uploads não-completados há > 24h | Idempotente |
| **Reconciliação Stripe** | `0 0 * * 0` | billing | Comparar subscriptions Stripe vs local, alertar divergências | Log (não-crítico) |

---

## Parte IV — Catálogo de domain events (cross-BC)

Estes eventos são publicados por um bounded context e consumidos por outro(s).

### 4.1 Identity Context

| Evento | Payload | Publisher | Subscribers | Side-Effect |
|--------|---------|-----------|-------------|------------|
| `UserCreated` | userId, email, role, timestamp | Middleware auth | Billing (cria Subscription em trial) | Trial auto-iniciado |
| `UserUpdated` | userId, email, name, ... | Zitadel webhook | Billing (sincroniza customer) | |
| `SessionInvalidated` | userId, sessionId | Logout handler | Billing cache (invalidar ABAC) | Usuário vira "not_authorized" |

### 4.2 Billing Context

| Evento | Payload | Publisher | Subscribers | Side-Effect |
|--------|---------|-----------|-------------|------------|
| `SubscriptionActivated` | userId, planTier, trialEndsAt | Stripe webhook | Institutional (desbloqueia features) | Síndico pode criar timeline |
| `SubscriptionCanceled` | userId, reason | Cancellation handler | Institutional (soft-block) | Síndico não consegue criar votação |
| `TrialExpired` | userId, planTier | Trial-check job | Notification (email "upgrade agora") | Email dispatch |
| `QuotaExhausted` | userId, quotaType, period | Quota-check middleware | Notification | Bloqueio de ação + UI msg |

### 4.3 Institutional Context

| Evento | Payload | Publisher | Subscribers | Side-Effect |
|--------|---------|-----------|-------------|------------|
| `CondominiumCreated` | condoId, name, cnpj | Onboarding | Commercial (cache) | Síndico consegue abrir assembleia |
| `MembershipAssigned` | userId, condoId, role | Membership handler | Billing (validar acesso), Commercial (ABAC cache) | User consegue votar |
| `TimelineAppended` | condoId, entryId, type, timestamp | Timeline handler | Notification (email histórico), Content (indexar) | Entrada imutável registrada |
| `AnnouncementPublished` | announcementId, condoId, message | Announcement handler | Notification (email) | Aviso enviado para todos |
| `MasterPlanActivityAdvanced` | planId, activityId, newStatus | Plano diretor | Notification (update progress bar) | Timeline entry criada |

### 4.4 Commercial Context

| Evento | Payload | Publisher | Subscribers | Side-Effect |
|--------|---------|-----------|-------------|------------|
| `CompanyApproved` | companyId, approvedAt | Moderation | Content (permite upload), Billing (ativa features) | Empresa consegue fazer upload |
| `ConnectMeOpened` | requestId, syndicId, companyId | RFP handler | Notification (email para empresa) | Empresa recebe notificação |
| `ProposalSubmitted` | proposalId, requestId | Proposal handler | Notification (email para síndico) | Síndico vê proposta nova |
| `DealClosed` | dealId, companyId, value | Deal handler | Notification, Timeline (entry criada) | Contrato registrado |
| `CompanyEvaluated` | companyId, rating, evaluatorUserId | Evaluation handler | Commercial (recalc reputação → `ReputationTierChanged`), Timeline | Reputação muda |
| `ReputationTierChanged` | companyId, oldTier, newTier | Reputation calculator | Notification (email "Parabéns, você é Ouro!"), Content (badge update) | Status visual muda |
| `HarassmentReported` | reportId, companyId, confirmedAt | Harassment handler | Commercial (suspend company), Notification (alert admin) | Empresa bloqueada |

### 4.5 Content Context

| Evento | Payload | Publisher | Subscribers | Side-Effect |
|--------|---------|-----------|-------------|------------|
| `VideoUploaded` | videoId, companyId, muxAssetId | Upload handler | Notification | Email "vídeo recebido" |
| `VideoApproved` | videoId, approvedAt | Moderation handler | Notification (email "aprovado") | Vídeo visível |
| `VideoPublished` | videoId, publishedAt, lockedUntil | Publish handler | Content (reindex), Notification | Vídeo locked por 90d, indexado |
| `VideoRemoved` | videoId | Deletion handler | Content (delete from search), Notification | Vídeo desaparece |
| `VideoLocked` | videoId, lockedUntil | Lock handler (90d trigger) | Notification (silent) | Vídeo não editável |

### 4.6 Assembly Context

| Evento | Payload | Publisher | Subscribers | Side-Effect |
|--------|---------|-----------|-------------|------------|
| `AssemblyScheduled` | assemblyId, condoId, scheduledDate | Assembly handler | Notification (T-3d job) | Assembleia salva |
| `EditalPublished` | assemblyId, editalUrl | Edital handler | Notification (email "edital disponível"), Institutional (timeline entry) | Edital distribuído |
| `AssemblyStarted` | assemblyId, liveRoomName | Start handler | Livekit (já criou room), Notification (live começou) | Check-in aberto |
| `VoteCast` | voteId, assemblyId, userId, choice | Vote handler | Assembly (tally result), Timeline (registrar voto) | Voto contado |
| `AssemblyClosed` | assemblyId, closedAt | Close handler | Assembly (gerar ata), Notification | Ata criada |
| `MinutesPublished` | minutesId, assemblyId, s3Key | Minutes handler | Institutional (append to timeline), Notification | Ata registrada |

---

## Parte V — Pipelines críticos end-to-end

### Pipeline 1 — Signup completo (Síndico)

```
┌─────────────────────────────────────────────────────────────────┐
│ 1. Frontend: GET /auth/login?role=syndic&redirect_uri=...       │
│    Gera code_verifier, code_challenge                            │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│ 2. Zitadel: Authorization Code Flow + PKCE                      │
│    - User registra (email, senha, name)                          │
│    - Email enviado por Zitadel                                   │
│    - Zitadel retorna authorization code                          │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│ 3. Backend: GET /auth/callback?code=...&state=...               │
│    - Validar state (CSRF)                                        │
│    - Exchange code → access_token, refresh_token                 │
│    - Decode JWT, extract claims (sub, email, roles)              │
│    - UPSERT users table (zitadel_id, email, role=syndic)         │
│    - CREATE sessions row                                         │
│    - DELETE other sessions (1-device policy)                     │
│    - Domain event: UserCreated(userId, email, role)              │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│ 4. Billing Module (subscriber de UserCreated)                    │
│    - Detectar persona (syndic) → trial_days = 15                 │
│    - IPaymentGateway.CreateCustomer(email, cpf)                  │
│    - IPaymentGateway.CreateSubscription(..., trial_days=15)      │
│    - INSERT subscriptions (status='trial', trial_ends_at=...)    │
│    - Domain event: SubscriptionActivated(...)                    │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│ 5. Institutional Module (subscriber de SubscriptionActivated)    │
│    - Unlock features: "Criar timeline", "Criar assembleia"       │
│    - UPSERT ABAC cache (user_id, action) → allowed               │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│ 6. Frontend: Redirect to /onboarding                             │
│    - Step 1: Dados pessoais (name, cpf)                          │
│    - Step 2: Tipo síndico (principal, auxiliar)                  │
│    - Step 3: Governance markers                                  │
│    - Step 4: Criar ou vincular condomínio                        │
│    - Step 5: Mandato (data início, valor)                        │
│    - Step 6: Empresa síndica (opcional)                          │
│    - Auto-save em Redis: onboarding:{user_id}:{step}             │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│ 7. Post-Onboarding: Migrate from Redis to DB                     │
│    - INSERT profiles_sindico (cpf, tempo_carreira, ...)          │
│    - INSERT condominios (or link existing)                        │
│    - INSERT vinculos_sindico_condominio                          │
│    - UPDATE users.status = 'active'                              │
│    - DELETE Redis onboarding key                                 │
│    - Domain event: SyndicProfileComplete(userId)                 │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│ 8. Final: Redirect to dashboard                                  │
│    - Trial: 15 dias de acesso completo                           │
│    - Pode criar assembleias, convidados, timeline                │
│    - Email: "Bem-vindo! Seu trial expira em 15 dias"             │
└─────────────────────────────────────────────────────────────────┘

Tempo total: ~5-10s (sync) + emails async
Falha points:
  - Zitadel indisponível → 503 Service Unavailable
  - Stripe indisponível → fail-soft (trial via local subscription mock)
  - Email não chega → user pode manually verify later
```

### Pipeline 2 — Envio de vídeo completo

```
┌─────────────────────────────────────────────────────────────────┐
│ 1. Frontend: POST /api/v1/videos/request-upload                  │
│    - User: empresa com plano Plus (4/mês)                        │
│    - Payload: title, description, type="tutorial"                │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│ 2. Backend Validations                                           │
│    - Quota check: SELECT COUNT(*) FROM videos WHERE company_id   │
│      AND created_at > NOW() - interval '1 month'                 │
│      → 3 vídeos encontrados, limite 4 OK                         │
│    - Trava trimestral: se atualizando vídeo, verificar 90d       │
│    - User permission: ABAC → role=enterprise, plan=plus OK       │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│ 3. Mux: Obter Presigned Upload URL                               │
│    - POST https://api.mux.com/video/v1/uploads                   │
│    - Response: { id, url, expires_at }                           │
│    - Response ao frontend: uploadUrl, uploadId, expiresIn        │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│ 4. Frontend: Upload Direto (JavaScript SDK Mux)                  │
│    - Chunked upload ao Mux (bypass backend)                      │
│    - Progress tracking                                           │
│    - Retry automático se falha chunk                             │
│    - Após completo: Mux webhook video.upload.asset_created       │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│ 5. Mux Webhook: asset_created                                    │
│    - POST /webhooks/mux (signature verify HMAC)                  │
│    - Idempotency: event.id check                                 │
│    - Payload: assetId, playbackId, status                        │
│    - Backend: INSERT videos (status=uploading, muxAssetId)       │
│    - Domain event: VideoUploaded(videoId, muxAssetId)            │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│ 6. Frontend: Poll Status (GET /api/v1/videos/{id}/status)        │
│    - Check status every 5s                                       │
│    - Status: uploading → processing → ready                      │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│ 7. Mux Webhook: asset_ready                                      │
│    - POST /webhooks/mux                                          │
│    - Backend: UPDATE videos (status=ready, playbackId)           │
│    - Duration extraído do Mux                                    │
│    - Thumbnail URL gerado                                        │
│    - Domain event: VideoReady(videoId, playbackId, duration)     │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│ 8. Frontend: "Seu vídeo está pronto"                             │
│    - Button: "Revisar e publicar"                                │
│    - User clica → POST /api/v1/videos/{id}/publish               │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│ 9. Backend: Publish Video                                        │
│    - Validar: status = ready, não keywords proibidas             │
│    - UPDATE videos: status=published, published_at, locked_until │
│    - Domain event: VideoPublished(videoId, publishedAt, ...)     │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│ 10. Content Module (subscriber de VideoPublished)                │
│     - ISearchProvider.IndexDocument(videoId, metadata)           │
│     - OpenSearch: indexar com tags, categoria, company_id        │
│     - Domain event: VideoIndexed(videoId)                        │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│ 11. Notification Module (subscriber de VideoPublished)           │
│     - Enviar email: "Seu vídeo foi publicado"                    │
│     - Link para visualizar                                       │
│     - Acesso será visível para moradores pagos                   │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│ 12. Trava de 90 Dias Ativa                                       │
│     - Qualquer UPDATE/DELETE video neste período → erro 403      │
│     - User não consegue substituir até 2026-07-23                │
└─────────────────────────────────────────────────────────────────┘

Tempo total: ~30s (mux processing) + webhooks async
Falha points:
  - Mux webhook nunca chega → status fica "uploading" (monitora com job noturno)
  - Mux asset error → backend notifica user, valida 403
```

### Pipeline 3 — Assembleia / Votação end-to-end (com Livekit live)

```
┌─────────────────────────────────────────────────────────────────┐
│ 1. Síndico: Criar Assembleia                                     │
│    - POST /api/v1/assemblies                                     │
│    - type=ordinaria, scheduled_date=2026-05-15, time=10:00       │
│    - status=scheduled                                            │
│    - Cria live_sessions (se recording_enabled=true)              │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│ 2. Síndico: Criar Pauta e Publicar                               │
│    - POST /api/v1/assemblies/{id}/agenda-items                   │
│    - Item 1: "Aprovação reforma elevador" (votacao_simples)      │
│    - Item 2: "Escolha fornecedor" (votacao_fornecedor)           │
│    - Publicar edital (draft → published)                         │
│    - Domain event: EditalPublished(...)                          │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│ 3. Notification Cron (T-3 dias)                                  │
│    - Job: `0 10 * * 0-5`                                         │
│    - SELECT assemblies WHERE scheduled = NOW + 3d                │
│    - Para cada morador:                                          │
│      * IEmailProvider.SendEmailTemplate(...)                     │
│      * IPushProvider.SendMulticast(...)                          │
│    - UPDATE assemblies.notification_3d_sent = true               │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│ 4. Morador: Dar Ciência da Pauta (App Obrigatório)               │
│    - Modal aparece: "Você precisa da ciência da pauta"           │
│    - Checkbox: "Li e entendi"                                    │
│    - POST /api/v1/assemblies/{id}/acknowledge                    │
│    - Backend: INSERT assembly_acknowledgments (user_id, time)    │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│ 5. Dia da Assembleia: Síndico Inicia (9:00)                      │
│    - PUT /api/v1/assemblies/{id}/status=in_progress              │
│    - Livekit: já criou room (sou criar link ao assembly)         │
│    - Status: broadcasting pauta ao vivo                          │
│    - Domain event: AssemblyStarted(...)                          │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│ 6. Morador: Check-in (App ou Terminal Kiosk)                     │
│    - Opção 1 (App): Scan QR code gerado pelo síndico             │
│    - Opção 2 (Terminal): CPF + bloco + unidade                   │
│    - Opção 3 (Manual): Síndico confirma presença                 │
│    - POST /api/v1/assemblies/{id}/check-in                       │
│    - Backend:                                                    │
│      * INSERT assembly_check_ins (user_id, assembly_id, time)    │
│      * Webhook Livekit notifica participant_joined               │
│      * WebSocket: presidente vê "Morador X presente" em P95<200ms│
│    - Cálculo de quorum atualizado em tempo real                  │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│ 7. Síndico: Abrir Item 1 (Aprovação Elevador)                    │
│    - Via Painel do Presidente (web)                              │
│    - PUT /api/v1/assemblies/{id}/agenda-items/1/open-voting      │
│    - Status: votacao_aberta                                      │
│    - WebSocket broadcast: moradores recebem "Votação aberta"     │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│ 8. Morador: Vota (App)                                            │
│    - Recebe notificação: "Item 1 aberto para votação"            │
│    - Toca em "Votar"                                             │
│    - Opções: Aprovar / Reprovar / Abster                          │
│    - POST /api/v1/assemblies/{id}/agenda-items/1/vote            │
│    - Backend:                                                    │
│      * Validar: user em memberships, check-in feito              │
│      * INSERT votes (agenda_item_id, user_id, choice)            │
│      * CONSTRAINT: UNIQUE(agenda_item_id, user_id) → no dup      │
│      * Domain event: VoteCast(...)                               │
│      * WebSocket: atualizar contagem real-time                   │
│      * Resposta ao morador: "Seu voto foi registrado"            │
│    - P95 latência: < 200ms                                       │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│ 9. Painel do Presidente: Resultado Real-Time                     │
│    - Gráfico atualiza: Aprovar: 75%, Reprovar: 25%               │
│    - Quorum: 85% de presença (fração ideal)                      │
│    - Botão: "Fechar votação"                                     │
│    - Síndico clica ao fim da discussão                           │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│ 10. Síndico: Fechar Votação                                      │
│     - PUT /api/v1/assemblies/{id}/agenda-items/1/close-voting    │
│     - Calcula resultado final (imutável)                         │
│     - Status: votacao_fechada                                    │
│     - Domain event: VotingClosed(result={"Aprovar": 45, ...})    │
│     - Telão mostra resultado final (bloqueado)                   │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│ 11. Síndico: Encerrar Assembleia                                 │
│     - PUT /api/v1/assemblies/{id}/status=closed                  │
│     - Livekit: room fecha (recording para)                       │
│     - Domain event: AssemblyClosed(...)                          │
│     - Async job: gerar ata PDF (template HTML → wkhtmltopdf)     │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│ 12. Webhook Livekit: recording_finished                          │
│     - POST /webhooks/livekit                                     │
│     - Backend:                                                   │
│       * S3Key: s3://bucket/assemblies/{id}/recording.mp4         │
│       * UPDATE live_sessions.recording_s3_key                    │
│       * Domain event: RecordingFinished(...)                     │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│ 13. Ata Gerada e Publicada na Timeline                           │
│     - Async job cria ata PDF                                     │
│     - Armazena em MinIO/S3                                       │
│     - Domain event: MinutesPublished(...)                        │
│     - Institutional module:                                      │
│       * INSERT timeline_entries (type=assembly_result, ...)      │
│       * Ata é imutável (sem DELETE)                              │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│ 14. Notification: Ata Disponível                                 │
│     - Email: "Ata da assembleia disponível"                      │
│     - Link: /assemblies/{id}/minutes.pdf                         │
│     - Push notification                                          │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│ 15. Síndico: Solicitar Homologação (Votação nova)                │
│     - PUT /api/v1/assemblies/{id}/request-homologation           │
│     - Nova votação: "Aprova a ata?"                              │
│     - Moradores votam novamente                                  │
│     - Resultado: Sim → ata.homologated_at = NOW (imutável)       │
│     - Resultado: Não → ata volta para edição (raro)              │
│     - Domain event: AssemblyHomologated(...)                     │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│ 16. Relatórios e Score de Transparência                          │
│     - Job calcula 10 componentes (pauta publicada, quorum, etc)  │
│     - Score 0-100: exibido publicamente                          │
│     - Histórico: últimas 12 assembleias (benchmark)              │
│     - Relatórios exportáveis: presença, votação, fala            │
└─────────────────────────────────────────────────────────────────┘

Tempo total: 1-3 horas (evento real)
Falha points:
  - WebSocket desconecta → fallback HTTP polling / modo offline
  - Mux room problema → fallback sem recording
  - Email não chega → ata visível no app
```

### Pipeline 4 — Connect Me (Síndico → Empresa, unidirecional com LGPD)

> Referência: COM-002, COM-003, COM-004, COM-005 do relatório specs.

```
1. Síndico busca empresa (categoria/localização/reputação).
2. Clica "Tenho interesse" (formulário unidirecional): envia necessidade textual,
   sem revelar dados pessoais do síndico.
3. Backend: POST /api/v1/connect-me/sindico-to-empresa
   - Valida quota anual ConnectMe do síndico (2/4/ilimitado por plano).
   - Decrementa `ms:quota:{userId}:connect_me:{year}` atomicamente.
   - INSERT connect_me_requests (status='awaiting_interest', auto_closed_at=NOW+48h).
   - Domain event: ConnectMeOpened(requestId, syndicId, companyId).
4. Notificação: empresa recebe push + email "Síndico demonstrou interesse".
5. Empresa clica "Tenho interesse":
   - Backend valida LGPD consent (versão de termos), registra timestamp+IP+finalidade em `lgpd_registry`.
   - Revela dados do síndico (nome, condomínio, contato) apenas agora.
   - Domain event: ContactDataRevealed(requestId).
6. Síndico e empresa conversam fora do produto (comercial). No app: status 'in_negotiation'.
7. Se 48h sem interesse → cron auto-close: `connect_me_requests.status='auto_closed'`.
8. Se síndico recebe abordagem indesejada: botão "Denunciar Assédio" → evidence log,
   Domain event `HarassmentReported`, admin actions (suspender empresa se confirmado).
```

### Pipeline 5 — Checkout Stripe (Trial → Paid)

> Referência: BIL-001, BIL-002, BIL-004, BIL-006, BIL-013 consolidados.

```
1. User atinge fim do trial (cron `trial expiring 24h` já notificou via email).
2. Trial expira → Stripe emite `subscription.trial_will_end` (24h antes) e,
   no fim, `invoice.payment_failed` se não houver método de pagamento.
3. Middleware `CheckSubscriptionActive` bloqueia rotas premium (403 QUOTA_EXCEEDED /
   mensagem "Essa funcionalidade está disponível nos planos ativos...").
4. User acessa Customer Portal Stripe via `IPaymentGateway.CreateBillingPortalSession(customerId, returnUrl)`.
5. User adiciona cartão no Stripe Customer Portal (fora do backend).
6. Stripe cobra → emite `invoice.paid` + `payment_intent.succeeded`.
7. Webhook Stripe:
   - StripeWebhookHandler valida HMAC + idempotency.
   - Upsert `subscriptions.status='active'`.
   - Domain event: `SubscriptionActivated(userId, planTier, ...)`.
8. Institutional subscriber desbloqueia ABAC cache → features premium liberadas.
9. Email de boas-vindas plano pago enviado via Resend/SES.
```

---

## Parte VI — Secrets e variáveis de ambiente (consolidado)

```env
# ========== STRIPE ==========
STRIPE_SECRET_KEY=sk_live_...
STRIPE_PUBLISHABLE_KEY=pk_live_...
STRIPE_WEBHOOK_SECRET=whsec_...
STRIPE_IDEMPOTENCY_TTL_HOURS=24
MAX_STRIPE_RETRIES=3

# ========== ZITADEL ==========
ZITADEL_ISSUER=https://zitadel.mastersindico.com.br
ZITADEL_CLIENT_ID=...
ZITADEL_CLIENT_SECRET=...
ZITADEL_WEBHOOK_SECRET=...
ZITADEL_ORG_ID=...
MAX_ACTIVE_SESSIONS=1
INTROSPECTION_CACHE_TTL_SECONDS=300

# ========== MUX ==========
MUX_TOKEN_ID=...
MUX_TOKEN_SECRET=...
MUX_WEBHOOK_SECRET=...
MUX_VIDEO_MAX_SIZE_MB=5000
MUX_ENCODING_TIER=baseline
VIDEO_LOCK_DAYS=90

# ========== LIVEKIT ==========
LIVEKIT_URL=https://livekit.mastersindico.com.br
LIVEKIT_API_KEY=...
LIVEKIT_API_SECRET=...
LIVEKIT_WEBHOOK_SECRET=...
LIVEKIT_WEBHOOK_API_KEY=...
LIVEKIT_MAX_PARTICIPANTS=300
ASSEMBLY_RECORDING_ENABLED=true

# ========== TWILIO ==========
TWILIO_ACCOUNT_SID=...
TWILIO_AUTH_TOKEN=...
TWILIO_PHONE_NUMBER=+5511999...
TWILIO_WEBHOOK_SECRET=...
SMS_OTP_ENABLED=false
SMS_OTP_TTL_SECONDS=600

# ========== SES/RESEND ==========
SES_REGION=us-east-1
SES_FROM_ADDRESS=noreply@mastersindico.com.br
SES_CONFIGURATION_SET=master-sindico
RESEND_API_KEY=...
EMAIL_PROVIDER=ses
EMAIL_BATCH_SIZE=100
EMAIL_BATCH_DELAY_MS=100

# ========== FCM ==========
FCM_PROJECT_ID=...
FCM_SERVICE_ACCOUNT_JSON=...
PUSH_NOTIFICATION_ENABLED=true
PUSH_MULTICAST_BATCH_SIZE=500

# ========== OPENSEARCH ==========
SEARCH_PROVIDER=opensearch
OPENSEARCH_HOST=https://opensearch.mastersindico.com.br
OPENSEARCH_USERNAME=...
OPENSEARCH_PASSWORD=...
OPENSEARCH_INDEX_PREFIX=master_sindico
SEARCH_TIMEOUT_MS=5000

# ========== NATS JETSTREAM ==========
NATS_JETSTREAM_ENABLED=false
NATS_URL=nats://nats.mastersindico.com.br:4222
NATS_EVENT_STREAM=master_sindico_events
NATS_CONSUMER_BACKOFF_INITIAL_MS=100
NATS_CONSUMER_BACKOFF_MAX_MS=60000

# ========== DATABASE ==========
DATABASE_URL=postgresql://user:pass@pg.mastersindico.com.br:5432/master_sindico
DATABASE_MAX_CONNS=20
DATABASE_IDLE_TIMEOUT=300

# ========== REDIS ==========
REDIS_URL=redis://redis.mastersindico.com.br:6379/0
REDIS_PASSWORD=...
REDIS_MAX_RETRIES=3

# ========== STORAGE ==========
S3_BUCKET=mastersindico
S3_REGION=us-east-1
S3_ENDPOINT=https://s3.amazonaws.com  # ou MinIO URL
S3_ACCESS_KEY_ID=...
S3_SECRET_ACCESS_KEY=...
CLOUDFRONT_DOMAIN=d123.cloudfront.net

# ========== OBSERVABILITY ==========
SENTRY_DSN=...
OTEL_EXPORTER_OTLP_ENDPOINT=http://otel-collector:4317
GRAFANA_URL=https://grafana.mastersindico.com.br
PROMETHEUS_PORT=9090

# ========== RAILWAY / AWS ==========
RAILWAY_TOKEN=...  # Deploy
AWS_ACCOUNT_ID=...
AWS_REGION=us-east-1
ECS_CLUSTER_NAME=master-sindico-prod
ECS_SERVICE_NAME=api-server

# ========== APP CONFIG ==========
APP_ENV=production
APP_PORT=8000
APP_DOMAIN=mastersindico.com.br
CORS_ALLOWED_ORIGINS=https://app.mastersindico.com.br,https://admin.mastersindico.com.br
```

---

## Conclusão e estatísticas

Este catálogo exaustivo mapeia (conforme conclusão do relatório original):

- **13 providers** (Stripe, Zitadel, Mux, Livekit, Twilio, SES, FCM, OpenSearch, PostgreSQL, NATS, S3, Route53, CloudWatch — acrescidos nesta consolidação CloudFront, Grafana, Sentry, OpenTelemetry, Railway, AWS ECS em Parte I.10).
- **27 webhooks** (entrada: Stripe 8, Zitadel 5, Mux 5, Livekit 6, SES 5, Twilio 1, FCM 1, Livekit extra 1).
- **22 crons** (notificações assembleia, resets de quota, reindexação, sincronização).
- **42 domain events** (estruturados por bounded context com subscribers).
- **5 pipelines críticos** (signup, upload vídeo, assembleia votação, Connect Me, checkout Stripe).
- **Configuração centralizada** (secrets, endpoints, policies).

Todo código segue padrões:

- **DDD**: agregados, events, value objects.
- **Clean Architecture**: isolamento de providers em `infrastructure/`.
- **CQRS**: commands vs queries separados.
- **SAGA + UoW**: compensação e transações.
- **BeyondCorp**: zero-trust, ABAC, tenant isolation.
- **Idempotência**: webhooks com ledger.
- **Observability**: traces, logs, metrics.

---

**Documento**: Consolidado — Providers, Fluxos, Webhooks, Crons e Eventos de Domínio
**Origem**: relatório Fase 1b toolu_01XSALYmMs5xjrsWkhNWwDEg (91.7 KB / 2528 linhas no texto extraído).
**Última atualização**: 2026-04-23.
**Status**: Completo e exaustivo (fidelidade total ao relatório-fonte).
