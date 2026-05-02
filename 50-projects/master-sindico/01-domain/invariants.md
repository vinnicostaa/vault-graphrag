---
title: Invariants — Master Síndico (testáveis, por BC)
type: spec
tags: [domain, ddd, invariants, pbt, master-sindico, v2]
source: 90-ingested/from-vault-50-projects-master-sindico/01-domain/{invariants,domain-rules,business-rules}.md + specs/requirements/*
created: 2026-04-23
updated: 2026-04-23
---

# Invariants — Master Síndico v2

Lista canônica dos **invariantes testáveis** do domínio. Cada invariante é uma propriedade que **nunca** violada em runtime. Violação = bug, não edge case.

**Formato**: `INV-###` | descrição | BC owner | enforcement (CHECK constraint / Factory validation / UNIQUE / FOR UPDATE / PBT / dual) | teste correspondente.

**Mapping legado**: IDs aqui (`INV-###`) são **novos** pra v2 e referenciam IDs legados (`I-###`, `DR-###`) quando aplicável, pra rastreabilidade.

---

## identity (INV-001 a INV-010)

| ID | Invariante | Owner | Enforcement | Teste |
|---|---|---|---|---|
| **INV-001** | `users.zitadel_id` UNIQUE (1 User MS = 1 User Zitadel) | identity | UNIQUE constraint (migration 00X) | Integration |
| **INV-002** | 1 CPF = 1 User (quando `user_type = PF`) | identity | UNIQUE constraint parcial | Integration |
| **INV-003** | 1 CNPJ = 1 User (quando `user_type = PJ`) | identity | UNIQUE constraint parcial | Integration |
| **INV-004** | `users.role` nunca string vazia — 403 `NO_ROLE_ASSIGNED` antes do UPSERT (T15) | identity | Middleware guard + CHECK `role IN enum` | Unit + Integration |
| **INV-005** | ≤ `MAX_ACTIVE_SESSIONS` Session ativa por User (default 1 — BR-004) | identity | `InvalidatePreviousByUserID` dentro de UoW antes de criar nova | Integration (concorrência) |
| **INV-006** | Toda `session.user_id` existe em `users` (FK) | identity | FK constraint | Integration |
| **INV-007** | `sessions.zitadel_session_id` nunca re-usado após revogação (INSERT, não UPDATE) | identity | Sem UPDATE; INSERT com check de duplicata | Integration |
| **INV-008** | Password nunca armazenado plain (delegado ao Zitadel Argon2) | identity | Design (campo `password_hash` removido do schema local) | Unit (schema test) |
| **INV-009** | DeviceFingerprint hash server-side SHA-256 — cliente não envia hash direto | identity | VO `DeviceFingerprint.New(...)` aplica SHA-256 (IR-011) | Unit |
| **INV-010** | Zitadel rejeita user sem role → middleware responde 403 sem UPSERT | identity | Middleware `Authenticate` guard (T15) | Integration |

---

## billing (INV-011 a INV-020)

| ID | Invariante | Owner | Enforcement | Teste |
|---|---|---|---|---|
| **INV-011** | `users.plan_id` aponta a Subscription ativa OU NULL (soft-block não apaga) | billing | FK com `ON DELETE RESTRICT`; query `users.plan_id IS NULL OR EXISTS(...)` | Integration |
| **INV-012** | `subscriptions` é append-only (novo registro a cada mudança; sem UPDATE) | billing | Convenção + CI grep; hook Postgres (opcional) | Integration + static check |
| **INV-013** | Trial nunca volta após conversão para paid (DR-011, BR-001) | billing | Factory `Subscription.Activate(...)` rejeita transição `active → trial` | Unit + PBT |
| **INV-014** | `Money.value` é int64 centavos BRL — nunca float (T10, DR-011) | billing | VO `Money` expõe só `int64`; operações preservam tipo | Unit (tipo) |
| **INV-015** | Stripe `webhook_events.event_id` UNIQUE (idempotency ledger) | billing | UNIQUE constraint | Integration |
| **INV-016** | Quota saldo ∈ [0, ∞]; `used ≤ limit` quando `!unlimited` (DR-010) | billing | VO `Quota.Consume(n)` + CHECK no banco; `FeatureUsage` via `SELECT FOR UPDATE` (A8 antipattern fix) | Unit + Integration + PBT (300 casos) |
| **INV-017** | TrialDuration persona-aware (síndico 15d / empresa 7d / parceiro 30d / morador —) (BR-001) | billing | Factory `Plan.TrialDurationFor(persona)` | Unit |
| **INV-018** | Webhook HMAC verificado **antes** de parse body (DR-020) | billing | Middleware `VerifyStripeSignature` antes handler | Integration |
| **INV-019** | Soft-block sem grace period (Decision 4) — ABAC imediatamente nega ao expirar | billing | Middleware ABAC consulta `plan_tier` em cache; invalidação por webhook | Integration |
| **INV-020** | Planos em cache Redis TTL 1h; invalidação manual em deploy ou via webhook | billing | Wrapper `IPlanCache` com TTL + `Invalidate()` | Unit + Integration |

---

## institutional (INV-021 a INV-035)

| ID | Invariante | Owner | Enforcement | Teste |
|---|---|---|---|---|
| **INV-021** | Σ `FracaoIdeal` de Units em Condominium ≤ 100% (DR-001) | institutional | CHECK constraint trigger + Factory `Condominium.AddUnit(u)` rejeita se excede | Unit + PBT (100 casos) + Integration |
| **INV-022** | `FracaoIdeal ∈ (0, 100]` (DR-002) | institutional | VO `FracaoIdeal.Validate()` rejeita 0 ou > 100; CHECK no banco | Unit + PBT |
| **INV-023** | `fracao_ideal` coluna NUMERIC(5,4) — nunca float (T10) | institutional | Schema migration | Schema test |
| **INV-024** | TimelineEntry imutável: sem UPDATE, sem DELETE, sem `deleted_at` (DR-007) | institutional | Schema sem `updated_at`/`deleted_at`; CI grep bloqueia `UPDATE timeline_entries`/`DELETE FROM timeline_entries`; trigger Postgres opcional | Schema test + CI grep + Unit |
| **INV-025** | Announcement imutável após `state = published` (DR-008) | institutional | Handler update rejeita se `state=published`; factory `Announcement.Edit()` retorna erro | Unit + Integration |
| **INV-026** | Membership ativa única por (user, condominium) em dado momento | institutional | UNIQUE parcial `(user_id, condominium_id) WHERE status='active'` | Integration |
| **INV-027** | ≤ 15 condomínios por síndico (Decision 11) | institutional | Validação no `AssignSyndicMandateUseCase` + CHECK em job | Unit + Integration |
| **INV-028** | Mandate `end_date > start_date` | institutional | CHECK constraint | Schema |
| **INV-029** | Unit tem exatamente 1 titular + até 2 dependentes (Req 9) | institutional | Factory `Unit.AddDependent(d)` rejeita se > 2; factory `Unit.AssignTitular(u)` UPSERT | Unit |
| **INV-030** | `condominium.public_id` UNIQUE (8 chars alfanumérico gerado) | institutional | UNIQUE constraint + validador `PublicID.New(...)` | Integration |
| **INV-031** | `MasterPlan.activities` tem `status` atualizado **apenas** via evento da Timeline (nunca direto) | institutional | Handler `ExecutionValidated` event; repo `MasterPlan.UpdateActivityStatus` internal-only | Integration |
| **INV-032** | Correção de Timeline = nova entry com `corrects_entry_id` link (nunca edita entry original) | institutional | Factory `TimelineEntry.Correct(entry_id, reason)` cria nova entry | Unit + Integration |
| **INV-033** | `audit_trail` append-only (sem UPDATE/DELETE; retenção 5 anos) | institutional/cross-domain | Schema sem `updated_at`; arquivamento após 1 ano em MinIO | Schema + Integration |
| **INV-034** | `compliance_declarations` é anual (uma por ano por condomínio) | institutional | UNIQUE parcial `(condominium_id, year)` | Integration |
| **INV-035** | `conflict_of_interest_declarations` obrigatória no onboarding síndico | institutional | Guard em `FinishOnboardingUseCase` | Unit + Integration |

---

## commercial (INV-036 a INV-055)

| ID | Invariante | Owner | Enforcement | Teste |
|---|---|---|---|---|
| **INV-036** | Reputação é função determinística de métricas — sem randomness, sem input humano direto (BR-003, DR-016) | commercial | Domain service `ReputationCalculator.Compute(company)` puro (mesmo input → mesmo output) | Unit (determinismo) |
| **INV-037** | Connect Me é unidirecional — sem thread, sem reply, sem histórico de conversa (Rule 3, I-022) | commercial | Schema sem `replies`/`thread_id`; API sem endpoint de reply | Schema + API contract test |
| **INV-038** | Proposta única por (ConnectMeRequest, company) | commercial | UNIQUE(connect_me_request_id, company_id) | Integration |
| **INV-039** | Proposal imutável após `status = sent` (Rule 2, Req 20) | commercial | Handler update rejeita; correção via Amendment (Req 25) | Unit + Integration |
| **INV-040** | ClosedDeal imutável após dupla assinatura (Rule 2, Req 22) | commercial | Flag `is_immutable=true` após `company_signed_at` + `sindico_signed_at` | Unit + Integration |
| **INV-041** | ClosedDeal auto-publica TimelineEntry tipo `registro_execucao` (Req 22) | commercial→institutional | Event `ClosedDealSigned` handler em institutional cria entry dentro de UoW | Integration |
| **INV-042** | SupplierVoteCast único por (session, voter) — UNIQUE DB (DR-005, A-026 legado fix) | commercial | UNIQUE(session_id, voter_id); UoW + `SELECT FOR UPDATE` contra TOCTOU | Integration |
| **INV-043** | CompanyEvaluation única por (company, evaluator, condominium) — UNIQUE DB (DR-006, A-029 legado fix) | commercial | UNIQUE constraint + `ErrConflict` em Create | Integration + PBT |
| **INV-044** | CompanyAssessment (Req 26) anônima ao avaliado — empresa não vê `evaluator_user_id` | commercial | API response mapper filtra; ABAC nega leitura do campo pra empresa | Integration |
| **INV-045** | Quota ConnectMe decrementa no `publish` (não no `draft`) | commercial | Use case `PublishConnectMeUseCase` dentro de UoW: consome quota + insere | Unit + Integration |
| **INV-046** | Quota ConnectMe reset anual (1º de janeiro) | commercial/billing | Reset key Redis `ms:quota:{userID}:connect_me:{year}` — expira 31/12 23:59 | Integration |
| **INV-047** | Connect Me auto-close após 48h sem interesse (Req 19) | commercial | Job agendado (cron NATS) + handler | Integration |
| **INV-048** | LGPD log obrigatório ao revelar dados síndico em Connect Me (Req 19) | commercial/cross-domain | Handler `InterestExpressed` dentro de UoW: reveal + log | Integration |
| **INV-049** | Marketing Agency: zero acesso a dados de negócio (deals, propostas, Connect Me) | commercial/identity | ABAC policy `delegated_marketing` nega `deal:*`, `proposal:*`, `connect_me:*` | Integration (100+ tests) |
| **INV-050** | Amendment (adendo) é o único mecanismo de modificação de registro imutável (Req 25) | commercial | Schema link `previous_record_id`; factory `Amendment.Create(original_id, reason)` | Unit + Integration |
| **INV-051** | NeighborhoodBenefit cupom: 4h de cooldown por (user, loja) — anti-spam (D-012 legado) | commercial | UNIQUE parcial + index `(user_id, vizinhanca_user_id, created_at)` + check 4h | Integration |
| **INV-052** | CouponCode formato `ID_LOJA(5) + SEQ(3) + PALAVRA(5)` (ex: `00123055PAO`) | commercial | VO `CouponCode.New(...)` valida regex | Unit |
| **INV-053** | Palavra do dia em cupom: máximo 5 letras maiúsculas | commercial | VO `PalavraDia.Validate()` rejeita > 5 chars / lowercase | Unit |
| **INV-054** | ReputationTierChanged event dispara recálculo e notificação | commercial | Handler `RecalculateReputation` idempotente | Integration |
| **INV-055** | HarassmentReport confirmado suspende empresa (reputação muda pra `suspended` flag, não rebaixa tier) | commercial | Handler event + flag | Integration |

---

## content (INV-056 a INV-070)

| ID | Invariante | Owner | Enforcement | Teste |
|---|---|---|---|---|
| **INV-056** | Vídeo `published` → `locked_until = published_at + 90d` (DR-015, BR-008) | content | Factory `Video.Publish()` seta `locked_until`; handler `RemoveVideo` valida | Unit + Integration |
| **INV-057** | Remove vídeo pre-90d requer `user.is_admin = true` + audit log do bypass | content | `RemoveVideoUseCase` valida; audit log sempre | Integration |
| **INV-058** | Moderação obrigatória antes de `published` (state `moderation → approved → published`) | content | State machine em `Video` aggregate | Unit (state transitions) |
| **INV-059** | Trava trimestral: substituir **mesmo** vídeo só após 90 dias (Rule 7) | content | Factory `Video.Replace(original_id)` valida `NOW() >= original.published_at + 90d` | Unit + Integration |
| **INV-060** | Métricas (views, likes, comments) não vazam entre tenants (Rule 5) | content | Query sempre escopada por `tenant_id`; ABAC valida | Integration (100+ cross-tenant tests) |
| **INV-061** | Vídeos empresa bloqueados pra morador por default | content | Rota `GET /videos/{id}/playback` valida `video.tenant_type='company' AND user.role='resident'` → ABAC nega | Integration |
| **INV-062** | Unlock morador só via `VideoVisibilityGrant` ativo (Rule 4, Req 103) | content | Tabela `video_visibility_grants` + join no query + ABAC valida | Integration |
| **INV-063** | `VideoVisibilityGrant` revogado automaticamente ao fim da votação (evento `VotingClosed`) | content/assembly | Handler event revoga grant dentro de UoW | Integration |
| **INV-064** | Mux webhook HMAC verificado **antes** de parse (DR-020) | content | Middleware `VerifyMuxSignature` público `/webhooks/mux` | Integration |
| **INV-065** | Saga UploadVideo com compensação: create Mux sucesso + create local falha → `CancelMuxUpload` (A-027 legado) | content | `UploadVideoSaga` com compensator explícito | Integration (simulate failure) |
| **INV-066** | Agência de Marketing: token escopo restrito `videos:*`, `analytics:read` (Req 30) | content/identity | ABAC policy por token | Integration |
| **INV-067** | Parceiro Vizinhança: sem acesso a LMS (Academia) | content | ABAC policy nega; UI oculta botão | Integration + E2E |
| **INV-068** | Certificate gerado automaticamente ao `Enrollment.progress = 100%` | content | Handler `LessonCompleted` verifica `CourseCompleted` → emite `CertificateGenerated` | Integration |
| **INV-069** | Ebook access filtrado por `plan_tier` (free/plus/pro) | content | ABAC + query filter | Integration |
| **INV-070** | Signed playback URL Mux: TTL curto (≤ 24h) — previne hotlinking | content | `GetPlaybackURL(id)` do `IVideoProvider` seta TTL | Unit |

---

## assembly (INV-071 a INV-085)

| ID | Invariante | Owner | Enforcement | Teste |
|---|---|---|---|---|
| **INV-071** | Vote único por (agenda_item_id, voter_id) — UNIQUE DB (DR-004, A-025 legado fix) | assembly | UNIQUE constraint + ErrConflict em Create | Integration + PBT TOCTOU |
| **INV-072** | Minutes (Ata) imutável após `published` (DR-009) + imutável após `homologated` | assembly | Handler update rejeita; correção via TimelineEntry nova em institutional | Unit + Integration |
| **INV-073** | Quórum calculado por Σ FracaoIdeal presente, não por pessoa (BR-007) | assembly | Domain service `QuorumCalculator.IsReached(assembly, presentes)` pesa por fração | Unit + PBT |
| **INV-074** | Assembly não inicia sem edital publicado (DR-014, A-013 legado fix) | assembly | Use case `StartAssemblyUseCase` valida `assembly.edital_published_at != NULL` | Unit + Integration |
| **INV-075** | ≤ 1 assembleia ativa por condomínio (Req 48) | assembly | UNIQUE parcial `(condominium_id) WHERE status IN ('published','in_progress')` | Integration |
| **INV-076** | Vote após timeout → registrado como `Absent` (ausência momentânea ≠ abstenção automática após 15min — Req 57) | assembly | Job timeout cria vote `Absent` idempotente | Integration |
| **INV-077** | Procuração: proxy não pode votar diferente do titular (Req 52) | assembly | Use case `CastVoteUseCase` valida se é proxy e se já houve voto do titular | Unit + Integration |
| **INV-078** | Procuração válida até 24h **após** assembleia encerrada (permite voto atrasado) | assembly | `Proxy.IsValid(now)` considera `assembly.ended_at + 24h` | Unit |
| **INV-079** | Procuração pode ser revogada pelo titular até 1h antes da assembleia | assembly | `Proxy.Revoke()` rejeita se `NOW() > assembly.scheduled_date - 1h` | Unit |
| **INV-080** | Pauta `published_at != NULL` → read-only (lock) | assembly | Handler update rejeita | Unit + Integration |
| **INV-081** | Presidente síndico **não** vota (imparcialidade, Req 60.2) | assembly | Use case `CastVoteUseCase` rejeita se `user.id == assembly.president_id` | Unit + Integration |
| **INV-082** | Fração ideal em votação: NUMERIC(5,4), nunca float (T10) | assembly | Schema + VO `FracaoIdeal` | Schema test |
| **INV-083** | `fracao_ideal_weight` em `assembly_votes` vem de `unit.fracao_ideal` no momento do voto (snapshot) | assembly | Factory `Vote.Cast(unit, ...)` snapshot fração | Unit |
| **INV-084** | Saga StartLive com compensação: falha local → `EndRoom` no Livekit (A-028 legado) | assembly | Saga `StartAssemblyLiveSaga` com compensator | Integration |
| **INV-085** | Homologação é votação separada obrigatória (não automática) pós-ata — Req 60.6 | assembly | State `closed → homologated` só via `HomologateMinutesUseCase` com voto aprovado | Unit + Integration |

---

## cross-domain (INV-086 a INV-100)

| ID | Invariante | Owner | Enforcement | Teste |
|---|---|---|---|---|
| **INV-086** | ABAC ordem estrita: `admin_bypass → action_configured → role_allowed → tenant_matches → plan_tier_sufficient` (DR-012) | cross-domain | Engine `abac/engine.go` sequencial | PBT (400 casos) |
| **INV-087** | ABAC default-deny — ação não configurada → nega (BR-012) | cross-domain | Engine retorna `Deny(action_not_configured)` | Unit + Integration |
| **INV-088** | Admin bypass exige `user.is_admin = true` + audit log do bypass (BR-012) | cross-domain | Engine emite `AuditEntry{action:'admin_bypass',...}` sempre que usa | Integration |
| **INV-089** | Tenant isolation: toda query escopada por `condominium_id`/`company_id` (DR-013) | cross-domain | Convenção sqlc + 100+ testes cross-tenant | Integration (cross-tenant) |
| **INV-090** | Audit trail append-only: sem UPDATE, sem DELETE (Req 37) | cross-domain | Schema sem `updated_at`; arquivamento periódico | Schema + static check |
| **INV-091** | LGPD logs retenção 5 anos; direito ao esquecimento após vencimento (BR-010) | cross-domain | Job expiração idempotente | Integration |
| **INV-092** | PII nunca em logs (CPF, CNPJ, email, token, password — DR-019) | cross-domain | CI grep com regex (lista negra) | Static check |
| **INV-093** | HMAC verify **antes** de parse em todo webhook (Stripe/Mux/Livekit — DR-020) | cross-domain | Padrão middleware obrigatório | Pattern enforce + Integration |
| **INV-094** | DomainEvent.id UUIDv7 (ordenável sem coordenação — DR-017) | cross-domain | `pkg/utils/uuid7.go` | Unit |
| **INV-095** | Events publicados dentro da mesma tx via outbox pattern (tabela `domain_events`) | cross-domain | `UoW.PublishEvent()` commit-a outbox junto; publisher async lê | Integration |
| **INV-096** | `config.Validate()` falha startup se matriz ABAC vazia (A1 antipattern fix) | cross-domain | `main.go` bootstrap valida antes de aceitar requests | Integration (startup test) |
| **INV-097** | Rate limit: `/auth/*` 20rpm burst 5; `/api/v1/*` 60rpm burst 10 (A-006, A-032 legado) | cross-domain | Middleware `RateLimiter` com cleanup goroutine | Integration |
| **INV-098** | CORS `*` bloqueado em staging/prod via `Config.Validate()` (A-005 legado) | cross-domain | Validação no startup | Integration |
| **INV-099** | Secrets gitignored (`.env`, `zitadel-key*.json`) (A-018 legado) | cross-domain | `.gitignore` + CI secret scanner | Static check |
| **INV-100** | Cookie httpOnly `.mastersindico.com.br` + Secure + SameSite=Lax (T9) | cross-domain | Middleware sessão seta flags | Integration |

---

## security hardening (INV-101 a INV-110) — Fase 5 pentest + dual-check SEC

Série novos invariantes **obrigatórios pré-M1** derivados de findings 🔴 SEC+PEN. Cada INV referencia o finding `A-DC-###` que a originou + ADR que formaliza a regra.

| ID | Invariante | Owner | Enforcement | Teste | Finding / ADR |
|---|---|---|---|---|---|
| **INV-101** | Toda operação de `Delete` em recurso user-scoped valida `resource.user_id == ctx.user_id` **antes** de qualquer query. Mismatch retorna 404 (não 403 — não vaza existência) | cross-domain / identity | Middleware `OwnershipCheck(resource)` + PBT: `∀ user_A, resource_B onde resource_B.owner_id != user_A → DELETE == 404` | Integration + PBT | [[../11-audit/dual-check-log/2026-04-23-security-gaps#A-DC-SEC-001]] |
| **INV-102** | Todo endpoint webhook tem `http.MaxBytesReader(r.Body, 64*1024)` **antes** de qualquer leitura/parse | cross-domain | Middleware `WebhookBodyLimit(64KB)` obrigatório em rotas `/webhooks/*` + CI grep bloqueia `io.ReadAll(c.Request.Body)` em handler webhook sem reader limited | Integration + static check | [[../11-audit/dual-check-log/2026-04-23-security-gaps#A-DC-SEC-002]] / [[../02-architecture/adr/0027-webhook-idempotency-db]] |
| **INV-103** | CORS allowlist é **específica** (sem wildcard subdomain em staging/prod): `app.mastersindico.com.br`, `admin.mastersindico.com.br`; cookie `Domain=app.mastersindico.com.br` (sem ponto-inicial) | cross-domain | `Config.Validate()` falha startup se `CORS_ALLOWED_ORIGINS` contém `*` em staging/prod + teste pre-flight `Origin: evil.mastersindico.com.br` → rejeitado | Integration + startup test | [[../11-audit/dual-check-log/2026-04-23-security-gaps#A-DC-SEC-003]] |
| **INV-104** | Hard-delete LGPD tem soft-flag `users._pending_hard_delete` + janela 48h de grace; todo webhook handler (Stripe/Mux/Livekit) consulta flag antes de UPDATE — se true + user deletado, skip + log em `lgpd_logs` | identity / billing / cross-domain | Schema `users._pending_hard_delete BOOLEAN NOT NULL DEFAULT FALSE` + handler pattern canônico + `AccountDeletionSaga` | Integration (race test) | [[../11-audit/dual-check-log/2026-04-23-security-gaps#A-DC-SEC-004]] / [[../02-architecture/adr/0030-saga-orchestration-default]] |
| **INV-105** | Idempotência de webhook tem **DB UNIQUE `(provider, event_id)`** como fonte-de-verdade; Redis é cache L1 opcional. Crash Redis = zero impacto em correção | cross-domain / billing / content | Schema `webhook_events UNIQUE(provider, event_id)` + handler pattern em [[../02-architecture/adr/0027-webhook-idempotency-db]] | Integration (chaos test Redis down) | [[../11-audit/audit-log/2026-04-23-pentest-specs#A-DC-PEN-016]] / [[../02-architecture/adr/0027-webhook-idempotency-db]] |
| **INV-106** | `session.Regenerate()` obrigatório em **toda** transição de auth-state: login bem-sucedido, MFA completion, step-up, role change, logout → re-login no mesmo device. Cookie antigo é invalidado server-side **antes** de emitir cookie novo | identity | Método `SessionStore.Regenerate(ctx, oldSID) → newSID` + chamada obrigatória nos handlers de transição; teste session fixation automatizado | Integration (session fixation test) | [[../11-audit/audit-log/2026-04-23-pentest-specs#A-DC-PEN-003]] |
| **INV-107** | Endpoints PATCH têm **DTO estrito** que não aceita `role`, `is_admin`, `user_id`, `tenant_id`, `company_id`, `condominium_id` em body. Role change via endpoint dedicado com MFA step-up | identity / institutional / commercial | OpenAPI DTOs tipados + CI grep bloqueia handler PATCH que aceite esses fields em body + PBT fuzz body com fields proibidos → 422 | Integration + static check + PBT | [[../11-audit/audit-log/2026-04-23-pentest-specs#A-DC-PEN-010]] + [[../11-audit/audit-log/2026-04-23-pentest-specs#A-DC-PEN-012]] |
| **INV-108** | `tenant_id` é **type-driven**: VOs distintos `CondominiumID` / `CompanyID` / `UserID` / `NeighborhoodID` em Go, com selador `isTenantID()` impedindo conversion cross-type. Nenhuma assinatura pública aceita `tenant_id string` bare | cross-domain / institutional / commercial | Types em `internal/shared/types/tenant.go` + sqlc overrides + CI grep bloqueia `tenant_id string` em assinaturas de repo/handler fora do package types | Compile-time + static check | [[../11-audit/dual-check-log/2026-04-23-security-gaps#A-DC-SEC-005]] + [[../11-audit/audit-log/2026-04-23-pentest-specs#A-DC-PEN-011]] / [[../02-architecture/adr/0029-tenant-id-typed]] |
| **INV-109** | Pseudonimização LGPD usa **HMAC keyed com secret rotating anual** (não SHA256 determinístico). Purpose scope separado por contexto (`timeline_anon`, `vote_anon`, `audit_anon`). Keys rotacionadas 1/jan; retention 5y, depois hard-delete da key (irreversibilidade forward) | cross-domain / identity | `pkg/lgpd/pseudonymize.go` + CI grep bloqueia `sha256.*user_id` em código de anonimização + teste determinismo (mesmo year+purpose+user → mesmo pseudônimo; year diferente → diferente) | Unit + Integration + static check | [[../11-audit/audit-log/2026-04-23-pentest-specs#A-DC-PEN-038]] / [[../02-architecture/adr/0028-lgpd-keyed-hmac]] |
| **INV-110** | Endpoints auth-críticos exigem **step-up MFA `≤ 5min`** via `X-Step-Up-Token` header: `DELETE /me`, `DELETE /me/devices/:id`, `GET /me/export`, `publish minutes`, `approve contract ≥ R$10k`, `transfer mandate`, `PATCH /me` (email/phone), `POST /admin/v1/*`. Ausência/expirado → 401 `step_up_required` | identity / cross-domain / assembly / commercial | Middleware `RequireStepUp(endpoint)` + Zitadel MFA enforce para admin + teste integration por endpoint | Integration | [[../11-audit/audit-log/2026-04-23-pentest-specs#A-DC-PEN-007]] / [[../02-architecture/adr/0026-admin-mfa-m1]] |

---

## Fase 9 hardening (INV-118 a INV-123) — reanálise research + Q-### / G-### consolidados

Série derivada da reanálise sistemática do research web (11 agentes — beyond-corp, google-arch, netflix, youtube, google-meet, instagram, tiktok, linkedin, condominial-br + 2 focados em quebras/ops) cruzada com quebras de negócio (Q-###) e práticas ops big-tech (G-###) identificadas em `11-audit/audit-log/2026-04-23-CONSOLIDADO-quebras-e-ops.md`.

| ID | Invariante | Owner | Enforcement | Teste | Finding / ADR |
|---|---|---|---|---|---|
| **INV-118** | Edital de assembleia exige **mín 8 dias corridos** de antecedência (Lei 4.591/64 Art. 24 §1). Handler `POST /assemblies` rejeita `scheduled_date - NOW() < 8d` com 422 `edital_insufficient_notice`. Sem override admin | assembly | Handler validator + DB CHECK `scheduled_date >= edital_published_at + INTERVAL '8 days'` + INSERT trigger `edital_publish_min_notice` | Unit + Integration + contract law test | Q-027 / ASM-001 revisado + D-088 |
| **INV-119** | `timeline_entries` e `assembly_minutes` (status `published`/`homologated`) são **imutáveis no DB-level**. `REVOKE UPDATE, DELETE` para `app_user` + trigger `BEFORE UPDATE OR DELETE` com `RAISE EXCEPTION`. INSERT-only pós-publish. Correções = nova TimelineEntry (adendo) | institutional / assembly | Migration com REVOKE + trigger `forbid_mutation()` + role `ms_admin_role` separado com stored proc `assembly.admin_timeline_correction(...)` 4-eyes | Integration (tentar UPDATE como app_user → ERROR P0001) + chaos test | Q-020 / [[../02-architecture/adr/0033-ata-timeline-db-immutability]] + D-088 |
| **INV-120** | `audit_log_entries` tem `REVOKE UPDATE, DELETE` absoluto + trigger `forbid_mutation()` sem exceção. Admin **nunca** edita audit; só INSERT. Retenção 5y aplicada via job de arquivamento particionado (não DELETE) | cross-domain | Migration REVOKE + trigger + partitioning por month → archive S3 post-5y | Integration | Q-020 + INV-090 promovido / ADR-0033 + D-088 |
| **INV-121** | Toda tabela de domínio com coluna `tenant_id` tem `ROW LEVEL SECURITY ENABLED` + policy `tenant_isolation AS RESTRICTIVE`. CI linter `rls-check.sh` bloqueia migration que cria tabela com `tenant_id` sem RLS + policy. Opt-out explícito só para tabelas globais (plans, country_codes) via comment `-- GLOBAL (no RLS by design)` | cross-domain | Migration linter `rls-check.sh` em CI + query `pg_policies` valida presença + policy restrictive | Static check + Integration (query sem `SET app.tenant_id` → 0 rows) | Q-024 + Q-030 + G-034 / [[../02-architecture/adr/0034-postgres-rls-default-on-tenant-guard]] + D-089 |
| **INV-122** | Toda rota HTTP com `{tenant_id}` (ou `{condominium_id}`, `{company_id}`) em path/query DEVE passar por middleware `TenantBoundaryGuard(param)` que compara `path.tenant_id == ctx.tenant_id` ANTES do handler. Mismatch → 403 + audit entry `tenant_boundary_violation`. Rotas globais marcadas explicitamente com tag `// global-route` | cross-domain | Middleware obrigatório + CI grep bloqueia rota com `{tenant_id}` em path sem middleware anexo | Integration (fuzz cross-tenant 500 cenários) + PBT | Q-024 / [[../02-architecture/adr/0034-postgres-rls-default-on-tenant-guard]] + D-089 |
| **INV-123** | Postgres tem **WAL archiving contínuo** via pgBackRest → S3/R2 (full semanal + diff diário + WAL 60s). Backup imutável offline com Object Lock 90d. RPO target **5 min** (0 min para audit). RTO target por BC em tabela canônica ([[../02-architecture/observability]] §3.3). **DR drill mensal obrigatório** ([[../06-execution/dr-drill]]) — 2 drills consecutivos fora do target = Sev2 + postmortem | cross-domain / observability | `postgresql.conf` archive_mode=on + pgBackRest stanza + alert `archive_command fail > 5min` Sev1 + cron DR drill mensal + registro em `11-audit/audit-log/drill-YYYY-MM.md` | Integration (restore em staging-dr) + property RPO_obs ≤ 5min + RTO_obs ≤ target | G-015 + G-016 + G-017 / [[../02-architecture/adr/0035-postgres-pitr-wal-archiving]] + D-090 |

---

## Total: **116 invariantes canônicos** (100 originais + 10 hardening Fase 5 + 6 hardening Fase 9)

Distribuição:

| BC | Count |
|---|---|
| identity | 10 |
| billing | 10 |
| institutional | 15 |
| commercial | 20 |
| content | 15 |
| assembly | 15 |
| cross-domain | 15 |
| security hardening Fase 5 (INV-101..110) | 10 |
| quebras/ops hardening Fase 9 (INV-118..123) | 6 |
| **total** | **116** |

---

## Como cada tipo de enforcement é validado

### PBT (Property-Based Testing — `pgregory.net/rapid`)

Usar quando:
- Há muitas combinações possíveis de input
- Mutações no estado preservam propriedade
- Regras matemáticas (soma, média, conservação)

Aplicações obrigatórias: **INV-016** (quotas, 300 casos), **INV-021** (fração ideal, 100 casos), **INV-086** (ABAC engine, 400 casos), **INV-071** (vote único TOCTOU), **INV-043** (evaluation única TOCTOU).

### Integration (testcontainers-go + PG real)

Usar quando: invariante depende de DB (UNIQUE, FK, CHECK), TOCTOU, cross-module, cross-tenant.

### Unit

Invariante enforceável por construção: VO validation, factory precondition, state transition simples.

### CI static check (grep)

Invariante por análise estática: padrões proibidos (PII em log, `_ = fn()`, `SELECT *`, bloco `else`, UPDATE em timeline).

### Prod monitoring

Além dos testes, invariants verificáveis em runtime:

- **INV-021**: metric `condominium_fracao_sum` — alerta > 100
- **INV-005**: metric `active_sessions_per_user` — alerta > MAX
- **INV-024**: audit log de UPDATE/DELETE em `timeline_entries` → alerta > 0
- **INV-089**: log "cross-tenant access attempt" (ABAC deny `tenant_mismatch`) → dashboard
- **INV-015**: dashboard webhook dedup rate

---

## Histórico de violações (legado)

| INV | Violação | Fix | AUDIT |
|---|---|---|---|
| INV-071 | Vote duplicado (TOCTOU race HasVoted + Create) | UNIQUE DB + ErrConflict em Create | A-025 |
| INV-042 | SupplierVote duplicado (race contador) | UoW + SELECT FOR UPDATE | A-026, A-030 |
| INV-043 | CompanyEvaluation duplicada (TOCTOU) | ErrConflict em Create | A-029 |
| INV-089 | Tenant isolation: 12 queries com `SELECT *` | Colunas explícitas | A-021 |
| INV-074 | Assembly iniciada sem edital | Rota PublishEdital adicionada | A-013 |
| INV-092 | PII em log (CNPJ, invite_token plain) | `Masked()` | A-003, A-004 |
| INV-064 | Mux webhook em `/api/v1` (Authenticate bloqueava) | Rota pública | A-022 |
| INV-065 | Saga UploadVideo sem compensar | `CancelMuxUpload` | A-027 |
| INV-084 | Saga StartLive sem compensar | `EndRoom` | A-028 |
| INV-005 | Race session invalidation | UoW + SELECT FOR UPDATE | A-023 |
| INV-097 | RateLimiter memory leak | Cleanup goroutine | A-032 |

---

## ⚠️ Pendências de dual-check (Fase 5)

- **P-INV-001**: INV-041 (ClosedDeal → TimelineEntry auto-publish) — confirmar se o evento é síncrono (dentro UoW do commercial) ou assíncrono (handler separado em institutional). Legado é síncrono via ports.
- **P-INV-002**: INV-046 (reset quota anual) — confirmar timezone: BRT (São Paulo) ou UTC? Legado não explicita; v2 deve padronizar BRT.
- **P-INV-003**: INV-027 (≤ 15 condomínios) — confirmar se é hard (CHECK trigger) ou soft (validação use case + admin bypass). Legado é hard.
- **P-INV-004**: INV-056 (lock 90d) — admin pode bypass, mas confirmar se é com motivo obrigatório + audit log (legado menciona "só em caso legal").
- **P-INV-005**: INV-091 (LGPD direito ao esquecimento) — confirmar se após vencimento os logs são pseudonimizados ou hard-deleted. Legado é hard-delete após 5 anos.
- **P-INV-006**: INV-038 — Proposta única por (Request, company): e se a empresa envia rascunho e depois envia de verdade? Confirmar se UNIQUE é só em `status != draft`.
- **P-INV-007**: INV-085 (homologação obrigatória) — confirmar se é obrigatória em **toda** assembleia ou só em extraordinária. Legado: toda.

---

## Fase 12 — Invariantes novos (2026-04-24)

### INV-GOV-001 — Score Governança do síndico (1-10)

**Fonte**: [[../STATE#D-103 — Score duplo no perfil do síndico (fecha Q25)|D-103]].

**Enunciado**: Todo síndico tem um valor `governance_score ∈ [1.0, 10.0]` calculado determinísticamente pela fórmula abaixo. O valor é **público** no perfil do síndico (morador, empresa, parceiro podem ver).

**Fórmula** (determinística — pesos **canonizados** via [[../STATE#d-109|D-109]] 2026-04-24; calibrar com dados reais em M2):
- 40% — **Avaliações regulares** (M12 bimestral): média das últimas 6 avaliações (3 critérios × escala 1-10, média simples por critério e ponderada)
- 25% — **Avaliações escondidas em eleição** ([[#INV-ASM-023]]): histórico agregado do mandato
- 15% — **Engajamento com timeline**: % de publicações LT com foto/vídeo anexo; % de atividades PD com status atualizado no prazo
- 10% — **Transparência**: % de registros de execução validados dentro de 7 dias
- 10% — **Resposta a Connect Me**: % de Connect Me recebidos com status atualizado em 48h (não auto-close)

**Cálculo**: disparado por evento (`Vote.Registered`, `Assessment.Submitted`, `TimelineEntry.Published`, etc.) OU recálculo noturno. Cache Redis TTL 1h.

**Obrigatório**: valor exibido sempre arredondado a 1 casa decimal (9.4, não 9.37...).

**Valor default**: 5.0 para síndico recém-empossado (neutro, sem histórico). Após 3 avaliações, fórmula toma controle.

### INV-GOV-002 — Score Compliance do síndico (0-100)

**Fonte**: [[../STATE#D-103 — Score duplo no perfil do síndico (fecha Q25)|D-103]].

**Enunciado**: Todo síndico tem um valor `compliance_score ∈ [0, 100]` calculado determinísticamente pela fórmula abaixo. Tela [[../03-subprojects/web/ui-catalog/compliance/C10]] exibe o valor e breakdown.

**Fórmula** (soma ponderada; cada componente vale N pontos até 100):
- 25 pts — **Declaração anual** (C2) em dia (+25 se current year; -5 por ano atrasado até 0)
- 20 pts — **Assinaturas obrigatórias** (C3): todas 4 assinaturas (síndico/preposto/administradora/conselho) em dia
- 15 pts — **Matriz conflito de interesse** (C4) atualizada no último trimestre
- 15 pts — **Auditoria sem alertas** (C5) nos últimos 90 dias
- 15 pts — **Adendos em dia** (C6): 0 adendos pendentes > 30 dias
- 10 pts — **Onboarding de conformidade do mandato**: todos os 15 marcadores autodeclarados preenchidos

**Gate ABAC**: `compliance_score < 60` bloqueia ações críticas (ex: novas publicações em timeline, encerramento de mandato) até regularizar. Mensagem padrão: "Regularize sua conformidade antes de prosseguir" + link para C1.

**Cálculo**: noturno (cron 03:00) + trigger em eventos de compliance.

**Valor default**: 100 no início do mandato; decai conforme prazos vencem.

### INV-ASM-023 — Eleição gera avaliação escondida obrigatória antes do voto no candidato

**Fonte**: [[../STATE#D-104 — Avaliação escondida de gestão em eleição ATIVADA (fecha Q40)|D-104]]; `ARQUITETURA FUNCIONAL DO MÓDULO ASSEMBLEIA.pdf` §4.4.

**Enunciado**: Em assembleia com pauta tipo `eleicao_sindico`, para cada morador participante, o sistema **bloqueia** submissão do voto no candidato até que o morador tenha **submetido uma avaliação escondida de gestão** do síndico atual para aquele mandato.

**Propriedades da avaliação escondida**:
- **Confidencialidade**: armazenada em tabela separada `hidden_assessments` com colunas `(mandate_id, morador_hash, answers, submitted_at)`. `morador_hash` é HMAC-keyed por `(mandate_id, user_id)` (mesmo padrão LGPD [[../02-architecture/adr/0028-lgpd-keyed-hmac]]).
- **Síndico atual NUNCA vê linhas individuais**; só agregações mensais publicadas em `governance_score` ([[#INV-GOV-001]]) ao fim do mandato.
- **Morador não vê próprio score** após submeter; UI mostra apenas "Obrigado; sua avaliação foi registrada".
- **Avaliação só é criada quando há pauta `eleicao_sindico`**; não é avaliação regular (bimestral, essa é [[../03-subprojects/web/ui-catalog/morador/M12]]).
- **Idempotência**: morador só pode submeter 1 avaliação por eleição; submissões subsequentes retornam `ErrBusiness("já avaliou esta eleição")`.

**Regra de bloqueio** (backend):
- `POST /api/v1/assembly/:id/votes` em item de pauta tipo `eleicao_sindico` verifica `hidden_assessments WHERE mandate_id = X AND morador_hash = HMAC(X, current_user_id)`.
- Se não existe → `ErrPreconditionFailed("avaliação escondida obrigatória antes do voto")` com link para submeter.
- Após submeter avaliação, voto liberado.

**UX**: tela [[../03-subprojects/web/ui-catalog/assembleia/ASM-AVAL-ELEICAO]] + mobile equivalente. Aviso visível: "Esta avaliação é confidencial — não vai para o síndico atual." NavGuard bloqueia escape.

**Escopo**: spec M1 (canônico agora); implementação M3 (junto com Livekit assembleia live).

## Links

- [[_moc]]
- [[bounded-contexts]]
- [[context-map]]
- [[business-rules]]
- [[domain-events]]
- [[state-machines]]
- [[aggregates/_moc]]
- [[../09-testing/_moc]] (a destilar)
- [[../11-audit/AUDIT]] (findings legados A-###)
