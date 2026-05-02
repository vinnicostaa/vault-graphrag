---
title: Business Model — Master Síndico (consolidado Fase F)
type: note
tags: [product, business-model, plans, quotas, trial, stripe, master-sindico, v2, fase-8, consolidado, fase-f-pdfs-absorvidos]
source: "ADR-0032 + STATE.md D-056..D-069 Fase 8 + D-094..D-103 Fase 11 + backend/.kiro/specs/master-sindico/requirements/personas-and-plans.md + functional-matrix.md (traduzida) + 00-product/sub-produtos/01..11 + _chaos/MATRIZ FUNCIONAL MÓDULO TRIAL.pdf (Fase F absorção 2026-04-25)"
created: 2026-04-23
updated: 2026-04-25
status: consolidado-fase-f
dual_check: pending
fase_f_absorvido: true
aliases: [business-model-fase-8, business-model-fase-f, modelo-negocio-canonico]
---

# Business Model — Master Síndico

> **⚠️ Atualização Fase F (2026-04-25)**: reconciliação com `_chaos/MATRIZ FUNCIONAL MÓDULO TRIAL.pdf` (2026-03-09) confirma trial durations canônicas (Síndico 15d / Empresa 7d / Parceiro Vizinhança 30d / Morador sem trial). Quotas de Connect Me Morador Base atualizadas para **2/ano** (D-094 sobrescreve D-058/D-079). Rótulo "Morador Pagante" REMOVIDO como label vivo em UI/backend; substituído por **"Morador Plus/Pro"** (tier universal). Cross-link com [[../04-requirements/matrix-functional]] §"Matriz de Trial detalhada". PDFs absorvidos arquivados em `60-sources/master-sindico-research/client-material/pdfs/2026-03-09-matriz-funcional-{geral,trial}.pdf`.

Modelo de negócio canonizado Fase F: **planos globais Stripe-style × matriz ABAC × trial persona-aware × cotas anuais e mensais × invariantes de billing**. Substitui integralmente a versão Fase 3 que modelava planos por persona com slugs compostos (`syndic_n1/n2/n3`, `enterprise_plus/pro`, `resident_base/paid`, `local_company_standard`, `marketing_standard`) — todos abolidos pela ADR-0032 / D-057 Fase 8 / D-066 Fase 7. Rótulo "Morador Pagante" também abolido como label vivo (D-067/D-096 espírito; substituído por tier universal `(resident, plus)`).

---

## 1. Princípios estruturantes

### 1.1 Planos globais Stripe-style (ADR-0032)

Referência de modelo: Stripe. Catálogo tem `Product` (família) + `Price` (tier/cadence). Customer compra um `Price`; permissão deriva de `(Product, Price)` aplicada a regras de negócio — **não do perfil do customer**.

Adaptação canônica Master Síndico:

```
Plan (entidade global)
  id:              UUID
  tier:            enum {trial, base, plus, pro}   ← único enum de plano
  name:            string                          ← ex: "Plano Base", "Plano Plus"
  label_ui:        string (i18n)                   ← ex: "Plano Essencial"
  description:     string
  features:        JSON (feature flags)
  quotas:          JSON (cotas numéricas)
  price:           Money (int64 centavos BRL, nullable se trial)
  stripe_price_id: string (external ref)
  active:          bool

Subscription (User → Plan, Stripe-backed)
  user_id:               UUID (FK User)
  plan_id:               UUID (FK Plan)
  status:                enum {trial, active, past_due, canceled, paused, expired}
  trial_ends_at:         timestamp?
  current_period_end:    timestamp
  stripe_subscription_id: string
  stripe_customer_id:     string

Permission Matrix (ABAC — não é entidade, é cálculo)
  (user.role, plan.tier, action, resource) → allow | deny | quota_limit
```

### 1.2 Regras duras (invioláveis)

- **Slugs compostos proibidos**: zero `syndic_n1`, `enterprise_plus`, `resident_base`, `marketing_standard`, `local_company_standard` no código, banco, UI, marketing, matriz funcional ou docs.
- **Enum canônico único**: `plan_tier ∈ {trial, base, plus, pro}`.
- **Role de usuário ortogonal ao tier**: `syndic | resident | enterprise | marketing | local_company | admin` × `trial | base | plus | pro`.
- **Catálogo configurável via seed/admin** (como Stripe Dashboard): 4 tiers × N planos opcionais por tier. Adicionar plano novo = 1 row em DB + 1 row na matriz ABAC; zero código Go novo.
- **Nomes comerciais ("Síndico Profissional", "Empresa Premium") são rótulos UI** — o dado canônico persistido é sempre `plan_tier`.
- **Cross-persona mobility**: mesma subscription atravessa mudança de role (ex: morador eleito síndico preserva assinatura).

### 1.3 Agnosticismo de provedor (D-056)

Stripe é provedor escolhido hoje. Interface `IPaymentGateway` no domínio abstrai: `CreateCustomer`, `CreateSubscription`, `UpdateSubscription`, `CancelSubscription`, `ParseWebhookEvent`, `CreateCheckoutSession`, `CreateCustomerPortalSession`. Troca de provedor (ex: Pagar.me para pix nacional) = nova impl em `infrastructure/providers/`, zero change em `domain/` ou `application/`.

Outros provedores agnósticos relevantes: `IEmailProvider` (Resend), `ISMSProvider` (Twilio), `IVideoProvider` (Mux), `IStorageProvider` (MinIO dev / R2 prod), `ILiveProvider` (Livekit), `IIdentityProvider` (Zitadel), `IMessaging` (NATS).

### 1.4 Admin = role transversal, sem plano

Admin Master Síndico não consome plano (D-058 Fase 7 / D-061 Fase 8). Bypass ABAC hierarquia 1 (`if user.Role == RoleAdmin { return nil }` — ADR-0032 §Engine).

---

## 2. Modelo de receita

Master Síndico monetiza via **assinatura SaaS recorrente mensal/anual** sobre `Subscription (User, Plan)`. Cinco fluxos de entrada de receita:

| # | Fluxo | Persona | Descrição |
|---|---|---|---|
| (a) | Síndico paga trial→tier pago | `syndic` | Trial 15 dias → upgrade para `base` / `plus` / `pro` |
| (b) | Morador upgrade Plus/Pro | `resident` | Base gratuito; paga para desbloquear votação, cursos, Banco Talentos, Vizinhança (rótulo legado "Morador Pagante" abolido — D-067/D-096 espírito) |
| (c) | Empresa condominial | `enterprise` | Trial 7 dias → `plus` / `pro` |
| (d) | Parceiro Vizinhança | `local_company` | Trial 30 dias → tier único operacional `plus` (no MVP) |
| (e) | Agência de Marketing | `marketing` | **Não consome plano próprio no MVP** — opera sob grant ABAC da empresa cliente que a convidou (empresa precisa estar em tier `plus+` para delegar — `functional-matrix.md:87`). M5+: possível plano próprio sob demanda validada (dashboard multi-empresa consolidado). |

Além dos 5 fluxos acima: **Admin Master Síndico** é interno — sem cobrança.

### 2.1 Canais

- **Stripe Subscriptions** como provedor canônico (`IPaymentGateway`). Recorrência mensal/anual.
- **Métodos de pagamento**: cartão de crédito + PIX + boleto (via Stripe BR).
- **Fallback futuro** (M5+): Pagar.me para PIX nacional se necessário (abstração `IPaymentGateway` já preparada).
- **Stripe Customer Portal**: reativação, upgrade, downgrade, cancelamento self-service.

### 2.2 Receita não-assinatura (backlog M4+)

Documentada aqui para alinhamento comercial; **fora de escopo MVP**:

- **Transaction fee** em contratos Connect Me fechados (M5+, se compliance BR permitir split pagamento).
- **Certificação / selo ABRACS** (M4+ parceria).
- **LMS curso premium** (M4+ pleno).
- **Marketplace financeiro** com split pagamento (M5+ com compliance BR).
- **Marketing Link paid placement** (M4+ em teste — sponsored tags em vídeos).

---

## 3. Catálogo canônico de planos por persona × tier

Tiers universais. Nome comercial é rótulo; o dado persistido é `plan_tier` + `plan.id` (UUID).

### 3.1 Síndico (`role=syndic`)

| tier | Trial | Label comercial sugerido | Preço mensal BRL* | Quotas-chave |
|---|---|---|---|---|
| `trial` | 15 dias | — | 0 | Soft-block em ações de criação; só leitura + 1 condomínio provisional |
| `base` | — | "Síndico Essencial" | faixa R$ 30–50 | 1 condomínio; sem publicar vídeo; sem criar atividade Timeline; sem comunicado; Connect Me Síndico→Empresa 2/ano |
| `plus` | — | "Síndico Atuante" | faixa R$ 80–150 | até 15 condomínios; vídeos 4/mês; criar atividade Timeline ✅; criar comunicado ✅; Connect Me 4/ano; exportar PDF; perfil público ✅ |
| `pro` | — | "Síndico Profissional" | faixa R$ 200–400 | até 15 condomínios; vídeos 12/mês; Connect Me ilimitado; mini-bio + vídeo perfil ✅; dashboard métricas completo |

\* Preços são **variável comercial fora do canônico** — mantidos em `plans` table no banco e editáveis via admin (Seção 8).

### 3.2 Morador (`role=resident`)

| tier | Trial | Label sugerido (UI) | Preço mensal BRL* | Quotas-chave |
|---|---|---|---|---|
| `base` | — (não aplica) | "Morador Base (gratuito)" | 0 | Votar ❌; Timeline ler ✅; enquetes ✅; Connect Me Morador→Síndico **2/ano** (D-094 Fase 11 sobrescreve D-058); Vizinhança ❌; certificado LMS ❌ |
| `plus` / `pro` | — | "Morador Plus" / "Morador Pro" | faixa R$ 10–25 | Votar ✅; procuração ✅; Connect Me 4/ano; vídeo-currículo 90s (lock 90d); Banco Talentos ✅ (5 vínculos D-099); cursos LMS ✅ com certificado; Vizinhança completa |

Nota: morador pagante (= `tier ∈ {plus, pro}`) hoje opera em **tier único operacional** (`plus`) — `pro` aqui é reservado para expansão futura (ex: "Morador Premium" com features adicionais).

> **Aviso terminológico (Fase F)**: rótulo "Morador Pagante" (legado pré-D-067/D-096) **não é usado** em UI/backend canônico vivo. UI mostra "Morador Plus" / "Morador Pro" conforme `plan_tier`. O termo "Morador Pagante" só aparece em fontes históricas com tradução explícita para `(role=resident, plan_tier ∈ {plus, pro})`. Ver [[../04-requirements/matrix-functional]] §"Tradução histórica".

### 3.3 Empresa Condominial (`role=enterprise`)

| tier | Trial | Label sugerido | Preço mensal BRL* | Quotas-chave |
|---|---|---|---|---|
| `trial` | 7 dias | — | 0 | Tudo visível; bloqueia publicar vídeos, registrar execuções, gerenciar usuários |
| `plus` | — | "Empresa Plus" | faixa R$ 150–250 | Responder Connect Me ✅; vídeos 8/mês; perfil público ✅; Banco Talentos buscar ✅; delegar agência ✅; Connect Me E→E ✅ |
| `pro` | — | "Empresa Pro" | faixa R$ 400–800 | Tudo do Plus + vídeos 12/mês; registrar execução ✅; gerenciar usuários ✅; criar curso Academia ✅ (certificação); Connect Me ilimitado; neighborhood benefit via síndico |

Empresa não tem tier `base` (empresa que não paga não opera — só permanece no trial ou `suspended`).

### 3.4 Parceiro da Vizinhança (`role=local_company`)

| tier | Trial | Label sugerido | Preço mensal BRL* | Quotas-chave |
|---|---|---|---|---|
| `trial` | 30 dias | — | 0 | Tudo visível; bloqueia criar promoções, ver métricas, criar campanhas |
| `plus` (tier operacional MVP) | — | "Parceiro da Vizinhança" | faixa R$ 50–100 | Perfil hyperlocal ✅; criar cupons (lock 4h); métricas ✅; Sistema Cupons `ID_LOJA+SEQ+PALAVRA` ✅; seal "Síndico-aprovado" receptor ✅; Connect Me Síndico→Parceiro receptor ✅ |
| `pro` | — | "Parceiro Pro" (opcional) | faixa R$ 100–200 | Tudo do `plus` + Palavra-chave do dia ✅; campanha dedicada ✅; destaque em feed hyperlocal |

Parceiro opera majoritariamente em um único tier no MVP (`plus`). `pro` é opcional para destaque/gamificação.

### 3.5 Agência de Marketing (`role=marketing`)

**Não consome plano próprio no MVP** (D-060). Opera sob `DelegationGrant` ABAC da empresa cliente; a capacidade "Delegar Agência" é da empresa e exige tier `plus+` (`functional-matrix.md:87`).

Futuro M5+: plano próprio sob demanda validada — dashboard consolidado multi-empresa + add-on fee (valor por empresa gerenciada).

### 3.6 Admin Master Síndico (`role=admin`)

**Sem plano** — role interna. Bypass ABAC hierarquia 1.

---

## 4. Quotas — matriz canônica

Legenda: ✓ = ilimitado/incluso · ⚠️ = limitado · ❌ = não incluso · ∞ = ilimitado após upgrade · N = número específico/período · ⏳ = trial bloqueia.

### 4.1 Matriz consolidada (traduzida de `functional-matrix.md:28-96` sem slugs)

| Feature | `resident`.base | `resident`.plus | `syndic`.trial | `syndic`.base | `syndic`.plus | `syndic`.pro | `enterprise`.trial | `enterprise`.plus | `enterprise`.pro | `local_company`.trial | `local_company`.plus | `marketing` (delegado) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|
| Publicar vídeo institucional | ❌ | ❌ | ⏳ | ❌ | 4/mês | 12/mês | ⏳ | 8/mês | 12/mês | ❌ | ❌ | ✅ (em nome da empresa) |
| Vídeo-currículo 90s Banco Talentos | ❌ | ⚠️ (lock 90d) | n/a | n/a | n/a | n/a | n/a | n/a | n/a | ❌ | ❌ | ❌ |
| Connect Me Síndico→Empresa | ❌ | ❌ | 0 | 2/ano | 4/ano | ∞ | n/a | n/a | n/a | ❌ | ❌ | ❌ |
| Connect Me Morador→Síndico | **2/ano** (D-094 sobrescreve D-058) | 4/ano | n/a | n/a | n/a | n/a | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ |
| Connect Me Empresa→Empresa (E→E) | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | ✅ | ✅ | ❌ | ❌ | ❌ |
| Connect Me Síndico→Parceiro | ❌ | ❌ | 0 | ⚠️ | ✅ | ✅ | n/a | n/a | n/a | — (recebe) | — (recebe) | ❌ |
| Responder Connect Me | ❌ | ❌ | n/a | n/a | n/a | n/a | ✅ | ✅ | ✅ | ✅ | ✅ | ❌ |
| Votar em assembleia | **❌ (hard paywall)** | ✅ | n/a | n/a | n/a | n/a | n/a | n/a | n/a | n/a | n/a | n/a |
| Procuração | ❌ | ✅ | n/a | n/a | n/a | n/a | n/a | n/a | n/a | n/a | n/a | n/a |
| Convocar assembleia | n/a | n/a | ⏳ | ✅ | ✅ | ✅ | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ |
| Criar condomínio | n/a | n/a | ⏳ (1 soft-block) | 1 | até 15 | até 15 | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ |
| Registrar execução empresa | n/a | n/a | n/a | n/a | n/a | n/a | ⏳ | ⏳ | ✅ | n/a | n/a | ❌ |
| Criar cupom Vizinhança | ❌ | ❌ | n/a | n/a | n/a | n/a | ❌ | ❌ | ❌ | ⏳ | ✅ (lock 4h) | ❌ |
| Palavra-chave do dia | n/a | n/a | n/a | n/a | n/a | n/a | n/a | n/a | n/a | ❌ | ⚠️ | ❌ |
| Seal "Síndico-aprovado" (emissor) | n/a | n/a | ❌ | ❌ | ✅ | ✅ | n/a | n/a | n/a | — (receptor) | — (receptor) | ❌ |
| Academia — consumir curso | ❌ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ❌ | ❌ | ❌ |
| Academia — certificado emitido | ❌ | ✅ | ❌ | ❌ | ✅ | ✅ | ❌ | ✅ | ✅ | ❌ | ❌ | ❌ |
| Academia — criar curso | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | ✅ | ❌ | ❌ | ❌ |
| Banco Talentos — buscar currículos | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | ✅ | ✅ | ❌ | ❌ | ✅ |
| Vizinhança — ver parceiros bairro | ❌ | ✅ | n/a | n/a | n/a | n/a | ✅ | ✅ | ✅ | — | — | ❌ |
| Vizinhança — benefícios exclusivos | ❌ | ✅ | n/a | n/a | n/a | n/a | ❌ | ❌ | ❌ | — | — | ❌ |
| Exportar PDF timeline | n/a | n/a | ⏳ | ⏳ | ✅ | ✅ | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ |
| Delegar Agência Marketing | n/a | n/a | n/a | n/a | n/a | n/a | ❌ | ✅ | ✅ | n/a | n/a | n/a |
| Ver audit trail completa | ❌ | ❌ | ✅ próprio | ✅ próprio | ✅ próprio | ✅ próprio | ✅ próprio | ✅ próprio | ✅ próprio | ❌ | ❌ | ❌ |

**Admin Master Síndico** tem `admin bypass` em TODAS as colunas (exceto operações invioláveis como `PATCH /companies/{id}/reputation_tier` — retorna 403 mesmo para admin; reputação é determinística e alimentada por inputs observáveis — GR-028).

### 4.2 Hard limits universais

| Limite | Valor | Aplicável a |
|---|---|---|
| Condomínios ativos por síndico | **15** | `syndic` (qualquer tier, inclusive pro) |
| Tamanho de vídeo-currículo morador | ≤500 MB, ≤90 segundos | `resident.plus+` |
| Lock pós-publicação de vídeo institucional (trava trimestral) | **90 dias** (GR-029) | `syndic`, `enterprise` |
| Lock de cupom Vizinhança por estabelecimento | **4 horas** | `local_company` |
| Formato canônico do código de cupom | `ID_LOJA(5)+SEQ(3)+PALAVRA(5)` = 13 chars | `local_company` |
| Vínculos profissionais visíveis no vídeo-currículo | **5** (D-099 Fase 11 revoga D-064 — alterável via matriz funcional) | `resident.plus+` |

### 4.3 Reset periódico de quotas

| Feature | Janela de reset | Base de cálculo |
|---|---|---|
| Publicar vídeo institucional (todas personas) | **Mensal** — 1º dia do mês 00:00 UTC | `feature_usages` + Redis cache |
| Connect Me (todas direções, todas personas) | **Anual** — 1º de janeiro 00:00 UTC (Decision 10) | `feature_usages.feature_key = "connect_me"` + Redis `ms:quota:{userID}:connect_me:{year}` |
| Cupom Vizinhança (ciclo de vida) | **4h lock** por estabelecimento — não reseta; cupom expira e pode emitir novo | Redis lock + DB `coupon_issuance.expires_at` |
| Palavra-chave do dia | **Diária** — a cada 24h UTC | Job noturno renova palavra |
| Ver vídeo empresa integral | **Sem reset** (enquanto subscription ativa + contexto de votação de fornecedor Rule 4 Req 103) | Subscription.status = active |

**Invariante**: `FeatureUsage` aggregate nunca < 0. `Consume()` falha com `ErrQuotaExceeded` se `current < requested`. Decremento atômico via `SELECT ... FOR UPDATE` + UPSERT (evita antipattern A8 de race condition documentado em [[../00-product/sub-produtos/02-connect-me]] §5).

---

## 5. Trial persona-aware

Fonte: `personas-and-plans.md:63-74` + `D-058 Fase 8` (soft-block sem grace period; trial durations confirmadas por PDF MÓDULO TRIAL 2026-03-09 absorvido na Fase F: Síndico 15d / Empresa 7d / Parceiro 30d).

### 5.1 Durações

| Persona | Duração | Status durante trial | O que bloqueia |
|---|---|---|---|
| **Síndico** (`syndic`) | **15 dias** | `subscription.status = trial`, `plan_tier = trial` | Criar atividade Timeline, criar comunicados, exportar relatórios, gerenciar múltiplos condomínios (1 soft-block) |
| **Empresa** (`enterprise`) | **7 dias** | `subscription.status = trial` | Publicar vídeos, registrar execuções, gerenciar usuários adicionais |
| **Parceiro** (`local_company`) | **30 dias** | `subscription.status = trial` | Criar promoções, ver métricas, criar campanhas |
| **Morador Plus/Pro** (`resident`) | **— (não aplica)** | Base gratuito permanente OU pagamento imediato para upgrade (sem trial — rótulo legado "Morador Pagante" abolido) | — |
| **Admin** (`admin`) | — | n/a | — |
| **Agência Marketing** (`marketing`) | — (herda da empresa) | Se empresa cliente em trial, agência opera nos limites do trial | — |

### 5.2 Regras canônicas do trial

- **Cartão obrigatório no signup** (pre-auth Stripe, não debita).
- **Debita automático** no primeiro dia pós-trial se usuário não cancelou.
- **Notificações**: D-3, D-1, D+0 (charge), D+1 (falha de pagamento com retry).
- **Soft-block após expiry, SEM grace period** (D-058 legado — `personas-and-plans.md:74`). Mensagem padrão: *"Essa funcionalidade está disponível nos planos ativos da plataforma Master Síndico."*
- **Dados permanecem intocados** — não deleta, não oculta leitura; apenas remove acesso a features premium. Reativação reconecta o user preservando histórico (LGPD compliance — só apaga com right-to-erasure explícito).
- **Trial único por (user, plano)**: uma vez consumido trial de um plano, não há novo trial ao tentar mesmo plano novamente (enforce DB — `UNIQUE (user_id, plan_id, status='trial')`).
- **Um trial ativo por vez por user**: não é possível ter trial simultâneo de múltiplos planos.

### 5.3 Fluxo soft-block (Decision 4)

```
1. Middleware ABAC identifica subscription.status ∈ {expired, canceled, past_due}
2. Features premium → 403 PLAN_REQUIRED
3. Dados permanecem intocados (não deleta, não oculta leitura)
4. Usuário vê banner "Reativar assinatura" + link para Stripe Customer Portal
5. Pagamento → webhook Stripe subscription.updated → upsert Subscription
6. Cache Redis de auth invalidado
7. Próximo request: novo plan_tier na ABACContext → liberado
```

---

## 6. Upgrade / Downgrade / Cancelamento

### 6.1 Upgrade

- Usuário escolhe novo plano no Stripe Customer Portal.
- Stripe emite `subscription.updated` (evento webhook).
- Backend `IPaymentGateway.ParseWebhookEvent` → `UpsertSubscription` saga.
- Atualiza `users.plan_id` (nova row Subscription).
- **Prorata charge** calculado automaticamente pelo Stripe.
- Features ativadas imediatamente.
- Quotas resetadas para o novo patamar (mas nunca zeradas para menor se já consumidas — preserva direito adquirido).
- Cache Redis `ms:auth:{zitadelUserID}` invalidado → próximo request traz novo tier.

### 6.2 Downgrade

- Agendado para próximo ciclo de billing (`current_period_end`).
- Features atuais preservadas até fim do ciclo pago.
- Após fim do ciclo: Stripe emite `subscription.updated` com novo `price_id`; webhook atualiza Subscription.
- Quotas do tier inferior aplicadas a partir do novo ciclo.
- Se usuário atingiu limite superior (ex: morador tier=plus assinou 4/ano Connect Me e usou 3 antes do downgrade): preservar o consumido; novas requisições cortadas pelo limite do tier novo.

### 6.3 Cancelamento

- Stripe Customer Portal → cancel at period end (default) OU immediate (caso especial).
- `subscription.status = canceled` ao fim do período pago.
- **Soft-block** aplicado (não revoga leitura; bloqueia ações premium).
- Dados preservados indefinidamente (LGPD art. 7º XII — conservação de dados por obrigação legal/contratual, 5 anos; ver [[../00-product/sub-produtos/09-compliance]]).
- **Re-ativação**: nova Subscription reconecta o user ao mesmo tenant preservando histórico.

### 6.4 Cross-persona mobility (ADR-0032)

Se Morador vira Síndico (eleição em assembleia) OU vice-versa:
- `user.role` muda no Zitadel.
- `plan_id` **permanece**.
- Quotas recalculadas pela nova matriz `(novo_role, plan.tier)`.
- Se novo role exige plano mínimo (ex: Síndico precisa ao menos `plus` para criar atividade Timeline), backend aplica **soft-block parcial** até upgrade voluntário.
- Webhook Stripe **não** é chamado (mudança de role não é mudança de plano).

---

## 7. Upsell / Cross-sell — triggers canônicos

Estratégias documentadas para alinhamento time comercial + produto.

### 7.1 Síndico `trial` → `base/plus/pro`

- **Trigger 1**: atingiu quota Connect Me anual (2/ano em `base`) → CTA upgrade inline ("Quer enviar mais Connect Me? Upgrade para Plus.")
- **Trigger 2**: tentou criar comunicado oficial em `base` → paywall + upgrade.
- **Trigger 3**: tentou criar atividade Timeline em `base` → paywall + upgrade.
- **Trigger 4**: tentou publicar vídeo institucional em `base` → paywall + upgrade.
- **Trigger 5**: quer gerenciar 2º condomínio → requer `plus+`.

### 7.2 Morador `base` → `plus`

- **Trigger 1**: tentou votar em assembleia → paywall + upgrade (hard paywall mais forte da plataforma).
- **Trigger 2**: tentou enviar 3º Connect Me Morador→Síndico (limite 2/ano em base — D-094) → paywall + upgrade (Plus tem 4/ano).
- **Trigger 3**: tentou acessar vídeo empresa integral (só preview 25% em base) → paywall.
- **Trigger 4**: tentou concluir curso LMS e receber certificado → paywall.
- **Trigger 5**: tentou ver parceiros Vizinhança / resgatar cupom → paywall.
- **Trigger 6**: tentou publicar vídeo-currículo Banco Talentos → paywall.

### 7.3 Empresa `plus` → `pro`

- **Trigger 1**: quer iniciar Connect Me E→E (Plus permite apenas responder/limitado) → upgrade.
- **Trigger 2**: quer gerenciar múltiplos usuários operadores → exige `pro`.
- **Trigger 3**: quer validar execução de seu próprio serviço → exige `pro`.
- **Trigger 4**: quer criar curso Academia (certificação profissional) → exige `pro`.
- **Trigger 5**: quer vídeos 12/mês (vs 8/mês em Plus) → upgrade.

### 7.4 Parceiro `trial` → `plus`

- **Trigger 1**: tentou emitir cupom após trial → paywall.
- **Trigger 2**: tentou ver métricas → paywall.

### 7.5 Cross-sell cross-persona

- **Síndico → Empresa**: síndico profissional que vira responsável legal de empresa condominial (administradora). Preserva conta; adiciona role `enterprise` em tenant separado.
- **Morador → Síndico**: morador eleito em assembleia migra para tenant condominial; preserva Subscription se compatível; soft-block se role exige tier superior.

---

## 8. Pricing — variáveis de negócio (FORA do canônico)

Ranges aproximados **2026 BR** — confirmação contínua com time comercial:

| Plano | tier | Range mensal BRL | Fonte |
|---|---|---|---|
| Síndico Essencial | `base` | R$ 30–50 | variável comercial |
| Síndico Atuante | `plus` | R$ 80–150 | variável comercial |
| Síndico Profissional | `pro` | R$ 200–400 | variável comercial |
| Morador Plus | `plus` | R$ 10–25 | variável comercial |
| Empresa Plus | `plus` | R$ 150–250 | variável comercial |
| Empresa Pro | `pro` | R$ 400–800 | variável comercial |
| Parceiro Vizinhança | `plus` | R$ 50–100 | variável comercial |

**Regra dura**: **NÃO fixar pricing no spec**. Valores são comerciais, testáveis via A/B Stripe, mudam ao longo do tempo. Manter em `plans` table no banco; editáveis via painel admin. Specs mantêm apenas **estrutura** (tier universal + quotas + features).

---

## 9. Procedimento canônico para instanciar plano novo (ADR-0032 §Apêndice)

1. Admin Master Síndico abre **Stripe Dashboard** → cria Price novo (ex: `price_1ABCxyz`) com R$ 49,90/mês.
2. Admin registra `Plan` novo no painel da plataforma:

```sql
INSERT INTO plans (id, tier, name, label_ui, features, quotas, stripe_price_id, active)
VALUES (gen_random_uuid(),
        'plus',
        'Plano Plus Síndico',
        'Plano para síndicos atuantes',
        '{"connect_me": true, "video_upload": true, "timeline_activity": true, "announcement_create": true}'::jsonb,
        '{"connect_me_year": 4, "video_upload_month": 4, "condominiums_max": 15}'::jsonb,
        'price_1ABCxyz',
        true);
```

3. ABAC matrix já conhece `tier=plus` — permissões derivam automaticamente da regra `(role, plan.tier)`.
4. UI mostra no catálogo quando user logado (filtrado por role compatível).
5. **Zero código Go novo.**

---

## 10. Invariantes de billing (invioláveis)

| # | Invariante | Enforcement |
|---|---|---|
| B-1 | **Money em centavos BRL** (`int64`), nunca float | Type system Go (`type Money int64 // cents BRL`); lint rule |
| B-2 | **Webhook Stripe idempotente**: `event.id` único | `StripeWebhookEvent` table UNIQUE constraint em `event_id`; reprocesso seguro (ADR-0027 webhook-idempotency-db) |
| B-3 | **Trial único por (user, plano)** | DB UNIQUE `(user_id, plan_id, status='trial')` |
| B-4 | **Um trial ativo por vez por user** | Backend valida antes de criar Subscription |
| B-5 | **Soft-block sempre (sem grace period)** — nunca revoga imediato (exceto fraude confirmada) | ABAC middleware retorna 403 `PLAN_REQUIRED`; não deleta dados |
| B-6 | **PII em logs billing proibido** | email/CPF/cartão mascarados (`joao***@dominio.com`, `***.***.***-**`); ADR-0008 zerolog + hmac-keyed-hash (ADR-0028) |
| B-7 | **Prorata calculado pelo Stripe, nunca no backend** | `IPaymentGateway.UpdateSubscription` delega para Stripe API |
| B-8 | **Quotas decrementam no ato da ação, não no rascunho** | Ex: `ConnectMeRequest` decrementa no `Submit()`, não no `CreateDraft()` |
| B-9 | **FeatureUsage nunca < 0** | Aggregate invariant + `Consume()` falha antes de decrementar |
| B-10 | **Reputação não é modificável via PATCH** mesmo por admin | GR-028 — endpoint `PATCH /companies/{id}/reputation_tier` sempre 403; tier é função pura dos inputs |
| B-11 | **Cross-persona mobility preserva Subscription** | `user.role` mutável; `plan_id` preservado; quotas recalculadas por nova matriz |
| B-12 | **Cancelamento preserva dados indefinidamente** (LGPD art. 7º XII — retenção contratual 5 anos) | Soft-delete apenas; right-to-erasure via fluxo dedicado LGPD ([[../00-product/sub-produtos/09-compliance]]) |
| B-13 | **Admin bypass não pula billing** | Admin não consome plano; mas operações financeiras (refund, dispute) exigem MFA + audit obrigatório |
| B-14 | **Stripe Customer Portal é self-service único** | Backend nunca altera plano unilateralmente; sempre via webhook Stripe |
| B-15 | **Slug-free** — código não pode conter strings hardcoded `syndic_n1`, `enterprise_plus`, etc. | ADR-0032; lint rule + code review |

---

## 11. Auditoria e LGPD

- **Toda transação billing é auditada**: `audit_log_entries` append-only (5 anos retenção — LGPD).
- **LGPD logs**: revelações de PII via Connect Me vinculadas a subscription (quem tinha plano ativo no momento da revelação).
- **Stripe webhooks são imutáveis**: `StripeWebhookEvent` table preserva payload raw para reprocessamento (ADR-0027).
- **Right-to-erasure** (LGPD art. 18 VI): fluxo dedicado em `09-compliance.md` — não afeta billing histórico (5 anos legal retention), mas mascara PII em logs e apaga dados pessoais não contratualmente exigidos.

---

## 12. Roadmap do Billing

| Marco | Escopo billing |
|---|---|
| **M1 (2026-05-08)** | Stripe Subscriptions mensal + webhook idempotente + trial persona-aware + soft-block + Customer Portal |
| **M2 (2026-06-20)** | Cotas por feature (Connect Me anual + vídeos mensais) + FeatureUsage aggregate + Redis cache + prorata upgrade |
| **M3 (2026-07-20)** | Painel admin Plans + dashboard MRR + churn + dunning (falhas de pagamento retry) |
| **M4+** | Split payment Connect Me fee (se compliance BR) + Marketplace financeiro + Pagar.me adapter (fallback) |
| **M5+** | Agência Marketing plano próprio (multi-empresa consolidado) se demanda validar |

---

## 13. Referências

- **ADR canônica**: [[../02-architecture/adr/0032-global-plans-stripe-style]]
- **Decisões vivas**: `STATE.md` Fase 8 D-057 (planos globais), ~~D-058 (Morador Base 0/ano)~~ revogada por **D-094 Fase 11 = 2/ano**, D-056 (agnosticismo provedor), D-061 (admin role transversal); Fase 7 D-066/D-067 (abolir N1/N2/N3); **Fase 11** D-094 (Morador Base 2/a), D-096 (materializa abolição N1/N2/N3 + slugs), D-099 (Banco Talentos 5 vínculos), D-097 (Lives Admin + Empresa Pro), D-103 (2 scores Governança+Compliance), D-115 (11 sub-produtos), D-126 (TenantKinds reduzidos a 2).
- **Matriz funcional (backend)**: `Development/backend/.kiro/specs/master-sindico/requirements/functional-matrix.md` (pendente DT-004 reescrita sem N1/N2/N3).
- **Personas e planos (backend)**: `Development/backend/.kiro/specs/master-sindico/requirements/personas-and-plans.md`.
- **Sub-produto Connect Me** (quotas por direção): [[../00-product/sub-produtos/02-connect-me]] §5.
- **Sub-produto Vizinhança** (Sistema Cupons): [[../00-product/sub-produtos/08-vizinhanca]] §4.
- **Sub-produto Reputação** (GR-028 imutabilidade via PATCH): [[../00-product/sub-produtos/03-reputacao]] §5.
- **ADRs relacionadas**:
  - ADR-0004 stripe-payment-gateway
  - ADR-0027 webhook-idempotency-db
  - ADR-0028 lgpd-keyed-hmac
  - ADR-0014 one-device-policy
  - ADR-0026 admin-mfa-m1
- **Links**: [[vision]], [[personas]], [[portfolio-de-produtos]], [[glossary]], [[../04-requirements/functional/billing]].
