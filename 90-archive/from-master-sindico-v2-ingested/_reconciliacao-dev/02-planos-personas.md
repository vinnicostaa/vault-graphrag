---
title: Reconciliação — Planos + Personas canônicos vs v2 legado
type: note
tags: [reconciliation, planos, personas, canonico, master-sindico, v2, fase-7]
source: Development/backend/.kiro/specs/master-sindico/* + Content/02-Jornadas/Onboarding-por-Persona + Content/regras-de-negocio/regras-canonicas
created: 2026-04-23
updated: 2026-04-23
---

# Reconciliação — Planos + Personas canônicos vs v2 legado

> **Origem**: Agent Explore (Fase 7c) leu `Development/backend/.kiro/specs/master-sindico/*`, `Development/backend/internal/modules/{billing,identity,institutional}/`, `Content/04-Listas-Mestre/Enums-Consolidados.md`, `Content/02-Jornadas/Onboarding-por-Persona.md`, `Content/regras-de-negocio/regras-canonicas.md`. Escopo: destilar sistema de **planos + personas REAIS** e identificar divergências vs v2.

---

## § 1. SISTEMA DE PLANOS CANÔNICO (trial/base/plus/pro UNIFICADO)

**ESTRUTURA CONFIRMADA — 4 TIERS + PERSONAS ESPECÍFICAS:**

```
tier=trial  → 15d (síndico/empresa), 30d (parceiro), — (morador)
tier=base   → gratuito (morador base), ou pago mínimo (síndico N1-equiv)
tier=plus   → intermediário (N2-equiv síndico, Plus empresa)
tier=pro    → premium (N3-equiv síndico, Pro empresa)
```

**Mapeamento canônico (Development backend `.kiro/specs`)**:

| Persona | Plano Slug | Nome | Trial | Preço | Notas |
|---|---|---|---|---|---|
| **Morador** | `resident_base` | Base (gratuito) | — | R$ 0 | Sem upgrade; acesso via síndico |
| **Morador** | `resident_paid` | Pagante | — | ~R$ 9,90/mês | Assinatura direta; sem trial |
| **Síndico** | `syndic_n1` | N1 Aprender | 15d | — | Consumo/aprendizado; Connect Me 2/ano; vídeos: ❌ |
| **Síndico** | `syndic_n2` | N2 Atuar | 15d | — | Operacional; 4 vídeos/mês; 4 Connect Me/ano |
| **Síndico** | `syndic_n3` | N3 Consolidar | 15d | — | Professional; 12 vídeos/mês; ∞ Connect Me |
| **Empresa** | `enterprise_plus` | Plus | 7d | — | Responde Connect Me; 8 vídeos/mês; ❌ E→E |
| **Empresa** | `enterprise_pro` | Pro | 7d | — | Plus + Connect Me E→E; 12 vídeos/mês |
| **Parceiro** | `local_company_standard` | Standard | 30d | — | Cupons; reputação; sem vídeos |
| **Agência** | `marketing_standard` | Standard (delegado) | Compartilha empresa | — | Actor; sem subscription própria |

**INVARIANTES CANÔNICOS**:
- `plan_tier ∈ {trial, base, plus, pro}` — tipos canônicos (ver `Plan.Tier` em Go)
- `plan_role ∈ {syndic, resident, enterprise, marketing, local_company}` — role em plano (não é `user.role` direto)
- Trial nunca reseta (Decision 4 legado, `subscriptions` append-only)
- Soft-block sem grace period: expirou = bloqueado imediatamente
- Money em centavos INT64 (GR-032)
- Quotas anual (Connect Me: reset 1 jan) e mensal (vídeos: reset 1º mês)

---

## § 2. MAPEAMENTO v2 LEGADO → CANÔNICO

| v2 Legado | Canônico | Status | Divergência |
|---|---|---|---|
| **Morador Base** | `resident_base` tier=base | ✅ Alinhado | — |
| **Morador Pagante** | `resident_paid` tier=base | ✅ Alinhado | — |
| **Síndico N1** | `syndic_n1` tier=base | ⚠️ Parcial | v2 chama "N1 Aprender"; canônico não usa "N1/N2/N3" como `tier_name`, usa slugs `syndic_n1/n2/n3` |
| **Síndico N2** | `syndic_n2` tier=plus | ⚠️ Divergência | v2 trata como tier separado; canônico alinha como `plus` (Plus = N2) |
| **Síndico N3** | `syndic_n3` tier=pro | ⚠️ Divergência | v2 trata como tier separado; canônico alinha como `pro` (Pro = N3) |
| **Empresa Plus** | `enterprise_plus` tier=plus | ✅ Alinhado | — |
| **Empresa Pro** | `enterprise_pro` tier=pro | ✅ Alinhado | — |
| **Parceiro Vizinhança** | `local_company_standard` tier=base | ✅ Alinhado | — |
| **Agência Marketing** | `marketing_standard` (actor delegado) | ✅ Alinhado | — |

**ACHADO CRÍTICO**: v2 trata "N1/N2/N3" como **tiers EXCLUSIVOS** da persona síndico. Canônico trata como **nomes de planos** (slugs), mapeados a **tiers unificados trial/base/plus/pro**. Síndico N2 = Empresa Plus = `tier=plus`, não um tier intermediário chamado "N2".

---

## § 3. PERSONAS COMPLETAS (CANÔNICO)

### 1. Síndico

**Sub-tipos encontrados no código**:
- Eleito (morador eleito síndico)
- Profissional (pode gerenciar múltiplos condomínios)
- Interino (temporário)

**Roles operacionais** (em `identity.md` + `institutional.md`):
- `principal` — síndico titular (obrigatório por condomínio)
- `auxiliary` — síndico auxiliar (até 2 por condomínio)
- `assistant` — assistente (até 3; sem voto)

**Enum canônico no Go**:
```go
// user_role_type (sub-contexto operacional)
const (
  RoleTypePrincipal     = "principal"      // síndico principal ou titular unidade
  RoleTypeAuxiliary     = "auxiliary"      // síndico auxiliar
  RoleTypeAssistant     = "assistant"      // assistente (sem voto)
  RoleTypeTitular       = "titular"        // titular de unidade (morador)
  RoleTypeDependent     = "dependent"      // dependente (familiar)
  RoleTypeAdministrator = "administrator"  // admin empresa
  RoleTypeOperator      = "operator"       // operador empresa (sem gestão)
)
```

**Planos aplicáveis**: trial/base (`syndic_n1`)/plus (`syndic_n2`)/pro (`syndic_n3`)

**Dados cadastro obrigatórios**: CPF, nome, email prof, telefone, data nasc, endereço prof, CEP, cidade/estado, tipo (eleito/prof/interino), anos experiência, número condomínios.

**Limite duro**: 15 condomínios por síndico (Decision 11)

### 2. Morador

**Sub-tipos encontrados** (em `institutional.md` + `identity.md`):
- **principal** (titular de unidade, voto em assembleia)
- **dependente** (até 2 por unidade, sem voto)
- **proprietário** (titular com fração ideal)
- **inquilino** (reside mas não é proprietário)
- **responsável financeiro** (paga taxa)
- **representante legal** (menor de idade)

**Nota**: v2 lista apenas "Base/Pagante"; canônico destaca sub-TIPOS por **relação à unidade** (não apenas por plano).

**Enum canônico**:
```go
membership_type ∈ {titular, dependente, proprietario, inquilino}  // unidade
user_role_type ∈ {titular, dependent}  // operacional
```

**Planos aplicáveis**: base (`resident_base`, gratuito) / plus (`resident_paid`, ~R$ 9,90/mês)

**Dados cadastro obrigatórios**: nome, email, CPF, telefone, data nasc (≥18), CEP, condomínio.

### 3. Empresa Condominial

**Sub-tipos reais** (em `commercial.md` + specs):
- **Prestadora condominial** — serviço ao condomínio (portaria, limpeza, segurança, etc.)
- **Administradora delegada** — gerencia condomínios em nome de síndico (backlog M5+)

**Categorias de serviço** (em schema TS): portaria, limpeza, manutenção, segurança, reforma, assessoria, paisagismo, desinsetização, etc. (controlada em config).

**Planos aplicáveis**: trial / plus (`enterprise_plus`, 8 vídeos/mês) / pro (`enterprise_pro`, 12 vídeos/mês, E→E Connect Me).

**Dados cadastro obrigatórios**: CNPJ, razão social, nome fantasia, email rep legal, telefone, endereço, categoria principal, até 5 subcategorias.

### 4. Parceiro da Vizinhança (Comércio Local)

**Sub-tipos**: MEI (CPF) ou PJ (CNPJ).

**Planos aplicáveis**: base (`local_company_standard`, trial 30d).

**Dados cadastro obrigatórios**: nome/razão social, email, CPF ou CNPJ, CEP, representante legal, financeiro.

### 5. Agência de Marketing

**Tratamento canônico**: Actor delegado, **não é tenant próprio** (MVP M1-M3). Impersona empresa cliente via token scoped.

**Sub-tipos**: implicitamente vinculada a Empresa cliente.

**Permissões delegadas**: publicar vídeos, editar metadados, ver analytics (video publishing scope).

**Trial**: compartilha trial da empresa cliente.

### 6. Admin Master Síndico

**Sub-tipos encontrados**:
- Super Admin (permissão total, ABAC bypass)
- Moderador (aprova/rejeita conteúdo)
- Suporte (acesso a dados de tenant)
- Financeiro (reconciliação Stripe, refunds)

**Permissões**: gerenciar tenants, moderar conteúdo, broadcasts, painel de métricas, audit logs, refunds.

**MFA obrigatório** em M2+; IP allowlist em M3+.

### 7. Outras personas descobertas

Nenhuma adicional além das 6 acima no specs canônico. Legado menciona "Vizinhança" como comércio local (já coberto por "Parceiro").

---

## § 4. MATRIZ PERSONA × PLANO × FEATURES (extrato exaustivo)

**Validação contra `functional-matrix.md` canônico**:

| Feature | Síndico N1 (base) | Síndico N2 (plus) | Síndico N3 (pro) | Morador Base | Morador Pago | Empresa Plus | Empresa Pro | Parceiro | Agência |
|---|---|---|---|---|---|---|---|---|---|
| Criar condomínio | ⏳ trial | ✅ | ✅ | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ |
| Criar timeline (atividade) | ⏳ | ✅ | ✅ | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ |
| Criar comunicado | ⏳ | ✅ | ✅ | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ |
| Publicar vídeo institucional | ❌ | 4/mês | 12/mês | ❌ | ❌ | 8/mês | 12/mês | ❌ | ↗ (delegado) |
| Vídeo-currículo (Morador) | ❌ | ❌ | ❌ | ❌ | ⚠️ lock 90d | ❌ | ❌ | ❌ | ❌ |
| Connect Me → Empresa | 2/ano | 4/ano | ∞ | ❌ | 2/ano | — | — | — | ↗ (delegado) |
| Connect Me ← Empresa | ❌ | ❌ | ❌ | ❌ | ❌ | ✅ | ✅ | — | — |
| Connect Me E→E | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | ⚠️ quota | — | — |
| Connect Me → Parceiro | ⚠️ | ✅ | ✅ | ❌ | ❌ | ❌ | ❌ | — | — |
| Convocar assembleia | ⏳ | ✅ | ✅ | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ |
| Votar em assembleia | — | — | — | ❌ | ✅ | — | — | — | — |
| Ver vídeos empresa (integral) | ❌ | ❌ | ✅ | ❌ | ✅ | ❌ | ❌ | ❌ | ❌ |
| Ver vídeos empresa (25% preview) | ✅ | ✅ | ✅ | ✅ | ✅ | — | — | ❌ | — |
| Banco Talentos (consultar) | ❌ | ❌ | ❌ | ❌ | ❌ | ✅ | ✅ | ✅ | ✅ |
| Academia LMS (consumir) | ⚠️ | ✅ | ✅ | ❌ | ✅ | ✅ | ✅ | ❌ | ✅ |
| Academia LMS (criar cursos) | ❌ | ❌ | ✅ | ❌ | ❌ | ❌ | ✅ | ❌ | ❌ |
| Perfil público visibilidade | Privado | ✅ | ✅ | — | — | ✅ | ✅ | ✅ | ✅ |
| Exportar relatório PDF | ⏳ | ✅ | ✅ | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ |
| Publicar cupom Vizinhança | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | ⚠️ | ✅ | ❌ |
| Métricas dashboard | ❌ | ⚠️ | ✅ | ❌ | ❌ | ✅ | ✅ | ⚠️ | ✅ (delegado) |

---

## § 5. DIVERGÊNCIAS v2 vs CANÔNICO — LISTA ACIONÁVEL

| Aspecto | v2 Diz | Canônico Diz | Ação Proposta |
|---|---|---|---|
| **Tiers do Síndico** | N1/N2/N3 são tiers próprios | N1/N2/N3 são planos mapeados a tiers trial/base/plus/pro | Renomear em v2: `syndic_n1` → `tier=base`, `syndic_n2` → `tier=plus`, `syndic_n3` → `tier=pro` (mantém slug) |
| **Estrutura enum** | `plan_tier` não existe ou tem tiers N1/N2/N3 | `plan_tier ∈ {trial, base, plus, pro}` (canônico) | Unificar v2: criar `plan_tier` enum com 4 valores (não 7) |
| **Síndico N1: publicar vídeo** | v2 lista "4/mês em N1" ou "sem publicar" conforme documento | Canônico: **N1 ❌ não publica** (ver `business-model.md` linha 47, `matrix-functional` linha 163) | Corrigir v2: N1 → ❌ vídeos; apenas N2 (4/mês) e N3 (12/mês) |
| **Morador dependente** | v2 menciona brevemente | Canônico detalha: até 2 por unidade, acesso leitura, sem voto assembleia, revogação imediata | Expandir v2: morador dependente é sub-tipo com permissões explícitas |
| **Agência Marketing** | Persona com plano próprio (backlog) | MVP: actor delegado sem subscription separada | Alinhar v2: Agência não é tenant M1-M3, impersona empresa via token scoped |
| **Admin interno** | Mencionado vagamente | 4 sub-tipos: Super Admin, Moderador, Suporte, Financeiro | Detalhar v2: admin sub-tipos com matriz ABAC explícita |
| **Connect Me morador → síndico** | v2 não menciona explicitamente | Canônico: Morador Base ❌, Morador Pagante 2/ano | Adicionar v2: Connect Me Morador→Síndico como feature (não só Síndico→Empresa) |
| **Empresa Sub-tipos** | v2 genérico "Empresa Plus/Pro" | Canônico: Prestadora condominial vs Administradora delegada (M5+) | Deixar claro v2: M1-M3 = Prestadora; Administradora é backlog |
| **Trial resumido** | v2: síndico 15d, empresa 7d, parceiro 30d, morador — | Canônico: idem + **cartão obrigatório no signup** (pré-auth, não debita), D-3/D-1/D+0 notificações | Adicionar v2: regras trial completas (notificações, pre-auth) |
| **Plan_id vs plan_tier** | v2 não diferencia | Canônico: `plan_id` PK (ex: "syndic_n1_2024"), `plan_tier` enum normalizado | Alinhar v2: structure Plan com (id, role, tier, name, ...) explícito |
| **Enums: user_role** | Mencionado em v2 | Canônico: `syndic \| resident \| enterprise \| marketing \| local_company \| admin` (6 valores) | Validar v2 tem esses 6; se tem variações, reconciliar |
| **Morador Base → Pagante** | Trigger por tentativa paywall | Canônico: idêntico; detalhes em BIL-002/003 | ✅ Alinhado |

---

## § 6. ARQUIVOS v2 QUE PRECISAM RE-DESTILAÇÃO (PRIORIZADA)

1. **`personas.md`** (CRÍTICO)
   - Renomear "N1/N2/N3" como plano-names (manter slugs); explicar tier=base/plus/pro
   - Expandir morador: dependente como sub-tipo
   - Detalhar admin: 4 sub-tipos explícitos
   - Adicionar Connect Me morador→síndico

2. **`business-model.md`** (CRÍTICO)
   - Corrigir: Síndico N1 → ❌ vídeos (não 4/mês)
   - Explicar tier unificado (trial/base/plus/pro)
   - Adicionar quotas N1 Connect Me (2/ano confirmado)

3. **`billing.md`** (CRÍTICO)
   - Adicionar BIL-017 correção: N1 ❌ publicação
   - Confirmar plan_tier enum {trial, base, plus, pro}
   - Detalhar cartão obrigatório + notificações trial
   - Quotas Connect Me Morador→Síndico (2/4/ano)

4. **`matrix-functional.md`** (MÉDIO)
   - Criar v2: gerar matriz análoga ao canônico `functional-matrix.md`
   - Validar: Síndico N1 linha "Publicar vídeo" = ❌ (não ⚠️)
   - Adicionar: Connect Me morador→síndico

5. **`institutional.md`** (MÉDIO)
   - Adicionar: morador dependente estrutura completa
   - Confirmar: roles operacionais (principal/auxiliary/assistant para síndico; titular/dependent para morador)

6. **Admin internal personas** (NOVO ARQUIVO)
   - Criar: `admin-personas.md` com 4 sub-tipos (Super/Moderador/Suporte/Financeiro)

---

## § 7. ENUMS RELEVANTES DESTILADOS (CANÔNICO)

```go
// user_role — 6 personas principais (Zitadel + local)
const (
  UserRoleSyndic        = "syndic"
  UserRoleResident      = "resident"
  UserRoleEnterprise    = "enterprise"
  UserRoleMarketing     = "marketing"
  UserRoleLocalCompany  = "local_company"
  UserRoleAdmin         = "admin"
)

// user_role_type — sub-contexto operacional (banco local, não Zitadel)
const (
  RoleTypePrincipal     = "principal"      // síndico principal ou titular unidade
  RoleTypeAuxiliary     = "auxiliary"      // síndico auxiliar
  RoleTypeAssistant     = "assistant"      // assistente (sem voto)
  RoleTypeTitular       = "titular"        // titular de unidade (morador)
  RoleTypeDependent     = "dependent"      // dependente (familiar)
  RoleTypeAdministrator = "administrator"  // admin empresa
  RoleTypeOperator      = "operator"       // operador empresa
)

// plan_tier — unificado em 4 tiers (canônico)
const (
  PlanTierTrial = "trial"
  PlanTierBase  = "base"
  PlanTierPlus  = "plus"
  PlanTierPro   = "pro"
)

// plan_role — persona do plano (separado de user_role)
const (
  PlanRoleSyndic       = "syndic"
  PlanRoleResident     = "resident"
  PlanRoleEnterprise   = "enterprise"
  PlanRoleMarketing    = "marketing"
  PlanRoleLocalCompany = "local_company"
)

// membership_type — tipo de membro em unidade (Morador)
const (
  MembershipTypeTitular      = "titular"
  MembershipTypeDependent    = "dependente"
  MembershipTypeProprietario = "proprietario"
  MembershipTypeInquilino    = "inquilino"
)

// subscription_status — estado de assinatura (Stripe + local)
const (
  SubscriptionStatusTrialing          = "trialing"
  SubscriptionStatusActive            = "active"
  SubscriptionStatusPastDue           = "past_due"
  SubscriptionStatusCanceled          = "canceled"
  SubscriptionStatusPaused            = "paused"
  SubscriptionStatusIncomplete        = "incomplete"
  SubscriptionStatusIncompleteExpired = "incomplete_expired"
)
```

---

## § 8. TL;DR

**Canônico**:
- **Tiers unificados**: trial/base/plus/pro para TODAS as personas (não N1/N2/N3 como tiers)
- **Síndico**: 3 planos mapeados a base/plus/pro; N1 **não publica vídeos**; Connect Me 2/4/∞ por tier
- **Morador**: Base gratuito (2 Connect Me/ano), Pagante ~R$ 10 (4/ano); sub-tipos (principal, dependente, proprietário, inquilino, responsável financeiro)
- **Empresa**: Plus/Pro mapeados a plus/pro; 8/12 vídeos/mês; E→E Connect Me só Pro
- **Parceiro**: único tier (base); cupons
- **Agência**: Actor delegado, não tenant próprio
- **Admin**: 4 sub-tipos internos; Super/Mod/Support/Finance

**v2 Legado erros principais**:
1. Trata N1/N2/N3 síndico como tiers distintos (não mapeados)
2. N1 listado com "4/mês vídeos" (canônico: ❌)
3. Morador dependente sub-explicado
4. Admin interno não detalhado
5. Connect Me morador→síndico não mencionado
6. Agência como persona com plano próprio (MVP: delegado)

**Top 3 correções urgentes para v2**:
1. Renomear tier mapping: N1→base, N2→plus, N3→pro (manter slugs)
2. N1 vídeos: ❌ (não 4/mês)
3. Unificar enum `plan_tier` com 4 valores (não 6-7)
