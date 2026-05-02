---
title: ADR Index — Master Síndico v2
type: guide
tags: [adr, index, architecture, master-sindico, v2]
source: 02-architecture/adr/ + STATE + DT-002 + Fase 5 dual-check
created: 2026-04-23
updated: 2026-04-23
---

# ADR Index — Master Síndico v2

Índice de **Architecture Decision Records** (ADR-####). Cada ADR formaliza uma decisão arquitetural correspondente a um D-### em [[../STATE]] + é gerado via [[../06-execution/playbooks/research-loop]] + [[../06-execution/playbooks/dual-check]].

**Dívida técnica DT-002** ([[../STATE#dt-002-formalizar-20-adrs-planejadas]]): legado lista 20 ADRs "planejadas" sem arquivo concreto; esta remontagem formaliza todas.

---

## ADRs planejadas (do legado, a formalizar em Fase 3)

| ADR | Título | Status | D-### correspondente | Data alvo |
|---|---|---|---|---|
| ADR-0001 | Clean Architecture estrita com camadas domain ← app ← infra | 📝 a escrever | (inerente ao STEERING §6) | Sprint 10 |
| ADR-0002 | DDD tático — aggregates privados + factories NewX/ReconstructX | 📝 a escrever | (STEERING §7) | Sprint 10 |
| ADR-0003 | CQRS — command ≠ query (arquivos separados) | 📝 a escrever | (STEERING §8) | Sprint 10 |
| ADR-0004 | Multi-tenant estrito — `WHERE condominium_id` em toda query escopada | 📝 a escrever | D-006 (topologia, 🟡 aberto) | Sprint 11 |
| ADR-0005 | Zitadel como IdP (OIDC Authorization Code + PKCE) | ✅ implementado | D-001 legado | — |
| ADR-0006 | Stripe como billing provider (único; sem fallback) | ✅ implementado | D-005 legado | — |
| ADR-0007 | Mux como vídeo provider (HLS + presigned upload + signed playback) | ✅ implementado | (D-### implícito) | — |
| ADR-0008 | Livekit como SFU WebRTC (live assembleias) | ✅ implementado | D-006 legado + D-009 v2 | Sprint 14 (dual-check D-009) |
| ADR-0009 | UUIDv7/ULID PK (time-ordered) — **updated 2026-04-23** (UUIDv7 default em novas tabelas pós-PG18) | 🔄 atualizado | D-055 | Fase 5 |
| ADR-0010 | Redis 7 — ABAC cache TTL 5min + invalidação via webhook | ✅ implementado | D-001 legado (🟡 aberto) | Sprint 11 |
| ADR-0011 | sqlc v1.30 (abandonar Drizzle beta — AP-011) | ✅ implementado | — | — |
| ADR-0012 | goose v3 — migrations versionadas | ✅ implementado | — | — |
| ADR-0013 | zerolog structured + PII mask no middleware | ✅ implementado | (AP-010 mitigado) | — |
| ADR-0014 | UnitOfWork intra-módulo + Saga inter-módulo | ✅ implementado | (STEERING §9) | Sprint 11 |
| ADR-0015 | UoW intra-BC / Saga inter-BC — **updated 2026-04-23** (saga **orchestration explícita** como default em M1; choreography só em saga ≤ 2 steps + 1 compensation sem provider externo) | 🔄 atualizado | D-009 legado + P-CM-004 | Fase 5 |
| ADR-0016 | Event-driven seletivo — fluxos específicos NATS JetStream | 📝 a escrever | D-007 v2 | Sprint 12 |
| ADR-0017 | OpenSearch prod + PG tsvector dev — **partial-superseded** por ADR-0042 (search real M1) | 🔄 partial-superseded | D-007 legado / D-120 revoga | — |
| ADR-0018 | Resend (abandonar SES) | ✅ implementado | D-026 legado | — |
| ADR-0019 | Railway infra (vs AWS ECS Fargate pós-escala) | 🟡 aberto | D-024 legado + D-002 v2 | Q3 2026 (pós-M3) |
| ADR-0020 | Reputação Bronze→Diamante — fórmula congelada vs parametrizável | 📝 a escrever | D-010 v2 | Sprint 11 |

---

## ADRs novas pós-research (Fase 2-3 da remontagem)

| ADR | Título | Status | D-### | Data alvo |
|---|---|---|---|---|
| ADR-0021 | Feed/recomendação Connect Me — determinístico vs ML | 📝 pending research | D-008 v2 | Sprint 12+ |
| ADR-0022 | Agência Marketing — actor delegado (scoped tokens) ou tenant próprio | 📝 pending | D-011 v2 | M5+ |
| ADR-0023 | Empresa-síndica — MVP ou M5+ | 📝 pending | D-012 v2 | M5+ |
| ADR-0024 | LMS — M3 thin vs M4 pleno | 📝 pending research | D-013 v2 | Sprint 14 |
| ADR-0025 | Moderação — manual vs ML híbrido | 📝 pending research | D-014 v2 | M4+ |

> ⚠️ **Renumeração Fase 5 (2026-04-23)**: os antigos placeholders "ADR-0026 Observability stack" e "ADR-0027 Livekit vs custom" foram **deslocados** para ADR-0032 / ADR-0033 (backlog) — os slots 0026-0031 passam a ser usados pelas ADRs da Fase 5 (dual-check SEC+PEN+QA) listadas na seção seguinte. Justificativa: ADRs Fase 5 são **pré-M1 bloqueantes**; os placeholders são M3+ research. Ver [[../STATE#decisoes-fase-5-dual-check-arquitetural-d-046-d-055]].

| ADR-0032 | Observability stack — LGTM vs Datadog vs New Relic (antigo 0026) | 📝 pending | D-015 v2 | M3+ |
| ADR-0033 | Livekit vs WebRTC custom vs Meet-like SFU (antigo 0027) | 📝 pending research | D-009 v2 | Sprint 14 |

---

## ADRs Fase 5 — Dual-check SEC+PEN+QA (2026-04-23)

Seis ADRs novas geradas pelo pass adversarial (dual-check de security + pentester + QA) sobre specs da remontagem. Cada uma fecha um finding crítico ou high de [[../AUDIT]].

| ADR | Título | Status | D-### / Finding | Data | Fonte |
|---|---|---|---|---|---|
| ADR-0026 | admin-mfa-m1 — Step-up MFA obrigatório em admin M1 (DELETE /me, publish minutes, approve ≥ R$10k) | 📝 a escrever | A-DC-PEN-007 + Gap-SEC-002 reclassificado | 2026-04-23 | Pentester pass §2.7 + dual-check SEC §Gap-SEC-002 |
| ADR-0027 | webhook-idempotency-db — Idempotência webhook via **DB UNIQUE** (durável) em `webhook_events`; Redis é cache | 📝 a escrever | A-DC-PEN-016 + A-DC-PEN-027 | 2026-04-23 | Pentester pass §3 (inconsistência api-design §9.1 vs BIL-010) |
| ADR-0028 | lgpd-keyed-hmac — Pseudonimização LGPD via **HMAC keyed** com secret rotating (não hash determinístico) | 📝 a escrever | A-DC-PEN-038 + IDN-022 refinement | 2026-04-23 | Pentester pass + LGPD Art. 12 |
| ADR-0029 | tenant-id-typed — VOs distintos em Go (`CondominiumID`, `CompanyID`, `UserID`) em vez de `TenantID` genérico | 📝 a escrever | A-DC-SEC-005 + A-DC-PEN-005 + P-CM-003/P-EVT-001 | 2026-04-23 | Dual-check SEC + arquitetura §3.2 |
| ADR-0030 | saga-orchestration-default — Orchestration > choreography em M1 (sagas ≥ 3 steps + compensation + provider externo) | 📝 a escrever | P-CM-004 + ressalva ADR-0015 | 2026-04-23 | Dual-check arquitetural §1.12, §3.5 |
| ADR-0031 | email-provider-resend-m1 — Resend M1-M2, SES M3+ (revisão D-026 legado) | 📝 a escrever | D-046 | 2026-04-23 | Dual-check arquitetural §2.3 |

Updates em ADRs existentes (mesma data):

| ADR | Atualização | Motivo |
|---|---|---|
| **ADR-0009** | UUIDv7 nativo PG18 como default em novas tabelas pós-2026-04-23; tabelas antigas permanecem ULID | D-055 + PG18 `uuidv7()` nativo |
| **ADR-0015** | Saga **orchestration explícita** como default em M1 (sagas ≥ 3 steps + compensation + provider externo usam orchestrator in-house `saga_instances`) | P-CM-004 + dual-check §1.12 |

---

## ADRs potenciais (identificadas mas não priorizadas)

| Tema | Quando decidir |
|---|---|
| Disaster Recovery — single-region vs multi-region | Q3 2026 pós-escala |
| Backup strategy (snapshots PG) | M1 (runbook) |
| Testing DB — testcontainers vs shared CI DB | já implementado testcontainers-go |
| Secrets management — Railway env vs AWS Secrets Manager vs Vault | Q3 2026 |
| Mobile offline-first — sync strategy | M6+ |

---

## Processo de criação de ADR

Ver [[how-to/criar-adr]].

Template: [[../02-architecture/adr/template]] (a criar — herdar de `vault/40-templates/`).

---

## Tabela de consulta rápida

| # | Título | Status | Impacto |
|---|---|---|---|
| 0001 | Clean Architecture | 📝 | Todo o código |
| 0002 | DDD tático | 📝 | Modelagem domain |
| 0003 | CQRS | 📝 | Use cases |
| 0004 | Multi-tenant | 📝 | Queries + ABAC |
| 0005 | Zitadel IdP | ✅ | Auth |
| 0006 | Stripe billing | ✅ | Pagamentos |
| 0007 | Mux vídeo | ✅ | Content |
| 0008 | Livekit SFU | ✅ | Assembly |
| 0009 | UUIDv7/ULID PK (updated) | 🔄 | PK/Persistence |
| 0010 | Redis cache | ✅ | ABAC + quota |
| 0011 | sqlc | ✅ | DB |
| 0012 | goose | ✅ | Migrations |
| 0013 | zerolog + PII mask | ✅ | Logging |
| 0014 | UoW + Saga | ✅ | Transações |
| 0015 | UoW/Saga (updated: orch default) | 🔄 | Transações |
| 0016 | NATS JetStream seletivo | 📝 | Event-driven |
| 0017 | OpenSearch prod | ✅ | Busca |
| 0018 | Resend email | ✅ | Email |
| 0019 | Railway infra | 🟡 | Deploy |
| 0020 | Reputação fórmula | 📝 | Commercial |
| 0021 | Feed Connect Me ML? | 📝 | Commercial |
| 0022 | Agência tenant? | 📝 | Identity |
| 0023 | Empresa-síndica? | 📝 | Institutional |
| 0024 | LMS thin/pleno | 📝 | Content |
| 0025 | Moderação ML | 📝 | Content |
| 0026 | admin-mfa-m1 (Fase 5) | 📝 | Auth/Admin |
| 0027 | webhook-idempotency-db (Fase 5) | 📝 | Integrations |
| 0028 | lgpd-keyed-hmac (Fase 5) | 📝 | LGPD/Crypto |
| 0029 | tenant-id-typed (Fase 5) | 📝 | DDD/Types |
| 0030 | saga-orchestration-default (Fase 5) | 📝 | Transactions |
| 0031 | email-provider-resend-m1 (Fase 5) | 📝 | Email |
| 0032 | global-plans-stripe-style (Fase 8) | ✅ | Billing/Plans |
| 0033 | ata-timeline-db-immutability (Fase 9) | ✅ | Institutional/LGPD |
| 0034 | postgres-rls-default-on-tenant-guard (Fase 9) | ✅ | Security/Multi-tenant |
| 0035 | postgres-pitr-wal-archiving (Fase 9) | ✅ | DR/Backup |
| 0036 | (reservado) | 📝 | — |
| **0037** | **soft-delete-universal-pseudonymize (Fase 11)** | ✅ | **LGPD/Privacy** |
| **0038** | **validatable-interface-cross-bc (Fase 12)** | ✅ | **DDD/Cross-BC** |

---

## ADRs Fase 11 — fechamento LGPD (2026-04-24)

- **[[../02-architecture/adr/0037-soft-delete-universal-pseudonymize\|ADR-0037]]** — soft_delete universal + pseudonimização HMAC-keyed + retenção 5y. Resolve A-DC-SEC-004 + LGPD-M1-007 (2 bloqueadores M1). Deriva de [[../STATE#d-101-—-lgpd-soft-delete-universal-pseudonimização-hmac-keyed-fecha-a-dc-sec-004-lgpd-m1-007\|STATE D-101]]. Reusa mecanismo HMAC de ADR-0028.

## ADRs Fase 12 — DDD cross-BC + produto (2026-04-24)

- **[[../02-architecture/adr/0038-validatable-interface-cross-bc\|ADR-0038]]** — IValidatable interface cross-BC (Opção B DDD puro). Em vez de aggregate único polimórfico `ValidationItem`, define interface implementada por 5 aggregates commercial (Proposal, ServiceRecord, TechnicalActivity, Adjustment, PostDealAnnouncement). Hub institutional consome via view materializada `pending_validations` ou endpoint agregador. Deriva de [[../STATE#d-105\|STATE D-105]]. Spec pattern em [[../01-domain/patterns/validatable-interface]].

---

## Links

- [[../02-architecture/adr/_moc]]
- [[../STATE]]
- [[../06-execution/playbooks/research-loop]]
- [[../06-execution/playbooks/dual-check]]
- [[how-to/criar-adr]]
