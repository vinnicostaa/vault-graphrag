---
name: Findings preservados de backend/.kiro/AUDIT.md (Sprint 9/10)
description: >-
  Migração dos A-### que estavam em backend/.kiro/AUDIT.md antes da remoção;
  preserva histórico + 3 findings ainda abertos para Sprint 10
type: audit
date: 2026-04-23T00:00:00.000Z
origin: 'Development/backend/.kiro/AUDIT.md (última modificação 2026-04-22 17:07)'
sprint_source: 9
---

# Findings preservados de `backend/.kiro/AUDIT.md`

Este arquivo preserva os findings A-### gerados pela skill `sprint-auditor` no repositório do backend Go antes da remoção de `Development/backend/.kiro/` (D-092). Ver contexto em [[../../STATE#d-092]] e [[../AUDIT#findings-fase-10]].

**Gates Sprint 9 (22/04)**: `build ✅` · `vet ✅` · `test -race ✅` · `gosec 0 issues (314 files) ✅` · `govulncheck 0 CVEs ✅`.

---

## 🔴 Abertos (Sprint 10)

### A-023 — 1-device change sem audit log / compare 🔴 Aberto
- **Descoberta**: 2026-04-22 | **Sprint**: 9 | **Severidade**: medium
- **Arquivo**: `internal/modules/identity/application/use_cases/sync_user_use_case.go:106`
- **Raiz**: `InvalidatePreviousByUserID` é chamado sempre no sync, mas middleware `Authenticate` não compara fingerprint atual vs anterior antes de invalidar — perde sinal "device trocou".
- **Fix proposto**: antes de chamar `InvalidatePrevious`, ler última session ativa, comparar fingerprints; se diferentes, logar `Warn("device changed", user_id, old_fp, new_fp, ip)` + coluna `invalidation_reason="device_changed"` em `sessions`.

### A-024 — `rawBodyReader` interface privada acopla handlers ao adapter 🔴 Aberto
- **Descoberta**: 2026-04-22 | **Sprint**: 9 | **Severidade**: medium
- **Arquivos**: `internal/modules/billing/infrastructure/http/routes/webhook_handler.go:108`, `internal/modules/content/infrastructure/http/routes/video_handler.go:25-27`, `internal/server/gin_adapter.go`
- **Raiz**: cada handler de webhook redefine interface privada `rawBodyReader { RawBody() io.Reader }`. Sem compile-time check; duplicação viola DRY.
- **Fix proposto**: elevar `RawBody() io.Reader` para método público em `internal/core/contracts/Context`. Implementação em `GinContext` obrigatória. Handlers chamam `ctx.RawBody()` direto.

### A-039 — Rate limiting só por IP, spec exige IP + userId 🔴 Aberto
- **Descoberta**: 2026-04-22 (BeyondCorp audit) | **Sprint**: 9 | **Severidade**: medium
- **Arquivo**: `internal/shared/middleware/rate_limiter.go`
- **Raiz**: `sync.Map[ip]→limiter` apenas. Steering dita 300 req/min por userId em rotas protegidas; 60 req/min por IP em `/auth/*`. IP corporativo NAT compartilha quota.
- **Fix proposto**: segundo tier `userLimiters sync.Map[userID]→limiter` via middleware `RateLimitPerUser(userRpm, userBurst)` após `Authenticate`. Rotas `/api/v1/*` ganham os dois limites; `/auth/*` continua só IP.

---

## ✅ Histórico Sprint 9 (resolvidos)

Preservados para rastreabilidade — todos têm fix committado e verificado.

### A-025 — Assembly Vote TOCTOU
- **Sprint**: 9 · **Severidade**: critical · **Resolução**: 2026-04-22
- UNIQUE(agenda_item_id, voter_id) em `00504_create_votes.sql` + `isUniqueViolation(err)` em `VoteRepository.Create` mapeia para `apperrors.ErrConflict`.

### A-026 — Commercial Vote increment fora de tx (race em contador)
- **Sprint**: 9 · **Severidade**: critical · **Resolução**: 2026-04-22
- `CastVoteUseCase.Execute` envolto em `unitOfWork.Run`; `FindSessionByIDForUpdate` serializa.

### A-027 — UploadVideoUseCase sem Saga (Mux órfão)
- **Sprint**: 9 · **Severidade**: critical · **Resolução**: 2026-04-22 (commit `c32ab5a`)
- Novo `CancelUpload(ctx, uploadID)` em `IVideoProvider`; compensação na falha de Create.

### A-028 — StartLiveSessionUseCase sem Saga (Livekit órfão)
- **Sprint**: 9 · **Severidade**: critical · **Resolução**: 2026-04-22 (commit `c32ab5a`)
- Compensação usa `liveProvider.EndRoom` na falha de `GenerateToken`.

### A-029 — Company Evaluation TOCTOU (duplicate evaluation)
- **Sprint**: 9 · **Severidade**: high · **Resolução**: 2026-04-22
- `isUniqueViolation(err)` + `ErrConflict` → 409 correto em race.

### A-030 — Commercial Vote session status race (voto pós-close)
- **Sprint**: 9 · **Severidade**: high · **Resolução**: 2026-04-22 (commit `af6fcc7`)
- Query `GetVoteSessionByIDForUpdate` + `FindSessionByIDForUpdate`.

### A-031 — Event Response TOCTOU
- **Sprint**: 9 · **Severidade**: medium (reclassificada: nenhuma) · **Resolução**: 2026-04-22
- Falso positivo — `UpsertEventResponse` é `INSERT ... ON CONFLICT DO UPDATE`, atômico.

### A-032 — Goroutines com lifecycle explícito (cleanup)
- **Sprint**: 9 · **Severidade**: medium · **Resolução**: 2026-04-22 (commit `0b83094`)
- `RateLimiter(ctx, rpm, burst)` recebe lifecycleCtx; `Server.Shutdown` cancela. Parcial (restante: company_evaluation — falso positivo).

### A-033 — Join/ListParticipants com retry + reconciliação
- **Sprint**: 9 · **Severidade**: medium · **Resolução**: 2026-04-22 (commit `76d8e98`)
- Helper `retryLivekit[T]` com backoff 100/300/900ms; `isNotFound` retorna `[]string{}`.

### A-034 — EndRoom retry + NotFound terminal
- **Sprint**: 9 · **Severidade**: medium · **Resolução**: 2026-04-22 (commit `76d8e98`)
- Mesmo `retryLivekit`; NotFound é reconciliação implícita (room encerrou por idle timeout).
- **Pendente** (Sprint 10+): flag `live_ended_externally_pending` + job noturno para cenários em que retry esgota E não-NotFound (raro).

### A-022 — Mux webhook dentro de `/api/v1` (Authenticate bloqueava)
- **Sprint**: 9 · **Severidade**: high · **Resolução**: 2026-04-22
- Rota movida para `POST /webhooks/mux` (pública, sem Authenticate); HMAC-SHA256 próprio.

### A-019 — Gosec G109/G115 em handlers e repositories (66 findings)
- **Sprint**: 9 · **Severidade**: high · **Resolução**: 2026-04-22
- `internal/shared/pagination` (PBT 1000 casos) + `pkg/safecast` (`Int32`, `Int32FromInt64`). Gosec `-severity=high` saiu de 66 → 0.

### A-020 — N+1 UPDATE em MasterPlanTimelineHandler
- **Sprint**: 9 · **Severidade**: medium · **Resolução**: 2026-04-22 (commit `bfdd157`)
- `BatchAdvanceMasterPlanStatus` UNNEST de 3 arrays (ids, statuses, entry_ids).

### A-021 — `SELECT *` em queries sqlc commercial
- **Sprint**: 9 · **Severidade**: low · **Resolução**: 2026-04-22 (commit `a0e9cf6`)
- 7 queries trocaram `SELECT *` por lista explícita.

### A-017 — tasks.md Sprint 9 desatualizado
- **Sprint**: 9 · **Severidade**: low · **Resolução**: 2026-04-22
- Marcadas `[x]` 9.1..9.18 exceto 9.10 (email templates 2/5) e 9.19 (Zitadel Railway pendente).

### A-018 — zitadel-key.json fora do .gitignore
- **Sprint**: 9 · **Severidade**: high · **Resolução**: 2026-04-22
- `.gitignore` ganhou `zitadel-key*.json`, `/migrate`, `/zitadel-setup`.

---

## BeyondCorp Audit 12/14 ✅ validados (2026-04-22)

- Never trust frontend · Zero Trust · 1-device per account · Cookie httpOnly+Secure+SameSite · Least privilege credentials · Secrets fora do repo · Middleware stack ordenado · Zitadel JWT Profile · ABAC engine · Webhook signature antes de parse · Device fingerprint · Vulnerability scanning

Pendentes: A-023 + A-039 (listados acima).

---

## Links

- [[../AUDIT]] — AUDIT principal do projeto
- [[../../STATE#d-092]] — decisão de consolidar canônico no vault
- [[2026-04-23-analise-development-vs-vault-sistematica]] — análise sistemática que motivou a migração
- [[../../03-subprojects/backend/README]]
