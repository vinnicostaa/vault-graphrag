---
title: Beyond-Corp / Zero-Trust — Aplicações concretas ao Master Síndico v2
type: note
tags: [research, beyond-corp, zero-trust, security, architecture, master-sindico, gaps, recomendacoes]
source: 13-research/beyond-corp/_destilado.md + 08-security/BEYOND_CORP.md + 08-security/threat-model.md + 02-architecture/adr/ + 01-domain/invariants.md + STATE.md + AUDIT.md
created: 2026-04-23
updated: 2026-04-23
aliases: [beyond-corp-aplicacoes, gaps-research-vs-projeto]
---

# Beyond-Corp / Zero-Trust — Aplicações concretas ao Master Síndico v2

Cruzamento sistemático entre o [[_destilado]] (research canônico NIST/BeyondCorp/BeyondProd/CISA/Cloudflare/Okta/Microsoft) e a arquitetura atual do **Master Síndico v2** (`master-sindico-v2/`).

**Objetivo**: identificar gaps concretos, propor melhorias priorizadas por M1 (GA 2026-05-08) / M2 / M3+, indicar exatamente qual arquivo editar e qual ADR/INV/D-### criar.

**Método**: cada recomendação referencia (a) conceito do research por fonte, (b) localização na codebase-spec atual, (c) esforço S/M/L, (d) prioridade por milestone.

---

## 1. Estado atual — o que o projeto JÁ aplica

O Master Síndico v2 tem **maturidade BeyondCorp média-alta** pra um SaaS brasileiro pré-GA. Aplicações já formalizadas:

### 1.1 Identidade-centric (pilar 1 BeyondCorp)

- **ADR-0003** — Zitadel como IdP (OIDC + PKCE + MFA).
- **INV-001..INV-010** — identidade formal: 1 User MS = 1 User Zitadel; Device Fingerprint hash server-side; password delegado a Zitadel Argon2.
- **ADR-0026** — MFA step-up obrigatório para admin em M1.
- **INV-110** — step-up MFA `≤ 5min` em endpoints auth-críticos (delete /me, publish minutes, approve contract ≥ R$10k, transfer mandate, PATCH email/phone, POST /admin/v1/*).
- Cobertura do conceito Okta/Microsoft "verify explicitly" em [[../08-security/BEYOND_CORP]] §1 Tenet 6.

### 1.2 Device-centric (pilar 2 BeyondCorp)

- **ADR-0014** — 1-Device Policy (session bound a device_fingerprint).
- **INV-005** — ≤ MAX_ACTIVE_SESSIONS por user (default 1, BR-004).
- **INV-009** — DeviceFingerprint SHA-256 server-side (IR-011).
- **A-023** (Sprint 10 ativo) — device change audit log comparativo.
- [[../08-security/BEYOND_CORP]] §4 detalha trust_tier enum (`new | confirmed | suspicious | banned`) + roadmap M2 (Play Integrity / DeviceCheck).

### 1.3 ABAC / Policy Engine (pilar 3 BeyondCorp + NIST PE/PA/PEP)

- **ADR-0012** — ABAC + Redis cache.
- **INV-086** — ordem estrita `admin_bypass → action_configured → role_allowed → tenant_matches → plan_tier_sufficient` (DR-012).
- **INV-087** — default-deny.
- **INV-088** — admin bypass exige audit log.
- **INV-096** — `config.Validate()` falha startup se matriz ABAC vazia.
- PBT ≥ 400 casos em [[../08-security/BEYOND_CORP]] §3.4.
- Matriz detalhada em [[../08-security/BEYOND_CORP]] §3.

### 1.4 Per-session / per-request (pilar 3 NIST SP 800-207)

- JWT curto (access 10min mobile, 15min web) + refresh 30d rotativo bound a device_id ([[../08-security/BEYOND_CORP]] §1 Tenet 3).
- INV-106 — `session.Regenerate()` obrigatório em toda transição de auth-state (A-DC-PEN-003).
- INV-019 — soft-block imediato em expiração de plano.

### 1.5 Segmentação multi-tenant + assume breach

- **ADR-0021** — multi-tenant row-based.
- **ADR-0029** — `tenant_id` type-driven (INV-108).
- **INV-089** — toda query escopada por `condominium_id`/`company_id`.
- **INV-108** — VOs distintos `CondominiumID` / `CompanyID` / `UserID` / `NeighborhoodID` (A-DC-SEC-005 + A-DC-PEN-011).

### 1.6 Audit trail append-only

- **INV-024** — TimelineEntry imutável (DR-007).
- **INV-033** — `audit_trail` append-only 5 anos.
- **INV-090** — sem UPDATE/DELETE em audit.
- **DT-010** — `IAuditLogger` cross-módulo (Sprint 10).
- [[../08-security/BEYOND_CORP]] §11 schema `audit_log` com REVOKE UPDATE/DELETE.

### 1.7 Webhooks & HMAC (BeyondProd input boundary)

- **ADR-0027** — Webhook Idempotency DB (INV-105 UNIQUE `(provider, event_id)`).
- **INV-093** — HMAC verify antes de parse (DR-020).
- **INV-102** — `http.MaxBytesReader(64KB)` obrigatório (A-DC-SEC-002).
- [[../08-security/BEYOND_CORP]] §8 — Stripe/Mux/Livekit HMAC com timing attack protection.

### 1.8 CORS / edge / rate limit

- **INV-098** — CORS `*` bloqueado em staging/prod.
- **INV-103** — allowlist sem wildcard subdomain (A-DC-SEC-003).
- **INV-097** — rate limit `/auth/*` 20rpm / `/api/v1/*` 60rpm (A-032).
- [[../08-security/BEYOND_CORP]] §9 detalha allowlist e §6 rate limit escalonado.

### 1.9 LGPD + PII masking

- **ADR-0028** — LGPD keyed HMAC (INV-109 pseudonimização rotacionada anual).
- **INV-092** — PII nunca em logs (DR-019) + CI grep.
- **INV-104** — soft-flag `_pending_hard_delete` com grace 48h (A-DC-SEC-004).

**Resumo**: dos 7 tenets NIST + 6 princípios BeyondCorp + 3 Microsoft, o projeto já tem **~70% formalizado em spec** (sem código ainda). Os 30% pendentes são o foco das seções 2-6.

---

## 2. Gaps identificados — o que o research pede e ainda não está

### 2.1 Gap — Continuous Authentication Score ([[_destilado]] §2.2 Princípio 7 + Okta source-12)

**Research**: Microsoft/Okta/Google IAP exigem `risk_score` dinâmico alimentando policy engine (geolocalização nova, device novo, horário atípico, falhas ABAC sucessivas).

**Projeto**: [[../08-security/BEYOND_CORP]] §12.3 menciona "M2", mas **não existe** `risk_signals` table, nem endpoint, nem INV, nem ADR. Só placeholder. Token ABAC `context.risk_score` existe no pseudo-DSL mas ninguém popula.

**Severidade**: M2 blocker para hardening pós-GA (não bloqueia M1).

### 2.2 Gap — Device Trust Tier promotion automation ([[_destilado]] §3.1)

**Research**: Okta Device Assurance + Google IAP atribuem trust_tier automático por heurística (uptime + 0 falhas → `confirmed`; 3+ IPs/24h → `suspicious`).

**Projeto**: INV-005 + ADR-0014 cobrem 1-device, mas a **promoção/demoção do trust_tier é manual** (só ops). Heurística está documentada em [[../08-security/BEYOND_CORP]] §4.3 mas sem INV, sem job, sem ADR formal.

**Severidade**: M2 — M1 aceita tier-new default.

### 2.3 Gap — Platform Attestation (Play Integrity / DeviceCheck) ([[_destilado]] §3.7 + source-11 Tailscale)

**Research**: assembleia live tem **risco crítico** de fraude de votação (bot, device virtual, usuário duplicando voto entre devices). BeyondCorp exige atestação de plataforma.

**Projeto**: [[../08-security/BEYOND_CORP]] §4.3 planeja Play Integrity / DeviceCheck em M2, mas **INV-071 a INV-085 (assembly)** não exigem atestação. O voto só tem UNIQUE DB + fração ideal snapshot. Não há header/token de atestação.

**Severidade**: **M1 blocker para assembleia** — sindico pode contestar ata judicialmente se prova de identidade do votante for fraca. Proposta: **M1 lite (reCAPTCHA Enterprise + device fingerprint reforçado)** + **M2 full (Play Integrity)**.

### 2.4 Gap — BeyondProd mTLS leste-oeste ([[_destilado]] §1.4 + source-05)

**Research**: "Service trust should depend on characteristics like code provenance, trusted hardware, and service identity, rather than the location in the production network."

**Projeto**: [[../08-security/BEYOND_CORP]] §12.6 admite ausência em M1 (monolito-modular justifica), planeja M2 manual 3-caminhos e M3 Linkerd. **Nenhum ADR formaliza** a decisão M1-sem-mTLS e a migração pra M2.

**Severidade**: M2 blocker pós-decomposição microserviços. Precisa ADR explícito agora pra travar expectativa.

### 2.5 Gap — Binary Authorization / Code Provenance ([[_destilado]] §1.4 BeyondProd)

**Research**: "only binário que passou por revisão + build verificável pode rodar em prod" (cosign + SLSA).

**Projeto**: ADR-0017 (Railway M1 → AWS M4+) não fala de binary auth. `master-sindico-v2/10-schedule/` e `master-sindico-v2/11-audit/` não têm pipeline canônico com cosign. [[../08-security/BEYOND_CORP]] §14.3 menciona "Binary Authorization (cosign + Kyverno/OPA Gatekeeper) no pipeline" como M3 sem ADR.

**Severidade**: M3 nice-to-have.

### 2.6 Gap — WAF / Edge formal ([[_destilado]] §2.3)

**Research**: Cloudflare One ou CloudFront+AWS WAF como "Access Proxy" equivalente (tenet 2 NIST).

**Projeto**: [[../08-security/BEYOND_CORP]] §6 menciona "⚠️ PENDÊNCIA D-### a decidir" Cloudflare vs AWS WAF. **Sem ADR, sem D-###, sem prazo**. Railway não dá WAF nativo.

**Severidade**: M2 blocker (sem WAF em prod = DDoS vulnerability).

### 2.7 Gap — PDP externalizado (OPA / Cedar) ([[_destilado]] §3.2 + source-08 AWS Verified Permissions)

**Research**: NIST PE/PA/PEP separados. ABAC in-process é acceptable em startup mas vira débito conforme o monolito cresce.

**Projeto**: [[../08-security/BEYOND_CORP]] §12.2 menciona "⚠️ PENDÊNCIA ADR: OPA vs Cedar". Recomendação provisória OPA está no destilado mas **sem ADR formal**. ABAC engine é in-process hoje (ADR-0012 Redis cache, não externaliza decisão).

**Severidade**: M2 se o monolito-modular durar (performance dos policy eval em Go é boa); M3 se migrar pra microserviços.

### 2.8 Gap — Audit WORM / hash-chain ([[_destilado]] §3.4)

**Research**: LGPD Art. 16 + disputa judicial de ata exigem prova forense imutável. S3 Object Lock Compliance ou hash-chain rolling.

**Projeto**: INV-024 + INV-033 + INV-090 cobrem append-only por REVOKE GRANTS, mas **não hash-chain, não WORM storage**. [[../08-security/BEYOND_CORP]] §11.4 menciona "M2 hash-chain, M3 S3 Object Lock" sem ADR formal.

**Severidade**: **M2 blocker jurídico** — primeira assembleia contestada judicialmente sem hash-chain expõe o cliente (e o Master Síndico por omissão).

### 2.9 Gap — JIT/JEA para admin prod ([[_destilado]] §4.2 M2)

**Research**: sem credenciais admin permanentes; elevação temporária com TTL + approval (Teleport, AWS Session Manager, Cloudflare Access).

**Projeto**: ADR-0026 exige MFA admin M1, INV-088 exige audit de bypass, mas **não há JIT**. `painel admin MS` (D-061/D-072) é "role privilegiada transversal" — sem TTL em elevação.

**Severidade**: M3+ (Master Síndico tem time pequeno; M1-M2 aceitam admin permanente com MFA + audit).

### 2.10 Gap — Segmentação de rede defesa-em-profundidade ([[_destilado]] §3.6)

**Research**: "assume breach" — isolation via VPC/namespace + Postgres RLS além de WHERE tenant_id.

**Projeto**: ADR-0021 row-based multi-tenant é pragmático pro M1, mas **Postgres Row-Level Security (RLS) não está habilitado** — só convenção WHERE em sqlc. Um handler com bug de WHERE ausente vaza tenant.

**Severidade**: M2 — RLS como duplo-barrier é defesa-em-profundidade clássica.

### 2.11 Gap — Step-up auth via context awareness ([[_destilado]] source-07 Google IAP)

**Research**: IAP context-aware Access — política adaptativa por IP/geo/device/plataforma (não só per-endpoint).

**Projeto**: INV-110 cobre step-up por endpoint, mas não por **contexto dinâmico** (ex: IP inesperado → step-up mesmo em endpoint normal). [[../08-security/BEYOND_CORP]] §1 Tenet 7 menciona mas sem formalização.

**Severidade**: M3 nice-to-have.

### 2.12 Gap — Observability → Policy feedback loop ([[_destilado]] §1.2 Tenet 7)

**Research**: telemetria alimenta policy engine. Sinais → risk_score → policy.

**Projeto**: ADR-0020 (OpenTelemetry + Grafana + Sentry) coleta mas **não alimenta PDP**. Fluxo unidirecional: sinais → dashboard. Sem pipeline automático sinais → PDP.

**Severidade**: M3+ (depende de gap 2.1 `risk_signals` existir primeiro).

### 2.13 Gap — Maturity Assessment CISA ZTMM ([[_destilado]] §1.5)

**Research**: autoavaliação em 5 pilares × 4 estágios (Traditional → Initial → Advanced → Optimal).

**Projeto**: [[../08-security/BEYOND_CORP]] §14.3 menciona M3 "maturidade Advanced CISA ZTMM" mas **não há documento de assessment** que indique estágio atual por pilar. Sem baseline não há métrica de progresso.

**Severidade**: M2 — documento vivo fácil de manter, alimenta roadmap.

### 2.14 Gap — ZTNA para integrações externas ([[_destilado]] §4.3 + source-10 Cloudflare + source-11 Tailscale)

**Research**: Open Banking, registros, CRM via Cloudflare Access ou Tailscale (não VPN clássica).

**Projeto**: `03-subprojects/` e `04-requirements/` mencionam Open Banking (billing) mas sem ZTNA layer. Não há ADR pra conexão outbound segura.

**Severidade**: M3+ (depende de feature Open Banking ativar).

---

## 3. Recomendações priorizadas (top-10)

### R1 — Platform Attestation Lite para assembleia (M1)

| Campo | Valor |
|---|---|
| **Origem** | [[_destilado]] §3.7 + source-11 Tailscale + source-07 IAP |
| **Onde** | [[../08-security/BEYOND_CORP]] §4.3; **novo INV-111** em [[../01-domain/invariants]]; **novo ADR** `0033-assembly-attestation-lite.md`; afeta BC-assembly ([[../03-subprojects]]) |
| **Prioridade** | **M1** |
| **Esforço** | M |
| **Justificativa** | Risco jurídico crítico: ata contestada com prova de identidade fraca expõe cliente. Lite = reCAPTCHA Enterprise + fingerprint reforçado + UNIQUE device por assembly session. Full Play Integrity fica M2. |

### R2 — WAF / Edge ADR (M1→M2)

| Campo | Valor |
|---|---|
| **Origem** | [[_destilado]] §2.3 + source-10 Cloudflare |
| **Onde** | **novo ADR** `0034-edge-waf-cloudflare-vs-aws.md`; **novo D-###** em [[../STATE]]; atualizar [[../08-security/BEYOND_CORP]] §6 removendo "⚠️ PENDÊNCIA" |
| **Prioridade** | **M1 (decisão) → M2 (deploy)** |
| **Esforço** | S (decisão) / M (deploy) |
| **Justificativa** | Railway não dá WAF nativo; sem WAF → DDoS vulnerability em prod. Decisão pode travar agora (provável Cloudflare pelo free tier + DNS); deploy fica M2 com a migração de infra. |

### R3 — Hash-chain rolling no audit_log (M2 blocker)

| Campo | Valor |
|---|---|
| **Origem** | [[_destilado]] §3.4 + NIST source-02 Tenet 5 |
| **Onde** | **novo ADR** `0035-audit-hash-chain.md`; **novo INV-112**; migration em [[../02-architecture]] + spec em [[../08-security/BEYOND_CORP]] §11.4; impacta DT-010 |
| **Prioridade** | **M2 (pós-M1)** |
| **Esforço** | M |
| **Justificativa** | Primeira assembleia contestada sem prova forense imutável = risco LGPD + risco reputacional. REVOKE GRANTS não basta (admin postgres contorna). Hash-chain `prev_hash || row_content` adiciona prova matemática. |

### R4 — Postgres Row-Level Security (RLS) como duplo-barrier (M2)

| Campo | Valor |
|---|---|
| **Origem** | [[_destilado]] §3.6 "assume breach" |
| **Onde** | **novo ADR** `0036-postgres-rls-duplo-barrier.md`; atualizar ADR-0021; **novo INV-113**; migration por BC |
| **Prioridade** | **M2** |
| **Esforço** | M (migration por tabela + policies) / L (se incluir custom role por tenant) |
| **Justificativa** | Hoje um handler com WHERE ausente vaza tenant. RLS enforce no DB impede mesmo com bug de app. Defesa em profundidade clássica. Zero impacto em M1 (adicionar é transparente). |

### R5 — Continuous Authentication Score + risk_signals (M2)

| Campo | Valor |
|---|---|
| **Origem** | [[_destilado]] §2.2 Princípio 7 + source-07 IAP + source-12 Okta |
| **Onde** | **novo ADR** `0037-continuous-auth-risk-signals.md`; **novo INV-114**; nova tabela `risk_signals` + worker em [[../08-security/BEYOND_CORP]] §12.3; atualizar ABAC matrix §3 pra consumir `risk_score` (já previsto mas sem populador) |
| **Prioridade** | **M2** |
| **Esforço** | L |
| **Justificativa** | Gap mais citado no research (Tenet 4+7 NIST). Destrava step-up dinâmico + soft-lock. Começa simples: 4 sinais (IP novo, device novo, horário atípico, falhas ABAC > N/h) → tabela `risk_signals` → worker calcula `users.risk_score`. |

### R6 — Device trust_tier promotion automation (M2)

| Campo | Valor |
|---|---|
| **Origem** | [[_destilado]] §3.1 + source-12 Okta Device Assurance |
| **Onde** | **novo ADR** `0038-device-trust-tier-automation.md`; **novo INV-115**; worker job + schema `devices.trust_tier_updated_at`; atualizar [[../08-security/BEYOND_CORP]] §4.3 |
| **Prioridade** | **M2** |
| **Esforço** | S |
| **Justificativa** | Heurística já está escrita (`new → confirmed` em 7d / `suspicious` se 3+ IPs em 24h). Só falta formalizar em job + INV + ADR. Alimenta R5. |

### R7 — ADR mTLS policy (M1 decisão / M2 exec)

| Campo | Valor |
|---|---|
| **Origem** | [[_destilado]] §1.4 BeyondProd + source-05 |
| **Onde** | **novo ADR** `0039-mtls-policy-m1-m2-m3.md`; atualizar [[../08-security/BEYOND_CORP]] §12.6 removendo "⚠️ PENDÊNCIA" |
| **Prioridade** | **M1 (ADR) / M2 (exec 3-caminhos) / M3 (Linkerd)** |
| **Esforço** | S (ADR) / M (manual 3-caminhos M2) / L (mesh M3) |
| **Justificativa** | M1 monolito-modular dispensa mTLS leste-oeste (in-process). ADR formaliza "por que não agora + quando sim". Impede drift na decomposição. |

### R8 — CISA ZTMM Maturity Assessment vivo (M2)

| Campo | Valor |
|---|---|
| **Origem** | [[_destilado]] §1.5 + source-06 CISA |
| **Onde** | **novo arquivo** `08-security/ztmm-assessment.md` ([[../08-security/_moc]] atualizar); review trimestral em [[../08-security/BEYOND_CORP]] §15 |
| **Prioridade** | **M2** |
| **Esforço** | S (doc único + checklist) |
| **Justificativa** | Sem baseline por pilar (Identity/Devices/Networks/Apps/Data × Traditional/Initial/Advanced/Optimal), roadmap M3 "maturidade Advanced" é wishful thinking. Doc vivo + review trimestral = métrica de progresso real + input de auditor externo M3 (ISO 27001 / SOC 2). |

### R9 — Step-up auth context-aware (M3)

| Campo | Valor |
|---|---|
| **Origem** | [[_destilado]] source-07 Google IAP |
| **Onde** | **novo ADR** `0040-step-up-context-aware.md` (depende de R5); atualizar INV-110 pra aceitar step-up disparado por contexto; update [[../08-security/BEYOND_CORP]] §12.3 |
| **Prioridade** | **M3** |
| **Esforço** | M (depende de risk_signals) |
| **Justificativa** | INV-110 cobre step-up por endpoint; R9 estende pra "IP novo + endpoint normal = step-up". Depende de R5. |

### R10 — Binary Authorization no pipeline (M3)

| Campo | Valor |
|---|---|
| **Origem** | [[_destilado]] §1.4 BeyondProd + source-05 |
| **Onde** | **novo ADR** `0041-binary-authorization-cosign.md`; pipeline CI em `10-schedule/` + `11-audit/`; update [[../08-security/BEYOND_CORP]] §14.3 |
| **Prioridade** | **M3+** |
| **Esforço** | M (cosign + admission controller) |
| **Justificativa** | Só após migração Railway → AWS (ADR-0017 M4+). Prematuro pra Railway (sem Kubernetes). Bloco pra certificação ISO 27001 / SOC 2 em M4+. |

**Já coberto (não duplicar)**:
- "1-device policy" → já em ADR-0014 + INV-005 + [[../08-security/BEYOND_CORP]] §4.
- "MFA admin M1" → já em ADR-0026 + INV-110.
- "PKCE obrigatório" → já em ADR-0003 + [[../08-security/BEYOND_CORP]] §5.1.
- "HMAC webhook + idempotency" → já em ADR-0027 + INV-093 + INV-105.
- "CORS strict" → já em INV-098 + INV-103.
- "tenant_id type-driven" → já em ADR-0029 + INV-108.
- "LGPD keyed HMAC" → já em ADR-0028 + INV-109.
- "ABAC matrix" → já em ADR-0012 + INV-086..088 + [[../08-security/BEYOND_CORP]] §3.

---

## 4. Novas ADRs sugeridas

Numeração continua 0033+ (último existente: ADR-0032 `global-plans-stripe-style`):

| ADR | Nome-sugerido | Contexto (1 frase) |
|---|---|---|
| **ADR-0033** | `0033-assembly-attestation-lite` | Assembleia M1 usa reCAPTCHA Enterprise + device fingerprint reforçado + UNIQUE device por assembly session; full Play Integrity/DeviceCheck fica M2. |
| **ADR-0034** | `0034-edge-waf-cloudflare-vs-aws` | Decide WAF + edge: Cloudflare (DNS + Access + Workers) vs AWS WAF+CloudFront; provável Cloudflare pelo free tier + portabilidade (Railway agnostic). |
| **ADR-0035** | `0035-audit-hash-chain` | `audit_log` ganha `prev_hash` + `row_hash` calculado no INSERT; permite prova forense matemática independente de GRANTs Postgres; M3 dual-write S3 Object Lock Compliance. |
| **ADR-0036** | `0036-postgres-rls-duplo-barrier` | Habilita Postgres Row-Level Security por tabela tenant-scoped como duplo-barrier além do WHERE em sqlc; custom role por tenant fica M3+. |
| **ADR-0037** | `0037-continuous-auth-risk-signals` | Tabela `risk_signals` + worker calcula `users.risk_score` a partir de 4 sinais (IP novo, device novo, horário atípico, falhas ABAC > N/h); alimenta ABAC matrix check #9 (já previsto). |
| **ADR-0038** | `0038-device-trust-tier-automation` | Job promove `devices.trust_tier` `new → confirmed` em 7d sem falha ABAC; demote `→ suspicious` se 3+ IPs/24h ou 2+ fingerprints/7d; `banned` manual. |
| **ADR-0039** | `0039-mtls-policy-m1-m2-m3` | M1 monolito-modular sem mTLS leste-oeste (in-process); M2 manual 3-caminhos (identity↔billing, identity↔assembly, gateway↔BCs); M3 Linkerd automático. |
| **ADR-0040** | `0040-step-up-context-aware` | Estende INV-110: step-up MFA disparado por contexto dinâmico (IP/geo/device inesperado), não só por endpoint. Depende de ADR-0037. |
| **ADR-0041** | `0041-binary-authorization-cosign` | Pipeline CI assina imagens com cosign; admission controller (Kyverno/OPA Gatekeeper) rejeita deploy de imagem não-assinada. M4+ pós-migração AWS. |
| **ADR-0042** | `0042-jit-jea-admin-access` | Elevação admin em prod via Teleport / AWS Session Manager com TTL 1h + approval 4-eyes; sem credenciais permanentes em workstation dev. M3+. |
| **ADR-0043** | `0043-ztna-outbound-integrations` | Open Banking / registros / CRM outbound via Cloudflare Access tunnel ou Tailscale; evita IP allowlist frágil. M3+ condicional a feature ativar. |

---

## 5. Novas D-### e DT-### sugeridas

### Decisões (D-087..D-092) — registrar em [[../STATE]]

| ID | Título | Contexto |
|---|---|---|
| **D-087** | Assembleia M1 usa atestação-lite (reCAPTCHA + fingerprint reforçado) | Risco jurídico de ata contestada exige prova-de-humano já em M1; full Play Integrity fica M2 por custo de integração mobile. |
| **D-088** | Edge + WAF = Cloudflare (provisional) | Free tier + DNS + Workers + Access bundle; decisão re-visitável no M2 após migração Railway→AWS se houver incompatibilidade. |
| **D-089** | Audit hash-chain é M2 blocker, não M1 | M1 com REVOKE GRANTS aceita; M2 adiciona hash-chain antes da primeira assembleia contestável ir a juízo. |
| **D-090** | Postgres RLS é duplo-barrier opt-in por tabela em M2 | Não reescreve sqlc; habilita `ENABLE ROW LEVEL SECURITY` + policies `USING (condominium_id = current_setting('app.current_tenant')::uuid)`; SET via middleware. |
| **D-091** | `risk_score` é float [0,100] calculado async | Alimentado por worker que consome `risk_signals` table; nunca calculado no request path. ABAC lê valor cacheado (TTL 5min). |
| **D-092** | mTLS leste-oeste NÃO é M1 | Monolito-modular = comunicação in-process. Formalizar em ADR-0039 pra não virar drift na decomposição. |

### Dívidas técnicas (DT-011..DT-015) — registrar em [[../STATE]] §Dívidas

| ID | Título | Contexto |
|---|---|---|
| **DT-011** | WAF / edge sem ADR | Gap identificado: [[../08-security/BEYOND_CORP]] §6 tem "⚠️ PENDÊNCIA" sem prazo. Fechar com ADR-0034 antes de GA. |
| **DT-012** | Trust_tier promotion é manual (só ops) | Heurística documentada mas sem worker/job. Fechar com ADR-0038 em M2. |
| **DT-013** | `risk_signals` table não existe | ABAC matrix §3 lê `context.risk_score` mas ninguém popula. Fechar com ADR-0037 em M2. |
| **DT-014** | mTLS policy sem ADR formaliza M1→M2→M3 | Risco de drift na decomposição de microserviços. Fechar com ADR-0039 antes de GA. |
| **DT-015** | CISA ZTMM baseline ausente | Sem baseline por pilar, roadmap M3 é wishful thinking. Fechar com doc `ztmm-assessment.md` em M2. |

---

## 6. Novos invariantes sugeridos (INV-111..INV-120)

Extensão da série `security hardening` (INV-101..INV-110) em [[../01-domain/invariants]] §9.

| ID | Invariante | Owner | Enforcement |
|---|---|---|---|
| **INV-111** | Voto em assembleia requer `X-Assembly-Attestation` header: reCAPTCHA Enterprise token score ≥ 0.7 + device_fingerprint match com sessão; M2 Play Integrity/DeviceCheck token verificado server-side | assembly | Middleware `AssemblyAttestationCheck` + Integration test (fake recaptcha) + PBT (score < 0.7 → 403) |
| **INV-112** | `audit_log.prev_hash` = SHA256 da linha anterior; `audit_log.row_hash` = SHA256(prev_hash \|\| row_content); INSERT único fecha a cadeia; verificação offline via CLI pode provar tampering | cross-domain | Trigger Postgres `BEFORE INSERT` calcula hashes; CLI `audit-verify` em `/scripts/` valida cadeia |
| **INV-113** | Toda tabela tenant-scoped tem `ENABLE ROW LEVEL SECURITY` + policy `USING (condominium_id = current_setting('app.current_tenant')::uuid OR current_setting('app.is_admin')::bool)`; middleware `SetTenantCtx` injeta via `SET LOCAL app.current_tenant = $1` | cross-domain | Migration + Integration cross-tenant test sem WHERE no handler → PG rejeita linha |
| **INV-114** | `users.risk_score` ∈ [0, 100] calculado async por worker; stale ≤ 5min; ABAC consulta cache Redis; ausência do score → trata como 0 (fail-open em edge case, não bloqueia ação legítima por faltar sinal) | cross-domain / identity | Worker `risk_score_calculator` + Redis cache + PBT (score ≥ 80 + endpoint sensível → step_up_required) |
| **INV-115** | `devices.trust_tier` promovido `new → confirmed` após 7d sem falha ABAC e sem IP change > 3/24h; demote `→ suspicious` se fingerprint muda 2x/7d ou falhas ABAC > 5/h; `banned` manual por ops com `admin_reason` obrigatório | identity | Worker `device_trust_updater` (daily) + audit_log emission + Integration |
| **INV-116** | mTLS leste-oeste entre serviços internos em M2+: cert rotacionado ≤ 90d via SPIFFE/SPIRE (M3) ou CronJob manual (M2); handshake falha → fail-closed | cross-domain | Ingress mesh policy + health check mTLS + alert se cert expira < 14d |
| **INV-117** | Toda imagem container em prod é assinada com cosign usando key guardada em AWS KMS; admission controller rejeita unsigned; rollback permitido apenas de image-tag previamente assinada | cross-domain (infra) | Cosign em CI + Kyverno policy + Integration pipeline deploy-reject |
| **INV-118** | Admin prod access via JIT: TTL ≤ 1h + approval 4-eyes + `admin_reason` obrigatório + audit log; sem credenciais admin permanentes em workstation dev | identity | Teleport ou AWS SSM Session Manager + audit_log + policy |
| **INV-119** | Outbound integrations (Open Banking, registros, CRM) via Cloudflare Access tunnel ou Tailscale; IP allowlist frágil não é suficiente | cross-domain (integrações) | Network policy + Integration health check |
| **INV-120** | Policy ABAC versionada em Git com review obrigatório (2 approvals); deploy via CI, nunca hot-patch; rollback preserva auditabilidade | cross-domain | GitOps + CODEOWNERS + CI deploy-gate |

---

## 7. Resumo executivo (≤ 400 palavras)

O Master Síndico v2 chega pré-GA (2026-05-08) com **maturidade BeyondCorp média-alta**: 7 tenets NIST mapeados em [[../08-security/BEYOND_CORP]], 110 invariantes canônicos, 32 ADRs formalizadas, ABAC matrix com PBT ≥ 400 casos, step-up MFA per-endpoint (INV-110), 1-device policy (ADR-0014), webhook idempotency DB (ADR-0027), LGPD keyed HMAC (ADR-0028) e tenant_id type-driven (ADR-0029/INV-108). Cobre ~70% do research destilado em spec.

Os **top-5 gaps com maior impacto** que o cruzamento revelou:

1. **Assembly Attestation Lite (R1, M1 blocker)** — risco jurídico de ata contestada sem prova-de-humano. Fix: reCAPTCHA Enterprise + fingerprint reforçado em M1 (ADR-0033 + INV-111); Play Integrity/DeviceCheck em M2. Único gap que ainda bloqueia GA.
2. **Audit hash-chain (R3, M2 blocker jurídico)** — REVOKE GRANTS não basta pra disputa judicial; admin DB pode contornar. Fix: `prev_hash || row_hash` em `audit_log` via trigger Postgres + CLI offline de verificação (ADR-0035 + INV-112).
3. **Continuous Auth Risk Signals (R5, M2)** — ABAC matrix já lê `context.risk_score` mas ninguém popula. Fix: tabela `risk_signals` + worker calculando score por 4 sinais (ADR-0037 + INV-114). Destrava step-up dinâmico e soft-lock.
4. **Postgres RLS duplo-barrier (R4, M2)** — handler com WHERE ausente vaza tenant. Fix: `ENABLE ROW LEVEL SECURITY` por tabela tenant-scoped + middleware `SET LOCAL app.current_tenant` (ADR-0036 + INV-113). Zero impacto em M1.
5. **WAF/Edge ADR (R2, M1 decisão / M2 deploy)** — Railway não dá WAF; sem WAF em prod = DDoS vulnerability. Fix: decidir Cloudflare (free tier + DNS + Access) vs AWS WAF já em M1 (ADR-0034); deploy fica M2 com migração de infra.

**Artefato gerado**: `/home/vinni/Documentos/Projetos/Privados/automation-agents/master-sindico-v2/13-research/beyond-corp/_aplicacoes-master-sindico.md`

Contém: Estado atual em 9 facetas (§1), 14 gaps pormenorizados (§2), 10 recomendações priorizadas com arquivo-nominal/esforço/justificativa (§3), 11 ADRs novas sugeridas ADR-0033..0043 (§4), 6 D-### + 5 DT-### novos (§5), 10 INVs novos INV-111..120 (§6).

**Próximos passos**: consolidar R1+R2 como pré-M1 em [[../STEERING]] §não-negociáveis; abrir ADR-0033 e ADR-0034 na próxima sessão com dual-check; registrar D-087..D-092 e DT-011..DT-015 em [[../STATE]] ao ritmo do roadmap.

---

## 8. Links

- [[_destilado]] — research canônico destilado
- [[../08-security/BEYOND_CORP]] — spec atual BeyondCorp do projeto
- [[../08-security/beyond-corp-references]] — ponte research × projeto
- [[../08-security/threat-model]] — STRIDE + LINDDUN por BC
- [[../01-domain/invariants]] — 110 INVs canônicos (este doc propõe +10)
- [[../02-architecture/adr/_moc]] — 32 ADRs ativas (este doc propõe +11)
- [[../STATE]] — decisões vivas D-### (este doc propõe +6 D + 5 DT)
- [[../AUDIT]] — findings abertos
- [[../STEERING]] — não-negociáveis
- [[../ROADMAP]] — milestones M1/M2/M3
- Sources: [[source-01]]..[[source-12]]
