---
title: Bounded Contexts — Master Síndico
type: spec
tags: [domain, ddd, bounded-contexts, master-sindico, v2]
source: 90-ingested/from-vault-50-projects-master-sindico/01-domain/bounded-contexts.md + specs/requirements/{identity,billing,institutional,commercial,content,assembly,cross-domain}.md
created: 2026-04-23
updated: 2026-04-23
---

# Bounded Contexts — Master Síndico v2

Catálogo canônico dos 6 bounded contexts (BCs) + camada `cross-domain`. Cada BC é **coeso** (uma responsabilidade nuclear) e **separado** (não importa `domain/` nem `application/` de outro BC; cruza via events + `shared/types/`).

> **Regra dura 14 de [[../STEERING]]**: nenhum BC importa `domain/` ou `application/` de outro BC. Comunicação **obrigatória** via domain events ou `shared/types/`.

Fonte canônica do conteúdo destilado: legado `01-domain/bounded-contexts.md` + `specs/requirements/*.md`.

---

## Visão geral (mapa)

```
┌─────────────┐    ┌─────────────┐    ┌──────────────┐
│  identity   │───▶│   billing   │───▶│institutional │
│  (Zitadel)  │    │  (Stripe)   │    │ (Condomínio) │
└──────┬──────┘    └──────┬──────┘    └──────┬───────┘
       │                  │                  │
       ▼                  ▼                  ▼
┌─────────────┐    ┌─────────────┐    ┌──────────────┐
│  commercial │    │   content   │    │   assembly   │
│ (Connect Me)│    │    (Mux)    │    │  (Livekit)   │
└──────┬──────┘    └──────┬──────┘    └──────┬───────┘
       ▲                  ▲                  ▲
       └──────────────────┴──────────────────┘
                    [ cross-domain ]
          (ABAC, AuditLogger, EventBus, LGPD)
```

Setas = dependência conceitual (via event ou `shared/types/`, nunca import direto).

---

## 1. `identity`

### Responsabilidade primária

Gestão de **identidade única** por CPF/CNPJ, sessões, autenticação via Zitadel OIDC (Authorization Code + PKCE), 1-device policy, sync `users` local ↔ Zitadel (mirror pattern). Guarda ABAC claims (`role`, `role_type`).

### Linguagem ubíqua (BC)

`User`, `Session`, `DeviceFingerprint`, `Role` (`syndic|resident|enterprise|marketing|local_company|admin`), `RoleType` (`principal|auxiliary|assistant|titular|dependent|administrator|operator`), `ZitadelID`, `PKCE`, `Cookie httpOnly` `.mastersindico.com.br`.

### Aggregates principais

- `User` (raiz) — identidade única (CPF ou CNPJ)
- `Session` — sessão ativa com device fingerprint + IP

### VOs

`Email`, `CPF`, `CNPJ`, `Password` (nunca armazenada plain — Zitadel), `Phone` (E.164), `DeviceFingerprint` (SHA-256 hash server-side — IR-011).

### Invariantes

- **I-001**: `users.zitadel_id` UNIQUE (1 User MS = 1 User Zitadel)
- **I-002**: 1 CPF = 1 User; 1 CNPJ = 1 User (chave natural)
- **I-003**: ≤ 1 Session ativa por User (configurável via `MAX_ACTIVE_SESSIONS`; default 1 — BR-004)
- **I-004**: `role` nunca string vazia — middleware rejeita `403 NO_ROLE_ASSIGNED` antes do UPSERT (T15)
- **I-005**: `sessions.zitadel_session_id` nunca re-usado após revogação

### Dependências (eventos consumidos)

- Nenhum (BC raiz de identidade)

### Eventos publicados

`UserCreated`, `UserUpdated`, `UserDeleted`, `SessionCreated`, `SessionInvalidated`, `DeviceChanged`.

### Provider externo isolado

- **Zitadel** via `IAuthProvider` / `IZitadelProvider` (`internal/shared/providers/auth/`). OIDC `zitadel/oidc/v3`.

### Status legado

✅ Produção (Sprint 0-1 legado).

### Marco de entrega

**M1 (2026-05-08)** — parte da base contratual.

### Links

- [[context-map]] (U/D com billing, institutional)
- [[ubiquitous-language#identity]]
- [[invariants#identity]]
- [[domain-events#identity]]
- [[state-machines#user-session]]
- [[aggregates/User]] · [[aggregates/Session]]

---

## 2. `billing`

### Responsabilidade primária

Planos, assinaturas via Stripe, **trial persona-aware** (síndico 15d, empresa 7d, parceiro 30d, morador —), **quotas por plano** (Connect Me anual, vídeos mensal), webhooks idempotentes, **soft-block sem grace period** (Decision 4 legado).

### Linguagem ubíqua (BC)

`Subscription`, `Plan` (`trial|base|plus|pro` + variantes por persona), `PlanTier`, `TrialDuration` (persona-aware), `Money` (int64 centavos BRL), `FeatureUsage`, `Quota` (`limit`+`unlimited`), `StripeWebhookEvent`, `Paywall`, `Soft block`.

### Aggregates principais

- `Subscription` (raiz) — assinatura Stripe mapeada; contém Seats
- `Plan` — definição de plano (ID, tier, features, preço)
- `FeatureUsage` (raiz) — contador consumo vs quota
- `StripeWebhookEvent` — idempotency ledger

### VOs

`Money` (int64 centavos BRL — T10), `PlanTier`, `TrialDuration`, `Quota` (limit + unlimited flag).

### Invariantes

- **I-006**: `users.plan_id` aponta a Subscription ativa ou NULL (soft-block não apaga)
- **I-007**: `subscriptions` é append-only — novo registro a cada mudança
- **I-008**: Trial nunca volta após conversão para paid
- **I-009**: Valores monetários sempre int64 centavos (T10, DR-011)
- **I-010**: Webhook `event.id` único no ledger (Stripe)
- **I-011**: Quota saldo ∈ [0, ∞]; `used ≤ limit` quando `!unlimited` (DR-010)
- **I-012**: Trial duration persona-aware — nunca global (BR-001)

### Dependências (eventos consumidos)

- `UserCreated` (de `identity`) → cria Subscription em trial
- `CompanyCreated` (de `commercial`) → cria Subscription empresa

### Eventos publicados

`SubscriptionActivated`, `SubscriptionCanceled`, `SubscriptionExpired`, `TrialStarted`, `TrialExpiringSoon`, `TrialExpired`, `QuotaExhausted`, `PlanChanged`.

### Provider externo isolado

- **Stripe** via `IPaymentGateway` (`internal/shared/providers/payment/`).
- Webhook público `POST /webhooks/stripe` (fora `/api/v1`; HMAC verify antes de parse — IR-012).

### Status legado

✅ Produção (Sprint 1).

### Marco de entrega

**M1 (2026-05-08)** — base contratual.

### Links

- [[context-map]] (U/D com identity; fornece feature-gate pra institutional/commercial/content/assembly)
- [[ubiquitous-language#billing]]
- [[invariants#billing]]
- [[domain-events#billing]]
- [[state-machines#subscription]]
- [[aggregates/Subscription]] · [[aggregates/Plan]] · [[aggregates/FeatureUsage]]

---

## 3. `institutional`

### Responsabilidade primária

**Condomínio** como tenant primário; **Unidades** com fração ideal (Σ ≤ 100%); **Memberships** (`User↔Condomínio↔Unit↔Role`); **Timeline imutável** (append-only, sem UPDATE/DELETE/`deleted_at`); **Plano Diretor** plurianual; **Mandatos** de síndicos; **Comunicados/Eventos/Enquetes**; **Compliance** (checklists, Governance Score, declaração anual).

### Linguagem ubíqua (BC)

`Condominium`, `Unit`, `FracaoIdeal` (NUMERIC(5,4) — nunca float), `Membership`, `Role` (`sindico|morador|subsindico|conselho`), `Mandate` (tipo, start/end), `TimelineEntry` (imutável), `MasterPlan` (Plano Diretor), `Announcement`, `Event` (condominial), `Poll` (enquete informal), `ValidationItem`, `ComplianceAssessment`, `GovernanceScore` (0-100), `Edital`, `PublicID` (8 chars alfanumérico), `QRCode`, `SyndicMandate`.

### Aggregates principais

- `Condominium` (raiz — **tenant primário**) — contém Units, Memberships
- `Unit` — tem fração ideal
- `Membership` — vínculo ativo por período
- `TimelineEntry` — **imutável** (sem UPDATE/DELETE)
- `MasterPlan` (Plano Diretor) — plurianual; contém atividades com status
- `Announcement` — comunicado oficial (pós-publish imutável)
- `Event` — evento condominial
- `Poll` — enquete informal
- `ValidationItem` + `ComplianceAssessment` — checklists legais
- `SyndicMandate` — período em que alguém é síndico
- `ConflictOfInterestDeclaration` — Req 36
- `RiskMap` — Req 38 (matriz de riscos)

### VOs

`FracaoIdeal` (DECIMAL 4 casas, ∈ (0, 100]), `AddressBR` (CEP + rua + nº + complemento), `UnitNumber` (string), `PublicID` (8 chars alfa), `CondominiumType` (`residencial|comercial|misto|shopping_center|galeria_comercial|edificio_garagem`).

### Invariantes

- **I-013**: Σ `FracaoIdeal` de Units em Condominium ≤ 100% (DB CHECK + PBT — DR-001)
- **I-014**: `FracaoIdeal ∈ (0, 100]` (DR-002)
- **I-015**: TimelineEntry imutável — sem UPDATE/DELETE/`deleted_at` (DR-007)
- **I-016**: Announcement imutável pós-publish (DR-008)
- **I-017**: Membership ativa única por (user, condominium) em dado momento
- **I-018**: ≤ 15 condomínios por síndico (Decision 11 legado)
- **I-019**: Mandate `end_date > start_date`
- **I-020**: Toda Unit tem exatamente 1 titular (Morador principal) + até 2 dependentes (Req 9)

### Dependências (eventos consumidos)

- `UserCreated` (de `identity`) — habilita Membership
- `SubscriptionActivated`/`SubscriptionExpired` (de `billing`) — libera/trava features
- `ExecutionValidated` (de `commercial`) — atualiza status de atividade no `MasterPlan`
- `AssemblyClosed`/`MinutesPublished` (de `assembly`) — gera TimelineEntry

### Eventos publicados

`CondominiumCreated`, `UnitRegistered`, `UnitUpdated`, `MembershipCreated`, `MembershipEnded`, `TimelineEntryAdded`, `AnnouncementPublished`, `EventScheduled`, `PollOpened`, `PollClosed`, `MandateStarted`, `MandateTransferred`, `MandateEnded`, `MasterPlanActivityAdvanced`, `ComplianceDeclarationSubmitted`, `GovernanceScoreUpdated`.

### Provider externo isolado

- **ViaCEP** via `ICEPLookup` — validação CEP
- (M2+) Receita Federal via `ICNPJValidator` — unicidade CNPJ

### Status legado

✅ Produção (Sprints 2-3).

### Marco de entrega

**M1 (2026-05-08)** — base (Condominium, Unit, Membership, Timeline, Plano Diretor básico).
**M2 (2026-06-20)** — Poll + Announcement completo.
**M3 (2026-07-20)** — Event + Compliance + Governance Score.

### Links

- [[context-map]] (Partnership com assembly, Customer/Supplier com commercial)
- [[ubiquitous-language#institutional]]
- [[invariants#institutional]]
- [[domain-events#institutional]]
- [[state-machines#masterplan]]
- [[aggregates/Condominium]] · [[aggregates/Unit]] · [[aggregates/Membership]] · [[aggregates/TimelineEntry]] · [[aggregates/MasterPlan]] · [[aggregates/Announcement]] · [[aggregates/Event]] · [[aggregates/Poll]]

---

## 4. `commercial`

### Responsabilidade primária

**Empresas** (CNPJ), **Connect Me** (4 fluxos unidirecionais — não chat), **Propostas** (imutáveis pós-send), **Negócios Fechados** (dupla assinatura → imutável), **Avaliações de Empresa** (anônimas), **Reputação Bronze→Diamante** (função determinística), **SupplierVote** (colegiada do conselho), **Empresa-Síndica Link** (delegado), **Harassment Reports**, **Vizinhança** (parceiros locais + cupons lock 4h + seal de síndico), **Banco de Talentos** (busca de morador por empresa).

### Linguagem ubíqua (BC)

`Company`, `CompanyUser` (role interno), `ConnectMeRequest`, `ConnectMeFluxo` (`Vizinhanca|Academia|Marketplace|Direto`; legado 4 fluxos: `SindicoEmpresa|MoradorSindico|EmpresaEmpresa|SindicoParceiro`), `Proposal`, `ClosedDeal`, `ExecutionRecord`, `CompanyEvaluation`, `ReputacaoTier` (`Bronze|Prata|Ouro|Platina|Diamante`), `SupplierVoteSession`, `SupplierVoteCast`, `EmpresaSindicaLink`, `HarassmentReport`, `Amendment` (adendo — único mecanismo de alteração — Req 25), `NeighborhoodBenefit` (cupom), `Seal` ("Síndico-aprovado").

### Aggregates principais

- `Company` (raiz) — empresa cadastrada com CNPJ
- `CompanyUser` — vínculo user↔company (role na empresa)
- `ConnectMeRequest` (raiz) — RFP aberto
- `Proposal` — resposta de empresa
- `ClosedDeal` — contrato fechado
- `ExecutionRecord` — histórico execução contrato
- `CompanyEvaluation` (raiz) — avaliação pós-contrato
- `SupplierVoteSession` + `SupplierVoteCast`
- `EmpresaSindicaLink` — convite síndico↔empresa com TTL
- `HarassmentReport`
- `Amendment` — adendo (corrige imutabilidade)
- `Partner` (Vizinhança) — comércio local
- `NeighborhoodBenefit` — cupom/promoção (lock 4h)
- `CompanyAssessment` (Req 26) — avaliação estruturada 5 critérios

### VOs

`ReputacaoTier` (enum pt-BR), `ExecutionRecord`, `CNPJ` (compartilhado com `identity` via `shared/types/`), `Money` (compartilhado com `billing`), `ConnectMeFluxo`, `DealNumber` (formato `DL-YYYYMMDD-NNN`).

### Invariantes

- **I-021**: Reputação é função determinística de métricas — nenhum input humano direto altera tier (BR-003, DR-016)
- **I-022**: Connect Me é **unidirecional** — sem thread, sem reply (Rule 3)
- **I-023**: Proposta única por (ConnectMeRequest, empresa) — UNIQUE DB
- **I-024**: Proposal imutável após `sent` (Rule 2) — correção via Amendment (Req 25)
- **I-025**: ClosedDeal imutável após dupla assinatura (Rule 2)
- **I-026**: SupplierVoteCast único por (session, voter) — UNIQUE DB (DR-005; A-026 legado fix)
- **I-027**: CompanyEvaluation única por (company, evaluator, condominium) — UNIQUE DB (DR-006; A-029 legado fix)
- **I-028**: Avaliação anônima ao avaliado (Req 26)
- **I-029**: NeighborhoodBenefit cupom lock 4h por usuário (D-012 legado)
- **I-030**: Quota ConnectMe decrementa no `publish` (não no `draft`)

### Dependências (eventos consumidos)

- `UserCreated`, `MembershipCreated` (de `identity`/`institutional`) — cache ABAC
- `SubscriptionActivated`, `PlanChanged` (de `billing`) — desbloqueia features
- `AssemblyClosed` com SupplierVoteSession (de `assembly`) — consolida resultado de votação de fornecedor

### Eventos publicados

`CompanyCreated`, `CompanyApproved`, `CompanySuspended`, `ConnectMeRequestPublished`, `ConnectMeInterestExpressed`, `ConnectMeAutoClosed`, `ProposalSubmitted`, `ProposalAccepted`, `ProposalRejected`, `ClosedDealSigned`, `ExecutionRecordSubmitted`, `ExecutionValidated`, `ExecutionRejected`, `CompanyEvaluationSubmitted`, `ReputationTierChanged`, `SupplierVoteSessionOpened`, `SupplierVoteCastRegistered`, `SupplierVoteClosed`, `HarassmentReported`, `AmendmentSigned`, `NeighborhoodBenefitPublished`, `CouponRedeemed`.

### Provider externo isolado

- Nenhum direto; **delega mídia para `content`** via events (upload vídeo empresa).
- FCM/SES/Twilio notificações via `ICommsProvider`.

### Status legado

✅ Produção (Sprint 4); Vizinhança parcial.

### Marco de entrega

**M2 (2026-06-20)** — Connect Me 4 fluxos + Reputação Bronze→Diamante + Vizinhança cupons + Banco Talentos.

### Links

- [[context-map]] (Customer/Supplier com content; Partnership com institutional)
- [[ubiquitous-language#commercial]]
- [[invariants#commercial]]
- [[domain-events#commercial]]
- [[state-machines#connectmerequest]] · [[state-machines#company-reputation]]
- [[aggregates/Company]] · [[aggregates/ConnectMeRequest]] · [[aggregates/Proposal]] · [[aggregates/ClosedDeal]] · [[aggregates/CompanyEvaluation]] · [[aggregates/SupplierVoteSession]] · [[aggregates/HarassmentReport]] · [[aggregates/Partner]]

---

## 5. `content`

### Responsabilidade primária

**Vídeos institucionais** (empresa/síndico com `plan_tier ∈ {plus, pro}` — D-067/D-081/D-096) e **vídeo-currículos** (morador via addon Banco de Talentos opt-in — D-099 5 vínculos M3) via **Mux** (upload presigned direto, HLS transcoding, signed playback URL). **Moderação manual** 48h (pre-publish). **Trava 90 dias pós-publish** (churn prevention). **Privacidade de métricas cross-tenant** (views/likes não vazam). **Busca via `ISearchProvider`** real (OpenSearch ou Meilisearch — D-120/ADR-0042 pending dual-check). **LMS** (Academia): cursos, trilhas, quizzes, certificados. **Ebooks**. **Lives broadcast** institucionais (1→N) em `BroadcastLive` aggregate (D-133 — provider Cloudflare Stream/Mux Live ADR-0044).

### Linguagem ubíqua (BC)

`Video`, `VideoState` (`uploading|processing|moderation|approved|published|locked|unlocked|removed|rejected`), `LockedUntil` (timestamp 90d), `VideoModeration`, `VideoType` (12 tipos: apresentação, portfólio, depoimento, tutorial, notícia, evento, demo técnica, consultoria, webinar, explicativo, entrevista, outro), `VideoVisibilityGrant` (unlock momentâneo durante assembleia — Rule 4), `MuxAssetID`, `PlaybackID`, `SearchIndex`, `Course`, `CourseModule`, `CourseLesson`, `Quiz`, `Certificate`, `Enrollment`, `ForumTopic`, `ForumPost`, `LiveSessionLMS` (distinto da assembly live), `Ebook`, `MediaProvider` (`Mux|S3`).

### Aggregates principais

- `Video` (raiz) — Mux-backed
- `VideoModeration` — estado de aprovação/rejeição
- `VideoVisibilityGrant` — unlock temporário durante votação fornecedor (Rule 4)
- `SearchIndex` — materialized representation
- `Course` (raiz) — contém Modules, Lessons, Quiz
- `Enrollment` (raiz) — matrícula user↔course, tracking progresso
- `Certificate` — emitido ao 100% conclusão
- `ForumTopic` (raiz) — contém Posts
- `Ebook` — PDF/EPUB hospedado

### VOs

`VideoDuration`, `LockedUntil`, `MuxAssetID`, `PlaybackID`, `VideoType` (enum 12), `CertificateSerialNumber`.

### Invariantes

- **I-031**: Vídeo `published` → `locked` por 90 dias (`locked_until = published_at + 90d`) — remove requer `NOW() >= locked_until` OR `user.is_admin` (DR-015)
- **I-032**: Moderação obrigatória antes de `published` (VideoState: `moderation → approved → published`)
- **I-033**: Métricas não vazam entre tenants (ABAC + LGPD Req 9 rule)
- **I-034**: Vídeos empresa bloqueados default pra morador; unlock só durante votação fornecedor com `autorizar_exibicao_votacao=true` (Rule 4, Req 103)
- **I-035**: Trava trimestral — substituir mesmo vídeo só após 90d (Rule 7)
- **I-036**: Agência de Marketing: zero acesso a dados de negócio (delegado restrito)
- **I-037**: Certificado emitido automaticamente ao 100% de Enrollment
- **I-038**: Parceiro Vizinhança: zero acesso a LMS (botão oculto)

### Dependências (eventos consumidos)

- `CompanyApproved` (de `commercial`) — permite upload empresa
- `SubscriptionActivated` (de `billing`) — habilita quota upload
- `AssemblyVotingOpened` (de `assembly`) — cria VideoVisibilityGrant
- `AssemblyVotingClosed` (de `assembly`) — revoga grant
- `ExecutionValidated` (de `commercial`) — recomenda curso LMS relevante

### Eventos publicados

`VideoUploaded`, `VideoModerationRequested`, `VideoApproved`, `VideoRejected`, `VideoPublished`, `VideoLocked`, `VideoLockExpired`, `VideoRemoved`, `VideoVisibilityGranted`, `VideoVisibilityRevoked`, `CourseCreated`, `CoursePublished`, `EnrollmentCreated`, `LessonCompleted`, `CourseCompleted`, `CertificateGenerated`, `ForumTopicCreated`, `ForumPostAdded`, `EbookDownloaded`.

### Provider externo isolado

- **Mux** via `IVideoProvider` (`internal/shared/providers/video/`). Upload presigned; webhook público `POST /webhooks/mux` (HMAC verify antes de parse).
- **AWS S3 / Cloudflare R2** via `IStorageProvider` — thumbnails, ebooks.
- **`ISearchProvider` real** — OpenSearch ou Meilisearch (ADR-0042 `proposed pending dual-check`); D-120 Fase 14 revoga D-067 (tsvector vigente é stub M1).
- **Saga com compensação** (A-027 legado): se `UploadVideo` (Create Mux asset) sucesso mas Create local falha → `CancelUpload` no Mux.

### Status legado

✅ Produção (Sprint 5); LMS parcial; ebooks parcial.

### Marco de entrega

**M2 (2026-06-20)** — Vídeos + Mux + Lock 90d + Moderação manual + Busca.
**M3 (2026-07-20)** — LMS thin (5 áreas, 3 trilhas, certificados).
**M4+** — LMS pleno + moderação ML + ebooks biblioteca completa.

### Links

- [[context-map]] (Customer/Supplier com commercial; Partnership com assembly via VideoVisibilityGrant)
- [[ubiquitous-language#content]]
- [[invariants#content]]
- [[domain-events#content]]
- [[state-machines#videomoderation]]
- [[aggregates/Video]] · [[aggregates/VideoModeration]] · [[aggregates/Course]] · [[aggregates/Enrollment]] · [[aggregates/Certificate]]

---

## 6. `assembly`

### Responsabilidade primária

**Assembleias** (ordinária, extraordinária, ratificação, outra) com **edital** obrigatório pré-início, **pauta** (5 tipos item), **ciência obrigatória** morador, **check-in** (app/terminal/manual), **votação real-time** (WebSocket < 200ms) com **quórum fracional** (peso = fração ideal), **procuração** (proxy), **ausência momentânea** (15min), **live streaming** (Livekit), **fila de fala** cronometrada, **ata imutável** auto-gerada, **homologação**, **score de transparência** (0-100 em 10 componentes).

### Linguagem ubíqua (BC)

`Assembly` (Assembleia), `AssemblyType` (`Ordinaria|Extraordinaria|Ratificacao|Outra`), `AssemblyStatus` (`scheduled|published|in_progress|closed|homologated`), `Modality` (`presencial|hibrida|online`; MVP só presencial), `AgendaItem`, `AgendaItemType` (`informativo|consulta_moradores|votacao_simples|votacao_fracao_ideal|votacao_fornecedor`), `Vote`, `VoteChoice` (`Aprovar|Rejeitar|Abster|Absent`), `VotingStatus` (`open|closed`), `Minutes` (Ata), `LiveSession`, `Proxy` (Procuração), `AttendanceRecord` (Presença), `ScienceRecord` (Ciência), `Acknowledgment` (ciência pauta), `SpeechQueue`, `SpeechTurn`, `QuorumThreshold`, `QuorumType` (`simple_majority|qualified_majority|unanimous`), `TransparencyScore` (0-100), `Homologation`, `Edital`, `PresidentPanel`, `ContingencyMode` (offline fallback).

### Aggregates principais

- `Assembly` (raiz) — contém Agenda, Attendance, Votes, Minutes, Proxies
- `AgendaItem` — item de pauta (pode ter fração ideal requerida)
- `Vote` — voto individual; UNIQUE(agenda_item_id, voter_id)
- `Minutes` (raiz — Ata) — imutável pós-publish
- `LiveSession` (raiz) — sessão Livekit correspondente
- `AttendanceRecord` — check-in (quem compareceu, modalidade)
- `Proxy` (raiz) — procuração (outorga de voto)
- `SpeechQueue` + `SpeechTurn` — fila de fala
- `Acknowledgment` — ciência obrigatória de pauta (Req 50.2)
- `Homologation` — votação pós-ata
- `PreliminarySurvey` — enquete preliminar consultiva (Req 51)

### VOs

`QuorumThreshold` (NUMERIC), `VoteChoice` (enum pt-BR), `Modality`, `SpeechDuration`, `TransparencyScoreValue`.

### Invariantes

- **I-039**: Vote único por (agenda_item_id, voter_id) — UNIQUE DB (DR-004; A-025 legado fix)
- **I-040**: Minutes imutável pós-publish (DR-009)
- **I-041**: Quórum calculado por fração ideal, não por pessoa (BR-007)
- **I-042**: Assembly não inicia sem edital publicado (DR-014; A-013 legado fix)
- **I-043**: ≤ 1 assembleia ativa por condomínio (Req 48)
- **I-044**: Vote após timeout → registrado como `absent` (Req 57)
- **I-045**: Procuração: proxy não pode votar diferente do titular
- **I-046**: `published_at` em pauta → read-only (lock)
- **I-047**: Síndico presidente **não** vota (imparcialidade — Req 60.2)
- **I-048**: Live room órfã evitada por Saga (A-028 legado)
- **I-049**: Fração ideal em votação: NUMERIC (T10), nunca float
- **I-050**: `mandate.end_date > mandate.start_date` (herdado de `institutional`)

### Dependências (eventos consumidos)

- `CondominiumCreated`, `UnitRegistered`, `MembershipCreated` (de `institutional`) — habilita assembleia
- `SubscriptionActivated` (de `billing`) — desbloqueia live + features
- `ProposalValidated` (de `commercial`) — habilita votação fornecedor (Req 21)

### Eventos publicados

`AssemblyScheduled`, `EditalPublished`, `AgendaPublished`, `AcknowledgmentRegistered`, `AssemblyStarted`, `CheckInRegistered`, `ProxyGranted`, `ProxyRevoked`, `VotingOpened`, `VoteCast`, `VotingClosed`, `SpeechQueueJoined`, `SpeechTurnStarted`, `SpeechTurnEnded`, `AssemblyClosed`, `MinutesGenerated`, `MinutesPublished`, `HomologationOpened`, `HomologationApproved`, `TransparencyScoreCalculated`.

### Provider externo isolado

- **Livekit** via `ILiveProvider` (WebRTC SFU; rooms `assembly:{id}`; API corte microfone).
- **PDF gen lib** (wkhtmltopdf ou Puppeteer) — edital + ata.
- **Saga compensation**: A-028 (`StartLive` → `EndRoom` se Assembly create falha), A-033 (homologação async), A-034 (notification fan-out).

### Status legado

✅ Backend produção (Sprint 7); frontend parcial.

### Marco de entrega

**M3 (2026-07-20)** — sessão live completa + votação + ata + proxy + quórum fracional.
**M4+** — modalidade online/híbrida pleno.

### Links

- [[context-map]] (Partnership com institutional; Conformist com commercial pra SupplierVote)
- [[ubiquitous-language#assembly]]
- [[invariants#assembly]]
- [[domain-events#assembly]]
- [[state-machines#assembly]] · [[state-machines#votesession]]
- [[aggregates/Assembly]] · [[aggregates/AgendaItem]] · [[aggregates/Vote]] · [[aggregates/Minutes]] · [[aggregates/LiveSession]] · [[aggregates/Proxy]] · [[aggregates/AttendanceRecord]]

---

## 7. Cross-Domain

### Responsabilidade primária

Artefatos transversais que **não pertencem** a BC único. Concentra:

- **IAuditLogger** (DT-010 legado — pendente) — log cross-módulo ações sensíveis
- **Domain event bus** — publica/consome entre módulos (in-process hoje; NATS JetStream quando escalar — Sprint 10+)
- **ABAC engine** + policy catalog `(resource_type, action) → allowed_roles, plan_tier`
- **Redis cache** com decisão ABAC (TTL 5min); invalidação via webhook Stripe
- **LGPD Registry** (`lgpd_logs` append-only 5 anos)
- **Audit Trail** append-only (Req 37)
- **Governance Score** job diário (Req 33)
- **Compliance Declaration** anual obrigatória (Req 34)
- **Antifraude** (Req 102): CAPTCHA 3 falhas, bloqueio múltiplos IPs, detecção anomalia IP novo
- **Providers externos transversais**: Resend (email), Twilio (SMS), FCM (push), ViaCEP

### Linguagem ubíqua

`AuditEntry`, `AuditTrail` (append-only), `AbacDecision` (`Permit|Deny(reason)`), `DomainEvent` (UUIDv7 ordenável — DR-017), `LGPDLog`, `ComplianceDeclaration`, `GovernanceScoreComponent`, `ConflictOfInterestDeclaration`, `RiskMap`, `TermAcceptance`, `RateLimitBucket`.

### Aggregates principais (transversais; vivem em `cross-domain/`)

- `AuditEntry` — append-only
- `LGPDLog` — append-only
- `ComplianceDeclaration` — anual
- `TermAcceptance` — versionado
- `ConflictOfInterestDeclaration`
- `RiskMap`
- `GovernanceScoreSnapshot` — snapshot diário

### VOs

`AbacDecision`, `AbacReason` (`admin_bypass|role_not_allowed|action_not_configured|tenant_mismatch|scope_restricted|resource_locked`), `GovernanceScoreValue` (0-100), `AuditActorRef` (`{user_id, tenant_id, role}`).

### Invariantes

- **I-051**: Audit trail append-only (nunca UPDATE/DELETE — DR-007 generalizado)
- **I-052**: LGPD logs retenção 5 anos; direito ao esquecimento após vencimento (BR-010)
- **I-053**: HMAC verify **antes** de parse em todo webhook (Stripe, Mux, Livekit — DR-020)
- **I-054**: PII nunca em logs — CI grep bloqueia (DR-019)
- **I-055**: ABAC ordem estrita: admin → role → action → tenant → scope (DR-012)
- **I-056**: Tenant isolation: toda query escopada por `condominium_id` (DR-013; 100+ integration tests)
- **I-057**: Providers externos agnósticos via interface (IR-001 DIP)
- **I-058**: Events têm UUIDv7 ordenável (DR-017)
- **I-059**: ABAC engine falha startup se matriz vazia (A1 antipattern legado)

### Dependências

- Consome eventos de **todos** os BCs (pra audit + ABAC cache invalidation + LGPD logging)
- Publica **poucos** eventos: `AuditEntryAppended`, `LGPDLogAppended`, `GovernanceScoreCalculated`, `TermAccepted`, `AntiFraudTriggered`.

### Providers externos isolados

- **Resend** via `IEmailProvider`
- **Twilio / Zenvia** via `ISMSProvider`
- **FCM** via `IPushProvider`
- **ViaCEP** via `ICEPLookup`
- **reCAPTCHA v3 / Cloudflare Turnstile** (antifraude)

### Status legado

🟡 Em evolução. `IAuditLogger` pendente (DT-010). ABAC engine produção. LGPD parcial.

### Marco de entrega

**M1-M3** — transversal; cada marco herda especializações.

### Princípio inegociável

**Cross-domain NÃO pode importar `domain/` ou `application/` de dois módulos ao mesmo tempo**. Se precisar agregar dados cross-BC: evento de domínio + handler local.

### Links

- [[context-map]] (Shared Kernel com todos)
- [[ubiquitous-language#cross-domain]]
- [[invariants#cross-domain]]
- [[domain-events#cross-domain]]
- [[aggregates/AuditEntry]] (a criar)

---

## Tabela-resumo (status + marco + provider)

| BC | Responsabilidade primária | Provider externo | Status legado | Marco |
|---|---|---|---|---|
| `identity` | User + Session + Auth Zitadel + 1-device | Zitadel (`IAuthProvider`) | ✅ Produção | **M1** |
| `billing` | Subscription + Plan + Trial + Quotas + Stripe | Stripe (`IPaymentGateway`) | ✅ Produção | **M1** |
| `institutional` | Condomínio + Timeline imutável + Plano Diretor + Compliance | ViaCEP; (M2+) Receita Federal | ✅ Produção | **M1** base / **M2-M3** evolução |
| `commercial` | Connect Me + Reputação + Vizinhança + Banco Talentos | FCM/SES via `cross-domain` | ✅ Produção (parcial) | **M2** |
| `content` | Vídeos Mux + LMS + Busca + Moderação + Lock 90d | Mux (`IVideoProvider`), S3/R2, OpenSearch | ✅ Produção (parcial) | **M2** base / **M3** LMS thin |
| `assembly` | Assembleias + Votação fracional + Ata imutável + Livekit | Livekit (`ILiveProvider`), PDF lib | ✅ Backend produção | **M3** |
| `cross-domain` | ABAC + Audit + LGPD + Providers transversais | Resend, Twilio, FCM, ViaCEP | 🟡 Em evolução (DT-010) | **M1-M3** transversal |

---

## ⚠️ Pendências de dual-check (Fase 5)

- **P-BC-001**: confirmar se `Partner` (Vizinhança) deve ser aggregate próprio em `commercial` ou sub-agregado de `Company` (legado trata como persona separada `local_company` com tenant próprio — manter).
- **P-BC-002**: `Council` (Conselho) aparece como role em `Membership` + como participante de `SupplierVoteSession`. Confirmar se é VO/enum ou entidade própria em `institutional`.
- **P-BC-003**: `Agencia de Marketing` é **actor delegado** (token empresa), não persona tenant. Confirmar que não justifica BC próprio (legado: "não é tenant").
- **P-BC-004**: `LMS` (Academia) tá em `content` — confirmar que não merece BC próprio em M4+ pleno (legado mantém em `content`; v2 herda).
- **P-BC-005**: `MasterPlan` evento `ExecutionValidated` vem de `commercial` mas afeta `institutional`. Confirmar se é via event bus ou shared view.

---

## Links

- [[_moc]]
- [[context-map]]
- [[ubiquitous-language]]
- [[invariants]]
- [[domain-events]]
- [[business-rules]]
- [[state-machines]]
- [[aggregates/_moc]]
- [[../00-product/portfolio-de-produtos]]
- [[../STEERING]]
- [[../CLAUDE]]
