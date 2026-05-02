---
title: 'Admin MS — App Dedicada apps/admin (D-134) [superseded transversal D-072]'
type: spec
tags:
  - admin
  - abac
  - privileges
  - master-sindico
  - v2
  - d-134
  - superseded-d-072
source: >-
  backend/.kiro/specs (legado destilado em 04-requirements) + Obsidian (System
  Architecture - Identity) + Content/03-Modulos/Compliance-Detalhado (C5
  Auditoria) + STATE.md D-054 (4-eyes), D-056 (agnosticismo), D-058 (role
  transversal) + ADR-0026 (admin MFA M1) + ADR-0024 (moderation cascade) +
  00-product/personas §6 (Admin Master Síndico) + 03-subprojects/web/ui-catalog
  §7 (AD1-AD8) + 04-requirements/matrix-functional §"Módulo Transversal — Admin
  Master Síndico" + 01-domain/invariants (INV-086..088, INV-090, INV-091,
  INV-110) + 01-domain/aggregates/AuditEntry
created: 2026-04-23T00:00:00.000Z
updated: '2026-04-25'
status: superseded-in-part-by-D-134
aliases:
  - admin-role-transversal
  - admin-master-sindico
  - admin-privileges
---

# Admin MS — App Dedicada `apps/admin` (D-134 2026-04-25)

> 🔴 **DOCUMENTO SUPERSEDED EM PARTE — 2026-04-25**
>
> Este documento foi escrito sob a vigência de **D-072** (anteriormente referenciada erroneamente como D-058 — D-058 é Connect Me Morador, já revogada por D-094). D-072 estabelecia admin como **role privilegiada transversal embutida** nas apps existentes.
>
> **D-134 (2026-04-25) REVOGA D-072 explicitamente** — admin owner Master Síndico agora é **app dedicada `apps/admin`** no monorepo web, atrás de Cloudflare Access SSO+MFA (D-117). Trigger da revogação: viabilização Cloudflare Access (não existia em D-072) + benchmark big tech (Stripe Dashboard, AWS Console, Vercel admin separados — D-113).
>
> **O que permanece válido neste documento**:
> - Sub-papéis operacionais (super-admin, moderador, suporte, financeiro) — ainda aplicáveis dentro de `apps/admin`
> - Permissões ABAC granulares (`admin_bypass`, scoped permissions) — ainda canônicos via D-116 IAM Cloudflare-style
> - Telas AD1-AD8 — agora vivem em `apps/admin` (não em `cms`)
> - Controles LGPD + audit trail + step-up MFA + 4-eyes — ainda canônicos
> - Persona "Admin Master Síndico" interna como 6ª persona global — ainda canônica (sem tenant_id)
>
> **O que foi revogado por D-134**:
> - "App dedicada rejeitado" (§1.2) — agora é a decisão atual
> - "Mesmo bundle SolidJS feature-slice" (§1.3) — agora `apps/admin` é bundle separado
> - "Não há `admin-web/` separado" (§1.2) — agora há `apps/admin` separado
>
> **Outros sub-roles** (administradora condominial D-114, agência marketing D-102, empresa-síndica D-073, auditor assembly D-132) **continuam em rotas internas dos produtos** — não criam apps separadas. **APENAS o admin owner** Master Síndico tem app dedicada.
>
> **Reescrita completa deste doc**: pendente Fase H da consolidação 2026-04-25. Por ora, ler com este banner como filtro: tudo que cita "embutido", "rejeitado app dedicada", "mesmo bundle" → REVOGADO. Tudo sobre sub-papéis, ABAC, LGPD, audit, step-up, 4-eyes → ainda canônico.
>
> Ver: [[../STATE#D-134]] (decisão atual); [[../STATE#D-072]] (revogada); [[../STATE#D-117]] (Cloudflare Access viabiliza); [[../STATE#D-113]] (benchmark big tech).

---

# Admin MS — Role Privilegiada Transversal (TEXTO ORIGINAL — ler com banner acima)

> Canônico da Fase 8. ~~Fecha [[../STATE#D-058]]~~ → **REVOGADO**: na verdade fechava D-072 (admin role transversal); D-058 era Connect Me Morador. Herda [[../02-architecture/adr/0026-admin-mfa-m1]], [[../STATE#D-054 — Lock 90d bypass admin via 4-eyes policy]] e [[../STATE#D-056 — Agnosticismo de provedor como invariante arquitetural (crítico)]].

Admin Master Síndico **não é aplicação separada**. É **role privilegiada (`admin`) transversal**, implementada como **1 privilégio ABAC a mais** (`admin_bypass` — [[../01-domain/invariants#INV-088]]) sobre o mesmo backend + mesmas apps web/mobile que as demais personas. As telas privilegiadas vivem **dentro** dos portais existentes (`sindico-web`, `empresa-web`, `marketing-web`, `morador-web` — hoje fisicamente consolidados em `web/`), **gated por permissão**. Sub-papéis operacionais (super-admin, moderador, suporte, financeiro) são **scoped permissions** ABAC, não tenants nem apps distintas.

Este documento é o contrato transversal que reconcilia: a persona interna ([[../00-product/personas#6-admin-master-síndico-interno-da-plataforma]]), o conjunto de telas (AD1–AD8 em [[web/ui-catalog#7-admin-master-síndico]]), os requisitos ABAC/auth/audit ([[../04-requirements/functional/identity#IDN-007]], [[../04-requirements/functional/identity#IDN-036]], [[../04-requirements/functional/cross-domain#XD-008]]/[[../04-requirements/functional/cross-domain#XD-009]]/[[../04-requirements/functional/cross-domain#XD-010]]), os controles LGPD ([[../04-requirements/compliance-lgpd]]) e as decisões de governança (D-054, D-056, D-058 em [[../STATE]]).

---

## § 1. Filosofia (D-058)

### 1.1 O que **é**

Admin MS é:

1. **Role** primária Zitadel `admin` — 1:1 com [[../04-requirements/functional/identity#IDN-015]] roles primárias `syndic | resident | enterprise | marketing | local_company | admin`. Zitadel é o IdP ([[../02-architecture/adr/0003-zitadel-oidc-provider]]), porém **agnosticismo de provedor** (D-056) exige que a role venha via claim OIDC padrão, não via API proprietária.
2. **1 privilégio ABAC a mais** sobre o motor `RequirePermission(action, resource)` ([[../04-requirements/functional/cross-domain#XD-004]]): `admin_bypass` é o primeiro check na cadeia ordenada `admin_bypass → role_check → action_check → tenant_check → scope_check` ([[../STEERING#16]], [[../01-domain/invariants#INV-086]]).
3. **Telas privilegiadas gated por permissão** dentro das apps existentes — rota `/admin/*` ([[web/ui-catalog#7-admin-master-síndico]] AD1–AD8) convive no mesmo bundle web que `/app/*`, renderizada condicionalmente a partir do resultado ABAC.
4. **Sub-papéis operacionais** escopados por ABAC (super-admin, moderador, suporte, financeiro) — conceituais, não são personas próprias.

### 1.2 O que **NÃO é**

- ❌ **App dedicada** `admin-web/` separada — **rejeitado**: duplicaria features (dashboards, player de vídeo, billing UI), manteria duas bases de código, violaria reuso das mesmas features do backend.
- ❌ **Microserviço** `admin-api` separado — o mesmo backend Go + mesmas rotas `/api/v1/*` + 1 bucket `/admin/v1/*` para endpoints exclusivos (impersonate, ABAC policy editor, bypass 4-eyes). **Nenhum** código admin fora do monolito modular.
- ❌ **Tenant** próprio — admin é **global** ([[../00-product/personas#6-admin-master-síndico-interno-da-plataforma]]: "Tenant Isolation: Nenhum — sem `tenant_id`"); opera cross-tenant por design, não é um tenant.
- ❌ **Persona auto-registrável** — [[../04-requirements/functional/identity#IDN-001]]: "`admin` NÃO auto-registra (provisionado manualmente no Zitadel)"; [[../04-requirements/functional/identity#IDN-003]] Req 3 legado: "`admin` — provisionada manualmente no Zitadel (sem fluxo de auto-registro)".

### 1.3 Consequência arquitetural

| Dimensão | Efeito D-058 |
|---|---|
| **Código backend** | Zero módulo `admin/` separado. Handlers admin vivem junto do BC afeto (ex: moderação de vídeo em `content/`, refund em `billing/`, ban de user em `identity/`); o gate é ABAC. Única fronteira distinta: prefixo `/admin/v1/*` em router para compor step-up MFA obrigatório (ADR-0026). |
| **Código web** | Mesmo bundle SolidJS. `admin-panel` é **uma feature-slice transversal** (`03-subprojects/web/README` §módulos) que consome as mesmas APIs + algumas rotas exclusivas, com `gate="admin.*"` renderizando condicionalmente. |
| **Código mobile** | **Admin não opera via mobile em M1** — telas admin requerem teclado físico + MFA TOTP + IP allowlist. Mobile admin fica em backlog M3+ com escopo mínimo (dashboards read-only). |
| **Deploy** | Opção A (default M1): `/admin` como path, cookie compartilhado `.mastersindico.com.br`, CSP strict. Opção B (M2+, pendência `P-UI-003` em [[web/ui-catalog#pendências]]): subdomínio `admin.mastersindico.com.br` para IP allowlist separada. Decisão diferida para M2 com dual-check + ADR específica. |
| **DevOps** | Uma infra, um pipeline. Role `admin` não move features críticas para fora do escopo auditado do backend principal. |

### 1.4 Contraste com o legado

O Obsidian Client Vault ([[System Architecture - Identity]]) descrevia Admin apenas como um consumidor do `AuthModule` com roles `sindico, morador_titular, morador_dependente, empresa_administrador, empresa_operador, agencia` — sem admin explícito. A v2 destilada ([[../00-product/personas#6-admin-master-síndico-interno-da-plataforma]]) adiciona Admin como 6ª persona global + 4 sub-tipos operacionais. D-058 fecha a ambiguidade: admin é **role**, não app.

---

## § 2. Sub-papéis operacionais (4)

> Fonte canônica: [[../00-product/personas#6-admin-master-síndico-interno-da-plataforma]] §"Sub-tipos".

Todos compartilham `role = admin` no Zitadel. A **diferença** é `admin_sub_role` (campo local, espelho de um claim Zitadel `admin:sub_role` ou metadados) + **scope de permissões ABAC**. Implementação: tabela `admin_profiles {user_id, sub_role, ip_allowlist[], allowed_actions[], mfa_backup_codes_set, ip_allowlist_effective_at}` + enforcement em middleware.

### 2.1 Super Admin

- **Hierarquia**: 1 (topo). Apenas **2–3 pessoas** do time Master Síndico em M1 (CEO, CTO, DPO).
- **Permissões**: `admin_bypass = TRUE` em 100% das ações (todos os resource_types, todos os tenants). Pode:
  - Editar ABAC policies (AD4) — inclui criar novas actions/resources ([[web/ui-catalog#ad4-abac-matrix-override]]).
  - Executar bypass 4-eyes como Admin B (aprovador) — cumpre segregação D-054.
  - Acessar console LGPD com poder de hard-delete fora de janela (AD7).
  - Gerenciar `admin_profiles` — promover/rebaixar outros admins.
- **MFA**: **obrigatório desde M1** ([[../02-architecture/adr/0026-admin-mfa-m1]] — reclassificação de IDN-007 de M2→M1 pós-[[../11-audit/audit-log/2026-04-23-pentest-specs#A-DC-PEN-007]]).
- **IP allowlist**: **obrigatória M3+** ([[../00-product/personas#6]]: "IP allowlist pra Super Admin M3+"). Até M3: hardening via rate-limit agressivo em `/admin/v1/*` (5 rpm por user_id — IDN-036) + alertas Sentry em login de IP novo ([[../04-requirements/functional/identity#IDN-028]]).
- **Step-up MFA**: **obrigatório em toda rota `POST /admin/v1/*`** — [[../02-architecture/adr/0026-admin-mfa-m1#decisão]] lista "POST /admin/v1/* (qualquer rota admin)" como endpoint auth-crítico.

### 2.2 Moderador

- **Hierarquia**: 2. Time de trust & safety (0–5 pessoas em M1; cresce com volume — gatilhos em [[../02-architecture/adr/0024-moderation-cascade]] §"Gatilhos para avançar fase").
- **Permissões** (scoped, **não** bypass total):
  - `content.video.approve`, `content.video.reject_with_reason` — fila de moderação ([[web/ui-catalog#ad2-moderação-de-vídeos]] + [[../04-requirements/functional/content]] moderação pré-publish, SLA **48h manual M1**).
  - `content.video.remove_pre90d` (com reason obrigatório — INV-AuditEntry `RequiresReason`) — bypass do Lock 90d, sujeito a 4-eyes D-054.
  - `identity.user.ban` (72h suspensão) e `identity.user.unban` — resposta a harassment reports ([[web/ui-catalog#ad3-harassment-reports]]).
  - `commercial.harassment.archive`, `commercial.harassment.escalate` — workflow de apelação.
  - `content.comment.remove`, `commercial.review.remove` — moderação secundária de artefatos derivados.
- **Ações auditadas (`action` em `audit_trail`)**: `moderation.video.approved`, `moderation.video.rejected`, `moderation.harassment.resolved`, `moderation.user.suspended`, `moderation.appeal.decided`. Toda ação carrega `reason` obrigatório (≥ 20 chars) quando remove artefato ou suspende conta.
- **Limite**: **NÃO** pode editar ABAC policies; **NÃO** pode hard-delete; **NÃO** pode refund; **NÃO** pode impersonate cross-tenant para fluxos financeiros.
- **MFA**: obrigatório M1 (IDN-007 reclassificado — todos os `role = admin` cobertos).

### 2.3 Suporte

- **Hierarquia**: 3. Time de CS (0–10 pessoas em M1; terceirizável M3+).
- **Permissões**:
  - `identity.user.read_cross_tenant` — ver dados de qualquer tenant para resolver ticket.
  - `identity.session.impersonate` — impersonation explícita com audit ([[web/ui-catalog#ad5-usuários-internos]] ação "Impersonate (com audit)"). **Requer** step-up MFA + `reason` obrigatório + TTL máximo **30 min** por sessão de impersonation.
  - `institutional.*.read`, `commercial.*.read`, `content.*.read`, `billing.subscription.read` — telemetria cross-tenant read-only.
  - `lgpd.user.export` — acionar portabilidade de dados em nome do titular (AD7 [[web/ui-catalog#ad7-lgpd-console]]).
  - `identity.session.revoke_other` — forçar logout cross-device do user em crise (ex: celular roubado).
- **Limite**: **NÃO** pode editar dados, aprovar refunds, remover vídeos, editar ABAC policies. Operação é essencialmente read + 2 writes bem escopados (impersonate, revoke session).
- **Audit estendido**: **toda** ação de suporte entra em **duas** tabelas: `audit_trail` (operacional) + `lgpd_logs` (quando acessa PII — [[../04-requirements/functional/cross-domain#XD-010]]). PII vista durante impersonation **não** é logada em raw: `actor_id`, `target_user_id`, `reason`, `session_duration`, `resources_viewed_count` — sem payloadAfter/Before.

### 2.4 Financeiro

- **Hierarquia**: 3. Time financeiro (0–3 pessoas em M1).
- **Permissões**:
  - `billing.subscription.read_cross_tenant` — busca subscription por user/tenant (AD8 [[web/ui-catalog#ad8-billing-admin]]).
  - `billing.refund.create` — refund manual via Stripe com audit + reason (inclui categoria: "erro operacional" / "reembolso contratual" / "fraude"). SLA 72h dunning.
  - `billing.plan.change_manual` — alterar plano com audit (quando cliente paga fora do fluxo Stripe ou corrige erro admin).
  - `billing.webhook.replay` — reprocessar webhook Stripe da DLQ (idempotent — [[../02-architecture/adr/0027-webhook-idempotency-db]]).
  - `billing.invoice.mark_as_paid` — casos manuais (boleto fora Stripe, ordem judicial).
- **Limite**: sem poder de ban, impersonation, ABAC edit. Read em content/institutional apenas para contexto do ticket financeiro (read-only).
- **Moneyflow safeguards**: refund acima de **R$ 10.000** exige 4-eyes (alinhamento com [[../02-architecture/adr/0026-admin-mfa-m1#decisão]] step-up para contracts.approve ≥ R$ 10.000).

### 2.5 Quadro-resumo dos sub-papéis

| Sub-papel | admin_bypass | Escopo ABAC | MFA | IP allowlist | 4-eyes required | Audit duplo (trail + lgpd) |
|---|---|---|---|---|---|---|
| Super Admin | 100% | sem restrição | M1 | **M3+** obrig | Sim, como aprovador | Sim |
| Moderador | parcial | `content.*`, `identity.user.ban`, `commercial.harassment.*` | M1 | não | Para `remove_pre90d` | Sim (se toca PII) |
| Suporte | leitura ampla | read cross-tenant + impersonate + revoke session | M1 | não | Para `impersonate` ≥ 30 min | **Sempre** |
| Financeiro | parcial | `billing.*` cross-tenant | M1 | não | Para refund ≥ R$ 10k | Sim (se toca dados fiscais) |

---

## § 3. Matriz ABAC — Admin × Action × Resource

> Ordenação canônica da engine: `admin_bypass → action_configured → role_allowed → tenant_matches → plan_tier_sufficient` ([[../01-domain/invariants#INV-086]]). Cache Redis 5 min invalidado via webhook ([[../04-requirements/functional/cross-domain#XD-006]]).

### 3.1 Como `admin_bypass` é avaliado

Referência: [[backend/security#1-abac-engine]] pseudo-código.

```go
func (e *Engine) Evaluate(ctx context.Context, sub Subject, res Resource, act Action) (Decision, Reason) {
    // 1. admin_bypass
    if sub.IsAdmin() && e.adminBypassEnabled {
        return Permit, ReasonAdminBypass
    }
    // ... cadeia role → action → tenant → scope
}
```

Contrato (reforçado em D-058 + [[../01-domain/invariants#INV-088]]):

1. `sub.IsAdmin()` **deriva da role Zitadel = `admin`**, não de flag booleana local. Sub-papel vive em `admin_profiles.sub_role`.
2. `adminBypassEnabled = true` **por BC**, mas **scope do sub-papel restringe**: middleware consulta `admin_profiles.allowed_actions` antes de devolver `Permit`. Se `action ∉ allowed_actions`, cai para avaliação normal (role-based) e geralmente nega.
3. **Toda** decisão `admin_bypass` **sempre** gera `AuditEntry` com `reason` obrigatório ([[../01-domain/aggregates/AuditEntry#invariantes]] INV-088) — zero exceção. Falha em logar bloqueia a operação (fail-closed).
4. **Em rotas `/admin/v1/*`**, step-up MFA é gate anterior ao ABAC — sem `X-Step-Up-Token` válido, a engine nem roda ([[../02-architecture/adr/0026-admin-mfa-m1]]).

### 3.2 Actions canônicas por sub-papel (matriz exaustiva M1–M3)

Legenda: ✅ permitido direto · 🧑‍🤝‍🧑 exige 4-eyes · 🔒 exige step-up MFA `≤ 5min` (toda rota `/admin/v1/*` tem step-up por default; ✅ abaixo **inclui** step-up) · ❌ bloqueado.

| Action | Resource | SuperAdmin | Moderador | Suporte | Financeiro | Tenant scope | Reason obrig |
|---|---|---|---|---|---|---|---|
| `identity.user.read` | `user` | ✅ | ✅ | ✅ | ✅ | cross | não |
| `identity.user.ban` (72h) | `user` | ✅ | ✅ | ❌ | ❌ | cross | **sim** |
| `identity.user.unban` | `user` | ✅ | ✅ | ❌ | ❌ | cross | **sim** |
| `identity.user.hard_delete_lgpd` | `user` | 🧑‍🤝‍🧑 | ❌ | ❌ | ❌ | cross | **sim (categoria LGPD)** |
| `identity.session.impersonate` | `session` | ✅ | ❌ | ✅ (TTL 30min) | ❌ | cross | **sim** |
| `identity.session.revoke_other` | `session` | ✅ | ✅ | ✅ | ❌ | cross | **sim** |
| `identity.admin_profile.manage` | `admin_profile` | ✅ | ❌ | ❌ | ❌ | global | **sim** |
| `content.video.approve` | `video` | ✅ | ✅ | ❌ | ❌ | cross | não |
| `content.video.reject` | `video` | ✅ | ✅ | ❌ | ❌ | cross | **sim** |
| `content.video.remove_pre90d` | `video` | 🧑‍🤝‍🧑 | 🧑‍🤝‍🧑 | ❌ | ❌ | cross | **sim (categoria)** |
| `content.comment.remove` | `comment` | ✅ | ✅ | ❌ | ❌ | cross | **sim** |
| `commercial.harassment.resolve` | `harassment_report` | ✅ | ✅ | ❌ | ❌ | cross | **sim** |
| `commercial.contract.lock_bypass_pre90d` | `contract` | 🧑‍🤝‍🧑 | ❌ | ❌ | 🧑‍🤝‍🧑 (≥ R$ 10k) | cross | **sim (categoria)** |
| `commercial.harassment.archive` | `harassment_report` | ✅ | ✅ | ❌ | ❌ | cross | sim |
| `commercial.harassment.escalate` | `harassment_report` | ✅ | ✅ | ❌ | ❌ | cross | sim |
| `billing.subscription.read` | `subscription` | ✅ | ❌ | ✅ | ✅ | cross | não |
| `billing.refund.create` | `invoice` | ✅ | ❌ | ❌ | ✅ (< R$ 10k) / 🧑‍🤝‍🧑 (≥) | cross | **sim (categoria)** |
| `billing.plan.change_manual` | `subscription` | ✅ | ❌ | ❌ | ✅ | cross | **sim** |
| `billing.webhook.replay` | `webhook_event` | ✅ | ❌ | ❌ | ✅ | cross | não |
| `billing.invoice.mark_paid` | `invoice` | 🧑‍🤝‍🧑 | ❌ | ❌ | 🧑‍🤝‍🧑 | cross | **sim** |
| `institutional.tenant.suspend` | `condominium | company` | ✅ | ❌ | ❌ | ❌ | cross | **sim** |
| `institutional.tenant.reactivate` | `condominium | company` | ✅ | ❌ | ❌ | ❌ | cross | **sim** |
| `institutional.cnpj.approve_manual` | `company` | ✅ | ❌ | ❌ | ✅ | cross | **sim** |
| `abac.policy.create/update/delete` | `abac_policy` | ✅ | ❌ | ❌ | ❌ | global | **sim** |
| `lgpd.user.export_on_behalf` | `user_export` | ✅ | ❌ | ✅ | ❌ | cross | **sim** |
| `lgpd.deletion.process` | `deletion_request` | ✅ | ❌ | ✅ | ❌ | cross | **sim** |
| `platform.broadcast.send_email_global` | `broadcast` | ✅ | ❌ | ❌ | ❌ | global | **sim (categoria)** |
| `platform.broadcast.send_push_global` | `broadcast` | ✅ | ❌ | ❌ | ❌ | global | **sim (categoria)** |
| `marketing.token.approve_or_revoke` | `marketing_agency_token` | ✅ | ✅ | ❌ | ❌ | cross | **sim** |
| `platform.live.rtmp_start` | `live_room` | ✅ | ❌ | ❌ | ❌ | global | não |
| `platform.feature_flag.toggle` | `feature_flag` | ✅ | ❌ | ❌ | ❌ | global | **sim** |

### 3.3 Invariantes ABAC do admin

- **INV-086 ordem estrita** da cadeia — não se troca a ordem em nenhum hot-path. PBT com 400 casos ([[../01-domain/invariants#INV-086]]).
- **INV-087 default-deny** — ação não configurada nas `abac_policies` devolve `Deny(action_not_configured)`, **mesmo para admin** (se `admin_profile.allowed_actions` não a inclui). Evita escalada silenciosa ao criar rota nova ([[../01-domain/invariants#INV-087]]).
- **INV-088 admin bypass auditado** — `audit_trail.action = 'admin_bypass'` **sempre** que `reason = ReasonAdminBypass` é retornado ([[../01-domain/invariants#INV-088]]).
- **INV-089 tenant isolation** — admin em `admin_bypass` bypassa tenant check por design, mas **toda** query cross-tenant loga `tenant_id` visitado em metadata ([[../01-domain/invariants#INV-089]]).
- **INV-110 step-up admin** — toda rota `/admin/v1/*` exige `X-Step-Up-Token` fresh ≤ 5 min ([[../01-domain/invariants]] + [[../02-architecture/adr/0026-admin-mfa-m1]]).

---

## § 4. Telas admin por app/produto

> Catálogo canônico: [[web/ui-catalog#7-admin-master-síndico]] AD1–AD8. Complementado abaixo pelo contexto transversal (onde admin convive com telas de persona).

### 4.1 Em `web/` — portal único, feature-slice `admin-panel`

Arquitetura web é **CSR feature-sliced** ([[web/architecture#routing--split]]). `admin-panel` convive com `morador`, `sindico`, `empresa`, `marketing`, `vizinhanca` no mesmo bundle. Rotas `/admin/*` estão em chunk lazy-loaded ([[web/architecture]]: "Cada rota pesada (…, admin/*) vem em chunk separado"). Decisão de path-vs-subdomínio fica em `P-UI-003` ([[web/ui-catalog#pendências]]).

| Tela | Rota | Gated por | Sub-papel principal | Endpoint API principal | Dependências |
|---|---|---|---|---|---|
| **AD1 — Painel admin (home)** | `/admin` | `admin.*` | todos | `GET /admin/v1/kpis` | [[web/ui-catalog#ad1-painel-admin-home]] |
| **AD2 — Moderação de vídeos** | `/admin/moderacao/videos` | `content.video.approve` | moderador | `GET /admin/v1/moderation/videos`, `POST /admin/v1/moderation/videos/:id/{approve,reject}` | [[../02-architecture/adr/0024-moderation-cascade]], [[../04-requirements/functional/content]], SLA 48h M1 |
| **AD3 — Harassment reports** | `/admin/harassment-reports` | `commercial.harassment.resolve` | moderador | `GET /admin/v1/harassment`, `POST /admin/v1/harassment/:id/{suspend,archive,escalate}` | [[../04-requirements/functional/commercial]], [[web/ui-catalog#ad3-harassment-reports]] |
| **AD4 — ABAC matrix (override)** | `/admin/abac` | `abac.policy.*` | super-admin | `GET/POST/PATCH /admin/v1/abac/policies` | [[../04-requirements/functional/cross-domain#XD-007]], INV-087 — edição recarrega cache Redis |
| **AD5 — Usuários internos** | `/admin/users` | `identity.user.*` | suporte + super-admin | `GET /admin/v1/users`, `POST /admin/v1/users/:id/impersonate`, `POST /admin/v1/users/:id/ban`, `GET /admin/v1/users/:id/sessions` | [[../04-requirements/functional/identity#IDN-030]] (ban local), IDN-036 (step-up), IDN-022 (deletion) |
| **AD6 — Dashboards técnicos** | `/admin/dashboards` | `platform.telemetry.read` | super-admin | embed Grafana iframe + OTel links ([[../04-requirements/functional/cross-domain#XD-032]]) | [[../02-architecture/adr/0020-opentelemetry-grafana-sentry]] |
| **AD7 — LGPD console** | `/admin/lgpd` | `lgpd.*` | suporte + super-admin | `POST /admin/v1/lgpd/export/:user_id`, `POST /admin/v1/lgpd/deletion/:user_id/process`, `GET /admin/v1/lgpd/breaches` | [[../04-requirements/compliance-lgpd]], [[../04-requirements/functional/identity#IDN-022]] (SLA 15d), [[../02-architecture/adr/0028-lgpd-keyed-hmac]] |
| **AD8 — Billing admin** | `/admin/billing` | `billing.*` cross-tenant | financeiro + super-admin | `GET /admin/v1/billing/subscriptions`, `POST /admin/v1/billing/refund`, `POST /admin/v1/billing/plan-change`, `GET /admin/v1/billing/webhooks-dlq` | [[../02-architecture/adr/0004-stripe-payment-gateway]], [[../02-architecture/adr/0027-webhook-idempotency-db]] |

### 4.2 Em `sindico-web` (conceitual — hoje em `web/`): não há telas admin exclusivas

Admin que precisa operar dentro do contexto de um condomínio (ex: ver timeline imutável, inspecionar governance score [[../04-requirements/functional/cross-domain#XD-011]]) **usa as mesmas telas do síndico** com o gate ABAC permitindo cross-tenant read. Não há rota `/app/governanca/.../admin/*`; o que há é `admin_bypass = true` liberando acesso mesmo sem membership.

**Exceções** — telas síndico com **affordance adicional** visível só para admin (dentro do mesmo componente Solid):

- `S52 Trilha de auditoria visível` ([[web/ui-catalog#s52-trilha-de-auditoria-visível]]): admin vê **mais colunas** (actor_role + IP + trace_id) e **toda** entrada (não só do próprio tenant).
- `S53 Imutabilidade + Adendo Formal`: admin pode anexar "adendo administrativo" com categoria "correção operacional" — vai para `audit_trail` com dupla marcação.
- Timeline: admin vê entries `visible_to_admins_only = true` (reservado para flags antifraude).

### 4.3 Em `empresa-web` (conceitual — hoje em `web/`)

- **AD8 (billing admin)** acessa subscriptions de empresas cross-tenant na mesma UI de billing self-service da empresa (`/empresa/conta/plano`), com banner "Visualizando como admin — {user.name}" + toda ação loga impersonate-ish.
- `institutional.cnpj.approve_manual` (AD1 CTA + tela dedicada): aprovação manual de CNPJ falhado no fluxo automático ([[../04-requirements/functional/cross-domain#XD-020]]) — UI compartilhada com onboarding da empresa em estado "pending_manual_review", admin vê fila.

### 4.4 Em `marketing-web` (conceitual — hoje em `web/`)

- `marketing.token.approve_or_revoke` (AD1 CTA): admin vê todos os tokens de agência emitidos por empresas ([[../04-requirements/functional/identity#IDN-025]]). Pode revogar globalmente (fraude) sem precisar que a empresa dona aja.
- Auditoria de uploads: admin vê cross-tenant "vídeos uploadados por agência X nos últimos 7d" para investigação.

### 4.5 Em `morador-web` / mobile: **não há telas admin**

- **Justificativa**: moderação, ban, refund, ABAC edit — **nenhuma** dessas operações é apropriada para thumb-friendly UX em smartphone; todas requerem audit rico (reason text ≥ 20 chars, categoria, upload de evidência). Mobile tem **tela pública** de "reportar usuário / conteúdo" que alimenta AD3 no web, mas o processamento é no web.
- **Backlog M3+**: dashboards read-only mobile para super-admin em "modo observação" (ver MRR, KPIs) — não executa nenhuma mutação.

### 4.6 Resumo da matriz telas × app

| App/Portal | Telas admin dedicadas | Admin consome telas de persona | Step-up req |
|---|---|---|---|
| `web/ (admin-panel slice)` | **AD1–AD8** | N/A (é a dedicada) | toda rota `/admin/v1/*` |
| `web/ (sindico slice)` | ❌ | S52, S53, timeline com colunas+ | só em mutações admin específicas |
| `web/ (empresa slice)` | ❌ | Billing cross-tenant, CNPJ approve | sim em todas mutações admin |
| `web/ (marketing slice)` | ❌ | Token agency revoke cross-tenant | sim |
| `web/ (morador slice + vizinhança)` | ❌ | — (apenas reportar, consumido por AD3) | N/A |
| `mobile/ (Flutter)` | ❌ (M1–M2) / read-only dashboards (M3+) | — | N/A em M1 |

---

## § 5. 4-eyes policy (ADR-0026 + D-054)

> Referência canônica: [[../STATE#D-054 — Lock 90d bypass admin via 4-eyes policy]]. Princípio: SOX 4-eyes + NIST SP 800-53 AC-5 (segregação de função).

### 5.1 Quando é obrigatório

4-eyes é **obrigatório** em operações de alto valor jurídico/financeiro/reputacional:

1. **Lock 90d bypass de vídeo** (`content.video.remove_pre90d`) — [[../STATE#D-054]] + GR-029 trava trimestral + [[../01-domain/aggregates/Video]].
2. **Lock 90d bypass de contrato** (`commercial.contract.lock_bypass_pre90d`) — [[../04-requirements/functional/commercial]] locks de contrato.
3. **Hard-delete LGPD fora da janela 15d** (`identity.user.hard_delete_lgpd` pré-SLA ou pós-SLA com exceção) — [[../04-requirements/functional/identity#IDN-022]].
4. **Refund / mark_as_paid ≥ R$ 10.000** — alinha com [[../02-architecture/adr/0026-admin-mfa-m1#decisão]] step-up para `contracts.approve ≥ R$ 10k`.
5. **Suspensão de tenant inteiro** (`institutional.tenant.suspend`) — impacto em centenas de moradores.
6. **Edição de ABAC policies** (`abac.policy.create/update/delete`) — toca o motor de autorização.
7. **Broadcast global** (`platform.broadcast.*`) — reach plataforma inteira.

### 5.2 Fluxo canônico (D-054)

```
Admin A (solicita)                                          Admin B (aprova, ≤ 24h)
─────────────────                                           ───────────────────────
1. POST /admin/v1/bypass-requests                           1. GET /admin/v1/bypass-requests?status=pending
   body: {
     target_resource_type: "video",                         2. Revisa: solicitante, recurso, reason,
     target_resource_id: "vid_xyz",                            categoria, evidência
     action: "content.video.remove_pre90d",
     admin_reason: "...texto ≥ 20 chars...",                3. POST /admin/v1/bypass-requests/:id/approve
     category: "ordem_judicial"                                body: { approver_reason: "..." }
       | "erro_operacional"
       | "fraude_detectada"                                 4. Se approver == admin_A → 409 conflict
       | "solicitacao_titular_lgpd",                          (enforcement de segregação)
     evidence_url: "..." (opcional, obrigatório
       se categoria == ordem_judicial)                      5. Se approved < 24h após request → executa
   }                                                           handler original do recurso
                                                               + audit_trail (acão + both admin_ids)
2. Retorna { bypass_id, expires_at: now+24h }                  + lgpd_logs (se toca PII)
                                                               + email DPO + CEO (LGPD Art. 50)
3. Espera aprovação; notifica A por email/push               6. Se expired (24h sem aprovação) →
                                                               status = "expired", sem execução,
                                                               audit_trail entry "bypass.expired"
```

### 5.3 Categorias obrigatórias de `admin_reason`

Texto livre ≥ 20 chars + **uma** das categorias enumeradas (D-054):

| Categoria | Quando usar | Trilha extra |
|---|---|---|
| `erro_operacional` | Admin ou síndico errou (upload duplicado, categoria errada) | audit_trail |
| `ordem_judicial` | Juiz determinou remoção/retenção. **Exige** `evidence_url` apontando para ofício/decisão | audit_trail + lgpd_logs + **email DPO + jurídico** |
| `fraude_detectada` | Resposta a incidente de segurança interno | audit_trail + Sentry alert + security-incident record |
| `solicitacao_titular_lgpd` | Titular exerceu direito LGPD (deletion, portabilidade, retificação fora do fluxo padrão) | audit_trail + lgpd_logs + notifica DPO |

### 5.4 Regras duras

1. **Admin A ≠ Admin B** — segregação estrita; enforcement em DB CHECK + application-level guard em `/approve`.
2. **24h TTL** — após expira, o bypass é **morto**; re-solicitar é possível, mas cada solicitação é um AuditEntry novo.
3. **Reason obrigatório em A e B** — ambos motivos entram em `audit_trail.reason_chain` (campo estruturado).
4. **Step-up MFA** em A e B — ambos precisam emitir `/auth/step-up` antes de requester/approver.
5. **DPO + CEO notificados** em `ordem_judicial` e `solicitacao_titular_lgpd` — canal email transactional via `IEmailProvider` ([[../04-requirements/functional/cross-domain#XD-015]]), domínio `@mastersindico.com.br`.
6. **Zero exceção** — super-admin solo **não pode** bypassar 4-eyes; o próprio código não tem rota curta.

### 5.5 Interação com ADR-0026

ADR-0026 exige step-up em `POST /admin/v1/*`. 4-eyes **adiciona** uma camada: o handler de `/approve` exige step-up **do aprovador** E valida que o solicitante emitiu step-up na criação. A matriz fica:

| Rota | Step-up A | Step-up B | 4-eyes enforcement |
|---|---|---|---|
| `POST /admin/v1/bypass-requests` | **sim** | N/A | valida reason + categoria |
| `POST /admin/v1/bypass-requests/:id/approve` | N/A | **sim** | valida A ≠ B + expira 24h |

---

## § 6. MFA obrigatório M1

> Fonte canônica: [[../02-architecture/adr/0026-admin-mfa-m1]] (reclassificação IDN-007 de M2 para M1).

### 6.1 Baseline: MFA em login

- **Todos** com `role = admin` **precisam** ter TOTP configurado no Zitadel antes do primeiro login M1. Zitadel project config: `mfa_type = required`.
- Admin sem MFA → **Zitadel bloqueia no IdP**, antes de redirect para callback ([[../04-requirements/functional/identity#IDN-007]] Critério: "Admin sem MFA configurado não pode logar em M1").
- Backup codes: **10 códigos one-time-use** hasheados no Zitadel. Lockout operacional é **risco real** — runbook em `12-docs/runbooks/admin-mfa-lockout.md` (stub a criar, dependência DT-### a registrar em STATE).
- Recovery: via Super Admin com 4-eyes (outro super-admin aprova reset do TOTP — reaproveita o fluxo §5).

### 6.2 Step-up MFA em endpoints críticos

Header: `X-Step-Up-Token` (JWT curto 5 min, claim `amr=["mfa"]` + `acr=2` OIDC AAL2). Gerado por `POST /auth/step-up` (rate-limited 5 rpm por user_id — [[../04-requirements/functional/identity#IDN-036]]).

**Lista canônica de endpoints críticos** (alinhada a [[../02-architecture/adr/0026-admin-mfa-m1#decisão]]):

1. `DELETE /api/v1/me` — LGPD Art. 18 VI.
2. `DELETE /api/v1/me/devices/:id` — cobre IDOR A-DC-SEC-001.
3. `GET /api/v1/me/export` — portabilidade LGPD.
4. `POST /api/v1/assemblies/:id/minutes/publish` — Lei 4.591/64.
5. `POST /api/v1/contracts/:id/approve` quando `value ≥ R$ 10.000`.
6. `POST /api/v1/condominiums/:id/transfer-mandate`.
7. `PATCH /api/v1/me` com `email` ou `phone`.
8. **`POST /admin/v1/*`** — **qualquer** rota admin mutating.

### 6.3 Mecânica (ADR-0026)

- **Emissão**: `POST /auth/step-up` aceita código TOTP + emite JWT 5 min com `jti`.
- **Consumo**: handler crítico valida header `X-Step-Up-Token`, verifica (assinatura + expiração + `acr == 2` + denylist Redis `ms:stepup-denylist:<jti>`).
- **Burn after use**: `jti` vai para denylist com TTL = `token_exp - now`. Logout também invalida (insere em denylist).
- **Rotação de sessão** (IDN-038) revoga step-up token ativo.
- **Audit**: `step_up.emitted`, `step_up.consumed`, `step_up.rejected` em `audit_trail` por evento.

### 6.4 Alinhamento sub-papéis

| Sub-papel | MFA login M1 | Step-up em `/admin/v1/*` | Step-up em `/api/v1/*` não-admin |
|---|---|---|---|
| Super Admin | obrigatório | obrigatório | obrigatório se cross-tenant crítico |
| Moderador | obrigatório | obrigatório | N/A (não acessa rotas não-admin crítica) |
| Suporte | obrigatório | obrigatório (impersonate, revoke session) | obrigatório quando impersonando |
| Financeiro | obrigatório | obrigatório (refund, mark_paid) | N/A |

### 6.5 Futuro (M2+)

- **Passkeys WebAuthn** como alternativa ao TOTP — rejeitado em M1 (ADR-0026 §alternativas: UX iOS Safari inconsistente); **avaliar M2** em A-DC-SEC-014.
- **Hardware keys FIDO2** para super-admin — backlog M3+.

---

## § 7. Audit trail (LGPD + trilha organizacional)

> Fontes canônicas: [[../04-requirements/functional/cross-domain#XD-008]] (IAuditLogger DT-010), [[../04-requirements/functional/cross-domain#XD-009]] (append-only), [[../04-requirements/functional/cross-domain#XD-010]] (LGPD logs), [[../04-requirements/global#GR-026]], [[../01-domain/aggregates/AuditEntry]].

### 7.1 Duas tabelas distintas

| Tabela | Propósito | Base legal | Retenção | Trigger proibindo UPDATE/DELETE |
|---|---|---|---|---|
| `audit_trail` | Trilha **operacional** (todas ações com efeito de negócio: CRUD, aprovações, bans, bypass) | legitimate_interest + legal_obligation | **5 anos**, archive S3 Glacier após 1 ano | sim (`prevent_mutation_audit_trail`) |
| `lgpd_logs` | Trilha **LGPD** (revelação de PII, consent, deletion, portability) | legal_obligation (LGPD Art. 37) | **5 anos** + **purga** obrigatória pós-vencimento (direito ao esquecimento) | sim |

Admin ações **tocam as duas** quando há PII (ex: impersonation, export, hard-delete, aprovação de CNPJ acessando CPF). Regra de classificação:

- **Só audit_trail**: ações operacionais sem PII explícita (ex: `abac.policy.update`, `webhook.replay`).
- **Só lgpd_logs**: eventos puramente LGPD (ex: re-aceite de termo).
- **Ambas**: ações admin que revelam/modificam PII (ex: `identity.session.impersonate`, `lgpd.user.export_on_behalf`, `identity.user.hard_delete_lgpd`).

### 7.2 Schema resumido `audit_trail` (canônico — [[../01-domain/aggregates/AuditEntry]])

```
audit_trail (
  id UUIDv7 PK,
  action TEXT NOT NULL,                -- canônico: "admin_bypass", "delegation.revoked", ...
  actor_user_id UUID NULL,             -- NULL se system/cron
  actor_role TEXT NULL,
  target_type TEXT NOT NULL,           -- "user" | "video" | "subscription" | ...
  target_id TEXT NOT NULL,
  tenant_id UUID NULL,                 -- condominium_id | company_id
  outcome TEXT NOT NULL,               -- success | failure | denied
  reason TEXT NULL,                    -- ≥ 20 chars em ações que exigem
  metadata JSONB NOT NULL,             -- {ip_masked, user_agent, trace_id, request_id,
                                        --  payload_before_sanitized, payload_after_sanitized,
                                        --  consent_version, source_service}
  occurred_at TIMESTAMPTZ NOT NULL,    -- UTC (GR-033)
  archived_at TIMESTAMPTZ NULL,
  expires_at TIMESTAMPTZ NOT NULL      -- occurred_at + 5 anos
)
```

Invariantes (resumo — [[../01-domain/aggregates/AuditEntry#invariantes]]): INV-033 (append-only + archive 1 ano), INV-088 (admin_bypass sempre registra), INV-090 (schema sem updated_at + trigger Postgres), INV-091 (5 anos + hard-delete pós), INV-092 (PII masked em payload).

### 7.3 Ações admin que **sempre** gravam `audit_trail`

- Toda decisão ABAC `admin_bypass` (INV-088) — **sem exceção**.
- Toda rota `/admin/v1/*` (POST, PATCH, DELETE) — middleware `AuditLogger` obrigatório no roteador.
- Bypass 4-eyes: `bypass.requested`, `bypass.approved`, `bypass.executed`, `bypass.expired`, `bypass.denied`.
- Impersonation: `identity.session.impersonate.started` + `identity.session.impersonate.ended` + `identity.session.impersonate.action_in_scope`.
- Step-up: `step_up.emitted`, `step_up.consumed`, `step_up.rejected`.
- MFA: `mfa.enrolled`, `mfa.reset_requested` (4-eyes), `mfa.reset_applied`.

### 7.4 LGPD trail organizacional (LGPD Art. 50)

Quando admin usa 4-eyes com categoria `ordem_judicial` ou `solicitacao_titular_lgpd`:

1. Entrada em `lgpd_logs` com `legal_basis = legal_obligation | exercise_rights`, `purpose = "ordem_judicial" | "direito_esquecimento" | ...`.
2. Email transactional obrigatório para **DPO + CEO** com link para audit entry (LGPD Art. 50 trilha organizacional).
3. Relatório mensal consolidado (AD7) — super-admin exporta para auditoria externa.

### 7.5 Masking

PII nunca em logs zerolog (GR-### / [[../STEERING#20]] / INV-092). Em `audit_trail.metadata.payload_*`, campos de PII (CPF, CNPJ, email, telefone) são mascarados via `Masked()` antes de gravação. IP mascarado no render admin (`/24`, 3 primeiros octetos visíveis).

### 7.6 Integridade

- Checksums opcionais (M3+): hash-chain entre entries consecutivas para detecção de tampering.
- Trigger Postgres `prevent_mutation_audit_trail` bloqueia UPDATE/DELETE.
- CI static check grep: `UPDATE audit_trail`, `DELETE FROM audit_trail` ⇒ fail build.

---

## § 8. Broadcasts globais

> Telas: AD1 KPIs + formulário dedicado (stub M2 em [[web/ui-catalog]]). Actions: `platform.broadcast.send_email_global`, `platform.broadcast.send_push_global`.

### 8.1 Segmentação

- **Por persona**: `role IN (syndic, resident, enterprise, ...)` OU `role_type`.
- **Por plano**: `plan_tier IN (trial, base, plus, pro)` (enum universal — [[../02-architecture/adr/0032-global-plans-stripe-style]]).
- **Por tenant**: `tenant_id IN (...)` (condomínio/empresa específicos).
- **Por geografia**: CEP/bairro/cidade (derivado do profile).
- **Por status**: ativos em 30d / inadimplentes / em trial / suspended.
- **Por atributo**: consent opt-in email/push explicitamente válido (LGPD Art. 8º — [[../04-requirements/compliance-lgpd#LGPD-001]]).

### 8.2 Canais

- **Email** via `IEmailProvider` ([[../04-requirements/functional/cross-domain#XD-015]]) — Resend default, SES fallback (D-026 aberta → D-056 agnosticismo).
- **Push** via `IPushProvider` FCM ([[../04-requirements/functional/cross-domain#XD-019]]) — opt-in obrigatório.
- **SMS** **não é canal de broadcast M1–M2** — custo + LGPD sensível; só OTP/notificações críticas ([[../04-requirements/functional/cross-domain#XD-016]]).

### 8.3 Controles

- **Rate limit global**: máx 1 broadcast/dia por segmento, 5/dia plataforma.
- **Dry-run obrigatório**: preview com 10 destinatários aleatórios antes do send real.
- **Audit**: action + segmento serializado + count estimado + count enviado + bounce rate + complaint rate.
- **4-eyes**: obrigatório para broadcasts globais (`segment_type = "global"`).
- **Unsubscribe**: link obrigatório em email (CAN-SPAM + LGPD Art. 18).

---

## § 9. Moderação de conteúdo

> Fontes: [[../02-architecture/adr/0024-moderation-cascade]], [[web/ui-catalog#ad2-moderação-de-vídeos]] e [[web/ui-catalog#ad3-harassment-reports]], [[../04-requirements/functional/content]].

### 9.1 Vídeos — **48h SLA manual M1**

- M1: **100% manual**. Fila em AD2 com ações [Aprovar] / [Rejeitar com razão]. SLA 48h.
- Pre-publish: vídeos ficam `moderation_state = 'pending'`. Só viram `published` após aprovação.
- Lock 90d pós-publish ([[../STATE#D-054]]): remoção requer 4-eyes.
- Volume estimado M1: < 100 min vídeo / semana (startup). Operação suporta com 1 moderador parcial.
- M2: cascade Mux + Rekognition + OpenAI Moderation → triagem (gatilhos em ADR-0024 §gatilhos).
- M3: custom classifier + auto-approve low-confidence.

### 9.2 Harassment reports (AD3)

- Morador/síndico/empresa reporta usuário via `POST /api/v1/harassment-reports` (endpoint cross-tenant, rate-limited).
- Payload: `reporter_user_id`, `reported_user_id`, `context_type` (comment | dm | review | connect_me_msg), `context_id`, `reason`, `evidence_urls[]`.
- Moderador vê fila em AD3 com contexto completo.
- Ações:
  - `suspend_72h` (ban temporário — IDN-030).
  - `archive` (sem mérito).
  - `escalate` (sobe para super-admin + jurídico).

### 9.3 Appeals

- Usuário banido pode apelar via `POST /api/v1/me/appeal` com reason + evidence.
- Super-admin ou moderador sênior revê. Reverter ban pós-72h → `unban` + reason.
- Audit duplo (audit_trail + lgpd_logs — dados pessoais do envolvido).

### 9.4 LGPD direito à revisão humana

**LGPD Art. 20**: toda decisão automatizada (M2+) exige canal de revisão humana ([[../02-architecture/adr/0024-moderation-cascade#contexto]]). Moderação em cascade tem **sempre** opt-out para fila manual via AD2 mesmo quando classifier dá auto-approve.

---

## § 10. Gestão global de tenants

> Actions: `institutional.tenant.suspend`, `institutional.tenant.reactivate`, `institutional.cnpj.approve_manual`. Tela AD1 KPIs + CTA em AD5 (Usuários) filtrado por tenant.

### 10.1 Suspender

- `POST /admin/v1/tenants/:id/suspend` — 4-eyes obrigatório (§5).
- Efeito: `condominiums.status = 'suspended'` ou `companies.status = 'suspended'`. Cascata via NATS domain events: `institutional.tenant.suspended` consumido por billing (freeze subscription), content (revoga playback signed URLs), commercial (pausa Connect Me inflight).
- Login de usuários do tenant suspenso → resposta 403 `tenant_suspended` com mensagem.
- Dados preservados (não delete).

### 10.2 Reativar

- Idempotente. Reversão transacional via saga ([[../02-architecture/adr/0030-saga-orchestration-default]]).

### 10.3 Cross-tenant audit

- AD1 mostra **top 20 tenants por eventos críticos 7d** (governance_score queda, bans, suspensions, harassment reports).
- Consulta cross-tenant usa `admin_bypass = true` — cada linha do dashboard registra `audit_trail` com `action = 'admin.cross_tenant_query'` + lista de tenant_ids acessados.

---

## § 11. Aprovação manual de CNPJ

> Action: `institutional.cnpj.approve_manual`. Tela: AD1 CTA "Validações pendentes (moderação)" → drill-down em `/admin/cnpj-queue`.

### 11.1 Fluxo

```
Empresa registra via POST /signup?role=enterprise
  → CNPJ validador (ICNPJValidator) tenta algoritmo + receita federal stub (M1) / adapter real (M2+)
  → Se falha validação automática ou flag "pending_manual_review" → entra fila admin
  → Admin (financeiro ou super) revisa em AD1 drill-down:
      • CNPJ
      • Razão social
      • Situação cadastral (se integração M2+)
      • Documentos upload (contrato social, última alteração) — opcional M1
  → [Aprovar] cria empresa ativa + libera trial; [Rejeitar] manda email com motivo
  → Audit: action = "institutional.cnpj.approve_manual" | "institutional.cnpj.reject_manual", reason
```

### 11.2 Regras

- Reason obrigatório em aprovação **e** rejeição.
- Passar 7d em fila sem resposta → notifica admin + admin lead (SLA informal).
- Rejeição envia email com motivo + instruções para reenvio (integração IEmailProvider).

---

## § 12. Controle de inadimplentes

> Action: `billing.subscription.read` + CTAs em AD8. Flags e workflow.

### 12.1 Flags

- `subscriptions.status = 'past_due'` — webhook Stripe `invoice.payment_failed` atualiza.
- `users.flagged_as_delinquent = true` — derivado do `past_due`, facilita filtro cross-tenant.
- Dunning em 3 waves: 3d / 7d / 14d post-fail (emails via `IEmailProvider` + in-app banner).

### 12.2 Workflow admin

- AD8 lista `past_due` ordenado por valor + dias.
- Admin (financeiro) ações:
  - `billing.refund.create` se dunning errôneo.
  - `billing.plan.change_manual` se cliente mudou de plano sem sincronizar Stripe.
  - `billing.invoice.mark_paid` (4-eyes ≥ R$ 10k) — casos manuais.
  - Contatar cliente (log em support ticket externo — M2+).
- Após 30d sem pagamento → `billing.subscription.cancel` automático + downgrade para trial expired.

### 12.3 Audit

Toda ação em `billing.*` admin → audit_trail com action + invoice_id/subscription_id + reason + categoria + valor. Valores monetários sempre em cents (uuidv7+cents pattern).

---

## § 13. Dependências

Admin transversal depende **dos mesmos BCs** que ele privilegia. Mapa de dependências:

| BC | Dependência do Admin | Invariante chave |
|---|---|---|
| **identity** | role `admin` Zitadel + imutabilidade (IDN-029) + 1-device + step-up (IDN-036) + ban local (IDN-030) + sessions | INV-086..088, INV-101, INV-110 |
| **institutional** | cross-tenant read/suspend/reactivate, CNPJ approve, timeline read com affordance+ | INV-089 (tenant isolation desafiado por `admin_bypass` mas sempre auditado) |
| **billing** | Stripe refund/plan-change/webhook replay/mark_paid | [[../02-architecture/adr/0004-stripe-payment-gateway]], [[../02-architecture/adr/0027-webhook-idempotency-db]] |
| **content** | Moderação vídeos (AD2), lock 90d bypass (4-eyes), comment remove | [[../02-architecture/adr/0024-moderation-cascade]], GR-029 |
| **commercial** | Harassment reports (AD3), contract lock bypass | [[../04-requirements/functional/commercial]] |
| **assembly** | Audit read-only (integridade ata Lei 4.591/64). Admin **não** edita atas publicadas — nem com bypass | minutes_immutability |
| **cross-domain** | IAuditLogger (XD-008), audit append-only (XD-009), lgpd_logs (XD-010), ABAC engine (XD-004..007) | INV-090, INV-091 |

### 13.1 Providers (agnosticismo D-056)

Toda integração externa do admin passa por interface:

- `IIdentityProvider` (Zitadel hoje, troca → Keycloak/Auth0 amanhã sem código app quebrar).
- `IEmailProvider` (Resend/SES).
- `IPaymentGateway` (Stripe — refunds via interface).
- `IPushProvider` (FCM).
- `IAuditLogger` (DT-010 — legado acoplado vira interface; impl default Postgres `audit_trail`).

Admin **não pode** chamar SDK direto. Se uma tela admin precisa de uma operação nova do provider, a operação entra na interface canônica primeiro.

---

## § 14. Estado no código

### 14.1 O que existe hoje (derivado de `backend/internal/shared/middleware/` + [[legacy-reference]])

| Componente | Status |
|---|---|
| `RequirePermission(action, resource)` middleware | ✅ Produção (sprint 1-3 legado) |
| Engine ABAC `admin_bypass` primeiro check | ✅ Produção |
| Cache ABAC Redis 5 min | ✅ Produção |
| Tabela `users.role` com enum incluindo `admin` | ✅ Produção |
| Zitadel MFA TOTP (opt-in) | ✅ Produção — **precisa** mover para `required` M1 |
| Step-up MFA `/auth/step-up` | 🟡 Parcial / stub — ADR-0026 recém aceito 2026-04-23 |
| `audit_trail` table + trigger append-only | ✅ Produção — DT-010 ainda aberto (audit acoplado, promover para `IAuditLogger` interface) |
| `lgpd_logs` table | 🟡 Parcial — XD-010 spec canônico, schema ativo, integração com fluxos críticos em progresso |
| AD1–AD8 telas web | ❌ Stub — todas M2–M3 ([[../10-schedule/_moc]]: web "admin panel" é M3) |
| `/admin/v1/*` roteador | ❌ Stub — endpoints individuais existem espalhados; prefixo dedicado + middleware step-up pendente |
| 4-eyes workflow (`bypass_requests` table + handlers) | ❌ Não implementado — D-054 recém decidido |
| `admin_profiles` table (sub_role + allowed_actions) | ❌ Não implementado — D-058 recém decidido |
| IP allowlist admin | ❌ Backlog M3+ |

### 14.2 Sprints de implementação (alinha com [[../10-schedule/cronograma-macro]])

- **Sprint 8–9 (M2)**: `admin_profiles` table + migration · prefixo `/admin/v1/*` com middleware step-up · AD5 (usuários internos) + AD8 (billing admin) · IAuditLogger interface promoção (DT-010 closure).
- **Sprint 10 (M1 final 2026-05-08)**: MFA `required` em Zitadel para admin · step-up hardening + endpoints críticos (ADR-0026 closure).
- **Sprint 11–12 (M2)**: AD2 moderação vídeos (SLA 48h manual) + AD3 harassment + AD7 LGPD console + 4-eyes workflow.
- **Sprint 13–14 (M3)**: AD1 painel · AD4 ABAC editor · AD6 dashboards · IP allowlist super-admin.

---

## § 15. Gaps

Findings em aberto (registrar A-### em [[../AUDIT]] durante execução):

1. **G-ADMIN-01**: Mobile admin não coberto em M1–M2. Risk: emergências que exigem admin fora do office (ex: takedown de conteúdo em prime time). Mitigação M2: implementar AD2/AD3 em mobile read-only + "suspender conta" com 4-eyes via push notification para um segundo admin.
2. **G-ADMIN-02**: Path `/admin` vs subdomínio `admin.mastersindico.com.br` indeciso (`P-UI-003` em [[web/ui-catalog#pendências]]). Impacta IP allowlist, CSP, session cookie scope. **Decisão**: ADR específica pós-M1, default path em M1 (cookie compartilhado).
3. **G-ADMIN-03**: `admin_profiles.sub_role` esquema: enum rígido vs roles-as-data (tabela). Default: enum rígido em M2 (super_admin | moderator | support | finance), migração para tabela M3+ se aparecer quinto sub-papel.
4. **G-ADMIN-04**: Impersonation UX. Banner persistente, session termination on window close, confirmação extra para mutations. Especificar em [[web/ui-catalog]] antes de implementar (pendência de UX-spec).
5. **G-ADMIN-05**: Mobile push para 4-eyes approver — sem push mobile pré-M3, aprovador recebe email. Email ≤ 24h é apertado. Mitigação: notificação também em app web via WebSocket (se aprovador estiver logado).
6. **G-ADMIN-06**: Runbook lockout admin (perdeu TOTP + backup codes) ainda não escrito. DT-### a registrar.
7. **G-ADMIN-07**: `role_type` (sub_role) **não vai** para claim Zitadel (T8 legado — só no banco). Isso significa que `admin_profiles` é canonical, e race entre login e allowlist é possível se novo sub-role não for purge-invalidated no cache ABAC — cobrir em XD-006.
8. **G-ADMIN-08**: Fluxo `mfa.reset_applied` (recovery super-admin via 4-eyes) detalhado em runbook — não codificado em ADR ainda.

---

## § 16. Referências

### 16.1 Decisões (STATE)

- [[../STATE#D-054 — Lock 90d bypass admin via 4-eyes policy]] — 4-eyes obrigatório para bypass.
- [[../STATE#D-056 — Agnosticismo de provedor como invariante arquitetural (crítico)]] — providers via interface.
- [[../STATE#D-058 — Painel Admin MS NÃO é app separada — é role privilegiada transversal]] — fundação deste doc.

### 16.2 ADRs

- [[../02-architecture/adr/0003-zitadel-oidc-provider]] — IdP atual (abstraído via `IIdentityProvider`).
- [[../02-architecture/adr/0012-abac-redis-cache]] — cache ABAC 5 min.
- [[../02-architecture/adr/0024-moderation-cascade]] — moderação evolutiva.
- [[../02-architecture/adr/0026-admin-mfa-m1]] — MFA obrigatório M1 + step-up.
- [[../02-architecture/adr/0027-webhook-idempotency-db]] — idempotência webhook (relevante AD8).
- [[../02-architecture/adr/0028-lgpd-keyed-hmac]] — pseudonimização.
- [[../02-architecture/adr/0030-saga-orchestration-default]] — saga em suspender tenant.

### 16.3 Requirements

- [[../04-requirements/functional/identity#IDN-007]] — MFA admin M1.
- [[../04-requirements/functional/identity#IDN-015]] — role types (sub-papéis).
- [[../04-requirements/functional/identity#IDN-022]] — deletion LGPD 15d.
- [[../04-requirements/functional/identity#IDN-030]] — ban local.
- [[../04-requirements/functional/identity#IDN-036]] — step-up MFA.
- [[../04-requirements/functional/cross-domain#XD-004]]..[[../04-requirements/functional/cross-domain#XD-010]] — ABAC + audit + LGPD.
- [[../04-requirements/matrix-functional#módulo-transversal-admin-master-síndico]] — matriz v2 canônica.
- [[../04-requirements/compliance-lgpd]] — LGPD-001..LGPD-0NN.

### 16.4 Produto / UI

- [[../00-product/personas#6-admin-master-síndico-interno-da-plataforma]] — persona + sub-tipos canônicos.
- [[web/ui-catalog#7-admin-master-síndico]] — catálogo AD1–AD8.
- [[web/architecture]] — feature-sliced + routing.
- [[web/security]] — web security posture (CSP, CORS, subdomínio vs path).

### 16.5 Domínio

- [[../01-domain/invariants]] — INV-086 (ordem ABAC), INV-087 (default-deny), INV-088 (admin_bypass auditado), INV-089 (tenant isolation), INV-090 (audit append-only), INV-091 (retenção 5a + purga), INV-110 (step-up).
- [[../01-domain/aggregates/AuditEntry]] — agregado canônico.

### 16.6 Obsidian client vault (referência histórica)

- [[client-vault/01 - Arquitetura de Sistema/System Architecture - Identity]] — AuthModule + ABAC CASL (histórico; admin não explícito no doc legado).

### 16.7 Legislação

- Lei 13.709/2018 (LGPD) — Art. 18 VI, Art. 20, Art. 37, Art. 50.
- Lei 4.591/64 — nulidade de ata (impacta step-up em publish).
- NIST SP 800-53 AC-5 — segregação de função (base do 4-eyes).
- SOX 4-eyes principle.

### 16.8 Vizinhos

- [[_moc]] — MOC de 03-subprojects.
- [[../CLAUDE]] — contrato raiz do projeto.
- [[../STEERING]] — não-negociáveis (§16 ABAC, §20 PII).
- [[backend/security]] — implementação security posture.
- [[../08-security/BEYOND_CORP]] — zero-trust tenets (Tenet 6 — step-up).
- [[../11-audit/audit-log/2026-04-23-pentest-specs]] — findings A-DC-PEN-007 (MFA), A-DC-PEN-010, A-DC-PEN-012.
