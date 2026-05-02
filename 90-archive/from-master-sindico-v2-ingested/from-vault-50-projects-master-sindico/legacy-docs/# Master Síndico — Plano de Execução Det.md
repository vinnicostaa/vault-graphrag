# Master Síndico — Plano de Execução Detalhado

> **Versão:** 1.0 — Março 2026
> **Equipe:** 1 Full-Stack + 1 Mobile + AI Heavy
> **Marcos:** 08/05, 20/06, 20/07
> **Início:** 20/03/2026

---

## VISÃO GERAL DOS MARCOS

```
20/03 ─────────────── 08/05 ─────────────── 20/06 ──────────── 20/07
│      MARCO 1 (7 sem)      │   MARCO 2 (6 sem)   │  MARCO 3 (4 sem)  │
│      FOUNDATION            │   CORE GOVERNANCE    │  COMMERCIAL+CONTENT│
│                            │                      │                    │
│ Infra + IAM + Billing      │ Governança + Morador │ Empresa + Mídia    │
│ + Design System + Search   │ + Síndico + ConnectMe│ + Agência + Comply │
```

---

## MARCO 1 — FOUNDATION (20/03 → 08/05 = 7 semanas)

**Objetivo:** plataforma que autentica, cobra, faz onboarding e tem busca funcionando end-to-end.

**Entrega final:** usuário entra, escolhe perfil, faz onboarding, assina plano (ou trial), e consegue buscar empresas/síndicos.

---

### Semana 1 (20/03 → 27/03): ARQUITETURA E INFRA BASE

**Full-Stack:**

Dia 1-2: Setup do repositório
- Monorepo structure (pnpm workspaces)
- Backend: NestJS ou Fastify (decisão final)
- Package mastersindico-schemas: primeiros schemas Zod (User, Tenant, Plan)
- Biome config, TypeScript strict mode
- .env structure para dev/staging/prod

Dia 3-4: AWS Foundation
- VPC com 2 AZs (public + private subnets)
- RDS PostgreSQL 15 (db.t3.medium, Multi-AZ staging)
- ElastiCache Redis (cache.t3.micro)
- S3 bucket (mastersindico-assets) + CloudFront distribution
- Route53 hosted zone (mastersindico.com.br)
- ACM certificate (*.mastersindico.com.br)
- CloudWatch log groups e alarmes básicos

Dia 5: CI/CD Pipeline
- CodePipeline: main → build → deploy staging
- CodeBuild: lint + type-check + test + build
- ECR repository para container images
- ECS Fargate cluster (staging)
- Deploy automático no push pra main

**Mobile:**

Dia 1-3: Setup do projeto
- React Native com Expo (SDK 52+)
- TypeScript strict mode
- Estrutura de pastas espelhando o web (domains/)
- Navegação base (React Navigation 7)
- Conexão com mastersindico-schemas (shared workspace)

Dia 4-5: Ferramental
- Storybook mobile pra componentes isolados
- CI pipeline mobile (EAS Build)
- Configuração de signing (iOS + Android)

**Entrega Semana 1:** repos criados, infra AWS provisionada, CI/CD rodando, ambos deploys automáticos.

---

### Semana 2 (27/03 → 03/04): ZITADEL + DATABASE FOUNDATION

**Full-Stack:**

Dia 1-2: Zitadel Deploy
- Deploy Zitadel em ECS Fargate (container oficial)
- RDS PostgreSQL dedicado pra Zitadel (ou schema separado)
- Configuração de domínio: auth.mastersindico.com.br
- SSL via ACM + ALB

Dia 3: Zitadel Configuration
- Organization: MasterSíndico (root)
- Projects: web-app, mobile-app, backend-api
- Roles mapeados: sindico, morador_titular, morador_dependente, empresa_administrador, empresa_operador, agencia, parceiro_vizinhanca
- Actions (webhooks): UserRegistered, UserAuthenticated
- Branding: logo, cores, email templates

Dia 4: Zitadel Multi-Tenancy Mapping
- Decisão arquitetural: Zitadel Organizations como tenants OU metadata customizada
- Implementar tenant_type (condominium, company) como user metadata
- Implementar tenant_id como user grant
- Testar: user com múltiplos tenants (síndico com 2+ condomínios)

Dia 5: Database Schemas (Application DB)
- Migration 001: tenants (id, type, name, status, created_at)
- Migration 002: condominiums (tenant_id, address, cep, type, total_units, blocks, qr_code, mandate_*)
- Migration 003: companies (tenant_id, razao_social, cnpj, category, subcategories, status)
- Migration 004: subscriptions (user_id, plan, status, trial_start, trial_end, stripe_customer_id, stripe_subscription_id)
- Migration 005: user_profiles (user_id, tenant_id, role, onboarding_status, profile_data JSONB)

**Mobile:**

Dia 1-3: Auth SDK Integration
- Integrar Zitadel OIDC com React Native
- Secure token storage (expo-secure-store)
- Auth flow: login → token → refresh → logout
- Deep linking pra email verification

Dia 4-5: Navigation Structure
- Auth stack (login, register, verify, forgot-password)
- Main tab navigator (Home, Busca, Perfil, Mais)
- Stack navigators por tab
- Protected routes (redirect se não autenticado)

**Entrega Semana 2:** Zitadel rodando, login funcional web+mobile, database com schemas base.

---

### Semana 3 (03/04 → 10/04): STRIPE + DESIGN SYSTEM

**Full-Stack:**

Dia 1-2: Stripe Integration
- Stripe account setup (test mode)
- Products: trial_sindico, trial_empresa, trial_parceiro, plan_base, plan_plus, plan_pro
- Prices: monthly + annual pra cada plan
- Stripe Adapter no backend:
  - createCustomer(userId, email)
  - createSubscription(customerId, priceId)
  - createTrialSubscription(customerId, trialDays)
  - cancelSubscription(subscriptionId)
  - getSubscriptionStatus(customerId)
- Webhook handler: customer.subscription.created, customer.subscription.updated, customer.subscription.deleted, invoice.paid, invoice.payment_failed
- Trial logic: síndico 15d, empresa 7d, parceiro 30d

Dia 3-5: Design System Web (SolidJS)
- Tokens CSS (já definidos no main.css — validar todos)
- Componentes base com Kobalte + CVA:
  - Button (primary, secondary, destructive, ghost, link — sizes sm/md/lg)
  - Input (text, email, password, number, phone, cep — states normal/error/disabled)
  - Select (single, multi — com search)
  - Checkbox, Radio, Switch
  - Card (default, interactive, highlighted)
  - Badge (plan, status, notification)
  - Avatar (user, company, condominium)
  - Modal (dialog, confirmation, alert)
  - Toast (success, error, warning, info — via solid-sonner)
  - Tabs, Accordion
  - Skeleton loaders
  - Empty states
- Layouts:
  - AuthLayout (centered, logo, form)
  - DashboardLayout (sidebar navy + topbar + content area)
  - PublicLayout (topbar + content)
- Sidebar navigation com role-based items
- Dark/light mode toggle (já implementado no index.tsx)

**Mobile:**

Dia 1-5: Design System Mobile
- Componentes espelhados do web (mesmos tokens, mesma API de props)
- Button, Input, Select, Card, Badge, Avatar, Modal, Toast
- Bottom tab bar (padrão mobile)
- Header customizado
- SafeAreaView wrapper
- Keyboard avoiding behavior

**Entrega Semana 3:** Stripe cobrando, design system completo nos dois, layouts prontos.

---

### Semana 4 (10/04 → 17/04): AUTH SCREENS + ONBOARDING STRUCTURE

**Full-Stack:**

Dia 1-2: Telas de Auth (Web)
- TELA: Splash/Landing (logo, "Governança condominial estruturada", botão Entrar)
- TELA: Login (email + password, "Esqueci senha", link cadastro)
- TELA: Cadastro Inteligente (detecta role via SearchParams OU seleção manual)
- TELA: Verificação de Email (campo OTP via corvu/otp-field)
- TELA: Recuperação de Senha (email → token → nova senha)
- TELA: Seleção de Perfil (Morador, Síndico, Empresa, Parceiro — cards visuais)
- Integração real com Zitadel: register, login, verify, reset

Dia 3-5: Onboarding Framework + Síndico Flow
- Framework genérico de onboarding: stepper com progresso, auto-save, validação por step, resume
- Onboarding Síndico (6 telas — o mais complexo):
  - TELA 0: Boas-vindas + seleção perfil (já feita)
  - TELA 1: Identificação inicial (nome, email)
  - TELA 2: Identidade profissional (foto, nome, nascimento, email prof, telefone, CPF)
  - TELA 3: Atuação (endereço, cidade/estado, tipo síndico, experiência, qtd condos)
  - TELA 4: Marcadores de governança (15 checkboxes)
  - TELA 5: Presença profissional — N2/N3 only (bio, vídeo, certificações, administradoras)
  - TELA 6: Termos (3 aceites obrigatórios)
- TanStack Form + Zod validation em cada step
- Auto-save via localStorage + backend sync

**Mobile:**

Dia 1-2: Auth Screens Mobile
- Splash, Login, Cadastro, Verificação OTP, Recuperação Senha
- Espelhamento visual do web
- Biometric login preparation (FaceID/TouchID)

Dia 3-5: Onboarding Mobile
- Stepper mobile (horizontal scroll com indicador)
- Onboarding Síndico espelhado
- Camera integration pra foto de perfil
- Formulários adaptados pra mobile (keyboard types, auto-fill)

**Entrega Semana 4:** auth end-to-end funcionando, síndico faz onboarding completo, dados persistem.

---

### Semana 5 (17/04 → 24/04): ONBOARDING RESTANTE + TENANT REGISTRY

**Full-Stack:**

Dia 1-2: Onboarding restante (Web)
- Onboarding Morador (4 telas):
  - Identidade (foto, nome, nascimento, telefone, CPF)
  - Vínculo condomínio (CEP, endereço, nome/código condomínio)
  - Termos (Geral, LGPD, Avaliações)
- Onboarding Empresa (7 telas):
  - Identidade institucional (logo, razão social, fantasia, CNPJ, aniversário, representante)
  - Contatos (email/telefone comercial, endereço, dados financeiro)
  - Atuação (cidades/estados, categoria principal, subcategorias)
  - Marcadores compliance (25 checkboxes)
  - Conteúdo institucional (descrição, vídeo, portfólio)
  - Termos (Conteúdo Técnico, Connect Me)
- Onboarding Parceiro (4 telas):
  - Presença local (logo, nome, CNPJ/CPF, representante)
  - Contato + visual (email, telefone, endereço, até 3 fotos)
  - Termos (Promoções, Código de Conduta)
- Tela final comum: confirmação + "Acessar plataforma"

Dia 3-4: Condominium Registry (Backend + Frontend)
- API: POST /condominiums (create with address, type, units, blocks)
- API: GET /condominiums/:id
- API: GET /condominiums/by-code/:code
- Gerar ID alfanumérico (8 chars)
- Gerar QR Code (via qrcode library)
- Cadastro de mandato (tipo, datas)
- TELA S3: Cadastrar Condomínio
- TELA S4: Dados da Gestão
- TELA S5: Identidade do Condomínio (ID + QR + compartilhar)

Dia 5: Company Registry (Backend + Frontend)
- API: POST /companies (create with CNPJ, categories, profile)
- API: GET /companies/:id
- Validação CNPJ format
- Profile creation com status pending_verification
- TELA E11: Perfil Institucional (formulário)

**Mobile:**

Dia 1-3: Onboarding restante (Morador, Empresa, Parceiro)
- Espelhamento das telas web
- Camera pra documentos e fotos
- CEP auto-fill

Dia 4-5: Condominium + Company screens mobile
- Tela de cadastro condomínio
- Tela de busca condomínio por ID
- QR Code scanner pra vincular condomínio

**Entrega Semana 5:** todos os onboardings funcionais, condomínio e empresa cadastram, QR code funciona.

---

### Semana 6 (24/04 → 01/05): OPENSEARCH + BUSCA + TRIAL

**Full-Stack:**

Dia 1-2: OpenSearch Setup
- AWS OpenSearch Service (t3.small.search — dev)
- Index: companies (razao_social, fantasia, categories, city, state, description, rating)
- Index: sindicos (name, city, state, experience, type, markers)
- Index: parceiros (name, category, subcategory, bairro, city, promotions_active)
- Sync pipeline: database → OpenSearch (via events)
- API: GET /search?q=&type=&category=&location=

Dia 3-4: Search Frontend (Web)
- TELA: Busca Geral
  - Input com autocomplete
  - Filtros laterais: tipo (empresa/síndico/parceiro), categoria, localização, rating
  - Resultados em cards (thumbnail, nome, categoria, localização, badge de plano)
  - Paginação / infinite scroll
- TELA: Detalhe do resultado (preview do perfil público)
- Middleware de visibilidade: filtrar resultados baseado no plano do viewer
  - Base/N1: perfis limitados
  - Plus/Pro: perfis completos

Dia 5: Trial Logic End-to-End
- Middleware backend: checkFeatureAccess(userId, feature)
- Frontend: useFeatureAccess hook
- Componente TrialBlocker: overlay com mensagem de conversão
- Tela de upgrade: redireciona pro Stripe Customer Portal
- Testar fluxo completo: register → trial → usar features → bloqueio → upgrade → desbloqueio

**Mobile:**

Dia 1-3: Search Mobile
- Tela de busca com filtros
- Cards de resultado
- Integração com mesma API

Dia 4-5: Trial Mobile
- TrialBlocker component
- Deep link pra Stripe checkout (WebView ou browser externo)

**Entrega Semana 6:** busca funciona com OpenSearch, trial bloqueia/desbloqueia features, Stripe cobra.

---

### Semana 7 (01/05 → 08/05): TESTES + POLISH + DEPLOY PROD

**Full-Stack:**

Dia 1-2: Testes
- Testes de integração: auth flow completo (register → verify → login → refresh → logout)
- Testes de integração: onboarding completo por persona
- Testes de integração: Stripe webhooks (subscription created → active → cancelled → expired)
- Testes de integração: search (indexação → query → filtros → visibilidade)
- Testes de ABAC: verificar que cada role vê apenas o que deve

Dia 3-4: Bug fixes + Edge cases
- Token refresh silencioso
- Session recovery após crash
- Offline handling (queued actions)
- Error boundaries em todas as rotas
- Loading states em todas as telas
- Empty states em listas
- Validação de formulário com feedback visual

Dia 5: Deploy Production
- AWS production environment
- RDS Multi-AZ
- ECS com auto-scaling
- CloudFront invalidation
- Route53 apontando pra prod
- Zitadel production config
- Stripe live mode
- Monitoring + alertas
- Smoke tests em prod

**Mobile:**

Dia 1-3: Testes mobile
- Testes em dispositivos reais (iOS + Android)
- Deep linking
- Push notification test
- Performance profiling

Dia 4-5: Build & Submit
- EAS Build production
- TestFlight (iOS)
- Google Play Internal Testing (Android)

**Entrega Marco 1 (08/05):**
- ✅ Infra AWS completa (prod + staging)
- ✅ Zitadel configurado com roles e multi-tenancy
- ✅ Stripe integrado com trial e planos
- ✅ Design system completo (web + mobile)
- ✅ Auth end-to-end (login, cadastro, verificação, recuperação)
- ✅ Onboarding completo (4 personas com todos os campos)
- ✅ Condominium registry (cadastro, ID, QR code, mandato)
- ✅ Company registry (perfil institucional)
- ✅ OpenSearch com busca funcional (empresas, síndicos, parceiros)
- ✅ Trial logic end-to-end
- ✅ Mobile: auth + onboarding + busca espelhados
- **Total telas web: ~35-40**
- **Total telas mobile: ~25-30**

---

## MARCO 2 — CORE GOVERNANCE (08/05 → 20/06 = 6 semanas)

**Objetivo:** síndico gerencia condomínio completo, morador acompanha gestão, connect me funciona.

**Entrega final:** governança condominial operacional — timeline, plano diretor, eventos, comunicados, validações, avaliação, connect me.

---

### Semana 8 (08/05 → 15/05): TIMELINE + PLANO DIRETOR

**Full-Stack:**

Dia 1-2: Timeline Backend
- Migration: timeline_publications (id, tenant_id, type, title, description, area_impactada, natureza, importancia, risco, impacto_esperado, proxima_acao, empresa_id, periodo_inicio, periodo_fim, garantia, orientacao, anexos, master_plan_action_id, visibility, version, created_by, created_at)
- Migration: timeline_versions (publication_id, version, data JSONB, justificativa, created_at)
- Migration: timeline_adendos (id, publication_id, reason, description, changes_summary, created_by, created_at)
- API: POST /condominiums/:id/timeline (create — append only)
- API: GET /condominiums/:id/timeline (list with filters)
- API: GET /condominiums/:id/timeline/:pubId (detail with versions + adendos)
- API: POST /condominiums/:id/timeline/:pubId/adendo (create adendo)
- API: PUT /condominiums/:id/timeline/:pubId (edit — creates new version, requires justificativa)
- Event: TimelinePublished → triggers Master Plan status update
- Imutabilidade: após publicação, record é locked; edição cria versão nova

Dia 3-4: Timeline Frontend (Web)
- TELA S11: Linha do Tempo — lista cronológica com filtros (período, tipo, empresa, área, status, vínculo PD)
- TELA S12: Criar Atividade — formulário completo com 26 tipos, 9 riscos, 8 próximas ações, vínculo PD opcional, pergunta aos moradores opcional
- TELA S13: Editar Atividade — justificativa obrigatória, cria nova versão
- TELA S14: Histórico — filtros avançados, cards com badge de tipo
- Context Confirmation modal: "Publicar para Condomínio X?" (obrigatório pra multi-condo)

Dia 5: Plano Diretor Backend + Frontend
- Migration: master_plan_actions (id, tenant_id, name, description, area_impactada, natureza, importancia, impacto_esperado, prazo_limite, status, created_by, created_at)
- Status automático: listener de TimelinePublished → se publicação tem master_plan_action_id → atualiza status
- "Atrasada" calculada automaticamente (prazo_limite < now() AND status != concluida)
- API: CRUD /condominiums/:id/master-plan
- TELA S7: Plano Diretor (botões: cadastrar ação, ver ações)
- TELA S8: Cadastrar Ação (formulário com enums)
- TELA S9: Lista Ações (cards com status colorido)
- TELA S10: Detalhe Ação (info + seção "Registros vinculados" da Timeline)

**Mobile:**

Dia 1-5: Timeline + PD Mobile
- Telas de leitura da Timeline (morador view — M6, M7)
- Telas de leitura do Plano Diretor (morador view — M8, M9)
- Cards com indicadores visuais

**Entrega Semana 8:** Timeline imutável funcionando, Plano Diretor reativo, tudo integrado.

---

### Semana 9 (15/05 → 22/05): EVENTOS + COMUNICADOS + VALIDAÇÕES

**Full-Stack:**

Dia 1-2: Eventos
- Migration: events (id, tenant_id, title, description, event_type, start_date, end_date, location, attachments, require_confirmation, require_acknowledgment, status, created_by)
- Migration: event_responses (event_id, morador_id, response_type [ciente, confirmado], timestamp)
- API: CRUD /condominiums/:id/events
- API: POST /condominiums/:id/events/:eventId/respond
- TELA S15: Eventos (lista com métricas: taxa confirmação, taxa ciência)
- TELA S16: Criar Evento (13 tipos, campos, anexos)
- TELA S17: Lista Eventos (cards com status e respostas)
- TELA M10: Eventos (morador — cards com botões Ciente / Confirmar Participação)

Dia 3: Comunicados
- Migration: comunicados (id, tenant_id, title, description, tipo, priority, start_date, expiration_date, attachments, require_acknowledgment, origin [sindico, empresa], status, created_by)
- Migration: comunicado_reads (comunicado_id, morador_id, read_at)
- API: CRUD /condominiums/:id/comunicados
- TELA S23: Comunicados (lista + botão criar)
- TELA S24: Criar Comunicado (8 tipos, campos)
- TELA S25: Lista Comunicados (cards com taxa de leitura)
- TELA M11: Comunicados (morador — cards com botão Ciente)

Dia 4-5: Validation Hub
- Migration: pending_validations (id, tenant_id, item_type [execution, activity, comunicado], item_id, company_id, status [pending, validated, rejected, adjustment_requested], rejection_reason, validator_id, validated_at)
- API: GET /condominiums/:id/validations (list pending)
- API: POST /condominiums/:id/validations/:id/validate
- API: POST /condominiums/:id/validations/:id/reject
- API: POST /condominiums/:id/validations/:id/request-adjustment
- TELA S26: Validações Pendentes (3 seções: execuções, atividades, comunicados)
- Badge no sidebar com contagem de pendências
- Após validação: publicação automática na Timeline

**Mobile:**

Dia 1-5: Eventos + Comunicados Mobile
- M10: Eventos (morador)
- M11: Comunicados (morador)
- Push notifications pra novos eventos e comunicados

**Entrega Semana 9:** eventos com confirmação, comunicados com leitura, validações centralizadas.

---

### Semana 10 (22/05 → 29/05): MORADOR JOURNEY COMPLETA

**Full-Stack:**

Dia 1-2: Home do Morador + Condominium Selector
- TELA M1: Painel do Morador (mensagem institucional, botões: selecionar condo, vínculo, avaliar)
- TELA M2: Selecionar Condomínio (buscar por ID, ver vinculados)
- TELA M3: Buscar Condomínio (campo ID, validação, exibe dados)
- TELA M4: Cadastro da Unidade (bloco/torre, número, tipo, vínculo, termo de ciência)
- TELA M5: Home do Condomínio (cards: Timeline, PD, Eventos, Comunicados, Avaliação, Vínculo)

Dia 3: Avaliação da Gestão
- Migration: evaluations (id, tenant_id, morador_id, periodo, satisfacao_gestao, transparencia_comunicacao, percepcao_evolucao, created_at)
- API: POST /condominiums/:id/evaluations
- API: GET /condominiums/:id/evaluations/summary (agregado anônimo pra síndico)
- TELA M12: Avaliar Gestão (3 perguntas, escala 1-10, obrigatória bimestralmente)
- Regra: não mostrar autor, registrar apenas estatística

Dia 4: Meu Vínculo
- TELA M13: Meu Vínculo (bloco, unidade, tipo vínculo, status)
- TELA M14: Gerenciar Acessos (incluir/remover dependentes, atualizar status)
- TELA M15: Encerrar Vínculo (4 motivos, mantém histórico)
- Regras: 1 titular + 2 dependentes max, unidade vazia remove dependentes

Dia 5: Pergunta aos Moradores
- Migration: morador_questions (id, timeline_publication_id, question_text, question_type, options[], expiration_date, target_audience, created_at)
- Migration: morador_answers (question_id, morador_id, answer, timestamp)
- Bloco dentro de Criar Atividade (S12): toggle "Adicionar pergunta?"
- Tipos: sim_nao_nao_sei, multiple_choice, scale_1_to_5, campo_aberto
- Público: todos moradores ou apenas titulares
- Resultado consolidado vira publicação na Timeline

**Mobile:**

Dia 1-5: Jornada Morador completa mobile
- M1-M5: Painel, Seleção, Busca, Cadastro Unidade, Home Condo
- M12-M15: Avaliação, Vínculo, Gerenciar, Encerrar
- QR scanner pra buscar condomínio

**Entrega Semana 10:** jornada do morador completa web+mobile, avaliação funciona, vínculo gerenciável.

---

### Semana 11 (29/05 → 05/06): CONNECT ME + SÍNDICO HOME

**Full-Stack:**

Dia 1-2: Connect Me Backend
- Migration: connect_me_requests (id, requester_id, requester_type, target_type, target_id, category, subcategory, description, urgency, status, contact_revealed, contact_revealed_at, lgpd_consent_version, created_at, expires_at)
- Migration: connect_me_lgpd_log (request_id, data_revealed, recipient_id, timestamp, ip_address, consent_version)
- API: POST /connect-me (create request)
- API: GET /connect-me/received (list incoming)
- API: POST /connect-me/:id/interested (reveal contact + LGPD log)
- API: POST /connect-me/:id/refuse (with structured reason)
- Auto-close após 48h (cron job ou EventBridge scheduled)
- Quotas: Morador Base 2/ano, Morador Pg 4/ano, Síndico N1 2/ano, N2 4/ano, N3 unlimited

Dia 3: Connect Me Frontend (Web)
- TELA S18: Connect Me hub (criados, recebidos)
- TELA S19: Criar Connect Me (categoria, subcategoria, descrição, cidade, bairro, compartilhar dados)
- TELA S20: Connect Me Recebido (dados do morador, botões Interesse/Recusar)
- Botão "Denunciar Assédio" com log obrigatório

Dia 4: Home da Governança (Síndico)
- TELA S1: Entrada da Governança (mensagem institucional, botão Entrar — só 1° acesso)
- TELA S2: Meus Condomínios (cards ativos/encerrados, até 15, ordenado por ativos)
- TELA S6: Home da Governança (cards: PD, Timeline, Eventos, Connect Me, Registros, Comunicados, Validações, Dashboard, Compliance)
- Badges de pendência em cada card

Dia 5: Dashboard do Síndico
- TELA S27: Dashboard
- Indicadores: moradores cadastrados, % unidades com acesso, avaliação média, ações PD por status, atividades publicadas, eventos realizados, comunicados enviados, registros execução, taxa resposta perguntas, tendência opinião, taxa leitura, taxa confirmação
- Filtros por período
- Charts com Recharts ou Chart.js

**Mobile:**

Dia 1-5: Connect Me + Home Mobile
- Connect Me (receber e responder do lado do síndico)
- Home do morador com connect me (se pagante)
- Push notification pra novo Connect Me

**Entrega Semana 11:** Connect Me end-to-end com LGPD, home do síndico com dashboard e badges.

---

### Semana 12 (05/06 → 12/06): COMPLIANCE + EMPRESA SÍNDICA

**Full-Stack:**

Dia 1-2: Compliance
- Migration: compliance_terms (id, tenant_id, type [responsabilidade, declaracao_anual], user_id, version, ip_address, signed_at)
- Migration: audit_trail (id, tenant_id, user_id, action_type, entity_type, entity_id, before_state, after_state, ip_address, timestamp)
- TELA S28: Compliance (cards: Termo, Declaração, Auditoria)
- TELA S29: Termo de Responsabilidade (texto jurídico, checkbox, assinar)
- TELA S30: Declaração Anual (obrigatória 12 meses, alerta dashboard)
- TELA S31: Auditoria (lista imutável, 15+ tipos ação, filtros)
- Audit trail automático em todas as ações da governança

Dia 3: Empresa Síndica Vinculada
- Migration: empresa_sindica_links (id, condominium_id, company_name, cnpj, email, token, status, permissions JSONB, linked_at)
- API: POST /condominiums/:id/empresa-sindica (invite via token)
- API: PUT /condominiums/:id/empresa-sindica/:id/permissions
- API: DELETE /condominiums/:id/empresa-sindica/:id (revoke)
- TELA S5: Cadastro/Vínculo Empresa Síndica (campos + permissões toggle)
- Permissões: visualizar, editar, ocultar publicações, validar registros, validar comunicados
- Todas ações da empresa síndica entram na auditoria

Dia 4-5: Testes de integração Marco 2
- Fluxo completo: síndico cria condomínio → cria plano diretor → publica atividade → status PD atualiza
- Fluxo completo: morador vincula → vê timeline → avalia gestão → resultado agregado
- Fluxo completo: síndico cria Connect Me → empresa recebe → mostra interesse → dados revelados
- Fluxo completo: empresa envia registro → aparece em validações → síndico valida → publica na Timeline
- Compliance: assinar termo → declaração anual → auditoria registra
- ABAC: morador não vê o que não deve, empresa não acessa condo sem permissão

**Mobile:**

Dia 1-5: Testes mobile + polish
- Testes end-to-end nos fluxos do morador
- Push notifications testadas
- Performance profiling
- Bug fixes

---

### Semana 13 (12/06 → 20/06): TESTES + BUG FIXES + DEPLOY

**Full-Stack + Mobile:**

Dia 1-3: Bug fixes e edge cases
- Multi-tenancy: testar síndico trocando entre condomínios
- Context confirmation: garantir que nunca publica no condo errado
- Imutabilidade: garantir que timeline entries não são editáveis (só versionáveis)
- Plano Diretor reativo: testar todos os caminhos de status
- Trial: verificar bloqueio retroativo quando expira
- Quotas: verificar reset anual do Connect Me
- Offline mobile: graceful degradation

Dia 4: Deploy produção Marco 2
- Database migrations em prod
- OpenSearch reindex
- Feature flags pra novos módulos
- Smoke tests

Dia 5: Documentação
- API docs (OpenAPI/Swagger)
- README atualizado
- Runbook de deploy

**Entrega Marco 2 (20/06):**
- ✅ Timeline imutável com versionamento e adendos
- ✅ Plano Diretor reativo (status automático via Timeline)
- ✅ Eventos com confirmação e ciência
- ✅ Comunicados com tracking de leitura
- ✅ Validation Hub centralizado
- ✅ Jornada do Morador completa (M1-M15)
- ✅ Jornada do Síndico core (S1-S31 exceto assembleia)
- ✅ Connect Me Síndico→Empresa funcional com LGPD
- ✅ Dashboard do Síndico com indicadores
- ✅ Compliance básico (termo + declaração + auditoria)
- ✅ Empresa Síndica vinculada com permissões
- ✅ Pergunta aos Moradores com resultado na Timeline
- ✅ Mobile: morador journey completa + connect me + push
- **Total telas web acumulado: ~75-80**
- **Total telas mobile acumulado: ~50-55**

---

## MARCO 3 — COMMERCIAL + CONTENT (20/06 → 20/07 = 4 semanas)

**Objetivo:** empresa opera dentro da plataforma, mídia funciona, agência gerencia conteúdo.

**Entrega final:** ecossistema completo do fluxo síndico↔empresa↔morador.

---

### Semana 14 (20/06 → 27/06): PAINEL EMPRESARIAL

**Full-Stack:**

Dia 1-2: Service & Contract Backend
- Migration: execution_records (id, company_tenant_id, condominium_tenant_id, tipo_servico, descricao, area_impactada, natureza, status_servico, data_execucao, garantia, orientacoes, materiais, attachments, deal_id, submission_status, rejection_reason, created_by, created_at)
- Migration: technical_activities (id, company_tenant_id, condominium_tenant_id, tipo, descricao, area_impactada, importancia, risco, impacto, recomendacao, attachments, submission_status, created_by, created_at)
- Migration: technical_comunicados (id, company_tenant_id, condominium_tenant_id, deal_id, tipo, titulo, descricao, attachments, submission_status, created_by, created_at)
- API: CRUD pra cada + submit/validate/reject/adjust workflows
- Events: ExecutionSubmitted, ActivitySubmitted, ComunicadoSubmitted → cria PendingValidation

Dia 3-4: Painel Empresarial Frontend (Web)
- TELA E1: Home Painel Empresarial (cards dinâmicos: oportunidades, propostas, votações, negócios, execuções, garantias, avaliações)
- TELA E2: Oportunidades Connect Me (lista de oportunidades recebidas)
- TELA E3: Recusar Oportunidade (6 motivos estruturados)
- TELA E4: Detalhe Oportunidade (dados revelados após interesse)
- TELA E5: Registrar Execução (formulário completo)
- TELA E6: Atividade Técnica (formulário)
- TELA E7: Comunicados Técnicos (formulário)
- TELA E8: Status de Envios (3 seções, 7 status)
- TELA E9: Detalhe do Registro (feedback do síndico, editar/reenviar)

Dia 5: Company Dashboard + Profile
- TELA E10: Dashboard Empresarial (13 indicadores, charts, pendências)
- TELA E11: Perfil Institucional (edição com regras de conteúdo)
- TELA E12: Usuários da Empresa (adicionar/remover, 2 roles: admin + operador técnico)

**Mobile:**

Dia 1-5: Painel Empresarial Mobile
- E1: Home (cards)
- E2-E4: Oportunidades (receber, recusar, detalhar)
- E5: Registrar Execução (com câmera pra fotos)
- E10: Dashboard básico

**Entrega Semana 14:** painel empresarial completo, empresa recebe oportunidades e registra serviços.

---

### Semana 15 (27/06 → 04/07): AGÊNCIA + MEDIA + PLAYER

**Full-Stack:**

Dia 1-2: Agência de Marketing
- Migration: agency_links (id, company_tenant_id, agency_id, token, status, permissions, linked_at)
- Migration: marketing_link_requests (id, company_id, agency_id, service_type, description, priority, status, created_at)
- TELA E13: Gestão Agência (lista, status)
- TELA E14: Convidar Agência (campos + token)
- TELA E15: Gerenciar Acessos (encerrar a qualquer momento)
- TELA E16: Formulário Marketing Link
- TELA MK1: Painel da Agência (empresas, vídeos, biblioteca, dashboard, marketing links)
- TELA MK2: Empresas Atendidas
- TELA MK3: Gerenciar Empresa (publicar vídeo, biblioteca, dashboard)
- TELA MK4: Publicar Vídeo (12 tipos, subtema automático, checkbox autorizar votação)
- TELA MK5: Biblioteca de Vídeos
- TELA MK6: Dashboard da Empresa (views, retenção, tempo médio)
- TELA MK7: Dashboard Geral (empresas atendidas, total views, mais acessado)
- TELA MK8: Marketing Link Recebido (lista, marcar atendido)

Dia 3-4: Media Integration
- Mux Adapter: upload via signed URL, webhook pra processing complete, HLS streaming URL
- S3 Adapter: upload documentos/imagens/PDFs, signed URLs com expiração
- Migration: media_files (id, tenant_id, tenant_type, uploader_id, file_type, mux_asset_id, mux_playback_id, s3_key, status, duration, thumbnail_url, created_at)
- API: POST /media/upload-url (gera signed URL)
- API: POST /media/complete (webhook do Mux/S3)
- API: GET /media/:id/playback (retorna HLS URL com access control)
- Upload com trava trimestral: verificar último upload do tipo nos últimos 90 dias
- Limites mensais por plano: N1(4), N2(8), N3(12), Plus(8), Pro(12)

Dia 5: Player de Vídeo
- Player customizado com HLS.js (web) / expo-av (mobile)
- Paywall: se viewer é Base e vídeo é de empresa → corta em 25%
- Contagem de view: registra se >70% assistido
- Analytics: views, watch time, completion rate
- Regra de visibilidade: vídeos de empresa NÃO aparecem pra moradores automaticamente, apenas em votação de fornecedor na assembleia quando autorizar_exibicao_votacao=true

**Mobile:**

Dia 1-5: Player + Media mobile
- Player nativo com expo-av
- Upload de vídeo/foto da galeria ou câmera
- MK1 básico (se agência usar mobile)

**Entrega Semana 15:** agência funciona, upload de vídeo funciona, player com paywall funciona.

---

### Semana 16 (04/07 → 11/07): PROPOSALS + DEALS + EVALUATION

**Full-Stack:**

Dia 1-2: Proposals + Voting
- Migration: proposals (id, company_id, condominium_id, connect_me_id, title, description, scope, cost, duration, attachments, status, version, created_at)
- Migration: votings (id, condominium_id, title, description, type, start_date, end_date, status, proposals[], quorum_required, created_at)
- Migration: votes (id, voting_id, morador_id, proposal_id, timestamp)
- API: CRUD proposals + submit/validate/reject
- API: CRUD votings + cast vote + close + results
- Telas de proposta (síndico valida, morador vê na votação)
- Tela de votação (1 voto por unidade, quórum, resultados)

Dia 3: Closed Deal
- Migration: deals (id, condominium_id, company_id, proposal_id, description, cost, start_date, end_date, payment_terms, guarantee, company_signature, sindico_signature, status, locked_at, created_at)
- Dupla confirmação: empresa assina → síndico assina → status=confirmed → locked
- Após confirmação: publica automaticamente na Timeline
- Apenas adendo após lock
- Telas de negócio fechado com termos jurídicos

Dia 4: Company Evaluation
- Migration: company_evaluations (id, company_id, condominium_id, deal_id, qualidade_servico, cumprimento_prazo, atendimento, custo_beneficio, recomendacao, text_feedback, evaluator_id, created_at)
- API: POST /evaluations (after deal completion)
- API: GET /companies/:id/evaluations/summary (agregado anônimo)
- Tela de avaliação (5 critérios, escala 1-10)
- Resultados visíveis pro empresa (anônimo) e pra outros síndicos

Dia 5: Post-Deal Comunicados
- Empresa cria comunicado vinculado ao deal
- 5 tipos: orientação técnica, recomendação, alerta, conclusão, atualização garantia
- Passa por validação do síndico
- Após validação publica na Timeline e notifica moradores

**Mobile:**

Dia 1-5: Propostas + Deals mobile
- Visualização de propostas
- Votação mobile
- Avaliação de empresa

**Entrega Semana 16:** fluxo comercial completo: proposta → votação → deal → execução → avaliação.

---

### Semana 17 (11/07 → 20/07): TESTES FINAIS + POLISH + DEPLOY

**Full-Stack + Mobile:**

Dia 1-3: Testes end-to-end completos
- Fluxo completo: empresa recebe oportunidade → mostra interesse → envia proposta → síndico valida → morador vota → deal fechado → empresa executa → síndico valida registro → publica na Timeline → avalia empresa
- Agência: empresa convida → agência publica vídeo → vídeo disponível no perfil
- Media: upload → processing → player → paywall → view count
- Cross-tenant: empresa opera em múltiplos condomínios, dados isolados
- ABAC: operador técnico vs administrador (permissões corretas)
- Billing: trial expira → features bloqueadas → upgrade → desbloqueio retroativo
- Regression: todos os fluxos do Marco 1 e 2 continuam funcionando

Dia 4-5: Bug fixes + Deploy produção
- Performance tuning (queries N+1, index optimization)
- Mobile: testes em dispositivos reais, performance
- Deploy produção com migrations
- OpenSearch reindex com novos dados
- Feature flags
- Smoke tests completos
- Mobile: submit pra TestFlight + Play Store

**Entrega Marco 3 (20/07):**
- ✅ Painel Empresarial completo (E1-E16)
- ✅ Agência de Marketing completa (MK1-MK8)
- ✅ Service & Contract: execução, atividades técnicas, comunicados
- ✅ Submission tracking com feedback do síndico
- ✅ Proposals + Supplier Voting
- ✅ Closed Deal com dupla confirmação e imutabilidade
- ✅ Company Evaluation (5 critérios)
- ✅ Media: upload Mux, player HLS, paywall 25%, trava trimestral
- ✅ Company Users (admin + operador técnico)
- ✅ Marketing Link
- ✅ Mobile: painel empresa + player + votação
- **Total telas web acumulado: ~110-114**
- **Total telas mobile acumulado: ~75-80**

---

## O QUE FICA PRA DEPOIS (Marco 4+)

| Feature | Estimativa | Prioridade |
|---------|-----------|-----------|
| Assembly Engine (ao vivo completa) | 8-10 semanas | Alta |
| LMS/Academia (15 telas, 5 áreas) | 4-5 semanas | Média |
| Vizinhança completa (29 telas, 4 jornadas) | 3-4 semanas | Média |
| Banco de Talentos (11 telas + visão empresa) | 3-4 semanas | Baixa |
| Painel Admin MasterSíndico | 3-4 semanas | Alta |
| Connect Me: Morador→Síndico + Empresa→Empresa | 2 semanas | Média |
| Dossiê Exportável + Relatório de Gestão | 1-2 semanas | Média |
| Mobile polish + offline mode + deep linking | 2-3 semanas | Média |

**Estimativa pra completar TUDO:** +20-25 semanas adicionais (Outubro/Novembro 2026)

---

## RISCOS IDENTIFICADOS

| Risco | Probabilidade | Impacto | Mitigação |
|-------|-------------|---------|-----------|
| Zitadel multi-tenancy não encaixa perfeitamente | Média | Alto | Semana 2 é discovery — se não funcionar, pivot pra Zitadel + custom tenant layer |
| Mux latency no Brasil | Baixa | Médio | Cloudflare Stream como fallback |
| OpenSearch sizing incorreto | Baixa | Baixo | Start small, scale up |
| Mobile dev bottleneck | Alta | Alto | Priorizar morador no mobile, empresa e agência ficam web-only inicialmente |
| Scope creep do cliente | Alta | Alto | Requirements.md é o contrato — o que não está ali não entra |
| AI-generated code com bugs sutis | Média | Médio | Code review rigoroso, testes de integração em todo fluxo cross-domain |

---

## DECISÕES QUE PRECISAM SER TOMADAS AGORA

1. **Backend framework:** NestJS ou Fastify? (recomendo Fastify pela performance + simplicidade)
2. **Mobile framework:** React Native com Expo ou outro?
3. **Zitadel:** self-hosted em ECS ou Zitadel Cloud?
4. **Mux ou Cloudflare Stream?**
5. **Database:** RDS PostgreSQL single instance ou Aurora Serverless?
6. **O mobile dev começa junto no dia 20/03 ou entra depois?**

---

**FIM DO PLANO DE EXECUÇÃO**