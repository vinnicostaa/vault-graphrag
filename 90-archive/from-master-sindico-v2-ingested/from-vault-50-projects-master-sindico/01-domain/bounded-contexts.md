---
title: Bounded Contexts — Master Síndico
type: project
tags: [master-sindico, ddd, bounded-contexts]
project: master-sindico
created: 2026-04-22
---

# Bounded Contexts — Master Síndico

6 bounded contexts principais + 1 cross-domain. Cada um é coeso (coisa faz uma coisa só) e separado (não compartilha tabela/code interno com outro).

---

## Visão geral

```
┌─────────────┐    ┌─────────────┐    ┌──────────────┐
│  identity   │───▶│   billing   │───▶│institutional │
│             │    │             │    │              │
└─────────────┘    └─────────────┘    └──────────────┘
       │                 │                   │
       ▼                 ▼                   ▼
┌─────────────┐    ┌─────────────┐    ┌──────────────┐
│  commercial │    │   content   │    │   assembly   │
└─────────────┘    └─────────────┘    └──────────────┘
       ▲                 ▲                   ▲
       └─────────────────┴───────────────────┘
                    [ cross-domain ]
```

Setas: relacionamento de dependência conceitual (não import direto — comunicação via `contracts` e domain events).

---

## 1. `identity`

**Responsabilidade**: usuários, sessões, autenticação (via Zitadel), 1-device policy, sync user local ↔ Zitadel.

**Agregados**:
- `User` — identidade única (CPF ou CNPJ); metadata de contato
- `Session` — sessão ativa com device fingerprint

**VOs**: `Email`, `CPF`, `CNPJ`, `Password`, `Phone`, `DeviceFingerprint`

**Invariantes-chave**:
- 1 CPF = 1 User
- 1 Session ativa por User (configurável)

**Eventos publicados**: `UserCreated`, `UserUpdated`, `SessionCreated`, `SessionInvalidated`, `DeviceChanged` (Sprint 10)

**Eventos consumidos**: nenhum

**Interações externas**: Zitadel OIDC (IAuthProvider); não tem SDK próprio

**Status**: ✅ Produção

---

## 2. `billing`

**Responsabilidade**: planos, assinaturas, trial por persona, cobrança (Stripe), quotas/feature_usage, webhooks.

**Agregados**:
- `Subscription` — raiz; contém Seats
- `Plan` — definição de plano (ID, tier, features, preço)
- `FeatureUsage` — consumo vs quota (Connect Me/mês, uploads/mês)
- `StripeWebhookEvent` — idempotency ledger

**VOs**: `Money` (int64 centavos), `PlanTier`, `TrialDuration` (persona-aware)

**Invariantes-chave**:
- Trial duration por persona (Síndico 15, Empresa 7, Parceiro 30)
- Quota nunca < 0; Unlimited nunca decrementa
- Webhook event.id único

**Eventos publicados**: `SubscriptionActivated`, `SubscriptionCanceled`, `TrialExpired`, `QuotaExhausted`, `PlanChanged`

**Eventos consumidos**: `UserCreated` (cria Subscription em trial)

**Interações externas**: Stripe API via `IPaymentGateway`

**Status**: ✅ Produção

---

## 3. `institutional`

**Responsabilidade**: condomínio, unidades, memberships, timeline imutável, plano diretor, compliance, anúncios, eventos.

**Agregados**:
- `Condominium` — raiz; contém Units, Memberships
- `Unit` — imóvel no condomínio; tem fração ideal
- `Membership` — vínculo user ↔ condomínio ↔ unidade com role (sindico/morador/subsindico/conselho)
- `TimelineEntry` — **imutável**; sem UPDATE, sem DELETE
- `MasterPlan` (Plano Diretor) — plano multi-ano
- `Announcement` — comunicado oficial (publicado → imutável)
- `Event` — evento do condomínio (reunião, obra, festa)
- `Poll` — enquete (diferente de votação formal de assembleia)
- `ValidationItem` + `ComplianceAssessment` — checklists legais

**VOs**: `FracaoIdeal` (percentual 0-100), `AddressBR`, `UnitNumber`

**Invariantes-chave**:
- Soma de FracaoIdeal de Units = 100% no condomínio
- TimelineEntry imutável (violação = bug)
- Membership ativa única por (user, condomínio) em um momento

**Eventos publicados**: `CondominiumCreated`, `UnitRegistered`, `MembershipAssigned`, `MembershipEnded`, `TimelineAppended`, `AnnouncementPublished`, `MandateTransferred`, `MandateEnded`, `MasterPlanActivityAdvanced`

**Eventos consumidos**: `SubscriptionActivated` (desbloqueia features)

**Interações externas**: nenhuma externa direta (usa Content para vídeos institucionais via eventos)

**Status**: ✅ Produção

---

## 4. `commercial`

**Responsabilidade**: Connect Me (4 fluxos), empresas, contratos, reputação, marketplace, empresa-sindica-link, harassment-reports, avaliações de empresa.

**Agregados**:
- `Company` — empresa cadastrada com CNPJ
- `CompanyUser` — vínculo user↔company (com role na empresa)
- `ConnectMeRequest` — "RFP" aberto pelo síndico
- `Proposal` — resposta de empresa ao ConnectMe
- `ClosedDeal` — contrato fechado
- `CompanyEvaluation` — avaliação pós-contrato (alimenta reputação)
- `SupplierVoteSession` + `SupplierVoteCast` — escolha colegiada de fornecedor (quando síndico delega ao conselho)
- `EmpresaSindicaLink` — convite síndico⇔empresa com TTL
- `HarassmentReport` — denúncia de abuso em Connect Me

**VOs**: `ReputacaoTier` (Bronze|Prata|Ouro|Platina|Diamante), `ExecutionRecord`

**Invariantes-chave**:
- Reputação é função determinística de métricas
- SupplierVote unique(session_id, voter_id) — duplicate vote via TOCTOU protegido (A-026 fix)
- CompanyEvaluation unique(company_id, evaluator_user_id, condominium_id) — duplicate avaliação protegida (A-029 fix)

**Eventos publicados**: `CompanyApproved`, `ConnectMeOpened`, `ProposalSubmitted`, `DealClosed`, `CompanyEvaluated`, `ReputationTierChanged`, `HarassmentReported`

**Eventos consumidos**: `CompanyCreated` (subscrição), `MembershipAssigned` (cache ABAC)

**Interações externas**: nenhuma direta; delega mídia para Content

**Status**: ✅ Produção

---

## 5. `content`

**Responsabilidade**: vídeos institucionais (Mux), moderação, trava 90d, busca (OpenSearch/tsvector), privacidade de métricas, LMS infra.

**Agregados**:
- `Video` — raiz; contém metadata Mux
- `VideoModeration` — estado de aprovação
- `SearchIndex` — materialized do OpenSearch

**VOs**: `VideoDuration`, `LockedUntilTimestamp`

**Invariantes-chave**:
- Vídeo `published` → bloqueado por 90d (política anti-churn)
- Métricas não vazam entre tenants

**Eventos publicados**: `VideoUploaded`, `VideoApproved`, `VideoPublished`, `VideoRemoved`, `VideoLocked`

**Eventos consumidos**: `CompanyApproved` (permite upload), `SubscriptionActivated` (upgrade features)

**Interações externas**:
- Mux via `IVideoProvider` (upload URL, webhooks, ready/errored)
- OpenSearch via `ISearchProvider`
- Saga com compensação: A-027 `UploadVideo` — se Create local falha, cancela upload no Mux

**Status**: ✅ Produção

---

## 6. `assembly`

**Responsabilidade**: assembleias, edital, convocação, agenda, votação com quórum fracional, live session (Livekit), fila de fala, atas imutáveis, attendance.

**Agregados**:
- `Assembly` — raiz; contém Agenda, Attendance, Votes, Minutes
- `AgendaItem` — item de pauta; pode ter fração ideal requerida
- `Vote` — voto individual; unique(agenda_item_id, voter_id)
- `Minutes` (Ata) — imutável após publicação
- `LiveSession` — sessão Livekit correspondente
- `AttendanceRecord` — quem compareceu
- `Proxy` — procuração

**VOs**: `QuorumThreshold`, `VoteChoice` (aprovar/rejeitar/abster)

**Invariantes-chave**:
- Vote unique(agenda_item_id, voter_id) — double-voting impossível (A-025 fix via UNIQUE + ErrConflict)
- Minutes imutável após publish
- Quórum calculado por fração ideal, não por pessoa
- Live room órfão evitado por Saga (A-028)

**Eventos publicados**: `AssemblyScheduled`, `EditalPublished`, `AssemblyStarted`, `VoteCast`, `AssemblyClosed`, `MinutesPublished`

**Eventos consumidos**: `CondominiumCreated` (permite abrir assembleia), `SubscriptionActivated` (upgrade live)

**Interações externas**:
- Livekit via `ILiveProvider`
- Saga compensation (A-028, A-033, A-034)

**Status**: ✅ Produção

---

## Cross-Domain

Operações e integrações que cruzam contextos.

**Artefatos**:
- **IAuditLogger** (DT-010, Sprint 10) — log cross-módulo de ações sensíveis
- **Domain event bus** — publica/consome entre módulos (hoje in-process; NATS quando precisar escalar)
- **ABAC policies** — catalog de (resource_type, action) → allowed_roles, plan_tier
- **Cache Redis** com ABAC decisão invalidated por webhook Stripe

**Princípio**: cross-domain NÃO pode importar `domain/` ou `application/` de dois módulos ao mesmo tempo. Se precisa, é evento de domínio + handler.

**Status**: 🟡 em evolução (IAuditLogger pendente)

---

## Mapa de Contextos (relacionamento)

| De | Para | Tipo | Como |
|---|---|---|---|
| identity | billing | Customer-Supplier | Evento `UserCreated` |
| billing | institutional | Customer-Supplier | Evento `SubscriptionActivated` desbloqueia features |
| institutional | commercial | Shared Kernel (Condominium ID) | Via ID apenas; Company referencia tenants |
| commercial | content | Customer-Supplier | CompanyApproved → upload permitido |
| institutional | assembly | Partnership | Assembly precisa Condominium + Units |
| commercial | assembly | Conformist | Assembly pode votar fornecedor → SupplierVote |
| all | cross-domain | Shared infrastructure | AuditLogger, EventBus, ABAC |

---

## Links

- [[_moc]]
- [[context-map]]
- [[ubiquitous-language]]
- [[business-rules]]
- [[domain-rules]]
- [[invariants]]
- [[domain-events]]
- [[../02-architecture/_moc]]
- [[../04-requirements/_moc]]
- [[../client-vault/02 - Domínios e Requisitos/Dominio Identidade]]
- [[../client-vault/02 - Domínios e Requisitos/Dominio Comercial]]
- [[../client-vault/02 - Domínios e Requisitos/Dominio Conteudo]]
- [[../client-vault/02 - Domínios e Requisitos/Arquitetura Assembleia]]
