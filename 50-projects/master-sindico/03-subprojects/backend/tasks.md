---
title: Backend — Tasks (destilação Fase 8, pós-F1-F33)
type: note
tags: [tasks, backend, sprint, master-sindico, v2, destilacao-fase-8]
source: /home/vinni/Documentos/Projetos/Clientes/Joao/MasterSindico/Development/backend/.kiro/specs/master-sindico/tasks.md + AUDIT + STATE
created: 2026-04-23
updated: 2026-04-23
---

# Backend — Tasks

Estado real pós-sessão autônoma 2026-04-22 (F1-F33 resolvidas). Este arquivo
serve de **seed de tasks para Sprint 10+** e backlog transversal. Tasks de
Sprints 0-9 estão marcadas com status do `backend/.kiro/specs/master-sindico/tasks.md`.

---

## 1. Fontes consultadas (2026-04-23)

| Path | Uso |
|---|---|
| `backend/.kiro/specs/master-sindico/tasks.md:1-250` | Roadmap monolítico Sprints 0-9 |
| `backend/.kiro/SESSION_CHARTER.md:87-305` | Tabela F1-F33 + métricas finais |
| `backend/.kiro/STATE.md:15-60, 230-333` | Sessão autônoma + dívidas técnicas DT-001..DT-010 |
| `backend/.kiro/AUDIT.md:23-205` | Findings A-017..A-039 (12 abertos para Sprint 10) |

---

## 2. Matriz F1-F33 (sessão autônoma 2026-04-22) — ✅ 33/33

| ID | Título | Status | Evidência |
|---|---|---|---|
| F1 | Corrigir 8 violações de `_ = fn()` em I/O críticas | ✅ | A-001..A-008 fechadas |
| F2 | Fix logger contexto (err.Error → err direto; webhook sem request_id) | ✅ | 6 arquivos + billing/content webhooks |
| F3 | Validação obrigatória de DecodeJSON nos handlers | ✅ | 3 handlers + scan zero ocorrências |
| F4 | Eliminar `else` block + if/else aninhado | ✅ | `register_membership` strategy pattern |
| F5 | CORS wildcard bloqueado no startup em staging/prod | ✅ | `Config.Validate()` rejeita `*` |
| F6 | PBT instalado (rapid) + primeiro fração ideal | ✅ | `agenda_item_pbt_test.go` |
| F7 | Primeiro integration test testcontainers | ✅ | `testutil/pg.go` + condominium_test |
| F8 | Deletar web/.kiro/specs/ stale + CLAUDE.md update | ✅ | README-DEPRECATED |
| F9 | Reconciliar Sprint 0 tasks 0.5 / 0.8 / 0.9 | ✅ | Notas de reconciliação Sprint 9 |
| F10 | Gosec instalado + zerar findings high | ✅ | 66 → **0** via safecast + pagination |
| F11 | N+1 queries audit em ListBy/GetMany | ✅ | A-020 baixo volume Sprint 10 |
| F12 | Race conditions audit | ✅ | A-025/A-026 fechados; A-029..A-034 documentados |
| F13 | UoW vs Saga auditoria | ✅ | A-027/A-028 documentados e fechados |
| F14 | Backdoors / secrets / bypass audit | ✅ | Zero backdoors; zero `InsecureSkipVerify` |
| F15 | BeyondCorp audit (14 invariantes) | ✅ | 12/14 ok; A-023 + A-039 abertos |
| F16 | Criar recortes requirements/ + design/ | ✅ | 8/8 (89.7KB extraídos do monolítico) |
| F17 | Atualizar CLAUDE.md + AGENTS.md (desduplicar) | ✅ | backend/CLAUDE.md 141 → 80 linhas |
| F18 | Atualizar memórias desatualizadas | ✅ | `observabilidade_sprint7` + `autonomous_session_2026_04_21` |
| F19 | MCPs alignment | ✅ | D-029 registrado; `.mcp.json` auditado |
| F20 | Coverage drive integration tests | ✅ | 5 integration tests via agent |
| F21 | Fix CRITICAL races (A-025/A-026) | ✅ | Assembly Vote + Commercial Vote |
| F22 | Validar integration tests compilam | ✅ | `go build -tags=integration` exit 0 |
| F23 | Snapshot STATE + SESSION_CHARTER + AUDIT | ✅ | Charter §9+§10 |
| F24 | Drive gosec ZERO | ✅ | exit 0, 0 findings |
| F25 | Audit web/ qualidade | ✅ | Agent bg; axios 1.15 CVE flagged |
| F26 | Audit app/ qualidade | ✅ | Dio CVE patched; stack mobile OK |
| F27 | Update tasks.md Sprint 9 progresso | ✅ | 9.H1-9.H13 adicionadas |
| F28 | Fix web/AGENTS.md refs stale | ✅ | Fastify/SES/ECS removidos |
| F29 | Setup .env placeholders Flutter | ✅ | `String.fromEnvironment` |
| F30 | Templates email pt-BR | ✅ | 7 templates |
| F31 | PBT quotas Consume/Unlimited/ZeroLimit | ✅ | `feature_usage_pbt_test.go` 300 casos |
| F32 | PBT ABAC engine | ✅ | 400 casos rapid ~22ms |
| F33 | 2 integration tests (session + announcement) | ✅ | Identity 1-device + Institutional |

**Gates finais (snapshot 06:30)**:
- `go build ./...` ✅
- `go vet ./...` ✅
- `go test -race -count=1 ./...` ✅
- `go build -tags=integration ./...` ✅
- `gosec -severity=high` ✅ 0 findings (de 66)
- `govulncheck ./...` ✅ 0 CVEs

---

## 3. Sprint 9 — Integrations (iniciado 2026-04-22)

Extraído de `backend/.kiro/specs/master-sindico/tasks.md` + evidência real.

| ID | Task | Status | Evidência / Observação |
|---|---|---|---|
| 9.1 | Dockerfile multi-stage + railway.toml | ✅ | commit `5114325` |
| 9.2 | GitHub Actions CI | ✅ | `.github/workflows/ci.yml` (deploy staging token pendente) |
| 9.3 | Mux SDK real (`muxinc/mux-go`) | ✅ | `mux_provider.go` — webhook HMAC + upload + thumbnail |
| 9.4 | Livekit SDK real + retry | ✅ | `livekit_provider.go` + `retryLivekit[T]` backoff 100/300/900ms (A-033/A-034) |
| 9.6 | Resend adapter | ✅ | `resend_provider.go` |
| 9.7 | MinIO adapter S3-compatible | ✅ | `minio_provider.go` |
| 9.8 | OTel completo | ✅ | `pkg/telemetry` + `middleware.Tracing` + PgxTracer + Prometheus |
| 9.9 | Zitadel v4 + Traefik + zitadel-login em docker-compose | ✅ | commits 218a7a3, 70dd90f |
| **9.10** | **Templates email pt-BR** | 🟡 parcial | 7 entregues (verificacao-conta, connect-me, boas-vindas, 2× assembly, 2× billing); **faltam 2** (welcomes extensivo + billing-alerts detalhado) |
| 9.11 | OpenAPI /docs (Scalar UI) | ✅ | `server/docs.go` |
| 9.13 | Idempotency key em Stripe Create* | ✅ | `stripe_gateway.go` |
| 9.14 | Mux webhook público (fora de `/api/v1`) | ✅ | A-022 fechado — `POST /webhooks/mux` |
| 9.15 | pgxpool ConnMaxIdleTime env-configurável | ✅ | `DB_CONN_MAX_IDLE_TIME` |
| 9.16 | Saga UploadVideo | ✅ | A-027 `c32ab5a` |
| 9.17 | Saga StartLiveSession | ✅ | A-028 `c32ab5a` |
| 9.18 | `SaveBatchAdvance` MasterPlan | ✅ | A-020 `bfdd157` |
| **9.19** | **Zitadel managed em Railway + DNS custom** | 🔴 aberto | Dev local via docker-compose funciona; prod em Railway + DNS `auth.mastersindico.com.br` pendente |

---

## 4. Sprint 10 — Hardening pós-MVP (proposta)

**Marco 1** (MVP contratual) em 2026-05-08. Sprint 10 começa **depois** — foco
em fechar dívidas e preparar GA.

### 4.1 Críticas (cliente master-sindico)

- [~] **BE-001 — Unit aggregate: adicionar `unit_type` + `fracao_ideal`** 🟢 **foundation entregue 2026-04-27 (Agente 2)** — refactor mecânico restante
  - ✅ **Migration `00211_add_unit_type_and_fracao_ideal.sql`** entregue:
    - `CREATE TYPE unit_type AS ENUM ('residencial','comercial','vaga_garagem','deposito','sala_comercial','loja');`
    - `ALTER TABLE units ADD COLUMN unit_type unit_type NOT NULL DEFAULT 'residencial';`
    - `ALTER TABLE units ADD COLUMN fracao_ideal NUMERIC(10,4) CHECK (fracao_ideal IS NULL OR (fracao_ideal > 0 AND fracao_ideal <= 100));` *(precisão NUMERIC(10,4) per sprint-10 T-UNIT-01 mais recente)*
    - Trigger `units_check_fracao_sum_trg` BEFORE INSERT/UPDATE garante Σ ≤ 100 por condomínio (excluindo row em edição); RAISE EXCEPTION com ERRCODE 23514 mapeável para `422 fraction_sum_exceeded`.
    - Index condicional `idx_units_condominium_unit_type ON units(condominium_id, unit_type) WHERE active`.
  - ✅ **Value Objects** novos em `internal/modules/institutional/domain/value_objects/`:
    - `unit_type.go` + `unit_type_test.go` — enum 6 valores, `ParseUnitType`, `IsHabitable`, `DefaultUnitType=residencial`.
    - `fracao_ideal.go` + `fracao_ideal_test.go` — VO com `math/big.Rat` (zero float drift), parsing exato, faixa 0<x≤100, max 4 casas decimais, métodos `String/Numeric/Equal/Cmp/IsZero/IsWithinCondominiumLimit/SumFracoes/MarshalText/UnmarshalText`.
  - ✅ **Entity `Unit`** ampliada (backward compat preservada — `NewUnit` + `ReconstructUnit` legados continuam funcionando, delegando para defaults):
    - Novos: `NewUnitWithDetails`, `ReconstructUnitFull`, `UnitType()`, `FracaoIdeal()`, `HasFracaoIdeal()`, `ChangeUnitType(t)`, `SetFracaoIdeal(f)`, `Deactivate()`.
    - Erro novo `ErrInvalidUnitType` exposto.
  - ✅ Tests novos: 7 entity + 6 fracao_ideal + 4 unit_type = 17 casos cobrindo round-trip, validações, edge cases. `go build/vet ./...` e `go test ./internal/modules/institutional/domain/...` ✅.
  - ⏳ **Refactor sequencial** (não bloqueia M1, ~1-2h mecânico):
    1. `infrastructure/database/queries/unit.sql` — adicionar `unit_type, fracao_ideal` em SELECT/INSERT.
    2. `sqlc generate` (regenera `infrastructure/database/generated/unit.sql.go`).
    3. `infrastructure/database/unit_repository_impl.go` — usar `ReconstructUnitFull` + parse `pgtype.Numeric` ↔ `vo.FracaoIdeal`.
    4. `application/use_cases/create_unit_use_case.go` — receber `UnitType` + `FracaoIdeal` opcionais (default residencial).
    5. `infrastructure/http/routes/unit_handler.go` — DTO accepting fields + mapping ERRCODE 23514 → 422 fraction_sum_exceeded.
    6. PBT `fracao_ideal_pbt_test.go` (migrar padrão de `agenda_item_pbt_test.go`).
  - 📍 Detalhes: [[../../11-audit/audit-log/2026-04-27-agente-2-systematic-pass]]

- [ ] **BE-002 — A-023 — 1-device change audit log comparativo**
  - Coluna nova em `sessions`: `invalidation_reason` TEXT.
  - Em `SyncUserUseCase`, antes de `InvalidatePreviousByUserID`, ler última sessão ativa; comparar fingerprints; logar `device_changed` + persistir `invalidation_reason`.

- [ ] **BE-003 — A-039 — Rate limit per-userID**
  - `RateLimitPerUser(userRpm=300, userBurst=30)` em `shared/middleware/`.
  - Registrar em server.go **após** `AuthMiddleware` no grupo `/api/v1`.
  - Spec steering: 300 rpm por user, 60 rpm por IP em rotas protegidas.

- [ ] **BE-004 — DT-010 — `IAuditLogger` cross-módulo**
  - Interface em `shared/types/audit_logger.go`: `Append(ctx, AuditLogRequest) error` (best-effort).
  - Adapter `institutional/infrastructure/audit_logger_adapter.go` → `IComplianceRepository.CreateAuditLog`.
  - DI em `main.go` nos 3 módulos: billing (checkout, webhook), commercial (deal.sign, empresa_sindica), identity (membership.register).
  - Testes verificando que `Append` é chamado.

- [ ] **BE-005 — DT-003 — Circuit breaker em Zitadel introspection**
  - Instalar `sony/gobreaker`.
  - Wrap `zitadelProvider.Introspect` com Breaker(`FailureThreshold=5`, `Timeout=30s`).
  - Em open, servir cache stale até TTL 2x + log warning.

### 4.2 Importantes (SRE + qualidade)

- [ ] **BE-006 — NATS JetStream + outbox pattern**
  - Adicionar `github.com/nats-io/nats.go` em go.mod.
  - Tabela `outbox_events(id, event_type, payload JSONB, created_at, published_at)`.
  - Publisher cross-process via NATS JetStream.
  - Worker job lê outbox → NATS → marca `published_at`.
  - Substitui `in_memory_publisher.go` nos dois módulos.

- [ ] **BE-007 — A-024 — `Context.RawBody()` público**
  - Promover `RawBody() io.Reader` ao `contracts.Context` interface.
  - Implementar em `GinContext`.
  - Remover 3 interfaces privadas `rawBodyReader`.

- [ ] **BE-008 — DT-004 — Timeout em `UnitOfWork.Run`**
  - `context.WithTimeout(ctx, cfg.UoWTimeout)` dentro do Run.
  - `DB_UOW_TIMEOUT` env, default 30s.

- [ ] **BE-009 — CI tenant-isolation grep**
  - Script `scripts/check-tenant-isolation.sh` que grep queries sqlc procurando
    `WHERE.*id.*=.*\$[0-9]` sem `condominium_id`. Falha se lista ≠ allowlist conhecida.

- [ ] **BE-010 — A-020 / A-021 — Queries remanescentes**
  - A-020 já fechado via `SaveBatchAdvance`. Confirmar em PR.
  - A-021 já fechado (7 queries SELECT *). Confirmar.

- [ ] **BE-011 — Templates email Sprint 9.10 restantes**
  - Criar template `welcomes-extensive.pt-BR.html` (boas-vindas com guia de próximos passos).
  - Criar template `billing-alerts-detailed.pt-BR.html` (detalhe de falha de cobrança com CTAs).

- [ ] **BE-012 — Sprint 9.19 — Zitadel Railway managed**
  - Criar serviço Zitadel em Railway.
  - DNS `auth.mastersindico.com.br` + SSL automático.
  - Atualizar env vars prod.

### 4.3 SRE / Observabilidade / Scale

- [ ] **BE-013 — Worker jobs dedicated**
  - Binário `cmd/worker` — roda cron + outbox publisher + reconciliação live sessions.
  - Systemd-like graceful shutdown.

- [ ] **BE-014 — PBT Trial Expiration migrar para rapid**
  - `check_feature_access_use_case_test.go` usa `math/rand/v2` — migrar para `pgregory.net/rapid`.
  - Cobrir cenários: trial ativo, expirado + webhook atrasado, downgrade após trial.

- [ ] **BE-015 — Stubs reais: SMS, Push, PDF**
  - Twilio: OTP + transactional.
  - FCM: registration de device token + SendToUser.
  - gotpl: GenerateDossie(condo) PDF conforme Req 16.

---

## 5. Backlog transversal

### 5.1 Compliance / LGPD

- [ ] Hard delete de dados pós-pedido do usuário (Sprint 11+)
- [ ] Export de dados pessoais (direito de portabilidade LGPD)
- [ ] Data retention policies automatizadas (compliance 5 anos + auto-purge para dados expirados)

### 5.2 Frontend / Mobile integration

- [ ] Web SolidJS: Sprints 1-8 de telas (backend já pronto para consumo)
- [ ] Flutter: paridade mobile com web (auth + onboarding + home + timeline)

### 5.3 Performance

- [ ] Cache de planos em Redis TTL 1h (spec billing.md — pendente)
- [ ] Materialized views para dashboard síndico (13 indicadores — se latência >500ms)

### 5.4 Observabilidade

- [ ] Application-layer tracing em use cases críticos (CheckoutSaga, UploadVideo, StartLiveSession, Assembly lifecycle)
- [ ] Grafana dashboards padrão: p95 latency por rota, error rate, webhook lag

---

## 6. Done vs Pendente por módulo

| Módulo | Sprint 0-8 | Sprint 9 | Sprint 10+ |
|---|---|---|---|
| identity | ✅ | — | A-023 (BE-002) |
| billing | ✅ | idempotency keys ✅ | DT-010 (BE-004) |
| institutional | ✅ (14 aggregates) | — | **BE-001 Unit gap crítico** |
| commercial | ✅ (10 aggregates + 4.2/4.4/4.7) | — | — |
| content | ✅ (upload Mux) | Saga A-027 ✅ + webhook público ✅ | SMS/Push stubs (BE-015) |
| assembly | ✅ (7 aggregates + live) | Saga A-028 ✅ + Livekit retry ✅ | — |
| shared/middleware | ✅ ABAC PBT 400 + matrix 100+ | rate limiter A-032 ✅ | A-039 (BE-003) |
| providers | ✅ Mux, Livekit, Stripe, Resend, MinIO | — | DT-003 Zitadel breaker (BE-005) |
| observabilidade | — | ✅ OTel + Prometheus | — |
| deploy | — | ✅ Dockerfile + railway.toml | BE-012 Zitadel managed |
| eventos | ✅ in-memory | — | **BE-006 NATS outbox** |

---

## 7. Definition of Done (recorte backend)

Task pode ser `[x]` quando todos abaixo são verdade:

1. Código no branch com commits semânticos (ex: `feat(billing): idempotency key em CreateCustomer`).
2. Spec (`backend/.kiro/specs/master-sindico/**`) atualizada se mudança de contrato.
3. Testes novos:
   - Domain: 95% do código novo (PBT se invariante numérica/combinatória).
   - Application: 90% (happy path + error paths).
   - Infra DB: integration test se repo tocado.
   - Infra HTTP: handler smoke test.
4. Gates passam localmente:
   - `go build ./...`
   - `go vet ./...`
   - `go test -race -count=1 ./...`
   - `gosec -severity=high -exclude-dir=generated -exclude-dir=cmd/zitadel-setup ./...` → 0 findings
   - `govulncheck ./...` → 0 CVEs
5. Context7 MCP consultado para SDK oficial se mudou integração externa.
6. `AUDIT.md` atualizado se novo finding descoberto.
7. `STATE.md` atualizado se decisão não-trivial tomada.
8. `tasks.md` marcado com `[x]` + evidência (commit hash, caminho de arquivo).
9. Readme do sub-projeto atualizado se feature-level (bug-fix não atualiza README).

---

## 8. Referências

- Tasks oficial: `backend/.kiro/specs/master-sindico/tasks.md`
- Sessão charter: `backend/.kiro/SESSION_CHARTER.md`
- Audit: `backend/.kiro/AUDIT.md`
- State: `backend/.kiro/STATE.md`
- Neste vault: [[README]] · [[architecture]] · [[patterns]] · [[requirements]] · [[security]] · [[_moc]]
- Roadmap v2: [[../../ROADMAP]]
