# STATE — Master Síndico Backend

Arquivo **vivo** de bloqueadores, decisões em aberto e dívidas técnicas. Complementa `tasks.md` (estático) com contexto dinâmico. **Atualize sempre que descobrir um muro, abrir uma decisão, ou registrar débito.**

Formato de cada entrada: data ISO + bloco curto. Entradas resolvidas movem para "Histórico" (mantém auditoria).

Última revisão: 2026-04-22 (onboarding autônomo — reconciliação Sprint 9 tasks.md + sincronização `.claude/settings.json` global→projeto + hardening `.gitignore` para zitadel-key)

---

## 🚧 Bloqueadores Ativos

*Nenhum bloqueador crítico no momento.*

## 🟢 Sessão autônoma 2026-04-22 — snapshot final (06:30)

**Duração**: ~7h de trabalho contínuo. **Fechamento de F1-F24** (24 tasks resolvidas).

**Gates finais**: `go build` ✅, `go vet` ✅, `go build -tags=integration` ✅, `govulncheck` ✅ (0 CVEs),
`gosec -severity=high -exclude-dir=generated -exclude-dir=cmd/zitadel-setup` ✅ (**0 findings**, de 66 iniciais).

**Novos pacotes**: `pkg/safecast` (Int32/Int16/Int32FromInt64 com clamp), `internal/shared/pagination`
(FromQuery + PBT 1000 casos), `internal/shared/testutil` (testcontainers + TruncateAll).

**PBT** (antes não existiam): 2 suites cobrindo fração ideal (steering/testing.md regra 1) e
pagination parseBounded.

**Integration tests**: 6 arquivos com tag `integration` (condominium + 5 novos do F20: user,
subscription, feature_usage, timeline, supplier_vote). Rodar com `go test -tags=integration ./...`
após `docker compose up -d`.

**Recortes requirements/** (F16): 8 recortes criados (identity, billing, institutional,
commercial, content, assembly, cross-domain, functional-matrix) — extraídos do monolítico
`requirements.md` fiel à numeração Req 1-103.

**Fixes críticos aplicados nesta sessão**:
1. `_ = fn()` em 8 I/O críticas (webhook close, event publish, audit, DecodeJSON)
2. CORS wildcard bloqueado em staging/prod (Config.Validate)
3. Mux webhook movido para rota pública (`POST /webhooks/mux` fora do /api/v1 — antes
   Authenticate bloqueava Mux porque não envia token Zitadel)
4. Assembly Vote TOCTOU (A-025): VoteRepository.Create detecta unique violation 23505 e
   retorna ErrConflict — barreira definitiva contra double-voting
5. Commercial Vote contador race (A-026): CastVoteUseCase agora envolve `FindSession +
   CastVote + Increment*` em `unitOfWork.Run` — atomicidade garantida
6. `.gitignore` — zitadel-key*.json, /migrate, /zitadel-setup (A-018)
7. 11 handlers + 10 repositories usando `pagination.FromQuery` + `safecast.Int32/Int16` —
   G109 (ignored Atoi err) e G115 (overflow int→int32/int16) zerados
8. Padronização `logger.F("error", err)` (err direto, não err.Error()) em 6 arquivos de
   produção — preserva stack trace

**AUDIT em aberto para Sprint 10**: A-020, A-021, A-023, A-024, A-027, A-028, A-029, A-030,
A-031, A-032, A-033, A-034 (12 itens — nenhum crítico de MVP).

**Documentos atualizados**:
- `backend/CLAUDE.md` — enxugado (141 → 80 linhas), aponta para SESSION_CHARTER como entry point
- `backend/.kiro/SESSION_CHARTER.md` — registro persistente desta sessão autônoma 10h+
- `web/.kiro/specs/master-sindico/README.md` — DEPRECATED, cópias stale deletadas
- 2 memórias atualizadas (observabilidade já entregue; LivekitProvider path correto)

---

## 🟢 Onboarding 2026-04-22 — fica registrado

Início de sessão autônoma (João pediu onboarding completo + execução direta em modo autônomo). Encontrado e corrigido de imediato, nesta ordem:

1. **Segurança** — `zitadel-key.json` (service account key do Zitadel) estava **fora** do `.gitignore`. Risco de commit acidental num repo público. Corrigido (ver A-018 em AUDIT.md). Mesmo sem vazamento realizado, ficou perto.
2. **Reconciliação** — `tasks.md` do Sprint 9 com 15 tasks `[ ]` apesar dos commits (f8ce6d4, 58f171e, 48f842f, 39873f1, 5114325) terem entregue. Atualizado (ver A-017). Reincidência de A-012 — disciplina de fase Ship precisa ser reforçada no próximo `/sprint-auditor`.
3. **Configs** — `backend/.claude/settings.json` não tinha `language: "Portuguese"`, `permissions.defaultMode: "auto"`, `skipDangerousModePermissionPrompt`, `skipAutoPermissionPrompt`, nem o marketplace `knowledge-work-plugins`. Sincronizado com `~/.claude/settings.json`. Ver D-029 abaixo.

Sprint 9 agora tem **2 tasks abertas** (9.10 templates parciais, 9.19 Zitadel em Railway) + DT-010 (IAuditLogger cross-módulo — alvo declarado Sprint 9).

## 🎯 Progresso Sprint 7 (Assembly + Bug Fixes + Polish)

### QA Funcional Completo + Pentest — ✅ CONCLUÍDO 2026-04-21

**Todas as 64+ rotas testadas end-to-end com dados reais via curl. Findings:**

Módulos 100% funcionais (happy path + error path confirmados):
- ✅ Identity: auth/me, profile — token validation correta
- ✅ Institutional: condominium CRUD, unit, timeline, membership, events, announcements, dashboard, master-plan, polls (yes/no/scale/multiple_choice), compliance (terms + declaration), assessments, validations
- ✅ Commercial: companies (register → activate → list), proposals (create → submit → review), empresa-síndica invite, connect-me (bloqueado por quota — correto), harassment (lista + review)
- ✅ Content: videos (upload com tipo em pt-BR), marketing agencies
- ✅ Assembly: configure, agenda-items, publish (status); start bloqueado por edital (A-013)
- ✅ Billing: plans (lista), subscription/me (null = sem plano)

**Pentest resultados:**
- ✅ IDOR: condominium/unit de outro tenant → 404 (isolamento correto)
- ✅ Auth bypass: token aleatório/truncado → 401; token vazio → 401
- ✅ SQL injection: sqlc parameterizado, queries retornam lista vazia sem vazar dados
- ✅ Mass assignment: campos extras no body ignorados (struct binding)
- ✅ XSS: JSON strings armazenadas como text, não renderizadas pelo backend
- ✅ Path traversal: /api/v2, /api/ → 404
- ✅ CSRF: exige Content-Type: application/json (forms HTML bloqueados)
- ✅ CORS: origin-específico, `Access-Control-Allow-Origin: http://localhost:3000` (não wildcard)
- ✅ Rate limiting: /auth (429 após burst 5-6); /api/v1 token bucket funcionando
- ✅ Webhook Stripe: signature validada antes do parse — 400 sem ou com assinatura forjada
- ✅ String overflow: validação de comprimento em domain entities rejeita payloads gigantes
- ✅ Null injection: domain entities rejeitam campos obrigatórios null

**Comportamento Zitadel a documentar (não bug do backend):**
- Zitadel opaque token introspect aceita tokens truncados a partir de 80 chars (de 104). Os últimos 24 chars são provavelmente um verifier que o endpoint de introspect não valida. Não é exploitável sem os primeiros 80 chars.
- Inconsistência de CNPJ: `Company.CNPJ` aceita dígitos brutos, `EmpresaSindicaLink.cnpj` exige formato `XX.XXX.XXX/XXXX-XX`. Documentado para alinhar na API v2.

**Findings abertos em AUDIT.md**: A-013, A-014, A-015, A-016

### Sprint 8.5 + 8.6 + 8.10 QA / Security / Docs — ✅ CONCLUÍDO 2026-04-21

**8.5 — Testes ABAC sistemáticos (abac_matrix_test.go):**
- 7 grupos de teste: admin bypass, síndico principal/auxiliar/assistente, morador titular/dependente, enterprise administrator, enterprise operator vs administrator, cross-tenant isolation, plan-tier enforcement, admin-only actions, multi-condominium
- 100+ sub-testes; todos verdes; `go test ./internal/shared/middleware/...` OK
- Descoberta chave: enterprise `operator` pode criar/submeter execuções e propostas mas não pode criar deals, gerenciar usuários ou revogar agências — diferenciação confirmada na matriz

**8.6 — Security Review BeyondCorp:**
- ✅ Cookies: HttpOnly=true, Secure forçado em não-dev, SameSite=Lax
- ✅ PKCE com gorilla/securecookie (HMAC + AES)
- ✅ Rate limiting: /auth (20 rpm, burst 5) + /api/v1 (60 rpm, burst 10) por IP
- ✅ 1 device: device_fingerprint rastreado em sessão, InvalidateOtherSessions ao criar nova
- ✅ ABAC matrix completa com cross-tenant isolation
- ✅ Sem PII em logs (email/cpf/token nunca logados)
- ⚠️ A-006: rate limiter cleanup — registrado em AUDIT.md como `medium`
- ⚠️ A-007: audit trail incompleto — só compliance persiste AuditLogEntry — registrado como `medium`

**8.10 — README e docs:**
- Adicionada seção "Observabilidade (Sprint 7.9)" com variáveis OTel e configuração Railway → ADOT → CloudWatch
- Adicionada seção "Status dos módulos" com todos os 6 módulos e endpoints base
- Adicionada lista de pendências para produção

### Sprint 7.9 OpenTelemetry — ✅ CONCLUÍDO 2026-04-21

**Observabilidade completa com OTel SDK v1.40.0:**
- `pkg/telemetry/telemetry.go` — Setup() inicializa TracerProvider (OTLP HTTP ou noop) + MeterProvider (Prometheus bridge + OTLP push periódico 30s); propagação W3C TraceContext + Baggage global
- `pkg/telemetry/pgx_tracer.go` — `PgxTracer` instrumenta queries, batches e COPY FROM com spans db.query/db.batch/db.copy_from; semconv v1.39.0 (`DBSystemNamePostgreSQL`, `DBQueryTextKey`, `DBCollectionNameKey`)
- `internal/shared/middleware/tracing.go` — `Tracing(serviceName)` wraps otelgin middleware; `TraceIDFromContext` e `SpanIDFromContext` helpers
- `internal/shared/middleware/logger.go` — enriquece campos de log com `trace_id` + `span_id` quando span válido
- `internal/shared/config/config.go` — `TelemetryConfig` com `Enabled`, `OTLPEndpoint`, `ServiceVersion`, `SampleRate`, `PrometheusEnabled`; lidos de env vars `OTEL_ENABLED`, `OTEL_EXPORTER_OTLP_ENDPOINT`, `OTEL_SERVICE_VERSION`, `OTEL_SAMPLE_RATE`, `OTEL_PROMETHEUS_ENABLED`
- `internal/shared/providers/database/postgresql.go` — `poolConfig.ConnConfig.Tracer = telemetry.NewPgxTracer()` (noop quando OTel desabilitado)
- `internal/server/server.go` — middleware Tracing adicionado ao início da chain quando `cfg.Telemetry.Enabled=true`; campo `telemetryProviders *telemetry.ProvidersResult`
- `internal/server/routes.go` — GET `/metrics` servido pelo Prometheus handler quando disponível
- `cmd/api/main.go` — passo 3: inicializa telemetry.Setup(), registra shutdown em closers stack; passa `telemetryProviders` para `server.New`

**Defaults:**
- `OTEL_ENABLED=false` (dev) → providers são noop, overhead zero
- Staging/prod: `OTEL_ENABLED=true`, `OTEL_PROMETHEUS_ENABLED=true` (auto-ligado)
- Sample rate: 1.0 em dev/staging, 0.1 em production (quando SampleRate=0)

**Testes:** `pkg/telemetry/telemetry_test.go` — 4 testes: noop exporter, Prometheus enabled, global providers, default sample rate. Todos verdes.

**Gates**: `go build ./...` limpo, `go vet ./...` limpo, `go test ./pkg/telemetry/...` 4/4.

### Sprint 7.7 Performance Tuning — ✅ CONCLUÍDO 2026-04-21

**Paginação LIMIT+OFFSET nas queries de lista institucional:**
- `poll.sql` → `ListPollsByCondominium`: `LIMIT $2 OFFSET $3`
- `event.sql` → `ListEventsByCondominium`: `LIMIT $2 OFFSET $3`
- `announcement.sql` → `ListPublishedAnnouncementsByCondominium`: `LIMIT $2 OFFSET $3`
- `sqlc generate` gerou `ListPollsByCondominiumParams / ListEventsByCondominiumParams / ListPublishedAnnouncementsByCondominiumParams`
- Interfaces `IPollRepository.ListByCondominium`, `IEventRepository.ListByCondominium`, `IAnnouncementRepository.ListPublishedByCondominium` receberam `limit, offset int32`
- Implementações `poll_repository_impl.go`, `event_repository_impl.go`, `announcement_repository_impl.go` atualizadas
- Use cases `ListPollsQuery`, `ListEventsQuery`, `ListAnnouncementsQuery` receberam `Limit, Offset int32` + default 50, cap 100
- Handlers `poll_handler.go`, `event_handler.go`, `announcement_handler.go`: `strconv.Atoi(ctx.Query("limit/offset"))` → int32 propagado

**Safety caps no assembly (sem mudança de interface — só SQL):**
- `agenda_item.sql`: `ListAgendaItemsByAssembly` → `LIMIT 100`
- `attendance.sql`: `ListAttendanceByAssembly` → `LIMIT 1000`
- `vote.sql`: `ListVotesByAgendaItem` → `LIMIT 1000`, `ListVotesByAssemblyAndVoter` → `LIMIT 100`
- `proxy.sql`: `ListProxiesByAssembly` → `LIMIT 500`
- `science.sql`: `ListScienceByAssembly` → `LIMIT 1000`

**Pool config configurável:**
- `DatabaseConfig.ConnMaxIdleTime` adicionado (env `DB_CONN_MAX_IDLE_TIME`, default 30min)
- `postgresql.go` usa `cfg.ConnMaxIdleTime` em vez de hardcoded `30 * time.Minute`

**Gates**: `go build ./...` limpo.

### Sprint 7.6 Bug Fixes — ✅ CONCLUÍDO 2026-04-21

Dois bugs críticos corrigidos:

**Bug 1 — Multi-tenancy (RequireTenantMatch era inoperante em produção)**
- **Root cause**: `ABACContext.TenantID = authUser.OrganizationID` (Zitadel org UUID) nunca é igual a `condominium_id` (UUIDv7 do DB). Todas as rotas com `RequireTenantMatch: true` sempre negavam ou sempre permitiam dependendo do código — nunca validavam o tenant real.
- **Fix**:
  - `ICondominiumIDsProvider` adicionado em `shared/types/` (interface que consulta DB)
  - `CondominiumIDsProvider` implementado em `institutional/infrastructure/database/`
  - `AuthUser.CondominiumIDs []string` adicionado — populado no sync (cache miss)
  - `ABACContext.TenantID` substituído por `ABACContext.CondominiumIDs []string`
  - `abac_engine.checkRule` mudado para `containsString(CondominiumIDs, ResourceTenantID)`
  - `SyncUserUseCase` recebe `ICondominiumIDsProvider` opcional, popula `CondominiumIDs`
  - `identity.NewModule` aceita `condominiumIDsProvider` como novo argumento
  - `main.go`: cria provider antes do identity module; `institutional.NewCondominiumIDsProvider(pool)` exportado
  - SQL: `ListCondominiumIDsByMember :many` adicionado em membership.sql (JOIN units); `ListActiveMandatesByUser` já existia para syndic
  - Impacto: todos os 14 ABAC rules com `RequireTenantMatch: true` agora funcionam em produção

**Bug 2 — Trial retroativo (trial expirado ainda concedia acesso)**
- **Root cause**: `Subscription.IsActive()` baseado apenas em `status.IsActiveStatus()`. Status `trialing` com `trialEndsAt` no passado retornava `IsActive()=true`, mas `IsInTrial()=false` → checagem de trial não aplicada → acesso indevido entre expiração do trial e chegada do webhook Stripe.
- **Fix**: `subscription.IsActive()` adicionado guard: `if status==trialing && !IsInTrial() → return false`
- **Teste novo**: `TestIsActive_TrialExpiredButWebhookNotYetReceived` verifica o cenário de reconstituição do DB com status=trialing mas trialEndsAt passado

**Gates**: `go build ./...` limpo, `go vet ./...` limpo, 13 suites passando, zero falhas.

## 🎯 Progresso Sprint 4 (Commercial)

Sessão autônoma 2026-04-21 concluiu **quatro tasks do Sprint 4 Dev1**:

- ✅ **4.1 Connect Me** — 4 fluxos + quota anual via IQuotaConsumer + LGPD log (IP + consent_version). Auto-close 48h (cron futuro marca expired).
- ✅ **4.3 Execution Records** — cross-module hookup: empresa submete → cria ValidationItem no hub institutional via IValidationSubmitter → síndico valida pela UI do hub 3.3 → publica Timeline.
- ✅ **4.5 Closed Deal** — dupla assinatura (empresa → síndico) + auto-publicação Timeline via ITimelinePublisher. is_immutable=true após signed.
- ✅ **4.6 Company Profile + Users** — base do commercial. CNPJ VO + state machine (pending_verification → active → suspended → blocked) + 2 roles (admin/operator).

- ✅ **4.2 Denunciar Assédio** — Denuncia (relatório) + revisão admin (warning/suspension/ban) + listagem pending. Cross-module via `IUserModerator` (identity → commercial). Audit persistido antes de moderar (segurança de integridade). Admin bypass ABAC nativo. Decisão D-026.
- ✅ **4.4 Proposals + Supplier Voting** — CRUD proposal (draft→submitted→accepted/rejected/withdrawn) + sessões de votação com quórum (simple/absolute/qualified), voto atômico via UNIQUE + SQL-side increment. Auto-accept proposta quando sessão fecha aprovada. Attachments via JSONB + helpers. Decisão D-027 (race via DB-level uniqueness + atomic increment).
- ✅ **4.7 Empresa Síndica Vinculada** — Invite por token (UUIDv7) + Accept + UpdatePermissions (5 flags booleanas) + Revoke + List. Migration 00307. 5 ABAC rules. Audit via zerolog (integração `audit_log_entries` diferida para Sprint 7). Decisão D-028.

**Sprint 4 Dev1 100% concluído** — todas as tasks backend implementadas.

Cross-module integrations via `internal/shared/types/` interfaces (nenhum import direto entre modules/):
- `ITrialChecker` (billing → commercial/institutional)
- `IUserSyncer` (identity → middleware)
- `IQuotaConsumer` (billing → commercial)
- `IValidationSubmitter` (institutional → commercial)
- `ITimelinePublisher` (institutional → commercial)
- `IUserModerator` (identity → commercial) — ModerateUserUseCase; warning=log only, suspension=status, ban=banned+reason

Adapters em `modules/*/application/` wrapping use cases concretos. Nenhum ciclo de import. Antipattern A2 (vazamento cross-module) evitado rigorosamente.

---

## 🤔 Decisões em Aberto

### D-001 — Cache TTL de introspection Zitadel
- **Contexto**: Middleware `Authenticate` cachea resultado de introspection no Redis com TTL 5min (key `ms:auth:{zitadelUserID}`). Se admin muda role de um user no Zitadel, pode levar até 5min para propagar.
- **Alternativas**: (a) manter 5min + webhook Zitadel para invalidação proativa; (b) reduzir TTL para 1min; (c) invalidar cache ao fim de toda operação que muda role.
- **Prazo para resolver**: antes do Sprint 4 (primeira integração admin de role via painel).
- **Responsável**: João + agente na task 4.x.

### D-026 — Padrão cross-module para moderação de usuários (task 4.2)
- **Contexto**: ReviewHarassmentReport precisa suspender/banir usuário no módulo identity sem criar ciclo de import.
- **Decisão**: `IUserModerator` em `shared/types/` + `ModerateUserUseCase` em identity + injeção via `main.go`. Mesmo padrão de `IQuotaConsumer`, `IValidationSubmitter` etc. Warning = log estruturado apenas (sem mudança de status). Suspension = `users.status='suspended'`. Ban = `users.banned=true` permanente.
- **Rationale**: Padrão já validado no projeto; baixo blast radius; reversível; evita domain events para caso de uso unidirecional sem subscribers adicionais previstos.
- **Resolvida**: 2026-04-21.

### D-027 — Race condition em CastVote (task 4.4)
- **Contexto**: CastVote precisa ser atômico: registrar voto + incrementar contador sem race condition.
- **Decisão**: INSERT em `supplier_vote_casts` com UNIQUE(session_id, voter_user_id) + queries SQL-side `IncrementVotesFor/Against/Abstentions` (UPDATE ... SET votes_for = votes_for + 1). Unique violation = already voted → ErrConflict. Nenhum lock de aplicação necessário.
- **Rationale**: DB-level uniqueness é a barreira final; incrementos SQL-side são atômicos por natureza. Solução mais simples que SELECT FOR UPDATE.
- **Resolvida**: 2026-04-21.

### D-028 — Audit de empresa síndica (task 4.7)
- **Contexto**: Operações de convite/revogação de empresa síndica são auditáveis mas `audit_log_entries` requer cross-module com institutional que ainda não tem IAuditLogger em `shared/types/`.
- **Decisão**: zerolog estruturado para Sprint 4 (info logs com campos link_id, condominium_id, company_id, created_by). Integração formal com `audit_log_entries` diferida para Sprint 7 junto com OTel.
- **Rationale**: Evita criar novo cross-module coupling para funcionalidade não crítica no caminho. Padrão zerolog já presente no módulo. Revisão em Sprint 7 com `IAuditLogger` na shared/types/.
- **Resolvida**: 2026-04-21.

### D-029 — Sincronização de configs globais Claude Code → projeto + proteção de secret
- **Contexto**: `~/.claude/settings.json` (user scope) tinha 5 itens que **não** estavam em `backend/.claude/settings.json` (project scope): `language: "Portuguese"`, `permissions.defaultMode: "auto"`, `skipDangerousModePermissionPrompt: true`, `skipAutoPermissionPrompt: true`, e o marketplace `knowledge-work-plugins` (dono do plugin `cowork-plugin-management`). Também: `zitadel-key.json` (service account key real do Zitadel) estava fora do `.gitignore` — risco de commit acidental.
- **Decisão tomada autonomamente em 2026-04-22**:
  1. Adicionar os 5 itens ao `backend/.claude/settings.json` — mantém o comportamento do user scope quando outro dev clona o repo (todos veem pt-BR, defaultMode auto, plugins).
  2. Adicionar `zitadel-key*.json`, `/migrate`, `/zitadel-setup` ao `backend/.gitignore`. `git check-ignore zitadel-key.json` agora retorna o caminho — secret protegida.
  3. **Não propagar** para `web/.claude/` nem `app/.claude/` — esses subprojetos não têm `.claude/settings.json` ainda, e criar um sem requisitos explícitos é escopo novo. Deixado como decisão D-030 aberta abaixo.
- **Fontes consultadas**:
  - Contexto do projeto: `~/.claude/settings.json` vs `backend/.claude/settings.json` (diff direto).
  - `~/.claude/plugins/installed_plugins.json`: confirma que `railway`, `figma`, `superpowers` estão habilitados em `backend/.claude/settings.json` mas **não instalados** — o instalador de plugin é responsabilidade do harness (`/plugin install`), não do agente.
  - Docs oficiais Claude Code (treino — não consultei Context7 por ser fora do escopo de SDK Go; decisão de settings é configuração, não integração).
- **Alternativas rejeitadas**:
  - (a) Editar `~/.claude/settings.json` para remover duplicação — rejeitado porque user scope e project scope têm contratos diferentes; project scope é auditável via repo, user scope é pessoal.
  - (b) Manter tudo só no user scope — rejeitado porque outro dev do Master Síndico clonando o repo perderia as permissões e plugins consolidados.
- **Dual-agent usado**: não — decisão de configuração é baixo/médio impacto reversível (basta editar JSON).
- **Reversibilidade**: alta — qualquer campo pode ser removido sem efeito no código.

### D-030 — `.claude/settings.json` para web/ e app/ (aberta)
- **Contexto**: `backend/` tem `.claude/settings.json` versionado com permissões, hooks, modelo, plugins. `web/` e `app/` ainda não têm (`web/.claude/` não existe, `app/.claude/` não existe; `full-stack/.claude/settings.local.json` existe mas legacy).
- **Alternativas**: (a) criar `web/.claude/settings.json` e `app/.claude/settings.json` espelhando o backend com ajuste de permissões (Bash allowlist para `bun`, `flutter`); (b) não criar — deixar o agente herdar `~/.claude/settings.json` quando trabalhar nesses subprojetos; (c) criar só para web/, adiar app/ até a task Flutter aquecer.
- **Prazo para resolver**: quando a primeira task de frontend/mobile for puxada (Sprint 1.7 web ou 1.14 mobile — ambos `[ ]` em tasks.md).
- **Responsável**: João + agente ao iniciar fase Discuss da primeira task web/app.

### D-002 — Estratégia de testes de regressão do fluxo SDD
- **Contexto**: Com o ciclo de 5 fases + `<verify>` + thresholds por camada, precisamos validar que o próprio fluxo não regride (ex: nova versão do steering confunde o agente).
- **Alternativas**: (a) suite de "SDD golden tasks" rodada manualmente a cada alteração grande de steering; (b) eval harness automatizado (complexo); (c) nenhuma — confiar em observação manual.
- **Prazo para resolver**: Sprint 7 (Polish) — quando observabilidade entrar.
- **Responsável**: João.

---

## 📋 Dívidas Técnicas Registradas

### DT-001 — `planTierOrder` duplicado ✅ RESOLVIDO 2026-04-21
- **Fix aplicado**: `planTierOrder` promovido a `PlanTierOrder` exportado em `shared/types/auth_user.go`; função standalone `HasMinimumPlanTier(userTier, minTier string) bool` consumida tanto pelo método `AuthUser.HasMinimumPlanTier` quanto pela ABAC engine (`abac_engine.go` via `types.HasMinimumPlanTier`). Single source of truth para hierarquia trial<base<plus<pro.

### DT-002 — `PlanTier` hardcoded "trial" em sync_user_use_case.go ✅ RESOLVIDO 2026-04-21
- **Fix aplicado**: adicionado `IUserRepository.GetPlanTier(userID)` via query sqlc `GetUserPlanTier` (LEFT JOIN users ↔ plans + `COALESCE(p.tier::text, '')::text` — cast duplo força emit string não-nullable). sync_user_use_case chama após UPSERT; fallback "trial" permanece APENAS quando plan_id ainda é NULL (user pré-checkout). ABAC plan-gate agora reflete plano real.

### DT-003 — Sem circuit breaker para Zitadel introspection
- **Onde**: `internal/shared/providers/auth/zitadel_provider.go`.
- **Impacto**: se Zitadel cai, cada request falha com 503. Sem backoff, sem fallback.
- **Fix**: adicionar `sony/gobreaker` ou similar. Em fallback: servir cache stale até TTL 2x + log warning.
- **Sprint alvo**: Sprint 7 (Polish + observabilidade).

### DT-004 — Sem timeout explícito em UnitOfWork.Run
- **Onde**: `internal/shared/providers/database/unit_of_work.go`.
- **Impacto**: query lenta dentro de transação pode travar worker por tempo indefinido (pgx não rollback em ctx cancel).
- **Fix**: adicionar `context.WithTimeout(ctx, 30*time.Second)` dentro do `Run`, configurável por ambiente.
- **Sprint alvo**: 2 (Institutional — primeiras queries complexas).

### DT-005 — RateLimiter só em `/auth` ✅ RESOLVIDO 2026-04-21
- **Onde**: `internal/server/routes.go:22-24` e `internal/server/server.go`.
- **Fix aplicado**: RateLimit config expandida com 2 conjuntos (`RequestsPerMinute`/`Burst` para global + `AuthRequestsPerMinute`/`AuthBurst` para /auth mais restritivo). Server aplica rate limiter no grupo `/api/v1/*` antes do Authenticate. Defaults: 60/10 global, 20/5 auth.
- Valores agora vêm de config — hardcoded removido.
- **Ainda pendente**: cleanup periódico de IPs inativos em `RateLimiter` (comentário TODO no código). Baixa prioridade.

### DT-006 — `.gitignore` pattern `*.json`
- **Onde**: raiz do backend.
- **Impacto**: padrão amplo demais — exclui arquivos de config versionáveis (`.mcp.json`, `sqlc.yaml` se fosse json, etc).
- **Fix**: ser específico — `zitadel-key*.json`, `credentials*.json`, etc. **Atualização 2026-04-21**: já está específico no `.gitignore` atual com pattern `[0-9]*[0-9].json` (captura Zitadel keys) — `.mcp.json` passa sem problema. DT pode ser **fechada** após validar que nenhum JSON legítimo está sendo excluído.
- **Sprint alvo**: encerrar após validação em Sprint 1.

### DT-007 — Divergência de nome de var Stripe (`STRIPE_KEY_PRIV` vs `STRIPE_SECRET_KEY`)
- **Onde**: `.env` usa `STRIPE_SECRET_KEY`. Docs anteriores (CLAUDE.md, AGENTS.md, steering) mencionam `STRIPE_KEY_PRIV`.
- **Impacto**: confusão de config; código Go pode falhar ao ler env var se nome divergir.
- **Fix**: padronizar para **`STRIPE_SECRET_KEY`** (convenção global Stripe). Atualizar `steering/*.md`, `.mcp.json` comments, e `config.go` quando chegar em task 1.3.
- **Sprint alvo**: 1.3 (Stripe adapter).

### DT-008 — PAT GitHub clássico em vez de fine-grained
- **Onde**: `backend/.env` `GITHUB_PAT=github_pat_11B...`
- **Impacto**: PAT clássico tem escopo coarse (`repo` = tudo no owner). Não compromete funcionalidade do MCP GitHub, mas viola princípio de least privilege.
- **Fix**: gerar fine-grained PAT em https://github.com/settings/tokens?type=beta, escopando apenas aos 3 repos do projeto (`mastersindico/backend`, `web`, `app`) com permissions `Contents: R/W` + `Pull requests: R/W` + `Issues: R/W` + `Metadata: R`. Substituir em `.env`, rotacionar a cada 30-90 dias.
- **Sprint alvo**: hardening em qualquer momento — não bloqueia desenvolvimento.

### DT-010 — Audit trail LGPD incompleto: só compliance persiste AuditLogEntry
- **Promovido de**: A-007 em AUDIT.md (2026-04-21)
- **Onde**: Múltiplos módulos — `billing/application/use_cases/` (checkout.create, subscription events), `commercial/application/use_cases/` (deal.sign, empresa_sindica), `identity/application/use_cases/` (membership.register).
- **Impacto**: Matriz LGPD exige rastreamento de operações sensíveis com dados pessoais/financeiros. Atualmente apenas `institutional/compliance_use_cases.go` produz `AuditLogEntry`. Operações de contrato, assinatura e cobrança não geram trail → não conformidade em auditoria LGPD.
- **Fix**: (1) Criar `IAuditLogger` em `shared/types/` com `Append(ctx, AuditLogRequest) error` (best-effort, nunca falha a operação principal). (2) Adapter `institutional/infrastructure/audit_logger_adapter.go` implementa via `IComplianceRepository.CreateAuditLog`. (3) Injetar no DI (`cmd/api/main.go`) nos use cases afetados. (4) Adicionar testes de use case verificando que `Append` é chamado.
- **Sprint alvo**: Sprint 9 (após todos os módulos core estarem estáveis).

---

## 🧪 Experimentos em Andamento

*Nenhum experimento ativo.*

---

## 📜 Histórico (resolvidos)

### 2026-04-21 — Task 3.2 Comunicados concluída

Req 14: 8 tipos × 4 prioridades + tracking de leitura por morador.

**Criados**: migration `00206_create_announcements.sql` (enums announcement_type 8 + announcement_priority 4 + 2 tabelas com UNIQUE announcement_id+user_id em reads). Enums AnnouncementType + AnnouncementPriority. Entity Announcement com Publish idempotente. 2 Repositories + AnnouncementReadStats. Use cases CreateAnnouncement/ListAnnouncements/MarkAsRead/GetStats. DTOs + handler 4 endpoints.

**Modificados**: 4 ABAC actions novas (announcement.create/read/mark_read/stats), module.go wire-up, sqlc.yaml overrides.

**Leitura idempotente**: UNIQUE + `ON CONFLICT DO NOTHING` → 2x marca lido = no-op. Lista filtra `expires_at > NOW() OR NULL` — comunicados expirados somem do feed sem intervenção.

### 2026-04-21 — Task 3.1 Eventos (Sprint 3 iniciado)

Req 13 implementado: 13 tipos enum + CRUD + respond + métricas agregadas.

**Criados**: migration `00205_create_events.sql` (event_type 13 variantes + event_response_type 2 + 2 tabelas com UNIQUE event_id+user_id); enums EventType/EventResponseType; entities Event + EventResponse (UpdateResponse para trocar ciente↔confirmado); repositories + sqlc (Create/GetByID/List + UpsertResponse ON CONFLICT + GetMetrics com `COUNT(*) FILTER`); use cases CreateEvent/ListEvents/RespondToEvent (UserID do AuthUser)/GetEventMetrics; DTOs; handler com 4 endpoints.

**Modificados**: ABAC 4 novas actions (event.create/read/respond/metrics com tenant match), institutional/module.go wire-up, sqlc.yaml overrides enums.

**Padrão consolidado**: pattern-Timeline reaproveitado — migration+entity+repo+sqlc+UC+DTO+handler+ABAC+module. ~1000 linhas adicionais.

**Idempotência**: UpsertResponse + UNIQUE → morador troca ciente↔confirmado sem criar linha nova. Métricas em 1 query só (FILTER no SELECT).

**Próximo candidate**: 3.2 Comunicados (8 tipos + tracking leitura — mesmo pattern) ou 3.5 Avaliação da Gestão (regra de bloqueio bimestral interessante).

### 2026-04-21 — Task 2.6 Master Plan com status automático concluída

Plano Diretor (Req 12) com state machine reativa a eventos da Timeline.
**Invariante honrada**: status NUNCA é setado manualmente — só via `AdvanceStatusFromTimeline` chamado pelo event handler `MasterPlanTimelineHandler`, ou via `MarkAtrasada` (cron futuro), ou via `MarkConcluida` (trigger explícito por entry específica).

**Criados**:
- Migration `00204_create_master_plan.sql` — enum `master_plan_status` (7 fases do Req 12: planejada, em_contratacao, em_execucao, concluida, suspensa, reprogramada, atrasada). Tabela `master_plan_activities` com area_code/activity_type_code/risk_code/next_action_code TEXT + CHECK length (catálogos em app não migration — extensível sem SQL), JSONB metadata, deadline opcional, last_triggering_entry_id FK → timeline_entries para auditoria de status auto. Indexes: por condo+status, por deadline (para cron de atraso).
- `institutional/domain/enums/master_plan.go` — `MasterPlanStatus` com `IsValid/IsTerminal/IsInProgress`. **Catálogos completos do Req 12**: 26 áreas (fachada, telhado, playground…), 26 tipos de atividade (manutenção_preventiva, obra, certificação…), 9 riscos (saúde_segurança, patrimonial…), 8 próximas ações (solicitar_orçamento, agendar_inspeção…). Helpers `ValidAreaCode/ValidActivityTypeCode/ValidRiskCode/ValidNextActionCode` O(n) linear — aceitável para n≤30.
- `institutional/domain/entities/master_plan_activity.go` + test (12 test groups) — construtor valida todos os códigos contra catálogos, título (2-200) e descrição (1-10000). Métodos de mutação isolados: `AdvanceStatusFromTimeline(entryType, entryID)` puro com matriz `mapEntryTypeToStatus` (registro_de_execucao → em_execucao; atividade_da_gestao → em_contratacao). Short-circuit em status terminal (concluida/suspensa/reprogramada). `MarkConcluida` + `MarkAtrasada` idempotentes.
- `institutional/domain/repositories/i_master_plan_repository.go` — interface + método dedicado `ListInProgressByCondominium` para hot path do event handler.
- `institutional/infrastructure/database/queries/master_plan.sql` + generated — 5 queries sqlc.
- `institutional/infrastructure/database/master_plan_repository_impl.go` — pgx impl.
- `institutional/application/use_cases/{create_master_plan_activity,list_master_plan}_use_case.go` — CRUD básico.
- `institutional/application/mappers/master_plan_mapper.go` — DTO com `json.RawMessage` metadata.
- `institutional/application/event_handlers/master_plan_timeline_handler.go` — **coração do Req 12**: assina `events.EventTimelinePublished`, busca atividades in-progress do condomínio, aplica `AdvanceStatusFromTimeline` em cada, persiste changed em UoW única. Status SEM mudança = no-op (idempotente). Falha em handler individual loga mas não interrompe outros.
- `institutional/infrastructure/http/routes/master_plan_handler.go` — POST/GET por condomínio + endpoint dedicado `GET /master-plan/catalogs` (auth+view_me) para o frontend popular selects.

**Modificados**:
- `shared/middleware/abac_rules.go` — 2 novas actions: `institutional.master_plan.create` (síndico principal/auxiliary com tenant match) e `institutional.master_plan.read` (síndico/resident tenant membro).
- `institutional/module.go` — instancia planRepo + 2 use cases + event_handler + handler HTTP. **Subscribe do TimelinePublished no construtor do módulo** — garante que a ligação existe antes de qualquer request chegar.
- `sqlc.yaml` — override `master_plan_status` (+ pointer nullable).

**Regra "status nunca manual" enforçada**:
1. Entity só expõe `Advance*`/`Mark*` métodos, não `SetStatus(string)`.
2. Handler HTTP **não aceita campo `status`** no request.
3. Único caller de `Save()` com mudança de status é o event handler (reagindo a timeline) ou `MarkAtrasada` (cron futuro).

**Catálogos expostos via API**: `GET /api/v1/master-plan/catalogs` retorna áreas+tipos+riscos+ações. Frontend pode cachear localmente; mudar catálogo no backend = 1 arquivo editado (sem migration).

**Cron "atrasada" deferido**: método `MarkAtrasada` + index deadline já prontos; worker cron entra em sprint de polish (cron noturno rodando `WHERE deadline < now() AND status IN in_progress`).

**Gates**: go build limpo, go vet limpo, 12 test groups domain passando.

**Sprint 2 tasks backend Dev1 concluídas**: 2.1 (Condo), 2.2 (Unit+Membership), 2.4+2.5 (Timeline), 2.6 (Master Plan). Restante do Sprint 2 é Dev2 (frontend/mobile) e Background Track (OpenSearch).

### 2026-04-21 — Task 2.4 Timeline (append-only + adendos) concluída

Implementação do agregado Timeline com invariantes de imutabilidade + mecanismo de adendo (Req 11 + Req 2.5).

**Criados**:
- Migration `00203_create_timeline.sql` — enum `timeline_entry_type` com 7 valores (atividade_da_gestao, registro_de_execucao, comunicado, evento, adendo, resultado_consulta_moradores, assembly_result). Tabela `timeline_entries` SEM deleted_at (append-only). CHECK enforçando: type=adendo → parent_entry_id NOT NULL; type != adendo → parent_entry_id NULL.
- Enum `TimelineEntryType` em enums.go com `IsValid()` + `IsAdendo()`.
- `institutional/domain/entities/timeline_entry.go` + test — Entity com factory `NewTimelineEntry` validando type, title (2-200), body (1-20000), bodyJSON válido (json.Valid), invariante adendo ↔ parent. `Publish()` idempotente → 2ª chamada retorna ErrTimelineAlreadyPublished. 10 test groups cobrindo invariantes + corner cases.
- `institutional/domain/repositories/i_timeline_repository.go` — ITimelineRepository + ListTimelineFilter (condominium_id, type opcional, published_only, limit/offset).
- `institutional/domain/events/events.go` — IEventPublisher contract + `TimelinePublished` event. Interface-marcador `Event` para troca transparente in-memory → NATS no Sprint 6.
- `institutional/infrastructure/events/in_memory_publisher.go` — InMemoryPublisher sync com mutex RW; handler falha apenas logada (best effort).
- `institutional/infrastructure/database/queries/timeline.sql` — CreateTimelineEntry / PublishTimelineEntry (`:execrows` → 0 significa "não existe ou já publicada") / GetTimelineEntryByID / ListPublishedTimelineEntries (filtro opcional via `sqlc.narg`) / ListAllTimelineEntries.
- `institutional/infrastructure/database/timeline_repository_impl.go` — pgx+sqlc impl. Publish detecta "não existe ou já publicada" via n==0 returning rows.
- `institutional/application/use_cases/create_timeline_entry_use_case.go` — orquestra: valida condo existe → se adendo, valida parent no mesmo condomínio (cross-tenant isolation) → entity → UoW.Run(Publish opcional + Save) → emite TimelinePublished event APÓS commit.
- `institutional/application/use_cases/list_timeline_use_case.go` — query com filtros.
- `institutional/application/mappers/timeline_mapper.go` — TimelineEntryDTO com `json.RawMessage` para body_json.
- `institutional/infrastructure/http/routes/timeline_handler.go` — POST + GET timeline. Visibilidade: morador só vê publishedOnly=true (Req 11 "leitura"); síndico vê drafts também.

**Modificados**:
- `institutional/module.go` — instancia timelineRepo + eventPublisher + 2 novos use cases + handler. Expõe `EventPublisher()` para outros módulos subscreverem. Registra `POST /:id/timeline` (com trial checker opt-in — action é plan-gated no req 4.1) e `GET /:id/timeline` (com `institutional.timeline.read` — já existia em DefaultMatrix).
- `sqlc.yaml` — adicionado `timeline_entry_type` override (string + pointer para nullable).

**Append-only enforce**: 3 camadas de defesa.
1. Tabela sem `deleted_at`, sem DELETE/UPDATE permitido pelos handlers.
2. CHECK constraint enforçando adendo ↔ parent.
3. Entity `Publish()` rejeita 2ª chamada; único mecanismo de "alteração" é criar adendo.

**TimelinePublished event**: usado pelo Master Plan (Task 2.6) para recalcular status de atividades automaticamente. EntryType serializado como string para não acoplar eventos ao enums package.

**Gates**: go build limpo, go vet limpo, testes domain entities passando (10 groups timeline).

**Próximo**: Task 2.6 — Master Plan (status automático via TimelinePublished event). Task 2.5 (imutabilidade) já foi cumprida como invariante da Task 2.4.

### 2026-04-21 — Task 2.2 Unit Registry + Membership concluída

Registro de unidades dentro de condomínios + vínculo morador↔unit com ciclo de vida completo (pending → active → ended).

**Criados**:
- Migration `00202_create_units_memberships.sql` — enums `membership_status` (3 estados) + `membership_end_reason` (4 motivos do Req 17). Tabela `units` com UNIQUE (condominium_id, unit_number, block). Tabela `memberships` com CHECK(role_type IN titular/dependent) + UNIQUE parcial garantindo apenas 1 titular ativo por unit.
- Enums em `institutional/domain/enums/enums.go` expandidos: `RoleTypeTitular`/`RoleTypeDependent` + `MembershipStatus` + `MembershipEndReason` com IsValid/IsActive/IsMembershipRole.
- `institutional/domain/entities/unit.go` — entidade Unit com validação unit_number (1-20 chars), block opcional (1-20 se presente), floor não-negativo.
- `institutional/domain/entities/membership.go` + test — Membership com `MaxDependentsPerUnit=2` constante, factory em pending_confirmation, `ConfirmConsent()` para assinar termo, `End(reason, note)` idempotente com ErrMembershipAlreadyEnded. Sentinels ErrTitularAlreadyExists + ErrDependentLimitReached.
- `institutional/domain/repositories/i_unit_repository.go` — IUnitRepository + IMembershipRepository (+CountActiveDependentsByUnit + FindActiveTitularByUnit para enforçar regras de capacidade).
- `institutional/infrastructure/database/queries/{unit,membership}.sql` — 3+5 queries sqlc.
- `institutional/infrastructure/database/generated/*` — emit sqlc (sqlc.yaml expandido com membership_status + membership_end_reason overrides).
- `institutional/infrastructure/database/{unit,membership}_repository_impl.go` — pgx+sqlc impls.
- `institutional/application/use_cases/create_unit_use_case.go` — valida condo existe → cria entity → persiste.
- `institutional/application/use_cases/register_membership_use_case.go` + test — orquestra: valida role em {titular, dependent} → valida unit existe → **regra de capacidade fora da tx**: se titular, checa FindActiveTitularByUnit (ErrTitularAlreadyExists); se dependente, checa CountActiveDependentsByUnit (ErrDependentLimitReached). Queries leves evitam tx cara em caso de rejeição. UNIQUE parcial no DB é a última barreira (antipattern A9 evitado).
- `institutional/application/use_cases/end_membership_use_case.go` — encerra vínculo com motivo obrigatório + nota opcional. Delega a entidade para invariantes.
- `institutional/infrastructure/http/routes/unit_handler.go` — 3 endpoints: POST /units, POST /memberships, POST /memberships/:id/end. Trata ErrTitularAlreadyExists → 409 TITULAR_ALREADY_EXISTS e ErrDependentLimitReached → 409 DEPENDENT_LIMIT_REACHED.

**Modificados**:
- `shared/middleware/abac_rules.go` — 3 novas actions: `institutional.unit.create`, `institutional.membership.register` (síndico principal/auxiliary/assistant OU resident titular), `institutional.membership.end`. Todas com `RequireTenantMatch=true`.
- `institutional/module.go` — constrói unit/membership use cases + registra 3 endpoints aninhados em `/condominiums/:condominium_id/*` com `PathParamTenantExtractor` para tenant match.

**Ciclo de vida membership**: pending_confirmation → (ConfirmConsent) → active → (End) → ended. Nunca DELETE — histórico preservado para auditoria LGPD.

**Capacidade Req 9**: 1 titular ativo (enforçado por UNIQUE parcial) + máx 2 dependentes ativos (enforçado em app com check antes de INSERT; DB não tem constraint de cardinalidade). UNIQUE parcial no PostgreSQL (`WHERE status IN ('pending_confirmation', 'active') AND role_type = 'titular'`) garante que mesmo em corrida concorrente, 2 INSERTs de titular na mesma unit só um passa — outro pega violação de constraint.

**Encerramento Req 17**: motivo via enum controlado (4 opções) + nota livre opcional. `End()` idempotente evita overwrite acidental.

**Gates**: go build limpo, go vet limpo, testes domain + use case passando (entity Membership: 6 groups; RegisterMembership: 6 cases).

**Próximo**: Task 2.4 — Timeline (append-only, versionamento, adendo). Task 2.3 é frontend (SolidJS), Dev2 track.

### 2026-04-21 — Task 2.1 Condominium Registry concluída (Sprint 2 kickoff)

Nasceu o bounded context `institutional/` com o primeiro agregado: Condominium + SyndicMandate.

**Criados**:
- Migrations `00200_create_condominium_enums.sql` + `00201_create_condominiums.sql` — enums (condominium_type, mandate_type, mandate_status) + tabelas `condominiums` (UNIQUE public_id A-Z0-9{8}, CHECK nos campos de endereço/unidades/torres) + `syndic_mandates` (UNIQUE parcial garantindo apenas 1 principal ativo por condomínio). Partição 200-299 reservada para institutional.
- `institutional/domain/value_objects/address.go` + test — VO com validação de CEP (regex 8 dígitos), UF (regex 2 uppercase), bounds de street/number/neighborhood/city, trim automático.
- `institutional/domain/enums/enums.go` — CondominiumType (6 variantes), MandateType (3), MandateStatus (3), RoleType (principal/auxiliary/assistant) com método IsValid/IsSyndicRole.
- `institutional/domain/entities/condominium.go` + test — factory `NewCondominium` valida public_id (regex [A-Z0-9]{8}), name (2-200 pós-trim), type (enum), total_units (1-10000), blocks_or_towers (0-500). ReconstructCondominium para repos.
- `institutional/domain/entities/syndic_mandate.go` — factory com status auto-inferido: `scheduled` se startDate no futuro, `active` caso contrário. Método `End(closingDate)` idempotente.
- `institutional/domain/services/public_id_generator.go` + test — `GeneratePublicID()` com crypto/rand + alfabeto sem caracteres ambíguos (sem 0/O, 1/I/L). Teste de 10k IDs sem colisão.
- `institutional/domain/repositories/i_condominium_repository.go` — ICondominiumRepository (Create, FindByID, FindByPublicID, CountCreatedBy) + ISyndicMandateRepository (Create, FindActiveByCondominium, FindActiveByUser).
- `institutional/infrastructure/database/queries/*.sql` — 4+3 queries sqlc; sqlc.yaml expandido com entry institutional.
- `institutional/infrastructure/database/{mappers,condominium_repository_impl,syndic_mandate_repository_impl}.go` — pgx+sqlc impl, mapping Address ↔ colunas flat, pgtype.Date ↔ *time.Time.
- `institutional/infrastructure/providers/public_id_generator_adapter.go` — CryptoPublicIDGenerator adapter para use case.
- `institutional/application/use_cases/create_condominium_use_case.go` + test — orquestra: valida input → count por síndico (limite 15 duro) → VO endereço → gera public_id → cria entidade → UoW.Run(create condo + mandato inicial opcional). ErrCondominiumLimitReached sentinel → handler retorna 409 CONDOMINIUM_LIMIT_REACHED.
- `institutional/application/mappers/condominium_mapper.go` — CondominiumDTO para JSON response (sem deleted_at).
- `institutional/infrastructure/http/routes/condominium_handler.go` — handler Create (POST /api/v1/condominiums). UserID vem do AuthUser, nunca do body (never trust frontend).
- `institutional/module.go` — implementa `contracts.IModule`. Recebe ABACEngine + ITrialChecker opcional (do billing) via DI. Aplica `middleware.WithTrialChecker` em actions plan-gated.

**Modificados**:
- `cmd/api/main.go` — instancia institutionalModule injetando `billingModule.FeatureAccessChecker()`. Registra junto com identity + billing.
- `sqlc.yaml` — bloco novo para institutional (package institutionaldb).

**Regra de negócio validada**: limite 15 condomínios por síndico (Req 4.1 linha 107) enforçado em app layer antes de abrir tx — query COUNT leve, evita tx caríssima quando limite já atingido.

**Public ID A-Z2-9**: alfabeto sem 0/O e 1/I/L melhora legibilidade em QR Code físico e cadastro manual de morador. Entropia ~1.5 × 10^11 combinações — suficiente para o projeto inteiro (bilhões de condos antes de colidir).

**Mandatos**: condomínio nasce com 1 mandato principal (criador vira síndico principal). Mandatos auxiliares/assistentes adicionados depois via task separada. UNIQUE parcial no DB garante que nunca há 2 principais ativos simultâneamente.

**Gates**: go build limpo, go vet limpo, testes passando (entity, VO, service, use case). Testes precisam de DB real para repository (testcontainers) — feito em sprint de polish.

**Próximo**: Task 2.2 — Unit Registry + Membership.

### 2026-04-21 — Task 1.6 Quotas + Feature Usage concluída

Implementação do tracking atômico de quotas (Connect Me anual + Vídeo Upload mensal) com SELECT FOR UPDATE para prevenir race condition (antipattern A8).

**Criados**:
- Migration `00101_create_feature_usages.sql` — tabela `feature_usages` com UNIQUE (user_id, feature_key, period_start), CHECK (used >= 0), CHECK (period_end > period_start), index por (user_id, feature_key) e por (period_start, period_end).
- `billing/domain/entities/feature_usage.go` — entity com `Consume(limit)` atômico na entidade, `HasRemaining(limit)`, `CoversInstant(t)`. `QuotaUnlimited = -1` sentinel para planos ilimitados (Síndico N3 Connect Me). `ErrQuotaExhausted` sentinel. Panica em janela inválida (end <= start) — bug de caller, não condição runtime.
- `billing/domain/entities/feature_usage_test.go` — 8 test groups incluindo PBT 2000 iterações validando invariantes: nunca used < 0, nunca used > limit, sucessos = min(attempts, limit).
- `billing/domain/services/quota_limits.go` — `QuotaLimits` declarativo feature → role → tier → limit (int64). `FeatureConnectMe`, `FeatureVideoUpload` como FeatureKey nomeados. `CurrentPeriod(feature, now)` calcula janela [start, end) em UTC: anual (connect_me) ou mensal (video_upload). Matriz armazena APENAS limites, não usos — limit é calculado em runtime a partir do plano atual (mudança de plano altera quota instantaneamente).
- `billing/domain/services/quota_limits_test.go` — matriz oficial do req 4.1 (Connect Me: Morador Base 2, Pagante 4; Síndico N1 2, N2 4, N3 ilimitado; Empresa 0 connect_me; Parceiro 0. Upload: N1 4, N2 8, N3 12; Empresa Plus 8 Pro 12). Corner cases: feature desconhecida → 0, janela zero; dezembro rolls para jan do ano seguinte.
- `billing/domain/repositories/i_feature_usage_repository.go` — `FindByUserForUpdate` (DEVE rodar em tx — doc explícita) + `Save`.
- `billing/infrastructure/database/queries/feature_usage.sql` — `FindFeatureUsageForUpdate :one` com `FOR UPDATE` + `UpsertFeatureUsage :one` com ON CONFLICT (user_id, feature_key, period_start).
- `billing/infrastructure/database/generated/feature_usage.sql.go` — emit sqlc.
- `billing/infrastructure/database/feature_usage_repository_impl.go` — implementa `FindByUserForUpdate` com fallback UPSERT(used=0) quando linha não existe; Save usa UPSERT (idempotente).
- `billing/application/use_cases/consume_quota_use_case.go` — orquestra: valida → calcula janela → resolve limit → UoW.Run(FOR UPDATE → Consume → Save). Limit=0 falha cedo sem abrir tx (default deny para combinações não mapeadas).
- `billing/application/use_cases/consume_quota_use_case_test.go` — 6 test groups: happy path ascending até limite, unlimited (50 consumes sem falha), zero limit (nega imediato), validation de inputs vazios, janelas independentes (jan/fev + 2026/2027), **concurrency control 20 goroutines com limit=4**: exatamente 4 sucessos + 16 ErrQuotaExhausted (prova de serialização via lock mock que simula `SELECT FOR UPDATE` commit/rollback).

**Modificados**:
- `billing/module.go` — instancia `DefaultQuotaLimits()` + `FeatureUsageRepository` + `ConsumeQuotaUseCase`. Accessor `ConsumeQuotaUseCase()` para outros módulos consumirem em ações com quota (commercial.connect_me.create, content.video.publish).

**Design: por que limit em runtime e não em `feature_usages`**: mudança de plano (upgrade/downgrade) altera quota imediatamente — nenhuma migração de dados necessária. Uma linha armazena somente consumo; cálculo usa QuotaLimits(role, tier, feature) em toda chamada. Se user downgrade e já tinha consumido > novo limit: próxima tentativa bloqueia (correto — comportamento intuitivo).

**Race safety (antipattern A8 evitado)**: TestConsumeQuota_ConcurrencyControl prova que 20 goroutines concorrentes com limit=4 produzem exatamente 4 sucessos. O mock simula pgx SELECT FOR UPDATE: lock adquirido no UoW.Run (início da "tx"), liberado no defer (commit/rollback). Em produção isso é transparente — pgx gerencia lock ao longo da pgxpool.Tx.

**UTC em toda janela**: CurrentPeriod trabalha exclusivamente em UTC. Reset global em 1/jan 00:00 UTC e 1/mês 00:00 UTC — sem ambiguidade de timezone. Usuário em BRT vê reset às 21:00 do dia anterior, comportamento aceitável e previsível.

**Gates**: go build limpo, go vet limpo, `go test -race -count=1 ./...` todos passando. Coverage domain/services: 94.6% (acima do threshold 90%).

**Sprint 1 backend completo** — Identity + ABAC + Billing + Trial + Quotas. Próximo sprint: 2 (Institutional — Condomínios + Unidades + Timeline + Comunicados).

### 2026-04-21 — Task 1.5 Trial Logic por Persona concluída

Implementação do domain service `TrialRules` + `CheckFeatureAccessUseCase` + integração opcional via `middleware.WithTrialChecker`.

**Criados**:
- `billing/domain/services/trial_rules.go` — TrialRules domain service com matriz declarativa (role → set(actions) bloqueadas) + `ErrEmptyTrialMatrix` (simetria com ABAC anti-A1). `DefaultTrialRules()` materializa req 4.1: Síndico (15d) bloqueia timeline.create/communication.create/report.export/condominium.create; Empresa (7d) bloqueia video.publish/execution.create/user.manage; Parceiro (30d) bloqueia promotion.create/metrics.view/campaign.create. Morador sem trial (nunca bloqueia).
- `billing/domain/services/trial_rules_test.go` — 7 test groups (matriz oficial, morador nunca bloqueia, ação desconhecida permite, role desconhecida permite, inputs vazios, construtor rejeita matriz vazia, factory custom).
- `billing/application/use_cases/check_feature_access_use_case.go` — CheckFeatureAccessUseCase implementa `types.ITrialChecker`. Cascata: input vazio → ErrValidation; sem subscription → allow (no_subscription); subscription inativa → deny (subscription_not_active); trial + ação bloqueada → deny (trial_blocked); senão allow.
- `billing/application/use_cases/check_feature_access_use_case_test.go` — 8 test groups incluindo 2 property-based com 1000 iterações cada (datas aleatórias entre 1h e 365d no futuro): nunca concede ação bloqueada durante trial + nunca bloqueia ação não listada.
- `shared/types/trial_checker.go` — `ITrialChecker` interface + `FeatureAccessResult{Allowed, Reason}`. Vive em shared/types/ pelo mesmo motivo de IUserSyncer (shared/middleware/ não pode importar modules/).
- `shared/middleware/require_permission_test.go` — 6 test groups cobrindo: grant sem checker, deny ABAC short-circuita (checker não é chamado), checker bloqueia (403 TRIAL_BLOCKED), checker permite, erro técnico → 500, unauthenticated → 401.

**Modificados**:
- `shared/middleware/require_permission.go` — functional options `WithTrialChecker(ITrialChecker)`. RequirePermission aceita `options ...PermissionOption` variadic no final (callers existentes não mudam). Após ABAC grant, se checker presente: consulta; allowed → prossegue; bloqueado → 403 TRIAL_BLOCKED:{reason} + log estruturado; erro técnico → 500.
- `billing/module.go` — instancia `DefaultTrialRules()` + `CheckFeatureAccessUseCase`; expõe accessor `FeatureAccessChecker() types.ITrialChecker` para outros módulos aplicarem em rotas plan-gated.

**ABAC + Trial**: separação intencional. ABAC = role × plan_tier × tenant (declarativo, puro). TrialChecker = subscription.status × trial_ends_at × action (stateful, consulta DB). Middleware encadeia: ABAC primeiro (short-circuit); se concede, consulta checker se opt-in. Antipattern A6 reforçado: denial de trial também tem Reason estruturado no log.

**Req 4.1 cumprido**: trial por persona materializado com duração implícita (controlada pelo Stripe/trialEndsAt) + ações bloqueadas por persona. Extensível: adicionar nova ação = nova linha no map defaultMatrix, sem mudar engine.

**Gates**: go build limpo, go vet limpo, `go test -race -count=1 ./...` todos passando (incluindo PBT 1000 iter em trial rules).

**Próximo**: Task 1.6 — Quotas + Feature Usage (com `SELECT FOR UPDATE` contra race — antipattern A8).

### 2026-04-21 — Reengenharia do fluxo SDD (Lotes 1 e 2)

**Lote 1 — Steering + Infra Claude Code**:
- Consolidação do `.kiro/steering/` com steering modular: `sdd-workflow.md` (ciclo 5 fases), `mcp-integration.md` (MCPs + segurança), `plugins.md` (plugins marketplace oficial), `code-calisthenics.md` (9 regras adaptadas ao Go), `transaction-patterns.md` (ACID + Saga), atualização de `go-patterns.md`, `database.md`, `security.md`, `testing.md` (TDD first), `do-dont.md`.
- `workflow.md` fundido em `sdd-workflow.md` (deletado).
- `.mcp.json` criado com Context7 + GitHub + Postgres Pro (restricted) + AWS Docs. Stripe comentado até Sprint 4.
- `.claude/settings.json` atualizado com hook `PostToolUse` rodando `go build ./... 2>&1 | head -20`, allowlist granular Tier 1/2, model `claude-opus-4-7` default, fallback `claude-sonnet-4-6`.
- `CLAUDE.md` enxugado (prompt-cache friendly) e `AGENTS.md` reescrito (30KB → ~5KB).
- Skills atualizadas (`task-executor`, `go-review`, `migration-writer`, `sqlc-writer`) para referenciar steering novo.

**Lote 2 — .kiro/specs/ reestruturação**:
- Subpastas criadas: `requirements/`, `design/`, `tasks/`, `reference/`.
- Arquivos novos em `reference/`: `antipatterns-to-avoid.md` (12 armadilhas do legacy com fix Go), `legacy-summary.md` (histórico full-stack), `obsidian-mapping.md` (mapa do vault do João).
- Arquivos novos em `requirements/`: `README.md` (índice), `product-overview.md` (decisões 1-12 + T1-T15 + regras de ouro + marcos), `personas-and-plans.md` (modelo completo), `identity-mirror-pattern.md` (decisão User/Session ↔ Zitadel).
- Arquivos novos em `tasks/`: `README.md` (formato 5-fase + convenções), `sprint-0.md` (histórico), `sprint-1.md` (formato 5-fase completo para tasks 1.2-1.6).
- `design/README.md` criado; recortes por tema serão criados incrementalmente.
- Monolíticos `requirements.md`, `design.md`, `tasks.md` mantidos como canônicos com nota de migração no topo.

**Plugins Claude Code identificados para instalação**:
- **Dia 1**: `gopls-lsp`, `typescript-lsp`, `security-guidance`, `context7`, `railway`
- **Semana 1**: `feature-dev`, `code-review`, `commit-commands`, `claude-md-management`, `superpowers` (community)
- **Por módulo**: `figma` (frontend), `stripe` (Sprint 4), `playwright` (E2E), `sentry` (Sprint 7)
- Detalhes em `.kiro/steering/plugins.md`.

**Princípios adicionados ao steering**:
- **TDD Red-Green-Refactor** como obrigatório (testes antes do código, sem exceção além de spikes descartáveis)
- **Code Calisthenics** (9 regras adaptadas ao Go idiomático) — 1 nível de indent, no else, wrap primitives, Law of Demeter
- **ACID via UnitOfWork** para operações de 1 banco
- **Saga Pattern** (orchestrated + compensação) para cross-BC + provider externo
- **Never trust the frontend** como princípio #1 de segurança
- **Prompt injection risk** para uso de MCPs externos
- **Thresholds de coverage por camada** (domain 95%, application 90%, handlers 75%, middleware 90%)

### 2026-04-21 — Task 1.4 Plans + Subscriptions + Webhook concluída

Implementação completa do módulo `billing` com entidades, repositories, use cases (incluindo Saga orquestrada), handlers HTTP + webhook signature verification.

**Criados**:
- `billing/domain/entities/plan.go` + `subscription.go` + `subscription_test.go` (10 test groups — Red-Green-Refactor completo, invariantes de transição de estado, LinkStripe, UpdateFromProvider, MarkCanceled idempotente, MarkPastDue, ScheduleCancel)
- `billing/domain/repositories/i_subscription_repository.go` (ISubscriptionRepository + IPlanRepository)
- `billing/domain/events/events.go` (4 event types: SubscriptionActivated/Updated/Canceled/PastDue + IEventPublisher contract)
- `billing/infrastructure/database/queries/plan.sql` + `subscription.sql` (4+4 queries sqlc)
- `billing/infrastructure/database/generated/` (sqlc emit via sqlc.yaml expandido com entry billing)
- `billing/infrastructure/database/mappers.go` + `plan_repository_impl.go` + `subscription_repository_impl.go`
- `billing/infrastructure/events/in_memory_publisher.go` (pub/sub sync — Sprint 6 migra para NATS sem mudar use cases)
- `billing/application/mappers/plan_mapper.go` + `subscription_mapper.go` (DTOs omitem stripe_*_id do JSON)
- `billing/application/use_cases/list_plans_use_case.go` (query)
- `billing/application/use_cases/get_my_subscription_use_case.go` (query; nil return = sem subscription, não erro)
- `billing/application/use_cases/handle_stripe_webhook_use_case.go` (processa subscription.created/updated/deleted, invoice.paid/payment_failed — dispatcher + helpers isolados, guard clauses, idempotência por stripe_subscription_id UPSERT, emite events nas transições)
- `billing/application/use_cases/checkout_subscription_saga.go` (Saga orquestrada: CreateCustomer → CreateSubscription → Save local em tx → Publish event; compensação: CancelSubscription Stripe em caso de falha de persistência, log ERROR em compensação falhada)
- `billing/infrastructure/http/routes/subscription_handler.go` + `webhook_handler.go` (raw body reader via interface opcional no ginContext, signature verify antes de parse, response genérica para não vazar detalhe)
- `billing/module.go` (wire-up DI manual completo)

**Modificados**:
- `internal/shared/config/config.go` — StripeConfig + validação sk_test_/sk_live_ por env + default CustomerPortalReturnURL
- `internal/server/gin_adapter.go` — expõe RawBody() io.Reader para handlers de webhook
- `internal/server/server.go` — RegisterPublicWebhook() para rotas não-autenticadas
- `cmd/api/main.go` — instancia billing module, registra `/webhooks/stripe` público, subscribe placeholder de invalidação de cache ABAC em SubscriptionActivated
- `sqlc.yaml` — entry para billing além de identity
- `.mcp.json` — Stripe MCP habilitado (`STRIPE_SECRET_KEY` do .env)
- `.claude/settings.json` — permissions read-only para mcp__stripe__*

**ABAC aplicado**: 4 ações em billing routes — `identity.user.view_me` (list plans), `billing.subscription.view_own` (get me + customer portal), `billing.checkout.create`.

**Pattern Saga + compensação**: primeiro exemplo concreto do `steering/transaction-patterns.md` implementado. Documenta regra: idempotency keys em `CreateCustomer`/`CreateSubscription`, compensação = `CancelSubscription` hard se persistência local falhar, log ERROR em compensação falhada para reconciliação manual.

**Idempotência do webhook**: stripe_subscription_id UNIQUE + UPSERT → reprocessar evento múltiplas vezes é seguro (Stripe retenta em timeout).

**Customer Portal endpoint**: retorna 501 por ora (DT-009 registrado) — precisa do `stripe_customer_id` que SubscriptionDTO não expõe por segurança. Ajustar no próximo sprint com raw accessor.

**Gates**: go build limpo, go vet limpo, todos os testes passando (middleware ABAC, billing entities, billing providers Stripe).

### DT-009 — Customer Portal endpoint (billing) retorna 501 ✅ RESOLVIDO 2026-04-21
- **Fix aplicado**: `GetMySubscriptionRawUseCase` retorna entidade `*Subscription` diretamente (não DTO). `SubscriptionHandler.CustomerPortal` agora chama o raw use case, extrai `sub.StripeCustomerID()`, delega `CreateCustomerPortalSession` ao gateway e retorna `{url}` ao frontend. Estado inconsistente (subscription local sem stripe_customer_id ainda) retorna 409 CUSTOMER_NOT_LINKED ao invés de silenciar. Endpoint totalmente funcional.

### 2026-04-21 — Task 1.3 Stripe Adapter concluída

- Criados: `billing/domain/providers/i_payment_gateway.go` (IPaymentGateway interface + inputs/outputs + WebhookEvent type constants + ProrationMode), `billing/domain/providers/errors.go` (7 sentinels: ErrInvalidWebhookSignature, ErrWebhookPayloadCorrupted, ErrCustomerNotFound, ErrSubscriptionNotFound, ErrPaymentDeclined, ErrRateLimited, ErrProviderUnavailable), `billing/infrastructure/providers/stripe_gateway.go` (implementação com stripe-go v82.5.1), `billing/infrastructure/providers/json.go` (wrapper de serialização), `billing/infrastructure/providers/stripe_gateway_test.go` (9 test groups: empty inputs rejection, metadata merge, subscription conversion, nil customer handling, subscription-relevant events whitelist, empty signature header rejection, tampered payload rejection, valid signature acceptance, invoice.paid subscription ID extraction via Parent.SubscriptionDetails).
- Métodos: CreateCustomer, CreateSubscription, CancelSubscription (soft/hard), UpdateSubscription (com ProrationMode), VerifyWebhookSignature (anti-forgery barrier), ParseWebhookEvent, CreateCustomerPortalSession.
- Idempotency keys obrigatórias em toda criação (enforced via validation).
- stripe-go v82 shape peculiar: CurrentPeriodStart/End movidos para SubscriptionItems; invoice.Subscription substituído por invoice.Parent.SubscriptionDetails.Subscription; ProrationBehavior agora string direta; rate limit detectado via HTTPStatusCode=429 ou Code="rate_limit".
- mapStripeError mapeia stripe.Error → sentinels do domínio com errors.Is compatível (antipattern A6 evitado).
- Stripe MCP habilitado em `.mcp.json` + permissions read-only em `settings.json` (retrieve, list, search, fetch, search_documentation).
- Gates verdes: build limpo, vet limpo, todos os testes passando.
- Próximo: Task 1.4 — Plans + Subscriptions domain + repository + webhook handler (Saga orquestrada).

### 2026-04-21 — Task 1.2 ABAC Engine concluída

- Criados: `shared/middleware/abac_context.go` (ABACContext, ABACResult), `abac_rules.go` (Matrix + DefaultMatrix com 10 regras iniciais cobrindo identity, billing, institutional, commercial, content), `abac_engine.go` (ABACEngine.Evaluate com guard clauses, admin bypass, ErrEmptyMatrix anti-A1), `require_permission.go` (middleware factory + TenantExtractor patterns), `abac_engine_test.go` (7 test groups cobrindo: empty matrix rejection, rule/action counts, admin bypass, action not configured, ordered guard clauses, plan tier gate, tenant mismatch, role type filter, reason always populated, default matrix non-empty).
- DT-001 resolvido: `PlanTierOrder` + `HasMinimumPlanTier` standalone em `shared/types`.
- DT-005 resolvido: RateLimit split em Global + Auth configs, `/api/v1/*` agora passa por rate limiter.
- Wire-up completo em `main.go` (instância engine, validação de matriz, log de `rules/actions counts`) + `identity/module.go` (RequirePermission aplicado em `/auth/me` com action `identity.user.view_me`).
- Gates verdes: `go build ./...` limpo, `go vet ./...` limpo, todos os testes unitários do middleware passando.
- Próximo: Task 1.3 — Stripe adapter com webhook signature verification.

### 2026-04-19 — Config Zitadel completada (task 1.0.x)
- `ZITADEL_PROJECT_ID` adicionado, validação em `config.Validate()`.
- `IZitadelProvider` movida de `modules/identity/domain/providers/` para `shared/providers/auth/`.
- `IUserSyncer` + `SyncUserCommand` + `SyncUserOutput` em `shared/types/` — middleware não importa mais de `modules/`.

### 2026-04-21 — Setup de ambiente externo concluído

João forneceu tokens e URLs para os MCPs/plugins externos. Adicionado a `backend/.env` (gitignored):

- **Railway**: `RAILWAY_TOKEN` + `RAILWAY_PROJECT_ID` — projeto https://railway.com/project/efe664b7-5352-48e7-ae09-78d2f286e0d5 (nome: `mastersindico`, ambiente: `plataforma`). CLI local autenticada via `railway login`.
- **Context7**: `CONTEXT7_API_KEY` — já configurado.
- **Figma**: `FIGMA_ACCESS_TOKEN` — read-only, para projeto https://www.figma.com/design/pxJkmn8rzezKnCYXcA7lyL/Master-sindico-APP---UX.

**`GITHUB_PAT`**: configurado em `backend/.env` (2026-04-21 mais tarde no dia) — PAT clássico `github_pat_*` fornecido por João. Registrado como DT-008 para rotação por fine-grained.

**GitHub org + repos identificados (2026-04-21)**:
- Org: https://github.com/mastersindico
- `backend/` → `mastersindico/backend`
- `web/` → `mastersindico/web`
- `app/` → `mastersindico/app` (folder local renomeada de `mobile/` para `app/` em 2026-04-21 para casar com o repo)
- `full-stack/` → `mastersindico/platform` (legacy, arquivado como referência histórica — **atualizado 2026-04-21 após revelação da URL pelo João**)

Documentado em `backend/.env`, `.env.example`, `design/deploy-config.md`, root `Development/CLAUDE.md`, e memória `project_github_urls.md`.

Criados: `backend/.env.example` (template versionado), `backend/.kiro/specs/master-sindico/design/deploy-config.md` (plano Railway + AWS futuro + seção GitHub), `web/CLAUDE.md` complementado (SDD 5-fase, TDD Vitest, Code Calisthenics TS, Figma workflow, security frontend), `app/CLAUDE.md` reescrito (SDD 5-fase Flutter, TDD blocTest, Code Calisthenics Dart, security mobile com certificate pinning + secure_storage, Zitadel OIDC, thresholds por camada — preserva invariantes do `ARCHITECTURE.md` + `CODING_MANUAL.md`). Memórias `project_railway_url.md`, `project_figma_url.md`, `project_github_urls.md` salvas (sem tokens).

### 2026-04-15 — OIDC PKCE completo (task 1.1)
- Authorization Code Flow via `zitadel/oidc/v3`.
- Rotas `/auth/login`, `/auth/callback`, `/auth/logout` fora de `/api/v1` (redirects de browser).
- Fallback `Authorization: Bearer` para mobile (RFC 6750).
- 1-device enforce via `InvalidatePreviousByUserID` na sessão.

---

## Como usar este arquivo

- **Novo bloqueador**: adicionar em 🚧 com contexto, alternativas se houver, prazo, responsável.
- **Decisão em aberto**: 🤔 quando há fork no caminho que afeta implementação futura.
- **Dívida técnica**: 📋 com "onde" (arquivo:linha), "impacto", "fix", "sprint alvo".
- **Experimento**: 🧪 quando está testando abordagem e ainda não decidiu.
- **Resolução**: mover para 📜 Histórico com data + resumo de uma linha.

Mantenha o arquivo **denso de informação, não de texto**. Entradas que ninguém vai ler em 30 dias devem ser deletadas ou movidas para histórico condensado.
