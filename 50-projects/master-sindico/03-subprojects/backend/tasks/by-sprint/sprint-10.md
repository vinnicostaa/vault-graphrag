---
title: Sprint 10 — Backend (M1 blockers + DT fechamentos)
type: sprint-spec
tags: [master-sindico, sprint-10, backend, tasks]
project: master-sindico
stack: backend
sprint: 10
period: "2026-04-23 → 2026-05-18"
status: in-progress
milestone_target: m1
created: 2026-04-24
---

# Sprint 10 Backend — M1 blockers + DT fechamentos

**Período**: 2026-04-23 → 2026-05-18 (em curso; sujeito a decisão macro A/B/C do usuário)
**Status**: 🟡 em execução
**Objetivo**: Fechar os bloqueadores do M1 — 14 🔴 findings pentester/SEC (A-DC-PEN/SEC), 7 bloqueadores LGPD humanos, DT-004 (matriz N1/N2/N3 vs canônico 4 tiers), DT-010 (IAuditLogger cross-BC), BE-001 (Unit `unit_type`+`fracao_ideal`), migrations RLS default-on, ata/timeline DB-immutable.

## Escopo desta sprint (backend)

Sprint 10 é o sprint mais crítico do M1. Está aberto a decisão macro do usuário (Opção A postergar 2026-05-18 / B ship 08/05 com known-issues / C remontagem inline). Até decisão, backend executa os fixes técnicos bloqueadores em paralelo ao fechamento humano LGPD.

## Tasks

| ID | Título | Status | Link detalhe | A-### relacionado | D-### |
|---|---|---|---|---|---|
| backend-10.1 | **BE-001** Unit `unit_type` ENUM + `fracao_ideal NUMERIC(5,2)` + entity + use case + PBT | ⏳ | `00211_add_unit_type_and_fracao_ideal.sql` | — | — |
| backend-10.2 | **BE-002 / A-023** 1-device change audit log comparativo (fingerprint diff + `invalidation_reason` TEXT) | ⏳ | `sync_user_use_case.go` | A-023 | — |
| backend-10.3 | **BE-003 / A-039** `RateLimitPerUser(300rpm, burst 30)` após `AuthMiddleware` em `/api/v1/*` | ⏳ | `shared/middleware/rate_limiter.go` | A-039 | — |
| backend-10.4 | **BE-004 / DT-010** `IAuditLogger` interface + DI em billing/commercial/identity + linter AST regressão | ⏳ | `shared/types/audit_logger.go` | A-FASE10-DEV-008 | DT-010 |
| backend-10.5 | **BE-005 / DT-003** Circuit breaker Zitadel introspection (`sony/gobreaker`, threshold 5, timeout 30s) | ⏳ | `zitadelProvider.go` | — | DT-003 |
| backend-10.6 | **BE-006** NATS JetStream + outbox pattern + worker job | ⏳ | `outbox_events` table + `cmd/worker` | — | — |
| backend-10.7 | **BE-007 / A-024** `Context.RawBody() io.Reader` público em `contracts.Context` + remove 3 interfaces privadas | ⏳ | `contracts/context.go` + `gin_adapter.go` | A-024 | — |
| backend-10.8 | **BE-008 / DT-004** `UnitOfWork.Run(ctx)` com `context.WithTimeout(cfg.UoWTimeout)` default 30s | ⏳ | `infrastructure/db/unit_of_work.go` | — | DT-004 |
| backend-10.9 | **DT-004 matriz N1/N2/N3** reescrita canônico 4 tiers `{trial,base,plus,pro}` + Admin | ⏳ | `functional-matrix.md` | A-FASE10-DEV-002 | ADR-0032, D-083 |
| backend-10.10 | **BE-012 / 9.19** Zitadel Railway managed + DNS `auth.mastersindico.com.br` | ⏳ | — | — | — |
| backend-10.11 | **Q-027** Edital 8d CHECK constraint + validator `POST /assemblies` | ⏳ | `00520_assembly_edital_8d_check.sql` | Q-027 | D-088, INV-118 |
| backend-10.12 | **Q-020** Migration ata/timeline DB-immutable (REVOKE + trigger) + role `ms_admin_role` + 4-eyes proc | ⏳ | `20260424_ata_timeline_immutability_db_level.up.sql` | Q-020 | D-088, INV-119/120 |
| backend-10.13 | **Q-024+Q-030** RLS default-on em todas tabelas `tenant_id` + `TenantBoundaryGuard` middleware + CI linter | ⏳ | migrations RLS + `shared/middleware/tenant_boundary_guard.go` | G-034 | D-089, ADR-0034 |
| backend-10.14 | **G-015+016+017** pgBackRest PITR + DR drill mensal + runbook dr-drill.md | ⏳ | Railway `archive_mode=on` + stanza `ms-prod` | — | D-090, ADR-0035 |
| backend-10.15 | **14 🔴 Pentester/SEC**: A-DC-SEC-001 (IDOR device delete), A-DC-SEC-002 (body size Livekit), A-DC-SEC-003 (CORS specific), A-DC-SEC-004 (LGPD delete race), A-DC-PEN-003 (session fixation), A-DC-PEN-010 (mass assignment PATCH /me/role), A-DC-PEN-012 (IDOR PATCH /memberships), A-DC-PEN-016 (webhook idempotency DB UNIQUE), A-DC-PEN-038 (keyed HMAC LGPD) | ⏳ | ADRs 0026-0031 | múltiplos | — |
| backend-10.16 | **BE-011** 2 templates email restantes (welcomes-extensive + billing-alerts-detailed) | ⏳ | `internal/modules/content/infrastructure/providers/resend/templates/` | 9.10 | — |
| backend-10.17 | **BE-014** PBT Trial Expiration migrar para rapid | ⏳ | `check_feature_access_use_case_test.go` | — | — |
| backend-10.18 | **BE-015** Stubs reais SMS (Twilio) + Push (FCM) + PDF (gotpl dossiê) | ⏳ | `infrastructure/providers/{twilio,fcm,pdf}/` | — | — |

## Dependências

- Sprint anterior: gates verdes + BeyondCorp 12/14 ([[sprint-9]]).
- Cross-stack: web ainda praticamente não iniciado (A-FASE10-DEV-005 🔴 bloqueador M1) — impacta Opção A/B/C; mobile também fora do escopo M1 (A-FASE10-DEV-004 🟡).
- Decisões: D-046..D-055 Fase 5; D-088/089/090 Fase 9; ADR-0026..0035.
- Humanos: LGPD-M1-001..007 (DPO + 3 DPAs + DPIA + política + delete race).

## Entregáveis

- AUDIT 🔴 = 0 em artefatos técnicos (condicional à Opção A).
- Unit aggregate com `unit_type` + `fracao_ideal` → AgendaItem pode calcular quórum por fração.
- RLS default-on em todas as tabelas multi-tenant.
- Ata/timeline DB-immutable com override 4-eyes auditável.
- pgBackRest PITR ativo + primeiro DR drill documentado.

## Gates (`<verify>`)

- `go build ./... && go vet ./... && go test -race -count=1 ./...` ✅ permanente
- `gosec -severity=high` ✅ 0
- `govulncheck ./...` ✅ 0 CVEs
- `scripts/rls-check.sh` CI linter ✅
- `scripts/check-tenant-isolation.sh` CI linter ✅
- Coverage global ≥ 85% [[../../../09-testing/coverage-thresholds]]

## Pós-sprint

- **O que deu certo**: (preencher ao fechar)
- **Bloqueadores**: web M1 inviável cronograma (A-FASE10-DEV-005) — decisão stakeholder; LGPD humanos (DPO + 3 DPAs) fora do controle técnico.
- **Dívidas criadas**: dependentes da Opção escolhida.
- **Próxima sprint**: Sprint 11 — hardening pós-M1 + ADRs físicas 0026-0031 (conteúdo completo) + A-FASE10-DEV-006 motor reputação (M2 prep).

## Links

- [[sprint-9]]
- [[../../../../_moc]]
- [[../../../../STATE]]
- [[../../../../AUDIT]]
- [[../../../../ROADMAP]]
- [[../../../../SESSION_CHARTER]]
