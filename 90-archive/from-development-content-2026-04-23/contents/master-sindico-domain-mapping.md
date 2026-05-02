# Master Síndico — Domain Mapping & Architecture

> **Versão:** 1.0 — Março 2026
> **Tipo:** Documento Técnico — Domain-Driven Design
> **Complementa:** master-sindico-arquitetura-solucoes.md
> **Status:** Em construção — aguardando documentos do cliente

---

## 1. PRINCÍPIOS ARQUITETURAIS

### 1.1 Erlang OTP Pattern

Pensamos o sistema como **processos isolados** que se comunicam via mensagens:

```
┌─────────────────────────────────────────────────────────────┐
│                      SUPERVISOR TREE                         │
│                                                              │
│                    ┌──────────────┐                         │
│                    │   IDENTITY   │ ← Root Supervisor       │
│                    └──────┬───────┘                         │
│                           │                                  │
│          ┌────────────────┼────────────────┐                │
│          │                │                │                │
│    ┌─────▼─────┐    ┌────▼────┐    ┌──────▼──────┐        │
│    │INSTITUTION│    │COMMERCIAL│    │   CONTENT   │        │
│    │    AL     │    │          │    │             │        │
│    └───────────┘    └──────────┘    └─────────────┘        │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

**Regras OTP aplicadas:**
- Cada HIGH-DOMAIN é um supervisor independente
- Cada MID-DOMAIN é um worker process
- Falha em um worker não derruba o supervisor
- Comunicação via message passing (eventos/commands)
- State isolado por processo (tenant isolation)

### 1.2 DNS Architecture Analogy

Pensamos o sistema como **resolução de nomes**:

```
mastersindico.com (ROOT)
    │
    ├── accounts.mastersindico.com     → IDENTITY
    ├── governance.mastersindico.com   → INSTITUTIONAL
    ├── marketplace.mastersindico.com  → COMMERCIAL
    └── media.mastersindico.com        → CONTENT
```

Cada HIGH-DOMAIN é um **subdomínio autônomo** que pode escalar independentemente.

---

## 2. SEPARAÇÃO: EXTERNAL vs INTERNAL

### 2.1 EXTERNAL SERVICES (O que NÃO somos)

Serviços de terceiros que **consumimos** mas **nunca implementamos**:

| Service | Provider | Protocol | Consumed By | Purpose |
|---------|----------|----------|-------------|---------|
| **Video Streaming** | Mux / Cloudflare Stream | REST API + HLS | Content/Media | Upload, encoding, adaptive streaming |
| **Payment Gateway** | Mercado Pago | REST API + Webhooks | Identity/Billing | Assinaturas, cobrança recorrente |
| **Email Transacional** | AWS SES / SendGrid | SMTP + REST API | Identity, Commercial | Validação, notificações, Connect Me |
| **SMS Gateway** | Twilio / Zenvia | REST API | Identity | OTP, verificação 2FA |
| **Object Storage** | AWS S3 / Cloudflare R2 | S3 API | Content, Institutional | PDFs, imagens, anexos, documentos |
| **CEP Lookup** | ViaCEP | REST API | Institutional/Registry | Auto-fill endereço |
| **Real-time (se necessário)** | Socket.io / Ably / Pusher | WebSocket | Assembly Engine | Assembleia ao vivo (pendente decisão) |

**Adapter Layer (Anti-Corruption Layer):**
```
Internal Domain → Adapter → External Service
                    ↑
                    └── Abstração que permite trocar provider sem impactar domínios
```

### 2.2 INTERNAL DOMAINS (O que somos)

Domínios que **implementamos** e **mantemos**:

#### HIGH-DOMAINS (Nível 1 — Autônomos)

| Domain | Responsibility | Depends On | Depended By |
|--------|---------------|------------|-------------|
| **Identity** | Autenticação, autorização, billing, planos | NINGUÉM | TODOS |
| **Institutional** | Governança condominial (tenant: condomínio) | Identity | Commercial, Content |
| **Commercial** | Marketplace, relações cross-tenant | Identity | Institutional, Content |
| **Content** | Distribuição de mídia, busca, visibilidade | Identity | Institutional, Commercial |

#### MID-DOMAINS (Nível 2 — Bounded Contexts)

**Dentro de IDENTITY:**
- Authentication
- Authorization (ABAC)
- Billing & Subscriptions
- Onboarding
- User Registry

**Dentro de INSTITUTIONAL:**
- Condominium Registry
- Tenant Membership
- Governance Engine
  - Timeline (LOW)
  - Master Plan (LOW)
  - Events (LOW)
  - Communications (LOW)
  - Validations (LOW)
  - Polls (LOW)
  - Dashboard (LOW)
  - Compliance (LOW)
  - Evaluation (LOW)
- Assembly Engine (pendente escopo)

**Dentro de COMMERCIAL:**
- Connect Me
- Service & Contract
- Institutional Profile
- Marketing Agency

**Dentro de CONTENT:**
- Media
- Search
- Video Curriculum (pendente confirmação)
- Player

---

## 3. DOMAIN CONTRACTS (API Contracts)

### 3.1 Identity → Institutional

**Contract:** "Esse usuário tem contexto nesse condomínio?"

```typescript
// Request
GET /api/identity/authorization/check
{
  userId: string,
  condominiumId: string,
  action: "view_timeline" | "create_activity" | "validate_execution"
}

// Response
{
  allowed: boolean,
  role: "sindico" | "morador_titular" | "morador_dependente" | "empresa_sindica",
  plan: "trial" | "base" | "plus" | "pro",
  features: string[]
}
```

### 3.2 Identity → Commercial

**Contract:** "Esse plano permite essa ação comercial?"

```typescript
// Request
GET /api/identity/authorization/check
{
  userId: string,
  action: "create_connect_me" | "view_opportunity" | "register_execution"
}

// Response
{
  allowed: boolean,
  quota: {
    used: number,
    limit: number,
    resetDate: string
  }
}
```

### 3.3 Identity → Content

**Contract:** "Esse plano permite ver esse conteúdo?"

```typescript
// Request
GET /api/identity/authorization/check
{
  userId: string,
  contentType: "video_institucional" | "video_curriculum" | "document",
  contentId: string
}

// Response
{
  allowed: boolean,
  accessLevel: "full" | "preview_25" | "blocked",
  reason?: string
}
```

### 3.4 Institutional → Commercial

**Contract:** "Esse condomínio precisa de serviço, essa empresa oferece"

```typescript
// Event (Institutional emite)
EVENT: ConnectMeCreated
{
  requestId: string,
  condominiumId: string,
  sindicoId: string,
  category: string,
  subcategory: string,
  description: string,
  location: { city: string, neighborhood: string },
  shareContact: boolean
}

// Commercial consome e cria Opportunity
```

### 3.5 Commercial → Institutional

**Contract:** "Essa empresa enviou registro, síndico valida"

```typescript
// Event (Commercial emite)
EVENT: ExecutionRegistered
{
  registrationId: string,
  companyId: string,
  condominiumId: string,
  serviceType: string,
  description: string,
  impactedArea: string,
  attachments: string[]
}

// Institutional consome e cria Pending Validation
```

### 3.6 Content → Identity

**Contract:** "Quem pode ver resolve com o plano do usuário"

```typescript
// Request
GET /api/content/media/{videoId}/access
{
  userId: string
}

// Response (após consultar Identity)
{
  canView: boolean,
  streamUrl?: string,
  accessLevel: "full" | "preview_25" | "blocked"
}
```

---

## 4. TENANT ISOLATION STRATEGY

### 4.1 Multi-Tenancy Model

**Tipo:** Hybrid (Shared Database + Tenant Scoping)

```
┌─────────────────────────────────────────────────────────────┐
│                      DATABASE LAYER                          │
│                                                              │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐     │
│  │   IDENTITY   │  │ INSTITUTIONAL│  │  COMMERCIAL  │     │
│  │   (Shared)   │  │  (Scoped)    │  │   (Scoped)   │     │
│  └──────────────┘  └──────────────┘  └──────────────┘     │
│                                                              │
│  Identity: sem tenant_id (global)                           │
│  Institutional: tenant_type='condominium', tenant_id=UUID   │
│  Commercial: tenant_type='company', tenant_id=UUID          │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

**Regras de Isolamento:**

1. **Identity Domain:** Sem tenant_id (dados globais)
   - users
   - subscriptions
   - plans
   - billing_history

2. **Institutional Domain:** tenant_type='condominium'
   - condominiums (tenant_id = condominium_id)
   - units
   - timeline_publications (tenant_id = condominium_id)
   - master_plan_actions (tenant_id = condominium_id)
   - events (tenant_id = condominium_id)
   - communications (tenant_id = condominium_id)

3. **Commercial Domain:** tenant_type='company'
   - companies (tenant_id = company_id)
   - opportunities (tenant_id = company_id)
   - execution_registrations (tenant_id = company_id)
   - technical_activities (tenant_id = company_id)

4. **Content Domain:** Hybrid
   - media_files (tenant_id pode ser condominium_id OU company_id)
   - tenant_type indica o tipo

**Middleware de Tenant Scoping:**
```typescript
// Toda query automática inclui tenant_id
const timeline = await db.timeline_publications
  .where('tenant_id', currentTenantId)
  .where('tenant_type', 'condominium')
  .get();
```

### 4.2 Cross-Tenant Operations

**Cenário:** Empresa registra execução para condomínio

```
1. User autenticado no tenant Company (tenant_id = company_123)
2. Cria execution_registration (armazenado em Commercial com tenant_id = company_123)
3. Emite evento: ExecutionRegistered
4. Institutional consome evento
5. Cria pending_validation (armazenado em Institutional com tenant_id = condominium_456)
6. Síndico valida (contexto: tenant_id = condominium_456)
7. Após validação, cria timeline_publication (tenant_id = condominium_456)
8. Emite evento: ExecutionValidated
9. Commercial consome e atualiza status (tenant_id = company_123)
```

**Regra de Ouro:** Cada domínio armazena dados no seu próprio tenant. Comunicação via eventos.

---

## 5. EVENT-DRIVEN ARCHITECTURE

### 5.1 Domain Events

**Padrão:** Event Sourcing Lite (eventos como integração, não como source of truth)

```
┌─────────────────────────────────────────────────────────────┐
│                      EVENT BUS                               │
│                                                              │
│  Publisher (Domain A) → Event Bus → Subscriber (Domain B)   │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

**Eventos Identificados:**

| Event | Publisher | Subscriber(s) | Payload |
|-------|-----------|---------------|---------|
| **UserRegistered** | Identity | Institutional, Commercial | userId, email, userType |
| **SubscriptionExpired** | Identity | ALL | userId, plan, expiredAt |
| **ConnectMeCreated** | Institutional | Commercial | requestId, condominiumId, sindicoId, category |
| **OpportunityAccepted** | Commercial | Institutional | opportunityId, companyId, sindicoId |
| **ExecutionRegistered** | Commercial | Institutional | registrationId, companyId, condominiumId |
| **ExecutionValidated** | Institutional | Commercial | registrationId, status, feedback |
| **TimelinePublished** | Institutional | Content (indexação) | publicationId, condominiumId, type |
| **MasterPlanActionUpdated** | Institutional | Dashboard | actionId, condominiumId, newStatus |
| **VideoUploaded** | Content | Commercial, Institutional | videoId, uploaderId, tenantId |

### 5.2 Event Bus Implementation

**Opções:**

1. **Fase 1 (Modular Monolith):** In-memory event bus
   - Simples, sem dependência externa
   - Eventos síncronos dentro do mesmo processo
   - Suficiente para MVP

2. **Fase 2 (Microservices):** Message Broker
   - RabbitMQ / AWS SQS / Google Pub/Sub
   - Eventos assíncronos entre serviços
   - Retry, dead-letter queue, garantias de entrega

---

## 6. DEPLOYMENT STRATEGY

### 6.1 Fase 1: Modular Monolith (Recomendado para MVP)

```
┌─────────────────────────────────────────────────────────────┐
│                   SINGLE DEPLOYMENT UNIT                     │
│                                                              │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐     │
│  │   IDENTITY   │  │ INSTITUTIONAL│  │  COMMERCIAL  │     │
│  │   Module     │  │    Module    │  │    Module    │     │
│  └──────────────┘  └──────────────┘  └──────────────┘     │
│                                                              │
│  ┌──────────────┐                                           │
│  │   CONTENT    │                                           │
│  │   Module     │                                           │
│  └──────────────┘                                           │
│                                                              │
│  Shared: Database, Event Bus (in-memory), Config            │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

**Vantagens:**
- Deploy simples (um único artefato)
- Desenvolvimento rápido (sem overhead de rede)
- Debugging fácil (tudo no mesmo processo)
- Transações ACID entre domínios (se necessário)

**Estrutura de Pastas (Backend):**
```
backend/
├── src/
│   ├── domains/
│   │   ├── identity/
│   │   │   ├── authentication/
│   │   │   ├── authorization/
│   │   │   ├── billing/
│   │   │   └── onboarding/
│   │   ├── institutional/
│   │   │   ├── condominium-registry/
│   │   │   ├── tenant-membership/
│   │   │   ├── governance-engine/
│   │   │   └── assembly-engine/
│   │   ├── commercial/
│   │   │   ├── connect-me/
│   │   │   ├── service-contract/
│   │   │   ├── institutional-profile/
│   │   │   └── marketing-agency/
│   │   └── content/
│   │       ├── media/
│   │       ├── search/
│   │       └── player/
│   ├── shared/
│   │   ├── event-bus/
│   │   ├── database/
│   │   ├── adapters/ (external services)
│   │   └── utils/
│   └── api/
│       ├── routes/
│       └── middleware/
```

### 6.2 Fase 2: Microservices (Quando escalar)

```
┌─────────────────────────────────────────────────────────────┐
│                      API GATEWAY                             │
│                                                              │
└────┬────────────────┬────────────────┬────────────────┬─────┘
     │                │                │                │
┌────▼─────┐    ┌─────▼────┐    ┌─────▼────┐    ┌─────▼────┐
│ IDENTITY │    │INSTITUTION│    │COMMERCIAL│    │ CONTENT  │
│ Service  │    │  Service  │    │ Service  │    │ Service  │
└──────────┘    └───────────┘    └──────────┘    └──────────┘
     │                │                │                │
     └────────────────┴────────────────┴────────────────┘
                      │
                ┌─────▼─────┐
                │ EVENT BUS │
                │ (RabbitMQ)│
                └───────────┘
```

**Quando migrar:**
- Equipe > 10 devs
- Necessidade de escalar domínios independentemente
- Requisitos de alta disponibilidade por domínio
- Assembleia ao vivo confirmada (Content precisa escalar separado)

---

## 7. DECISÕES ARQUITETURAIS PENDENTES

### 🔴 Críticas (Bloqueiam desenvolvimento)

| Decisão | Impacto | Depende de |
|---------|---------|-----------|
| **Assembleia ao vivo ou assíncrona?** | Se ao vivo: WebSocket, real-time, escala complexa | Cliente |
| **Matriz Funcional (planos)** | Define middleware de autorização em TODOS os domínios | Cliente |
| **Painel Admin MasterSíndico** | Define se é HIGH-DOMAIN ou módulo dentro de Identity | Cliente |

### 🟡 Importantes (Impactam arquitetura)

| Decisão | Impacto | Depende de |
|---------|---------|-----------|
| **Vizinhança/Comércio Local** | Define se é HIGH-DOMAIN ou MID dentro de Commercial | Cliente |
| **LMS (Learning Management System)** | Define se Content vira HIGH-DOMAIN ou split | Cliente |
| **Banco de Talentos** | Define se continua no escopo | Cliente |

### 🟢 Internas (Podemos decidir)

| Decisão | Opções | Recomendação |
|---------|--------|--------------|
| **Database** | PostgreSQL / MySQL / MongoDB | PostgreSQL (JSONB para flexibilidade) |
| **Event Bus (Fase 1)** | In-memory / Redis Pub/Sub | In-memory (simplicidade) |
| **Event Bus (Fase 2)** | RabbitMQ / AWS SQS / Google Pub/Sub | RabbitMQ (controle total) |
| **Object Storage** | AWS S3 / Cloudflare R2 / MinIO | Cloudflare R2 (custo) |
| **Video Streaming** | Mux / Cloudflare Stream | Mux (qualidade) |
| **Payment Gateway** | Mercado Pago / Stripe | Mercado Pago (Brasil) |

---

## 8. PRÓXIMOS PASSOS

### 8.1 Aguardando Cliente

1. Matriz Funcional completa (planos por persona)
2. Decisão sobre Assembleia (ao vivo ou assíncrona)
3. Documentos: Admin, Vizinhança, LMS
4. Fluxos de onboarding por persona

### 8.2 Podemos Avançar Agora

1. ✅ Definir contratos de API entre domínios (seção 3)
2. ✅ Estrutura de pastas do backend (Modular Monolith)
3. ✅ Setup inicial: Database schema (Identity + Institutional básico)
4. ✅ Implementar Adapter Layer para External Services
5. ✅ Event Bus in-memory (Fase 1)

### 8.3 Após Receber Documentos

1. Refinar HIGH-DOMAINS (se Admin/Vizinhança/LMS forem novos high-domains)
2. Atualizar contratos de API com regras de planos
3. Implementar middleware de autorização (ABAC)
4. Definir cronograma de entregas por domínio

---

## 9. GLOSSÁRIO

| Termo | Definição |
|-------|-----------|
| **HIGH-DOMAIN** | Domínio autônomo de nível 1, pode ser deployado independentemente |
| **MID-DOMAIN** | Bounded context dentro de um HIGH-DOMAIN |
| **LOW-DOMAIN** | Sub-módulo dentro de um MID-DOMAIN |
| **Tenant** | Unidade de isolamento de dados (condomínio ou empresa) |
| **Actor** | Entidade que opera dentro ou entre tenants (usuário, agência, administradora) |
| **External Service** | Serviço de terceiro que consumimos via adapter |
| **Internal Domain** | Domínio que implementamos e mantemos |
| **Event Bus** | Barramento de eventos para comunicação entre domínios |
| **Adapter Layer** | Camada anti-corrupção que abstrai external services |

---

**FIM DO DOCUMENTO**

> Próxima atualização: após receber documentos pendentes do cliente
