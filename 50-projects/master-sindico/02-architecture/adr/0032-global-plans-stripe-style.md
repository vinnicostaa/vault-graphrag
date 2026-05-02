---
title: "ADR-0032 — Planos globais Stripe-style, personas carregam matriz de permissão"
type: adr
tags: [adr, architecture, billing, abac, plans, master-sindico, v2]
source: conversa usuário 2026-04-23; matriz funcional Development/backend/.kiro/specs/master-sindico/requirements/functional-matrix.md
created: 2026-04-23
updated: 2026-04-23
status: accepted
supersedes: [none, but invalidates slugs pattern usado em v2 Fase 3]
---

# ADR-0032 — Planos globais (Stripe-style), personas carregam matriz de permissão

## Status

**Accepted** — 2026-04-23.

## Contexto

A modelagem anterior da v2 tratou planos como **específicos por persona**:

- Síndico: `syndic_n1`, `syndic_n2`, `syndic_n3`
- Empresa: `enterprise_plus`, `enterprise_pro`
- Morador: `resident_base`, `resident_paid`
- Parceiro: `local_company_standard`
- Agência: `marketing_standard`

Slugs compostos viraram identificadores no código, UI, matriz funcional e docs. O usuário (product owner) identificou que **esse modelo quebrou a regra de negócio do produto**: planos por persona geram combinatória, acoplam billing à taxonomia de personas, impossibilitam cross-sell simples, dificultam migração entre tiers quando a persona muda (ex: morador vira síndico preservando assinatura).

Exemplos concretos de quebra:
- Se morador vira síndico (eleito em assembleia), não consegue "levar" a assinatura porque o slug muda
- Adicionar nova persona (ex: auditor externo) exige reescrita da enumeração de planos
- Feature flag "plus" significa coisas diferentes em `enterprise_plus` e `syndic_n2` — sobrecarga semântica
- A matriz funcional precisa de colunas por slug (multiplica cardinalidade)

A referência correta é a **Stripe**: catalog tem `Product` (família) + `Price` (tier/cadence). Customer compra um `Price`, a permissão deriva de `(Product, Price)` aplicada a regras de negócio — **não do perfil do customer**.

## Decisão

### Modelo canônico adotado

```
Plan (entidade global)
  id:          UUID
  tier:        enum {trial, base, plus, pro}   ← único enum de plano
  name:        string                           ← ex: "Plano Base", "Plano Plus"
  label_ui:    string (i18n)                   ← ex: "Plano Essencial"
  description: string
  features:    JSON (feature flags)
  quotas:      JSON (cotas numéricas)
  price:       Money (int64 centavos BRL, nullable se trial)
  stripe_price_id: string (external ref)
  active:      bool

Subscription (User → Plan, Stripe-backed)
  user_id:   UUID (FK User)
  plan_id:   UUID (FK Plan)
  status:    enum {trial, active, past_due, canceled, paused, expired}
  trial_ends_at: timestamp?
  current_period_end: timestamp
  stripe_subscription_id: string

Permission Matrix (ABAC — não é entidade, é cálculo)
  (user.role, plan.tier, action, resource) → allow | deny | quota_limit
```

### Regra dura: slugs compostos proibidos

- Não existem `syndic_n1`, `enterprise_plus`, `resident_base` etc. no código, banco, UI, marketing, matriz funcional, docs.
- Enum canônico único: `plan_tier ∈ {trial, base, plus, pro}`.
- Catálogo de planos é **configurável via seed/admin** (como Stripe Dashboard): 4 tiers × N planos opcionais por tier.
- Role de usuário (`syndic | resident | enterprise | marketing | local_company | admin`) é **ortogonal** ao tier de plano.

### Matriz funcional reescrita

Colunas da matriz funcional deixam de ser `N1 | N2 | N3 | Plus | Pro | Base | Pagante` e passam a ser **`Trial | Base | Plus | Pro` universais**. Features e quotas são filtradas por `role` na linha:

```
Feature                    | Trial | Base | Plus | Pro   | Aplicável a role
---------------------------|-------|------|------|-------|-----------------
Criar condomínio           | ⏳    | ❌   | ✅   | ✅    | syndic
Publicar vídeo institucional| ⏳   | ❌   | 4/mês| 12/mês| syndic, enterprise
Connect Me → Empresa       | 0/ano | 0/ano| 4/ano| ∞     | syndic, resident
Votar assembleia           | ✅    | ✅   | ✅   | ✅    | resident
...
```

Cada linha adiciona coluna "aplicável a role" quando a feature só existe pra certas personas. Feature universal (ex: "ver timeline") omite essa coluna.

### Upgrade/downgrade entre personas

Se Morador vira Síndico (eleição) ou vice-versa:
- **Subscription preserva**: `user.role` muda; `plan_id` permanece; quotas recalculadas pela nova matriz `(novo_role, plan_tier)`.
- Se o novo role exige plano mínimo (ex: Síndico precisa pelo menos `plus`), webhook Stripe não é chamado — backend aplica soft-block até upgrade voluntário.

### ABAC engine (como derivar permissão)

```go
func (abac *Engine) Allow(ctx context.Context, action, resource string) error {
    user := ctx.User()
    plan := ctx.Subscription().Plan()
    
    // 1. Admin bypass (role=admin curto-circuita)
    if user.Role == RoleAdmin { return nil }
    
    // 2. Check matrix (role × tier × action)
    entry := matrix.Lookup(user.Role, plan.Tier, action, resource)
    if entry == nil { return ErrForbidden("NO_MATCH") }
    if !entry.Allowed { return ErrForbidden("ACTION_DISALLOWED") }
    
    // 3. Check quota (se feature tem cota)
    if entry.Quota > 0 {
        used, err := quotaRepo.GetUsage(user.ID, entry.QuotaKey, plan.ID)
        if err != nil { return err }
        if used >= entry.Quota { return ErrQuotaExceeded }
    }
    
    // 4. Check tenant scope (se action é tenant-scoped)
    if entry.RequiresTenant && !ctx.HasTenant() {
        return ErrForbidden("NO_TENANT")
    }
    
    return nil
}
```

## Consequências

### Positivas

- **Cardinalidade reduzida**: de ~8 slugs pra 4 tiers universais
- **Cross-persona mobility**: mesmo plano atravessa mudança de role
- **Config simples**: admin MS cria plano via Stripe Dashboard + seed da matriz; zero código novo
- **ABAC engine único**: matriz de permissão é dado, não código
- **Alinhamento com Stripe**: práticas padrão de billing SaaS
- **Novos planos sem redeploy**: adicionar `Plano Enterprise Pro+` = 1 row em DB + 1 row na matriz

### Negativas

- **Matriz funcional mais densa**: cada linha tem coluna "role aplicável"
- **UI pode ficar mais verbosa**: "Plano Plus para Síndicos" em vez de "Síndico N2"
- **Migração de artefatos v2 destilados**: arquivos em `00-product/`, `01-domain/aggregates/`, `04-requirements/` com slugs precisam ser reescritos (ver DT-005)
- **Matriz funcional oficial (backend/.kiro/specs/.../functional-matrix.md, v1.0 stable 2026-04-22)** precisa ser reescrita — task separada pro time backend (DT-004)

### Neutras

- `Plan.name` pode continuar usando nomes comerciais ("Síndico Profissional", "Empresa Premium"), mas o **dado de tier** é universal
- Marketing pode chamar publicamente como quiser; no dado fica `plan_tier`

## Alternativas consideradas

### Alt-1: Manter slugs compostos (status quo v2 Fase 3)

- Vantagem: zero refactor na v2
- Desvantagem: mantém a causa-raiz que quebrou o produto
- **Rejeitado** (decisão explícita do usuário)

### Alt-2: Tier por persona (matrix separada por role)

- Ex: `syndic_plan_tier = {trial, n1, n2, n3}`, `enterprise_plan_tier = {trial, plus, pro}`
- Vantagem: respeita nomenclatura histórica
- Desvantagem: duplica enum; cross-persona mobility quebrada
- **Rejeitado**

### Alt-3: Global tier + hierarquia dentro (plus < premium < enterprise)

- Ex: `tier ∈ {trial, base, plus, premium, enterprise, elite}`
- Vantagem: granularidade maior
- Desvantagem: excesso de tiers pro MVP (YAGNI)
- **Rejeitado** (começar simples; expandir se validar demanda)

## Impacto na v2 existente

- **ADR-0031 (email-provider)**: mantido.
- **ADR-0027 (webhook-idempotency-db)**: mantido.
- **Arquivos que referenciam `syndic_n1/n2/n3`, `enterprise_plus/pro`, `resident_base/paid`, `local_company_standard`, `marketing_standard`**: reescrever removendo slugs + trocar por `(role, plan_tier)`. Lista:
  - `00-product/personas.md`
  - `00-product/business-model.md`
  - `00-product/portfolio-de-produtos.md`
  - `00-product/glossary.md` (entradas Plano, Slug)
  - `01-domain/aggregates/Plan.md`
  - `01-domain/aggregates/Subscription.md`
  - `01-domain/aggregates/FeatureUsage.md`
  - `01-domain/ubiquitous-language.md`
  - `04-requirements/functional/billing.md`
  - `04-requirements/matrix-functional.md`
  - `04-requirements/registration-data.md` (trial por persona ainda existe, mas sem slug)
- **Matriz funcional oficial (Development)**: DT-004, reescrita separada.
- **Código Go**: `backend/internal/modules/billing/` pode ter enums contaminados; auditar em Fase 8 backend sub-projeto.

## Referências

- Conversa usuário 2026-04-23 (transcrição na sessão): "planos por persona quebrou a regra de negócio do produto"
- Matriz funcional canônica: `/home/vinni/Documentos/Projetos/Clientes/Joao/MasterSindico/Development/backend/.kiro/specs/master-sindico/requirements/functional-matrix.md`
- Stripe Products & Prices: https://stripe.com/docs/products-prices/overview
- ADR anterior: `ADR-0031 email-provider-resend-m1` (mantido)
- STATE decisões: D-066 (decisão crítica), D-067 (abolir N1/N2/N3), D-058 (Admin MS role privilegiada)

---

## Apêndice: Como instanciar um plano novo (procedimento)

1. Admin MS abre **Stripe Dashboard** → cria Price novo (ex: `price_1ABCxyz`) com R$49,90/mês.
2. Admin MS (no painel da plataforma) registra `Plan` novo:
   ```sql
   INSERT INTO plans (id, tier, name, label_ui, features, quotas, stripe_price_id)
   VALUES (gen_random_uuid(), 'plus', 'Plano Plus Síndico',
           'Plano para síndicos atuantes',
           '{"connect_me": true, "video_upload": true}'::jsonb,
           '{"connect_me_year": 4, "video_upload_month": 8}'::jsonb,
           'price_1ABCxyz');
   ```
3. ABAC matrix já conhece `tier=plus` — permissões derivam automaticamente.
4. UI mostra no catálogo quando user logado (filtrado por role).
5. Zero código Go novo.
