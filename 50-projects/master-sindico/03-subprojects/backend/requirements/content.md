---
title: Content — Backend Requirements (Go)
type: requirement
tags: [master-sindico, backend, go, content, video, mux, lms, forum, live, feature-req]
project: master-sindico
stack: backend
bounded_context: content
milestone: m2
status: in-progress
created: 2026-04-24
---

# Content — Backend Requirements (Go)

## Escopo do bounded context

Módulo Go `internal/modules/content/` responsável por:

- **Video** via `IVideoProvider` Mux (presigned direct upload) + `CancelUploadSaga` (A-027).
- **12 tipos de vídeo** + 3 subtipos (institucional, instrucional, video_curriculum).
- **Moderação editorial** (M1-M2 manual; pipeline semi-auto M3+).
- **Lock 90d** pós-publish por tipo de vídeo (CNT-004).
- **MarketingAgencyLink** — agência uploada em nome de empresa (scope controlado via token IDN-025).
- **Paywall 25%** via Mux signed playback URL `policy.duration_limit` (CNT-008).
- **`ISearchProvider` real** (OpenSearch ou Meilisearch — ADR-0042 `proposed pending dual-check`) + PostGIS geo facetas. **D-120 Fase 14 revoga D-067**: tsvector vigente é stub temporário até search real.
- **LMS thin** (M3): Course/Module/Lesson + quiz 70% + certificado PDF.
- **Forum** (M2): 7 categorias, moderação reativa.
- **Live institucional** `ILiveStreamProvider` Mux Live ou Cloudflare Stream (pendente ADR-0044 CNT-022) — **admin + Empresa Pro + agência delegada** (D-097 Fase 11 revoga D-063/D-076 que diziam admin-only MVP).
- **Ebooks, podcast YouTube integration** (M3).
- **Admin broadcast, moderação global** (CNT-027, CNT-028).
- **LGPD log em publicações** (video.approved, video.rejected, moderation actions).

**Não é responsabilidade**: quota/plan gate (billing via `IQuotaConsumer`); ABAC engine (identity); timeline entry (institutional via `ITimelinePublisher` para eventos publicação).

## Status atual (vault canônico)

- **Sprints 6-7 legado**: Video + Mux integration + upload presigned + webhook `video.asset.ready` + moderação manual.
- **A-###** fechados: A-027 (`CancelUploadSaga` Mux), A-028 (EndRoom saga assembly/livekit futuro), A-031 (`ITimelinePublisher` interface), A-033 (parceiro+ILiveProvider).
- **A-### abertos**:
  - **CNT-005 search real M1** — D-120 revoga tsvector (D-067); ADR-0042 pending dual-check OpenSearch vs Meilisearch. Implementação tsvector vigente é stub.
  - **CNT-022 ADR Mux Live vs Cloudflare Stream** — aberto.
- **D-###**: D-097 Lives = admin + Empresa Pro + agência delegada (revoga D-063/D-076 admin-only); D-120 search real M1 (revoga D-067 tsvector — stub temporário); ADR-0027 webhook idempotency aplicado também a Mux; ADR-0044 (proposed) decide Mux Live vs Cloudflare Stream.

## Agregados (domain)

- [[../../../01-domain/aggregates/Video]] — `mux_asset_id`, `status {pending_upload, pending_moderation, approved, rejected}`, `published_at`, `locked_until`, `video_type` enum 12.
- `MarketingAgencyLink` — delegation tracking.
- [[../../../01-domain/aggregates/Course]] — Module → Lesson hierarchy.
- [[../../../01-domain/aggregates/Certificate]] — PDF via QR code validation.
- [[../../../01-domain/aggregates/LiveSession]] — compartilhado com assembly (ver assembly.md).
- `VideoModeration` — `{video_id, moderator_id, status, reason}`.
- `VideoVisibilityGrant` — temporary grant durante supplier vote (CNT-009).
- `VideoInteraction`, `VideoView` (analytics privados dono).
- `ContentLike`, `ContentComment` — registrados mas visíveis só ao dono (CNT-012).
- `ForumTopic`, `ForumPost`, `ForumLike`.
- `QuizQuestion`, `QuizResponse`, `Enrollment`.
- `DigitalContent` (ebooks).

## Entidades e Value Objects

| Tipo | Elemento | Invariante chave |
|---|---|---|
| Entity | `Video` | `mux_asset_id` UNIQUE; imutável pós `published_at`; lock 90d `locked_until` |
| Entity | `Course` | Hierarquia Course → Module → Lesson (FK + order) |
| Entity | `Certificate` | Emitido só em 100% (quiz ≥ 70%); QR code validation |
| VO | `VideoType` | enum 12 + subtipos especiais |
| VO | `VideoStatus` | state machine controlada |
| VO | `PlaybackDuration` | usado em Mux signed URL `policy.duration_limit` |
| VO | `LockUntil` | `published_at + 90d`; consultado em CNT-001 |
| VO | `CertificateQRCode` | hash único público validável |

## Invariantes (INV-CNT-###)

1. **INV-CNT-001**: Vídeo imutável após publicação (edição via adendo).
2. **INV-CNT-002**: Trava trimestral 90d entre atualizações do mesmo tipo (GR-029).
3. **INV-CNT-003**: Vídeo-currículo único por morador (não galeria).
4. **INV-CNT-004**: Visibilidade vídeo empresa a morador = preview 25% OU durante votação fornecedor.
5. **INV-CNT-005**: Métricas só visíveis ao dono ("cega" para terceiros).
6. **INV-CNT-006**: Agência zero acesso a dados de negócio.
7. **INV-CNT-007**: Parceiro Vizinhança sem LMS.
8. **INV-CNT-008**: Certificado emitido só em 100% completude (quiz ≥ 70%).
9. **INV-CNT-009**: Lives restritas ao Admin (D-063).
10. **INV-CNT-010**: Upload direto via Mux presigned (backend nunca proxy de bytes).

## Eventos de domínio

Publica:

- `content.video.uploaded {video_id, user_id, mux_asset_id}` — pré-moderação.
- `content.video.ready` — webhook Mux `video.asset.ready`.
- `content.video.approved {video_id, company_id?}` → commercial reputation + search index.
- `content.video.rejected {video_id, reason}` → notificação.
- `content.video.visibility_granted|revoked {video_id, assembly_id}` — supplier vote.
- `content.video.deleted` (pós-LGPD deletion).
- `content.course.created|updated|published`.
- `content.certificate.issued {user_id, course_id}` → `ITimelinePublisher` (timeline entry opcional).
- `content.live.started|ended` (admin broadcast).
- `content.forum.topic.created`, `content.forum.post.created|reported`.
- `content.ebook.published`.
- `content.admin.broadcast.sent`.

Consome:

- `assembly.voting.opened {assembly_id, session_id}` → grant video visibility (CNT-009).
- `assembly.voting.closed` → revoga grants.
- `commercial.company.suspended` → marcar vídeos inativos.
- `identity.user.banned` → ocultar conteúdo user-created.
- `institutional.timeline.entry_created {type=atividade_da_gestao}` → opcional trigger de vídeo explicativo (M3).

## Requirements funcionais (FR-BE-CNT-###)

- **FR-BE-CNT-001** (CNT-001) — *Quando* `POST /videos {video_type, metadata}`, *o handler deve* (UoW): (a) `IQuotaConsumer.Consume(user_id, "content.video.publish", monthly)`, (b) validar lock 90d via query `WHERE user_id=$1 AND video_type=$2 AND locked_until > NOW()` — existe → `409 video_locked`, (c) `IVideoProvider.CreateDirectUpload()` (Mux), (d) INSERT `videos {status=pending_upload, mux_asset_id, locked_until=NULL}`, (e) retornar `{upload_url, video_id}`; compensação via `CancelUploadSaga` se downstream falhar.
- **FR-BE-CNT-002** (CNT-001 + A-027) — *Quando* frontend aborta upload ou timeout 15min, *o `CancelUploadSaga` deve* (a) `IVideoProvider.DeleteAsset(mux_asset_id)`, (b) DELETE row pending_upload, (c) retornar quota consumido via `IQuotaConsumer.Release`; idempotente.
- **FR-BE-CNT-003** (CNT-002) — *Quando* video é criado, *o DTO deve* aceitar `video_type ∈ enum 12` + subtipos especiais (`institucional`, `instrucional`, `video_curriculum`); subtipo determina regras lock + visibility.
- **FR-BE-CNT-004** (CNT-003) — *Quando* webhook Mux `video.asset.ready` chega, *o handler deve* (HMAC verify ADR-0027 + UNIQUE event_id idempotência): (a) UPDATE `videos SET status='pending_moderation'`, (b) enqueue em `moderation_queue`; admin review via `POST /admin/videos/:id/moderate {approved|rejected, reason}`.
- **FR-BE-CNT-005** (CNT-004) — *Quando* moderation approves, *o handler deve* (UoW): (a) SET `status='approved', published_at=NOW(), locked_until=NOW()+interval '90 days'`, (b) publicar `content.video.approved`, (c) reindex via `ISearchProvider.Index(...)` (D-120 — stub tsvector M1; OpenSearch/Meili pós-ADR-0042).
- **FR-BE-CNT-006** (CNT-005 + D-120 revoga D-067) — *Quando* `GET /search?q&type&cep&rating_min`, *o handler deve* query `ISearchProvider` (OpenSearch ou Meilisearch — ADR-0042 `proposed pending dual-check`) em `videos`, `companies`, `syndics`, `partners`; paginação 20/página. Implementação tsvector vigente é stub temporário.
- **FR-BE-CNT-007** (CNT-006) — *Quando* search tem `cep` ou `geo=<lat,lng>`, *o backend deve* PostGIS `ST_DWithin` radius default 5km; ordenação `_score * 0.7 + (1 - distance/radius) * 0.3`.
- **FR-BE-CNT-008** (CNT-008 + matriz) — *Quando* morador Base `GET /videos/:id/playback`, *o handler deve* `IVideoProvider.CreateSignedPlaybackURL(asset_id, policy={duration_limit: video.duration * 0.25})`; Plus/Pro → integral.
- **FR-BE-CNT-009** (CNT-009 + INV-CNT-004) — *Quando* supplier vote abre com `autorizar_exibicao_votacao=true`, *o listener `assembly.voting.opened` deve* INSERT `video_visibility_grants {video_id, assembly_id, granted_at}`; listener `assembly.voting.closed` → SET `revoked_at=NOW()`; middleware ABAC `content.video.read` consulta grants ativos.
- **FR-BE-CNT-010** (CNT-011) — *Quando* frontend pulse a cada 15s com `watch_percentage`, *o handler `POST /videos/:id/view-progress` deve* ser idempotente via `(user_id, video_id)` key; incrementa `video_views.count` apenas quando `watch_percentage ≥ 70%` pela primeira vez.
- **FR-BE-CNT-011** (CNT-012 + CNT-034) — *Quando* `GET /videos/:id/analytics` ou `/metrics`, *o ABAC deve* exigir owner; non-owner → `403`; PBT `cross-tenant req → 403` sempre.
- **FR-BE-CNT-012** (CNT-013) — *Quando* `POST /videos/:id/likes`, *o handler deve* idempotente UNIQUE `(user_id, content_id)`; `GET /videos/:id/comments` não-dono → array vazio (privacy cega).
- **FR-BE-CNT-013** (CNT-014) — *Quando* agência (via token IDN-025 scope `videos:upload`) `POST /videos {company_id}`, *o handler deve* validar token scope, criar vídeo atribuído à empresa (não à agência); audit trail marker `marketing_agency_user_id`; agência **NÃO** vê deals/Connect Me/users.
- **FR-BE-CNT-014** (CNT-016 + CNT-017) — *Quando* admin/empresa Pro `POST /courses`, *o handler deve* aceitar hierarquia Course → Module → Lesson `{video_id, descricao, anexos}`; ABAC exige `role=admin OR (role=enterprise AND plan.tier=pro)`; 5 áreas × 10 categorias × 3 trilhas enum.
- **FR-BE-CNT-015** (CNT-018) — *Quando* aluno `POST /lessons/:id/quiz-response`, *o handler deve* calcular score; ≥ 70% → marca lesson `completed`; < 70% → permite retry.
- **FR-BE-CNT-016** (CNT-019) — *Quando* aluno completa 100% course, *o job async `IssueCertificate` deve* gerar PDF (nome + curso + data + QR code único), S3 upload via `IStorageProvider`, criar `certificates` row com URL signed; `GET /certificates/verify/:qr_code` público.
- **FR-BE-CNT-017** (CNT-020) — *Quando* user acessa LMS, *o ABAC deve* aplicar matriz: Base→lives+forum+biblioteca; Pagante/N2/N3→cursos+cert; Enterprise Plus→consume+cert; Enterprise Pro→**create**+cert; Parceiro Vizinhança→**denied**.
- **FR-BE-CNT-018** (CNT-021) — *Quando* `POST /forum/topics` (enterprise/syndic), *o handler deve* aceitar categoria + texto; moderação reativa via `POST /forum/posts/:id/report`; moradores apenas `POST /forum/posts` (comentário).
- **FR-BE-CNT-019** (CNT-022 + D-063) — *Quando* `POST /lives` (admin only via ABAC shortcut BIL-035), *o handler deve* chamar `ILiveStreamProvider.CreateLive()` (Mux Live ou Cloudflare Stream — ADR pendente); persistir replay 30d.
- **FR-BE-CNT-020** (CNT-025 + CNT-026) — *Quando* admin/enterprise Pro `POST /digital-contents` (ebook), *o backend deve* armazenar em S3 via `IStorageProvider`; `GET /digital-contents/:id/download` signed URL TTL 24h; sem DRM.
- **FR-BE-CNT-021** (CNT-027) — *Quando* admin `POST /admin/videos/:id/moderate`, *o handler deve* validar ABAC `role=admin`, persistir `video_moderations`, dispatch event (approved → reindex; rejected → notification empresa com `reason`).
- **FR-BE-CNT-022** (CNT-028) — *Quando* admin `POST /admin/broadcasts`, *o handler deve* permitir segmentação `{persona[], plan_tier[], condominium_id?}`, enfileirar jobs via `IEmailProvider` + `ISMSProvider`; log de disparos.

## Endpoints REST expostos

| Método | Path | Auth | Descrição | Status |
|---|---|---|---|---|
| POST | /api/v1/videos | Bearer + ABAC + quota | Create presigned | ✅ |
| GET | /api/v1/videos/:id/playback | Bearer + ABAC | Signed URL (paywall aware) | ✅ |
| GET | /api/v1/videos/:id/analytics | Bearer + owner | Métricas privadas | ✅ |
| POST | /api/v1/videos/:id/view-progress | Bearer + idempotent | Pulse ≥70% | ✅ |
| POST | /api/v1/videos/:id/likes | Bearer + idempotent | Like UNIQUE | ✅ |
| POST | /api/v1/videos/:id/comments | Bearer | Comentário | ✅ |
| GET | /api/v1/videos/:id/comments | Bearer + owner | Lista (cega cross-tenant) | ✅ |
| POST | /webhooks/mux | público + HMAC + UNIQUE | Mux webhook | ✅ |
| POST | /admin/v1/videos/:id/moderate | Bearer + admin | Aprovar/rejeitar | ✅ |
| GET | /api/v1/search | Bearer + cursor | `ISearchProvider` (stub tsvector M1; OpenSearch/Meili pós-ADR-0042) + PostGIS | 🟡 search real + geo |
| POST | /api/v1/courses | Bearer + ABAC | LMS create | ⏳ M3 |
| POST | /api/v1/courses/:id/modules | Bearer + owner | Module | ⏳ M3 |
| POST | /api/v1/courses/:id/modules/:mid/lessons | Bearer + owner | Lesson | ⏳ M3 |
| POST | /api/v1/enrollments | Bearer + ABAC | Enroll em course | ⏳ M3 |
| POST | /api/v1/lessons/:id/quiz-response | Bearer | ≥70% marca completed | ⏳ M3 |
| GET | /api/v1/certificates/:id | Bearer + owner | Certificate | ⏳ M3 |
| GET | /certificates/verify/:qr_code | público | Validação pública | ⏳ M3 |
| POST | /api/v1/forum/topics | Bearer + ABAC | Create (ent+sindic) | ⏳ M2 |
| POST | /api/v1/forum/posts | Bearer | Comentário | ⏳ M2 |
| POST | /api/v1/forum/posts/:id/report | Bearer | Moderação reativa | ⏳ M2 |
| POST | /api/v1/lives | Bearer + admin | Admin-only | 🟡 ADR pendente |
| POST | /api/v1/digital-contents | Bearer + admin/ent-pro | Ebook | ⏳ M3 |
| GET | /api/v1/digital-contents/:id/download | Bearer + ABAC | Signed URL 24h | ⏳ M3 |
| GET | /api/v1/podcast/episodes | Bearer + cache | YouTube Data API | ⏳ |
| POST | /admin/v1/broadcasts | Bearer + admin | Segmented | ✅ |

## Integração com outros BCs

**Consome**:

- `IQuotaConsumer`, `IQuotaCheck` (billing).
- `ITimelinePublisher` (institutional) — publish certificate issued (opcional).
- `IStorageProvider`, `IVideoProvider` (XD-013 Mux), `ILiveStreamProvider` (XD-014 Livekit/Mux Live/Cloudflare).
- `IEmailProvider`, `IPushProvider`.
- `IUserModerator` (identity) — para banned users hide content.
- `IAuditLogger` (DT-010).
- `ICondominiumIDsProvider` (institutional) para scope em forum por condomínio.

**Expõe**:

- `IVideoReader` — `GetVideo(ctx, id) VideoPublic`, `ListByCompany(ctx, company_id)`.
- `IVideoVisibility` — `GrantForSession(ctx, video_ids, assembly_id)`, `RevokeForSession(ctx, assembly_id)`.
- `IVideoProvider` (adapter) exposto a assembly para attachments Mux (ASM-008).

**Publica eventos**: `content.video.*`, `content.course.*`, `content.certificate.*`, `content.live.*`, `content.forum.*`, `content.ebook.*`, `content.admin.broadcast.*`.

**Consome eventos**: `assembly.voting.opened|closed`, `commercial.company.suspended`, `identity.user.banned`.

## Persistência

**Tabelas principais**:

- `videos` — `mux_asset_id` UNIQUE, `video_type` enum, `status`, `published_at`, `locked_until`, `owner_type` (`user|company`).
- `video_moderations` — append-only; `reason` obrigatório para rejected.
- `video_visibility_grants` — `assembly_id` FK; `granted_at`, `revoked_at`.
- `video_views`, `video_interactions` — analytics.
- `content_likes` — UNIQUE `(user_id, content_id)`.
- `content_comments` — audit trail; report button.
- `marketing_agency_links` — delegation.
- `courses`, `course_modules`, `course_lessons`.
- `enrollments`, `quiz_questions`, `quiz_responses`.
- `certificates` — QR code UNIQUE; S3 URL.
- `forum_topics`, `forum_posts`, `forum_likes`, `forum_reports`.
- `digital_contents` (ebooks).
- `lives` — admin-scheduled.
- `admin_broadcasts` — log.
- `search_index_videos`, `search_index_companies`, etc. — materialized views tsvector (stub M1); migrar para índices `ISearchProvider` real pós-ADR-0042 (D-120).

**Migrations**: `00700..00899` content.

**RLS default-on**: `videos` (owner scope), `video_views`, `video_interactions`, `content_comments`, `enrollments`, `quiz_responses`, `certificates`.

## Providers externos

- **Mux** (`IVideoProvider` — XD-013) — direct upload, signed playback, thumbnails, webhooks.
- **Mux Live / Cloudflare Stream** (`ILiveStreamProvider` — XD-014) — ADR pendente CNT-022.
- **Storage S3/R2** (`IStorageProvider`) — certificates PDF, ebooks, assets.
- **YouTube Data API** — podcast (cache 1h).
- **wkhtmltopdf/Puppeteer** — certificate PDF gen.
- **Resend/Twilio** — admin broadcasts.

**Sagas**:

- **CancelUploadSaga** (A-027) — frontend abort/timeout → `IVideoProvider.DeleteAsset` + DELETE pending + `IQuotaConsumer.Release`; idempotente.
- **IssueCertificateSaga** — 100% complete → PDF gen → S3 upload → row INSERT → email user; compensação se PDF falha: retry 3x com backoff, depois admin alert.
- **EndLiveRoomSaga** (A-028 relacionada a assembly) — livekit end → recording S3 → replay metadata.

## Testes

**Coverage target**: Domain 90%, Application 80%, Infra 60%.

**PBT**:

- Lock 90d: `∀ sequência (upload, publish, attempt-upload dia 89) → 409`.
- Privacy cega: `∀ user_A request analytics user_B → 403` ({A,B}∈users).
- Paywall: `∀ Base user stream → duration_limit ≤ 0.25 * video.duration`.
- Quiz threshold: `∀ score < 70 → retry allowed; ∀ score ≥ 70 → lesson completed`.
- Certificate idempotency: `∀ duplicate trigger → 1 certificate row`.

**Integration** (testcontainers + Mux mock):

- CancelUploadSaga compensação (A-027 regression).
- Webhook Mux + HMAC + UNIQUE event_id idempotency.
- Video visibility grant/revoke flow via assembly events.

## Segurança específica do BC

**LGPD**:

- `lgpd_logs` emitido em: video.approved (purpose=content_publication), video.deletion (esquecimento), admin.broadcast (legitimate_interest + segmentation).
- Content comments retidos com user_id mas invisíveis cross-tenant (CNT-012).
- Audit trail em toda moderação (approved/rejected com reason).

**ABAC actions canônicos**:

- `content.video.publish`, `content.video.publish.institutional`, `content.video.publish.curriculum`.
- `content.video.read`, `content.video.moderate`.
- `content.lms.course.create`, `content.lms.certificate.issue`.
- `content.live.start`, `content.live.view`.
- `content.forum.topic.create`, `content.forum.post.create`, `content.forum.moderate`.
- `content.admin.broadcast.send`, `content.admin.moderation.global`.

**Rate limits**:

- `POST /videos` — quota mensal (billing) + 10/h por user.
- `POST /forum/topics` — 20/dia por user.
- `POST /forum/posts` — 50/dia por user.
- `POST /videos/:id/likes` — 100/min per user.
- `POST /admin/broadcasts` — 10/dia admin.

## Decisões e dívidas

**D-###**:

- D-063 (Lives admin-only M1) — **REVOGADA** por D-097 Fase 11 (admin + Empresa Pro + agência delegada).
- D-067 (OpenSearch → tsvector temporário) — **PARCIAL-REVOGADA** por D-120 Fase 14 (search real obrigatório M1; tsvector vira stub).
- D-076 (Lives admin-only MVP Fase 7) — **REVOGADA** por D-097 Fase 11.
- D-097 (Lives = admin + Empresa Pro + agência delegada) — **ATIVA** Fase 11.
- D-120 (search real M1) — **ATIVA** Fase 14; ADR-0042 dual-check OpenSearch vs Meilisearch pendente.
- ADR-0027 (webhook idempotency — aplicado também a Mux) — fechado.
- **CNT-022 ADR-0044 Mux Live vs Cloudflare Stream** — proposed; research-loop.

**DT-###**:

- **A-027 CancelUploadSaga** — fechado.
- **CNT-005 search real M1** — 🔴 ISearchProvider real obrigatório M1 (D-120 revoga D-067); tsvector é stub. ADR-0042 pending dual-check decide OpenSearch vs Meilisearch. Facetas + geo proximity entram com search real.
- Pendências CNT-003 (SLA moderação M3+ semi-auto), CNT-019 (assinatura digital BR certificate), CNT-028 (template segmentação), CNT-022 (ADR Live provider).

## Gaps e bloqueadores Sprint 10 (M1)

1. **Lives provider ADR** (CNT-022) — bloqueia start de lives admin formais.
2. **Search geo facets completas** (CNT-005 / CNT-006) — PostGIS ST_DWithin OK mas facetas avançadas pendentes.
3. **LMS** — ⏳ M3; fundação de schema pendente (courses/modules/lessons).
4. **Forum** — ⏳ M2; sem código ainda.
5. **Certificate PDF + QR validation** — ⏳ M3 dependente de LMS.
6. **Ebook download** — ⏳ M3.
7. **Podcast integration** — ⏳ baixa prioridade.

## Ligações

- [[../../../04-requirements/functional/content]] — requirement canônico global
- [[../../../01-domain/aggregates/Video]]
- [[../../../01-domain/aggregates/Course]]
- [[../../../01-domain/aggregates/Certificate]]
- [[../../../01-domain/aggregates/_moc]]
- [[../../../02-architecture/adr/0027-webhook-idempotency-db]]
- [[../../../11-audit/AUDIT]]
- [[../architecture]]
- [[../patterns]] — Saga pattern (CancelUploadSaga, IssueCertificateSaga)
- [[../security]]
- [[../tasks]] — Sprints 6-7
- Correspondente web: [[../../web/requirements/cms]] + [[../../web/requirements/lms]] + [[../../web/requirements/forum]]
- Correspondente mobile: a definir

## Links
- [[../../../_moc]]
- [[../../../CLAUDE]]
