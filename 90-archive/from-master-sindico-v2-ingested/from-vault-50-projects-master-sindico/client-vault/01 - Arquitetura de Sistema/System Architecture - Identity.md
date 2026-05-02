---
title: System Architecture — Identity Domain
parent: System Architecture
domain: IDENTITY
status: Sprint 1 Complete
tags:
  - mastersindico
  - architecture
  - identity
  - ddd
---

# HIGH-DOMAIN: Identity

> **Status:** Sprint 1 Completo ✅
> **Tenant Isolation:** Nenhum (global — sem `tenant_id`)
> **Módulos:** 7 implementados
> **Source of Truth:** Requirements 1-6.1

---

## Visão Geral

O domínio Identity é a fundação de todo o ecossistema. Gerencia autenticação, autorização, perfis de todas as personas, onboarding, billing e comunicação. Todos os outros domínios dependem dele.

```
IDENTITY (Global)
├── AuthModule ✅
├── UserModule ✅
├── ProfileModule ✅
├── OnboardingModule ✅
├── BillingModule ✅
├── SearchModule ✅ (migrar para Meilisearch)
└── CommunicationModule ✅
```

---

## Módulos Detalhados

### AuthModule
**Path:** `modules/auth/`
**Req:** 1, 2

| Aspecto | Detalhe |
|---------|---------|
| **Entities** | User, Session, Account, Verification |
| **Use Cases** | Login, Register, VerifyEmail, ResetPassword, RefreshToken |
| **Auth** | JWT (jose) + Argon2 + Arctic OAuth + Cookie/Session |
| **Sessions** | DB + Redis cache, 1 device por conta, fingerprint + IP tracking |
| **CASL ABAC** | Roles: sindico, morador_titular, morador_dependente, empresa_administrador, empresa_operador, agencia |

**Events Published:**
- `identity.auth.user-registered`
- `identity.auth.user-verified`
- `identity.auth.session-created`
- `identity.auth.session-invalidated`

**Events Consumed:** Nenhum (root dependency)

---

### UserModule
**Path:** `modules/user/`
**Req:** 3

| Aspecto | Detalhe |
|---------|---------|
| **Entities** | User, Vinculo |
| **Use Cases** | GetUser, UpdateUser, ManageVinculos |
| **Status Flow** | pending_verification → active → suspended → deleted |

**Events Published:**
- `identity.user.updated`
- `identity.user.vinculo-created`
- `identity.user.vinculo-ended`

**Events Consumed:**
- `identity.auth.user-registered`

---

### ProfileModule
**Path:** `modules/profile/`
**Req:** Perfis por persona

| Aspecto | Detalhe |
|---------|---------|
| **Entities** | SyndicProfile, EnterpriseProfile, ResidentProfile, MarketingProfile, LocalCompanyProfile |
| **Use Cases** | CreateProfile, UpdateProfile, GetProfile, ManageComplianceMarkers |
| **Compliance Markers** | Governance markers no perfil do síndico (impacta busca/reputação) |

**Events Published:**
- `identity.profile.sindico-created`
- `identity.profile.company-created`
- `identity.profile.partner-created`
- `identity.profile.updated`

**Events Consumed:**
- `identity.auth.user-registered` → trigger criação de perfil

---

### OnboardingModule
**Path:** `modules/onboarding/`
**Req:** 5

| Aspecto | Detalhe |
|---------|---------|
| **Entities** | OnboardingProgress |
| **Fluxos** | Síndico (condomínio + mandato), Morador (busca condo + unidade), Empresa (CNPJ + categorias), Agência (portfolio) |
| **Persistência** | Save progress para retomar onboarding |

**Events Published:**
- `identity.user.onboarding-completed`

---

### BillingModule
**Path:** `modules/billing/`
**Req:** 4, 4.1, 6, 6.1

| Aspecto | Detalhe |
|---------|---------|
| **Entities** | Plan, Subscription, Transaction, Invoice, PaymentMethod, FeatureUsage, PlanQuota, PlanPermission, Permission |
| **Provider** | **Stripe** (Adapter Layer para futura troca) |
| **Planos** | trial → base → plus → pro |
| **Trials** | Síndico: 15 dias, Empresa: 7 dias, Parceiro: 30 dias |
| **Modelo** | "Veja tudo, bloqueie ações estratégicas" durante trial |
| **Quotas** | Connect Me requests, video uploads, storage limits |

**Events Published:**
- `identity.billing.subscription-created`
- `identity.billing.subscription-expired`
- `identity.billing.payment-succeeded`
- `identity.billing.payment-failed`
- `identity.billing.trial-expired`
- `identity.billing.plan-changed`

**Events Consumed:**
- `identity.auth.user-registered` → criar trial

---

### SearchModule
**Path:** `modules/search-engine/`

| Aspecto | Detalhe |
|---------|---------|
| **Status Atual** | PostgreSQL tsvector via provider interface |
| **Migração Planejada** | **Meilisearch** — implementar `MeilisearchSearchProvider` usando interface existente |
| **Entities** | SearchIndex, SearchQuery |
| **Visibilidade** | Plan-based (Base profiles visible to Gestor/Plus/Pro) |

**Events Published:**
- `identity.search.performed`

**Events Consumed:**
- `identity.profile.*` → re-index
- `content.video.*` → re-index

---

### CommunicationModule
**Path:** `modules/communication/`

| Aspecto | Detalhe |
|---------|---------|
| **Canais** | Email (Resend + Gmail fallback), Push (FCM), SMS (Twilio/Zenvia — pendente) |
| **Evolução** | Será promovido a **NotificationModule** cross-domain |
| **Entities** | EmailTemplate, Notification |

**Events Published:**
- `identity.communication.email-sent`
- `identity.communication.push-sent`

**Events Consumed:**
- Todos os domain events que disparam notificações

---

## Dependências

```
AuthModule ← (root, sem dependências)
    ↓
UserModule ← AuthModule
    ↓
ProfileModule ← AuthModule, UserModule
    ↓
OnboardingModule ← AuthModule, ProfileModule
    
BillingModule ← AuthModule
SearchModule ← AuthModule, BillingModule
CommunicationModule ← AuthModule
```

---

## Navegação

← [[System Architecture]] | [[System Architecture - Institutional]] →
