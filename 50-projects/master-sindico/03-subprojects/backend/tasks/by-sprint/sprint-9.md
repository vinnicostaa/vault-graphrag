---
title: Sprint 9 — Backend (Sessão autônoma + reconciliação F1-F33)
type: sprint-spec
tags: [master-sindico, sprint-9, backend, tasks]
project: master-sindico
stack: backend
sprint: 9
period: "2026-04-21 → 2026-04-22 (sessão autônoma 10h+)"
status: completed
milestone_target: m1
created: 2026-04-24
---

# Sprint 9 Backend — Sessão autônoma F1-F33 + BeyondCorp 12/14 + gates verdes

**Período**: 2026-04-21 → 2026-04-22 (sessão autônoma compacta 10h+)
**Status**: ✅ concluído
**Objetivo**: Materializar em código 33 fixes (F1-F33) derivados da auditoria; zerar gosec; validar BeyondCorp 12/14; deixar todos os gates verdes para habilitar a Fase 5 de governança.

## Escopo desta sprint (backend)

Sprint atípico — janela curtíssima, alta densidade, 10h+ de execução autônoma. Fecha 16 findings A-### (A-017, A-018, A-019, A-020, A-021, A-022, A-025, A-026, A-027, A-028, A-029, A-030, A-031, A-032, A-033, A-034). A-023, A-024 e A-039 ficam abertos para Sprint 10. Ver [[../../../../SESSION_CHARTER]] tabela F1-F33.

## Findings resolvidos nesta sprint

Cada A-### abaixo foi fechado no Sprint 9 — listado individualmente para dual-check. Fonte de verdade: [[../../../11-audit/audit-log/2026-04-23-findings-backend-kiro]].

| ID | Título | Severidade | Arquivo/Módulo | Fix aplicado | Commit | Status |
|---|---|---|---|---|---|---|
| [[../../../11-audit/audit-log/2026-04-23-findings-backend-kiro#a-017]] | tasks.md Sprint 9 desatualizado | low | `03-subprojects/backend/tasks/by-sprint/sprint-9.md` | Reconciliação 17/19 + pendências explícitas (9.10 templates, 9.19 Zitadel Railway) | — | ✅ |
| [[../../../11-audit/audit-log/2026-04-23-findings-backend-kiro#a-018]] | `zitadel-key.json` versionado sem gitignore | high | `apps/backend/.gitignore` | Entrada `zitadel-key.json` adicionada + rotação planejada Sprint 10 | — | ✅ |
| [[../../../11-audit/audit-log/2026-04-23-findings-backend-kiro#a-019]] | gosec 66 findings (G109/G115 integer overflow) | high | múltiplos `int→int32/uint64` casts | 66→0 findings; casts com `math.MaxInt32` guards + `#nosec G115` justificados | — | ✅ |
| [[../../../11-audit/audit-log/2026-04-23-findings-backend-kiro#a-020]] | N+1 UPDATE em `BatchAdvance` assembly | high | `assembly/agenda_item_repo.go` | `UPDATE ... FROM VALUES` single-query batch | — | ✅ |
| [[../../../11-audit/audit-log/2026-04-23-findings-backend-kiro#a-021]] | `SELECT *` em commercial_vote_session_repo | medium | `commercial/vote_session_repo.go` | Colunas explícitas + index coverage | — | ✅ |
| [[../../../11-audit/audit-log/2026-04-23-findings-backend-kiro#a-022]] | Mux webhook bypass — `Authenticate` middleware aplicado antes da rota pública | critical | `cmd/api/router.go` + `billing/webhook_handler.go` | Rota webhook movida para stack público com verify-signature; `Authenticate` só em `/api/v1/*` | — | ✅ |
| [[../../../11-audit/audit-log/2026-04-23-findings-backend-kiro#a-025]] | Assembly Vote TOCTOU (double-voting) | critical | `assembly/vote_use_cases` + migration 00504 | `UNIQUE(agenda_item_id, voter_id)` + `isUniqueViolation → ErrConflict` | — | ✅ |
| [[../../../11-audit/audit-log/2026-04-23-findings-backend-kiro#a-026]] | Commercial Vote increment race | critical | `commercial/vote_session_use_cases` | `UoW.Run` + `FindSessionByIDForUpdate` | af6fcc7 | ✅ |
| [[../../../11-audit/audit-log/2026-04-23-findings-backend-kiro#a-027]] | Upload Video Saga sem compensação | high | `content/video_upload_saga.go` | Saga com `compensate()` deletando asset Mux em falha pós-commit | c32ab5a | ✅ |
| [[../../../11-audit/audit-log/2026-04-23-findings-backend-kiro#a-028]] | StartLive Saga sem compensação | high | `content/start_live_saga.go` | Saga com rollback de room Livekit + estado `live_session` | c32ab5a | ✅ |
| [[../../../11-audit/audit-log/2026-04-23-findings-backend-kiro#a-029]] | Company Evaluation TOCTOU | high | `commercial/company_evaluation_use_cases` | `UoW.Run` + `SELECT ... FOR UPDATE` + UNIQUE constraint | — | ✅ |
| [[../../../11-audit/audit-log/2026-04-23-findings-backend-kiro#a-030]] | Commercial Vote session `FOR UPDATE` faltando | high | `commercial/vote_session_repo.go` | `FindSessionByIDForUpdate` com lock pessimista | — | ✅ |
| [[../../../11-audit/audit-log/2026-04-23-findings-backend-kiro#a-031]] | Event Response race (falso positivo → UPSERT atômico) | medium | `institutional/event_response_repo.go` | `INSERT ... ON CONFLICT DO UPDATE` atômico; reclassificado falso positivo | — | ✅ |
| [[../../../11-audit/audit-log/2026-04-23-findings-backend-kiro#a-032]] | Goroutines lifecycle sem cancelamento | high | `shared/worker/*` + `cmd/api/main.go` | `context.Context` propagado + `WaitGroup` + shutdown signal | — | ✅ |
| [[../../../11-audit/audit-log/2026-04-23-findings-backend-kiro#a-033]] | Livekit chamadas sem retry helper | medium | `content/livekit_client.go` | `retryWithBackoff` com jitter exponencial e circuit breaker | — | ✅ |
| [[../../../11-audit/audit-log/2026-04-23-findings-backend-kiro#a-034]] | EndRoom NotFound não-terminal | medium | `content/livekit_client.go` | `isNotFound(err) → return nil` (idempotente terminal) | — | ✅ |

**Total resolvidos**: 16 A-### (A-017, A-018, A-019, A-020, A-021, A-022, A-025, A-026, A-027, A-028, A-029, A-030, A-031, A-032, A-033, A-034).

## Findings abertos para Sprint 10

| ID | Título | Severidade | Arquivo | Fix proposto | Observação |
|---|---|---|---|---|---|
| [[../../../11-audit/audit-log/2026-04-23-findings-backend-kiro#a-023]] | 1-device change sem audit log comparativo | medium | `identity/sync_user_use_case.go:106` | Ler session anterior, comparar fingerprints, logar `device_changed` | BeyondCorp strength |
| [[../../../11-audit/audit-log/2026-04-23-findings-backend-kiro#a-024]] | `rawBodyReader` interface privada duplicada | medium | `billing/webhook_handler.go:108`, `content/video_handler.go:25-27` | Elevar `RawBody()` para `contracts.Context` | Refactor core |
| [[../../../11-audit/audit-log/2026-04-23-findings-backend-kiro#a-039]] | Rate limiting só por IP | medium | `shared/middleware/rate_limiter.go` | Adicionar tier per-`userID` | Spec exige IP+userId |

## BeyondCorp — 12/14 invariantes validados

Invariantes BeyondCorp validados no Sprint 9 (ver [[../../../11-audit/audit-log/2026-04-23-findings-backend-kiro#beyondcorp-audit-1214]]):

- Never trust frontend ✅
- Zero Trust (`Authenticate` em todo `/api/v1/*`) ✅
- 1-device per account ✅
- Cookie `httpOnly` + `Secure` + `SameSite` ✅
- Least privilege credentials ✅
- Secrets fora do repo (A-018) ✅
- Middleware stack ordenado ✅
- Zitadel JWT Profile ✅
- ABAC engine com matriz ✅
- Webhook signature antes de parse ✅
- Device fingerprint `X-Device-Fingerprint` ✅
- Vulnerability scanning (gosec 0 / govulncheck 0) ✅

**Pendentes 2/14**: **1-device change audit** (A-023) e **rate limit per-userID** (A-039) — ambos endereçados no Sprint 10.

## Tasks

| ID | Título | Status | Link detalhe | A-### relacionado | D-### |
|---|---|---|---|---|---|
| backend-9.1 | F1-F5 higiene código (`_ = fn()`, logger contexto, DecodeJSON, else-if-nested, CORS wildcard) | ✅ | SESSION_CHARTER §F1-F5 | A-001..A-008 | — |
| backend-9.2 | F6 PBT fração ideal + F7 primeiro integration test testcontainers | ✅ | `testutil/pg.go` + `agenda_item_pbt_test.go` | — | — |
| backend-9.3 | F10 gosec severity=high zero (de 66 findings) | ✅ | CI workflow | A-019 | — |
| backend-9.4 | F21 fix CRITICAL races A-025 (Assembly Vote) + A-026 (Commercial Vote) | ✅ | commits UoW + UNIQUE | A-025, A-026 | — |
| backend-9.5 | F13 Saga auditoria — A-027 (UploadVideo) + A-028 (StartLiveSession) documentadas + fechadas | ✅ | commit `c32ab5a` | A-027, A-028 | — |
| backend-9.6 | F14 backdoor/secret audit zero; `InsecureSkipVerify` zero | ✅ | — | — | — |
| backend-9.7 | F15 BeyondCorp 12/14 invariantes validadas | ✅ | [[../../../11-audit/audit-log/2026-04-23-findings-backend-kiro#beyondcorp-audit-1214]] | A-023, A-039 abertos | — |
| backend-9.8 | 9.1-9.19 Sprint 9 original (Dockerfile, CI, Mux, Livekit, Resend, MinIO, OTel, Zitadel dev, etc.) | ✅ 17/19 | [[../../tasks#sprint-9-integrations-iniciado-2026-04-22]] | — | — |
| backend-9.9 | 9.10 templates email — 7/9 entregues; 2 pendentes | 🟡 | — | — | BE-011 Sprint 10 |
| backend-9.10 | 9.19 Zitadel Railway managed + DNS | 🔴 aberto | — | — | BE-012 Sprint 10 |
| backend-9.11 | F32 PBT ABAC engine 400 casos rapid (~22ms) | ✅ | `engine_pbt_test.go` | — | — |
| backend-9.12 | F33 2 integration tests (session + announcement) | ✅ | `identity_integration_test.go` + `institutional_integration_test.go` | — | — |
| backend-9.13 | F16 recortes requirements/ + design/ (8/8; 89.7KB extraídos) | ✅ | `03-subprojects/backend/requirements/` | — | — |

## Dependências

- Sprint anterior: security hardening ([[sprint-8]]).
- Cross-stack: reconciliação web/mobile documentada (F25 axios CVE flag; F26 Dio CVE patched; F28 web/AGENTS.md limpa; F29 Flutter .env placeholders).
- Decisões: D-092 (consolidação canônica backend/.kiro → vault).

## Entregáveis

- Todos os gates verdes (build/vet/test-race/gosec/govulncheck) ✅
- AUDIT/STATE/SESSION_CHARTER snapshots consolidados
- Backend pronto para Fase 5 (dual-check + QA + pentester pass) e Fase 6 (migração vault)

## Gates (`<verify>`)

- `go build ./...` ✅
- `go vet ./...` ✅
- `go test -race -count=1 ./...` ✅
- `go build -tags=integration ./...` ✅
- `gosec -severity=high` ✅ 0 findings (de 66)
- `govulncheck ./...` ✅ 0 CVEs

## Pós-sprint

- **O que deu certo**: Sprint compacto de 10h+ materializou backlog de 3-4 sprints cronológicos; sem regressão em nenhum gate; BeyondCorp 12/14 validado.
- **Bloqueadores**: 3 findings restantes (A-023, A-024, A-039) + 2 pendências Sprint 9.19 (Zitadel Railway) + 9.10 (templates email 2/5).
- **Dívidas criadas**: DT-001..DT-010 (ver [[../../../../STATE]]); ADR-0026..0031 reservadas na Fase 5.
- **Próxima sprint**: [[sprint-10|Sprint 10 — M1 blockers]].

> **Nota de honestidade temporal**: a sessão autônoma 22/04 consolidou em código o que a narrativa de sprints 2-8 descreve como fluxo cronológico. Ver [[../../../STATE#d-092]] e [[../../../../SESSION_CHARTER]].

## Links

- [[sprint-8]]
- [[sprint-10]]
- [[../../../../_moc]]
- [[../../../../STATE]]
- [[../../../../SESSION_CHARTER]]
- [[../../../11-audit/audit-log/2026-04-23-findings-backend-kiro]]
