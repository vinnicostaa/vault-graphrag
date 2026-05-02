---
title: MOC — Backend Requirements por Bounded Context (Go)
type: moc
tags: [moc, master-sindico, backend, go, clean-arch, requirements, feature-req, per-bc]
project: master-sindico
stack: backend
created: 2026-04-24
updated: 2026-04-24
---

# MOC — Backend Requirements por Bounded Context (Go)

Requirements específicos do **backend Go** por bounded context. Complementa [[../requirements]] (requirements transversais), [[../architecture]] (Clean Arch 4 camadas), [[../patterns]] (UoW, Saga, Repository) e [[../security]] (BeyondCorp server-side + ABAC + tenant isolation).

Quebra simetria com `web/requirements/` e `mobile/requirements/` — um MD denso por BC, focado em **o que o módulo Go entrega**, não no que o produto expõe ao usuário.

**Fontes canônicas** (destiladas):
- [[../../../04-requirements/functional/identity]] — spec global Identity (IDN-001..IDN-039)
- [[../../../04-requirements/functional/billing]] — spec global Billing (BIL-001..BIL-055)
- [[../../../04-requirements/functional/institutional]] — spec global Institutional (INS-001..INS-043)
- [[../../../04-requirements/functional/commercial]] — spec global Commercial (COM-001..COM-045)
- [[../../../04-requirements/functional/content]] — spec global Content (CNT-001..CNT-035)
- [[../../../04-requirements/functional/assembly]] — spec global Assembly (ASM-001..ASM-033)
- [[../../../04-requirements/functional/cross-domain]] — spec global Cross-Domain (XD-001..XD-035)
- [[../../../01-domain/aggregates/_moc]] — 30+ agregados canônicos
- [[../../../01-domain/invariants]] — invariantes INV-###
- [[../../../11-audit/audit-log/2026-04-23-findings-backend-kiro]] — findings Sprint 9/10 (A-###, DT-###)
- [[../../../STATE]] — decisões D-### vivas
- [[../architecture]] + [[../patterns]] + [[../security]] + [[../tasks]]

---

## 7 Bounded Contexts × Marco × Status real (código `Development/backend/`)

| Arquivo | BC | Marco | Status real | FR count | Agregados principais |
|---|---|---|---|---|---|
| [[identity]] | identity | **M1** ✅ delivered | Zitadel OIDC + sessions + ABAC + users-mirror | 28 | User, Session, MarketingAgencyToken |
| [[billing]] | billing | **M1** ✅ delivered | Stripe subs + trial + quotas + webhook HMAC + Saga | 32 | Plan, Subscription, FeatureUsage, Refund, WebhookEvent |
| [[institutional]] | institutional | **M1** ⚠️ partial | Timeline immutable + MasterPlan + 11 entities; ⚠️ falta unit_type+fracao_ideal | 38 | Condominium, Unit, Membership, TimelineEntry, MasterPlan, Announcement, Event, Poll, ComplianceDeclaration |
| [[commercial]] | commercial | **M2** ⏳ in-progress | Company + ConnectMe + Proposal + Deal + Reputation motor ausente (D-051) | 30 | Company, ConnectMeRequest, Proposal, ClosedDeal, CompanyEvaluation, Partner, HarassmentReport, SupplierVoteSession |
| [[content]] | content | **M2** ⏳ in-progress | Mux IVideoProvider + CancelUpload saga; ILiveProvider wired futuro | 22 | Video, MarketingAgencyLink, Course, Certificate, Live |
| [[assembly]] | assembly | **M3** ⏳ not-started | Aggregates delineados; Livekit ILiveProvider + EndRoom saga | 26 | Assembly, AgendaItem, Vote, AttendanceRecord, Proxy, ScienceRecord, Minutes, LiveSession |
| [[cross-domain]] | cross-domain | **M1**+ ⚠️ partial | InMemoryPublisher atual (NATS threshold); IAuditLogger incompleto (DT-010) | 20 | DomainEvent, AuditEntry, OutboxEntry, AbacDecision |

**Total**: ~196 FRs backend-específicos + invariantes INV-### herdados + eventos de domínio publicados.

---

## Convenção de IDs backend

- `FR-BE-IDN-001` a `FR-BE-IDN-099` — identity
- `FR-BE-BIL-001` a `FR-BE-BIL-099` — billing
- `FR-BE-INS-001` a `FR-BE-INS-099` — institutional
- `FR-BE-COM-001` a `FR-BE-COM-099` — commercial
- `FR-BE-CNT-001` a `FR-BE-CNT-099` — content
- `FR-BE-ASM-001` a `FR-BE-ASM-099` — assembly
- `FR-BE-XD-001` a `FR-BE-XD-099` — cross-domain

IDs backend mapeiam 1:N para FRs canônicos globais (ex: `FR-BE-IDN-003` pode cobrir IDN-003 + IDN-004 + IDN-005 quando o módulo Go implementa num único handler).

---

## Dependências inter-BC (eventos)

```
identity ──┬──▶ billing     (user.created → auto-start trial)
           ├──▶ institutional (user.role=syndic → habilita condomínio CRUD)
           └──▶ cross-domain  (session events → audit + abac invalidation)

billing ──┬──▶ cross-domain (subscription.changed → ABAC cache invalidate)
          ├──▶ institutional (quota override)
          └──▶ commercial    (plan_tier → quota Connect Me)

institutional ──┬──▶ content   (timeline publish threshold)
                ├──▶ commercial (membership muta visibilidade Connect Me)
                └──▶ assembly   (condominium + units → assembly scope)

commercial ──┬──▶ institutional (deal.signed → timeline entry)
             ├──▶ content       (video_visibility_grants)
             └──▶ cross-domain  (audit + LGPD log)

content ──▶ cross-domain (video.published → reindex + LGPD)

assembly ──┬──▶ institutional (assembly.closed → timeline entry)
           ├──▶ commercial    (supplier vote result)
           └──▶ content       (attachment uploads via IVideoProvider)

cross-domain ◀── TODOS (IAuditLogger, IEventBus, ABAC engine)
```

Comunicação **sem cross-BC import** (XD-003); apenas interfaces em `internal/shared/` + eventos via `IEventBus` (InMemoryPublisher agora, NATS JetStream threshold).

---

## Shared interfaces canônicas (`internal/shared/types`)

Interfaces exposed por BC e consumidas por outros — solução para XD-003 (no cross-BC import) sem degradar Clean Arch:

- **`ITrialChecker`** (billing expõe) — identity consulta pra gate onboarding.
- **`IQuotaConsumer`** (billing expõe) — content/commercial/institutional decrementam quota.
- **`IValidationSubmitter`** (institutional expõe) — commercial submete execution records.
- **`ITimelinePublisher`** (institutional expõe) — commercial/assembly publicam.
- **`IUserModerator`** (identity expõe) — content/commercial aplicam ban/suspension.
- **`ICondominiumIDsProvider`** (institutional expõe) — commercial/assembly resolvem scope.
- **`IAuditLogger`** (cross-domain expõe — DT-010 aberta) — TODOS chamam.
- **`IEventBus`** (cross-domain expõe) — publica domain events.
- **`IPaymentGateway`**, **`IVideoProvider`**, **`ILiveProvider`**, **`IEmailProvider`**, **`ISMSProvider`**, **`IStorageProvider`**, **`ICEPLookup`**, **`ICNPJValidator`**, **`IPushProvider`** — providers externos (XD-012 a XD-020).

---

## Stack tecnológica backend (confirmada código real)

- **Linguagem**: Go 1.23+ (generics Go 1.18+)
- **HTTP adapter**: Gin via `HTTPRouter` abstraction (única impl; troca Echo/Chi isolada)
- **DB**: PostgreSQL 16 + pgx pool + sqlc + goose embedded migrations
- **Cache**: Redis 7 (ABAC cache, quotas, rate limit, onboarding auto-save)
- **Event bus**: InMemoryPublisher (atual) → NATS JetStream threshold (XD-001)
- **Logging**: zerolog estruturado; PII scrubbed (BIL-048)
- **Auth**: Zitadel OIDC + PKCE (ADR-0036 identity-mirror-pattern)
- **Providers**: Stripe, Mux, Livekit (M3+), Resend/SES, Twilio, ViaCEP, FCM
- **Observabilidade**: OpenTelemetry + Sentry (XD-031, XD-032)
- **Testes**: testcontainers (integration), `gopter`/`rapid` (PBT), Go testing (unit)
- **Deploy**: Railway → AWS ECS Fargate futuro (D-024 / XD-033)

---

## Links

- [[../../../_moc]] — MOC backend
- [[../architecture]] — Clean Arch 4 camadas
- [[../patterns]] — UoW, Saga, Repository
- [[../security]] — BeyondCorp server-side
- [[../tasks]] — sprints 1-10 monolítico
- [[../../../CLAUDE]] — contrato agente master-sindico
- [[../../../04-requirements/_moc]] — MOC requirements globais
- [[../../../STATE]] — decisões vivas D-###
- [[../../../11-audit/AUDIT]] — audit trail A-###
- [[../../web/requirements/_moc]] — MOC web (cross-reference)
- [[../../mobile/requirements/_moc]] — MOC mobile (cross-reference)
