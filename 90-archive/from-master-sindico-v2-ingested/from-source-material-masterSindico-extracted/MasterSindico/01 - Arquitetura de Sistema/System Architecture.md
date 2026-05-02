---
title: "System Architecture (canvas)"
type: canvas
tags: [canvas, obsidian, converted]
source: 50-projects/master-sindico/MasterSindico_extracted/MasterSindico/01 - Arquitetura de Sistema/System Architecture.canvas
converted: 2026-04-22
---

# System Architecture

> Canvas Obsidian convertido. Estrutura original: JSON com nodes + edges.

## Nodes

- **group**: 💼 COMMERCIAL (tenant: company_id, cross-tenant)
- **group**: 📨 NATS JetStream (Event Bus)
- **group**: 🏛️ INSTITUTIONAL (tenant: condominium_id)
- **group**: 📱 CLIENTS
- **group**: 🆔 IDENTITY (Global, sem tenant)  ✅ Sprint 1
- **group**: 🏛️ ASSEMBLY (tenant: condominium_id)
- **group**: 🎬 CONTENT (tenant: owner_id + tenant_type)
- **group**: 🔐 FASTIFY 5 API GATEWAY
- **group**: 🔌 EXTERNAL SERVICES (Adapter Layer)
- **group**: ⚙️ CONSUMER GROUPS (durable, pull-based)
- **group**: 🔄 PIPELINE COMPLETO DE CONTRATAÇÃO
- **group**: 🗄️ DATA STORES
- **group**: 📅 FASES DE IMPLEMENTAÇÃO
- **group**: 📦 TRANSACTIONAL OUTBOX
- **group**: 🔒 TENANT ISOLATION
- **text**: **DI Container:** Awilix / SINGLETON: stateless services / SCOPED: request-bound (repos, use cases) / **UoW:** AsyncLocalStorage / **Module:** register(app) + bootstrap(app)
- **text**: **timeline-projector** / Auto-publica na Timeline / quando items validados/deals confirmados
- **text**: **master-plan-updater** / Recalcula status Master Plan / via Timeline events
- **text**: **notification-dispatcher** / Roteia → email/push/SMS / baseado em regras
- **text**: **Fluxo Atômico:** / 1. Use Case dentro do UnitOfWork / 2. Persiste dados + outbox_events /    na MESMA transação / 3. TX commit = zero perda /  / **Publisher Worker:** / • Poll 100ms, batch 100 / • Publica no NATS JetStream / • Retry exponencial / • Dead-letter após 10 falhas
- **text**: **PostgreSQL** / Drizzle ORM │ snake_case / UUIDv7 PKs │ Soft delete / ~70 tabelas (25 existentes + 45 novas)
- **text**: **Redis** / Sessions cache │ Rate limit / Assembly pub/sub (WS fan-out)
- **text**: **Meilisearch** / 7 índices: companies, sindicos, / videos, partners, curricula, / courses, forum_topics
- **text**: **meilisearch-syncer** / Sincroniza índices / Meilisearch ← PostgreSQL
- **text**: **audit-logger** / Grava todos eventos / audit_trail (append-only)
- **text**: **assembly-broadcaster** / NATS → Redis Pub/Sub / → WebSocket clients
- **text**: **Stripe** ✅ / Billing + Subscriptions / Pix, Boleto, Card
- **text**: **Mux** / HLS Adaptive Streaming / Vídeos + Lives
- **text**: **Cloudflare R2** / Storage (S3-compatible) / Documentos, Attachments
- **text**: **Resend** (primário) / **Gmail SMTP** (fallback) / Email transacional
- **text**: **Firebase Cloud Messaging** / Push Notifications
- **text**: **Twilio / Zenvia** ⚠️ pendente / SMS
- **text**: **Google / Apple OAuth** / Social Login (Arctic)
- **text**: **ViaCEP** / Geocoding + Validação
- **text**: **Pré-Assembleia** / Configurador, Pauta (agenda) / Edital, Ciência, Notificações / Polls Preliminares / Procuração, Análise Fornecedor / Panel Administradora / Simulador de Quorum
- **text**: **Live-Day** 🔴 / Check-in (app/QR/terminal) / Votação real-time (WS) / Fila de Fala + Timer / Telão (2 áreas) / Controles Presidente / Contingência / Clock + Termos Legais
- **text**: **Pós-Assembleia** / Finalização + Relatórios / Consolidação de dados / Indicador Transparência (0-100) / Homologação / Notificações automáticas / → Auto-publica na Timeline
- **text**: **ContentInteractionModule** 🆕 / Likes, Ratings, Fórum / Moderação reativa
- **text**: **AcademiaModule** 🆕 / Cursos, Trainings, Lives / Certificados, Biblioteca / Learning Paths
- **text**: **TalentBankModule** 🆕 / 11 telas cadastro morador / 7 funcionalidades empresa / Vídeo 90s + LGPD heavy / ⚠️ Pendente confirmação escopo
- **text**: **VideoModule** ⚠️ expandir / Mux HLS adaptive / Quotas por plano
- **text**: **AuthModule** ✅ / JWT + Argon2 + OAuth / 1 device/conta
- **text**: **UserModule** ✅ / CRUD + Vínculos
- **text**: **ProfileModule** ✅ / Síndico, Empresa, Morador / Agência, Parceiro
- **text**: **OnboardingModule** ✅ / 4 fluxos por persona
- **text**: **BillingModule** ✅ / Stripe │ trial/base/plus/pro / Quotas + Feature Access
- **text**: **SearchModule** ✅ / → Migrar para Meilisearch
- **text**: **CommunicationModule** ✅ / Email + Push + SMS / → Evoluir para NotificationModule
- **text**: **Telão Assembleia** / WebSocket real-time
- **text**: **SolidJS Web SPA** / TanStack Router + UnoCSS
- **text**: **Mobile App** / (futuro) iOS/Android
- **text**: **Admin Panel** / (futuro)
- **text**: Swagger + Scalar / /v1/* routes
- **text**: WebSocket / (@fastify/websocket)
- **text**: Rate Limit │ CORS │ Helmet │ Cookie Auth │ Tenant Scope │ CASL ABAC
- **text**: **ClosedDealModule** 🆕 / Dupla assinatura, imutável / Auto-publica na Timeline
- **text**: **ServiceExecutionModule** 🆕 / Registros + Atividades Técnicas / → Validações Pendentes
- **text**: **VizinhancaModule** 🆕 / Parceiros + Promoções / Exclusivas por condomínio
- **text**: **MarketingAgencyModule** 🆕 / Ator delegado, não é tenant
- **text**: **ConnectMeModule** ⚠️ expandir / 4 fluxos: / • Síndico→Empresa / • Morador→Síndico / • Empresa→Empresa / • Síndico→Parceiro / LGPD by design, auto-close 48h
- **text**: **ProposalModule** 🆕 / draft→sent→validated→accepted
- **text**: **SupplierVotingModule** 🆕 / 2+ propostas, quorum, real-time
- **text**: **CondominiumModule** / Registro, Mandato, Unidades / Máx 15 condos/síndico
- **text**: **GovernanceModule** / ┌ Timeline (7 types, append-only) / │ Master Plan (26 atividades) / │ Eventos (13 types) / │ Comunicados (8 types) / └ Consultas Moradores
- **text**: **ValidationHubModule** / Hub centralizado validações / Empresa → Síndico → Timeline
- **text**: **ComplianceModule** / Declarações, Auditoria, LGPD / Risk Map, Governance Score / Dossier Export
- **text**: **EvaluationModule** / Gestão (bimestral) / Empresa (pós-serviço) / 5 critérios, 1-10
- **text**: **MandateModule** / Transferência + Encerramento / Block if not compliant
- **text**: 1️⃣ **Search** (Meilisearch) / Síndico busca empresa
- **text**: 2️⃣ **Connect Me** / "Tenho Interesse" + LGPD log
- **text**: 3️⃣ **Proposta** / Empresa → Síndico valida
- **text**: 4️⃣ **Votação Fornecedor** / 2+ propostas, quorum moradores
- **text**: 5️⃣ **Negócio Fechado** / Dupla assinatura → Timeline
- **text**: 6️⃣ **Execução + Avaliação** / Registro → Validação → Timeline → Score
- **text**: **Fase 0** — Infra: NATS + Outbox + Meilisearch
- **text**: **Fase 1** — Institutional Core (2-3 sprints)
- **text**: **Fase 2** — Commercial Pipeline (2-3 sprints)
- **text**: **Fase 3** — Content Platform (1-2 sprints)
- **text**: **Fase 4** — Assembly Engine (3-4 sprints)
- **text**: **Stream: INSTITUTIONAL** / institutional.condominium.> / institutional.governance.> / institutional.compliance.> / institutional.validation.>
- **text**: **Stream: CONTENT** / content.video.> / content.academy.> / content.talent.>
- **text**: **Stream: ASSEMBLY** / assembly.config.> / assembly.live.> / assembly.vote.> / assembly.speech.> / (retenção: 365 dias)
- **text**: **Stream: AUDIT** / audit.> (todos eventos) / (retenção: 5 anos LGPD)
- **text**: **Stream: IDENTITY** / identity.auth.> / identity.billing.> / identity.user.> / identity.profile.>
- **text**: **Stream: COMMERCIAL** / commercial.connect-me.> / commercial.proposal.> / commercial.voting.> / commercial.deal.> / commercial.execution.>

## Edges

- n_web → g_gateway (HTTPS (cookie))
- n_telao → n_gw_ws (WSS)
- g_gateway → g_institutional 
- g_gateway → g_commercial 
- g_gateway → g_assembly 
- n_connectme → n_proposal (interesse)
- n_proposal → n_voting (2+ validadas)
- n_voting → n_deal (vencedor)
- n_deal → n_execution (contrato)
- n_deal → n_gov (auto-publica)
- n_execution → n_validation (submete)
- n_validation → n_gov (aprovado → Timeline)
- n_post_assembly → n_gov (resultados)
- g_institutional → n_stream_institutional 
- g_commercial → n_stream_commercial 
- g_content → n_stream_content 
- g_nats → g_consumers 
- g_outbox → g_nats (publica eventos)
- g_outbox → n_pg (TX atômica)
- n_meili_sync → n_meili 
- n_assembly_ws → n_redis (pub/sub)
- n_billing → n_stripe 
- n_video → n_mux 
- n_comm → n_resend 
- n_pipe1 → n_pipe2 
- n_pipe2 → n_pipe3 
- n_pipe3 → n_pipe4 
- n_pipe4 → n_pipe5 
- n_pipe5 → n_pipe6 
- n_phase0 → n_phase1 
- n_phase1 → n_phase2 
- n_phase2 → n_phase3 
- n_phase3 → n_phase4 
- n_live_assembly → n_gw_ws (WebSocket)
- g_identity → n_stream_identity 
- g_gateway → g_content 
- g_assembly → n_stream_assembly 
- g_gateway → g_identity 
