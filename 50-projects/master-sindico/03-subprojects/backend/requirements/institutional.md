---
title: Institutional — Backend Requirements (Go)
type: requirement
tags: [master-sindico, backend, go, institutional, condominium, timeline, master-plan, feature-req]
project: master-sindico
stack: backend
bounded_context: institutional
milestone: m1
status: partial
created: 2026-04-24
---

# Institutional — Backend Requirements (Go)

## Escopo do bounded context

Módulo Go `internal/modules/institutional/` (BC mais denso — 11 entities) responsável por:

- **Condominium** como tenant (`tenant_type='condominium'`), 8-char alphanumeric ID, ViaCEP validation.
- **Unit** com `fracao_ideal NUMERIC(5,4)` + `unit_type` enum (⚠️ bloqueador M1: colunas pendentes migration).
- **Membership** titular + dependentes + co-admins síndico (limites: 1 principal + 2 aux + 3 assist + 1 titular + 2 dep).
- **TimelineEntry append-only** (ADR-0033 immutability via REVOKE UPDATE/DELETE + DB trigger).
- **MasterPlanActivity** + 7 status state machine + `BatchAdvance` (A-020 fix N+1).
- **Event, Announcement, Poll, ComplianceDeclaration, GovernanceScore, RiskMap, ReserveFund** (11 entities total).
- **InMemoryPublisher** atual → NATS JetStream threshold (XD-001).
- `ITimelinePublisher` exposto a commercial (deal.signed) e assembly (assembly.closed).
- `ICondominiumIDsProvider` exposto a commercial/assembly (scope resolution).

**Não é responsabilidade**: propostas/deals (commercial); votação formal de fornecedor (assembly/commercial cooperam); vídeos de comunicados (content).

## Status atual (vault canônico)

- **Sprints 1 + 4 + 6 legado**: condomínio base, membership, timeline immutable (ADR-0033), master plan 7 status, announcement, event CRUD.
- **A-###** fechados: A-020 (BatchAdvance fix N+1), A-021 (timeline REVOKE trigger).
- **A-### abertos**:
  - **A-022 aberto**: `unit_type + fracao_ideal` columns migration — **blocker M1** para assembly e supplier vote.
  - **DT-010 aberto**: `IAuditLogger` cross-BC; institutional chama audit local shim.
  - **A-031 aberto**: `ITimelinePublisher` interface + cross-BC contract test.
- **D-###**: ADR-0033 (timeline immutability via REVOKE+trigger), ADR-0034 (RLS default-on), D-051 (governance score motor), D-080 (planos globais afetam limite condomínios).

## Agregados (domain)

- [[../../../01-domain/aggregates/Condominium]] — tenant; ID alfanumérico 8 chars; `mandate` inicial.
- [[../../../01-domain/aggregates/Unit]] — `fracao_ideal`, `unit_type` enum, soma ≤ 1.0000 por condomínio.
- [[../../../01-domain/aggregates/Membership]] — compartilhado c/ identity; institutional valida invariantes de limite.
- [[../../../01-domain/aggregates/TimelineEntry]] — 7 tipos; INSERT-only; `signature_hash` HMAC chain futuro (M3+).
- [[../../../01-domain/aggregates/MasterPlan]] — 26 áreas default; activities com 7 status.
- [[../../../01-domain/aggregates/Announcement]] — 8 tipos × 4 prioridades; imutável pós-publish.
- [[../../../01-domain/aggregates/Event]] — 13 tipos; confirmações; ciência obrigatória.
- [[../../../01-domain/aggregates/Poll]] — 3 tipos (sim_nao_nao_sei, multiple_choice, scale_1_a_5); anônima.
- `ComplianceDeclaration`, `GovernanceScoreHistory`, `RiskMap`, `ReserveFundContribution`.
- `Mandate` — histórico de gestão; encerramento → dossiê PDF.

## Entidades e Value Objects

| Tipo | Elemento | Invariante chave |
|---|---|---|
| Entity | `Condominium` | ID alfanumérico UNIQUE; hard limit 15 por síndico |
| Entity | `Unit` | `unit_identifier` UNIQUE por condo; Σ fracao ≤ 1.0000 (trigger) |
| Entity | `TimelineEntry` | INSERT-only (REVOKE + trigger); chain via `previous_entry_id` |
| Entity | `MasterPlanActivity` | status state machine; mudança APENAS via timeline event |
| Entity | `Announcement` | imutável pós `published_at` NOT NULL |
| VO | `FracaoIdeal` | `NUMERIC(5,4)`, `0 < x ≤ 1.0000` |
| VO | `CondominiumID` | 8 chars alphanumeric `[A-Z0-9]{8}` |
| VO | `UnitType` | enum `{apartamento, casa, sala_comercial, loja, garagem, outro}` |
| VO | `MandateType` | enum `{sindico_eleito, sindico_profissional, sindico_interino}` |

## Invariantes (INV-INS-###)

1. **INV-INS-001**: Timeline INSERT-only — REVOKE UPDATE/DELETE + trigger `prevent_mutation_timeline`.
2. **INV-INS-002**: Master Plan status muda APENAS via timeline event (INS-013).
3. **INV-INS-003**: Condomínio hard limit 15 por conta síndico (tier≥plus); tier=base → 1.
4. **INV-INS-004**: Síndico no condomínio: 1 principal + 2 auxiliares + 3 assistentes.
5. **INV-INS-005**: Fração ideal `NUMERIC(5,4)`, Σ ≤ 1.0000.
6. **INV-INS-006**: Mandato `end_date > start_date`.
7. **INV-INS-007**: Unidade: 1 titular + máx 2 dependentes.
8. **INV-INS-008**: Comunicado imutável pós-publish.
9. **INV-INS-009**: Adendo vinculado via `previous_entry_id` (chain).
10. **INV-INS-010** (INV-107): PATCH memberships DTO estrito; sem `user_id|condominium_id` em body.

## Eventos de domínio

Publica (via InMemoryPublisher atual → NATS threshold):

- `institutional.condominium.created` → analytics + billing verifica quota.
- `institutional.timeline.entry_created {type, condominium_id, entry_id}` → listener interno `master_plan.sync` (INS-013); assembly/commercial podem consumir.
- `institutional.master_plan.activity_status_changed`.
- `institutional.announcement.published` → cria timeline entry automaticamente.
- `institutional.announcement.read_tracked` — agregado batch.
- `institutional.event.confirmed|acknowledged`.
- `institutional.poll.closed` → timeline entry `resultado_consulta_moradores`.
- `institutional.mandate.transferred|ended` → commercial (reset governance score).
- `institutional.membership.co_admin_invited|accepted|revoked`.
- `institutional.compliance.declaration.submitted`.
- `institutional.governance_score.computed` — daily.

Consome:

- `commercial.deal.signed_complete` → via `ITimelinePublisher.Publish(type=registro_de_execucao)`.
- `assembly.closed` → via `ITimelinePublisher.Publish(type=assembly_result)`.
- `billing.subscription.changed` → re-avalia quota condomínios (hard limit 15).
- `identity.user.banned` → `IUserModerator` retro-chain: membership `active=false`.

## Requirements funcionais (FR-BE-INS-###)

- **FR-BE-INS-001** (INS-001) — *Quando* `POST /condominiums`, *o handler deve* (UoW única): (a) validar CEP via `ICEPLookup`, (b) gerar ID alfanumérico UNIQUE via retry loop, (c) INSERT condominium + mandate + membership(role_type=principal), (d) `IQuotaCheck.Check(syndic_id, "institutional.condominium")` ≥ limit → rollback `403 QUOTA_EXCEEDED`.
- **FR-BE-INS-002** (INS-005 + A-022) — *Quando* `POST /condominiums/:id/units`, *o handler deve* criar Unit com `fracao_ideal NUMERIC(5,4)` + `unit_type` enum; trigger DB valida Σ `fracao_ideal` ≤ 1.0000 (WHERE active); violação → `22000 check_violation` → mapeado para `422 validation_error code=fraction_sum_exceeded`.
- **FR-BE-INS-003** (INS-007 + INV-107) — *Quando* `PATCH /api/v1/memberships/:id`, *o DTO `UpdateMembershipRequest {role_type?, active?, metadata?}`* aceita apenas esses campos; `user_id|condominium_id|tenant_id|company_id|role|is_admin` em body → `422`.
- **FR-BE-INS-004** (INS-011 + INS-012) — *Quando* entrada Timeline é criada, *o handler deve* INSERT em `timeline_entries` (UoW); tentativa `UPDATE/DELETE` → DB trigger raises `exception` (ADR-0033).
- **FR-BE-INS-005** (INS-013) — *Quando* entry `type ∈ {atividade_da_gestao, registro_de_execucao}` é inserida (mesma UoW), *o backend deve* publicar `institutional.timeline.entry_created` via `IEventBus`; listener `master_plan.sync` aplica via **outbox pattern** (XD-002) garantindo entrega at-least-once.
- **FR-BE-INS-006** (INS-014) — *Quando* `GET /condominiums/:id/timeline?cursor&type&from&to`, *o handler deve* validar membership ativo do caller; non-member → `403`; paginação cursor-based (GR-016).
- **FR-BE-INS-007** (INS-016) — *Quando* adendo criado, *o backend deve* vincular `previous_entry_id` FK `timeline_entries`; `GET /timeline_entries/:id/chain` retorna árvore.
- **FR-BE-INS-008** (INS-017 + INS-018) — *Quando* master plan é inicializado, *o seed deve* popular 26 áreas default em `master_plan_areas`; `MasterPlanActivity.TransitionTo` valida state machine `planejada→em_contratacao→em_execucao→concluida + suspensa|reprogramada|atrasada`.
- **FR-BE-INS-009** (A-020) — *Quando* batch advance master plan activities é requisitado, *o handler `BatchAdvance(condominium_id, activity_ids[])` deve* executar em **uma query** (CTE) sem N+1; PBT: 1000 activities → ≤ 2 queries totais (SELECT + UPDATE).
- **FR-BE-INS-010** (INS-020) — *Quando* `GET /condominiums/:id/master-plan/export?format=pdf`, *o backend deve* validar ABAC `institutional.master_plan.export_pdf` (feature flag plan.features); > 100 activities → job async + email presigned URL.
- **FR-BE-INS-011** (INS-021 + INS-022) — *Quando* `POST /announcements`, *o handler deve* criar draft; `POST /announcements/:id/publish` → set `published_at=NOW()`, lock, auto-criar timeline entry `type=comunicado` (mesma UoW).
- **FR-BE-INS-012** (INS-023) — *Quando* morador visualiza announcement, *o handler deve* INSERT `communication_reads` ON CONFLICT DO NOTHING (idempotente); agregação `GET /announcements/:id/read-stats` retorna `%`.
- **FR-BE-INS-013** (INS-024) — *Quando* cron mensal roda, *o job deve* mover announcements > 1 ano para `announcements_archive` (soft move preservando ID).
- **FR-BE-INS-014** (INS-025 + INS-026 + INS-027 + INS-028) — *Quando* event é criado, *o handler deve* aceitar enum 13 tipos; `POST /events/:id/confirmations {sim|nao|talvez}` idempotente UNIQUE `(user_id, event_id)`; `POST /events/:id/acknowledgments` com tracking 6 meses; scheduler agenda notificações T-3d/T-1d/T-3h via `IPushProvider` + `IEmailProvider`.
- **FR-BE-INS-015** (INS-029 + INS-030) — *Quando* `POST /polls`, *o handler deve* aceitar 3 tipos; `POST /polls/:id/responses` anônima (síndico vê `count`, nunca `user_id`); `POST /polls/:id/close` → publica timeline entry `resultado_consulta_moradores`.
- **FR-BE-INS-016** (INS-031) — *Quando* passa 61d desde última `governance_assessments`, *o cron deve* abrir nova (3 perguntas escala 1-10); atrasada → bloqueia export dossiê + certas atividades (ABAC consulta flag `assessment_overdue`).
- **FR-BE-INS-017** (INS-032 + XD-011 + D-051) — *Quando* cron noturno roda, *o job `CalcGovernanceScore` deve* calcular 0-100 com 10 componentes; motor ainda **parcial** (D-051 aberta) — componentes LGPD, assinaturas, reclamações, fundo reserva, assembleias, rotinas, avaliação registrados; falta formulação final.
- **FR-BE-INS-018** (INS-033) — *Quando* aniversário anual do mandato, *o cron deve* solicitar `compliance_declarations` (checklist); atrasada → bloqueia export dossiê.
- **FR-BE-INS-019** (INS-034) — *Quando* `mandates.ended_at` setado, *o job async `BuildDossier` deve* consolidar (timeline + atas + relatórios) → PDF via wkhtmltopdf/Puppeteer → `IStorageProvider` S3; signed URL pro síndico encerrado.
- **FR-BE-INS-020** (INS-037) — *Quando* `GET /condominiums/:id/public`, *o handler deve* retornar apenas dados públicos (nome, endereço parcial, governance_score, total mandatos); sem PII moradores; autenticado (não anon).
- **FR-BE-INS-021** (INS-039 + INS-040) — *Quando* syndic completa onboarding, *o handler deve* persistir `SyndicProfile` com foto mín 400×400 via `IStorageProvider` + `governance_markers` checkbox list (15 autodeclarados com disclaimer fixo).
- **FR-BE-INS-022** (INS-041) — *Quando* morador completa onboarding, *o handler deve* persistir `ResidentProfile` + validar CPF algorithm; `cpf` imutável (IDN-029 cross-BC).
- **FR-BE-INS-023** (INS-042) — *Quando* `GET /condominiums/search?q`, *o backend deve* usar `ISearchProvider` real (D-120 Fase 14 revoga D-067 PG tsvector; ADR-0042 `proposed pending dual-check` decide OpenSearch vs Meilisearch); busca por ID alfanumérico + nome + CEP. Implementação tsvector vigente é **stub temporário** até search real entrar.
- **FR-BE-INS-024** (INS-043) — *Quando* Base morador acessa perfil empresa, *o handler deve* servir preview limitado (2 fotos + 1 vídeo 60s via CNT-008 paywall); Plus/Pro → full.
- **FR-BE-INS-025** (INS-002) — *Quando* `POST /condominiums/:id/transfer-mandate`, *o handler deve* exigir `reason`, preservar timeline read-only, resetar `governance_score`; audit trail completo.
- **FR-BE-INS-026** (INS-003) — *Quando* síndico emite encerramento, *o handler deve* validar `compliance=em_dia`; bloqueio se atrasada; trigger `BuildDossier` job.
- **FR-BE-INS-027** (INS-010 + matriz) — *Quando* co-admin convite aceita, *o ABAC deve* aplicar matriz `role_type ∈ {principal, auxiliary, assistant}` → actions permitidas (criar atividade timeline, validar execução, etc.); principal única pode transferir mandato.
- **FR-BE-INS-028** (INS-008) — *Quando* encerramento membership morador, *o handler deve* exigir `reason`, SET `revoked_at=NOW()`, trigger logout via `IUserModerator.RevokeSessions(user_id)`; reativação possível.
- **FR-BE-INS-029** — `ITimelinePublisher` interface (A-031) exposta em `internal/shared/types`: `Publish(ctx, TimelineEntry) error`; commercial e assembly chamam para cross-BC sem import.
- **FR-BE-INS-030** — `ICondominiumIDsProvider` exposto: `ListByUser(ctx, user_id) []CondominiumID`; commercial usa para filtrar deals, assembly para escopo de voting.
- **FR-BE-INS-031** (INS-035) — *Quando* síndico registra contribuição fundo reserva, *o handler deve* persistir `reserve_fund_contributions {amount_cents, contributed_at}`; governance_score componente 6 consome.
- **FR-BE-INS-032** (INS-036) — *Quando* síndico atualiza risk map, *o handler deve* aceitar 5 tipos risco (financeiro, compliance, estrutural, social, segurança) vinculando `master_plan_activities`.
- **FR-BE-INS-033** — RLS default-on (ADR-0034): `units`, `memberships`, `timeline_entries`, `announcements`, `events`, `polls` com policy `USING (condominium_id = ANY(current_setting('app.user_condominium_ids')::uuid[]))`.
- **FR-BE-INS-034** (INS-015) — *Quando* `GET /condominiums/:id/timeline/verify` (M3+), *o handler deve* computar `signature_hash = HMAC(prev_hash + entry_data, secret)` pra cada entry e retornar chain válida/inválida.
- **FR-BE-INS-035** — `IAuditLogger` (DT-010) — toda mutation institutional chama via shim local; extração pendente.
- **FR-BE-INS-036** (INS-006) — *Quando* unidade criada, *o handler deve* exigir upload de foto (`IStorageProvider`, 5MB max, JPG/PNG/WebP) + aceite termo LGPD versionado (GR-023).
- **FR-BE-INS-037** (INS-038) — *Quando* condomínio tem ≥1 assembleia homologada, *o `GET /condominiums/:id/transparency` deve* retornar badge `{score, history_12_months, benchmark}` (acima/média/abaixo percentil).
- **FR-BE-INS-038** (INS-009) — *Quando* síndico com > 1 condomínio, *o `GET /me/condominiums` deve* retornar lista; troca contexto invalida ABAC cache local (IDN-014).

## Endpoints REST expostos

| Método | Path | Auth | Descrição | Status |
|---|---|---|---|---|
| POST | /api/v1/condominiums | Bearer + ABAC(syndic) | Criar condomínio | ✅ |
| GET | /api/v1/condominiums/:id | Bearer + ABAC(membership) | Detalhe | ✅ |
| PATCH | /api/v1/condominiums/:id | Bearer + ABAC(principal) | Update (strict DTO) | ✅ |
| POST | /api/v1/condominiums/:id/transfer-mandate | Bearer + step-up | INS-002 | 🟡 |
| POST | /api/v1/condominiums/:id/close-mandate | Bearer + step-up | INS-003 | 🟡 |
| GET | /api/v1/condominiums/search | Bearer | `ISearchProvider` (stub tsvector M1; OpenSearch/Meili pós-ADR-0042) | 🟡 |
| GET | /api/v1/condominiums/:id/public | Bearer | Perfil público | ✅ |
| GET | /api/v1/condominiums/:id/transparency | Bearer | Badge INS-038 | 🟡 |
| POST | /api/v1/condominiums/:id/units | Bearer + ABAC | Criar unidade | ⚠️ blocker A-022 |
| GET | /api/v1/condominiums/:id/units | Bearer | Lista | ✅ |
| POST | /api/v1/condominiums/:id/timeline/:type | Bearer + ABAC | Criar entry | ✅ |
| GET | /api/v1/condominiums/:id/timeline | Bearer + cursor | Feed imutável | ✅ |
| GET | /api/v1/condominiums/:id/timeline/:entry_id/chain | Bearer | Adendos | ✅ |
| GET | /api/v1/condominiums/:id/master-plan | Bearer | Activities + áreas | ✅ |
| POST | /api/v1/condominiums/:id/master-plan/batch-advance | Bearer + ABAC | A-020 fix | ✅ |
| GET | /api/v1/condominiums/:id/master-plan/export?format=pdf | Bearer + plan.features | Async job | 🟡 |
| POST | /api/v1/announcements | Bearer + ABAC | Draft | ✅ |
| POST | /api/v1/announcements/:id/publish | Bearer + ABAC | Lock + timeline | ✅ |
| GET | /api/v1/announcements/:id/read-stats | Bearer | % leitura | ✅ |
| POST | /api/v1/events | Bearer + ABAC | 13 tipos | ✅ |
| POST | /api/v1/events/:id/confirmations | Bearer | sim/nao/talvez | ✅ |
| POST | /api/v1/events/:id/acknowledgments | Bearer | Ciência tracking 6m | ✅ |
| POST | /api/v1/polls | Bearer + ABAC | 3 tipos | ✅ |
| POST | /api/v1/polls/:id/responses | Bearer | Anônima | ✅ |
| POST | /api/v1/polls/:id/close | Bearer + ABAC | Fecha + timeline | ✅ |
| GET | /api/v1/governance-assessments | Bearer | Lista | ✅ |
| POST | /api/v1/governance-assessments/:id/respond | Bearer (anon) | Resposta | ✅ |
| POST | /api/v1/compliance-declarations | Bearer | Checklist anual | 🟡 |
| GET | /api/v1/condominiums/:id/governance-score | Bearer | Score + breakdown | 🟡 D-051 |
| GET | /api/v1/condominiums/:id/dossier | Bearer + step-up | Download PDF | 🟡 |
| POST | /api/v1/risk-maps | Bearer + ABAC | Risk map | ⏳ |
| POST | /api/v1/reserve-fund-contributions | Bearer + ABAC | Input manual | ⏳ |

## Integração com outros BCs

**Expõe** (via `internal/shared/types`):

- `ITimelinePublisher` (A-031) — `Publish(ctx, TimelineEntry) error`.
- `ICondominiumIDsProvider` — `ListByUser(ctx, user_id) []CondominiumID`, `GetMembershipType(ctx, user_id, cond_id) Role+RoleType`.
- `IValidationSubmitter` (commercial uses) — `SubmitExecutionRecord` via institutional-side mapping to timeline.
- `IGovernanceScoreReader` — `Read(ctx, cond_id) ScoreSnapshot`.

**Consome**:

- `IQuotaCheck` (billing) — hard limit condomínios.
- `IQuotaConsumer` (billing) — consumo Connect Me (via institutional-side handler de `/me/connect-me` proxied).
- `IEventBus`, `IAuditLogger`, `IStorageProvider`, `IEmailProvider`, `IPushProvider`, `ICEPLookup`.
- `IUserModerator` (identity) — revocation em membership encerramento.

**Publica eventos**: `institutional.condominium.*`, `institutional.timeline.*`, `institutional.master_plan.*`, `institutional.announcement.*`, `institutional.event.*`, `institutional.poll.*`, `institutional.mandate.*`, `institutional.compliance.*`, `institutional.governance_score.*`.

**Consome eventos**: `commercial.deal.signed_complete`, `assembly.closed`, `billing.subscription.changed`, `identity.user.banned`.

## Persistência

**Tabelas principais**:

- `condominiums` — ID alfanumérico 8 chars; JSONB `governance_config`.
- `units` — `fracao_ideal NUMERIC(5,4) CHECK (> 0 AND <= 1.0000)`; UNIQUE `(condominium_id, unit_identifier)`; trigger Σ ≤ 1.0000.
- `memberships` — (mesma do identity, cross-BC schema) UNIQUE constraints de limite.
- `mandates` — histórico; `ended_at` triggera dossier.
- `timeline_entries` — INSERT-only; REVOKE UPDATE/DELETE no role app; trigger `prevent_mutation_timeline`.
- `master_plan_areas` (seed 26), `master_plan_activities`, `master_plan_risks`, `master_plan_next_actions`.
- `announcements`, `communication_reads`, `announcements_archive`.
- `events`, `event_confirmations`, `event_acknowledgments`.
- `polls`, `poll_responses` (anon, só `response` + `created_at`).
- `governance_assessments`, `governance_score_history`.
- `compliance_declarations` (retenção 5 anos).
- `risk_maps`, `risk_items`.
- `reserve_fund_contributions`.

**Migrations**: `00300..00499` institutional.

**RLS default-on** (ADR-0034) para tabelas com `condominium_id`.

## Providers externos

- **ViaCEP** (`ICEPLookup` — XD-018) — validação CEP; cache Redis TTL 30d.
- **Resend/SES** (`IEmailProvider`) — notificações event/announcement/mandate transfer.
- **FCM** (`IPushProvider`) — push urgente (announcement prioridade `urgente`).
- **Storage S3/R2** (`IStorageProvider`) — fotos unidade, dossiê PDF, anexos.
- **wkhtmltopdf/Puppeteer** (isolado em `infrastructure/providers/pdf/`) — dossiê, export master plan.

**Sagas**:

- `MandateClosureSaga` — compliance check → dossier build → S3 upload → email syndic → mark `ended_at`.
- `CoAdminInviteSaga` — gerar token → enviar email → aguardar aceite → criar membership → notificar principal.

## Testes

**Coverage target**: Domain 90%, Application 80%, Infra 60%.

**PBT**:

- Fração ideal: `∀ add/remove units → Σ ≤ 1.0000` invariante preservado.
- Timeline immutability: `∀ UPDATE|DELETE attempt → exception`.
- BatchAdvance: `∀ N activities → ≤ 2 queries` (A-020 regression test).
- Membership caps: `∀ 4th auxiliary → 422`; `∀ 3rd dependent → 422`.
- Announcement lock: `published_at != NULL → ∀ UPDATE core fields → rejected`.

**Integration** (testcontainers):

- Timeline → Master Plan sync via outbox (INS-013).
- `GovernanceScoreHistory` computation with 12 months fixture.
- `IPushProvider` event T-3d/T-1d/T-3h cron with mock clock.

## Segurança específica do BC

**LGPD**:

- `compliance_declarations` retenção 5 anos.
- `polls` são anônimas — no `user_id` persistido no response; apenas `poll_id + response + created_at`.
- Audit trail em todas mutations (IAuditLogger — DT-010 aberta).
- `dossier` pós-mandato contém dados pessoais → signed URL TTL curta + audit acesso.

**ABAC actions canônicos**:

- `institutional.timeline.create`, `institutional.timeline.read`.
- `institutional.announcement.create`, `institutional.announcement.publish`.
- `institutional.condominium.create`, `institutional.condominium.edit.core`, `institutional.condominium.transfer_mandate`.
- `institutional.master_plan.export_pdf`, `institutional.master_plan.edit`.
- `institutional.membership.invite.co_admin`.
- `institutional.poll.create`, `institutional.event.create`.

**Rate limits**:

- `POST /condominiums` — 5/h por syndic.
- `POST /announcements` — 10/h por condomínio.
- `POST /events` — 20/h por condomínio.

## Decisões e dívidas

**D-###**:

- ADR-0033 (timeline immutability via REVOKE + trigger) — fechado.
- ADR-0034 (RLS default-on) — fechado.
- D-051 (governance score motor) — **aberta**; componentes parciais, formulação final pendente.
- D-067 (OpenSearch → Postgres tsvector) — **PARCIAL-REVOGADA** por D-120 Fase 14 (search real obrigatório M1 — tsvector vira stub temporário).
- D-080 (planos globais afetam hard limit condomínios) — fechado.

**DT-###**:

- **A-022 aberto** — `unit_type + fracao_ideal` columns + trigger → **blocker M1**.
- **A-031 aberto** — `ITimelinePublisher` interface exposta + contract test cross-BC.
- **DT-010 aberta** — `IAuditLogger` extraction.

## Gaps e bloqueadores Sprint 10 (M1)

1. **⚠️ BLOCKER M1**: `unit_type` + `fracao_ideal` migration + trigger Σ ≤ 1.0000 (A-022).
2. **Governance score motor** (INS-032 / D-051) — componentes registram mas score final não calcula em todos os cenários.
3. **`ITimelinePublisher` interface** (A-031) — commercial/assembly atualmente chamam função concreta (cross-BC import potencial).
4. **Dossiê PDF job** (INS-034) — 🟡 template básico; jurídico review pendente.
5. **Export master plan PDF** (INS-020) — 🟡 endpoint mas async job não configurado.
6. **Transparency badge** (INS-038) — 🟡 endpoint retorna dados mas benchmark percentil não computado.
7. **`compliance_declarations` anual** (INS-033) — 🟡 tabela existe; cron de lembrete pendente.

## Ligações

- [[../../../04-requirements/functional/institutional]] — requirement canônico global
- [[../../../01-domain/aggregates/Condominium]]
- [[../../../01-domain/aggregates/Unit]]
- [[../../../01-domain/aggregates/TimelineEntry]]
- [[../../../01-domain/aggregates/MasterPlan]]
- [[../../../01-domain/aggregates/_moc]]
- [[../../../02-architecture/adr/0033-ata-timeline-db-immutability]]
- [[../../../02-architecture/adr/0034-postgres-rls-default-on-tenant-guard]]
- [[../../../11-audit/AUDIT]]
- [[../architecture]]
- [[../patterns]] — Outbox pattern (INS-013), UoW, Repository
- [[../security]]
- [[../tasks]] — Sprint 1, 4, 6
- Correspondente web: [[../../web/requirements/cms]]
- Correspondente mobile: a definir

## Links
- [[../../../_moc]]
- [[../../../CLAUDE]]
