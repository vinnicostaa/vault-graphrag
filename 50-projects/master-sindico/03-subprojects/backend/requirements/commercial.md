---
title: Commercial — Backend Requirements (Go)
type: requirement
tags: [master-sindico, backend, go, commercial, connect-me, reputation, vizinhanca, feature-req]
project: master-sindico
stack: backend
bounded_context: commercial
milestone: m2
status: in-progress
created: 2026-04-24
---

# Commercial — Backend Requirements (Go)

## Escopo do bounded context

Módulo Go `internal/modules/commercial/` responsável por:

- **Company** (tenant_type=company) + `CNPJ VO` + compliance markers (25 autodeclarados) + status lifecycle.
- **ConnectMeRequest** em 4 direções (S→E, M→S, E→E, S→P) com unidirectional enforcement.
- **Proposal** lifecycle imutável pós `sent`; versionamento via adendo.
- **ClosedDeal** (dupla assinatura imutável) + auto-publica `registro_de_execucao` via `ITimelinePublisher`.
- **ExecutionRecord** + validação obrigatória síndico + fotos Mux/S3.
- **HarassmentReport** + admin actions (warning/suspension/ban via `IUserModerator`).
- **SupplierVoteSession** com `FOR UPDATE` row lock (A-030 fix) + fração ideal weight.
- **EmpresaSindicaLink** (feature flag M5+).
- **CompanyEvaluation** (5 critérios, anônima) + idempotency (A-029 fix).
- **PostDealComunicado** (5 tipos).
- **Partner (Vizinhança)** + cupons com 4h cooldown (D-059 ATIVO; D-078/D-064 legado).
- **Reputation motor** Bronze→Diamante (função pura) — **D-051 aberta** (parcial).

**Não é responsabilidade**: upload de vídeo (content); quota Connect Me (billing); timeline entry (institutional via `ITimelinePublisher`); assembly formal voting (assembly).

## Status atual (vault canônico)

- **Sprints 6-7 legado**: Company, ConnectMe (4 fluxos), Proposal, Deal base, Evaluation.
- **A-###** fechados: A-025 (vote UNIQUE DB), A-027 (`CancelUpload` saga content), A-029 (`CompanyEvaluation` idempotency), A-030 (`SupplierVoteSession FOR UPDATE` fix).
- **A-### abertos**:
  - **D-051 aberta**: Reputation motor — thresholds registrados mas cálculo final e degradação parciais.
  - **A-033 aberto**: Partner + coupon full flow (M2 lockdown).
  - **A-032 aberto**: `EmpresaSindicaLink` M5+ (backlog).
- **D-###**: D-010 (tier thresholds), D-059 (cupons ATIVO Fase 8), D-078 (cupons 4h lock), D-064 (legado), D-073 (Agência sub-produto).

## Agregados (domain)

- [[../../../01-domain/aggregates/Company]] — tenant; CNPJ UNIQUE; `tier_reputation` calculated.
- [[../../../01-domain/aggregates/ConnectMeRequest]] — 4 direções; unidirectional; auto-close 48h.
- [[../../../01-domain/aggregates/Proposal]] — imutável pós sent; versionamento `previous_proposal_id`.
- [[../../../01-domain/aggregates/ClosedDeal]] — dupla assinatura; `DL-YYYYMMDD-NNN` auto-gen.
- [[../../../01-domain/aggregates/CompanyEvaluation]] — 5 critérios 1-10; anônima.
- [[../../../01-domain/aggregates/HarassmentReport]] — context preserved.
- `SupplierVoteSession` — FOR UPDATE row lock (A-030); UNIQUE `(session, voter)`.
- [[../../../01-domain/aggregates/Partner]] — Vizinhança; `palavra_do_dia`; seal síndico.
- `CouponGeneration` — lookup 4h cooldown; formato `ID_LOJA(5)SEQ(3)PALAVRA(5)`.
- `ExecutionRecord` — validação obrigatória; fotos+vídeos.
- `TechnicalActivity` — sem deal; contribui master plan.
- `Amendment` — chain de modificações (INS-016 integração).

## Entidades e Value Objects

| Tipo | Elemento | Invariante chave |
|---|---|---|
| Entity | `Company` | CNPJ UNIQUE; status `{pending_verification → active → suspended → blocked}` |
| Entity | `ConnectMeRequest` | Unidirectional — sem thread/reply; dados ocultos até "Tenho Interesse" |
| Entity | `Proposal` | Imutável pós `sent`; amendment via `amendments` |
| Entity | `ClosedDeal` | Dupla assinatura; append-only |
| Entity | `SupplierVoteSession` | FOR UPDATE lock (A-030) prevenindo race |
| VO | `CNPJ` | 14 dígitos + algoritmo verificador; UNIQUE |
| VO | `ReputationTier` | enum `{bronze, prata, ouro, platina, diamante}` |
| VO | `DealNumber` | `DL-YYYYMMDD-NNN` auto-gen sequential per day |
| VO | `CouponCode` | `{ID_LOJA(5)}{SEQ(3)}{PALAVRA(5)}` ex: `00123055PAO` |
| VO | `PalavraDoDia` | `[A-Z]{1,5}`, reset 00:00 SP |
| VO | `BenefitType` | 7 tipos (desconto_percentual, desconto_fixo, etc.) |

## Invariantes (INV-COM-###)

1. **INV-COM-001**: Connect Me é unidirectional — nunca thread/reply.
2. **INV-COM-002**: Proposals imutáveis após `sent` — único mecanismo de mudança: amendment.
3. **INV-COM-003**: Deal imutável após dupla assinatura.
4. **INV-COM-004**: Execução de serviço exige validação obrigatória síndico.
5. **INV-COM-005**: Avaliações são anônimas ao avaliado.
6. **INV-COM-006**: Reputação é função pura (GR-028) — inputs → mesmo tier.
7. **INV-COM-007**: Fração ideal em supplier vote usa NUMERIC(5,4).
8. **INV-COM-008**: Vote único por `(session, voter)` — UNIQUE DB (A-025).
9. **INV-COM-009**: Agência tem acesso ZERO a dados de negócio (IDN-025).
10. **INV-COM-010**: Cupom cooldown 4h por `(morador, parceiro)` (D-059).

## Eventos de domínio

Publica:

- `commercial.company.created|updated|suspended|blocked`.
- `commercial.connect_me.sent {direction, from, to?, category}`.
- `commercial.connect_me.interest_shown {request_id, company_id|syndic_id}` → LGPD log.
- `commercial.connect_me.expired` (auto-close 48h).
- `commercial.proposal.sent|accepted|rejected`.
- `commercial.deal.signed_complete {deal_id, company_id, condominium_id, value_cents}` → institutional publica timeline.
- `commercial.execution_record.submitted|approved|rejected`.
- `commercial.evaluation.submitted` → reputation recalc.
- `commercial.reputation.tier_changed {company_id, old_tier, new_tier}`.
- `commercial.harassment_report.created|confirmed` → `IUserModerator.SuspendUser`.
- `commercial.supplier_vote.session_closed` → assembly + content.
- `commercial.partner.seal_granted` (M3+).
- `commercial.coupon.generated`.
- `commercial.amendment.signed`.

Consome:

- `identity.user.banned` → marca `companies.suspended_until` / partner revogação.
- `assembly.voting.closed {session_id}` → atualiza SupplierVote result + revoga video grants.
- `content.video.approved {video_id, company_id}` → reputation recalc component.
- `billing.subscription.changed` → re-avalia visibilidade Vizinhança (plan gating).

## Requirements funcionais (FR-BE-COM-###)

- **FR-BE-COM-001** (COM-001) — *Quando* `POST /companies`, *o handler deve* criar tenant `company`, validar `CNPJ` via `ICNPJValidator` (stub M1, Receita M2+), persistir EnterpriseProfile com compliance markers JSONB (25 checkboxes).
- **FR-BE-COM-002** (COM-005) — *Quando* CNPJ submetido, *o backend deve* validar algoritmo dígito (M1 sync stub) + cache Redis `cnpj:{cnpj}` TTL 30d; M2+ chama adapter Receita via `ICNPJValidator`.
- **FR-BE-COM-003** (COM-006) — *Quando* `POST /connect-me/syndic-to-company` chega, *o handler deve* (UoW única): (a) `IQuotaConsumer.Consume(syndic_id, "commercial.connect_me", yearly)`, (b) criar ConnectMeRequest status=sent com payload estruturado (não texto livre), (c) notificar empresas qualificadas categoria via `IPushProvider`+`IEmailProvider`, (d) mascarar dados do síndico na notificação.
- **FR-BE-COM-004** (COM-007) — *Quando* empresa clica "Tenho Interesse" (`POST /connect-me/:id/interest`), *o handler deve* idempotente reveal dados síndico + gravar `lgpd_logs {purpose=connect_me_reveal, legal_basis=legitimate_interest, terms_version}`; 2ª chamada → no-op.
- **FR-BE-COM-005** (COM-008) — *Quando* cron 1h, *o job `ExpireConnectMe` deve* `UPDATE ... SET status='expired' WHERE status='sent' AND created_at < now() - interval '48 hours'`; notificação → síndico (quota NÃO devolvida).
- **FR-BE-COM-006** (COM-009) — *Quando* `POST /connect-me/resident-to-syndic`, *o handler deve* validar quota (0/ano Base, 4/ano Pagante — D-058 Fase 8), criar request unidirectional sem thread.
- **FR-BE-COM-007** (COM-010) — *Quando* `POST /connect-me/company-to-company`, *o ABAC deve* exigir `plan_tier ∈ {plus, pro}`; aceitar `partnership_type` enum 5 valores.
- **FR-BE-COM-008** (COM-011) — *Quando* `POST /connect-me/syndic-to-partner`, *o handler deve* aceitar `promotion_type` enum 5; parceiro responde aceite ou contra-proposta.
- **FR-BE-COM-009** (COM-012) — *Quando* `role=marketing` tenta Connect Me, *o ABAC deve* denegar `403`; UI não renderiza botão.
- **FR-BE-COM-010** (COM-013) — *Quando* `POST /harassment-reports`, *o handler deve* persistir `HarassmentReport` com context do Connect Me como evidência; trigger admin review; confirmação → `IUserModerator.SuspendUser` + `companies.suspended_until = now() + 72h` (COM-026).
- **FR-BE-COM-011** (COM-014) — *Quando* user digita Connect Me draft, *o frontend deve* POST `/connect-me/draft` auto-save Redis `ms:connect_me_draft:{user_id}` TTL 24h; limpa ao publicar.
- **FR-BE-COM-012** (COM-015) — *Quando* `POST /proposals`, *o handler deve* criar status=draft livremente editável; `POST /proposals/:id/send` → status=sent lock; amendment via COM-043.
- **FR-BE-COM-013** (COM-016) — *Quando* síndico rejeita proposta, *o backend deve* permitir empresa criar v2 com `previous_proposal_id` FK; chain navigable.
- **FR-BE-COM-014** (COM-018 + A-030) — *Quando* `POST /supplier-vote-sessions/:id/votes`, *o repository deve* fazer `SELECT ... FOR UPDATE` em `supplier_vote_sessions` WHERE id=$1 **antes** de INSERT vote; prevenindo race; UNIQUE `(session_id, voter_id)` (A-025).
- **FR-BE-COM-015** (COM-019) — *Quando* dupla assinatura deal ok, *o handler deve* (UoW): (a) lock deal imutável, (b) auto-gen `DealNumber` `DL-YYYYMMDD-NNN` sequential via trigger sequence per-day, (c) `ITimelinePublisher.Publish(type=registro_de_execucao)`, (d) publica `commercial.deal.signed_complete` → reputation recalc listener.
- **FR-BE-COM-016** (COM-020) — *Quando* `POST /execution-records`, *o handler deve* aceitar fotos (`IStorageProvider` S3 presigned) + vídeos (`IVideoProvider` Mux presigned via content BC); status `pending|approved|rejected`.
- **FR-BE-COM-017** (COM-021) — *Quando* síndico `POST /execution-records/:id/validate {approved|rejected, reason}`, *o handler deve* (UoW): rejeição exige `reason`; aprovação → `ITimelinePublisher.Publish(type=atividade_da_gestao)` + master_plan update (institutional listener).
- **FR-BE-COM-018** (A-029 + COM-024) — *Quando* `POST /company-evaluations {deal_id, scores}`, *o handler deve* ser **idempotente** via UNIQUE `(deal_id, reviewer_id)` (A-029 fix); 2ª submissão → `409 already_evaluated`; scores 5 critérios escala 1-10.
- **FR-BE-COM-019** (COM-025 + D-051) — *Quando* evento afeta reputação (`deal.signed_complete`, `evaluation.submitted`, `video.approved`, `harassment.confirmed`), *o listener deve* chamar `RecalculateReputation(company_id)` função pura — thresholds Bronze/Prata/Ouro/Platina/Diamante determinísticos; operador **não** pode alterar tier direto; degradação permitida.
- **FR-BE-COM-020** (COM-027) — *Quando* `GET /companies/:id/reputation-breakdown`, *o handler deve* retornar métricas que geraram o tier (contratos, avaliações, condomínios distintos, vídeos, tempo plataforma); zero black box.
- **FR-BE-COM-021** (COM-028 + COM-029) — *Quando* `GET /talents?categoria&regiao&palavra`, *o ABAC deve* exigir `role ∈ {enterprise, local_company, marketing, admin} + tier ∈ {plus, pro}`; listar vídeo-currículos ativos de moradores pagantes; `POST /videos?type=resume` (content handler) valida lock 90d (BIL-033).
- **FR-BE-COM-022** (COM-030 + COM-031) — *Quando* `POST /partners`, *o handler deve* criar parceiro Vizinhança com logo + fotos (≤3 via `IStorageProvider`); `POST /partners/:id/benefits` aceita 7 tipos enum.
- **FR-BE-COM-023** (COM-032) — *Quando* `POST /partners/:id/promotions`, *o handler deve* validar `scope ∈ {aberta_bairro, exclusiva_condominio}`; exclusiva requer `condominium_id` negociado.
- **FR-BE-COM-024** (COM-033 + BIL-034) — *Quando* morador Plus `POST /partners/:id/coupons/generate`, *o handler deve* (UoW): (a) validar cooldown 4h via `SELECT NOT EXISTS`, (b) `IQuotaConsumer.Consume` se aplicável, (c) gerar `CouponCode {ID_LOJA, SEQ day sequence, palavra_do_dia}`, (d) INSERT `coupon_generations`, (e) retornar código; cooldown violado → `429 coupon_cooldown` + `Retry-After`.
- **FR-BE-COM-025** (COM-034) — *Quando* `PATCH /partners/:id/palavra-do-dia`, *o handler deve* aceitar `[A-Z]{1,5}` + persistir `palavras_do_dia {partner_id, palavra, date}`; reset via cron 00:00 SP.
- **FR-BE-COM-026** (COM-035) — *Quando* síndicos N2+ votam seal (M3+), *o agregador deve* marcar `partners.seal_syndic_approved=true` quando threshold atingido (3 síndicos de condomínios distintos).
- **FR-BE-COM-027** (COM-036) — *Quando* parceiro consulta dashboard, *o handler deve* retornar views/cliques/cupons_gerados/resgates — privadas ao parceiro + admin.
- **FR-BE-COM-028** (COM-037) — *Quando* morador Plus `GET /neighborhood/feed?cep`, *o handler deve* filtrar parceiros por geo (CEP radius) + categorias; badge "Síndico-aprovado" se aplicável.
- **FR-BE-COM-029** (COM-039) — *Quando* `GET /companies/:id/dashboard`, *o ABAC deve* exigir dono da empresa; retornar condomínios conectados, CTR Connect Me, taxa aprovação execução, avaliação média; segmentação temporal.
- **FR-BE-COM-030** (COM-042) — *Quando* `GET /companies/search?category&cep&rating_min`, *o handler deve* usar `ISearchProvider` real (OpenSearch ou Meilisearch — ADR-0042 pending) + PostGIS geo; facetas (categorias, tiers); ordenação `_score` + distância. **D-120 revoga D-067**: tsvector vigente é stub.

## Endpoints REST expostos

| Método | Path | Auth | Descrição | Status |
|---|---|---|---|---|
| POST | /api/v1/companies | Bearer + role=enterprise | Onboarding empresa | ✅ |
| GET | /api/v1/companies/:id | Bearer | Detalhe (ABAC visibility) | ✅ |
| PATCH | /api/v1/companies/:id | Bearer + ABAC + strict DTO | Update (compliance markers) | ✅ |
| GET | /api/v1/companies/search | Bearer | `ISearchProvider` (stub tsvector M1; OpenSearch/Meili pós-ADR-0042) + PostGIS geo | 🟡 |
| GET | /api/v1/companies/:id/reputation-breakdown | Bearer | Métricas transparentes | 🟡 D-051 |
| GET | /api/v1/companies/:id/dashboard | Bearer + owner | Dashboard privado | 🟡 |
| POST | /api/v1/connect-me/syndic-to-company | Bearer + ABAC + quota | S→E | ✅ |
| POST | /api/v1/connect-me/resident-to-syndic | Bearer + quota | M→S | ✅ |
| POST | /api/v1/connect-me/company-to-company | Bearer + ABAC(plus/pro) | E→E | 🟡 |
| POST | /api/v1/connect-me/syndic-to-partner | Bearer + ABAC | S→P | 🟡 |
| POST | /api/v1/connect-me/:id/interest | Bearer + ABAC + idempotent | Revela dados + LGPD | ✅ |
| POST | /api/v1/connect-me/draft | Bearer | Auto-save Redis | ✅ |
| POST | /api/v1/harassment-reports | Bearer | Denúncia | ✅ |
| POST | /api/v1/proposals | Bearer + ABAC | Draft | ✅ |
| POST | /api/v1/proposals/:id/send | Bearer + ABAC | Lock | ✅ |
| POST | /api/v1/proposals/:id/amend | Bearer + dual signature | Amendment | 🟡 |
| POST | /api/v1/supplier-vote-sessions | Bearer + ABAC(syndic) | Abrir (min 2 propostas) | ✅ |
| POST | /api/v1/supplier-vote-sessions/:id/votes | Bearer + FOR UPDATE | Voto UNIQUE | ✅ |
| POST | /api/v1/deals/:id/sign | Bearer + dual | Dupla assinatura | ✅ |
| POST | /api/v1/execution-records | Bearer + ABAC(enterprise) | Registro | ✅ |
| POST | /api/v1/execution-records/:id/validate | Bearer + ABAC(syndic) | Aprovar/rejeitar | ✅ |
| POST | /api/v1/company-evaluations | Bearer + idempotent | 5 critérios | ✅ A-029 |
| GET | /api/v1/talents | Bearer + ABAC | Banco talentos | ⏳ M3 |
| POST | /api/v1/partners | Bearer + role=local_company | Cadastro Vizinhança | 🟡 A-033 |
| GET | /api/v1/partners/:id | Bearer | Perfil público | 🟡 |
| POST | /api/v1/partners/:id/benefits | Bearer + owner | 7 tipos | 🟡 |
| POST | /api/v1/partners/:id/coupons/generate | Bearer + quota + cooldown | Gera cupom | 🟡 |
| PATCH | /api/v1/partners/:id/palavra-do-dia | Bearer + owner | Reset diário | 🟡 |
| GET | /api/v1/neighborhood/feed | Bearer + ABAC | Feed por CEP | ⏳ |
| POST | /api/v1/marketing-links | Bearer + role=marketing | Sponsored link | ⏳ M3 |

## Integração com outros BCs

**Consome** (via `internal/shared/types`):

- `ITimelinePublisher` (institutional) — deal signed, execution approved.
- `IQuotaConsumer`, `IQuotaCheck` (billing).
- `IVideoProvider` (content/infra) — execution records video upload.
- `IStorageProvider` — fotos execução.
- `IUserModerator` (identity) — harassment → suspend.
- `ICondominiumIDsProvider` (institutional) — filtrar deals/supplier votes por condomínio escopo.
- `ICNPJValidator` (XD-020).
- `IEventBus`, `IAuditLogger`, `IEmailProvider`, `IPushProvider`.

**Expõe**:

- `IReputationReader` — `ReadTier(ctx, company_id) ReputationTier + Breakdown`.
- `IPartnerCatalog` — `ListByCondominium(ctx, cond_id) []Partner`.
- `ISupplierVoteReader` — assembly consulta resultado final.

**Publica eventos**: `commercial.company.*`, `commercial.connect_me.*`, `commercial.proposal.*`, `commercial.deal.*`, `commercial.execution_record.*`, `commercial.evaluation.*`, `commercial.reputation.*`, `commercial.harassment_report.*`, `commercial.supplier_vote.*`, `commercial.partner.*`, `commercial.coupon.*`, `commercial.amendment.*`.

**Consome eventos**: `identity.user.banned`, `assembly.voting.closed`, `content.video.approved`, `billing.subscription.changed`.

## Persistência

**Tabelas principais**:

- `companies` — CNPJ UNIQUE; JSONB compliance markers; `tier_reputation` calculated; `suspended_until TIMESTAMPTZ`.
- `connect_me_requests` — 4 variantes (tipo + from/to); `status`; `expires_at` para auto-close.
- `connect_me_reveals` — LGPD reveal log idempotent.
- `proposals` — `status` enum; `previous_proposal_id` FK.
- `deals` — `deal_number` UNIQUE auto-gen; dupla assinatura fields; append-only pós-sign.
- `execution_records` — validação FK.
- `technical_activities` — sem deal.
- `company_evaluations` — UNIQUE `(deal_id, reviewer_id)` (A-029); anônimas ao avaliado.
- `company_reputation_history` — recalc tracking.
- `harassment_reports` — context preservado.
- `supplier_vote_sessions` — FOR UPDATE lock.
- `supplier_votes` — UNIQUE `(session_id, voter_id)` (A-025).
- `partners` (Vizinhança) — palavra_do_dia, seal_syndic_approved.
- `palavras_do_dia` — histórico (auditabilidade).
- `partner_benefits`, `partner_promotions`.
- `coupon_generations` — 4h cooldown lookup.
- `amendments` — chain de modificações FK.
- `marketing_links` (M3+).
- `lgpd_logs` (cross-BC mas commercial é grande emissor).

**Migrations**: `00500..00699` commercial.

**RLS default-on**: `connect_me_requests`, `proposals`, `deals`, `execution_records`, `company_evaluations`, `coupon_generations` — policies por `(company_id|syndic_id|condominium_id)`.

## Providers externos

- **ViaCEP** (`ICEPLookup`) — validação CEP empresas/parceiros.
- **ICNPJValidator** stub M1, Receita Federal adapter M2+ (XD-020).
- **Storage** (`IStorageProvider`) — logo, fotos execução, compliance docs.
- **Mux** (`IVideoProvider`) via content — videos execução.
- **FCM/Email** — notificações Connect Me, Tenho Interesse, harassment admin alert.

**Sagas**:

- **DealSigningSaga** — empresa assina → síndico assina → (UoW) lock + DealNumber gen + `ITimelinePublisher` + reputation event publish; compensação se timeline publisher falha: rollback signatures (append row `deal_sign_rollback`).
- **CouponGenerationSaga** (leve) — cooldown check → quota consume → coupon code gen → INSERT; compensação: return quota se code gen falha.
- **ReputationRecalcSaga** — subscribes aos 4 eventos; idempotent recompute; publica tier_changed se mudou.

## Testes

**Coverage target**: Domain 90%, Application 80%, Infra 60%.

**PBT**:

- Reputation function purity: `∀ inputs → mesmo tier` (PBT seed determinístico).
- Supplier vote concurrency: `∀ 100 req simultâneas → ≤ vote count voters` (A-030 regression).
- Connect Me unidirectional: `∀ sequence → sem thread/reply cross-request`.
- CompanyEvaluation idempotent: `∀ duplicate POST → 409` (A-029 regression).
- Coupon cooldown: `∀ 2ª req < 4h → 429`.

**Integration**:

- DealSigningSaga E2E: Stripe-free flow + timeline cross-BC via `ITimelinePublisher`.
- HarassmentReport → admin action → `IUserModerator` suspend + companies.suspended_until.
- SupplierVoteSession com 2 propostas + fração ideal weight calc → result imutável.

## Segurança específica do BC

**LGPD**:

- Connect Me reveals → `lgpd_logs {purpose=connect_me_reveal, legal_basis=legitimate_interest}`.
- Harassment reports preservam context; retenção conforme LGPD.
- Evaluations anônimas — backend nunca retorna `reviewer_id` ao avaliado.
- Company dashboard: privado ao dono + admin.

**ABAC actions canônicos**:

- `commercial.connect_me.to_company|to_syndic|e2e|to_partner`.
- `commercial.proposal.create|send|amend`.
- `commercial.deal.sign.company|sign.syndic`.
- `commercial.execution_record.submit|validate`.
- `commercial.evaluation.submit`.
- `commercial.supplier_vote.create|vote`.
- `commercial.marketing_link.create`.
- `commercial.harassment_report.create|confirm`.
- `commercial.partner.create`, `commercial.partner.coupon.generate|create`.

**Rate limits**:

- `POST /connect-me/*` — 10/dia por user (anti-spam além da quota).
- `POST /partners/:id/coupons/generate` — 4h cooldown enforcement (não RL, é lock).
- `POST /harassment-reports` — 5/dia por user (anti-abuse).
- `POST /supplier-vote-sessions/:id/votes` — 1/(session, voter) UNIQUE DB.

## Decisões e dívidas

**D-###**:

- D-010 (tiers Bronze→Diamante thresholds numéricos) — pendência stakeholder.
- D-051 (Reputation motor) — **aberta**; função pura delineada mas cálculo final incompleto.
- D-059 (Sistema Cupons ATIVO) — fechado Fase 8.
- D-064 + D-078 (cupons 4h lock) — consolidado em D-059.
- D-073 (Agência de Marketing sub-produto próprio) — fechado Fase 7 renumerado.
- ADR-0029 (idempotency CompanyEvaluation) — fechado (A-029).
- ADR-0030 (saga orchestration default) — fechado.

**DT-###**:

- **A-030 aplicado** — `SupplierVoteSession FOR UPDATE` — fechado.
- **A-033 aberto** — Partner + coupon full flow ainda parcial.
- **D-051 bloqueador M2** — reputation motor finalização.
- Pendências COM-005 (Receita Federal adapter), COM-025 (thresholds finais), COM-033 (SEQ global vs loja), COM-017 (assinatura digital BR).

## Gaps e bloqueadores Sprint 10 (M1)

1. **Reputation motor completo** (D-051) — thresholds finais + degradação paths.
2. **Partner + coupon full flow** (A-033) — M2 target mas fundação M1.
3. **Connect Me E→E e S→P** — 🟡 endpoints; falta quota override e flow completo.
4. **CNPJ Receita adapter** (COM-005) — M2; M1 stub algoritmo.
5. **Amendment flow** — 🟡 tabela; fluxo dupla assinatura ainda parcial.
6. **Marketing links + Agência dashboards** (COM-040 + COM-041) — ⏳ M3.

## Ligações

- [[../../../04-requirements/functional/commercial]] — requirement canônico global
- [[../../../01-domain/aggregates/Company]]
- [[../../../01-domain/aggregates/ConnectMeRequest]]
- [[../../../01-domain/aggregates/Proposal]]
- [[../../../01-domain/aggregates/ClosedDeal]]
- [[../../../01-domain/aggregates/CompanyEvaluation]]
- [[../../../01-domain/aggregates/Partner]]
- [[../../../01-domain/aggregates/_moc]]
- [[../../../11-audit/AUDIT]]
- [[../architecture]]
- [[../patterns]] — Saga pattern aplicado a DealSigningSaga + ReputationRecalcSaga
- [[../security]]
- [[../tasks]] — Sprint 6-7 (legado), Sprint M2 lockdown
- Correspondente web: [[../../web/requirements/cms]]
- Correspondente mobile: a definir

## Links
- [[../../../_moc]]
- [[../../../CLAUDE]]
