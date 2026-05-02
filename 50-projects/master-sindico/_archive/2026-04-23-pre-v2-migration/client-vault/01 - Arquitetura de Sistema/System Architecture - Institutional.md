---
title: System Architecture — Institutional Domain
parent: System Architecture
domain: INSTITUTIONAL
status: Not Started
tags:
  - mastersindico
  - architecture
  - institutional
  - governance
  - ddd
---

# HIGH-DOMAIN: Institutional

> **Status:** Não iniciado — Sprint 2+
> **Tenant Isolation:** `condominium_id`
> **Módulos:** 6 novos
> **Source of Truth:** Requirements 7-15, 25, 28, 33-47

---

## Visão Geral

O domínio Institutional é o coração da plataforma — a governança condominial. Controla a Timeline imutável, o Plano Diretor reativo, validações centralizadas, compliance, e avaliações. É o backbone que conecta as ações comerciais ao registro institucional.

```
INSTITUTIONAL (Tenant: condominium_id)
├── CondominiumModule        — Registro, mandato, unidades
├── GovernanceModule         — Timeline + MasterPlan + Events + Comunicados + Polls
├── ValidationHubModule      — Hub centralizado de validações
├── ComplianceModule         — Declarações, auditoria, LGPD, risk map
├── EvaluationModule         — Avaliação gestão + empresa
└── MandateModule            — Transferência e encerramento de mandato
```

---

## CondominiumModule (NOVO)
**Req:** 7, 8, 9, 10

| Aspecto | Detalhe |
|---------|---------|
| **Entities** | Condominium, Mandate, EmpresaSindica, Unit |
| **Use Cases** | CreateCondominium, EndMandate, TransferMandate, LinkEmpresaSindica, RegisterUnit, ManageMoradorAccess |
| **Limites** | Máximo 15 condomínios por síndico (plan-based) |
| **Unidades** | 1 titular + até 2 dependentes por unidade |

**Events Published:**
- `institutional.condominium.created`
- `institutional.condominium.mandate-ended`
- `institutional.condominium.mandate-transferred`
- `institutional.condominium.unit-registered`

**Events Consumed:**
- `identity.auth.user-registered` (síndico)
- `identity.user.onboarding-completed`

**Dependências:** AuthModule, BillingModule

---

## GovernanceModule (NOVO) — O Módulo Mais Complexo
**Req:** 11, 12, 12.2, 13, 14, 14.1, 25

### Sub-contextos

#### Timeline (Memória Institucional)
- **Append-only**: Entradas nunca são editadas ou deletadas
- **7 entry_types**: `atividade_da_gestao`, `registro_de_execucao`, `comunicado`, `evento`, `adendo`, `resultado_consulta_moradores`, `assembly_result`
- **Adendo**: Única forma de "corrigir" — cria nova entrada vinculada
- **Auto-publicação**: Deals confirmados e execuções validadas publicam automaticamente via NATS consumer

#### Master Plan (Plano Diretor Reativo)
- **26 tipos_atividade**: Manutenção estrutural, elétrica, hidráulica, pintura, jardinagem, limpeza, segurança, elevadores, portaria, piscina, playground, salão de festas, garagem, iluminação, impermeabilização, dedetização, recarga extintores, AVCB, para-raios, cisterna, geradores, interfones, câmeras, controle acesso, portões, automação
- **26 áreas_impactadas**
- **9 tipos_risco**
- **7 statuses**: `planejado` → `em_andamento` → `concluido` | `atrasado` | `cancelado` | `suspenso` | `nao_iniciado`
- **Status automático**: Recalculado via eventos da Timeline (implicit event sourcing)

#### Events (Eventos do Condomínio)
- **13 event_types**: assembleia, reunião, festa, manutenção, inspeção, etc.
- Participation tracking, calendar view

#### Comunicados
- **8 comunicado_types**: urgente, informativo, manutenção, financeiro, etc.
- Priority levels, read tracking por morador

#### Consultas aos Moradores
- **3 question types**: sim/não, múltipla escolha, aberta
- Resultados anônimos, auto-publicação na Timeline

**Events Published:**
- `institutional.governance.timeline-published`
- `institutional.governance.masterplan-created`
- `institutional.governance.masterplan-status-changed`
- `institutional.governance.event-created`
- `institutional.governance.comunicado-created`
- `institutional.governance.question-created`
- `institutional.governance.question-closed`
- `institutional.governance.adendo-created`

**Events Consumed:**
- `commercial.deal.confirmed` → auto-publica na Timeline
- `institutional.validation.resolved` (aprovado) → auto-publica na Timeline
- `institutional.governance.timeline-published` → recalcula status Master Plan

**Dependências:** AuthModule, CondominiumModule, BillingModule

---

## ValidationHubModule (NOVO)
**Req:** 28

Hub centralizado onde o síndico vê e resolve todas as validações pendentes de diferentes fontes.

| Aspecto | Detalhe |
|---------|---------|
| **Entities** | ValidationItem |
| **Use Cases** | ListPendingValidations, ValidateItem, RejectItem, BulkValidate |
| **Fontes** | Registros de execução, atividades técnicas, comunicados de empresa, propostas |

**Events Published:**
- `institutional.validation.validated`
- `institutional.validation.rejected`

**Events Consumed:**
- `commercial.execution.submitted`
- `commercial.proposal.sent`
- `commercial.technical-activity.sent`
- `commercial.comunicado.sent`

---

## ComplianceModule (NOVO)
**Req:** 33-42

Compliance robusto para blindagem institucional do síndico.

| Aspecto | Detalhe |
|---------|---------|
| **Entities** | AnnualDeclaration, MandatorySignature, ConflictOfInterest, AuditTrailEntry, RiskMap, GovernanceScore, ContractCompliance, LGPDRegistry |
| **Funcionalidades** | Declaração anual, assinaturas obrigatórias, matriz de conflitos, trilha de auditoria, mapa de riscos, governance score, compliance contratual, LGPD registry, dossier export |

**Governance Score:** Calculado automaticamente baseado em:
- Timeline activity frequency
- Master Plan execution rate
- Compliance completeness
- Assembly transparency score
- Morador evaluation scores

**Events Published:**
- `institutional.compliance.declaration-submitted`
- `institutional.compliance.signature-completed`
- `institutional.compliance.conflict-declared`
- `institutional.compliance.governance-score-updated`
- `institutional.compliance.issue-detected`

**Events Consumed:**
- `institutional.governance.timeline-published`
- `commercial.deal.confirmed`
- `institutional.condominium.mandate-ended` (bloqueia se não compliant)
- Todos eventos LGPD-relevant

---

## EvaluationModule (NOVO)
**Req:** 15, 26

| Aspecto | Detalhe |
|---------|---------|
| **Entities** | ManagementEvaluation, CompanyEvaluation |
| **Avaliação de Gestão** | Bimestral, moradores avaliam síndico (5 critérios, escala 1-10) |
| **Avaliação de Empresa** | Após execução de serviço (5 critérios, escala 1-10) |
| **Trigger** | Cron bimestral (gestão) + DealCompleted event (empresa) |

**Events Published:**
- `institutional.evaluation.management-submitted`
- `institutional.evaluation.company-submitted`

---

## Fluxo de Dados do Domínio

```
Empresa submete execução
    │
    ▼ NATS: commercial.execution.submitted
ValidationHubModule
    │ Síndico aprova
    ▼ NATS: institutional.validation.validated
GovernanceModule (Timeline)
    │ Auto-publica entry
    ▼ NATS: institutional.governance.timeline-published
GovernanceModule (Master Plan)
    │ Recalcula status da ação vinculada
    ▼ NATS: institutional.governance.masterplan-status-changed
ComplianceModule
    │ Atualiza governance score
    └─ Grava audit trail
```

---

## Navegação

← [[System Architecture - Identity]] | [[System Architecture]] | [[System Architecture - Commercial]] →
