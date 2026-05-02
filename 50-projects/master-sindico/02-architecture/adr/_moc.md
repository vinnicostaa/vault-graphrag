---
title: MOC — 02-architecture/adr
type: moc
tags: [moc, master-sindico, v2, architecture, adr]
created: 2026-04-23T00:00:00.000Z
updated: 2026-04-24T00:00:00.000Z
---

# MOC — 02-architecture/adr

Architecture Decision Records (ADR-0001..ADR-0042). Cada ADR formaliza uma decisão. Format: [Michael Nygard template](https://github.com/joelparkerhenderson/architecture-decision-record) — Status / Context / Decision / Consequences / Alternatives / References.

> Herança: 20 decisões pré-existentes do legado vivo + 5 decisões surgidas do research web destilado em `13-research` *(consolidada em [[../../../../60-sources/master-sindico-research/_moc]])* + 6 ADRs novas (0026..0031) derivadas da Fase 5 dual-check/pentest + 1 ADR Fase 7 (0032 planos globais Stripe-style) + **3 ADRs Fase 9** (0033 ata DB-immutable, 0034 RLS default-on, 0035 PITR) derivadas da reanálise research (Q-### + G-###) + **3 ADRs Fase 14** (0039 IAM Cloudflare-style, 0040 Cloudflare edge, 0041 SMS Twilio) + **1 ADR Fase 14 ext** (0042 search provider real M1, **proposed pending dual-check**). Mudanças de decisão criam nova ADR (superseded / deprecated), **updates explícitos** em ADR accepted (ver 0009, 0015) quando o contexto ganha dimensão nova.

## Nota 2026-04-24 — materialização e nomes

Auditoria pediu materializar 9 ADRs físicas (0009 uuidv7-monotonic, 0015 saga-orch, 0026 admin-mfa, 0027 webhook-idem, 0029 typed-tenant-ids, 0034 RLS, 0037 soft-delete, 0040 Cloudflare, 0041 SMS, 0042 search). Resultado: **8 já existiam fisicamente com nomes ligeiramente diferentes** (mesmo número e mesma decisão); apenas **0042 foi criado novo**. Mapeamento abaixo:

| Pedido auditoria | Arquivo físico canônico | Status |
|---|---|---|
| `0009-uuidv7-monotonic.md` | `0009-uuidv7-ulid-pk.md` | accepted (2026-04-23, update PG18 nativo) |
| `0015-saga-orchestration.md` | `0015-uow-intra-saga-inter.md` + `0030-saga-orchestration-default.md` (decisão completa) | accepted |
| `0026-admin-mfa-m1.md` | `0026-admin-mfa-m1.md` (idem) | accepted |
| `0027-webhook-idempotency-db.md` | `0027-webhook-idempotency-db.md` (idem) | accepted |
| `0029-typed-tenant-ids.md` | `0029-tenant-id-typed.md` | accepted |
| `0034-postgres-rls-default-on-tenant-guard.md` | `0034-postgres-rls-default-on-tenant-guard.md` (idem) | accepted |
| `0037-soft-delete-universal-pseudonymize.md` | `0037-soft-delete-universal-pseudonymize.md` (idem) | accepted |
| `0040-cloudflare-edge.md` | `0040-cloudflare-edge-primary-railway-origin.md` | accepted (Fase 14, dual-check feito em 2026-04-23) |
| `0041-sms-twilio.md` | `0041-sms-twilio-prod-noop-dev.md` | accepted (Fase 14, dual-check feito em 2026-04-23) |
| `0042-search-provider.md` | **CRIADA 2026-04-24** | **proposed (pending dual-check — OpenSearch vs MeliSearch)** |

ADRs 0040 e 0041 chegaram ao estado `accepted` via dual-checks da Fase 14 ([[../../11-audit/audit-log/2026-04-23-fase14-iam-cloudflare-sms]]). Auditoria 2026-04-24 inicialmente apontou estes dois como `proposed pending dual-check`; a checagem do log de auditoria mostrou que **dual-check já foi executado em 2026-04-23**, justificando manter `accepted`. ADR-0042, criada hoje, **não tem dual-check** — é a única `proposed` agora.

---

## ADRs por ordem

### Fundamentos arquiteturais

- [[0001-clean-architecture-ddd-cqrs]] — Clean Arch + DDD tático + CQRS
- [[0002-gin-abstracted-router]] — Gin abstraído via `contracts.HTTPRouter`
- [[0015-uow-intra-saga-inter]] — UoW intra-módulo, Saga inter-módulo / provider externo (update 2026-04-23)
- [[0030-saga-orchestration-default]] — Saga orchestration default, choreography edge case

### Persistência

- [[0005-sqlc-typed-queries]] — sqlc para queries tipadas
- [[0006-pgx-v5]] — pgx/v5 como driver PostgreSQL
- [[0007-goose-migrations-partitioned]] — Goose migrations particionadas por módulo
- [[0009-uuidv7-ulid-pk]] — ULID/UUIDv7 como PK time-ordered (update 2026-04-23 — PG18 nativo default em tabelas novas)
- [[0018-opensearch-tsvector-dev]] — PostgreSQL tsvector em dev, OpenSearch em prod **(amenda parcial via [[0042-search-provider]] 2026-04-24)**
- [[0021-multi-tenant-row-based]] — Multi-tenant row-based + PK composto + RLS — reforçado por [[0034-postgres-rls-default-on-tenant-guard]]
- [[0034-postgres-rls-default-on-tenant-guard]] — RLS default-on + TenantBoundaryGuard (Fase 9)
- [[0035-postgres-pitr-wal-archiving]] — PITR via pgBackRest WAL archiving + DR drill mensal (Fase 9)

### Identidade & Segurança

- [[0003-zitadel-oidc-provider]] — Zitadel como IdP OIDC
- [[0012-abac-redis-cache]] — ABAC + Redis cache 5min + invalidação webhook (keys revisadas em 0039)
- [[0014-one-device-policy]] — 1-device policy via device_fingerprint
- [[0016-beyondcorp-security-posture]] — BeyondCorp como security posture fundadora
- [[0026-admin-mfa-m1]] — Admin MFA obrigatório M1 + step-up em endpoints críticos
- [[0027-webhook-idempotency-db]] — Webhook idempotency via DB UNIQUE + Redis cache L1
- [[0028-lgpd-keyed-hmac]] — Pseudonimização LGPD via HMAC keyed com secret rotating
- [[0029-tenant-id-typed]] — Tenant ID type-driven (VOs distintos)
- [[0037-soft-delete-universal-pseudonymize]] — Soft delete universal + pseudonimização LGPD (Fase 12)
- [[0038-validatable-interface-cross-bc]] — IValidatable interface cross-BC (Fase 12, Opção B DDD puro)
- [[0039-iam-cloudflare-style-master-sindico]] — IAM Cloudflare-style (Fase 14, conditional accepted) — Member + Group + Membership + Role + PG + Quota + Plan

### Providers externos

- [[0004-stripe-payment-gateway]] — Stripe via IPaymentGateway
- [[0010-mux-video-provider]] — Mux como video provider
- [[0011-livekit-sfu]] — LiveKit (SFU WebRTC) Cloud → self-host sob gatilho
- [[0022-circuit-breaker-gobreaker]] — Circuit breaker sony/gobreaker em providers
- [[0031-email-provider-resend-m1]] — Email Resend M1-M2, SES M3+
- [[0040-cloudflare-edge-primary-railway-origin]] — Cloudflare edge primário (Fase 14) + Railway permanece origin
- [[0041-sms-twilio-prod-noop-dev]] — SMS Twilio prod / NoopProvider dev (Fase 14)
- [[0042-search-provider]] — **NOVA 2026-04-24** — ISearchProvider real M1 (revoga D-067), OpenSearch vs Meili a fechar — **proposed (pending dual-check)**

### Domínio

- [[0013-timeline-immutable]] — Timeline imutável (INSERT-only) — promovido a DB-level em [[0033-ata-timeline-db-immutability]]
- [[0023-recsys-deterministic-m1]] — Ranking Connect Me determinístico M1
- [[0024-moderation-cascade]] — Moderação cascade evoluindo por volume
- [[0032-global-plans-stripe-style]] — Planos globais Stripe-style (aboliu slugs por persona)
- [[0033-ata-timeline-db-immutability]] — Ata e Timeline imutáveis via REVOKE + trigger DB (Fase 9)

### Mensageria & Eventos

- [[0019-nats-jetstream-threshold]] — NATS JetStream sob critério 3×3/replay/latência

### Infra & Observability

- [[0008-zerolog-structured-logs]] — zerolog para logs estruturados
- [[0017-railway-initial-aws-m4]] — Railway em M1, AWS ECS em M4+
- [[0020-opentelemetry-grafana-sentry]] — OTel + Grafana + Sentry
- [[0025-sli-slo-error-budget]] — SLI/SLO 99.5% + error budget + postmortem

---

## Tabela síntese

| # | Título | Status | Marco |
|---|---|---|---|
| 0001 | Clean Arch + DDD + CQRS | accepted | M1 |
| 0002 | Gin abstraído HTTPRouter | accepted | M1 |
| 0003 | Zitadel OIDC | accepted | M1 |
| 0004 | Stripe IPaymentGateway | accepted | M1 |
| 0005 | sqlc queries tipadas | accepted | M1 |
| 0006 | pgx/v5 | accepted | M1 |
| 0007 | Goose particionado | accepted | M1 |
| 0008 | zerolog | accepted | M1 |
| 0009 | ULID/UUIDv7 PK | accepted | M1 |
| 0010 | Mux video provider | accepted | M1/M2 |
| 0011 | LiveKit SFU | accepted | M3 |
| 0012 | ABAC + Redis cache | accepted | M1 |
| 0013 | Timeline imutável | accepted | M1 |
| 0014 | 1-device policy | accepted | M1 |
| 0015 | UoW intra / Saga inter | accepted | M1 |
| 0016 | BeyondCorp posture | accepted | M1 |
| 0017 | Railway → AWS ECS | accepted | M1 / M4+ |
| 0018 | tsvector dev / OpenSearch prod (amenda parcial via 0042) | accepted (partial-superseded) | M1 |
| 0019 | NATS JetStream threshold | accepted | M1 outbox / M2+ NATS |
| 0020 | OTel + Grafana + Sentry | accepted | M1 |
| 0021 | Multi-tenant row-based | accepted | M1 |
| 0022 | Circuit breaker gobreaker | accepted | M1 |
| 0023 | Recsys determinístico | accepted | M1 |
| 0024 | Moderação cascade | accepted | M1 manual / M2+ cascade |
| 0025 | SLI/SLO 99.5% + error budget | accepted | M1 |
| 0026 | Admin MFA M1 + step-up críticos | accepted | M1 |
| 0027 | Webhook idempotency DB UNIQUE primary | accepted | M1 |
| 0028 | LGPD pseudonimização keyed HMAC | accepted | M1 |
| 0029 | Tenant ID type-driven VOs | accepted | M1 |
| 0030 | Saga orchestration default | accepted | M1 |
| 0031 | Email Resend M1-M2, SES M3+ | accepted | M1 |
| 0032 | Planos globais Stripe-style | accepted | M1 |
| 0033 | Ata/Timeline imutável DB-level (REVOKE+trigger) | accepted | pré-M1 (Fase 9) |
| 0034 | PostgreSQL RLS default-on + TenantBoundaryGuard | accepted | pré-M1 (Fase 9) |
| 0035 | Postgres PITR via pgBackRest + DR drill mensal | accepted | pré-M1 (Fase 9) |
| 0037 | Soft-delete universal + pseudonimização LGPD | accepted | M1 (Fase 12) |
| 0038 | IValidatable interface cross-BC | accepted | M1 (Fase 12) |
| 0039 | IAM Cloudflare-style (Member+Group+Membership+Role+PG+Quota+Plan) | accepted (conditional) | pré-M1 (Fase 14) |
| 0040 | Cloudflare edge primário + Railway origin | accepted | pré-M1 (Fase 14) |
| 0041 | SMS Twilio prod / Noop dev | accepted | M1 (Fase 14) |
| **0042** | **ISearchProvider real M1 (revoga D-067) — OpenSearch vs Meili** | **proposed (pending dual-check)** | **M1** |

---

## Decisões pendentes (⚠️)

- **D-026 (legado)**: ✅ resolvido em [[0031-email-provider-resend-m1]].
- **Receita Federal provider** para validação CNPJ em Connect Me (Serpro / parceiro).
- **RLS por tabela**: lista definitiva de tabelas com RLS on.
- **Citus vs particionamento PG nativo** — M3 ADR futura.
- **OPA/Cedar** se políticas ABAC ficarem complexas — M2+ ADR futura (debt em [[0039-iam-cloudflare-style-master-sindico]] D-IAM-CEL).
- **PIM/JIT elevation** para `platform_admin` — M2+ ADR futura (debt em [[0039-iam-cloudflare-style-master-sindico]] D-IAM-PIM).
- **SCIM provisioning** enterprise/agência — M3+ ADR futura (debt em [[0039-iam-cloudflare-style-master-sindico]] D-IAM-SCIM).
- **Cloudflare Containers GA** — re-avaliar origin Go M3+ (debt em [[0040-cloudflare-edge-primary-railway-origin]] D-INFRA-CONTAINERS).
- **Cloudflare D1** read replica edge-side leituras — M3+ (debt em [[0040-cloudflare-edge-primary-railway-origin]] D-INFRA-D1).
- **WhatsApp Business** via Twilio Conversations API — M2+ se cliente pedir (debt em [[0041-sms-twilio-prod-noop-dev]] D-SMS-WHATSAPP).
- **Service mesh (Linkerd)** — M2+ ADR futura (quando extrair 1º serviço).
- **🔴 [[0042-search-provider]] dual-check pendente** (criada 2026-04-24): decidir OpenSearch vs MeliSearch + confirmar revogação parcial de D-067 + confirmar BR-residency LGPD do provider escolhido. **Bloqueia accepted desta ADR e implementação search M1**. Abrir task em [[../../05-tasks/_moc]] na próxima triage.

---

## Vizinhos

- [[../_moc]]
- [[../c4-context]]
- [[../c4-containers]]
- [[../c4-components]]
- [[../clean-arch-mapping]]
- [[../patterns]]
- [[../research-inspirations]]
- [[../../STATE]]
- [[../../AUDIT]]
