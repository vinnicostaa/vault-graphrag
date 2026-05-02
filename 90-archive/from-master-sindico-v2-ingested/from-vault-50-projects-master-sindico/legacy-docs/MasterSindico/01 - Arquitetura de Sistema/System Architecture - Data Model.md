---
title: System Architecture — Data Model
parent: System Architecture
domain: ALL
status: Draft
tags:
  - mastersindico
  - architecture
  - database
  - drizzle
  - schema
---

# Data Model — Schemas Completos

> **ORM:** Drizzle (PostgreSQL beta)
> **Convenções:** UUIDv7 PKs, snake_case, created_at/updated_at/deleted_at (soft delete)
> **Tabelas existentes:** ~25 (Sprint 1)
> **Tabelas novas:** ~45
> **Total estimado:** ~70

> Veja o canvas visual: [[Data Model.canvas]]

---

## 1. Tabelas Existentes (Sprint 1) ✅

### Identity

```
users: id, email, password_hash, name, phone, user_type, status, 
       email_verified_at, created_at, updated_at, deleted_at

accounts: id, user_id, provider, provider_id, 
          access_token, refresh_token, created_at

sessions: id, user_id, token, ip_address, user_agent, 
          device_fingerprint, expires_at, created_at

verifications: id, user_id, type, token, expires_at, 
               verified_at, created_at
```

### Profiles

```
syndic_profiles: id, user_id, sindico_type, governance_markers, 
                 mandate_start, city, state, ...

enterprise_profiles: id, user_id, cnpj, fantasy_name, description, 
                     categories, compliance_markers, ...

resident_profiles: id, user_id, condominium_id, unit_id, ...

marketing_profiles: id, user_id, agency_name, portfolio, ...

local_company_profiles: id, user_id, partner_name, category, 
                        neighborhood, city, ...
```

### Billing

```
plans: id, name, slug, type, price_monthly, price_annual, 
       features, quotas, trial_days, ...

subscriptions: id, user_id, plan_id, stripe_subscription_id,
               status (active/incomplete/past_due/canceled/...),
               current_period_start, current_period_end, ...

transactions: id, subscription_id, stripe_payment_intent_id,
              amount, status, payment_method, ...

invoices: id, subscription_id, stripe_invoice_id, ...

feature_usage: id, user_id, feature_name, used, limit, 
               reset_date, ...
```

### Content (parcial)

```
videos: id, owner_id, tenant_type, title, description, 
        video_type, mux_asset_id, mux_playback_id, 
        status, duration, thumbnail_url, ...

categories: id, name, slug, parent_id, ...
```

### Connect Me (parcial)

```
connect_me: id, user_id, to_user_id, description, urgency, 
            status, ...
```

### Buildings (parcial)

```
buildings: id, name, address, city, state, cep, 
           total_units, ...
```

---

## 2. Tabelas Novas — Institutional

### Condominium & Mandate

```
condominiums: (expandir buildings)
  id, name, cnpj, address, city, state, cep
  total_units, total_floors, construction_year
  condominium_type (residencial/comercial/misto)
  created_by (sindico)
  created_at, updated_at, deleted_at

mandates:
  id, condominium_id, sindico_id
  start_date, end_date
  status (ativo/encerrado/transferido)
  empresa_sindica_id (nullable)
  created_at, updated_at

units:
  id, condominium_id
  number, block, floor
  fracao_ideal (numeric)
  titular_id (FK users)
  created_at, updated_at, deleted_at
```

### Timeline

```
timeline_entries:
  id                   UUID PK (UUIDv7)
  condominium_id       UUID FK
  author_id            UUID FK
  entry_type           ENUM(7 types — ver abaixo)
  title                TEXT NOT NULL
  description          TEXT NOT NULL
  date                 TIMESTAMP NOT NULL
  published_at         TIMESTAMP NOT NULL
  is_amended           BOOLEAN DEFAULT false
  original_entry_id    UUID FK (nullable, para adendo)
  master_plan_action_id UUID FK (nullable)
  source_entity_type   TEXT (nullable: 'closed_deal', 'execution_record')
  source_entity_id     UUID (nullable)
  attachments          JSONB
  created_at, updated_at
  -- SEM deleted_at: imutável, nunca soft-deleted

entry_type ENUM:
  'atividade_da_gestao'
  'registro_de_execucao'
  'comunicado'
  'evento'
  'adendo'
  'resultado_consulta_moradores'
  'assembly_result'
```

### Master Plan

```
master_plan_actions:
  id                  UUID PK
  condominium_id      UUID FK
  created_by          UUID FK
  title               TEXT NOT NULL
  description         TEXT
  tipo_atividade      ENUM(26 types)
  area_impactada      ENUM(26 areas)
  tipo_risco          ENUM(9 risks)
  status              ENUM(planejado/em_andamento/concluido/atrasado/
                           cancelado/suspenso/nao_iniciado)
  priority            ENUM(baixa/media/alta/critica)
  start_date          DATE
  due_date            DATE
  completed_at        TIMESTAMP (nullable)
  budget_estimated    INTEGER (cents)
  budget_actual       INTEGER (cents, nullable)
  linked_timeline_entries JSONB (array de ids)
  created_at, updated_at, deleted_at
```

### Compliance

```
annual_declarations:
  id, condominium_id, sindico_id
  year, declaration_type, content
  signed_at, ip_address
  created_at

conflict_of_interest:
  id, condominium_id, declared_by
  conflict_type, description, parties_involved
  declared_at, created_at

governance_scores:
  id, condominium_id
  score (0-100), breakdown (JSONB)
  calculated_at, created_at

audit_trail: (ver Infrastructure)
```

### Validations

```
validation_items:
  id                  UUID PK
  condominium_id      UUID FK
  source_type         TEXT (execution_record/technical_activity/comunicado/proposal)
  source_id           UUID
  company_id          UUID FK
  status              ENUM(pending/approved/rejected/adjustment_requested)
  reviewed_by         UUID FK (nullable)
  reviewed_at         TIMESTAMP (nullable)
  rejection_reason    TEXT (nullable)
  created_at, updated_at
```

---

## 3. Tabelas Novas — Commercial

### Proposals (Req 20)

```
proposals:
  id                      UUID PK
  company_id              UUID FK
  condominium_id          UUID FK
  connect_me_request_id   UUID FK (nullable)
  title                   TEXT NOT NULL
  description             TEXT NOT NULL
  scope                   TEXT NOT NULL
  estimated_cost          INTEGER (cents)
  estimated_duration      TEXT
  status                  ENUM(draft/sent/awaiting_validation/validated/rejected/accepted)
  rejection_reason        TEXT (nullable)
  validated_by            UUID FK (nullable)
  validated_at            TIMESTAMP (nullable)
  version                 INTEGER DEFAULT 1
  attachments             JSONB
  created_at, updated_at, deleted_at
```

### Supplier Voting (Req 21)

```
supplier_votings:
  id                  UUID PK
  condominium_id      UUID FK
  created_by          UUID FK (síndico)
  title               TEXT NOT NULL
  description         TEXT
  voting_type         ENUM(simple_majority/qualified_majority/unanimous)
  start_date          TIMESTAMP NOT NULL
  end_date            TIMESTAMP NOT NULL
  required_quorum     INTEGER NOT NULL
  total_units         INTEGER NOT NULL
  status              ENUM(draft/open/closed/cancelled)
  winner_proposal_id  UUID FK (nullable)
  closed_early        BOOLEAN DEFAULT false
  created_at, updated_at, deleted_at

supplier_voting_proposals:
  id, voting_id, proposal_id, display_order

supplier_votes:
  id, voting_id, proposal_id, morador_id, unit_id
  voted_at TIMESTAMP NOT NULL
  UNIQUE(voting_id, morador_id)
```

### Closed Deals (Req 22)

```
closed_deals:
  id                      UUID PK
  condominium_id          UUID FK
  company_id              UUID FK
  proposal_id             UUID FK (nullable)
  voting_id               UUID FK (nullable)
  service_description     TEXT NOT NULL
  agreed_cost             INTEGER (cents)
  start_date, end_date    TIMESTAMP
  payment_terms           TEXT
  status                  ENUM(draft/awaiting_company_signature/
                               awaiting_sindico_signature/confirmed/cancelled)
  company_signed_at       TIMESTAMP (nullable)
  company_signed_by       UUID FK (nullable)
  sindico_signed_at       TIMESTAMP (nullable)
  sindico_signed_by       UUID FK (nullable)
  confirmed_at            TIMESTAMP (nullable)
  is_immutable            BOOLEAN DEFAULT false
  attachments             JSONB
  timeline_entry_id       UUID FK (nullable)
  created_at, updated_at, deleted_at
```

### Service Execution (Req 23)

```
execution_records:
  id, company_id, condominium_id, closed_deal_id
  title, description, execution_date
  status (draft/submitted/validated/rejected)
  attachments JSONB
  created_at, updated_at, deleted_at

technical_activities:
  id, company_id, condominium_id, closed_deal_id
  activity_type, description, date
  status, attachments
  created_at, updated_at, deleted_at
```

### Connect Me Expandido (4 fluxos)

```
connect_me (ALTER TABLE — adicionar colunas):
  + flow_type           ENUM(sindico_empresa/morador_sindico/empresa_empresa/sindico_parceiro)
  + subject             TEXT (nullable — Morador→Síndico)
  + category            ENUM(5 cats) (nullable — Morador→Síndico)
  + response_message    TEXT (nullable)
  + responded_at        TIMESTAMP (nullable)
  + partnership_type    ENUM(5 types) (nullable — Empresa→Empresa)
  + promotion_type      ENUM(5 types) (nullable — Síndico→Parceiro)
  + target_audience_size INTEGER (nullable)
  + desired_discount    TEXT (nullable)
  + budget_range        TEXT (nullable)
  + auto_closed_at      TIMESTAMP (nullable)
  + interest_shown_at   TIMESTAMP (nullable)
  + data_revealed_at    TIMESTAMP (nullable — LGPD)
  + data_revealed_ip    TEXT (nullable — LGPD)
  + consent_version     TEXT (nullable)
  + status expandido    ENUM(pending/sent/open/interested/proposal_sent/
                             em_negociacao/em_analise/respondido/resolvido/
                             closed/expired/failed)
```

### Vizinhança (Req 27)

```
partners:
  id, user_id, name, category
  neighborhood, city, state, cep
  description, logo_url
  plan_type, status
  created_at, updated_at, deleted_at

promotions:
  id, partner_id
  title, description
  discount_type, discount_value
  keyword (para busca)
  start_date, end_date
  is_exclusive (boolean)
  target_condominium_id (nullable)
  status (active/expired/cancelled)
  views_count, activations_count
  created_at, updated_at, deleted_at
```

---

## 4. Tabelas Novas — Content

### Academia / LMS (Req 98.1)

```
courses:
  id, created_by, title, description
  category, level, type (free/paid)
  target_audience, status
  created_at, updated_at, deleted_at

lessons:
  id, course_id, title, description
  video_id (FK), display_order
  duration_minutes, created_at

certificates:
  id, user_id, course_id
  issued_at, certificate_url
  created_at
```

### Talent Bank / Banco de Talentos (Decision 6)

```
curricula:
  id, morador_id (UNIQUE)
  status (draft/active/suspended/deleted)
  full_name, phone, email, cep, age, marital_status
  has_children, cnh_type
  nr_courses TEXT[] (NR-10, NR-33, NR-35, etc.)
  contract_types TEXT[]
  available_schedules TEXT[]
  start_availability ENUM
  commute_time ENUM
  min_salary INTEGER (cents)
  ideal_salary INTEGER (cents)
  self_assessment TEXT
  lgpd_consented_at, lgpd_consent_ip, lgpd_consent_version
  video_updated_at (nullable)
  created_at, updated_at, deleted_at

curriculum_videos:
  id, curriculum_id, video_id
  duration_seconds (max 90)
  locked_until (90 dias)
  created_at

curriculum_professional_profiles:
  id, curriculum_id
  question_number (1-6), question_text, answer
  created_at, updated_at

curriculum_areas_of_interest:
  id, curriculum_id
  area ENUM(18 áreas), desired_function TEXT
  created_at

curriculum_work_experience:
  id, curriculum_id
  company_name, position, activities, tenure
  exit_reason ENUM
  display_order (1-5)
  created_at

curriculum_education:
  id, curriculum_id
  course_name, institution, professional_relevance
  created_at

company_curriculum_favorites:
  id, company_id, curriculum_id
  created_at
  UNIQUE(company_id, curriculum_id)

company_curriculum_tags:
  id, company_id, curriculum_id, tag TEXT
  created_at

curriculum_search_logs:
  id, company_id
  filters_used JSONB, results_count INTEGER
  profiles_viewed INTEGER DEFAULT 0
  searched_at TIMESTAMP
```

---

## 5. Tabelas Novas — Assembly

```
assemblies:
  id, condominium_id, created_by
  title, assembly_type, scheduled_date, scheduled_time, location
  quorum_type, speech_time_minutes, speech_extension_allowed
  modalidade, status
  administradora_id (nullable)
  started_at, ended_at, total_duration_minutes
  contingency_active, transparency_score
  created_at, updated_at, deleted_at

assembly_agenda_items:
  id, assembly_id
  title, description, item_type
  voting_required, quorum_required
  display_order, item_status
  opened_at, closed_at, attachments, is_locked
  created_at, updated_at

assembly_checkins:
  id, assembly_id, morador_id, unit_id
  check_in_method, attendance_status, checked_in_at
  UNIQUE(assembly_id, morador_id)

assembly_proxies:
  id, assembly_id
  delegator_id, delegatee_id
  accepted_at (nullable), status
  created_at

assembly_votes:
  id, assembly_id, agenda_item_id, morador_id, unit_id
  vote_value, is_proxy, proxy_id
  is_presencial_assistido, operator_id
  fracao_ideal_weight (numeric)
  voted_at, is_homologated
  UNIQUE(assembly_id, agenda_item_id, morador_id)

assembly_speeches:
  id, assembly_id, agenda_item_id, morador_id
  started_at, ended_at
  duration_seconds, was_extended
  created_at

assembly_reports:
  id, assembly_id, report_type
  content JSONB, file_url TEXT
  generated_at, created_at
```

---

## 6. Tabela Outbox (Infrastructure)

```
outbox_events:
  id              UUID PK (UUIDv7)
  aggregate_type  TEXT
  aggregate_id    UUID
  event_type      TEXT
  payload         JSONB
  metadata        JSONB (user_id, tenant_id, correlation_id, ip)
  status          ENUM(pending/published/failed)
  published_at    TIMESTAMP (nullable)
  retry_count     INTEGER DEFAULT 0
  created_at      TIMESTAMP
```

---

## 7. Enums Novos (para enums.ts)

```typescript
// Timeline
export const timelineEntryType = pgEnum('timeline_entry_type', [
  'atividade_da_gestao', 'registro_de_execucao', 'comunicado',
  'evento', 'adendo', 'resultado_consulta_moradores', 'assembly_result'
]);

// Connect Me
export const connectMeFlowType = pgEnum('connect_me_flow_type', [
  'sindico_empresa', 'morador_sindico', 'empresa_empresa', 'sindico_parceiro'
]);

// Proposals
export const proposalStatus = pgEnum('proposal_status', [
  'draft', 'sent', 'awaiting_validation', 'validated', 'rejected', 'accepted'
]);

// Voting
export const votingType = pgEnum('voting_type', [
  'simple_majority', 'qualified_majority', 'unanimous'
]);

// Deals
export const dealStatus = pgEnum('deal_status', [
  'draft', 'awaiting_company_signature', 'awaiting_sindico_signature', 'confirmed', 'cancelled'
]);

// Assembly
export const assemblyType = pgEnum('assembly_type', [
  'ordinaria', 'extraordinaria', 'implantacao'
]);
export const assemblyStatus = pgEnum('assembly_status', [
  'em_preparacao', 'publicada', 'em_andamento', 'encerrada', 'cancelada'
]);
export const assemblyModalidade = pgEnum('assembly_modalidade', [
  'presencial', 'hibrida', 'online'
]);
export const agendaItemType = pgEnum('agenda_item_type', [
  'informativo', 'votacao_simples', 'votacao_fracao_ideal', 'escolha_fornecedor', 'eleicao'
]);

// Master Plan (26 tipos)
export const masterPlanTipoAtividade = pgEnum('master_plan_tipo_atividade', [
  'manutencao_estrutural', 'manutencao_eletrica', 'manutencao_hidraulica',
  'pintura', 'jardinagem', 'limpeza', 'seguranca', 'elevadores',
  'portaria', 'piscina', 'playground', 'salao_festas', 'garagem',
  'iluminacao', 'impermeabilizacao', 'dedetizacao', 'recarga_extintores',
  'avcb', 'para_raios', 'cisterna', 'geradores', 'interfones',
  'cameras', 'controle_acesso', 'portoes', 'automacao'
]);

// Outbox
export const outboxStatus = pgEnum('outbox_status', [
  'pending', 'published', 'failed'
]);
```

---

## Navegação

← [[System Architecture - Infrastructure]] | [[System Architecture]] | [[System Architecture - API Design]] →
