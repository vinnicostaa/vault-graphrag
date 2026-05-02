---
title: Functional Requirements — Cross-Domain BC
type: requirement
tags:
  - requirements
  - functional
  - cross-domain
  - abac
  - audit
  - events
  - integration
  - master-sindico
  - v2
  - ears
source: >-
  legacy specs/requirements/cross-domain + legado domain/abac + DT-010 +
  _chaos/requirements(6).md (2026-04-13) + _chaos/mastersindico-requirements.pdf
  (2026-03-09)
created: 2026-04-23T00:00:00.000Z
updated: '2026-04-25'
fase_c_absorvido: true
---

# Functional Requirements — Cross-Domain BC

> **Banner Fase C (2026-04-25)**: enriquecido com R36/R40/R94 absorvidos do `_chaos/`. D-131 (Adjustment cross-BC), D-126 (2 tenant kinds), D-135 (device fingerprint progressivo) integrados.

Requisitos funcionais dos recursos **transversais**: infra de domain events (NATS), ABAC engine (matrix + cache + invalidation), audit trail append-only, LGPD logs, governance score, integrações (Stripe, Mux, Livekit, Resend/SES, Twilio, ViaCEP, FCM), antifraude, `IAuditLogger` (DT-010).

Formato **EARS**. Prioridade: M (M1), S (M2), C (M3+), W (backlog).

> Destilado de `specs/requirements/cross-domain.md` + [[STEERING]] §14, §16, §24 + [[00-product/portfolio-de-produtos]] + DT-010 legado.

---

## XD-001 — Domain events: NATS JetStream

**Prioridade**: M (básica) / S (completa)  
**EARS**: *Quando* BC produz evento de domínio, *o sistema deve* publicar via `IEventBus.Publish(subject, payload)`. Default: NATS JetStream (durável) pra eventos críticos; sync in-process pra intra-módulo via UoW.

**Critério de aceite**:
- Interface `IEventBus` em `internal/shared/events`.
- Subjects padrão: `{bc}.{aggregate}.{action}` (ex: `institutional.timeline.entry_created`, `billing.subscription.changed`, `commercial.deal.signed_complete`).
- Durability: at-least-once.

**Fonte**: legado infra, [[STEERING]] §14, §35.

---

## XD-002 — Outbox pattern pra eventos críticos

**Prioridade**: S  
**EARS**: *Quando* handler escreve em DB + precisa publicar evento, *o sistema deve* usar outbox: (1) persiste `outbox_events` na mesma transação, (2) publisher drena outbox assincronamente.

**Critério de aceite**:
- Tabela `outbox_events {id, subject, payload, status, attempts}`.
- Drain worker (job 10s).
- Retry com backoff.

**Fonte**: [[STEERING]] §14, patterns de microservices.

---

## XD-003 — Nenhum cross-BC import

**Prioridade**: M  
**EARS**: *Quando* code review roda, *o sistema deve* bloquear import cruzado entre BCs (ex: `commercial/application/` importando `institutional/domain/`). Comunicação só via events.

**Critério de aceite**:
- Linter (ArchUnit-like Go ou `go vet` custom).
- Regra em CI.

**Fonte**: [[STEERING]] §14.

---

## XD-004 — ABAC engine: matrix ordenada

**Prioridade**: M  
**EARS**: *Quando* request atinge handler protegido, *o sistema deve* avaliar na ordem:
1. `admin_bypass` — role `admin` bypass → permit.
2. `role_check` — role permite ação? → se não, deny `ROLE_NOT_ALLOWED`.
3. `action_check` — ação configurada pro plan_tier? → se não, deny `ACTION_NOT_CONFIGURED` ou `PLAN_REQUIRED`.
4. `tenant_check` — user tem membership ativo no tenant? → se não, deny `TENANT_MISMATCH`.
5. `scope_check` — user tem escopo pra sub-recurso? → se não, deny `OUT_OF_SCOPE`.

Resposta negativa: `403 forbidden` com `code` canônico.

**Dependências**: GR-002.

**Critério de aceite**:
- Middleware `RequirePermission(action, resource)`.
- Retorna `ABACDecision {allowed, role, plan_tier, reason}`.
- Teste: 100+ cenários.

**Fonte**: legado `identity.md` Req 2, `cross-domain.md` Req 2.

---

## XD-005 — ABAC cache Redis

**Prioridade**: M  
**EARS**: *Quando* ABAC resolve decisão, *o sistema deve* cachear em Redis `ms:abac:{user_id}:{tenant_id}` TTL 5min.

**Critério de aceite**:
- Cache hit: < 1ms.
- Miss: consulta DB + write cache.

**Fonte**: legado `cross-domain.md` Req 2.

---

## XD-006 — ABAC invalidation via webhook

**Prioridade**: M  
**EARS**: *Quando* evento que afeta permissões ocorre (Stripe subscription updated, Zitadel role changed — futuro), *o sistema deve* invalidar cache Redis `ms:abac:{user_id}:*`.

**Dependências**: BIL-012.

**Critério de aceite**:
- Subscription Stripe → invalida cache user afetado.
- Decision em aberto D-001 legado: purge instantâneo vs stale 5min — resolvido como invalida via webhook.

**Fonte**: legado `cross-domain.md` Req 2, D-001.

---

## XD-007 — ABAC matrix definition (feature × persona × plan × action)

**Prioridade**: M  
**EARS**: Matriz consolidada em `abac_policies {action, resource, roles_allowed[], plan_tiers_allowed[], scope_required}`. Ver [[../matrix-functional]] para tabela completa.

**Critério de aceite**:
- Seed de policies por migration.
- Frontend renderiza botão só se ABAC pré-resolve OK.

**Fonte**: legado `functional-matrix.md`, [[../matrix-functional]].

---

## XD-008 — IAuditLogger (DT-010)

**Prioridade**: M  
**EARS**: *Quando* evento crítico ocorre em qualquer BC, *o sistema deve* chamar `IAuditLogger.Log(ctx, AuditEntry {user_id, tenant_id, action, resource_type, resource_id, changes_before, changes_after, ip, user_agent, request_id})`.

**Dependências**: GR-026.

**Critério de aceite**:
- Interface `internal/shared/audit`.
- Implementação default: grava em `audit_trail` table.
- Assíncrono (não bloqueia request) via channel buffered.
- DT-010 legado: remover `audit.Log` atual acoplado e promover interface.

**Fonte**: legado DT-010, `cross-domain.md` Req 37.

---

## XD-009 — Audit trail append-only

**Prioridade**: M  
**EARS**: *Enquanto* row existe em `audit_trail`, *o sistema deve* rejeitar `UPDATE` e `DELETE` via DB trigger.

**Dependências**: GR-026.

**Critério de aceite**:
- Trigger `prevent_mutation_audit_trail`.
- Retenção 5 anos + archive S3 Glacier após 1 ano.

**Fonte**: legado `cross-domain.md` Req 37.

---

## XD-010 — LGPD logs (registro canônico R40, schema explícito)

**Prioridade**: M  
**EARS**: *Quando* dado pessoal é revelado (Connect Me, dossiê, export) ou tratado (consent, deletion, portability, sharing), *o sistema deve* gravar entry append-only em `lgpd_logs` com schema canônico (R40 absorção Fase C):

```
{
  id UUIDv7 PK,
  data_subject_id UUID,           -- usuário cujo dado foi tratado
  data_recipient_id UUID NULL,    -- quem recebeu (Connect Me reveal, partner_link, etc.)
  data_types_shared TEXT[],       -- enum: profile, contact, financial, location, document, evaluation, …
  action TEXT,                    -- enum: shared, exported, anonymized, deleted, consent_given, consent_revoked
  purpose TEXT,                   -- enum: connect_me_revelation, billing, fraud_prevention, customer_support, tax_reporting, marketing_optout, mandate_dossier, neighborhood_partner_link, external_integration
  legal_basis TEXT,               -- enum LGPD: consent, contract, legal_obligation, vital_interest, public_task, legitimate_interest
  consent_version TEXT,           -- versão do termo aceito (NULL se legal_basis ≠ consent)
  ip_address INET,
  user_agent TEXT,
  related_resource_type TEXT,     -- 'connect_me_request', 'deal', 'mandate_dossier', etc.
  related_resource_id UUID,
  occurred_at TIMESTAMPTZ NOT NULL,
  retention_until TIMESTAMPTZ NOT NULL DEFAULT (occurred_at + INTERVAL '5 years')
}
```

**Dependências**: GR-024, GR-025, GR-026, ADR-0028 (LGPD keyed HMAC pseudonymization).

**Critério de aceite**:
- Append-only (DB trigger `prevent_mutation_lgpd_logs`).
- Retenção mínima 5 anos (R40 §7).
- Direito ao esquecimento: ao DSAR delete, pseudonimizar `data_subject_id` via keyed HMAC (D-101 — preserva agregação estatística sem PII direto). NÃO purgar antes de 5y.
- Exportar lgpd_logs por user em `GET /me/lgpd-export` (R40 §8).
- Index em `(data_subject_id, occurred_at DESC)` + `(occurred_at, retention_until)` pra queries DSAR + retention purge.
- `data_types_shared` com taxonomia controlada (não free text) — facilita relatórios LGPD ao titular.

**Fonte**: legado `cross-domain.md` Req 39, LGPD Art. 37; **R40 absorção Fase C** (schema explícito + data_recipient_id + data_types_shared array + retention_until); ADR-0028; D-101 (LGPD soft delete + HMAC).

---

## XD-011 — Governance score (cross-BC)

**Prioridade**: S  
**EARS**: *Quando* job noturno roda, *o sistema deve* calcular `governance_score` por condomínio baseado em 10 componentes (ver INS-032). Alerta se < 60.

**Dependências**: INS-032, audit trail.

**Critério de aceite**:
- Histórico 12 meses.
- Público (transparência).

**Fonte**: legado `cross-domain.md` Req 33.

---

## XD-012 — Provider abstraction (Stripe)

**Prioridade**: M  
**EARS**: *Quando* billing interage com Stripe, *o sistema deve* usar `IPaymentGateway` (interface). SDK Stripe isolado em `infrastructure/providers/stripe/`.

**Dependências**: [[STEERING]] §10.

**Critério de aceite**:
- Métodos: `CreateCheckoutSession, CreateBillingPortalSession, UpsertSubscription, CreateRefund, ParseWebhookEvent, ListInvoices`.
- Teste via mock.

**Fonte**: legado `cross-domain.md` Req 61, BIL-*.

---

## XD-013 — Provider abstraction (Mux)

**Prioridade**: M  
**EARS**: *Quando* content interage com Mux, *o sistema deve* usar `IVideoProvider`. SDK isolado.

**Critério de aceite**:
- Métodos: `CreateDirectUpload, GetAsset, CreateSignedPlaybackURL, ParseWebhookEvent, DeleteAsset`.

**Fonte**: legado `cross-domain.md` Req 64, CNT-001.

---

## XD-014 — Provider abstraction (Livekit)

**Prioridade**: C  
**EARS**: *Quando* assembly interage com Livekit, *o sistema deve* usar `ILiveProvider` (create room, join token, mute participant, recording).

**Fonte**: legado `cross-domain.md` Req 68.

---

## XD-015 — Provider abstraction (Email)

**Prioridade**: M  
**EARS**: *Quando* sistema envia email transacional, *o sistema deve* usar `IEmailProvider`. Default Resend ou AWS SES (D-026 legado aberta).

**Critério de aceite**:
- Templates pt-BR pré-built: boas-vindas, reset senha, assembleia, Connect Me interesse, comunicado broadcast.
- Domínio: `noreply@mastersindico.com.br`.
- Bounce handling: invalida email se bounce persistente.

**Fonte**: legado `cross-domain.md` Req 62.

---

## XD-016 — Provider abstraction (SMS)

**Prioridade**: S  
**EARS**: *Quando* sistema envia SMS (OTP, notificações críticas), *o sistema deve* usar `ISMSProvider`. Default Twilio, fallback Zenvia.

**Critério de aceite**:
- Consentimento morador (opt-in).
- Rate limit: 1 SMS/min/user.
- Métrica de delivery rate.

**Fonte**: legado `cross-domain.md` Req 63.

---

## XD-017 — Provider abstraction (Storage S3-compatível)

**Prioridade**: M  
**EARS**: *Quando* sistema persiste objeto, *o sistema deve* usar `IStorageProvider` (MinIO dev, Cloudflare R2 ou AWS S3 prod).

**Critério de aceite**:
- Buckets: `videos` (via Mux na verdade), `documents`, `media`, `public`.
- Ciclo de vida: archive após 1 ano, delete 7 anos (LGPD).
- Presigned URL TTL default 24h.

**Fonte**: legado `cross-domain.md` Req 65.

---

## XD-018 — Provider abstraction (ViaCEP)

**Prioridade**: M  
**EARS**: *Quando* sistema valida CEP, *o sistema deve* usar `ICEPLookup`. Cache Redis `cep:{cep}` TTL 30d. Fallback manual se indisponível.

**Critério de aceite**:
- Validação contrarreja CEP inválido.

**Fonte**: legado `cross-domain.md` Req 66.

---

## XD-019 — Provider abstraction (Push FCM)

**Prioridade**: S  
**EARS**: *Quando* sistema envia push, *o sistema deve* usar `IPushProvider` (FCM default). Registro de `device_token` ao login mobile.

**Critério de aceite**:
- Topics: `assembly_{condominium_id}`, `user_{user_id}`.
- Opt-in obrigatório (GR-012 LGPD).

**Fonte**: legado `cross-domain.md` Req 67.

---

## XD-020 — Provider abstraction (CNPJ validator)

**Prioridade**: S (M2+ pleno)  
**EARS**: *Quando* sistema valida CNPJ, *o sistema deve* usar `ICNPJValidator`. M1 stub (algoritmo dígito). M2+ integração Receita Federal adapter.

**Fonte**: legado `commercial.md` Req 18.

---

## XD-021 — Analytics dashboards: síndico, empresa, agência

**Prioridade**: S  
**EARS**: *Quando* user acessa dashboard, *o sistema deve* expor métricas por role (ver Req 43-45 legado):
- Síndico: total moradores, governance score, % conclusão master plan, satisfação média, próximos marcos.
- Empresa: condomínios conectados, CTR Connect Me, taxa aprovação execução, avaliação média, receita (M3+).
- Agência: engajamento cruzado, views médios, retention, top performers.

**Dependências**: CNT-012 privacidade.

**Critério de aceite**:
- Atualização tempo real ou 1h.
- Histórico 12 meses.
- Export CSV/PDF.

**Fonte**: legado `cross-domain.md` Req 43-45.

---

## XD-022 — Antifraude: CAPTCHA + bloqueio IP + anomalia

**Prioridade**: S  
**EARS**: Ver IDN-026 (IP bloqueio), IDN-027 (CAPTCHA pós 3 falhas), IDN-028 (IP novo → SMS).

**Fonte**: legado `cross-domain.md` Req 102.

---

## XD-023 — Visibilidade estrita de vídeos (Rule 4)

**Prioridade**: M  
**EARS**: Ver CNT-009. Cross-domain enforcement via `video_visibility_grants` + NATS listener `assembly.voting.closed` → revoga.

**Fonte**: legado `cross-domain.md` Req 103.

---

## XD-024 — Mandato: encerramento + transferência

**Prioridade**: S  
**EARS**: Ver INS-002, INS-003. Cross-domain: gera dossiê via XD-015 + XD-017.

**Fonte**: legado `cross-domain.md` Req 46-47.

---

## XD-025 — Assinaturas obrigatórias de termos (versionamento)

**Prioridade**: M  
**EARS**: Ver IDN-024. Termos: Terms of Service, Privacy Policy (LGPD), Código de Conduta, Termos de Assembleia.

**Critério de aceite**:
- Re-aceite em nova versão.
- Audit trail.

**Fonte**: legado `cross-domain.md` Req 35.

---

## XD-026 — Declaração anual de compliance (síndico)

**Prioridade**: S  
**EARS**: Ver INS-033.

**Fonte**: legado `cross-domain.md` Req 34.

---

## XD-027 — Conflito de interesse declarado

**Prioridade**: C  
**EARS**: Ver IDN-034. Síndico declara no onboarding + anual. Impacta votação (abstenção obrigatória).

**Fonte**: legado `cross-domain.md` Req 36.

---

## XD-028 — Certificações e compliance markers (autodeclarados)

**Prioridade**: M  
**EARS**: Síndico e Empresa declaram via checkboxes (INS-040, COM-002). Disclaimer obrigatório.

**Fonte**: legado `cross-domain.md` Req 41-42.

---

## XD-029 — Mapa de riscos (condomínio)

**Prioridade**: C  
**EARS**: Ver INS-036. Integra com master_plan (INS-019).

**Fonte**: legado `cross-domain.md` Req 38.

---

## XD-030 — Dossiê PDF assinado digitalmente

**Prioridade**: S  
**EARS**: Ver INS-034. Armazenado em S3, verificável.

**Fonte**: legado `cross-domain.md` Req 40.

---

## XD-031 — Sentry integration (errors)

**Prioridade**: M  
**EARS**: *Quando* erro não-operacional ocorre, *o sistema deve* enviar pra Sentry com `request_id` tag, stack trace sanitizado (sem PII).

**Dependências**: GR-011, GR-007.

**Critério de aceite**:
- Scrubber regex (PII).
- Projects: backend, web, mobile.

**Fonte**: legado `CLAUDE.md` §3 observability.

---

## XD-032 — OpenTelemetry integration

**Prioridade**: M  
**EARS**: *Quando* request entra, *o sistema deve* criar root span; cada use case + SQL query → span filho. Metrics: counters de events + histograms de latência.

**Dependências**: GR-011.

**Critério de aceite**:
- OTel collector gRPC.
- Grafana dashboards.

**Fonte**: legado sprint 7.

---

## XD-033 — Railway → AWS ECS migration (backlog)

**Prioridade**: C (M4+)  
**EARS**: *Quando* load justifica, *o sistema deve* migrar de Railway pra AWS ECS Fargate (D-024 legado aberta).

**Fonte**: legado STATE D-024.

---

## XD-034 — Health checks

**Prioridade**: M  
**EARS**: *Quando* health check é chamado, *o sistema deve* responder:
- `/healthz` — liveness (sempre 200 se processo vivo).
- `/readyz` — readiness (200 só se DB + Redis + NATS OK).

**Critério de aceite**:
- Railway healthcheck usa `/readyz`.

**Fonte**: legado ops.

---

## XD-035 — Feature flag system (rollout gradual)

**Prioridade**: C  
**EARS**: *Quando* admin liga feature flag com % rollout, *o sistema deve* habilitar pra user subset (hash(user_id) módulo 100 < %).

**Dependências**: BIL-022.

**Critério de aceite**:
- Tabela `feature_flags`.
- Admin UI.

**Fonte**: legado `billing.md` Req 6.

---

## Fase C — Deltas absorvidos 2026-04-25

### XD-036 — Context Confirmer (anti-erro publicação cross-condomínio)

**Prioridade**: M (M1 — síndicos com 2+ condomínios ativos)  
**EARS**: *Quando* síndico com 2+ memberships ativos (`role=syndic, is_active_mandate=true`) emite ação de publicação em qualquer endpoint mutating, *o sistema deve* (R94 absorção Fase C) **ANTES** do handler executar mudança:
1. Detectar contexto ativo via `X-Tenant-Id` (IDN-013) + lista de memberships ativos.
2. Se `mandates_count ≥ 2`, retornar pré-resposta `200 OK` com `{requires_context_confirm: true, condominium_name, address_summary, action_type, alternatives: [...]}` **sem persistir** a ação.
3. Frontend exibe modal: "Confirmar publicação em **<nome do condo>** (<endereço resumido>)?" com 3 opções: `Confirmar | Trocar condomínio | Cancelar`.
4. Quando user confirma → re-emite request com `X-Context-Confirmed: <token-curto-válido-30s>` → backend processa e emite evento `ContextConfirmed`.
5. Síndicos com **1 só condomínio** → bypass automático (sem modal).

**Endpoints afetados** (R94 §6 — lista canônica):

```
POST /timeline-entries          (publicação)
POST /events                    (criar evento)
POST /communications            (criar comunicado)
POST /comunicados/:id/publish
POST /proposals                 (síndico envia para empresa)
POST /closed-deals
POST /assemblies
POST /master-plan/activities
PATCH/PUT em qualquer aggregate institutional do condomínio
```

**Critério de aceite**:
- Token de confirmação `X-Context-Confirmed`: HMAC-keyed JWT com claim `{user_id, condominium_id, action_signature_hash, exp: now+30s}` (ADR-0028).
- Cache de bypass: se síndico tem só 1 mandato ativo → endpoint pula confirmação (header opcional ignorado).
- Audit trail: `ContextConfirmed` event registrado em `audit_trail` com `condominium_id` + `action_signature` (proteção contra ataque CSRF cross-condo).
- LGPD: zero coleta adicional de dados (apenas sinaliza UI).
- Frontend tem responsabilidade de renderizar modal — backend só sinaliza.

**Dependências**: IDN-013 (multi-context membership), IDN-014 (seletor de contexto), ADR-0028 (HMAC), GR-002 (ABAC).

**Fonte**: **R94 absorção Fase C** (`_chaos/requirements(6).md`).

---

### XD-037 — Adjustment (Adendo) como aggregate cross-domain — schema canônico (D-131)

**Prioridade**: M (M1 doc; M2 implementação)  
**EARS**: *Quando* qualquer modificação em registro imutável é necessária, *o sistema deve* (D-131 Fase C ratificação) usar Adjustment aggregate em **cross-domain BC** (NÃO mais em commercial). *Quando* Adjustment é criado, *o sistema deve* aceitar `target_type` em enum 6 itens canônicos:

```
target_type ∈ {
  timeline_entry,       -- INS-012 (Timeline)
  closed_deal,          -- COM-019
  master_plan_action,   -- INS-017..019
  ata_minutes,          -- ASM-024 (Ata assembleia)
  announcement,         -- INS-022 (Comunicado)
  agenda_item           -- ASM-003 (item de pauta)
}
```

**Schema** (D-131 absorção Fase C):
```
adjustments {
  id UUIDv7 PK,
  target_type TEXT NOT NULL,      -- enum acima
  target_id UUID NOT NULL,
  previous_adjustment_id UUID,    -- chain (adendo de adendo)
  reason TEXT NOT NULL,
  description TEXT NOT NULL,
  changes_summary JSONB,
  signatures JSONB,               -- {company_signed_at, sindico_signed_at, ...}
  lgpd_redacted BOOL DEFAULT false,
  created_by UUID NOT NULL,
  created_at TIMESTAMPTZ NOT NULL
}
```

**Invariantes (consolidadas D-131)**:
- **INV-ADJ-001..005**: já existentes (imutabilidade, encadeamento, dupla assinatura quando aplicável, audit trail, NATS publish).
- **INV-ADJ-006 (NEW Fase C — D-131)**: profundidade máxima da cadeia adendo-de-adendo = **5**. Após 5, exige criar adendo ao **adendo-raiz** (não recursivo infinito).
- **LGPD interaction**: adendo de retratação por direito ao esquecimento marca `target.lgpd_redacted=true` (mascarar conteúdo na UI mas manter `audit_entries` 5y).

**Critério de aceite**:
- Trigger DB: `INSERT INTO adjustments` com `previous_adjustment_id` que já tenha 4 ancestrais → REJECT (retorna `previous_adjustment_id` chain depth ≥ 5).
- Frontend valida cadeia antes de submit (pré-check via `GET /adjustments/{id}/chain-depth`).
- Audit: `AdjustmentCreated` event + `lgpd_logs` se `lgpd_redacted=true`.
- Cross-link com [[../../03-subprojects/web/ui-catalog/compliance/C6]] (UI compliance — Fase D).

**Dependências**: D-131 (Adjustment cross-BC), D-105 (origem aggregate), ADR-0028, INS-012, COM-019, INS-022, ASM-024.

**Fonte**: **D-131 + R25 absorção Fase C**; consolida COM-043 + INS-016 (ambos referenciam Adjustment como mecanismo único de modificação imutável).

---

### XD-038 — Contractual Compliance Tracker (R39)

**Prioridade**: S (M2)  
**EARS**: *Quando* deal é confirmed (COM-019), *o sistema deve* extrair entregáveis declarados em `expected_deliverables` (estruturado a partir do scope da proposta) e popular `contract_compliance_items {deal_id, deliverable_id, expected_at, actual_at, status, evidence_url}`. *Quando* síndico marca deliverable status, *o sistema deve* aceitar `status ∈ {pending, completed, delayed, not_delivered}`. *Quando* `delayed` ou `not_delivered` em deliverable de deal ativo → emit `ComplianceIssueDetected` event + flag deal em "Empresa com pendências".

**Critério de aceite**:
- Calculate `compliance_percentage = completed / total` por deal (R39 §2).
- Compliance dashboard por empresa expõe `avg_compliance_pct` (input pra reputation COM-025).
- Display `flagged_deals` em `GET /companies/{id}/compliance-status`.
- Componente do governance_score (D-103 INV-GOV-001) — peso negativo se síndico tem deals com compliance issues.
- Componente do compliance_score (D-103 INV-GOV-002) — afeta empresa.

**Dependências**: COM-019, COM-021 (validação execução), D-103 (scores), XD-009 (audit_trail).

**Fonte**: **R39 absorção Fase C**.

---

## Invariantes cross-domain

1. Audit trail append-only — nunca DELETE, nunca UPDATE.
2. LGPD logs: retenção 5 anos + purga pós-vencimento.
3. Governance Score: calc diário, histórico 12 meses.
4. Todas as assinaturas: rastreadas `{timestamp, ip, version}`.
5. Integrações agnósticas (interfaces IPaymentGateway, etc).
6. Cache ABAC invalidação via webhook (não stale).
7. Nenhum cross-BC import (linter).

---

## Pendências

- ⚠️ **PENDÊNCIA XD-001**: NATS JetStream vs Kafka vs sync in-process pra intra-módulo — ADR.
- ⚠️ **PENDÊNCIA XD-015**: Resend vs AWS SES — D-026 legado aberta.
- ⚠️ **PENDÊNCIA XD-017**: Cloudflare R2 vs AWS S3 direto — custo/dev.
- ⚠️ **PENDÊNCIA XD-020**: provider Receita Federal (ReceitaWS? SERPRO?) — ADR.
- ⚠️ **PENDÊNCIA XD-033**: cronograma migração Railway → ECS — dependente de load.

---

## Fase 12 — atualizações 2026-04-24

### Req 33 atualizado — Score duplo do síndico (via D-103)

**Anterior**: 1 score unificado 0-100.
**Canônico**: **2 scores separados**:
- `governance_score ∈ [1.0, 10.0]` — ver [[../../01-domain/invariants#INV-GOV-001]]
- `compliance_score ∈ [0, 100]` — ver [[../../01-domain/invariants#INV-GOV-002]]

Ambos são calculados cross-BC (identity + institutional + commercial + compliance) e expostos no perfil do síndico.

Endpoints:
- `GET /api/v1/identity/syndics/:id/governance-score` → `{score, breakdown, updated_at}`
- `GET /api/v1/identity/syndics/:id/compliance-score` → `{score, breakdown, updated_at}`
- `GET /api/v1/identity/syndics/:id/scores` → combinado (conveniência S32)

**Distinção**:
- Scores síndico (D-103) são **individuais por síndico**.
- **Reputação Bronze→Diamante** ([[../../STATE#D-051]]) é de **empresas no marketplace** (Connect Me). Não misturar.

### IValidatable cross-BC interface (nova)

**Fonte**: [[../../STATE#D-105 — ValidationItem como conceito (Opção B DDD puro)|D-105]]; [[../../01-domain/patterns/validatable-interface]].

5 aggregates commercial implementam. Hub institutional consome via view materializada `pending_validations` ou endpoint agregador:

```
GET /api/v1/institutional/validation-items
  ?condominium_id=<uuid>&status=pending&type=all&limit=50&offset=0
```

ABAC: `syndic` OR `admin` do condomínio. Tenant isolation obrigatório. Ver pattern para spec completa.

---

## Referências

- Legado: `specs/requirements/cross-domain.md`, `identity.md` Req 2, DT-010.
- [[../global]] GR-002 (ABAC), GR-026 (audit trail), GR-035 (event-driven).
- [[../compliance-lgpd]]
- [[../../01-domain/bounded-contexts]]
- [[../matrix-functional]]
- [[_moc]]
