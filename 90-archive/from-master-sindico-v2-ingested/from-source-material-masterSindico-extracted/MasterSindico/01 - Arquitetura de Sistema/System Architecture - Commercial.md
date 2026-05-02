---
title: System Architecture — Commercial Domain
parent: System Architecture
domain: COMMERCIAL
status: Not Started
tags:
  - mastersindico
  - architecture
  - commercial
  - procurement
  - ddd
---

# HIGH-DOMAIN: Commercial

> **Status:** ConnectMe parcialmente existente, pipeline novo — Sprint 3+
> **Tenant Isolation:** `company_id` (principal), cross-tenant com `condominium_id`
> **Módulos:** 7 (1 existente + 6 novos)
> **Source of Truth:** Requirements 19-24, 26-27.3, 30, 101

---

## Visão Geral

O domínio Commercial cobre todo o pipeline de contratação: desde a busca e primeiro contato (Connect Me) até a execução e avaliação do serviço. Inclui também o marketplace de vizinhança e operações de agência de marketing.

> [!WARNING]
> **Divergência corrigida:** O blueprint v2.0 pulou completamente o pipeline Proposta → Votação → Negócio Fechado (Req 20-22). Esses 3 módulos são essenciais e foram adicionados.

```
COMMERCIAL (Tenant: company_id, cross-tenant)
├── ConnectMeModule          — 4 fluxos de marketplace ⚠️ (expandir)
├── ProposalModule           — Pipeline de propostas (NOVO)
├── SupplierVotingModule     — Votação de fornecedor (NOVO)
├── ClosedDealModule         — Negócio fechado com dupla assinatura (NOVO)
├── ServiceExecutionModule   — Registro de execução + atividades técnicas (NOVO)
├── VizinhancaModule         — Comércio local + promoções (NOVO)
└── MarketingAgencyModule    — Operações de agência (NOVO)
```

---

## Pipeline de Contratação Completo

```
Search (Meilisearch)
    ↓
Connect Me (1 dos 4 fluxos)        Req 19-19.3
    ↓ "Tenho Interesse" + LGPD log
Proposta                             Req 20
    ↓ Síndico valida
Votação Fornecedor (se 2+ propostas) Req 21
    ↓ Quorum + votos moradores
Negócio Fechado                      Req 22
    ↓ Dupla assinatura (empresa + síndico)
    ↓ Auto-publica na Timeline
Execução de Serviço                  Req 23
    ↓ Empresa registra → Validações Pendentes
    ↓ Síndico aprova → Timeline + Master Plan
Avaliação da Empresa                 Req 26
    ↓ 5 critérios, escala 1-10
```

---

## ConnectMeModule (EXISTENTE — expandir)
**Req:** 19, 19.1, 19.2, 19.3

> [!WARNING]
> **Divergência corrigida:** O blueprint modelou apenas 1 fluxo. São **4 fluxos** com schemas distintos.

### 4 Fluxos

| Fluxo | Req | Quem → Quem | Campos Específicos | Plano Mínimo |
|-------|-----|-------------|-------------------|--------------|
| **Síndico → Empresa** | 19 | Síndico busca empresa por categoria | service_category, description, urgency, budget_range | Base |
| **Morador → Síndico** | 19.1 | Morador contata síndico | subject, category (5 cats), urgency | Base (quota: 2/ano), Paid (4/ano) |
| **Empresa → Empresa** | 19.2 | Parceria entre empresas | partnership_type (5 types) | Plus/Pro |
| **Síndico → Parceiro** | 19.3 | Síndico negocia com comércio | promotion_type (5 types), target_audience_size | Base |

### Schema (tabela expandida)

```
connect_me_requests:
  flow_type: enum('sindico_empresa', 'morador_sindico', 'empresa_empresa', 'sindico_parceiro')
  status: enum('pending', 'sent', 'open', 'interested', 'proposal_sent', 
               'em_negociacao', 'em_analise', 'respondido', 'resolvido', 
               'closed', 'expired', 'failed')
  
  -- Flow-specific (nullable):
  category, partnership_type, promotion_type, target_audience_size, desired_discount
  
  -- LGPD:
  data_revealed_at, data_revealed_ip, consent_version
  
  -- Auto-close: 48h sem resposta
  auto_closed_at
```

**Events:** `commercial.connect-me.request-created`, `commercial.connect-me.interest-shown`, `commercial.connect-me.auto-expired` (por variante de fluxo)

---

## ProposalModule (NOVO)
**Req:** 20

| Aspecto | Detalhe |
|---------|---------|
| **Entities** | Proposal |
| **Status Flow** | `draft` → `sent` → `awaiting_validation` → `validated` → `accepted` / `rejected` |
| **Versionamento** | Propostas podem ser atualizadas (version++) antes de validação |
| **Attachments** | JSONB array de {url, type, name} via R2 |

**Events:**
- `commercial.proposal.sent`
- `commercial.proposal.validated`
- `commercial.proposal.rejected`

**Consumed:** `commercial.connect-me.interest-shown` → link proposta ao request

---

## SupplierVotingModule (NOVO)
**Req:** 21

| Aspecto | Detalhe |
|---------|---------|
| **Entities** | SupplierVoting, SupplierVotingProposal, SupplierVote |
| **Pré-requisito** | 2+ propostas validadas |
| **Quorum** | Configurável: simple_majority, qualified_majority, unanimous |
| **Real-time** | Progresso de votos visível para moradores |
| **Early Close** | Síndico pode encerrar antecipadamente se quorum atingido |

**Events:**
- `commercial.voting.created`
- `commercial.voting.started`
- `commercial.voting.vote-cast`
- `commercial.voting.ended`

---

## ClosedDealModule (NOVO)
**Req:** 22

| Aspecto | Detalhe |
|---------|---------|
| **Entities** | ClosedDeal, DealSignature |
| **Dupla Assinatura** | Empresa assina → Síndico assina → Deal confirmado |
| **Imutabilidade** | Após confirmação, `is_immutable = true` — zero edições |
| **Auto-publicação** | Deal confirmado → NATS → Timeline (automático) |
| **Status Flow** | `draft` → `awaiting_company_signature` → `awaiting_sindico_signature` → `confirmed` / `cancelled` |

**Events:**
- `commercial.deal.created`
- `commercial.deal.confirmed` ← trigger para Timeline
- `commercial.deal.cancelled`

---

## ServiceExecutionModule (NOVO)
**Req:** 23, 23.1, 24

| Aspecto | Detalhe |
|---------|---------|
| **Entities** | ExecutionRecord, TechnicalActivity, PostDealComunicado, SubmissionTracker |
| **Fluxo** | Empresa cria registro → submete → vai para Validações Pendentes do síndico |
| **Tipos** | Execução de serviço, atividade técnica, comunicado pós-negócio |
| **Tracking** | SubmissionTracker acompanha status de cada submissão |

**Events:**
- `commercial.execution.submitted`
- `commercial.technical-activity.submitted`
- `commercial.comunicado.submitted`
- `commercial.execution.status-changed`

**Consumed:** `institutional.validation.validated` → publica na Timeline

---

## VizinhancaModule (NOVO)
**Req:** 27, 27.1, 27.2, 27.3

Marketplace de comércio local conectando parceiros aos moradores do condomínio.

| Aspecto | Detalhe |
|---------|---------|
| **Entities** | Partner, Promotion, PromotionActivation, ExclusiveBenefit, PartnerInvitation |
| **Personas** | Parceiro, Síndico, Morador, Empresa |
| **Trial** | 30 dias para parceiro |
| **Promoções** | Criação, ativação, métricas, exclusivas por condomínio |
| **Convites** | Síndico pode convidar parceiros da região |

**Events:**
- `commercial.vizinhanca.partner-registered`
- `commercial.vizinhanca.promotion-created`
- `commercial.vizinhanca.promotion-activated`

---

## MarketingAgencyModule (NOVO)
**Req:** 30, 101

| Aspecto | Detalhe |
|---------|---------|
| **Entities** | AgencyLink, MarketingLinkRequest |
| **Papel** | Agência é ator delegado, não é tenant próprio |
| **Funcionalidades** | Upload de vídeos para empresa, gestão de acesso delegado, marketing links |
| **Vínculo** | Empresa convida agência → AgencyLink criado |

**Events:**
- `commercial.agency.video-uploaded`
- `commercial.agency.linked`

---

## Cross-Tenant Flow

```
TENANT: EMPRESA (company_id=C1)
│ POST /v1/companies/C1/executions
│ ├─ Cria execution_record (status=draft)
│ ├─ Submete → status=submitted
│ └─ Event: commercial.execution.submitted
│
│  ─── NATS JetStream ───
│
TENANT: CONDOMÍNIO (condominium_id=COND1)
│ Consumer: validation-projector
│ ├─ Cria pending_validation
│ └─ Event: institutional.validation.created
│
│ Síndico aprova:
│ ├─ Event: institutional.validation.validated
│ └─ Consumer: timeline-projector
│     ├─ Insere na Timeline
│     └─ Consumer: master-plan-updater
│         └─ Recalcula status
```

---

## Navegação

← [[System Architecture - Institutional]] | [[System Architecture]] | [[System Architecture - Content]] →
