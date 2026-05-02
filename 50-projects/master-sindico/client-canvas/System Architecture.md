---
title: System Architecture (diagrama do cliente)
type: note
tags: [master-sindico, client-canvas, diagram, mermaid]
project: master-sindico
source: Clients/MasterSindico original .canvas (2026-03-10 a 2026-03-24) — convertido para MD+mermaid em 2026-04-24
created: 2026-04-24
---

# System Architecture

Diagrama original do cliente convertido de `.canvas` (Obsidian Canvas) para Mermaid. **Visão visual** dos fluxos/arquitetura; conteúdo canônico vive em [[../04-requirements/_moc]] + [[../02-architecture/_moc]].

## Diagrama

```mermaid
graph TD
  g_commercial["📦 💼 COMMERCIAL (tenant: company_id, cross-tenant)"]:::group
  g_nats["📦 📨 NATS JetStream (Event Bus)"]:::group
  g_institutional["📦 🏛️ INSTITUTIONAL (tenant: condominium_id)"]:::group
  g_clients["📦 📱 CLIENTS"]:::group
  g_identity["📦 🆔 IDENTITY (Global, sem tenant)  ✅ Sprint 1"]:::group
  g_assembly["📦 🏛️ ASSEMBLY (tenant: condominium_id)"]:::group
  g_content["📦 🎬 CONTENT (tenant: owner_id + tenant_type)"]:::group
  g_gateway["📦 🔐 FASTIFY 5 API GATEWAY"]:::group
  g_external["📦 🔌 EXTERNAL SERVICES (Adapter Layer)"]:::group
  g_consumers["📦 ⚙️ CONSUMER GROUPS (durable, pull-based)"]:::group
  g_pipeline["📦 🔄 PIPELINE COMPLETO DE CONTRATAÇÃO"]:::group
  g_infra["📦 🗄️ DATA STORES"]:::group
  g_phases["📦 📅 FASES DE IMPLEMENTAÇÃO"]:::group
  g_outbox["📦 📦 TRANSACTIONAL OUTBOX"]:::group
  g_tenant["📦 🔒 TENANT ISOLATION"]:::group
  n_di["DI Container: Awilix<br/>SINGLETON: stateless services<br/>SCOPED: request-bo..."]
  n_timeline_proj["timeline-projector<br/>Auto-publica na Timeline<br/>quando items validados/de..."]
  n_masterplan_upd["master-plan-updater<br/>Recalcula status Master Plan<br/>via Timeline events"]
  n_notif_dispatch["notification-dispatcher<br/>Roteia → email/push/SMS<br/>baseado em regras"]
  n_outbox_flow["Fluxo Atômico:<br/>1. Use Case dentro do UnitOfWork<br/>2. Persiste dados + o..."]
  n_pg["PostgreSQL<br/>Drizzle ORM │ snake_case<br/>UUIDv7 PKs │ Soft delete<br/>~70 ..."]
  n_redis["Redis<br/>Sessions cache │ Rate limit<br/>Assembly pub/sub (WS fan-out)"]
  n_meili["Meilisearch<br/>7 índices: companies, sindicos,<br/>videos, partners, curricu..."]
  n_meili_sync["meilisearch-syncer<br/>Sincroniza índices<br/>Meilisearch ← PostgreSQL"]
  n_audit_log["audit-logger<br/>Grava todos eventos<br/>audit_trail (append-only)"]
  n_assembly_ws["assembly-broadcaster<br/>NATS → Redis Pub/Sub<br/>→ WebSocket clients"]
  n_stripe["Stripe ✅<br/>Billing + Subscriptions<br/>Pix, Boleto, Card"]
  n_mux["Mux<br/>HLS Adaptive Streaming<br/>Vídeos + Lives"]
  n_r2["Cloudflare R2<br/>Storage (S3-compatible)<br/>Documentos, Attachments"]
  n_resend["Resend (primário)<br/>Gmail SMTP (fallback)<br/>Email transacional"]
  n_fcm["Firebase Cloud Messaging<br/>Push Notifications"]
  n_sms["Twilio / Zenvia ⚠️ pendente<br/>SMS"]
  n_oauth["Google / Apple OAuth<br/>Social Login (Arctic)"]
  n_viacep["ViaCEP<br/>Geocoding + Validação"]
  n_pre_assembly["Pré-Assembleia<br/>Configurador, Pauta (agenda)<br/>Edital, Ciência, Notifica..."]
  n_live_assembly["Live-Day 🔴<br/>Check-in (app/QR/terminal)<br/>Votação real-time (WS)<br/>Fila..."]
  n_post_assembly["Pós-Assembleia<br/>Finalização + Relatórios<br/>Consolidação de dados<br/>Ind..."]
  n_interaction["ContentInteractionModule 🆕<br/>Likes, Ratings, Fórum<br/>Moderação reativa"]
  n_academia["AcademiaModule 🆕<br/>Cursos, Trainings, Lives<br/>Certificados, Biblioteca<br..."]
  n_talent["TalentBankModule 🆕<br/>11 telas cadastro morador<br/>7 funcionalidades empres..."]
  n_video["VideoModule ⚠️ expandir<br/>Mux HLS adaptive<br/>Quotas por plano"]
  n_auth["AuthModule ✅<br/>JWT + Argon2 + OAuth<br/>1 device/conta"]
  n_user["UserModule ✅<br/>CRUD + Vínculos"]
  n_profile["ProfileModule ✅<br/>Síndico, Empresa, Morador<br/>Agência, Parceiro"]
  n_onboard["OnboardingModule ✅<br/>4 fluxos por persona"]
  n_billing["BillingModule ✅<br/>Stripe │ trial/base/plus/pro<br/>Quotas + Feature Access"]
  n_search["SearchModule ✅<br/>→ Migrar para Meilisearch"]
  n_comm["CommunicationModule ✅<br/>Email + Push + SMS<br/>→ Evoluir para NotificationM..."]
  n_telao["Telão Assembleia<br/>WebSocket real-time"]
  n_web["SolidJS Web SPA<br/>TanStack Router + UnoCSS"]
  n_mobile["Mobile App<br/>(futuro) iOS/Android"]
  n_admin["Admin Panel<br/>(futuro)"]
  n_gw_swagger["Swagger + Scalar<br/>/v1/* routes"]
  n_gw_ws["WebSocket<br/>(@fastify/websocket)"]
  n_gw_mid["Rate Limit │ CORS │ Helmet │ Cookie Auth │ Tenant Scope │ CASL ABAC"]
  n_deal["ClosedDealModule 🆕<br/>Dupla assinatura, imutável<br/>Auto-publica na Timeline"]
  n_execution["ServiceExecutionModule 🆕<br/>Registros + Atividades Técnicas<br/>→ Validações..."]
  n_vizinhanca["VizinhancaModule 🆕<br/>Parceiros + Promoções<br/>Exclusivas por condomínio"]
  n_agency["MarketingAgencyModule 🆕<br/>Ator delegado, não é tenant"]
  n_connectme["ConnectMeModule ⚠️ expandir<br/>4 fluxos:<br/>• Síndico→Empresa<br/>• Morador..."]
  n_proposal["ProposalModule 🆕<br/>draft→sent→validated→accepted"]
  n_voting["SupplierVotingModule 🆕<br/>2+ propostas, quorum, real-time"]
  n_condo["CondominiumModule<br/>Registro, Mandato, Unidades<br/>Máx 15 condos/síndico"]
  n_gov["GovernanceModule<br/>┌ Timeline (7 types, append-only)<br/>│ Master Plan (26 ..."]
  n_validation["ValidationHubModule<br/>Hub centralizado validações<br/>Empresa → Síndico → T..."]
  n_compliance["ComplianceModule<br/>Declarações, Auditoria, LGPD<br/>Risk Map, Governance Sc..."]
  n_evaluation["EvaluationModule<br/>Gestão (bimestral)<br/>Empresa (pós-serviço)<br/>5 crité..."]
  n_mandate["MandateModule<br/>Transferência + Encerramento<br/>Block if not compliant"]
  n_pipe1["1️⃣ Search (Meilisearch)<br/>Síndico busca empresa"]
  n_pipe2["2️⃣ Connect Me<br/>'Tenho Interesse' + LGPD log"]
  n_pipe3["3️⃣ Proposta<br/>Empresa → Síndico valida"]
  n_pipe4["4️⃣ Votação Fornecedor<br/>2+ propostas, quorum moradores"]
  n_pipe5["5️⃣ Negócio Fechado<br/>Dupla assinatura → Timeline"]
  n_pipe6["6️⃣ Execução + Avaliação<br/>Registro → Validação → Timeline → Score"]
  n_phase0["Fase 0 — Infra: NATS + Outbox + Meilisearch"]
  n_phase1["Fase 1 — Institutional Core (2-3 sprints)"]
  n_phase2["Fase 2 — Commercial Pipeline (2-3 sprints)"]
  n_phase3["Fase 3 — Content Platform (1-2 sprints)"]
  n_phase4["Fase 4 — Assembly Engine (3-4 sprints)"]
  n_stream_institutional["Stream: INSTITUTIONAL<br/>institutional.condominium.><br/>institutional.gover..."]
  n_stream_content["Stream: CONTENT<br/>content.video.><br/>content.academy.><br/>content.talent.>"]
  n_stream_assembly["Stream: ASSEMBLY<br/>assembly.config.><br/>assembly.live.><br/>assembly.vote...."]
  n_stream_audit["Stream: AUDIT<br/>audit.> (todos eventos)<br/>(retenção: 5 anos LGPD)"]
  n_stream_identity["Stream: IDENTITY<br/>identity.auth.><br/>identity.billing.><br/>identity.user..."]
  n_stream_commercial["Stream: COMMERCIAL<br/>commercial.connect-me.><br/>commercial.proposal.><br/>..."]
  n_web -->|HTTPS (cookie)| g_gateway
  n_telao -->|WSS| n_gw_ws
  g_gateway --> g_institutional
  g_gateway --> g_commercial
  g_gateway --> g_assembly
  n_connectme -->|interesse| n_proposal
  n_proposal -->|2+ validadas| n_voting
  n_voting -->|vencedor| n_deal
  n_deal -->|contrato| n_execution
  n_deal -->|auto-publica| n_gov
  n_execution -->|submete| n_validation
  n_validation -->|aprovado → Timeline| n_gov
  n_post_assembly -->|resultados| n_gov
  g_institutional --> n_stream_institutional
  g_commercial --> n_stream_commercial
  g_content --> n_stream_content
  g_nats --> g_consumers
  g_outbox -->|publica eventos| g_nats
  g_outbox -->|TX atômica| n_pg
  n_meili_sync --> n_meili
  n_assembly_ws -->|pub/sub| n_redis
  n_billing --> n_stripe
  n_video --> n_mux
  n_comm --> n_resend
  n_pipe1 --> n_pipe2
  n_pipe2 --> n_pipe3
  n_pipe3 --> n_pipe4
  n_pipe4 --> n_pipe5
  n_pipe5 --> n_pipe6
  n_phase0 --> n_phase1
  n_phase1 --> n_phase2
  n_phase2 --> n_phase3
  n_phase3 --> n_phase4
  n_live_assembly -->|WebSocket| n_gw_ws
  g_identity --> n_stream_identity
  g_gateway --> g_content
  g_assembly --> n_stream_assembly
  g_gateway --> g_identity
  classDef group fill:#eef,stroke:#66a,stroke-width:2px,color:#000
```

## Nodes (85)

- **[GROUP]** `g_commercial` — 💼 COMMERCIAL (tenant: company_id, cross-tenant)
- **[GROUP]** `g_nats` — 📨 NATS JetStream (Event Bus)
- **[GROUP]** `g_institutional` — 🏛️ INSTITUTIONAL (tenant: condominium_id)
- **[GROUP]** `g_clients` — 📱 CLIENTS
- **[GROUP]** `g_identity` — 🆔 IDENTITY (Global, sem tenant)  ✅ Sprint 1
- **[GROUP]** `g_assembly` — 🏛️ ASSEMBLY (tenant: condominium_id)
- **[GROUP]** `g_content` — 🎬 CONTENT (tenant: owner_id + tenant_type)
- **[GROUP]** `g_gateway` — 🔐 FASTIFY 5 API GATEWAY
- **[GROUP]** `g_external` — 🔌 EXTERNAL SERVICES (Adapter Layer)
- **[GROUP]** `g_consumers` — ⚙️ CONSUMER GROUPS (durable, pull-based)
- **[GROUP]** `g_pipeline` — 🔄 PIPELINE COMPLETO DE CONTRATAÇÃO
- **[GROUP]** `g_infra` — 🗄️ DATA STORES
- **[GROUP]** `g_phases` — 📅 FASES DE IMPLEMENTAÇÃO
- **[GROUP]** `g_outbox` — 📦 TRANSACTIONAL OUTBOX
- **[GROUP]** `g_tenant` — 🔒 TENANT ISOLATION
- `n_di` — **DI Container:** Awilix · SINGLETON: stateless services · SCOPED: request-bound (repos, use cases) · **UoW:** AsyncLocalStorage · **Module:** register(app) + bootstrap(app)
- `n_timeline_proj` — **timeline-projector** · Auto-publica na Timeline · quando items validados/deals confirmados
- `n_masterplan_upd` — **master-plan-updater** · Recalcula status Master Plan · via Timeline events
- `n_notif_dispatch` — **notification-dispatcher** · Roteia → email/push/SMS · baseado em regras
- `n_outbox_flow` — **Fluxo Atômico:** · 1. Use Case dentro do UnitOfWork · 2. Persiste dados + outbox_events ·    na MESMA transação · 3. TX commit = zero perda ·  · **Publisher Worker:** · • Poll 100ms, batch 100 · • Publica no NATS JetStream · • Retry exponencial · • Dead-letter após 10 falhas
- `n_pg` — **PostgreSQL** · Drizzle ORM │ snake_case · UUIDv7 PKs │ Soft delete · ~70 tabelas (25 existentes + 45 novas)
- `n_redis` — **Redis** · Sessions cache │ Rate limit · Assembly pub/sub (WS fan-out)
- `n_meili` — **Meilisearch** · 7 índices: companies, sindicos, · videos, partners, curricula, · courses, forum_topics
- `n_meili_sync` — **meilisearch-syncer** · Sincroniza índices · Meilisearch ← PostgreSQL
- `n_audit_log` — **audit-logger** · Grava todos eventos · audit_trail (append-only)
- `n_assembly_ws` — **assembly-broadcaster** · NATS → Redis Pub/Sub · → WebSocket clients
- `n_stripe` — **Stripe** ✅ · Billing + Subscriptions · Pix, Boleto, Card
- `n_mux` — **Mux** · HLS Adaptive Streaming · Vídeos + Lives
- `n_r2` — **Cloudflare R2** · Storage (S3-compatible) · Documentos, Attachments
- `n_resend` — **Resend** (primário) · **Gmail SMTP** (fallback) · Email transacional
- `n_fcm` — **Firebase Cloud Messaging** · Push Notifications
- `n_sms` — **Twilio / Zenvia** ⚠️ pendente · SMS
- `n_oauth` — **Google / Apple OAuth** · Social Login (Arctic)
- `n_viacep` — **ViaCEP** · Geocoding + Validação
- `n_pre_assembly` — **Pré-Assembleia** · Configurador, Pauta (agenda) · Edital, Ciência, Notificações · Polls Preliminares · Procuração, Análise Fornecedor · Panel Administradora · Simulador de Quorum
- `n_live_assembly` — **Live-Day** 🔴 · Check-in (app/QR/terminal) · Votação real-time (WS) · Fila de Fala + Timer · Telão (2 áreas) · Controles Presidente · Contingência · Clock + Termos Legais
- `n_post_assembly` — **Pós-Assembleia** · Finalização + Relatórios · Consolidação de dados · Indicador Transparência (0-100) · Homologação · Notificações automáticas · → Auto-publica na Timeline
- `n_interaction` — **ContentInteractionModule** 🆕 · Likes, Ratings, Fórum · Moderação reativa
- `n_academia` — **AcademiaModule** 🆕 · Cursos, Trainings, Lives · Certificados, Biblioteca · Learning Paths
- `n_talent` — **TalentBankModule** 🆕 · 11 telas cadastro morador · 7 funcionalidades empresa · Vídeo 90s + LGPD heavy · ⚠️ Pendente confirmação escopo
- `n_video` — **VideoModule** ⚠️ expandir · Mux HLS adaptive · Quotas por plano
- `n_auth` — **AuthModule** ✅ · JWT + Argon2 + OAuth · 1 device/conta
- `n_user` — **UserModule** ✅ · CRUD + Vínculos
- `n_profile` — **ProfileModule** ✅ · Síndico, Empresa, Morador · Agência, Parceiro
- `n_onboard` — **OnboardingModule** ✅ · 4 fluxos por persona
- `n_billing` — **BillingModule** ✅ · Stripe │ trial/base/plus/pro · Quotas + Feature Access
- `n_search` — **SearchModule** ✅ · → Migrar para Meilisearch
- `n_comm` — **CommunicationModule** ✅ · Email + Push + SMS · → Evoluir para NotificationModule
- `n_telao` — **Telão Assembleia** · WebSocket real-time
- `n_web` — **SolidJS Web SPA** · TanStack Router + UnoCSS
- `n_mobile` — **Mobile App** · (futuro) iOS/Android
- `n_admin` — **Admin Panel** · (futuro)
- `n_gw_swagger` — Swagger + Scalar · /v1/* routes
- `n_gw_ws` — WebSocket · (@fastify/websocket)
- `n_gw_mid` — Rate Limit │ CORS │ Helmet │ Cookie Auth │ Tenant Scope │ CASL ABAC
- `n_deal` — **ClosedDealModule** 🆕 · Dupla assinatura, imutável · Auto-publica na Timeline
- `n_execution` — **ServiceExecutionModule** 🆕 · Registros + Atividades Técnicas · → Validações Pendentes
- `n_vizinhanca` — **VizinhancaModule** 🆕 · Parceiros + Promoções · Exclusivas por condomínio
- `n_agency` — **MarketingAgencyModule** 🆕 · Ator delegado, não é tenant
- `n_connectme` — **ConnectMeModule** ⚠️ expandir · 4 fluxos: · • Síndico→Empresa · • Morador→Síndico · • Empresa→Empresa · • Síndico→Parceiro · LGPD by design, auto-close 48h
- `n_proposal` — **ProposalModule** 🆕 · draft→sent→validated→accepted
- `n_voting` — **SupplierVotingModule** 🆕 · 2+ propostas, quorum, real-time
- `n_condo` — **CondominiumModule** · Registro, Mandato, Unidades · Máx 15 condos/síndico
- `n_gov` — **GovernanceModule** · ┌ Timeline (7 types, append-only) · │ Master Plan (26 atividades) · │ Eventos (13 types) · │ Comunicados (8 types) · └ Consultas Moradores
- `n_validation` — **ValidationHubModule** · Hub centralizado validações · Empresa → Síndico → Timeline
- `n_compliance` — **ComplianceModule** · Declarações, Auditoria, LGPD · Risk Map, Governance Score · Dossier Export
- `n_evaluation` — **EvaluationModule** · Gestão (bimestral) · Empresa (pós-serviço) · 5 critérios, 1-10
- `n_mandate` — **MandateModule** · Transferência + Encerramento · Block if not compliant
- `n_pipe1` — 1️⃣ **Search** (Meilisearch) · Síndico busca empresa
- `n_pipe2` — 2️⃣ **Connect Me** · "Tenho Interesse" + LGPD log
- `n_pipe3` — 3️⃣ **Proposta** · Empresa → Síndico valida
- `n_pipe4` — 4️⃣ **Votação Fornecedor** · 2+ propostas, quorum moradores
- `n_pipe5` — 5️⃣ **Negócio Fechado** · Dupla assinatura → Timeline
- `n_pipe6` — 6️⃣ **Execução + Avaliação** · Registro → Validação → Timeline → Score
- `n_phase0` — **Fase 0** — Infra: NATS + Outbox + Meilisearch
- `n_phase1` — **Fase 1** — Institutional Core (2-3 sprints)
- `n_phase2` — **Fase 2** — Commercial Pipeline (2-3 sprints)
- `n_phase3` — **Fase 3** — Content Platform (1-2 sprints)
- `n_phase4` — **Fase 4** — Assembly Engine (3-4 sprints)
- `n_stream_institutional` — **Stream: INSTITUTIONAL** · institutional.condominium.> · institutional.governance.> · institutional.compliance.> · institutional.validation.>
- `n_stream_content` — **Stream: CONTENT** · content.video.> · content.academy.> · content.talent.>
- `n_stream_assembly` — **Stream: ASSEMBLY** · assembly.config.> · assembly.live.> · assembly.vote.> · assembly.speech.> · (retenção: 365 dias)
- `n_stream_audit` — **Stream: AUDIT** · audit.> (todos eventos) · (retenção: 5 anos LGPD)
- `n_stream_identity` — **Stream: IDENTITY** · identity.auth.> · identity.billing.> · identity.user.> · identity.profile.>
- `n_stream_commercial` — **Stream: COMMERCIAL** · commercial.connect-me.> · commercial.proposal.> · commercial.voting.> · commercial.deal.> · commercial.execution.>

## Edges (38)

- `n_web` → `g_gateway` — _HTTPS (cookie)_
- `n_telao` → `n_gw_ws` — _WSS_
- `g_gateway` → `g_institutional`
- `g_gateway` → `g_commercial`
- `g_gateway` → `g_assembly`
- `n_connectme` → `n_proposal` — _interesse_
- `n_proposal` → `n_voting` — _2+ validadas_
- `n_voting` → `n_deal` — _vencedor_
- `n_deal` → `n_execution` — _contrato_
- `n_deal` → `n_gov` — _auto-publica_
- `n_execution` → `n_validation` — _submete_
- `n_validation` → `n_gov` — _aprovado → Timeline_
- `n_post_assembly` → `n_gov` — _resultados_
- `g_institutional` → `n_stream_institutional`
- `g_commercial` → `n_stream_commercial`
- `g_content` → `n_stream_content`
- `g_nats` → `g_consumers`
- `g_outbox` → `g_nats` — _publica eventos_
- `g_outbox` → `n_pg` — _TX atômica_
- `n_meili_sync` → `n_meili`
- `n_assembly_ws` → `n_redis` — _pub/sub_
- `n_billing` → `n_stripe`
- `n_video` → `n_mux`
- `n_comm` → `n_resend`
- `n_pipe1` → `n_pipe2`
- `n_pipe2` → `n_pipe3`
- `n_pipe3` → `n_pipe4`
- `n_pipe4` → `n_pipe5`
- `n_pipe5` → `n_pipe6`
- `n_phase0` → `n_phase1`
- `n_phase1` → `n_phase2`
- `n_phase2` → `n_phase3`
- `n_phase3` → `n_phase4`
- `n_live_assembly` → `n_gw_ws` — _WebSocket_
- `g_identity` → `n_stream_identity`
- `g_gateway` → `g_content`
- `g_assembly` → `n_stream_assembly`
- `g_gateway` → `g_identity`

## Links

- [[_moc]] — índice dos canvas do cliente
- [[../CLAUDE]] — contrato do projeto
- [[../02-architecture/_moc]]
- [[../04-requirements/_moc]]