---
title: Tasks — cross-domain (ABAC / Audit / LGPD / Providers)
type: spec
tags: [tasks, by-module, cross-domain, master-sindico, v2]
source: 04-requirements/functional + STATE DT-010 + AUDIT A-024/A-031/A-032 + backlog.md
created: 2026-04-23
updated: 2026-04-23
---

# Tasks — `cross-domain`

Tasks **transversais** que não pertencem a BC único — IAuditLogger, ABAC engine, Domain Event Bus, LGPD Registry, providers externos compartilhados (Resend, Twilio, FCM, ViaCEP), antifraude.

Alinhamento: [[../../01-domain/bounded-contexts#cross-domain]] + [[../../08-security/BEYOND_CORP]].

---

## M1

| ID | Título | Sprint | Status |
|---|---|---|---|
| T-700 | OTel completo em 3 sub-projetos (traces 80%+ endpoints) | 10 | ⏳ |
| T-701 | Sentry prod em backend + web + mobile | 10 | ⏳ |
| T-704 | DT-010 — `IAuditLogger` cross-módulo (append-only `audit_trail` 5 anos) | 10 | ⏳ |
| T-707 | LGPD auditoria externa (consent + retention + export + delete) | 10 | ⏳ |
| T-708 | `Config.Validate()` bloqueia CORS wildcard em staging/prod | 10 | ⏳ |
| T-800 | DT-001 — consolidar gaps client-vault × vault ativo | 10 | ⏳ |
| T-801 | DT-002 — formalizar 20 ADRs planejadas (ADR-0001..0020) | 10 | ⏳ |
| T-803 | DT-004 — Timeout em `UoW.Run` | 10 | ⏳ |

## M2

| ID | Título | Sprint | Status |
|---|---|---|---|
| T-703 | k6 load test 1000 concurrent users | 12 | ⏳ |
| T-704a | Mux webhook drop reprocessing | 13 | ⏳ |
| T-704b | Stripe webhook duplicado idempotency ledger | 13 | ⏳ |
| T-704c | Livekit disconnection contingency mode | 13 | ⏳ |
| T-704d | Redis cache down → ABAC fallback DB | 13 | ⏳ |
| T-811 | A-024 — `rawBodyReader` elevado ao contracts/core | 12 | ⏳ |
| T-812 | A-031 — Event Response TOCTOU dual-check fechamento | 12 | ⏳ |
| T-813 | A-032 — Goroutines "context-aware shutdown" pattern | 12 | ⏳ |
| T-802 | DT-003 — destilar `structured_dump.md` (grep dirigido) | 12 | ⏳ |

## M3

| ID | Título | Sprint | Status |
|---|---|---|---|
| T-710 | SLO dashboards Grafana por sub-produto | 16 | ⏳ |
| T-711 | Alertas PagerDuty (error rate, p99, webhooks) | 16 | ⏳ |

## Pós-M3 (fechar D-### abertos)

| ID | Título | Status |
|---|---|---|
| T-750 | Fechar D-006 (topologia multi-tenant) via research + dual-check → ADR | ⏳ |
| T-751 | Fechar D-007 (event-driven vs request-response) → ADR | ⏳ |
| T-752 | Fechar D-008 (feed/recomendação Connect Me) → ADR | ⏳ |
| T-753 | Fechar D-009 (Livekit vs custom WebRTC vs Meet SFU) → ADR (pós-M3) | ⏳ |
| T-754 | Fechar D-011 (Agência Marketing tenant vs actor) → ADR | ⏳ |
| T-755 | Fechar D-012 (Empresa-síndica MVP ou backlog) → ADR | ⏳ |
| T-756 | Fechar D-013 (LMS M3 thin vs M4 pleno) → ADR | ⏳ |
| T-757 | Fechar D-014 (moderação manual vs ML híbrido) → ADR | ⏳ |
| T-758 | Fechar D-015 (LGTM vs Datadog vs New Relic) → ADR | ⏳ |

---

## Invariantes cobertos

I-051..I-059 de [[../../01-domain/bounded-contexts#cross-domain]]:
- I-051 Audit trail append-only
- I-052 LGPD logs retenção 5 anos
- I-053 HMAC verify **antes** de parse em webhooks
- I-054 PII nunca em logs
- I-055 ABAC ordem: admin → role → action → tenant → scope
- I-056 Tenant isolation: toda query `WHERE condominium_id = $tenant_id`
- I-057 Providers externos agnósticos via interface
- I-058 Events UUIDv7 ordenável
- I-059 ABAC engine falha startup se matriz vazia

---

## Links

- [[_moc]]
- [[../../01-domain/bounded-contexts#cross-domain]]
- [[../../08-security/BEYOND_CORP]]
- [[../../08-security/lgpd]]
- [[../backlog#transversais-cross-cutting]]
