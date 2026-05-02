---
title: MOC — 03-subprojects/backend (destilação Fase 8, código real)
type: moc
tags: [moc, master-sindico, v2, subprojects, backend, go, destilacao-fase-8]
source: /home/vinni/Documentos/Projetos/Clientes/Joao/MasterSindico/Development/backend
created: 2026-04-23
updated: 2026-04-23
---

# MOC — 03-subprojects/backend

Sub-projeto **`backend/`** do Master Síndico — API Go 1.26, single binary,
Clean Architecture estrita + DDD tático + CQRS + BeyondCorp adaptado.

Destilação **fiel ao código real** em
`/home/vinni/Documentos/Projetos/Clientes/Joao/MasterSindico/Development/backend/`
(snapshot 2026-04-22 após sessão autônoma F1-F33). Divergências com legado
de spec ficam sinalizadas `DIVERGE-DO-LEGADO` em cada arquivo.

**Stack**: Go 1.26 + Gin 1.12 (abstraído) + pgx/v5 + sqlc v2 + goose/v3 + zerolog + Redis 7 +
PostgreSQL 18 + Zitadel OIDC + Stripe + Mux + Livekit + Resend + MinIO +
Twilio (stub) + FCM (stub) + gotpl PDF (stub) + ViaCEP + OTel + Railway.

**Marcos vivos**:
- Marco 1 (MVP contratual): **2026-05-08**.
- Sprint ativo: **9 (Integrations)** — 99% fechado (2 tasks 9.10 + 9.19 abertas).
- Sprint 10 backlog priorizado: **BE-001 Unit gap crítico** + A-023 + A-039 + DT-010.

---

## Arquivos

- [[README]] — visão geral, stack, 6 bounded contexts, 118 endpoints protegidos + 2 webhooks públicos + catálogo de providers inegociáveis (`IPaymentGateway`, `IVideoProvider`, `ILiveProvider`, `IEmailProvider`, `IStorageProvider`, `ISMSProvider`, `IPushProvider`, `IPDFGenerator`, `ICEPLookup`), 39 migrations goose, gaps críticos (Unit sem `unit_type`/`fracao_ideal`), coverage snapshot, gates 2026-04-22.
- [[architecture]] — layers Clean Arch, 18 passos de bootstrap em `main.go`, contratos core (`IModule`, `HTTPRouter`, `Context`, `IRepository[T,ID]`, `IUseCase[I,O]`, `IUnitOfWork`), middleware stack BeyondCorp ordenada, layout canônico de bounded context, patterns aplicados (Saga com compensação, UoW ponteado por ctx, Event Handler reativo, cross-module via `shared/types/`, Repository + sqlc), deploy Railway, observabilidade OTel, gap NATS JetStream.
- [[patterns]] — aggregate com estado privado + NewX + ReconstructX, state machines com guards, value objects imutáveis (CNPJ, CPF, Email, Address), error-as-value com sentinels, Saga checkout/upload/live, UoW via ctx + `DBFromContext`, zero-`else` Go-flavor, domain services declarativos (QuotaLimits, ABAC matrix, TrialRules), repository + sqlc + `isUniqueViolation`, PBT com `pgregory.net/rapid` (4 suites — fração ideal, quotas, ABAC 400 casos, pagination 1000 casos), integration tests com `testcontainers-go`, stubs hand-rolled sem mockery, `pkg/safecast` para G115, pagination canônica, idempotency Stripe/Mux, cleanup goroutines via `context.WithCancel`, antipatterns com regex de auditoria.
- [[requirements]] — Plans universais trial/base/plus/pro (ADR-0032), trial por persona (15/7/30/—), quotas (connect_me anual, video_upload mensal), coverage thresholds, security gates (gosec 0 findings / govulncheck 0 CVEs), rate limit, tenant isolation row-based, PKCE + cookies, webhooks HMAC, observability OTel, contratos externos (OpenAPI 3.0 via swag, webhooks Stripe + Mux), requisitos por módulo Req 1-103.
- [[security]] — 14 invariantes BeyondCorp (12/12 + 2 abertos A-023/A-039), middleware stack rígida, Zitadel introspection + 1-device, ABAC engine custom (789 linhas matrix, 70+ actions, fail-closed, admin bypass), webhooks HMAC antes de parse, rate limit token bucket com cleanup ctx-aware (A-032), secrets em `.gitignore` (A-018), PII never logged (CNPJ.Masked, invite_token fora de log), threat model STRIDE, 15 vulnerabilidades históricas fechadas 2026-04-22.
- [[tasks]] — matriz F1-F33 completa (sessão autônoma 2026-04-22), Sprint 9 com 2 tasks abertas (9.10 templates, 9.19 Zitadel managed), Sprint 10 backlog (BE-001 Unit crítico, BE-002 A-023, BE-003 A-039, BE-004 DT-010, BE-005 DT-003, BE-006 NATS outbox, BE-007..015 hardening), Done vs Pendente por módulo, Definition of Done.

---

## Vizinhos

- [[../../_moc]] — MOC 03-subprojects (backend + web + app + full-stack legado)
- [[../../CLAUDE]] — contrato do agente v2
- [[../../STATE]] — decisões vivas D-###
- [[../../AUDIT]] — findings A-### no v2
- [[../../STEERING]] — não-negociáveis (plans universais + providers inegociáveis + Admin = role privilegiada)
- [[../../ROADMAP]] — Marco 1 / M2 / M3
- [[../../02-architecture/clean-arch-mapping]] — camadas canônicas
- [[../../02-architecture/patterns]] — patterns gerais (aqui especializados Go)
- [[../../01-domain/bounded-contexts]] — mapa de BCs (6 implementados no backend)
- [[../../04-requirements/_moc]] — global + functional + NFR
- [[../../08-security/BEYOND_CORP]] — invariantes BeyondCorp (espelhados em [[security]])

---

## Snapshot de números (2026-04-22)

| Métrica | Valor |
|---|---|
| Módulos (bounded contexts) | **6** |
| Aggregates em produção | **38** (User, Session, Plan, Subscription, FeatureUsage, 14 institutional, 10 commercial, 2 content, 7 assembly + stubs) |
| Value objects | 4 (CPF, CNPJ, Email, Address + BimonthlyPeriod) |
| Enums domínio nomeados | 41 |
| Use cases (arquivos) | ~48 |
| Endpoints REST protegidos `/api/v1/*` | **118** (identity 1 + billing 4 + institutional 36 + commercial 41 + content 11 + assembly 25) |
| Webhooks públicos | 2 (`/webhooks/stripe`, `/webhooks/mux`) |
| Migrations goose | **39** |
| ABAC actions na matriz | 70+ |
| ABAC rules | ~130 |
| Arquivos de teste | 132 (excluindo generated) |
| PBT suites | 4 (agenda_item, feature_usage, abac_engine, pagination) |
| Integration tests | 9 arquivos com tag `integration` |
| Gosec high findings | 0 (de 66 iniciais) |
| govulncheck CVEs | 0 |
| Coverage domain/entities | 96-100% por módulo |
| Coverage application/use_cases | 91-100% por módulo |
| AUDIT findings abertos | 3 (A-023 medium, A-024 medium, A-039 medium) |
| STATE dívidas ativas | 4 (DT-003, DT-004, DT-007, DT-010) |
