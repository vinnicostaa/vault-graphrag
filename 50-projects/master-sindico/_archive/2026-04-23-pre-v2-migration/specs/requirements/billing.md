---
title: Billing — Planos, Assinaturas, Trial e Quotas
version: 1.0
status: stable
type: requirement
domain: billing
tags: [billing, stripe, subscription, trial, quota, paywall]
---

# Billing — Planos, Assinaturas, Trial e Quotas

Domínio de faturamento e controle de acesso a funcionalidades por plano. Integração com Stripe, suporte a trials por persona, quotas de uso e soft-block sem grace period.

## Req 4 — Billing e Assinaturas

**Stack:** Stripe (`IPaymentGateway`) com webhook de eventos; suporte a mensal e anual.

### Requisitos MUST

- Planos estruturados: `trial` → `base` → `plus` → `pro`
- Integração Stripe via `IPaymentGateway` (agnóstico — troca de provider = nova implementação)
- Frequências: mensal e anual
- Upgrade/downgrade com billing proporcional (Stripe faz automaticamente)
- Evento `SubscriptionExpired` → bloqueia premium, **dados permanecem** (soft-block — Decision 4)
- Webhook Stripe: `payment_intent.succeeded`, `subscription.updated`, `subscription.deleted`
- Trial: tudo visível, ações estratégicas bloqueadas
- Mensagem de bloqueio: _"Essa funcionalidade está disponível nos planos ativos da plataforma Master Síndico."_
- Paywall: **soft block** — dados intactos, acesso a premium bloqueado (Decision 4)
- **Sem grace period** — expirou = bloqueado imediatamente (Decision 4)

### Requisitos SHOULD

- Customer Portal Stripe integrado (auto-gerenciamento de pagamentos)
- Downgrades com confirmação de entendimento (Sprint 3+)

### Notas de implementação

- Implementação atual: `internal/modules/billing/` com domain + application + handlers
- Webhook Stripe em `POST /webhooks/stripe` (fora de `/api/v1`, sem middleware Authenticate)
- Validação de HMAC signature público (security.md)
- Cache de planos em Redis com TTL 1h
- Soft-block middleware intercepta requisição antes do handler se `plan_tier = expired/canceled`

---

## Req 4.1 — Trial por Persona

Duração e funcionalidades bloqueadas durante trial.

| Persona | Duração | Comportamento bloqueado |
|---------|---------|---|
| **Síndico** | 15 dias | Criar atividades timeline, criar comunicados, exportar relatórios, gerenciar múltiplos condomínios |
| **Empresa** | 7 dias | Publicar vídeos, registrar execuções, gerenciar usuários |
| **Parceiro da Vizinhança** | 30 dias | Criar promoções, ver métricas, criar campanhas |
| **Morador Pagante** | — | Sem trial — pagamento imediato ou permanece Base |

### Requisitos MUST

- Trial inicia automaticamente no primeiro login (UPSERT de subscription)
- Contagem regressiva exata por persona (em horas, não em dias redondos)
- Expirando em 24h: banner "Trial expira amanhã"
- Expirou: soft-block em features premium
- Reset: sem reset, transição para paid ou volta a Free

### Notas de implementação

- `subscriptions.trial_started_at`, `trial_days` → cálculo em SQL: `trial_started_at + interval '{trial_days} days'`
- Feature flag por persona para rollout gradual
- Ver `personas-and-plans.md` para mapeamento completo

---

## Req 5 — Onboarding

Fluxo pós-registro com auto-save de progresso em Redis, identificação de persona.

### Personas e telas obrigatórias

- **Morador (4 telas):** Buscar condo por ID → Registrar unidade → Termo LGPD → Foto
- **Síndico (6 telas):** Dados pessoais → Tipo síndico → Governance markers → Criar condo → Mandato → Empresa síndica
- **Empresa (7 telas):** Dados → CNPJ → Categorias → Compliance markers → Logo/fotos → Termos → Dashboard
- **Parceiro (4 telas):** Dados estabelecimento → Endereço/CEP → Logo/descrição → Dashboard

### Requisitos MUST

- Auto-save de progresso em Redis `onboarding:{user_id}:{step}` (não bloqueia fluxo)
- Identificação de persona via SearchParams ou seleção no primeiro acesso
- Validação de dados obrigatórios antes de avançar
- Termo LGPD **obrigatório** (regra de ouro 10)
- Foto de perfil: upload simples (via `IStorageProvider`)
- Persistência: após conclusão, migrar do Redis para tabelas de profile (institutional, commercial)

### Requisitos SHOULD

- Skip de telas já completadas (reload durante onboarding)
- Analytics: tracking de drop-off por etapa

### Notas de implementação

- Estado salvo em Redis com TTL 30 dias
- Telas validadas no frontend (Zod); backend valida novamente no POST
- Campos extras por persona armazenados em `profiles` tables por bounded context

---

## Req 6 — Acesso Baseado em Plano

**Matriz de features** mapeando planos → funcionalidades.

### Requisitos MUST

- Feature matrix: `plans` × `features` × `permissions` (ver `functional-matrix.md` para tabela completa)
- Quotas: Connect Me (anual), uploads de vídeo (mensal), storage total
- Reset de quotas: anual para Connect Me (1º de janeiro), mensal para uploads (1º do mês)
- Feature flags para rollout gradual de nova feature → lançamento → sunset

### Quotas especificamente

#### Connect Me (por **ano** — Decision 10)

| Persona/Plano | Cota anual |
|---|---|
| Morador Base | 2/ano |
| Morador Pagante | 4/ano |
| Síndico N1 | 2/ano |
| Síndico N2 | 4/ano |
| Síndico N3 | ilimitado |
| Empresa Plus | (não aplicável — E→E só Plus/Pro) |
| Empresa Pro | conforme contrato |

Tracking: `feature_usages` table + Redis `ms:quota:{userID}:connect_me:{year}` (cache com expiração 1º de janeiro).

#### Vídeos Institucionais (mensal)

| Plano | Uploads/mês |
|---|---|
| Síndico N1 | 4 |
| Síndico N2 | 8 |
| Síndico N3 | 12 |
| Empresa Plus | 8 |
| Empresa Pro | 12 |

Trava trimestral adicional: cada vídeo específico só pode ser **substituído** após 90 dias (Req 29, rule 7).

### Requisitos SHOULD

- Pré-cálculo de quotas em job noturno (Sprint 3+)
- Dashboard de uso de quotas para usuário (Sprint 2)

### Notas de implementação

- Tabelas: `plans`, `plan_features`, `feature_usages`
- Middleware ABAC consulta cache Redis antes do handler
- Decremento de quota em transação junto com criação de recurso (atomicidade)
- Exaustão de quota → 403 `QUOTA_EXCEEDED`

---

## Invariantes de billing

1. `users.plan_id` aponta a subscription ativa ou NULL (soft-block não apaga)
2. `subscriptions` é append-only — novo registro a cada mudança (nunca UPDATE)
3. Trial nunca volta após conversão para paid
4. Valores monetários sempre em centavos (`INTEGER` no PostgreSQL, `int64` em Go) — nunca float (T10)

---

## Referências cruzadas

- **Personas e Planos**: `personas-and-plans.md` — mapeamento completo
- **Functional Matrix**: `functional-matrix.md` — matriz feature × plano × ação
- **Product Overview**: `product-overview.md` — decisões 2, 4, 10
- **Institutional + Commercial**: profiles extensíveis em bounded contexts respectivos
- **Content**: Req 29 para quotas de vídeo e trava trimestral
