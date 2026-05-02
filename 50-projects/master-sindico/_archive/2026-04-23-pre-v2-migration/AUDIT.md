# AUDIT — Master Síndico Backend

Arquivo **vivo** de findings pós-sprint gerados pela skill **`sprint-auditor`**. Complementa `STATE.md` (bloqueadores e dívidas planejadas) com **bugs, más implementações e divergências de spec** descobertas em revisão sistemática ao final de cada sprint.

**Regra dura**: próximo sprint não começa enquanto existir item `🔴 Aberto` ou `🟡 Em andamento` neste arquivo. O agente que roda `task-executor` é obrigado a abrir este arquivo antes de puxar a próxima task de `tasks.md` — se houver items abertos, resolve primeiro.

Formato de cada entrada:
- ID sequencial (`A-###`) por ordem de criação (nunca reutilizar)
- Data ISO de descoberta (+ data de resolução quando fixado)
- Sprint alvo que originou o finding
- Severidade: `critical` (bloqueia deploy), `high` (bug com impacto), `medium` (cheiro forte), `low` (cosmético)
- Arquivo(s) afetado(s) com linha quando aplicável
- Por quê (raiz do problema, não sintoma)
- Fix proposto em uma frase
- Status: `🔴 Aberto` / `🟡 Em andamento` / `✅ Resolvido`

Entradas resolvidas movem para "Histórico" mantendo ID original para auditoria.

Última revisão: 2026-04-22 (rodada pós-reconciliação — labels A-025/A-026/A-019 ajustados para refletir o código; gates verdes: `build ✅`, `vet ✅`, `test -race ✅`, `gosec 0 issues em 314 files ✅`, `govulncheck No vulnerabilities found ✅`)

---

## 🔴 Itens Abertos

### A-023 — 1-device change sem audit log / compare 🔴 Aberto
- **Descoberta**: 2026-04-22 | **Sprint**: 9 | **Severidade**: `medium`
- **Arquivo**: `internal/modules/identity/application/use_cases/sync_user_use_case.go:106`
- **Raiz**: `InvalidatePreviousByUserID` é chamado sempre no sync, mas middleware Authenticate não compara fingerprint atual vs anterior antes de invalidar — perde sinal "device trocou" que seria útil para trail de segurança.
- **Fix proposto**: em `sync_user_use_case.go`, antes de chamar InvalidatePrevious, ler última session ativa (se houver), comparar fingerprints; se diferentes, logar `logger.Warn("device changed", user_id, old_fp, new_fp, ip)` e marcar session anterior com `invalidation_reason="device_changed"` (coluna nova na tabela sessions).
- **Sprint alvo**: Sprint 10 — complementa BeyondCorp sem bloquear marco 1 (08/05).

### A-024 — `rawBodyReader` interface privada acopla handlers ao adapter 🔴 Aberto
- **Descoberta**: 2026-04-22 | **Sprint**: 9 | **Severidade**: `medium`
- **Arquivos**: `internal/modules/billing/infrastructure/http/routes/webhook_handler.go:108`, `internal/modules/content/infrastructure/http/routes/video_handler.go:25-27`, `internal/server/gin_adapter.go`
- **Raiz**: Cada handler de webhook redefine a interface privada `rawBodyReader { RawBody() io.Reader }`. Se `GinContext` adapter for refatorado e remover `RawBody()`, type assertion falha silenciosamente no runtime — não há compile-time check. Duplicação viola DRY.
- **Fix proposto**: Elevar `RawBody() io.Reader` para método público em `internal/core/contracts/Context` interface. Implementação em `GinContext` vira obrigatória. Handlers passam a chamar `ctx.RawBody()` diretamente, sem type assertion. Compile-time check volta.
- **Sprint alvo**: Sprint 10 — refator core, precisa cuidado.

## 🟡 Em Andamento

*Vazio.*

## ✅ Resolvidos nesta Rodada

### A-025 — Assembly Vote TOCTOU (double-voting possível) ✅ Resolvido
- **Descoberta**: 2026-04-22 | **Resolução**: 2026-04-22 | **Sprint**: 9 | **Severidade**: `critical`
- **Arquivo**: `internal/modules/assembly/application/use_cases/vote_use_cases.go:82-100`, `internal/modules/assembly/infrastructure/database/vote_repository_impl.go:27-54`, `internal/shared/providers/database/migrations/00504_create_votes.sql`
- **Raiz**: `voteRepo.HasVoted` → `voteRepo.Create` sem UoW — janela TOCTOU clássica entre check e insert.
- **Fix aplicado**: A defesa é a **UNIQUE constraint no DB** (migration 00504 já contém `UNIQUE(agenda_item_id, voter_id)`, verificado via grep) combinada com tratamento explícito de `isUniqueViolation(err)` em `VoteRepository.Create` que mapeia para `apperrors.ErrConflict.WithMessage("votante já registrou voto neste item")`. Caller recebe 409 determinístico mesmo em race perfeita entre HasVoted e Create — o segundo INSERT falha no DB, não importa quantos requests passaram o check. O HasVoted sobrevive como fast-path para 99% dos casos (retorna 409 imediato sem tentar o INSERT). Pattern idêntico ao CastVote commercial (A-026) e Company Evaluation (A-029).
- **Verificação**: `grep -l "UNIQUE.*agenda_item\|UNIQUE.*voter" migrations/` retorna `00504_create_votes.sql`; `vote_repository_impl.go:48-50` tem o branch de conflict.

### A-026 — Commercial Vote increment fora de tx (race em contador) ✅ Resolvido
- **Descoberta**: 2026-04-22 | **Resolução**: 2026-04-22 | **Sprint**: 9 | **Severidade**: `critical`
- **Arquivo**: `internal/modules/commercial/application/use_cases/vote_session_use_cases.go:102-123`, `internal/modules/commercial/infrastructure/database/supplier_vote_repository_impl.go:101-124`
- **Raiz**: `CreateVoteCast` + `IncrementVotesFor/Against/Abstentions` eram 2 queries sem tx — contador podia divergir do cast sob race.
- **Fix aplicado**: `CastVoteUseCase.Execute` envolve todo o fluxo em `uc.unitOfWork.Run(ctx, func(ctx) { … })`. Dentro da tx, `FindSessionByIDForUpdate` (SELECT … FOR UPDATE — A-030) serializa múltiplos votos na mesma sessão, e `voteRepo.CastVote` herda a tx via `DBFromContext` — CreateVoteCast + Increment* ficam atômicos. Double-vote ainda é protegido pela UNIQUE em `supplier_vote_casts` (código 23505 → `ErrConflict`).
- **Verificação**: `vote_session_use_cases.go:108` tem `uc.unitOfWork.Run`; `supplier_vote_repository_impl.go:102` usa `database.DBFromContext(ctx, r.pool)` que devolve a tx ativa quando em UoW.

### A-027 — UploadVideoUseCase sem Saga (Mux órfão) ✅ Resolvido
- **Descoberta**: 2026-04-22 | **Resolução**: 2026-04-22 | **Sprint**: 9 | **Severidade**: `critical`
- **Arquivo**: `internal/modules/content/application/use_cases/video_use_cases.go:65-90` (commit `c32ab5a`)
- **Raiz**: `CreateUploadURL` → `repo.Create(video)`. Se Create falha, upload Mux fica órfão.
- **Fix aplicado**: Novo método `CancelUpload(ctx, uploadID)` em `IVideoProvider`; Mux usa `CancelDirectUpload`. Use case compensa na falha de Create chamando CancelUpload. Falha da compensação é logada (não sobrescreve o erro original). 2 tests novos cobrindo compensação bem-sucedida + failure path.

### A-028 — StartLiveSessionUseCase sem Saga (Livekit órfão) ✅ Resolvido
- **Descoberta**: 2026-04-22 | **Resolução**: 2026-04-22 | **Sprint**: 9 | **Severidade**: `critical`
- **Arquivo**: `internal/modules/assembly/application/use_cases/live_use_cases.go:62-82` (commit `c32ab5a`)
- **Raiz**: `CreateRoom` → `GenerateToken`. Se GenerateToken falha, room fica órfão no Livekit.
- **Fix aplicado**: Compensação usa `liveProvider.EndRoom` na falha de GenerateToken. Erro original do token prevalece no retorno; falha de compensação fica só em log. 2 tests novos simétricos ao content.

### A-029 — Company Evaluation TOCTOU (duplicate evaluation) ✅ Resolvido
- **Descoberta**: 2026-04-22 | **Resolução**: 2026-04-22 | **Sprint**: 9 | **Severidade**: `high`
- **Arquivo**: `internal/modules/commercial/infrastructure/database/company_evaluation_repository_impl.go:43-47`
- **Raiz**: `evaluationRepo.HasEvaluated()` + `Create()` sem UoW. Síndico pode enviar avaliação duplicada; UNIQUE(company_id, evaluator_user_id, condominium_id) existe em `00308_create_company_evaluations.sql:16` (✅ barreira DB presente) mas o Create retornava `ErrInternal` em unique violation — caller não distinguia.
- **Fix aplicado**: `Create` agora detecta `isUniqueViolation(err)` (helper em `mappers.go`) e retorna `ErrConflict.WithMessage("avaliação já registrada...")`. Use case passa a propagar 409 correto mesmo em race entre HasEvaluated + Create.
- **Próximo hardening**: Sprint 10 — envolver em UoW se quiser garantir que o check + create sejam atômicos também (hoje a barreira final é a UNIQUE, suficiente para o MVP).

### A-030 — Commercial Vote session status race (voto pós-close) ✅ Resolvido
- **Descoberta**: 2026-04-22 | **Resolução**: 2026-04-22 | **Sprint**: 9 | **Severidade**: `high`
- **Arquivo**: `internal/modules/commercial/application/use_cases/vote_session_use_cases.go` + sqlc queries (commit `af6fcc7`)
- **Raiz**: `FindSessionByID` dentro da UoW, mas sem FOR UPDATE. CloseVoteSession concorrente passava.
- **Fix aplicado**: Query nova `GetVoteSessionByIDForUpdate` (`SELECT ... FOR UPDATE`); método `FindSessionByIDForUpdate` no repo; use case usa a variante com lock dentro da UoW. Ordem determinística pelo ID evita deadlock — UNIQUE em `supplier_vote_casts` resolve o caso de double-vote.

### A-031 — Event Response TOCTOU (sem UNIQUE) ✅ Fechado (falso positivo)
- **Descoberta**: 2026-04-22 | **Resolução**: 2026-04-22 | **Sprint**: 9 | **Severidade**: `medium` (reclassificada: nenhuma)
- **Arquivo**: `internal/modules/institutional/application/use_cases/event_use_cases.go:151`, `internal/modules/institutional/infrastructure/database/event_repository_impl.go:101-103`
- **Investigação**: o uso efetivo no use case é `uc.responseRepo.UpsertResponse(ctx, response)`, implementado via `UpsertEventResponse` em sqlc — `INSERT ... ON CONFLICT (event_id, user_id) DO UPDATE`. Não há check + create; é **UPSERT atômico**. A migration `00205_create_events.sql:66` confirma `UNIQUE (event_id, user_id)`. Semanticamente, morador trocando resposta `ciente ↔ confirmado` sobrescreve — comportamento desejado por produto (Req 13).
- **Conclusão**: finding invalidado; não há race para fechar. Nenhuma alteração de código.

### A-032 — Goroutines com lifecycle explícito (cleanup) ✅ Resolvido (parcial — rate limiter)
- **Descoberta**: 2026-04-22 | **Resolução**: 2026-04-22 | **Sprint**: 9 | **Severidade**: `medium`
- **Arquivos**: `internal/shared/middleware/rate_limiter.go`, `internal/server/server.go` (commit `0b83094`)
- **Raiz investigada**:
  - `company_evaluation_use_cases.go`: `sync.WaitGroup` + 2 goroutines herdam o ctx do request — ambas bloqueiam o handler via wg.Wait. Não há leak per-request; as goroutines retornam assim que o ctx é cancelado (repo call aborta). **Finding invalidado** — sem fix necessário.
  - `rate_limiter.go`: goroutine de cleanup sem context cancel — vazava 1 por Server criado (problema em integration/E2E que criam/destroem Servers).
- **Fix aplicado**: `RateLimiter(ctx, rpm, burst)` recebe lifecycleCtx; goroutine usa `select { <-ctx.Done() | <-ticker.C }`. Server guarda `lifecycleCtx + cancel`; Shutdown cancela. Sem leak entre testes.

### A-033 — Join/ListParticipants com retry + reconciliação ✅ Resolvido
- **Descoberta**: 2026-04-22 | **Resolução**: 2026-04-22 | **Sprint**: 9 | **Severidade**: `medium`
- **Arquivo**: `internal/modules/assembly/infrastructure/providers/livekit_provider.go` (commit `76d8e98`)
- **Fix aplicado**: helper `retryLivekit[T]` genérico com backoff 100/300/900ms + respeito a ctx. `ListParticipants` retorna `[]string{}` quando `isNotFound(err)` (condômino chega antes do moderador) — reconciliação implícita. `GenerateToken` intocado (é JWT signing local, sem I/O). 6 tests novos.

### A-034 — EndRoom retry + NotFound terminal ✅ Resolvido
- **Descoberta**: 2026-04-22 | **Resolução**: 2026-04-22 | **Sprint**: 9 | **Severidade**: `medium`
- **Arquivo**: `internal/modules/assembly/infrastructure/providers/livekit_provider.go:158-168` (commit `76d8e98`)
- **Fix aplicado**: `EndRoom` usa o mesmo `retryLivekit` + `isNotFound` como permanent. Se a sala já encerrou (idle timeout de 5min fechou), NotFound é reconciliação implícita — retorna nil. Retry cobre transitória. **Pendente** (Sprint 10+): flag `live_ended_externally_pending` + job noturno para casos em que retry esgota E não-NotFound (falha persistente) — cenário raro, não bloqueante para marco 1.

### A-022 — Mux webhook dentro de `/api/v1` (Authenticate bloqueava Mux) ✅ Resolvido
- **Descoberta**: 2026-04-22 | **Resolução**: 2026-04-22 | **Sprint**: 9 | **Severidade**: `high`
- **Arquivo**: `internal/modules/content/module.go:105` (antes), `cmd/api/main.go:201-205`
- **Raiz**: Rota `POST /api/v1/content/webhooks/mux` estava dentro do grupo `/api/v1`, que aplica `Authenticate` (Zitadel introspection). Mux só envia header `Mux-Signature` HMAC — sem token Zitadel. Authenticate retornaria 401 antes do handler chegar a `verifyMuxSignature()`. Webhooks nunca processariam eventos `video.asset.ready`, vídeos ficariam sempre em "processing".
- **Fix aplicado**: Adicionado `(m *Module) MuxWebhookHandler() contracts.HandlerFunc` em `content/module.go` expondo o handler. Removida rota interna de `Register()`. Em `main.go:205`, registrado via `srv.RegisterPublicWebhook("/webhooks/mux", contentModule.MuxWebhookHandler())` — mesmo padrão do Stripe. Nova URL: `POST /webhooks/mux` (pública, sem Authenticate). Segurança 100% via `verifyMuxSignature()` HMAC-SHA256 já implementado.

### A-019 — Gosec G109/G115 em handlers e repositories (66 findings) ✅ Resolvido
- **Descoberta**: 2026-04-22 | **Resolução**: 2026-04-22 | **Sprint**: 9 | **Severidade**: `high`
- **Arquivos**: 11 handlers + 7 repositories + `shared/providers/database/postgresql.go`
- **Raiz**: `strconv.Atoi(ctx.Query("limit"))` com `_` descartando erro (G109) + cast direto `int32(v)` sem clamp disparando overflow warning (G115).
- **Fix aplicado**: `internal/shared/pagination` (wrapper `FromQuery()` + PBT 1000 casos) e `pkg/safecast` (`Int32`, `Int32FromInt64` com clamp em `[0, MaxInt32]`) cobrem 100% dos sites de overflow int32. Postgresql.go pool config usa `clampToInt32` local. **Gosec -severity=high saiu de 66 → 0 findings** (confirmado em 2026-04-22 após rodada atual: `Files: 314, Issues: 0`). Esforço distribuído entre a sessão anterior (66→20) e a rodada pós-push atual (20→0) via safecast replicado em condominium_repository, poll_repository, video_repository, supplier_vote_repository, management_assessment_repository e compliance_repository.

### A-020 — N+1 UPDATE em MasterPlanTimelineHandler ✅ Resolvido
- **Descoberta**: 2026-04-22 | **Resolução**: 2026-04-22 | **Sprint**: 9 | **Severidade**: `medium`
- **Arquivo**: `internal/modules/institutional/application/event_handlers/master_plan_timeline_handler.go` + repo (commit `bfdd157`)
- **Fix aplicado**: Nova query sqlc `BatchAdvanceMasterPlanStatus` usa UNNEST de 3 arrays paralelos (ids, statuses, entry_ids) num único UPDATE. `SaveBatchAdvance` no repo. Handler coleta activities advanced antes de persistir. Complexidade SQL O(1) em N.

### A-021 — `SELECT *` em queries sqlc commercial ✅ Resolvido
- **Descoberta**: 2026-04-22 | **Resolução**: 2026-04-22 | **Sprint**: 9 | **Severidade**: `low`
- **Arquivos**: queries em `commercial/.../queries/` + generated (commit `a0e9cf6`)
- **Fix aplicado**: 7 queries (supplier_vote + proposal + harassment_report + empresa_sindica) trocaram `SELECT *` por lista explícita. sqlc regenerado mantendo comportamento idêntico; robustez a ADD COLUMN drift sobe. RETURNING * permanece proibido em produção — nenhum dos arquivos afetados usa padrão.

### A-017 — tasks.md Sprint 9 desatualizado (reincidência de A-012) ✅ Resolvido
- **Descoberta**: 2026-04-22 | **Resolução**: 2026-04-22 | **Sprint**: 9 | **Severidade**: `low`
- **Arquivo**: `.kiro/specs/master-sindico/tasks.md` (seção Sprint 9)
- **Raiz**: Sprints 0-8 sofreram A-012 (tasks.md desatualizado) resolvido em 2026-04-21. Sprint 9 implementado ao longo do dia via commits f8ce6d4, 58f171e, 48f842f, 39873f1, 5114325 (wire-up em `content/module.go:42,105` e `assembly/module.go:42,95-97`), mas o tasks.md continuou com `[ ]` em 15 das 19 entradas. Mesmo sintoma do A-012 — disciplina de atualização no ciclo 5-fase não aderiu.
- **Fix aplicado**: Marcados com `[x]` 9.1, 9.2, 9.3, 9.4, 9.6, 9.7, 9.8, 9.9, 9.11, 9.13, 9.14, 9.15, 9.16, 9.17, 9.18. Mantidos `[ ]` apenas 9.10 (templates parciais — 2 de 5) e 9.19 (Zitadel ainda em docker-compose, não Railway). Cada linha referencia o arquivo/commit que evidencia.
- **Ação preventiva**: task-executor tem que atualizar tasks.md **dentro** da fase Ship (antes de commitar). Reforçar no próximo `/sprint-auditor`.

### A-018 — zitadel-key.json fora do .gitignore (risco de commit acidental) ✅ Resolvido
- **Descoberta**: 2026-04-22 | **Resolução**: 2026-04-22 | **Sprint**: 9 | **Severidade**: `high`
- **Arquivo**: `backend/.gitignore`, `backend/zitadel-key.json` (untracked — não commitado ainda)
- **Raiz**: O pattern `[0-9]*[0-9].json` cobre apenas nomes totalmente numéricos (ex: `368701784950062042.json`). Arquivo real fornecido pelo João se chama `zitadel-key.json` — não bateu com o pattern. `git check-ignore zitadel-key.json` retornava exit 1 (não ignorado). Um `git add -A` acidental vazaria a service account key no repo público `mastersindico/backend`.
- **Fix aplicado**: Adicionadas 3 entradas ao `.gitignore` (após a linha `[0-9]*[0-9].json`): `zitadel-key*.json`, `/migrate`, `/zitadel-setup` (binários compilados também untracked). `git check-ignore zitadel-key.json` agora retorna o caminho (✅ protegido).

### A-013 — assembly.PublishEdital sem rota HTTP ✅ Resolvido
- **Descoberta**: 2026-04-21 | **Resolução**: 2026-04-21 | **Sprint**: 7 | **Severidade**: `high`
- **Arquivo**: `internal/modules/assembly/application/use_cases/assembly_use_cases.go`, `internal/modules/assembly/infrastructure/http/routes/assembly_handler.go`, `internal/modules/assembly/module.go`
- **Fix aplicado**: Criados `PublishEditalCommand` + `PublishEditalUseCase` (novo use case dedicado). Adicionado handler `AssemblyHandler.PublishEdital` com decode do campo `edital_pdf_url`. Rota `POST /condominiums/:id/assemblies/:id/publish-edital` registrada no módulo com permissão `assembly.assembly.manage`. `Start()` agora funciona corretamente após chamar `publish-edital`.

### A-014 — harassment-reports FK violation → 500 ✅ Resolvido
- **Descoberta**: 2026-04-21 | **Resolução**: 2026-04-21 | **Sprint**: 4/5 | **Severidade**: `medium`
- **Arquivo**: `internal/modules/commercial/infrastructure/database/harassment_report_repository_impl.go`, `internal/modules/commercial/infrastructure/database/mappers.go`
- **Fix aplicado**: Adicionada função `isForeignKeyViolation(err error) bool` em `mappers.go` (detecção via `pgconn.PgError.Code == "23503"`). Repositório `HarassmentReportRepository.Create` agora captura FK violation e retorna `apperrors.ErrValidation.WithMessage("reported_user_id não encontrado")` → HTTP 422 em vez de 500.

### A-015 — Billing sem wrapper ApiSuccess ✅ Fechado (falso positivo)
- **Descoberta**: 2026-04-21 | **Resolução**: 2026-04-21 | **Sprint**: 1/2 | **Severidade**: `low`
- **Fix aplicado**: Investigação revelou que `contracts.ApiSuccess` não existe em nenhum módulo do projeto. Todos os handlers (institutional, commercial, content, assembly) usam `ctx.JSON(status, map[string]any{...})` — padrão idêntico ao billing. Finding inválido, fechado sem alteração de código.

### A-016 — Event creation zero-time → 500 ✅ Resolvido
- **Descoberta**: 2026-04-21 | **Resolução**: 2026-04-21 | **Sprint**: 3 | **Severidade**: `medium`
- **Arquivo**: `internal/modules/institutional/infrastructure/http/routes/event_handler.go`
- **Fix aplicado**: Adicionado guard `if req.StartsAt.IsZero() { ctx.SetError(apperrors.ErrValidation.WithMessage("starts_at obrigatório")); return }` imediatamente após `DecodeJSON`. Clientes que enviam `event_date` em vez de `starts_at` recebem 422 Unprocessable Entity em vez de 500.

---

### 2026-04-21 — Sprint 8 Security Review — findings registrados

**Itens verificados e aprovados:**
- ✅ Cookies: `HttpOnly=true`, `Secure=true` em não-dev (forçado), `SameSite=Lax`
- ✅ PKCE flow: code_verifier em cookie criptografado (gorilla/securecookie), validado no callback
- ✅ Rate limiting: /auth (20 rpm, burst 5) + /api/v1 (60 rpm, burst 10) separados por IP via token bucket
- ✅ 1 device per account: `device_fingerprint` de `X-Device-Fingerprint` registrado em sessão; `InvalidateOtherSessions` ao criar nova sessão
- ✅ ABAC matrix: admin bypass, role/roleType/planTier enforcement, tenant isolation — 100+ testes verdes
- ✅ Cross-tenant isolation: `RequireTenantMatch` en todas as ações sensíveis, testado sistematicamente
- ✅ Sem PII em logs: nenhum campo email/cpf/password/token logado nos handlers
- ✅ Secret keys não em código: todas em config/env vars (Stripe, Zitadel, DB password)
- ⚠️ A-006: rate limiter cleanup (médio) — Sprint 9
- ⚠️ A-007: audit trail incompleto (médio) — Sprint 9

---

## 📜 Histórico

### A-012 — tasks.md desatualizado ✅ Resolvido
- **Descoberta**: 2026-04-21 | **Resolução**: 2026-04-21 | **Sprint**: cross-sprint | **Severidade**: `low`
- **Fix aplicado**: Marcados com `[x]` 30 tasks backend concluídas: 1.2–1.6 (Identity/Billing), 3.1–3.7 (Institutional II), 4.1–4.7 (Commercial), 5.1–5.3+5.5 (Content), 6.6–6.8 (Integrações parcial), 7.1–7.4 (Assembly). Evidência: README.md + STATE.md confirmam todos os módulos como `✅ Produção`.

### A-007 — Audit trail LGPD incompleto ✅ Resolvido (promovido a DT-010)
- **Descoberta**: 2026-04-21 | **Resolução**: 2026-04-21 | **Sprint**: 8 | **Severidade**: `medium`
- **Fix aplicado**: Promovido a `DT-010` em `STATE.md` — mudança arquitetural cross-módulo (IAuditLogger + adapter + DI wiring em 3 módulos). Sprint 9 com plano detalhado registrado.

### A-006 — RateLimiter: sync.Map cresce sem bound em produção ✅ Resolvido
- **Descoberta**: 2026-04-21 | **Resolução**: 2026-04-21 | **Sprint**: 8 | **Severidade**: `medium`
- **Fix aplicado**: Adicionada goroutine de cleanup dentro de `RateLimiter()` em `internal/shared/middleware/rate_limiter.go`. `time.NewTicker(30 * time.Minute)` percorre a `sync.Map` a cada 30 min e deleta entradas com `lastSeen` anterior a 2h atrás. TODO removido.

### A-011 — commercial/content/assembly application/use_cases: cobertura 0% → ≥90% ✅ Resolvido
- **Descoberta**: 2026-04-21 | **Resolução**: 2026-04-21 | **Sprint**: 4/5/7 | **Severidade**: `high`
- **Fix aplicado**: Criados 11 arquivos de teste para `commercial/application/use_cases/` (stubs + 10 suites), 2 para `content/application/use_cases/`, e expandidos os 8 arquivos de `assembly/application/use_cases/` com casos faltantes. Total: ~220 testes cobrindo happy paths + error paths de todos os use cases. Resultados finais: commercial **92.8%**, content **100%**, assembly **91.5%** (threshold: 90%).

### A-008 — institutional/domain/entities: cobertura 23.9% → 98.7% ✅ Resolvido
- **Descoberta**: 2026-04-21 | **Resolução**: 2026-04-21 | **Sprint**: 3 | **Severidade**: `high`
- **Fix aplicado**: Criados testes para as 9 entidades faltantes (announcement, compliance, event, event_response, management_assessment_response, poll, syndic_mandate, unit, validation_item). Corrigidos nomes de enum incorretos (AnnouncementTypeGeral→InformativoGeral, EventTypeReuniao→ReuniaoMoradores). Adicionados testes de Reconstruct + getters em todos os 13 arquivos de entidade. Coverage final: **98.7%**.

### A-009 — assembly/domain/entities: cobertura 42.5% → 100% ✅ Resolvido
- **Descoberta**: 2026-04-21 | **Resolução**: 2026-04-21 | **Sprint**: 7 | **Severidade**: `high`
- **Fix aplicado**: Criados 4 novos arquivos de teste (attendance_record_test.go, minutes_test.go, proxy_test.go, science_record_test.go). Expandidos assembly_test.go (Reconstruct + branches de estado), agenda_item_test.go (InvalidType, Withdraw_NotPending, SetFractionIdeal edge cases, Reconstruct), vote_test.go (ReconstructVote). Coverage final: **100%**.

### A-010 — institutional/application/use_cases: cobertura 9% → 91.0% ✅ Resolvido
- **Descoberta**: 2026-04-21 | **Resolução**: 2026-04-21 | **Sprint**: 3 | **Severidade**: `high`
- **Fix aplicado**: Criados 16 arquivos de teste com stubs hand-rolled (sem mocktail): `announcement_use_cases_test.go`, `compliance_use_cases_test.go`, `event_use_cases_test.go`, `management_assessment_use_cases_test.go`, `poll_use_cases_test.go`, `validation_use_cases_test.go`, `dashboard_use_case_test.go`, `create_timeline_entry_use_case_test.go`, `create_unit_use_case_test.go`, `end_mandate_use_case_test.go`, `end_membership_use_case_test.go`, `transfer_mandate_use_case_test.go`, `list_master_plan_use_case_test.go`, `list_timeline_use_case_test.go`, `create_master_plan_activity_use_case_test.go`, `use_cases_extra_coverage_test.go`. Stubs compartilhados em `use_cases_common_stubs_test.go`. Total: ~110 testes cobrindo happy paths + failure paths. Coverage final: **91.0%** (threshold: 90%).

---

### 2026-04-21 — Sprint 4 auditado e fechado (commit 104fec7)
- 8 commits analisados (8a1c5e3..2e69b23), 89 arquivos revisados
- Gates finais: build ✅ | vet ✅ | test ✅ | coverage domain/entities 96.1% (threshold 95%) ✅
- gosec não instalado → DT registrada para Sprint 5 (não bloqueia)
- 5 findings abertos em 2026-04-21, todos resolvidos em 2026-04-21 via commit `104fec7`

#### A-001 — `_ = proposalRepo.Save` silencia erro em CloseVoteSession ✅ Resolvido
- **Data abertura**: 2026-04-21 | **Data resolução**: 2026-04-21 | **Sprint**: 4 | **Severidade**: `high`
- **Arquivo**: `internal/modules/commercial/application/use_cases/vote_session_use_cases.go:155`
- **Raiz**: `_ = uc.proposalRepo.Save(ctx, proposal)` — operação falível com erro descartado. Violação da regra `_ = fn()` proibido.
- **Fix aplicado**: Substituído por `if sErr := uc.proposalRepo.Save(...); sErr != nil { uc.log.Warn(...) }` — auto-accept best-effort com erro visível em log.
- **Commit**: `104fec7`

#### A-002 — Cobertura de testes abaixo do threshold em domain/entities ✅ Resolvido
- **Data abertura**: 2026-04-21 | **Data resolução**: 2026-04-21 | **Sprint**: 4 | **Severidade**: `high`
- **Arquivo**: `internal/modules/commercial/domain/entities/` (5 entidades sem testes)
- **Raiz**: Tasks 4.1, 4.3, 4.5, 4.6 não seguiram TDD. Coverage: 35.6% vs threshold 95%.
- **Fix aplicado**: Criados `company_test.go`, `company_user_test.go`, `connect_me_request_test.go`, `execution_record_test.go`, `closed_deal_test.go`. Expandidos `empresa_sindica_link_test.go`, `harassment_report_test.go`, `proposal_test.go`, `supplier_vote_session_test.go`. Coverage final: **96.1%**.
- **Commit**: `104fec7`

#### A-003 — invite_token logado em plain text ✅ Resolvido
- **Data abertura**: 2026-04-21 | **Data resolução**: 2026-04-21 | **Sprint**: 4 | **Severidade**: `medium`
- **Arquivo**: `internal/modules/commercial/application/use_cases/empresa_sindica_use_cases.go:53`
- **Raiz**: `logger.F("invite_token", link.Token())` expõe token OTP nos logs — qualquer acesso aos logs permite aceitar o convite por outra empresa.
- **Fix aplicado**: Campo `invite_token` removido do log; `link_id` suficiente para rastrear.
- **Commit**: `104fec7`

#### A-004 — CNPJ logado em plain text ✅ Resolvido
- **Data abertura**: 2026-04-21 | **Data resolução**: 2026-04-21 | **Sprint**: 4 | **Severidade**: `medium`
- **Arquivo**: `internal/modules/commercial/application/use_cases/company_use_cases.go:114`
- **Raiz**: `logger.F("cnpj", cnpj.Digits())` — LGPD exige mascaramento de identificador de pessoa jurídica.
- **Fix aplicado**: Adicionado `CNPJ.Masked() string` em `value_objects/cnpj.go` retornando `"**.***.***/****-**"`. Substituído `cnpj.Digits()` por `cnpj.Masked()` no log.
- **Commit**: `104fec7`

#### A-005 — `_ = n.Scan(...)` em helper float64ToNumeric ✅ Resolvido
- **Data abertura**: 2026-04-21 | **Data resolução**: 2026-04-21 | **Sprint**: 4 | **Severidade**: `low`
- **Arquivo**: `internal/modules/commercial/infrastructure/database/mappers.go:71`
- **Raiz**: `_ = n.Scan(...)` viola regra explícita do projeto mesmo quando falha é improvável.
- **Fix aplicado**: `if err := n.Scan(...); err != nil { panic(...) }` — panic aceitável para bug de programação (mesma semântica de `regexp.MustCompile`).
- **Commit**: `104fec7`

---

## Como usar

### Para o auditor (`sprint-auditor` skill)

1. Ler `STATE.md` + `tasks.md` do sprint que acabou + commits desde a última rodada (`git log --since="{última-data-histórico}" --oneline`).
2. Aplicar os check-points de `.claude/skills/go-review.md` em ordem de prioridade (dependency rule → error handling → padrões Go → DDD → segurança → antipatterns do legacy).
3. Para cada finding não-trivial: adicionar entrada `🔴 Aberto` com ID sequencial.
4. Se encontrar bug crítico fora de spec: **não** mover para `STATE.md` — AUDIT é fonte de verdade para bugs; STATE é de decisões e dívidas.
5. Ao terminar a rodada: deixar `AUDIT.md` aberto para o `task-executor` do próximo sprint resolver.

### Para o `task-executor` (agente de implementação)

1. Antes de qualquer coisa, abrir este arquivo.
2. Se seção `🔴 Aberto` ou `🟡 Em andamento` não estiver vazia: parar o fluxo da task normal. Tratar cada item como uma micro-task (Discuss → Plan → Execute → Verify por item).
3. Marcar item como `🟡 Em andamento` ao começar, `✅ Resolvido` quando o fix passou nos gates (build + vet + test + coverage).
4. Só quando tudo está `✅`: mover todos os `✅` para Histórico com data de resolução + diff-hash, e puxar a próxima task de `tasks.md`.

### Distinção vs `STATE.md`

| Arquivo | Escopo |
|---|---|
| `STATE.md` | Bloqueadores ativos, decisões em aberto, **dívidas técnicas declaradas conscientemente** (com sprint alvo) |
| `AUDIT.md` | Bugs e má implementação **descobertos em revisão**, sem sprint alvo escolhido — prioridade máxima de resolução |

Uma entrada de `AUDIT.md` pode virar `DT-###` no `STATE.md` se o fix exigir reescrita grande que não cabe no sprint atual — nesse caso, marcar `✅ Resolvido (promovido a DT-###)` e abrir a DT no STATE com link recíproco.
