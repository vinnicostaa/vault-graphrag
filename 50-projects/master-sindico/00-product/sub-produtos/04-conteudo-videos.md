---
title: "Sub-produto 4 — Conteúdo & Vídeos"
type: spec
tags: [sub-produto, content, videos, shorts, mux, lock-90d, master-sindico, v2, fase-8]
source: fontes listadas
created: 2026-04-23
updated: 2026-04-23
status: destilado
dual_check: pending
bc: content
req: [29, 30, 31, 32, 93, 103]
screens: [MK1..MK8, PL1..PL4]
---

# 04 — Conteúdo & Vídeos

> **Sub-produto Master Síndico** (Fase 8 v2). Biblioteca de **vídeos institucionais** de empresas, com **Mux** (`IVideoProvider`) codificando HLS, **trava trimestral de 90 dias** por vídeo (INV-RC-7/GR-029), **visibilidade estrita** a moradores apenas durante votação de fornecedor em assembleia (Req 57, Rule 4), **agência de marketing** como ator delegado (não-tenant), **busca** via OpenSearch/Meilisearch, e **recomendações** personalizadas.
>
> Planos globais Stripe-style (`trial | base | plus | pro` — ADR-0032). Admin = `apps/admin` dedicada (D-134 revoga D-058/D-072). Agnosticismo de provedor (D-056) — Mux SDK isolado em `infrastructure/providers/` via interface `IVideoProvider` em `content/domain/providers/`. Lives = admin + Empresa Pro + agência delegada (D-097 Fase 11 revoga D-062/D-063/D-076 que diziam admin-only MVP — detalhes em `06-lms-academia.md`).

---

## 1. Fontes consultadas

Destilação **exaustiva** arquivo-por-arquivo, ordenada por autoridade canônica (`vault/CLAUDE.md §8` hierarchy).

### 1.1 Requirements canônicos (Kiro specs)

- `Development/backend/.kiro/specs/master-sindico/requirements/content.md` (v1.0 `stable`, domain `content`) — **Req 29** (Vídeos Institucionais, 12 tipos, quotas mensais por plano, trava trimestral, visibilidade Rule 4), **Req 30** (Agência Marketing, não-tenant, tokens, restrições), **Req 31** (Busca OpenSearch/Meilisearch), **Req 32** (Recomendações per-user). Fonte primária.
- `Development/backend/.kiro/specs/master-sindico/requirements/cross-domain.md` (v1.0 `stable`) — **Req 64** (Mux `IVideoProvider`), **Req 65** (MinIO/R2 `IStorageProvider`), **Req 68** (Livekit `ILiveProvider`), **Req 103** (Visibilidade Estrita: Rule 4 formalizada, auto-revogação via NATS `assembly.voting.closed` → `video.visibility.revoked`), **Req 102** (antifraude aplicável a uploads).
- `Development/backend/.kiro/specs/master-sindico/requirements/functional-matrix.md` — matriz funcional referencia **Content.md Req 29 quotas de vídeo**; ver §5 Personas.
- `Development/backend/.kiro/specs/master-sindico/requirements/personas-and-plans.md` — tabela "Vídeos Institucionais (mensal)", matriz de acesso `Vídeos empresa (preview 25%)` vs `integral`, `Vídeo-currículo` (Banco de Talentos — fora deste sub-produto, ver `07-banco-de-talentos.md`).
- `Development/Content/requirements.md` — **Req 29 expanded** (limits + visibility), **Req 30** (8 screens MK1–MK8 da Agência Marketing), **Req 93** (Lives — live_manager), **Req 97** (Ebooks/Conteúdo Digital — fora deste doc, overlap com §4), **Req 98.1** (LMS — fora; ver `06-lms-academia.md`).

### 1.2 Domínio e entidades (código Go como fonte de verdade)

- `Development/backend/internal/modules/content/domain/entities/video.go` — aggregate **Video**: campos (id, ownerID, companyID, title, description, videoType, muxAssetID, muxPlaybackID, status, autorizarExibicaoVotacao, canBeReplacedAfter, viewCount, createdBy, createdAt, updatedAt), factories `NewVideo`/`ReconstructVideo`, comandos `MarkReady` (seta `canBeReplacedAfter = now+90d`), `MarkFailed`, `Hide`, `Unhide`, `AuthorizeForVoting`, `RevokeVotingAuth`, `RegisterView`, `CanBeReplaced`. Errors: `ErrVideoTitleLen` (2–200), `ErrVideoInvalidType`, `ErrVideoInvalidOwner`, `ErrVideoNotReady`, `ErrVideoAlreadyHidden`, `ErrVideoNotHidden`, `ErrVideoLocked` (90d lock), `ErrVideoTerminal`, `ErrVideoDescTooLong` (≤2000).
- `Development/backend/internal/modules/content/domain/entities/marketing_agency_link.go` — aggregate **MarketingAgencyLink**: VO `MarketingAgencyPermissions {ManageVideos, ViewAnalytics}`, factories `NewMarketingAgencyLink`/`ReconstructMarketingAgencyLink`, comandos `Accept(agencyUserID)`, `Revoke`, `UpdatePermissions`. Errors: `ErrMarketingAgencyEmptyCompanyID`, `ErrMarketingAgencyInvalidName` (2–200), `ErrMarketingAgencyEmptyEmail`, `ErrMarketingAgencyEmptyToken`, `ErrMarketingAgencyNotPending`, `ErrMarketingAgencyAlreadyRevoked`.
- `Development/backend/internal/modules/content/domain/enums/video.go` — enums **VideoType** (12 valores), **VideoStatus** (processing/ready/failed/hidden; `IsTerminal()` = failed), **MarketingAgencyLinkStatus** (pending/active/revoked).
- `Development/backend/internal/modules/content/domain/entities/video_test.go` e `marketing_agency_link_test.go` — covers: title length, invalid type, description limit, status transitions, lock window.

### 1.3 Aplicação (use cases)

- `Development/backend/internal/modules/content/application/use_cases/video_use_cases.go` — **UploadVideoUseCase** (saga: entity → `IVideoProvider.CreateUploadURL` → `videoRepo.Create`; compensação = `CancelUpload` com log de falha), **MarkVideoReadyUseCase** (webhook Mux → `MarkReady`), **MarkVideoFailedUseCase**, **GetVideoUseCase**, **ListVideosUseCase** (filtro `CompanyID`/`OwnerID`/limit/offset), **HideVideoUseCase**/**UnhideVideoUseCase**, **UpdateVotingAuthUseCase**.
- `Development/backend/internal/modules/content/application/use_cases/marketing_agency_use_cases.go` — **InviteMarketingAgencyUseCase** (gera ID+token), **AcceptMarketingAgencyInviteUseCase** (valida token, seta `agencyUserID`), **UpdateMarketingAgencyPermissionsUseCase**, **RevokeMarketingAgencyUseCase**, **ListMarketingAgenciesUseCase**.
- Testes: `video_use_cases_test.go`, `marketing_agency_use_cases_test.go`, `use_cases_stubs_test.go`.

### 1.4 Infraestrutura (providers, DB, HTTP)

- `Development/backend/internal/modules/content/infrastructure/providers/mux_provider.go` — **MuxProvider** implementa `IVideoProvider` (domain) e `ILiveStreamProvider` (local), wrappa SDK `github.com/muxinc/mux-go v1.1.1`; credenciais `MUX_TOKEN_ID`+`MUX_TOKEN_SECRET`. Métodos: `CreateUploadURL(corsOrigin)` (direct upload PUBLIC playback), `CancelUpload`, `GetAsset`, `GetPlaybackURL` (`https://stream.mux.com/{playbackID}`), `GetThumbnailURL` (`https://image.mux.com/{playbackID}/thumbnail.jpg`), `DeleteAsset`; lives: `CreateLiveStream` (reconnect 60s, RTMP `rtmps://global-live.mux.com:443/app`), `GetLiveStream`, `EndLiveStream`.
- `Development/backend/internal/modules/content/infrastructure/providers/i_live_stream_provider.go` — interface `ILiveStreamProvider` + VO `MuxLiveStream {StreamID, StreamKey, PlaybackID, RTMPIngest, Status}`.
- `Development/backend/internal/modules/content/infrastructure/database/` — `video_repository_impl.go`, `marketing_agency_repository_impl.go`, queries SQL em `queries/videos.sql` e `queries/marketing_agency.sql`, geração sqlc em `generated/`, helpers em `helpers.go`.
- `Development/backend/internal/modules/content/infrastructure/http/routes/` — `video_handler.go`, `marketing_agency_handler.go`, `handlers_common_stubs_test.go`, `video_webhook_test.go` (webhook Mux).

### 1.5 Migrations (partição 400–499 = BC content)

- `Development/backend/internal/shared/providers/database/migrations/00400_create_video_enums.sql` — `CREATE TYPE video_type` (12), `video_status` (4), `marketing_agency_link_status` (3).
- `Development/backend/internal/shared/providers/database/migrations/00401_create_videos.sql` — tabela `videos` (id TEXT, company_id UUID FK, status DEFAULT 'processing', `autorizar_exibicao_votacao BOOLEAN DEFAULT FALSE`, `can_be_replaced_after TIMESTAMPTZ`), índices `idx_videos_company_id`, `idx_videos_status`.
- `Development/backend/internal/shared/providers/database/migrations/00402_create_marketing_agency_links.sql` — tabela `marketing_agency_links` (token UNIQUE, perm_manage_videos/perm_view_analytics BOOLEAN), índice `idx_marketing_agency_links_company_id`.

### 1.6 Client-vault (Obsidian destilado)

- `Obsidian Vault/Clients/MasterSindico/02 - Domínios e Requisitos/Domínio - Conteúdo.md` (v3.0, 2026-03-10, `em-revisão`) — **Req 29** (7 critérios de aceite, quotas, 12 tipos, 5 status *com `deleted` adicional que não existe no enum Go*), Req 30 (Agência delegada), Req 30.1–30.3 (gestão usuários empresa + tracking submissões — overlap com BC `commercial`), Req 31–32 (busca, recomendações), Req 97 (Ebooks PDF/EPUB), Req 98.1 (LMS — fora).
- `Obsidian Vault/Clients/MasterSindico/01 - Arquitetura de Sistema/System Architecture - Content.md` (status `Not Started`) — 4 módulos: **VideoModule** (EXISTENTE — expandir), ContentInteractionModule (NOVO: likes/ratings/fórum), AcademiaModule (NOVO), TalentBankModule (NOVO, Decision 6, 11 telas morador + 7 funcs empresa). Este sub-produto cobre **VideoModule**; interaction/academia/talent estão em outros sub-produtos.

### 1.7 Arquitetura e decisões v2

- `master-sindico-v2/STATE.md` — **D-056** (agnosticismo de provedor), **D-058** (admin = role privilegiada), **D-062** (Lives admin-only MVP), **D-066/D-067** (abolir N1/N2/N3 em favor de `trial/base/plus/pro`).
- `master-sindico-v2/02-architecture/adr/0032-global-plans-stripe-style.md` — tiers universais.
- `master-sindico-v2/04-requirements/global.md` — **GR-029** (lock 90d em vídeo).

### 1.8 Pendências de leitura

- Canvas Obsidian `Dominio Conteudo.canvas` e `Visibilidade Videos.canvas` — arquivos binários; conteúdo semântico já absorvido via `Domínio - Conteúdo.md` §"Regra Crucial" e rotas de API.
- `Development/Content/03-Modulos/` e `02-Jornadas/Inventario-Telas.md` — não abertos nesta destilação (evitar overhead; canônico já coberto por `Development/Content/requirements.md`). Registrado como **G-CV-01** (gap de fontes secundárias).

---

## 2. Propósito

Oferecer à empresa prestadora um **canal de showcase institucional** (pitch vídeo + portfólio + depoimentos + tutoriais técnicos + tour virtual + casos de sucesso) **dentro do ambiente condominial**, substituindo tráfego para YouTube/Instagram (que expõem propaganda concorrente) e reforçando a relação com o síndico contratante.

Para o **síndico**, a biblioteca serve como **ferramenta de due diligence**: antes de aprovar um orçamento, o síndico abre a biblioteca da empresa e avalia profissionalismo, equipe, casos de sucesso, certificações técnicas. Para o **morador**, vídeos de empresas são **blindados por default** (INV-062 — Rule 4) — só aparecem em contexto de assembleia quando `autorizar_exibicao_votacao = true` e a votação está ativa.

Para a **agência de marketing** (ator delegado), este é o **único domínio acessível** — ela opera em nome da empresa, faz upload de vídeos, monitora analytics, mas não vê Connect Me, dados de síndicos, propostas, moradores ou dossiês (Req 30 restrições). É a tese comercial "empresa contrata agência audiovisual → agência cadastra vídeo profissional no Master Síndico → vídeo vira ativo da empresa no app".

**Missão**: transformar vídeo em ativo institucional auditável, com trava anti-remontagem (lock 90d — GR-029), visibilidade cirurgicamente controlada (Rule 4), e infra de streaming adaptativa (HLS via Mux) sem acoplar o domínio ao provedor (D-056).

---

## 3. Não-escopo (o que **NÃO** é este sub-produto)

| Tema | Onde está | Motivo |
|------|-----------|--------|
| Vídeo-currículo do morador (Banco de Talentos) | `07-banco-de-talentos.md` | Entidade distinta (`curriculum_videos`, 90s, Mux compartilhado mas contexto Talent) |
| Vídeo-aulas de cursos (LMS) | `06-lms-academia.md` | Aggregate `Course`/`Lesson`; bounded context `content` mas feature separada |
| Lives de Academia (webinars, palestras) | `06-lms-academia.md` | D-062 admin-only MVP; usa `ILiveStreamProvider` mas contexto LMS |
| Assembleia ao vivo (Req 60, Livekit) | `05-assembleia-live.md` | `ILiveProvider` (Req 68), não `IVideoProvider`; WebRTC e não HLS |
| Ebooks / PDFs / EPUBs (Req 97) | `06-lms-academia.md` §Biblioteca | Storage `IStorageProvider` (MinIO/R2), não Mux |
| Fórum, likes, ratings (Req 95) | `06-lms-academia.md` §Fórum | `ContentInteractionModule` (aggregate separado) |
| Busca full-text (Req 31) | **Parcial aqui**: índice `videos` + sync NATS. Implementação do search engine é infra transversal — ver `02-architecture/search/opensearch.md` (gap G-CV-02) | `OpenSearch` é provedor; busca é consumidora de `video.created`/`video.updated` |
| Recomendações cross-produto (Req 32) | **Parcial aqui**: `video_interactions`. Engine (CTR + materialized view) é infra transversal | Job noturno separado em `06-execution/jobs/recommendations.md` (gap G-CV-03) |
| Vídeo institucional do síndico (Req 6.2, trava trimestral de perfil) | `01-governanca-institucional.md` §Perfil Síndico | Mesma mecânica de lock 90d (GR-029), mas aggregate é `SyndicProfile` |
| Storage bruto (MinIO/R2) para vídeos não-Mux | BC `storage` / `IStorageProvider` (Req 65) | Mux já resolve storage HLS; R2 só para thumbnails, ebooks e fallback |
| Aprovação/moderação editorial de vídeo da empresa | Req 30.2 (tracking de submissões) — BC `commercial` parcial, ver `10-marketing-agencias.md` §Submissões | Status editorial (rascunho/enviado/aprovado/publicado) vive em `commercial.submission_tracking`, não em `content.videos` |
| Analytics de funil / dashboards da agência | Req 45 (Dashboard Agência Marketing) | Cross-domain, consolidado em `09-compliance.md` ou dashboard dedicado — ver `10-marketing-agencias.md` |

---

## 4. Tipos de conteúdo (resumo)

### 4.1 Vídeos institucionais — 12 tipos canônicos (enum `video_type`)

Fonte: `Development/backend/internal/modules/content/domain/enums/video.go` (código Go é **fonte de verdade**); migration `00400_create_video_enums.sql` em paridade.

> ⚠️ **Discrepância doc vs código**: `content.md` Req 29 §"12 Tipos" lista textos humanos diferentes em 5 casos (ex.: "Notícia/comunicado", "Evento registrado", "Consultoria", "Webinar", "Outro"), não presentes no enum Go. A versão **código é canônica** (matches migration). Documentado como **G-CV-04** (divergência doc→spec).

| # | Enum Go (`VideoType`) | Label pt-BR | Uso típico | Notas |
|---|----------------------|-------------|-----------|-------|
| 1 | `apresentacao_empresa` | Apresentação da empresa | Pitch institucional 1–2 min | Default para empresas novas |
| 2 | `portfolio_projetos` | Portfólio de projetos | Showcase de obras/serviços realizados | Substitui "portfólio de serviços" do doc |
| 3 | `depoimento_cliente` | Depoimento de cliente | Síndico/condomínio atestando qualidade | Sujeito a validação por síndico citado (Req 30.2) |
| 4 | `tutorial_servico` | Tutorial / como fazer | Demonstração de processo técnico | Bom para SEO interno e educação |
| 5 | `apresentacao_equipe` | Apresentação da equipe | Time técnico, responsáveis, credenciais | Alinha com Req 6.3 certificações |
| 6 | `case_de_sucesso` | Case de sucesso | Antes/depois, métricas, resultados | Sujeito a aprovação do condomínio-case |
| 7 | `servico_especializado` | Serviço especializado | Nicho: limpeza caixa d'água, CFTV, etc. | Usa categorias `service_category` de commercial |
| 8 | `obra_em_andamento` | Obra em andamento | Registro ao vivo de serviço em execução | Ligado a `execution_records` (Req 23) |
| 9 | `certificacao_tecnica` | Certificação técnica | Exibição de CRT, NR-10, NR-35, ISO | Autodeclarada (Req 41-42 disclaimer) |
| 10 | `proposta_valor` | Proposta de valor | USP/diferencial competitivo | Marketing puro |
| 11 | `tour_virtual` | Tour virtual | Walkthrough de instalações/escritório | Boa prática para empresas físicas |
| 12 | `treinamento_sindico` | Treinamento para síndico | Capacitação breve (1–5 min) de síndicos sobre uso do serviço | Fronteira com LMS Treinamentos — aqui é conteúdo **da empresa**, não da Academia |

### 4.2 Status (enum `video_status`)

Fonte: `domain/enums/video.go` + `00400_create_video_enums.sql`.

| Status | Terminal? | Significado | Transições permitidas |
|--------|-----------|-------------|----------------------|
| `processing` | ❌ | Mux está processando upload (HLS encoding) | → `ready` (webhook `video.asset.ready`), → `failed` (webhook erro) |
| `ready` | ❌ | Disponível para exibição; HLS pronto; lock 90d armado em `can_be_replaced_after` | → `hidden` (`Hide()`) |
| `hidden` | ❌ | Ocultado manualmente (empresa ou agência) — mantém no banco, sem visibilidade | → `ready` (`Unhide()`) |
| `failed` | ✅ **terminal** | Processamento Mux falhou; upload descartado | nenhuma (sem retorno) |

> ⚠️ **Doc diverge do código**: `Domínio - Conteúdo.md` cita status `deleted` ("5 status"); enum Go tem **4 status sem `deleted`**. Soft-delete não está implementado como status — provavelmente exclusão seria `DELETE` físico ou coluna `deleted_at` separada (não existe na migration). Registrado como **G-CV-05** (soft-delete ausente).

### 4.3 Agência de marketing — status (enum `marketing_agency_link_status`)

| Status | Significado | Transições |
|--------|-------------|-----------|
| `pending` | Convite criado, token emitido, agência ainda não aceitou | → `active` (`Accept`), → `revoked` (`Revoke`) |
| `active` | Agência aceitou o vínculo; pode operar conforme `Permissions` (manage_videos, view_analytics) | → `revoked` (`Revoke`) |
| `revoked` | Vínculo encerrado — token inválido, acesso zerado | (terminal) |

---

## 5. Personas, planos e quotas

### 5.1 Matriz persona × capacidade (Conteúdo & Vídeos)

Personas Zitadel (`personas-and-plans.md`): `syndic | resident | enterprise | marketing | local_company | admin`.

| Capacidade | syndic | resident | enterprise | marketing | local_company | admin |
|-----------|:------:|:--------:|:----------:|:---------:|:-------------:|:-----:|
| Upload de vídeo institucional próprio | — | — | ✅ (tier `plus`+) | ✅ via `MarketingAgencyLink.active` + `ManageVideos` | ❌ (vídeos em `local_partners` não usam este BC) | ✅ (override) |
| Editar metadados (title/description/type) antes de `ready` | — | — | ✅ | ✅ (se delegada) | — | ✅ |
| `Hide`/`Unhide` | — | — | ✅ | ✅ (se delegada) | — | ✅ |
| Assistir vídeo (integral) | ✅ (tier `plus`+ — Req 29 "Síndicos sempre acesso completo" + matriz) | ⚠️ **apenas via `video_visibility_grants` ativo** (Rule 4 / Req 103) | ✅ (próprios) | ✅ (da empresa vinculada) | — | ✅ |
| Assistir preview 25% (paywall Base) | ✅ (tier `base`) | ❌ (bloqueado default) | — | — | — | ✅ |
| `AuthorizeForVoting` (seta `autorizar_exibicao_votacao`) | — | — | ✅ | ✅ | — | ✅ |
| Convidar agência (`InviteMarketingAgency`) | — | — | ✅ (só admin da empresa — role `company_admin`) | — | — | ✅ |
| Aceitar convite (`AcceptMarketingAgencyInvite`) | — | — | — | ✅ (fluxo de onboarding) | — | ✅ |
| Revogar agência (`Revoke`) | — | — | ✅ | — | — | ✅ |
| Ler analytics (`views`, completion, retention) | — (só vê as métricas públicas) | — | ✅ | ✅ (se `ViewAnalytics`) | — | ✅ |
| Dashboard Agência Marketing (Req 45) | — | — | — | ✅ | — | ✅ |
| Configurar webhook Mux | — | — | — | — | — | ✅ (role-only) |
| Hard delete (`DeleteAsset`) | — | — | ⚠️ (com dupla confirmação; gap G-CV-06 não há use case) | — | — | ✅ |

Fonte da coluna `resident` e paywall 25%: `personas-and-plans.md` §"Vídeos Institucionais" linhas 116–117.

### 5.2 Quotas mensais de upload por plano (Stripe-style)

Original `content.md` Req 29 usa slugs N1/N2/N3/Plus/Pro. **ADR-0032 converte**:

| Plano (ADR-0032) | Persona dominante | Uploads/mês | Fonte original (legado) |
|------------------|-------------------|:-----------:|-------------------------|
| `trial` (qualquer persona) | trial time-boxed | 0 (visualização apenas) | Extensão ADR-0032 — trial não faz upload |
| `base` (syndic) | Síndico básico | **4** | Síndico N1 (content.md) |
| `plus` (syndic) | Síndico intermediário | **8** | Síndico N2 |
| `pro` (syndic) | Síndico profissional | **12** | Síndico N3 |
| `plus` (enterprise) | Empresa Plus | **8** | Empresa Plus |
| `pro` (enterprise) | Empresa Pro | **12** | Empresa Pro |

> ⚠️ **Nota**: a noção de "síndico fazendo upload de vídeo institucional" precisa ser cuidadosamente diferenciada — é o vídeo **de perfil profissional do síndico** (Req 6.2, `01-governanca-institucional.md`), distinto do vídeo institucional da empresa (Req 29 desta spec). As quotas acima consolidam ambos para fins de faturamento, mas a tabela `videos` atual só guarda `company_id` (FK obrigatória). Upload por síndico profissional precisa de schema adicional OU vira `company_id` do próprio síndico-como-empresa (Req 6.2 empresa unipessoal). Registrado como **G-CV-07** (modelagem ambígua síndico-profissional vs empresa).

### 5.3 Controle de quota (implementação)

**Estado atual (código)**: `UploadVideoUseCase` em `video_use_cases.go` **NÃO verifica quota**. A validação de `len(uploads_no_mês) < quota_do_plano` está ausente. Registrado como **G-CV-08** (enforcement de quota não implementado).

**Design alvo**:
- Service `QuotaService.CheckUploadQuota(ctx, companyID, planTier)` invocado antes de `videoProvider.CreateUploadURL`.
- Count via `videoRepo.CountByCompanyAndMonth(ctx, companyID, year, month)`.
- Erro `ErrQuotaExceeded` (application layer) → HTTP 429.
- Reset no 1º dia do mês (00:00 UTC ou TZ condomínio — decidir em **D-pendente**).

---

## 6. Pipeline de upload & processamento (Mux)

### 6.1 Fluxo end-to-end (saga resiliente)

```
[1] Frontend (empresa ou agência)
     │  POST /v1/videos  {title, description, video_type, company_id}
     ▼
[2] video_handler.go → UploadVideoUseCase.Execute(in)
     │
     ├─[2.1] entities.NewVideo(...)           ← valida regras de invariante
     │        (status = processing, id gerado)
     │
     ├─[2.2] videoProvider.CreateUploadURL()  ← Mux SDK: DirectUploadsApi.CreateDirectUpload
     │        → {uploadURL, uploadID}         ← playback_policy = PUBLIC
     │
     ├─[2.3] videoRepo.Create(ctx, video)     ← Postgres INSERT
     │        │
     │        └─ falhou? ──► saga compensation:
     │                        videoProvider.CancelUpload(uploadID)
     │                        (falha de cancel é logada mas não sobrescreve erro original)
     │
     └─[3] retorna {video_id, upload_url, upload_id}

[4] Frontend faz PUT direto no uploadURL do Mux
     │  (o backend NÃO passa o bytestream — upload direto ao provider)
     ▼
[5] Mux processa → gera HLS adaptativo (480p/720p/1080p) → webhook
     │
     ▼
[6] POST /v1/videos/webhooks/mux   (handler: video_webhook_test.go espelha)
     │  event: video.asset.ready | video.asset.errored
     │  signed: HMAC SHA256 com MUX_WEBHOOK_SECRET
     ▼
[7.a] video.asset.ready:
       MarkVideoReadyUseCase.Execute({videoID, muxAssetID, muxPlaybackID})
         └─ video.MarkReady()           ← status = ready
                                        ← canBeReplacedAfter = now + 90d (GR-029)
[7.b] video.asset.errored:
       MarkVideoFailedUseCase.Execute(videoID)
         └─ video.MarkFailed()          ← status = failed (terminal)
```

### 6.2 Saga de compensação (UoW + compensação)

| Passo | Operação | Falha → |
|-------|---------|---------|
| 1 | Validar entidade local | retorna 400 imediato (sem chamada externa) |
| 2 | `videoProvider.CreateUploadURL` (rede externa) | retorna 502 (bubbled up); nada a compensar |
| 3 | `videoRepo.Create` (Postgres) | **compensa**: `videoProvider.CancelUpload(uploadID)`; se cancel falha → log `saga compensation falhou` com `original_error` + `cancel_error`; retorna erro original |

**Invariante**: nunca sobrescrever o erro original com erro de compensação — usuário precisa saber a causa-raiz; reconciliação de órfãos Mux é job de background.

**Nota de implementação** (`video_use_cases.go` lines 70–84):
> "Saga: se persistir localmente falhar, o upload fica órfão no provider. Compensar cancelando o direct upload. Falha de compensação é logada mas não sobrescreve o erro original (usuário precisa saber o que deu errado; reconciliação do órfão é problema de background)."

### 6.3 Webhook Mux — segurança

**Crítico** (pentester STRIDE — **S**poofing):

- Validar header `Mux-Signature: t=<timestamp>,v1=<hmac>` contra `MUX_WEBHOOK_SECRET`.
- Rejeitar timestamps `> 5min` de skew (replay mitigation).
- Idempotência: `UPSERT` em `mux_webhook_events` com PK = (asset_id, event_type, timestamp) — não aplicar mesmo evento duas vezes.
- Whitelist de event types: `video.asset.ready`, `video.asset.errored`, `video.asset.deleted`, `video.upload.cancelled`, `video.upload.asset_created`.
- Rate limit: 100 req/min por IP (Mux publica faixas de IP; prefira-se validação por signature e não IP).

Gap: implementação do handler de webhook (só `video_webhook_test.go` está visível listando; corpo não lido). Registrado como **G-CV-09** (webhook handler secret validation a verificar).

### 6.4 Storage & reprodução

| Função | URL pattern | Expiração | Notas |
|--------|-------------|-----------|-------|
| HLS playback (stream adaptativo) | `https://stream.mux.com/{playbackID}.m3u8` | ∞ (PUBLIC policy atual) | **Risco** (**I**nformation disclosure): playback policy PUBLIC permite que qualquer pessoa com o ID veja o vídeo. Mitigação: tornar `SIGNED` para vídeos de empresa com `autorizar_exibicao_votacao=false` — **G-CV-10**. |
| Thumbnail JPG | `https://image.mux.com/{playbackID}/thumbnail.jpg` | ∞ | idem acima; thumbnails vazam mesmo com stream SIGNED |
| Storyboard/GIF preview | Mux padrão (não invocado no código) | ∞ | **G-CV-11** não implementado; útil para paywall 25% preview |
| Download original (MP4) | não exposto | — | Mux oferece; propositalmente não habilitado (evita redistribuição) |

### 6.5 Trava trimestral (lock 90d)

**Invariante RC-7** (alias GR-029): vídeo `ready` não pode ser **substituído** antes de 90 dias desde o último `MarkReady`.

Implementação (`entities/video.go`):
```go
func (v *Video) MarkReady(muxAssetID, muxPlaybackID string) error {
    if v.status.IsTerminal() { return ErrVideoTerminal }
    if v.status == enums.VideoStatusReady { return nil }  // idempotente
    next := time.Now().Add(90 * 24 * time.Hour)
    v.muxAssetID = muxAssetID
    v.muxPlaybackID = muxPlaybackID
    v.status = enums.VideoStatusReady
    v.canBeReplacedAfter = &next
    v.updatedAt = time.Now()
    return nil
}

func (v *Video) CanBeReplaced() bool {
    if v.canBeReplacedAfter == nil { return true }
    return time.Now().After(*v.canBeReplacedAfter)
}
```

> ⚠️ **Gap G-CV-12**: `CanBeReplaced()` está implementado mas **não é chamado por nenhum use case atual** — não há `ReplaceVideoUseCase`. O erro `ErrVideoLocked` está definido mas nunca retornado. A trava existe em dado mas não é enforced. Precisa de use case `ReplaceVideo(videoID)` que valide `CanBeReplaced()` antes de permitir novo upload sobrescrevendo o anterior.

### 6.6 Análise de vídeo (métricas)

| Métrica | Obtida via | Armazenamento | Uso |
|---------|-----------|---------------|-----|
| `views` | `RegisterView()` contador local (incremental) | `videos.view_count` | Exibição pública, ranking |
| `avg_watch_time` | Mux Data API (pendente) | `video_analytics.avg_watch_time` (tabela futura — G-CV-13) | Engagement |
| `completion_rate` | Mux Data API | idem | Qualidade de conteúdo |
| `retention_rate` | Mux Data API | idem | Edição (onde os viewers desistem) |
| `click_through_rate` | `video_interactions` (tabela futura — G-CV-14) | Req 32 | Recomendações |

**Estado atual**: só `view_count` está no schema. `video_analytics` e `video_interactions` (Req 29 notas de implementação + Req 32) **não foram criados**. Registrado como **G-CV-13/14** (tabelas de analytics pendentes).

---

## 7. Aggregates e modelo de domínio

### 7.1 Video (aggregate root)

Fonte: `Development/backend/internal/modules/content/domain/entities/video.go`.

```
Video (aggregate root)
├── Identidade: id (TEXT), companyID (UUID FK)
├── Autoria: ownerID (TEXT), createdBy (TEXT)
├── Conteúdo: title (2–200), description (≤2000), videoType (enum)
├── Mux: muxAssetID (TEXT), muxPlaybackID (TEXT)
├── Ciclo: status (VideoStatus), canBeReplacedAfter (nullable timestamptz)
├── Visibilidade: autorizarExibicaoVotacao (bool default false)
├── Métricas: viewCount (int default 0)
└── Timestamps: createdAt, updatedAt
```

**Comandos** (methods expostos):
| Método | Pré-condição | Pós-condição | Erro |
|--------|--------------|--------------|------|
| `NewVideo(...)` | title 2–200, ownerID não-vazio, type válido, desc ≤2000 | status=processing, timestamps agora | `ErrVideoTitleLen`, `ErrVideoInvalidOwner`, `ErrVideoInvalidType`, `ErrVideoDescTooLong` |
| `ReconstructVideo(...)` | — (usado por repo) | restaura do banco sem validação | — |
| `MarkReady(assetID, playbackID)` | status ≠ terminal (failed) | status=ready, `canBeReplacedAfter=now+90d` | `ErrVideoTerminal` |
| `MarkFailed()` | — | status=failed (terminal) | — |
| `Hide()` | status=ready | status=hidden | `ErrVideoNotReady` |
| `Unhide()` | status=hidden | status=ready | `ErrVideoNotHidden` |
| `AuthorizeForVoting()` | — | `autorizarExibicaoVotacao=true` | — |
| `RevokeVotingAuth()` | — | `autorizarExibicaoVotacao=false` | — |
| `RegisterView()` | — | `viewCount++`, updatedAt=now | — |
| `CanBeReplaced()` | — | bool (true se lock expirou ou nunca armado) | — |

### 7.2 MarketingAgencyLink (aggregate root)

Fonte: `entities/marketing_agency_link.go`.

```
MarketingAgencyLink (aggregate root)
├── Identidade: id (TEXT), companyID (UUID FK), agencyUserID (TEXT, set on Accept)
├── Dados: agencyName (2–200), email, token (UNIQUE)
├── Ciclo: status (pending | active | revoked)
├── Value Object: permissions (MarketingAgencyPermissions {ManageVideos, ViewAnalytics})
├── Linkage: linkedAt (nullable timestamptz, set on Accept)
├── Autoria: createdBy
└── Timestamps: createdAt, updatedAt
```

**Comandos**:
| Método | Pré-condição | Pós-condição | Erro |
|--------|--------------|--------------|------|
| `NewMarketingAgencyLink(...)` | companyID/agencyName(2–200)/email/token não-vazios | status=pending | 4 ErrX |
| `Accept(agencyUserID)` | status=pending | status=active, linkedAt=now, agencyUserID=<id> | `ErrMarketingAgencyNotPending` |
| `Revoke()` | status ≠ revoked | status=revoked | `ErrMarketingAgencyAlreadyRevoked` |
| `UpdatePermissions(p)` | — | permissions=p | — |

### 7.3 Relação entre aggregates

```
Company (BC commercial / institutional)
  │
  │ 1:N
  ▼
Video ─────────── videoType ∈ {12 enums}
  │
  │ 0..1 FK
  ▼
AssemblyVotingSession (BC assembly) ── gera ──► VideoVisibilityGrant (BC content, tabela futura G-CV-15)
                                                  │
                                                  │ autoriza morador a ver vídeo durante voting.active
                                                  ▼
Resident (BC identity)

Company 1:N MarketingAgencyLink N:1 AgencyUser (BC identity, role marketing)
```

### 7.4 Tabelas físicas (schema atual + futuras)

| Tabela | Estado | Migration | Colunas-chave |
|--------|--------|-----------|---------------|
| `videos` | ✅ existe | 00401 | `id TEXT PK`, `company_id UUID FK companies(id)`, `status video_status`, `autorizar_exibicao_votacao BOOL`, `can_be_replaced_after TIMESTAMPTZ`, `view_count INT`, indices company_id + status |
| `marketing_agency_links` | ✅ existe | 00402 | `id TEXT PK`, `company_id UUID FK`, `token TEXT UNIQUE`, `status`, `perm_manage_videos`, `perm_view_analytics`, índice company_id |
| `video_analytics` | ❌ **G-CV-13** | (404xx futuro) | `video_id FK`, `date`, `views`, `avg_watch_time`, `completion_rate`, `retention_rate` |
| `video_interactions` | ❌ **G-CV-14** | (404xx) | `video_id FK`, `user_id`, `action ENUM(view/click/share)`, `timestamp` |
| `video_visibility_grants` | ❌ **G-CV-15** | (404xx) | `video_id FK`, `assembly_voting_id FK`, `granted_at`, `revoked_at`, `grant_reason` |
| `mux_webhook_events` | ❌ **G-CV-09** | (404xx) | `asset_id`, `event_type`, `timestamp`, `payload JSONB`, PK composta |
| `marketing_agency_access_logs` | ❌ **G-CV-16** | (404xx) | `link_id FK`, `action`, `video_id`, `timestamp`, `ip` — Req 30 "Audit log de todas as ações" |

---

## 8. Enums canônicos (fonte: código Go)

### 8.1 `VideoType` (12 valores)

```go
// Development/backend/internal/modules/content/domain/enums/video.go
type VideoType string

const (
    VideoTypeApresentacaoEmpresa  VideoType = "apresentacao_empresa"
    VideoTypePortfolioProjetos    VideoType = "portfolio_projetos"
    VideoTypeDepoimentoCliente    VideoType = "depoimento_cliente"
    VideoTypeTutorialServico      VideoType = "tutorial_servico"
    VideoTypeApresentacaoEquipe   VideoType = "apresentacao_equipe"
    VideoTypeCaseDeSucesso        VideoType = "case_de_sucesso"
    VideoTypeServicoespecializado VideoType = "servico_especializado"
    VideoTypeObraEmAndamento      VideoType = "obra_em_andamento"
    VideoTypeCertificacaoTecnica  VideoType = "certificacao_tecnica"
    VideoTypePropostaValor        VideoType = "proposta_valor"
    VideoTypeTourVirtual          VideoType = "tour_virtual"
    VideoTypeTreinamentoSindico   VideoType = "treinamento_sindico"
)
```

> 🐞 **Inconsistência de nomenclatura Go**: identificador `VideoTypeServicoespecializado` (minúscula no `e` de "especializado") destoa do padrão CamelCase do restante (`VideoTypeServicoEspecializado` seria o esperado). String value está correta (`servico_especializado`). Registrado como **G-CV-17** (bug de nomenclatura, impacto zero em runtime).

### 8.2 `VideoStatus` (4 valores, 1 terminal)

```go
type VideoStatus string

const (
    VideoStatusProcessing VideoStatus = "processing"
    VideoStatusReady      VideoStatus = "ready"
    VideoStatusFailed     VideoStatus = "failed"
    VideoStatusHidden     VideoStatus = "hidden"
)

func (s VideoStatus) IsTerminal() bool {
    return s == VideoStatusFailed
}
```

### 8.3 `MarketingAgencyLinkStatus` (3 valores)

```go
type MarketingAgencyLinkStatus string

const (
    MarketingAgencyLinkStatusPending  = "pending"
    MarketingAgencyLinkStatusActive   = "active"
    MarketingAgencyLinkStatusRevoked  = "revoked"
)
```

### 8.4 Paridade enum Go ↔ Postgres

| Enum | Go | Postgres (migration 00400) | Paridade |
|------|-----|----------------------------|----------|
| `video_type` | 12 valores | 12 valores | ✅ |
| `video_status` | 4 | 4 | ✅ |
| `marketing_agency_link_status` | 3 | 3 | ✅ |

### 8.5 Enums pendentes (docs mencionam, código não tem)

| Enum sugerido | Fonte | Status |
|---------------|-------|--------|
| `token_scope` (videos:upload, videos:edit, analytics:read) | Req 30 "Escopo" | ❌ **G-CV-18**: esquema atual usa 2 BOOLEANs (`perm_manage_videos`, `perm_view_analytics`), não enum. Suficiente para MVP mas extensível via JSONB ou tabela `marketing_agency_scopes`. |
| `video_access_level` (free, plus, pro) | Req 97 "access_level" (ebooks) | ❌ — mas é para `digital_content`, não para `videos`. Fora deste sub-produto. |

---

## 9. Integração Mux (provider agnostic)

### 9.1 Interface canônica `IVideoProvider` (domain)

Convenção DDD + D-056: o domínio **define** o contrato; infraestrutura implementa.

```go
// Localização esperada: domain/providers/i_video_provider.go
package providers

type IVideoProvider interface {
    CreateUploadURL(ctx context.Context, corsOrigin string) (uploadURL, uploadID string, err error)
    CancelUpload(ctx context.Context, uploadID string) error
    GetAsset(ctx context.Context, assetID string) (*VideoAsset, error)
    GetPlaybackURL(playbackID string) string
    GetThumbnailURL(playbackID string) string
    DeleteAsset(ctx context.Context, assetID string) error
}

type VideoAsset struct {
    AssetID    string
    PlaybackID string
    Status     string   // string "plain" para desacoplar do enum interno do provider
}
```

### 9.2 Implementação `MuxProvider` (infrastructure)

Arquivo: `Development/backend/internal/modules/content/infrastructure/providers/mux_provider.go`.

- **SDK**: `github.com/muxinc/mux-go v1.1.1`.
- **Credenciais**: ambiente `MUX_TOKEN_ID`, `MUX_TOKEN_SECRET` (basic auth).
- **Config**: `muxgo.NewConfiguration(muxgo.WithBasicAuth(...))`.
- **Asserts**: `var _ domainproviders.IVideoProvider = (*MuxProvider)(nil)` + `var _ ILiveStreamProvider = (*MuxProvider)(nil)`.

**Responsabilidades implementadas**:

| Método | SDK call | Retorno | Observações |
|--------|----------|---------|-------------|
| `CreateUploadURL(corsOrigin)` | `DirectUploadsApi.CreateDirectUpload({CorsOrigin, NewAssetSettings:{PlaybackPolicy: PUBLIC}})` | `{resp.Data.Url, resp.Data.Id}` | PUBLIC — ⚠️ ver G-CV-10 |
| `CancelUpload(uploadID)` | `DirectUploadsApi.CancelDirectUpload(uploadID)` | error | Idempotência imperfeita: Mux erra se já completado/cancelado; caller deve tratar como warning |
| `GetAsset(assetID)` | `AssetsApi.GetAsset(assetID)` | `{AssetID, PlaybackID (primeiro), Status}` | Captura apenas 1º playback ID |
| `GetPlaybackURL(playbackID)` | — (format string) | `https://stream.mux.com/{playbackID}` | **Sem `.m3u8`** — caller concatena |
| `GetThumbnailURL(playbackID)` | — | `https://image.mux.com/{playbackID}/thumbnail.jpg` | Parâmetros time/width/height futuros — G-CV-19 |
| `DeleteAsset(assetID)` | `AssetsApi.DeleteAsset(assetID)` | error | Nenhum use case chama (ver G-CV-06) |

**Lives (interface local `ILiveStreamProvider`)**:

| Método | SDK call | Retorno | Uso |
|--------|----------|---------|-----|
| `CreateLiveStream()` | `LiveStreamsApi.CreateLiveStream({PlaybackPolicy:PUBLIC, ReconnectWindow:60})` | `MuxLiveStream{StreamID, StreamKey, PlaybackID, RTMPIngest:"rtmps://global-live.mux.com:443/app", Status}` | Admin agenda live LMS (D-062) |
| `GetLiveStream(id)` | `GetLiveStream(id)` | idem | Polling de status |
| `EndLiveStream(id)` | `DeleteLiveStream(id)` | error | Encerramento manual |

**Nota arquitetural**: `ILiveStreamProvider` está em `infrastructure/providers/i_live_stream_provider.go` — isso **viola Clean Architecture** (interface deveria estar em `domain/`). Registrado como **G-CV-20** (tech debt; mover para `domain/providers/`). O `IVideoProvider` está correto (em `domain/providers/`).

### 9.3 Segredos e observabilidade

| Segredo | Onde | Rotacionável? | Notas |
|---------|------|:-------------:|-------|
| `MUX_TOKEN_ID` | env | ✅ | Identidade da conta Mux |
| `MUX_TOKEN_SECRET` | env (Vault/KMS) | ✅ | Nunca em log; nunca commit |
| `MUX_WEBHOOK_SECRET` | env (Vault/KMS) | ✅ | Valida assinatura HMAC do webhook |
| `MUX_SIGNING_KEY_ID` (futuro SIGNED playback) | env | ✅ | JWT RS256 para URL pré-assinada — G-CV-10 |
| `MUX_SIGNING_KEY_PRIVATE` (futuro) | env (Vault) | ✅ | PEM; nunca log |

**Observabilidade**:
- Logger estruturado (`pkg/logger`): `log.Info("video upload iniciado", F("video_id"), F("upload_id"), F("company_id"), F("video_type"))`.
- Métricas Prometheus sugeridas (gap G-CV-21, não implementadas): `content_video_upload_total{status}`, `content_video_processing_duration_seconds`, `content_mux_api_duration_seconds{operation,status}`, `content_mux_api_errors_total{operation,code}`, `content_video_quota_exceeded_total{plan_tier}`.

### 9.4 Troca de provedor (D-056 agnostic)

**Cenários**:
1. **Cloudflare Stream** (alternativa): implementar `CloudflareStreamProvider` em `infrastructure/providers/`, registrar via `wire`/DI. Zero mudança em `video_use_cases.go`.
2. **Self-hosted (Bitmovin/Ozone)** (futuro enterprise): idem.
3. **Fallback multi-provider**: envelope provider que tenta primário → fallback; não implementado — G-CV-22.

**Custos de troca** (mitigar vendor lock-in):
- Playback URLs são **específicas do provedor** (`stream.mux.com` vs `customer-*.cloudflarestream.com`). A migração requer re-processar assets, não só swap do provider. Mitigação: armazenar **apenas `playback_id`** e derivar URL via `provider.GetPlaybackURL(id)` — já é o caso no schema (`mux_playback_id` — renomear para `playback_id` genérico? G-CV-23).

---

## 10. Telas e jornadas (MK1–MK8 Agência + Empresa)

### 10.1 Jornada Empresa (upload de vídeo institucional)

Referência: `requirements.md` §Requirement 29 + matriz de telas. **Gap G-CV-24**: telas de empresa não numeradas canonicamente — inferidas aqui como PL1–PL4 (PL = Plataforma empresa).

| Tela | Propósito | Estado | Campos | Ação |
|------|-----------|--------|--------|------|
| **PL1 — Biblioteca de Vídeos** | Lista de vídeos da empresa | — | Tabela: thumbnail, title, video_type, status, views, created_at, lock remaining | CTA "Novo vídeo" + filtros por status/type |
| **PL2 — Novo Vídeo (metadados)** | Pré-cadastro antes de upload | — | `title` (2–200), `description` (≤2000), `video_type` (select 12), `autorizar_exibicao_votacao` (checkbox explicado com aviso Rule 4) | Submit → backend cria entity + upload URL |
| **PL3 — Upload (Mux direct)** | Browser → Mux via `uploadURL` | processing | Progresso de upload (percentage), cancel button → `CancelUpload` | Feedback "Processando… avisaremos quando ficar pronto" |
| **PL4 — Detalhe/Analytics do Vídeo** | Pós-ready | ready/hidden | Player HLS (hls.js), analytics (views, retention chart), ações: Hide/Unhide, Autorizar para votação toggle, "Substituir vídeo" (desabilitado se `!CanBeReplaced()`, mostra data de liberação) | Hide / Unhide / Replace (após lock) |

### 10.2 Jornada Agência de Marketing (8 telas canônicas MK1–MK8)

Referência: `requirements.md` Req 30 "8 screens MK1-MK8" + `Development/Content/contents/JORNADA DA EMPRESA DE MARKETING.pdf` (citado mas não lido nesta destilação — G-CV-25). Inferência baseada em uso case set do módulo.

| Tela | Persona | Propósito | Inputs principais |
|------|---------|-----------|-------------------|
| **MK1 — Landing pública agência** | `marketing` não-autenticado | "Você foi convidado por {Empresa X}"; CTA "Aceitar convite" | token (query param) |
| **MK2 — Cadastro / login agência** | `marketing` | Cria conta Zitadel (role `marketing`) OU login existente | email, senha (Zitadel); token persistido |
| **MK3 — Aceite de vínculo** | `marketing` | Confirma termos e permissões recebidas | `AcceptMarketingAgencyInviteUseCase({token, agencyUserID})` |
| **MK4 — Dashboard agência** | `marketing` | Lista de empresas vinculadas (1 agência : N empresas) | Query `ListMarketingAgencies` por `agency_user_id` |
| **MK5 — Biblioteca de vídeos (contexto empresa)** | `marketing` | Lista vídeos da empresa selecionada (escopo `ManageVideos`) | `ListVideosUseCase{CompanyID}` |
| **MK6 — Novo vídeo (em nome da empresa)** | `marketing` | Idêntico a PL2 mas `owner_id = agency_user_id` e `company_id = empresa` | `UploadVideoUseCase` |
| **MK7 — Analytics consolidado (cross-empresa)** | `marketing` | Métricas agregadas; requer `ViewAnalytics` | Futuro — G-CV-26 (dashboard Req 45) |
| **MK8 — Convites e vínculos ativos** | `marketing` | Gestão própria: empresas que me convidaram, status (pending/active/revoked) | — |

### 10.3 Jornada Síndico (assistir vídeo de empresa no Connect Me / votação)

| Tela | Contexto | Visibilidade |
|------|----------|--------------|
| **Perfil da empresa no Connect Me** | Síndico tier `plus`+ visualizando empresa | Vídeos `ready` da empresa — integral (Req 29 "Síndicos sempre acesso completo") |
| **Síndico tier `base`** | Idem | **Preview 25%** (paywall) — Req 29 "Paywall Base: truncar stream aos 25%" — G-CV-10 requer SIGNED URL com playback restrito |

### 10.4 Jornada Morador (contexto restrito — votação)

| Tela | Condição de aparição | Implementação |
|------|---------------------|----------------|
| **Pauta de assembleia — Votação de fornecedor aberta** | Morador autenticado + assembleia `in_progress` + pauta de votação de fornecedor ativa + `video.autorizar_exibicao_votacao=true` | Middleware ABAC valida `video_visibility_grants` (G-CV-15) antes de servir playback |
| **Após `assembly.voting.closed` NATS event** | Grant revogado automaticamente | Morador recebe 403 ao tentar nova request; player se quebra com mensagem "Período de visualização encerrado" |

### 10.5 Jornada Admin (overrides)

Admin (role Zitadel `admin`, D-058) tem override **transversal**:
- Lista global de vídeos (todas as empresas) com filtros de moderação.
- Hard-delete (`DeleteAsset` Mux + DELETE physical videos row) — único uso case a implementar (G-CV-06).
- Moderar/banir vídeo (status custom `banned`? G-CV-27 decisão arquitetural pendente; ou usar `hidden` com flag `moderated_by_admin`).
- Reprocessar webhook (idempotência G-CV-09).
- Visualizar logs de acesso da agência (`marketing_agency_access_logs` G-CV-16).

---

## 11. Regras de negócio (RC-1 … RC-12)

Convenção: RC = Regra de Conteúdo (prefixo local); referenciam invariantes globais (INV-###) e requisitos globais (GR-###).

| ID | Regra | Enforcement | Invariante global |
|----|-------|-------------|-------------------|
| **RC-1** | Título do vídeo: 2–200 chars | `entities.NewVideo` retorna `ErrVideoTitleLen` | — |
| **RC-2** | Descrição ≤ 2000 chars | `entities.NewVideo` retorna `ErrVideoDescTooLong` | — |
| **RC-3** | Owner não-vazio (obriga autoria) | `entities.NewVideo` retorna `ErrVideoInvalidOwner` | INV-034 (auditoria de autor) |
| **RC-4** | `video_type` ∈ enum dos 12 valores canônicos | `VideoType.IsValid()` em `NewVideo` | — |
| **RC-5** | `company_id` é obrigatório (FK) e **sempre aponta para a empresa**, nunca para a agência | Constraint SQL + validação `NewVideo` | INV-068 (agência não é dona) |
| **RC-6** | Upload dispara criação local **antes** de criar upload Mux? **Não** — ordem é: valida entity → cria upload Mux → persiste local; compensação cancela upload Mux se persist falha | Saga em `UploadVideoUseCase` | — |
| **RC-7** | **Trava trimestral 90d**: vídeo `ready` não pode ser substituído por novo upload antes de `can_be_replaced_after` | `CanBeReplaced()` (ainda não chamado — G-CV-12) | **GR-029** (lock 90d global) |
| **RC-8** | `Hide` exige status `ready`; `Unhide` exige status `hidden` | `Hide()`/`Unhide()` retornam `ErrVideoNotReady`/`ErrVideoNotHidden` | — |
| **RC-9** | Status `failed` é **terminal** — nenhuma transição posterior | `VideoStatus.IsTerminal()` + `MarkReady` guard | — |
| **RC-10** | **Visibilidade a morador apenas durante votação** com `autorizar_exibicao_votacao=true` | Middleware ABAC (pendente — G-CV-15 `video_visibility_grants`) | **INV-062** (Rule 4) / **Req 103** |
| **RC-11** | Agência delegada **não é tenant**: token TTL, escopo restrito (manage_videos, view_analytics), revogação imediata | `MarketingAgencyLink` aggregate + middleware de validação de token | INV-069 (delegation boundary) |
| **RC-12** | Preview 25% para tier `base` síndico | Pendente SIGNED playback + truncamento de stream — G-CV-10 | GR-028 (paywall Base) |

### 11.1 Regras de quota (futuras — G-CV-08)

| ID | Regra | Enforcement |
|----|-------|-------------|
| **RC-Q1** | Tier `trial` = 0 uploads (pode listar/assistir mas não publicar) | `QuotaService` |
| **RC-Q2** | Tier `base` síndico = 4 uploads/mês | idem |
| **RC-Q3** | Tier `plus` = 8/mês (síndico ou empresa) | idem |
| **RC-Q4** | Tier `pro` = 12/mês | idem |
| **RC-Q5** | Quota reseta 1º dia do mês (timezone a decidir — D-pendente) | cron job |

### 11.2 Regras de delegação (Agência)

| ID | Regra |
|----|-------|
| **RC-A1** | Agência não acessa: Connect Me (Req 19), deals/propostas, dossiê mandato, dados de síndicos/moradores, execuções (Req 23), comunicados (Req 20) |
| **RC-A2** | Token de convite gerado por **admin da empresa** (`company_admin` role, não confundir com `admin` global) |
| **RC-A3** | Token TTL configurável (default 90d — G-CV-28, não implementado no schema atual) |
| **RC-A4** | Revogação do link invalida imediatamente **todas** as sessões da agência para aquela empresa (não só o token); agência continua ativa para outras empresas |
| **RC-A5** | `Accept` é **one-shot**: após aceite, token não pode ser reutilizado para re-aceitar (status `active` é terminal para fins de Accept) |
| **RC-A6** | Audit log (`marketing_agency_access_logs` G-CV-16) registra cada action da agência: upload, edit, hide, view analytics — com `who (agency_user_id) + when + what (video_id) + ip` |

### 11.3 Regras de visibilidade (ABAC + Rule 4)

Model decision: ABAC (Attribute-Based Access Control) com matriz policy externa (Open Policy Agent / Rego ou Casbin — ver `02-architecture/abac.md`).

**Attributes avaliados para `action = play_video`**:

```
subject.role ∈ {syndic, enterprise, marketing, admin} → permit (com sub-regras)
subject.role = resident →
  check resource.autorizar_exibicao_votacao = true
  AND exists(video_visibility_grants where
              video_id = resource.id
              AND resident_id = subject.id
              AND granted_at ≤ now
              AND (revoked_at IS NULL OR revoked_at > now)
              AND assembly_voting_id.status = 'open')
```

**Ordem de verificação (pentester STRIDE — Elevation)**:
1. Tenant isolation (mesma empresa/organização)? — sempre primeiro.
2. Persona permitida? — matriz 5.1.
3. Rule 4 se `resident`? — grant ativo.
4. Quota/plano cobre preview ou integral? — RC-12.
5. Auditoria: log LGPD (`lgpd_logs`) se resident acessa — Req 39.

---

## 12. Invariantes (INV)

Conformidade com `master-sindico-v2/01-domain/invariants.md` (linkagem futura — G-CV-29 não criar INV-### sem validar colisão).

### 12.1 Invariantes específicas de Conteúdo & Vídeos

| ID (propositivo) | Invariante | Como é preservada |
|------------------|-----------|-------------------|
| **INV-CV-1** | Vídeo é imutável após publicação — edição via adendo (cross-ref Req 25 commercial) | Entidade não expõe setter de `muxAssetID`/`muxPlaybackID` após `MarkReady`; idempotência de `MarkReady` |
| **INV-CV-2 (= GR-029 / RC-7)** | Trava trimestral de 90 dias entre atualizações do mesmo vídeo | `canBeReplacedAfter` armazenado e validado via `CanBeReplaced()` |
| **INV-CV-3 (= Rule 4)** | Morador só vê vídeo de empresa durante votação **e** se `autorizar_exibicao_votacao = true` | ABAC + `video_visibility_grants` + event-driven revogação |
| **INV-CV-4** | Agência de Marketing tem acesso zero a dados de negócio (deals/Connect Me/propostas/etc.) | Scopes restritos ao BC `content`; middleware bloqueia demais rotas |
| **INV-CV-5** | Certificado auto-emitido ao 100% de conclusão (LMS — fora deste sub-produto; listado para completude) | — (ver 06-lms-academia.md) |
| **INV-CV-6** | Vídeo de empresa sempre vinculado a `company_id` (não a `agency_id`) | FK + validação `NewVideo` |
| **INV-CV-7** | Status `failed` é terminal — asset Mux não entra mais no pipeline | `MarkReady` guarda `IsTerminal` |
| **INV-CV-8** | Agência **não é tenant** — não tem organização própria no Zitadel; é user com role `marketing` + `MarketingAgencyLink` | Role Zitadel + ausência de `organization_id` próprio |
| **INV-CV-9** | `view_count` é monotonicamente crescente (nunca decresce) | Método `RegisterView()` só faz `++`; nenhum método decrementa |
| **INV-CV-10** | Link de agência não pode ser aceito duas vezes | `Accept` retorna `ErrMarketingAgencyNotPending` se status ≠ pending |
| **INV-CV-11** | `token` de convite é **UNIQUE** globalmente | Constraint SQL `UNIQUE` em `marketing_agency_links.token` |
| **INV-CV-12** | Ação do admin sempre auditada em `audit_trail` (BC cross-domain Req 37) | Middleware transversal (fora deste BC) |

### 12.2 Invariantes transversais aplicáveis

- **INV-034** (auditoria de autoria): toda criação de vídeo tem `created_by` preenchido.
- **INV-068** (tenant delegation boundary): agência opera em nome de tenant mas não **é** tenant.
- **GR-028** (paywall tier base): preview 25% obrigatório para tier base em conteúdo institucional.
- **GR-029** (lock 90d global em conteúdo institucional): aplicável a Video, SyndicProfileVideo, CurriculumVideo, CompanyProfileVideo.

---

## 13. State machines

### 13.1 `Video.status`

```
            ┌──────────────┐
  upload →  │  processing  │
            └──┬────────┬──┘
               │        │
    video.     │        │  video.asset.errored
    asset.     │        │
    ready      ▼        ▼
            ┌──────┐  ┌────────┐ (terminal)
            │ ready│  │ failed │
            └─┬──┬─┘  └────────┘
      Hide() │  │ Unhide()
             ▼  ▲
           ┌────────┐
           │ hidden │
           └────────┘
```

Invariantes da máquina:
- `processing → ready` via **webhook Mux** exclusivamente (não há MarkReady manual).
- `ready ↔ hidden` via comandos manuais da empresa/agência/admin.
- `failed` é absorvente (sem retorno). Reprocessamento requer novo upload (novo registro `videos`).
- `MarkReady` é **idempotente** sobre `ready` (retorno `nil` se já ready) — importante para tolerância a webhooks duplicados.

### 13.2 `MarketingAgencyLink.status`

```
         ┌─────────┐
invite → │ pending │
         └─┬─────┬─┘
           │     │
  Accept() │     │ Revoke()
           ▼     ▼
        ┌──────┐ ┌────────┐ (terminal)
        │ active│ │revoked │
        └───┬──┘ └────────┘
            │
            │ Revoke()
            ▼
        ┌────────┐
        │ revoked│ (terminal)
        └────────┘
```

Invariantes da máquina:
- Token é válido **apenas em `pending`** (Accept só transita dessa origem).
- Revogação é **idempotente sobre si mesma** — chamada em `revoked` retorna `ErrMarketingAgencyAlreadyRevoked` (não re-transita).
- `UpdatePermissions` é ortogonal ao status: pode mudar em `active`; quando `revoked`, permissions viram irrelevantes (acesso bloqueado upstream).

### 13.3 `VideoVisibilityGrant.status` (futuro — G-CV-15)

```
                 ┌─────────────┐
  voting.open →  │   granted   │
                 └──────┬──────┘
                        │
        assembly.voting.closed (NATS)
                        │
                        ▼
                 ┌─────────────┐
                 │  revoked    │ (terminal dentro desta assembleia; re-granted se nova votação)
                 └─────────────┘
```

Modelo sugerido: `(video_id, resident_id, assembly_voting_id)` com `granted_at` + `revoked_at` (nullable). Consulta ABAC: `revoked_at IS NULL AND assembly_voting_id.status='open'`.

---

## 14. Events (domain + integration)

### 14.1 Domain events (emitidos pela aplicação)

Convenção: events no past tense, namespace `content.video.*`. Transport: NATS JetStream (`vault/50-projects/master-sindico/client-vault/01 - Arquitetura de Sistema/System Architecture - Content.md` §Events).

| Evento | Payload | Quando emitido | Consumidores |
|--------|---------|----------------|--------------|
| `content.video.uploaded` | `{video_id, company_id, owner_id, video_type, created_by, upload_id, created_at}` | Após `UploadVideoUseCase` retornar sucesso | Busca (index), audit trail, analytics funnel |
| `content.video.processed` (= `content.video.ready`) | `{video_id, company_id, mux_asset_id, mux_playback_id, can_be_replaced_after, processed_at}` | Após `MarkVideoReadyUseCase` | Notificação empresa ("vídeo pronto"), sync busca, dashboard agência |
| `content.video.failed` | `{video_id, company_id, reason?, failed_at}` | Após `MarkVideoFailedUseCase` | Notificação empresa ("retry upload"), alerta interno |
| `content.video.hidden` | `{video_id, company_id, hidden_by, hidden_at}` | Após `HideVideoUseCase` | Sync busca (remove do índice), dashboard |
| `content.video.unhidden` | `{video_id, company_id, unhidden_by, unhidden_at}` | Após `UnhideVideoUseCase` | Sync busca |
| `content.video.voting_auth_granted` | `{video_id, authorized_by, authorized_at}` | `AuthorizeForVoting` | — |
| `content.video.voting_auth_revoked` | `{video_id, revoked_by, revoked_at}` | `RevokeVotingAuth` | — |
| `content.video.viewed` | `{video_id, viewer_id, timestamp, context (preview/full/voting)}` | `RegisterView` | `video_interactions` (Req 32), analytics |
| `content.video.visibility.granted` | `{video_id, resident_id, assembly_voting_id, granted_at}` | No início da votação (BC assembly dispara) | BC content cria `video_visibility_grants` |
| `content.video.visibility.revoked` | `{video_id, resident_id, assembly_voting_id, revoked_at}` | `assembly.voting.closed` → reage | BC content atualiza `video_visibility_grants.revoked_at` |
| `content.marketing_agency.invited` | `{link_id, company_id, agency_email, created_by, created_at}` | `InviteMarketingAgencyUseCase` | Email outbound (Resend — Req 62) com link de aceite |
| `content.marketing_agency.accepted` | `{link_id, company_id, agency_user_id, accepted_at}` | `AcceptMarketingAgencyInviteUseCase` | Notificação empresa, audit log |
| `content.marketing_agency.revoked` | `{link_id, company_id, revoked_by, revoked_at}` | `RevokeMarketingAgencyUseCase` | Invalidação de sessões, audit log |
| `content.marketing_agency.permissions_updated` | `{link_id, company_id, permissions, updated_at}` | `UpdateMarketingAgencyPermissionsUseCase` | — |

**Estado atual**: eventos **NÃO estão sendo publicados** nos use cases visíveis. Apenas logs estruturados (`log.Info`). Registrado como **G-CV-30** (emissão de eventos NATS pendente — feature transversal, depende de `pkg/events/publisher`).

### 14.2 Integration events (consumidos por este BC)

| Evento origem | BC origem | Reação em content |
|---------------|-----------|-------------------|
| `assembly.voting.opened` | assembly | Para cada vídeo associado à pauta com `autorizar_exibicao_votacao=true`, cria `video_visibility_grants` para cada resident do condomínio (ou lazy: cria ao primeiro play attempt) — G-CV-15 |
| `assembly.voting.closed` | assembly | Seta `revoked_at = now` em `video_visibility_grants` da votação; emite `content.video.visibility.revoked` para clientes conectados |
| `billing.subscription.downgraded` | billing | Se tier cai abaixo de `plus`, bloquear novos uploads (quota 0) mas manter `ready` existentes; futuro: marcar excedente como `hidden` após N dias — G-CV-31 |
| `billing.subscription.cancelled` | billing | Após grace period (30d?), executar hard-delete (`DeleteAsset` Mux + `videos` row) — G-CV-32 |
| `identity.user.deactivated` | identity | Se era agência delegada, cascade revoga `MarketingAgencyLink` ativo |
| `commercial.company.deleted` | commercial | Cascade: hard-delete todos `videos` da company + revoga links de agência |

### 14.3 Webhook externo (Mux → content)

| Webhook | HTTP route | Ação |
|---------|-----------|------|
| `video.asset.ready` | `POST /v1/videos/webhooks/mux` | `MarkVideoReadyUseCase` |
| `video.asset.errored` | idem | `MarkVideoFailedUseCase` |
| `video.upload.cancelled` | idem | Idempotente (já pode estar failed); opcional log |
| `video.asset.deleted` | idem | Log apenas; delete já foi iniciado por admin action |
| `video.live_stream.*` (futuros) | idem | Tratamento em `06-lms-academia.md` (lives LMS) ou `05-assembleia-live.md` |

Validação obrigatória (ver §6.3): HMAC + whitelist + idempotência.

---

## 15. Dependências entre sub-produtos / BCs

### 15.1 Grafo de dependências

```
                ┌────────────────────┐
                │ 04-conteudo-videos │ (este sub-produto, BC content VideoModule)
                └─┬──────┬──────┬────┘
                  │      │      │
   depende de    │      │      │  é depended by
                  │      │      │
  ┌───────────────┴┐    │      └────────────────┐
  │ BC identity    │    │                       │
  │ (Zitadel users │    │                       │
  │  roles:        │    │                       │
  │  syndic/enterp/│    │                       │
  │  marketing/adm)│    │                       │
  └───────────┬────┘    │                       │
              │         │                       ▼
              │         │              ┌─────────────────┐
              ▼         │              │ 02-connect-me   │
  ┌──────────────────┐  │              │ (perfil empresa │
  │ BC commercial    │  │              │  exibe vídeos)  │
  │ (company entity, │  │              └─────────────────┘
  │  deals, tracking │  │
  │  de submissões)  │  │              ┌──────────────────────┐
  └──────────────────┘  │              │ 05-assembleia-live   │
                        │              │ (evento voting →     │
  ┌───────────────┐     │              │  grant visibilidade) │
  │ BC billing    │◄────┤              └──────────────────────┘
  │ (plan tier,   │     │
  │  quotas por   │     │              ┌──────────────────────┐
  │  subscription)│     │              │ 06-lms-academia      │
  └───────────────┘     │              │ (reúsa IVideoProvider│
                        │              │  para video-aulas;   │
  ┌──────────────────┐  │              │  ILiveStreamProvider │
  │ Mux (externo)    │◄─┤              │  para lives)         │
  │ IVideoProvider   │  │              └──────────────────────┘
  │ ILiveStreamProv  │  │
  └──────────────────┘  │              ┌──────────────────────┐
                        │              │ 09-compliance        │
  ┌──────────────────┐  │              │ (audit trail de      │
  │ BC search        │◄─┤              │  uploads, deletes,   │
  │ (OpenSearch/     │  │              │  visibility changes) │
  │  Meilisearch)    │  │              └──────────────────────┘
  │ sync via NATS    │  │
  └──────────────────┘  │              ┌──────────────────────┐
                        │              │ 10-marketing-agencias│
  ┌──────────────────┐  │              │ (jornada MK1-MK8,    │
  │ BC notification  │◄─┘              │  dashboard Req 45)   │
  │ (Resend, FCM)    │                 └──────────────────────┘
  │ invites + ready  │
  │ alerts           │
  └──────────────────┘
```

### 15.2 Interfaces de dependência

| Dependência | Tipo | Contrato |
|-------------|------|----------|
| **Zitadel (identity)** | sync gRPC/HTTP | Validar role do user ∈ {syndic, enterprise, marketing, admin}; `user.organization_id` para tenant isolation |
| **Mux SDK (video provider)** | sync HTTP | `IVideoProvider` interface; segredos TOKEN_ID/SECRET; webhook inbound |
| **Postgres** | sync SQL | Migrations 00400/00401/00402; sqlc generated queries |
| **NATS JetStream** | async pub/sub | Eventos §14 |
| **Resend (email)** | async via NATS → Resend job | Template invite agência, vídeo pronto |
| **OpenSearch/Meilisearch** | async via NATS listener | Index `videos` atualizado em uploaded/processed/hidden/unhidden |
| **BC billing** | sync query | `GetCurrentPlanTier(company_id) → trial|base|plus|pro` para quota check |
| **BC assembly** | async NATS consumer | `assembly.voting.opened` / `assembly.voting.closed` → grants |

### 15.3 Dependências reversas (quem depende de content)

- **02-connect-me**: perfil público da empresa exibe lista de vídeos `ready`.
- **05-assembleia-live**: pautas de votação de fornecedor linkam `video_id` → verifica `autorizar_exibicao_votacao`.
- **06-lms-academia**: reúsa `IVideoProvider` para `lesson.video_id`; reúsa `ILiveStreamProvider` (mover para `domain/providers/` G-CV-20).
- **10-marketing-agencias**: é o sub-produto consumidor primário; este BC é o **único domínio acessível** pela agência.

---

## 16. Cascade de moderação / deleção

### 16.1 Hard delete de vídeo (admin ou cancel subscription)

```
Admin.DeleteVideo(video_id)
  │
  ├─[1] videoRepo.FindByID(video_id) → Video (exists?)
  │
  ├─[2] Aggregate.MarkDeleted()  ← novo método (G-CV-27 decisão)
  │     (opção A: status custom; opção B: coluna deleted_at; opção C: DELETE físico)
  │
  ├─[3] videoProvider.DeleteAsset(mux_asset_id)   ← Mux SDK
  │     └─ falha? log + tentar reschedule via job; não bloquear
  │
  ├─[4] DELETE FROM video_analytics WHERE video_id=?
  │     DELETE FROM video_interactions WHERE video_id=?
  │     DELETE FROM video_visibility_grants WHERE video_id=?
  │
  ├─[5] DELETE FROM videos WHERE id=?
  │
  └─[6] emit content.video.deleted
        → BC search: remove from index
        → BC audit: log
        → BC analytics: snapshot preservation (se necessário LGPD — NÃO persiste PII após delete)
```

### 16.2 Cascade por eventos upstream

| Trigger | Ação |
|---------|------|
| `commercial.company.deleted` | Cascade **hard delete** de todos `videos` onde `company_id = X`; revogar todos `marketing_agency_links` da empresa |
| `billing.subscription.cancelled` | **Grace period** de 30d; ao fim, hard delete (ver G-CV-32) |
| `billing.subscription.downgraded` (plus→base) | Mark excedentes como `hidden`? Decisão pendente — G-CV-31. Proposta: manter ready, bloquear **novos** uploads via quota |
| `identity.user.deleted` (agency user) | Revogar `MarketingAgencyLink` onde `agency_user_id = X` em todas as empresas |
| `assembly.voting.closed` | Não deleta vídeo — apenas revoga `video_visibility_grants` da votação |

### 16.3 Moderação editorial (admin proativo)

**Caso 1**: admin detecta conteúdo inapropriado (spam, deepfake, copyright).

- Ação rápida: `Hide()` imediato + marker de moderação (`moderated_by_admin=true`, `moderation_reason`).
- Notificação empresa via email: "Seu vídeo foi ocultado por moderação; motivo: X. Recurso: abrir ticket."
- Se infração confirmada: hard delete após 7 dias se não houve recurso aprovado.

**Caso 2**: morador reporta vídeo via `POST /v1/videos/:id/report`.

- Cria `content_reports` row (tabela transversal — G-CV-33).
- Acumula 3 reports → flag automático para admin review.
- Admin decide Hide/DeleteAsset/Dismiss.

### 16.4 Cascade de revogação de agência

```
RevokeMarketingAgencyUseCase(link_id)
  │
  ├─[1] link.Revoke()  ← status=revoked
  ├─[2] Invalidar sessões Zitadel do agency_user apenas **para esta empresa**
  │     (se agência tem outros vínculos ativos, permanecer autenticada)
  │     → G-CV-34: scope-aware session invalidation (complexo Zitadel)
  ├─[3] emit content.marketing_agency.revoked
  └─[4] audit log: who revoked, when, company_id
```

**Importante**: revogação **NÃO** apaga vídeos já uploaded pela agência — eles continuam atribuídos à `company_id`, conforme INV-CV-6.

---

## 17. Shorts (achado investigativo)

### 17.1 Pergunta

Existe um conceito de "Vídeos Curtos / Shorts / Reels" neste sub-produto ou vertical separada?

### 17.2 Investigação realizada

**Buscas executadas**:

1. `grep -rn -i "shorts" Development/backend/internal/modules/content/` → **zero ocorrências**.
2. `grep -n "short\|curto\|reels\|tiktok" Development/Content/requirements.md` → ocorrências encontradas:
   - Linha 1321: `"Vídeo curto da Master Síndico explicando regras de participação"` (assembly, institutional, não é vertical Shorts)
   - Linha 2273: `"short practical trainings"` (Academia LMS §Área 2 Treinamentos — peças curtas, não é vertical Shorts)
   - Linha 2623/2697: `"short open field"` (form input, falso positivo textual)
3. `grep -rn -i "shorts|curto|reels" Obsidian Vault/Clients/MasterSindico/` → nenhuma ocorrência relacionada a formato Shorts.
4. Enum `VideoType` (12 valores) inspecionado — **nenhum** tipo corresponde a "short/curto/reels".
5. `System Architecture - Content.md` modulos listados: VideoModule, ContentInteractionModule, AcademiaModule, TalentBankModule — **nenhum módulo Shorts**.

### 17.3 Achado honesto

**❌ Shorts NÃO existe como vertical ou tipo canônico no Master Síndico v2.**

**Evidências**:
- Enum `video_type` não tem tipo `short`/`reel`/`curto`; os 12 tipos são todos longos-medianos (apresentação, portfólio, case, tour — tipicamente 30s–5min, mas **sem limite mínimo imposto** pelo domínio).
- Não há entidade, migration, use case, handler ou rota específica para Shorts.
- Mux suporta assets curtos nativamente (todo upload vira HLS adaptativo independente da duração), mas a **apresentação UX** de um feed vertical tipo Shorts/Reels/TikTok **não está modelada** em nenhum artefato.
- O "treinamento_sindico" (enum idx 12) é o tipo mais próximo conceitualmente a "vídeo curto educativo", mas ainda é institutional/empresa-publicado, não user-generated de feed.

### 17.4 Próximos caminhos (se Shorts vier a existir)

Se o produto futuramente adotar um formato Shorts, o caminho **menos disruptivo** seria:

**Opção A — Novo `VideoType`**: adicionar `short_institucional` ao enum + constraint de duração (≤ 60s) validado no webhook Mux (via `VideoAsset.Duration` futuro). Baixo esforço; reusa aggregate `Video`.

**Opção B — Novo aggregate `ShortVideo`**: contexto semanticamente diferente (feed infinito, engagement loop, moderação em tempo real). Novo BC ou submódulo `content/shorts/`. Alto esforço; justificado só se Shorts vira vertical estratégica.

**Opção C — Não fazer**: manter foco institucional atual; deixar conteúdo curto no formato "tutorial_servico" ou "treinamento_sindico" convencional.

**Recomendação**: registrar **G-CV-35** (Shorts não modelado) e aguardar Product Decision antes de implementar.

### 17.5 Gap registrado

**G-CV-35 — Shorts/formato curto vertical**: não modelado. Se Product Council decidir incluir, avaliar Opção A como MVP.

---

## 18. Estado atual (implementação)

### 18.1 Maturidade por componente (2026-04-23)

| Componente | Status | Cobertura | Observações |
|-----------|--------|-----------|-------------|
| Domain entity `Video` | ✅ implementado | testes em `video_test.go` | Falta `ReplaceVideo` use case (G-CV-12) |
| Domain entity `MarketingAgencyLink` | ✅ implementado | testes em `marketing_agency_link_test.go` | — |
| Enums `VideoType`/`VideoStatus`/`MarketingAgencyLinkStatus` | ✅ | — | Paridade Go ↔ Postgres |
| Migrations 00400/00401/00402 | ✅ | — | Sem `video_analytics`, `video_interactions`, `video_visibility_grants`, `marketing_agency_access_logs` |
| Use case `UploadVideoUseCase` | ✅ com saga | testes em `video_use_cases_test.go` | **Sem quota check (G-CV-08)** |
| Use cases `MarkReady`/`MarkFailed`/`Hide`/`Unhide`/`UpdateVotingAuth`/`Get`/`List` | ✅ | testes | — |
| Use cases `InviteMarketingAgency`/`Accept`/`UpdatePermissions`/`Revoke`/`List` | ✅ | testes | — |
| Repository SQL (videos, marketing_agency) | ✅ sqlc | — | Ver `queries/videos.sql`, `queries/marketing_agency.sql` |
| `MuxProvider` (`IVideoProvider` + `ILiveStreamProvider`) | ✅ | pendente teste integração real Mux | — |
| HTTP handlers (video, marketing_agency, webhook) | ✅ esboço | `handlers_common_stubs_test.go`, `video_handler_test.go`, `marketing_agency_handler_test.go`, `video_webhook_test.go` | **Webhook HMAC validation a confirmar (G-CV-09)** |
| Events NATS emission | ❌ pendente | — | G-CV-30 |
| Quota enforcement | ❌ pendente | — | G-CV-08 |
| `ReplaceVideoUseCase` (lock 90d enforced) | ❌ pendente | — | G-CV-12 |
| `DeleteVideoUseCase` (admin hard delete) | ❌ pendente | — | G-CV-06 |
| Video visibility grants (Rule 4 ABAC) | ❌ pendente | — | G-CV-15 |
| Video analytics (Mux Data API) | ❌ pendente | — | G-CV-13 |
| Video interactions (CTR, clicks) | ❌ pendente | — | G-CV-14 |
| Marketing agency access logs | ❌ pendente | — | G-CV-16 |
| SIGNED playback (paywall 25%, blindagem) | ❌ pendente | — | G-CV-10 |
| Busca (OpenSearch sync) | ❌ transversal (fora deste BC implementation-wise) | — | G-CV-02 |
| Recomendações (Req 32 engine) | ❌ transversal | — | G-CV-03 |

### 18.2 Roadmap sugerido (alinhado ROADMAP.md)

- **M1 (Sprint 10, 2026-05-08)**: já tem upload básico, hide/unhide, webhooks Mux. **Fechar**: quota enforcement (RC-Q1-Q5), SIGNED playback para vídeos empresariais, `ReplaceVideoUseCase` (RC-7 enforced), webhook HMAC validation.
- **M2 (Sprint 11–13)**: `video_visibility_grants` + integration com BC assembly; analytics Mux Data API; busca OpenSearch sync.
- **M3 (Sprint 14+)**: Shorts (se Product Council decidir — G-CV-35); hard delete admin; moderação editorial; recomendações engine.

---

## 19. Gaps documentados (G-CV-##)

Lista consolidada de todos os gaps encontrados nesta destilação:

| ID | Categoria | Descrição | Severidade | Owner sugerido |
|----|-----------|-----------|:----------:|----------------|
| G-CV-01 | Fontes | Não foi lido `Development/Content/03-Modulos/` nem `02-Jornadas/Inventario-Telas.md` — overhead vs benefício | baixa | próximo dual-check |
| G-CV-02 | Arquitetura | Busca OpenSearch/Meilisearch — spec de sync e indexação transversal pendente | média | `02-architecture/search.md` |
| G-CV-03 | Feature | Recomendação engine (Req 32) — job noturno, materialized view, CTR | média | `06-execution/jobs/recommendations.md` |
| G-CV-04 | Divergência | `content.md` Req 29 lista 12 tipos de vídeo com labels diferentes do enum Go em 5 casos | média | canônico = Go; docs a atualizar |
| G-CV-05 | Schema | Soft-delete ausente — doc menciona status `deleted`, enum Go e migration não têm | baixa | decidir soft vs hard delete |
| G-CV-06 | Feature | `DeleteVideoUseCase` (admin hard delete) não implementado | alta | backend M2 |
| G-CV-07 | Modelagem | Ambiguidade síndico-como-empresa vs empresa institucional (Req 6.2) | média | ADR pendente |
| G-CV-08 | Feature | Quota enforcement no UploadVideoUseCase — não chama QuotaService | **alta** | M1 obrigatório |
| G-CV-09 | Segurança | Webhook Mux — validação HMAC a confirmar em handler; tabela `mux_webhook_events` ausente | **alta** | M1 obrigatório |
| G-CV-10 | Segurança | Playback Mux é PUBLIC policy; precisa SIGNED JWT para empresas e paywall 25% | **alta** | M1/M2 |
| G-CV-11 | Feature | Storyboard/GIF preview via Mux não implementado | baixa | futuro |
| G-CV-12 | Feature | `CanBeReplaced()` existe mas `ReplaceVideoUseCase` não — lock 90d não é enforced | **alta** | M1 obrigatório |
| G-CV-13 | Schema | Tabela `video_analytics` não criada (Req 29 notas) | média | M2 |
| G-CV-14 | Schema | Tabela `video_interactions` não criada (Req 32) | média | M2 |
| G-CV-15 | Schema | Tabela `video_visibility_grants` não criada (Req 29 notas + Req 103) | **alta** | M2 (bloqueia Rule 4 enforcement) |
| G-CV-16 | Schema | Tabela `marketing_agency_access_logs` (Req 30 audit) | média | M1/M2 |
| G-CV-17 | Cosmético | Identificador Go `VideoTypeServicoespecializado` quebra convenção CamelCase | baixa | refactor opcional |
| G-CV-18 | Schema | `token_scope` enum vs 2 BOOLEANs — extensibilidade futura | baixa | decidir em M3 |
| G-CV-19 | Feature | Thumbnails parametrizadas (time/width) — não implementado | baixa | futuro |
| G-CV-20 | Arquitetura | `ILiveStreamProvider` em `infrastructure/` (Clean Arch violation) | média | refactor |
| G-CV-21 | Observabilidade | Métricas Prometheus content.* não implementadas | média | M2 |
| G-CV-22 | Arquitetura | Fallback multi-provider video — não implementado | baixa | futuro |
| G-CV-23 | Nomenclatura | Coluna `mux_playback_id` vs `playback_id` provider-agnostic | baixa | refactor opcional |
| G-CV-24 | UX | Telas empresa PL1–PL4 não numeradas canonicamente em doc-fonte | baixa | alinhar com UX |
| G-CV-25 | Fontes | PDF `JORNADA DA EMPRESA DE MARKETING.pdf` não lido nesta destilação (binário) | baixa | leitura manual futura |
| G-CV-26 | Feature | Dashboard Agência Marketing (Req 45) — métricas cross-empresa | média | M2 |
| G-CV-27 | Arquitetura | Decisão: status custom `banned` vs `hidden + flag moderação` vs coluna `deleted_at` | média | ADR |
| G-CV-28 | Schema | Token TTL (default 90d) não no schema — hoje token é permanente até revoke | média | M1/M2 |
| G-CV-29 | Doc | IDs INV-CV-# são propositivos; validar colisão com `01-domain/invariants.md` | baixa | dual-check |
| G-CV-30 | Feature | Publicação de eventos NATS nos use cases — nenhum event sendo emitido | **alta** | M1 obrigatório (bloqueia search/notification) |
| G-CV-31 | Regra | Ação em downgrade de plano (excedentes ficam ou são hide?) — decisão pendente | média | produto |
| G-CV-32 | Regra | Grace period hard delete em cancel subscription | média | produto |
| G-CV-33 | Feature | Sistema de reports de vídeo pelos moradores | baixa | M3 |
| G-CV-34 | Segurança | Scope-aware session invalidation Zitadel (revogar só para uma empresa) | alta | arquitetura Zitadel |
| G-CV-35 | Produto | **Shorts não modelado** — avaliar Opção A/B/C se Product Council decidir incluir | — | Product decision |

**Totais**: 35 gaps. **Críticos (M1 obrigatório)**: G-CV-08, G-CV-09, G-CV-10, G-CV-12, G-CV-15 (M2), G-CV-30.

---

## 19.5 Telas & Endpoints

Rastreabilidade consolidada em [[../../03-subprojects/traceability]] §2.4 (Conteúdo · LMS) + §3.3 (matriz inversa).

**Escopo UI**:
- Vídeo institucional empresa: E12 (perfil)
- Onboarding: ONB-S5 (síndico N2/N3 vídeo) · ONB-E6 (empresa conteúdo)
- LMS: LMS1-LMS15 (15 telas) — cursos, aulas, lives, fórum, biblioteca, certificados
- Banco de Talentos (vídeo-currículo): BT02
- Marketing agência: MK4 · MK5 · MK6

**Endpoints-chave**:
- Upload Mux: `POST /api/v1/videos?type=institutional|resume|course|short`
- CRUD vídeos: `/api/v1/content/videos` · `/:id` · `/:id/hide` · `/:id/restore`
- LMS: `/api/v1/academy/*` (courses, lessons, lives, forum, library, certificates)
- Público (LMS): `/certificates/verify/{serial}` (sem auth)
- Webhook: `/api/v1/webhooks/mux` ⚫ (HMAC antes parse)

**Invariantes críticos**: INV-056 (lock 90d) · INV-057 (admin bypass audit) · INV-058 (moderação obrigatória) · INV-060 (sem leak tenant) · INV-061/062 (unlock síndico via grant) · INV-063 (grant revogado pós votação) · INV-064 (HMAC Mux) · INV-065 (saga UploadVideo) · INV-067 (parceiro BLOCK LMS) · INV-068 (cert auto 100%) · INV-069 (ebook tier filter) · INV-070 (signed URL TTL ≤24h).

**Fluxo crítico** (traceability §7.5): Empresa convida agência (E14 · IDN-025) → agência MK4 upload → Mux presigned → `/webhooks/mux` HMAC → saga → state machine → INV-056 lock.

**Sprints**: Sprint 5 ✅ (Mux + marketing delegation). LMS ainda em stub — backlog M2/M3 (Lives provider ADR pendente — CNT-022).

---

## 20. Referências cruzadas

### 20.1 Sub-produtos v2

- [[01-governanca-institucional]] — vídeo perfil síndico (Req 6.2), mesmo lock 90d (GR-029).
- [[02-connect-me]] — perfil público da empresa exibe vídeos.
- [[03-reputacao]] — score de empresa leva em conta engajamento de vídeos (futuro).
- [[05-assembleia-live]] — trigger de visibility grants via `assembly.voting.opened/closed`.
- [[06-lms-academia]] — reúsa `IVideoProvider` para lesson.video; `ILiveStreamProvider` para lives LMS; Req 97 Ebooks.
- [[07-banco-de-talentos]] — `curriculum_videos` (Mux, 90s, lock 90d) — entidade separada.
- [[08-vizinhanca]] — sem dependência direta (conteúdo social intra-condomínio).
- [[09-compliance]] — audit trail de uploads/deletes/visibility changes (Req 37).
- [[10-marketing-agencias]] — **sub-produto consumidor primário** (MK1–MK8 + dashboard Req 45).
- [[11-administradoras-condominiais]] — potencial consumer futuro (vídeos institucionais de administradora).

### 20.2 Arquitetura e decisões

- [[../../STATE|STATE.md]] — D-056 (agnostic), D-058 (admin), D-062 (lives admin), D-066/D-067 (tiers).
- [[../../02-architecture/adr/0032-global-plans-stripe-style|ADR-0032]] — tiers universais.
- [[../../01-domain/bounded-contexts|bounded-contexts.md]] — BC `content`.
- [[../../01-domain/domain-events|domain-events.md]] — eventos canônicos.
- [[../../01-domain/invariants|invariants.md]] — INV globais.
- [[../../04-requirements/global|global.md]] — GR-028, GR-029.

### 20.3 Fontes externas (client + kiro)

- `Development/backend/.kiro/specs/master-sindico/requirements/content.md` — Req 29, 30, 31, 32, 97, 98.1.
- `Development/backend/.kiro/specs/master-sindico/requirements/cross-domain.md` — Req 64, 65, 68, 102, 103.
- `Development/backend/.kiro/specs/master-sindico/requirements/personas-and-plans.md` — quotas, matriz.
- `Development/backend/.kiro/specs/master-sindico/requirements/functional-matrix.md` — referência Req 29.
- `Development/Content/requirements.md` — Req 29 expanded, Req 30 MK1-MK8, Req 93 Lives.
- `Obsidian Vault/Clients/MasterSindico/02 - Domínios e Requisitos/Domínio - Conteúdo.md` — visão cliente.
- `Obsidian Vault/Clients/MasterSindico/01 - Arquitetura de Sistema/System Architecture - Content.md` — 4 módulos BC content.

### 20.4 Código (fonte de verdade)

- `backend/internal/modules/content/domain/entities/video.go` · `marketing_agency_link.go`
- `backend/internal/modules/content/domain/enums/video.go`
- `backend/internal/modules/content/application/use_cases/video_use_cases.go` · `marketing_agency_use_cases.go`
- `backend/internal/modules/content/infrastructure/providers/mux_provider.go` · `i_live_stream_provider.go`
- `backend/internal/modules/content/infrastructure/database/video_repository_impl.go` · `marketing_agency_repository_impl.go` · `queries/videos.sql` · `queries/marketing_agency.sql`
- `backend/internal/modules/content/infrastructure/http/routes/video_handler.go` · `marketing_agency_handler.go` · `video_webhook_test.go`
- `backend/internal/shared/providers/database/migrations/00400_create_video_enums.sql` · `00401_create_videos.sql` · `00402_create_marketing_agency_links.sql`

---

**Fim do sub-produto 04 — Conteúdo & Vídeos.** Próximo passo: **dual-check** (segundo agente em contexto limpo revisa §11 Regras, §12 Invariantes e §19 Gaps; confirma paridade Go↔Postgres; valida ausência de Shorts). Após dual-check aprovado, gaps **críticos M1** entram em `05-tasks/sprints/sprint-10.md`.



