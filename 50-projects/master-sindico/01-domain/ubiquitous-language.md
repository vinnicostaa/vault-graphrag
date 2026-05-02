---
title: Ubiquitous Language — Master Síndico (por Bounded Context)
type: spec
tags: [domain, ddd, ubiquitous-language, master-sindico, v2]
source: 90-ingested/from-vault-50-projects-master-sindico/01-domain/ubiquitous-language.md + contextos/CONTEXT_BRAIN.md + specs/requirements/*
created: 2026-04-23
updated: 2026-04-23
---

# Ubiquitous Language — por Bounded Context

Glossário **vinculante** código ↔ negócio, **organizado por BC**. Complementa [[../00-product/glossary]] (leitor-amigável; cross-BC). Aqui entra linguagem técnica do BC, com tipo DDD (`Aggregate Root`, `Entity`, `VO`, `Enum`, `Service`).

**Convenção** (herdada STEERING §3):
- **Domínio legal/condominial BR** (termo jurídico único): pt-BR preservado — `Assembleia`, `Sindico`, `Convencao`, `FracaoIdeal`, `Condomino`, `Ata`, `PlanoD iretor`.
- **Conceitos técnicos universais**: EN — `User`, `Session`, `Subscription`, `Plan`, `Webhook`, `Event`, `Role`.
- **Enums de estado técnico**: EN — `state: active | paused | canceled`.
- **Enums de domínio pt-BR**: pt-BR — `ReputacaoTier: Bronze | Prata | Ouro | Platina | Diamante`.

Mudança de termo = PR com justificativa + ADR + atualização de todos os usos.

---

## 1. identity

| Termo | Tipo DDD | Descrição |
|---|---|---|
| `User` | Aggregate Root | Identidade única por CPF ou CNPJ; mirror local do Zitadel |
| `Session` | Entity (interna) ou Aggregate Root | Sessão ativa com device fingerprint + IP + zitadel_session_id |
| `Email` | VO | Imutável, lowercase, validação RFC 5322 |
| `CPF` | VO | 11 dígitos + dígitos verificadores; `Masked()` pra log (`***.***.***-99`) |
| `CNPJ` | VO | 14 dígitos + dígitos verificadores; `Masked()` pra log |
| `Password` | VO | Nunca armazenado plain; gerenciado por Zitadel (Argon2) |
| `Phone` | VO | E.164; validação BR (DDD + número) |
| `DeviceFingerprint` | VO | SHA-256 hash server-side de `{ip + user_agent + canvas_hash | device_id}` — nunca reverter (IR-011) |
| `Role` | Enum EN | `syndic \| resident \| enterprise \| marketing \| local_company \| admin` — 1:1 com Zitadel (T7) |
| `RoleType` | Enum EN | `principal \| auxiliary \| assistant \| titular \| dependent \| administrator \| operator` — sub-contexto operacional, só local (T8) |
| `ZitadelID` | VO | ID externo do Zitadel; UNIQUE em `users` |
| `AuthUser` | DTO (`shared/types/`) | `{id, email, role, role_type, tenant_id, plan_tier}` — injetado em `contracts.Context` pós-Authenticate |
| `PKCE` | Conceito | `code_verifier` em cookie `gorilla/securecookie` |
| `UserStatus` | Enum | `pending_verification \| active \| suspended \| deleted` |
| `LoginAttempt` | Entity | rate limit + antifraude |

---

## 2. billing

| Termo | Tipo DDD | Descrição |
|---|---|---|
| `Subscription` | Aggregate Root | Assinatura Stripe mapeada; append-only (novo registro a cada mudança — I-007) |
| `Plan` | Entity | `{id, tier, slug, price_monthly_cents, price_yearly_cents, trial_days, features JSONB, quotas JSONB, stripe_product_id, stripe_price_*_id}` |
| `PlanTier` | Enum EN | `trial \| base \| plus \| pro` — compartilhado via `shared/types/` |
| `PlanSlug` | Enum | `resident_base \| resident_paid \| syndic_n1 \| syndic_n2 \| syndic_n3 \| enterprise_plus \| enterprise_pro \| marketing_standard \| local_company_standard` |
| `SubscriptionStatus` | Enum | `trial \| active \| past_due \| canceled \| paused \| expired` |
| `TrialDuration` | VO | Depende da persona: síndico 15d, empresa 7d, parceiro 30d, morador — (BR-001) |
| `Money` | VO | int64 centavos BRL — T10 (`shared/types/Money`) |
| `FeatureUsage` | Aggregate Root | `{id, user_id, feature_key, period, limit, used, unlimited, reset_at}` — contador vs quota |
| `Quota` | VO | `{limit: int, unlimited: bool}` |
| `StripeWebhookEvent` | Entity | `{id, provider, event_type, event_id UNIQUE, payload, processed, processed_at}` — idempotency ledger |
| `Paywall` | Conceito | Soft-block: dados permanecem, premium bloqueado (Decision 4 — sem grace period) |
| `QuotaKey` | Enum | `connect_me \| video_upload_institutional \| video_upload_curriculum \| live_session_minutes \| ...` |
| `QuotaPeriod` | Enum | `monthly \| annual \| once \| never` |

---

## 3. institutional

| Termo | Tipo DDD | Descrição |
|---|---|---|
| `Condominium` | Aggregate Root | **Tenant primário**; contém Units, Memberships, Mandates |
| `CondominiumType` | Enum pt-BR | `residencial \| comercial \| misto \| shopping_center \| galeria_comercial \| edificio_garagem` |
| `Unit` | Entity (dentro de Condominium) | Apartamento/casa/sala; tem fração ideal, titular, dependentes |
| `UnitType` | Enum pt-BR | `apartamento \| casa \| sala_comercial \| loja \| garagem \| outro` |
| `UnitIdentifier` | VO | String livre (ex: "101", "Apto 501", "Bloco B - 302") |
| `FracaoIdeal` | VO | NUMERIC(5,4) ∈ (0, 100]; 4 casas decimais — T10 |
| `Membership` | Aggregate Root ou Entity | Vínculo `user↔condo↔unit↔role` com período |
| `MembershipStatus` | Enum | `active \| suspended \| ended` |
| `TimelineEntry` | Aggregate Root | **Imutável**: sem UPDATE/DELETE/`deleted_at` (I-015) |
| `TimelineEntryType` | Enum pt-BR | `comunicado \| evento \| assembleia_result \| registro_execucao \| mandato_iniciado \| mandato_encerrado \| adendo \| outro` (7+ tipos) |
| `MasterPlan` (Plano Diretor) | Aggregate Root | Plurianual 3-5 anos; contém atividades com status |
| `MasterPlanActivity` | Entity | `{id, title, tipo_atividade (26 tipos), area_impactada (26), tipo_risco (9), status (7 estados), due_date, budget_estimated_cents}` |
| `Announcement` | Aggregate Root | Comunicado oficial; imutável pós-publish (I-016) |
| `AnnouncementType` | Enum pt-BR | `InformativoGeral \| Aviso \| Urgente \| Festa` |
| `Event` (Condominial) | Aggregate Root | Reunião, obra, festa, assembleia (como evento calendarizado) |
| `EventType` | Enum pt-BR | `ReuniaoMoradores \| Obra \| Festa \| Assembleia` |
| `EventResponse` | Entity | `ciente \| confirmado` (UPSERT) |
| `Poll` | Aggregate Root | Enquete informal (diferente de votação formal) |
| `PollType` | Enum | `sim_nao_nao_sei \| multiple_choice \| scale_1_to_5` |
| `SyndicMandate` | Entity | Período em que alguém é síndico |
| `MandateType` | Enum pt-BR | `sindico_eleito \| sindico_profissional \| sindico_interino` |
| `MandateStatus` | Enum pt-BR | `ativo \| encerrado \| transferido` |
| `ValidationItem` | Aggregate Root | Checklist legal (fundo reserva, laudos, certificações) |
| `ComplianceAssessment` | Aggregate Root | Avaliação periódica |
| `GovernanceScore` | VO | 0-100 (Req 33) |
| `AddressBR` | VO | `{cep, logradouro, numero, complemento, bairro, cidade, uf}` |
| `PublicID` | VO | 8 chars alfanumérico (ex: `CD2024AB`) |
| `EmpresaSindicaLink` | Entity | Convite síndico↔empresa síndica (actor delegado) com TTL |
| `ConflictOfInterestDeclaration` | Aggregate Root | Req 36 |
| `RiskMap` | Aggregate Root | Matriz de riscos (financeiro, compliance, estrutural, social, segurança) |

---

## 4. commercial

| Termo | Tipo DDD | Descrição |
|---|---|---|
| `Company` | Aggregate Root | Empresa cadastrada com CNPJ |
| `CompanyStatus` | Enum | `pending_verification \| active \| suspended \| blocked` |
| `CompanyUser` | Entity | Vínculo user↔company com role interno |
| `CompanyCategoria` | Entity (tabela `service_categories`) | Categoria de serviço; 32 categorias principais + subcategorias |
| `ConnectMeRequest` | Aggregate Root | RFP — formulário estruturado, unidirecional |
| `ConnectMeFluxo` | Enum pt-BR | `SindicoEmpresa \| MoradorSindico \| EmpresaEmpresa \| SindicoParceiro` (4 fluxos — legado) |
| `ConnectMeStatus` | Enum | `draft \| published \| interest_expressed \| auto_closed \| proposal_received \| closed` |
| `ConnectMeUrgency` | Enum pt-BR | `baixa \| media \| alta \| urgente` |
| `Proposal` | Entity (dentro de ConnectMeRequest?) ou Aggregate Root | Resposta de empresa; imutável pós-`sent` (I-024) |
| `ProposalStatus` | Enum EN | `draft \| sent \| awaiting_validation \| validated \| accepted \| rejected` |
| `ClosedDeal` | Aggregate Root | Contrato fechado; imutável pós-dupla-assinatura (I-025) |
| `DealNumber` | VO | Formato `DL-YYYYMMDD-NNN` |
| `ExecutionRecord` | Entity | Histórico de execução do contrato |
| `ExecutionStatus` | Enum | `registered \| pending_validation \| approved \| rejected` |
| `CompanyEvaluation` | Aggregate Root | Avaliação pós-contrato; UNIQUE (company, evaluator, condominium) |
| `CompanyAssessment` | Aggregate Root | Avaliação estruturada 5 critérios 1-10 (Req 26) — anônima ao avaliado |
| `ReputacaoTier` | VO (Enum pt-BR) | `Bronze \| Prata \| Ouro \| Platina \| Diamante` |
| `ReputationCalculator` | Domain Service | Função determinística (sem randomness) — DR-016 |
| `SupplierVoteSession` | Aggregate Root | Votação colegiada do conselho |
| `SupplierVoteCast` | Entity | Voto individual; UNIQUE (session, voter) |
| `EmpresaSindicaLink` | Entity | Convite síndico↔empresa síndica com TTL |
| `HarassmentReport` | Aggregate Root | Denúncia de abuso em Connect Me |
| `Amendment` (Adendo) | Aggregate Root | **Único mecanismo de modificação** de registros imutáveis (Req 25) |
| `AmendmentType` | Enum | `deal \| execution_record \| proposal \| timeline \| minutes` |
| `Partner` (Vizinhança) | Aggregate Root | Comércio local; CPF aceito, CNPJ não obrigatório |
| `PartnerCategory` | Enum pt-BR | ampla (pizzaria, petshop, farmácia, academia, etc) |
| `NeighborhoodBenefit` (Cupom) | Aggregate Root | Promoção do Dia; lock 4h por usuário |
| `BenefitType` | Enum pt-BR | `desconto_percentual \| desconto_fixo \| produto_gratuito \| combo_promocional \| avaliacao_gratuita \| brinde \| beneficio_exclusivo` (7 tipos) |
| `BenefitScope` | Enum | `aberta (bairro) \| exclusiva (condomínio)` |
| `CouponCode` | VO | Formato `ID_LOJA(5) + SEQ(3) + PALAVRA(5)` (ex: `00123055PAO`) |
| `Seal` | VO | "Síndico-aprovado" — flag em Partner concedida por síndico com `plan_tier ∈ {plus, pro}` (D-067/D-081/D-096) |
| `TechnicalActivity` | Entity | Req 23.1 — consultoria/auditoria/treinamento |
| `MarketingAgencyToken` | Entity | Token pra actor delegado agência (escopo `videos:upload`, `videos:edit`, `analytics:read`) |
| `Curriculum` | Aggregate Root | Vídeo-currículo do morador (Banco de Talentos) |

---

## 5. content

| Termo | Tipo DDD | Descrição |
|---|---|---|
| `Video` | Aggregate Root | Mux-backed; metadata local |
| `VideoState` | Enum | `uploading \| processing \| moderation \| approved \| published \| locked \| unlocked \| removed \| rejected` |
| `VideoType` | Enum pt-BR | 12 tipos: `apresentacao_empresa \| portfolio_servicos \| depoimento_cliente \| tutorial \| noticia \| evento_registrado \| demo_tecnica \| consultoria \| webinar \| explicativo \| entrevista \| outro` |
| `VideoModeration` | Aggregate Root | Estado de aprovação/rejeição |
| `VideoModerationStatus` | Enum | `pending \| approved \| rejected` |
| `LockedUntil` | VO | Timestamp = `published_at + 90d` (I-031) |
| `VideoVisibilityGrant` | Aggregate Root | Unlock temporário pra morador durante votação fornecedor (Rule 4) |
| `MuxAssetID` | VO | ID do asset no Mux |
| `PlaybackID` | VO | ID de playback (signed URL) |
| `AutorizarExibicaoVotacao` | Flag | Empresa autoriza visibilidade durante assembleia |
| `SearchIndex` | Aggregate Root | Materialized do `ISearchProvider` real (OpenSearch ou Meilisearch — ADR-0042 pending dual-check; D-120 revoga D-067 PG tsvector — vigente como stub M1) |
| `MediaProvider` | Enum | `Mux \| S3` |
| `Course` | Aggregate Root | Curso LMS; contém Modules + Lessons + Quizzes |
| `CourseArea` | Enum pt-BR | 5 áreas: `cursos \| treinamentos \| lives \| forum \| biblioteca` |
| `CourseCategory` | Enum pt-BR | 10 categorias: governança, legislação, finanças, RH, manutenção, compliance, segurança, relações, empreendedorismo, tecnologia |
| `CourseTrilha` | Enum pt-BR | 3 trilhas: `sindico \| empresa \| morador` |
| `CourseModule` | Entity | Módulo dentro de Course |
| `CourseLesson` | Entity | Lição dentro de Module |
| `Quiz` | Entity | Perguntas ao fim de Module/Course |
| `Enrollment` | Aggregate Root | Matrícula user↔course; progresso (%) |
| `Certificate` | Aggregate Root | Emitido ao 100% de Enrollment; serial number verificável |
| `ForumTopic` | Aggregate Root | Tópico de fórum |
| `ForumPost` | Entity | Post dentro de Topic |
| `ForumCategory` | Enum | 7 categorias por domínio de conhecimento |
| `LiveSessionLMS` | Entity | Live do LMS (distinta de Assembly live) |
| `LiveType` | Enum pt-BR | `aberta \| exclusiva \| com_replay \| seminar_serie` |
| `Ebook` | Aggregate Root | PDF ou EPUB hospedado em MinIO/R2 |
| `AccessLevel` | Enum | `free \| plus \| pro` |

---

## 6. assembly

| Termo | Tipo DDD | Descrição |
|---|---|---|
| `Assembly` (Assembleia) | Aggregate Root | Contém Agenda, Attendance, Votes, Minutes, Proxies |
| `AssemblyType` | Enum pt-BR | `Ordinaria \| Extraordinaria \| Ratificacao \| Outra` |
| `AssemblyStatus` | Enum EN | `scheduled \| published \| in_progress \| closed \| homologated` |
| `Modality` | Enum pt-BR | `presencial \| hibrida \| online` (MVP só `presencial`) |
| `QuorumType` | Enum EN | `simple_majority \| qualified_majority \| unanimous` |
| `QuorumThreshold` | VO | Mínimo fração ideal (ex: 0.5, 0.6667, 1.0) |
| `AgendaItem` | Entity (dentro de Assembly) | Item de pauta |
| `AgendaItemType` | Enum pt-BR | `informativo \| consulta_moradores \| votacao_simples \| votacao_fracao_ideal \| votacao_fornecedor` (5 tipos) |
| `AgendaItemState` | Enum | `Pending \| InDebate \| Voting \| Passed \| Rejected \| Withdrawn` |
| `Vote` | Entity (ou Aggregate Root) | Voto individual; UNIQUE(agenda_item_id, voter_id) |
| `VoteChoice` | Enum pt-BR | `Aprovar \| Rejeitar \| Abster \| Absent` (`Absent` = ausência momentânea timeout) |
| `VotingStatus` | Enum | `open \| closed` |
| `Minutes` (Ata) | Aggregate Root | Imutável pós-publish + homologation |
| `LiveSession` | Aggregate Root | Sessão Livekit associada |
| `ProxyGrant` (Procuração) | Aggregate Root | Outorga de voto |
| `ProxyStatus` | Enum | `active \| revoked \| expired` |
| `AttendanceRecord` | Entity | Check-in (app, terminal, manual) |
| `CheckInMode` | Enum pt-BR | `app \| terminal \| manual` |
| `ScienceRecord` (Ciência) | Entity | Registro de ciência pauta (Req 50.2) |
| `Acknowledgment` | Entity | Ciência obrigatória morador pré-assembleia |
| `SpeechQueue` | Aggregate Root | Fila de fala cronometrada |
| `SpeechTurn` | Entity | Turno de fala (timer + alerta 30s + timeout automático) |
| `Homologation` | Entity | Votação pós-ata (aprovação ata) |
| `HomologationStatus` | Enum | `pending \| approved \| rejected` |
| `TransparencyScore` | VO | 0-100 em 10 componentes (Req 60.7) |
| `Edital` | Entity | PDF de convocação; distribuição obrigatória |
| `PreliminarySurvey` | Aggregate Root | Enquete preliminar consultiva (Req 51; sem valor jurídico) |
| `ContingencyMode` | Conceito | Fallback Redis pra votos offline (Req 60.10) |

---

## 7. cross-domain

| Termo | Tipo DDD | Descrição |
|---|---|---|
| `AuditEntry` | Aggregate Root (append-only) | `{id, actor_id, action, resource_type, resource_id, tenant_id, metadata, ip, user_agent, created_at}` — retenção 5 anos (LGPD) |
| `AuditAction` | Enum | `create \| update \| delete \| login \| logout \| publish \| vote \| ...` |
| `LGPDLog` | Aggregate Root (append-only) | Log de revelação de dados pessoais (Connect Me, etc) |
| `AbacDecision` | VO | `{granted: bool, reason: AbacReason, details: map}` |
| `AbacReason` | Enum EN | `admin_bypass \| role_not_allowed \| action_not_configured \| tenant_mismatch \| scope_restricted \| resource_locked \| permitted` |
| `DomainEvent` | Interface (`core/contracts/`) | `{ID: UUIDv7, Type, AggregateID, OccurredAt, Payload}` — imutável (DR-017) |
| `ComplianceDeclaration` | Aggregate Root | Declaração anual obrigatória (Req 34) |
| `TermAcceptance` | Aggregate Root | `{user_id, term_version, accepted_at, ip}` — versionado (Req 35) |
| `RateLimitBucket` | VO | Token bucket `{identifier, action, count, window_start, expires_at}` |
| `GovernanceScoreSnapshot` | Aggregate Root | Score diário (Req 33) |
| `WebhookEvent` | Entity | Idempotency ledger cross-provider (`{provider, event_type, event_id UNIQUE, payload, processed}`) |
| `AntiFraudSignal` | Entity | IP suspeito, múltiplos cadastros, anomalia login |

---

## Termos proibidos (ambiguidade ou desuso)

- ❌ **`Tenant`** no código de domínio (ambíguo entre condo/company) — usar `Condominium` ou `Company` específico.
- ❌ **`Building`** — usar `Condominium` (termo do negócio).
- ❌ **`Customer`** — usar `User` ou `Company`.
- ❌ **`Invoice`** solto — está em Stripe; não é aggregate do Master Síndico.
- ❌ **`Resident`** solto — usar `Morador` (role) ou `User` (identidade) ou `Membership`.
- ❌ **`Manager`** — ambíguo; usar `Sindico` ou role explícita.
- ❌ **`Permission`** solto — usar `AbacDecision` ou `Policy`.
- ❌ **`Account`** — usar `User` ou `Subscription` conforme contexto.
- ❌ **`Admin`** como role geral — usar `SysAdmin` (global) ou `role: Admin` (contextual no BC).
- ❌ **`Chat`** / **`Message`** / **`Thread`** em Connect Me — Connect Me é **unidirecional** (Rule 3, I-022).

---

## Termos pt-BR em código Go (manter)

Preservar quando tem **semântica jurídica BR única** que EN não captura precisamente:

```go
// OK — jurídico brasileiro preciso
type Assembleia struct { ... }
type FracaoIdeal struct { ... }
type Sindico struct { ... }
type PlanoD iretor struct { ... }
type Ata struct { ... }
type Edital struct { ... }

// OK — enum legível (discriminator interno)
type ConnectMeFluxo string
const (
    FluxoSindicoEmpresa   ConnectMeFluxo = "sindico_empresa"
    FluxoMoradorSindico   ConnectMeFluxo = "morador_sindico"
    FluxoEmpresaEmpresa   ConnectMeFluxo = "empresa_empresa"
    FluxoSindicoParceiro  ConnectMeFluxo = "sindico_parceiro"
)

// RUIM — inglês sem ganho semântico
type Assembly struct { ... }        // perde precisão jurídica BR
type IdealFraction struct { ... }   // perde o termo da Lei 4.591/64
```

**API pública** (OpenAPI paths, JSON fields): **EN** por compatibilidade internacional:

```json
{
  "assembly_type": "ordinaria",  // path EN, valor pt-BR
  "agenda_items": [...],
  "quorum_threshold": 0.50,
  "connect_me_fluxo": "sindico_empresa"
}
```

---

## Quando adicionar/alterar/remover termo

- **Adicionar**: PR com justificativa + conceito de origem + BC + aggregate/VO onde vive. Atualiza este arquivo + [[../00-product/glossary]] se for cross-BC.
- **Alterar**: ADR + atualização de **todos os usos** (código, doc, spec, UI). Não mudar isoladamente.
- **Remover**: deprecation em 1 sprint (marca `deprecated: true` no frontmatter local); remoção em 2 sprints.

---

## ⚠️ Pendências de dual-check (Fase 5)

- **P-UL-001**: `Session` deveria ser Aggregate Root em `identity` (persistida + invariante 1-device) ou Entity dentro do `User`? Decisão: **Aggregate Root**, porque tem ciclo de vida próprio e transições. Confirmar.
- **P-UL-002**: `Proposal` é Entity dentro de `ConnectMeRequest` ou Aggregate Root próprio? Legado trata como tabela separada com FK — sugere AR próprio. Confirmar.
- **P-UL-003**: `CourseArea` vs `CourseCategory` vs `CourseTrilha` — hierarquia (Área > Categoria > Trilha) pode virar campos do aggregate `Course` (flat enums) ou sub-entidades. Decisão: flat enums por simplicidade M3 thin.
- **P-UL-004**: `Sindico` como Value Object (wrap do Role) ou simples enum? Legado trata como role no `Membership` + sub-tipo `RoleType: principal|auxiliary|assistant`. Manter enum.

---

## Links

- [[_moc]]
- [[bounded-contexts]]
- [[context-map]]
- [[invariants]]
- [[domain-events]]
- [[../00-product/glossary]] (leitor-amigável, cross-BC)
- [[../STEERING#princípios-do-produto]]
