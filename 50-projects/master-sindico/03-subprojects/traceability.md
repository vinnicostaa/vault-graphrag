---
title: Traceability Matrix — Master Síndico
type: note
tags:
  - master-sindico
  - traceability
  - rastreabilidade
  - cross-stack
project: master-sindico
created: 2026-04-24T00:00:00.000Z
updated: 2026-04-24T00:00:00.000Z
---

# Traceability Matrix — Master Síndico

Matriz **end-to-end** conectando `Tela (web/mobile) → Sub-produto → Requirement (FR-###) → Endpoint backend → Invariante (INV-###) → Sprint → Status`.

Use esta matriz para:

- Saber **qual endpoint** uma tela consome (web ou mobile)
- Saber **qual invariante** cada ação dispara
- Saber **quando** uma feature foi/será entregue (sprint)
- Identificar **gaps** (tela sem endpoint, endpoint sem tela, invariante sem UI)

Legenda de status:

- ✅ backend + UI entregues
- 🟢 backend ok, UI em spec
- 🟡 pendente (parcial / spec sem build)
- 🔴 bloqueado (gap de endpoint, ADR ausente ou dependência M1)
- ⚫ sistema (webhook / cron — sem UI por design)

---

## 1. Sumário por sub-produto (11)

Visão geral de cobertura por sub-produto. Fonte: [[../00-product/sub-produtos/_moc]].

| # | Sub-produto | Telas web | Telas mobile | Endpoints cobertos | Invariantes relevantes | Sprint entrega | Status |
|---|---|---|---|---|---|---|---|
| 1 | [[../00-product/sub-produtos/01-governanca-institucional\|Governança Institucional]] | 24 (S1-S17 · S21-S27 · M1-M15) = **39** | 15 (M1-M15) + 5 (S1, S2, S6-S8) = **20** | ~38 `/institutional/*`, `/syndic/governance/*`, `/condominiums/{id}/*` | INS-014 timeline append-only · INV-021 fração ≤100% · INV-024 imutabilidade · INV-029 dep ≤2 · INV-027 ≤15 condos · INV-026 membership única · INV-119 DB-level immutability | 2 · 3 (✅) · 10 (🟡 M1) | ✅ backend · 🟡 UI M1 |
| 2 | [[../00-product/sub-produtos/02-connect-me\|Connect Me]] | 4 (S18-S20 · E2) + E4-E7 = **8** | 0 (não planejado M1) | `/commercial/connect-me/*`, `/commercial/proposals/*`, `/commercial/marketing-link/*` | INV-037 unidirecional · INV-045/046 quota reset anual · INV-047 auto-close 48h · INV-048 LGPD log · INV-049 agência zero-access | 4 ✅ | 🟢 backend pronto · 🟡 UI M1 |
| 3 | [[../00-product/sub-produtos/03-reputacao\|Reputação]] | 4 (S22 · E6 · E10 · E11) = **4** | 0 | `/commercial/evaluations/*`, `/commercial/reputation/*`, `/commercial/executions/*` | INV-036 determinística · INV-043 evaluation única · INV-044 anonimato · INV-054 recalc idempotente · INV-055 suspend on harassment | 4 ✅ | 🟢 |
| 4 | [[../00-product/sub-produtos/04-conteudo-videos\|Conteúdo & Vídeos]] | 1 (E12) + LMS* = **16** | 0 | `/content/videos/*`, `/academy/*`, `/webhooks/mux` | INV-056 lock 90d · INV-060 sem leak cross-tenant · INV-061/062 unlock síndico · INV-064 HMAC Mux · INV-065/070 saga+URL signed · INV-067 parceiro BLOCK LMS · INV-068 cert auto | 5 ✅ | 🟢 |
| 5 | [[../00-product/sub-produtos/05-assembleia-live\|Assembleia Live]] | 21 (ASM-*) = **21** | 5 (ASM-CHKIN · ASM-CIEN · ASM-FILA-FALA · ASM-PROC-CAD · ASM-VOTO) = **5** | `/assemblies/*`, `/webhooks/livekit` | INV-071 voto único · INV-072 ata imutável · INV-073 quórum fracional · INV-074 sem edital = sem início · INV-081 síndico não vota · INV-084 saga Livekit · INV-085 homologação · INV-118 edital ≥8d · INV-119 DB-level | 7 ✅ | 🟢 backend · 🟡 UI M1 |
| 6 | [[../00-product/sub-produtos/06-lms-academia\|LMS Academia]] | 15 (LMS1-LMS15) = **15** | 0 | `/academy/*`, `/certificates/verify/*`, `/ws/academy/lives/*/chat` | INV-068 cert auto · INV-067 parceiro DENY · INV-069 ebook tier · CNT-022 lives = admin + Empresa Pro + agência delegada (D-097 revoga D-063/D-076) | M2 backlog | 🟡 spec |
| 7 | [[../00-product/sub-produtos/07-banco-de-talentos\|Banco de Talentos]] | 11 (BT01-BT11) + busca empresa TBD = **11** | 0 | `/talents/curriculum/*`, `/talents/search/*`, `/webhooks/mux` (vídeo) | INV-BT-01 ≤90s · INV-BT-04 consent · INV-BT-05 1 por morador · INV-BT lock 90d | M2 | 🟡 spec |
| 8 | [[../00-product/sub-produtos/08-vizinhanca\|Vizinhança]] | 29 (VS+VE+VM+VZ) = **29** | 7 (VM1-VM7) = **7** | `/neighborhood/promotions/*`, `/neighborhood/partners/*`, `/neighborhood/coupons/*` | INV-051 cooldown 4h · INV-052/053 coupon format · INV-067 parceiro sem LMS · ABAC `exclusive_benefit.view` tenant-match | M2 backlog | 🟡 spec |
| 9 | [[../00-product/sub-produtos/09-compliance\|Compliance]] | 15 (C1-C11 · S28-S31) = **15** | 0 | `/compliance/*`, `/condominiums/{id}/compliance/*` | INV-033 audit append-only · INV-034 declaração anual · INV-035 conflito onboarding · INV-090 audit DB level · INV-091 retenção 5a · INV-119/120 DB-level | 3 ✅ (audit) · 10 (🟡 UI) | 🟢 backend · 🟡 UI |
| 10 | [[../00-product/sub-produtos/10-marketing-agencias\|Marketing & Agências]] | 8 (MK1-MK8) + E14 · E16 = **10** | 0 | `/marketing/*`, `/enterprise/agencies/*`, `/commercial/marketing-link/*` | INV-049 zero-access deals/CM · INV-066 token escopo restrito · IDN-025 invite token | 5 ✅ | 🟢 backend · 🟡 UI |
| 11 | [[../00-product/sub-produtos/11-administradoras-condominiais\|Administradoras]] | 0 (visão dentro de S*) | 0 | `/institutional/enterprise-admin/*` | INS-004 token por email · INV-026 membership única | M2 | 🟡 spec |

**Totais globais**:

- **Telas web**: 161 (sindico 31 + morador 15 + empresa 16 + marketing 8 + compliance 11 + vizinhanca 29 + lms 15 + banco-talentos 11 + assembleia 21 + onboarding 23 + admin 1)
- **Telas mobile**: 59 (morador 15 + sindico 5 + assembly 5 + onboarding 21 + vizinhanca 7 + shared 6)
- **Cross-stack únicas**: ~186 superfícies de UI distintas
- **Endpoints identificados** (contagem bruta após dedup): ~240 rotas REST/WS distintas
- **FRs funcionais consultados**: 7 domínios (IDN · BIL · INS · COM · CNT · ASM · XD) — ~230 requisitos
- **INVs referenciados**: 116 invariantes canônicos (v2 + Fase 5 + Fase 9)

---

## 2. Matriz detalhada por sub-produto

### 2.1 Governança Institucional (M1 — Sprint 2/3 ✅ backend; Sprint 10 🟡 UI)

Catálogo S1-S17 · S21-S27 (síndico) + M1-M15 (morador). Mobile: M1-M15 + seletor S1/S2.

| Tela | Stack | Persona | Ação principal | Endpoint | Método | FR | INV | Sprint | Status |
|---|---|---|---|---|---|---|---|---|---|
| [[web/ui-catalog/sindico/S1]] | web | síndico | Aceitar termo governança (first-access) | `/api/v1/syndic/governance/first-access` · `/acceptance` | GET · POST | IDN-024 · XD-025 | INV-100 cookie · INV-024 audit | 10 | 🟡 endpoint inferido |
| [[web/ui-catalog/sindico/S2]] | web | síndico | Listar condomínios do síndico | `/api/v1/condominiums?owner=me` | GET | INS-001 · INS-002 | INV-027 ≤15 · INV-026 | 2 ✅ | ✅ |
| [[web/ui-catalog/sindico/S2]] | web | síndico | Encerrar mandato | `/api/v1/condominiums/{id}/end-mandate` | POST | INS-003 | INV-034 decl anual · INV-027 | 3 | 🟡 endpoint inferido |
| [[web/ui-catalog/sindico/S3]] | web | síndico | Criar condomínio (passo endereço) | `/api/v1/condominiums` | POST | INS-001 | INV-021 Σ fração · INV-030 public_id | 2 ✅ | ✅ |
| [[web/ui-catalog/sindico/S4]] | web | síndico | Atribuir mandato ao síndico | `/api/v1/condominiums/{id}/mandate` | POST | INS-002 · IDN-034 | INV-028 datas · INV-035 conflito | 2 ✅ | ✅ |
| [[web/ui-catalog/sindico/S5]] | web | síndico | ID/QR do condomínio | `/api/v1/condominiums/{id}` | GET | INS-001 | INV-030 public_id | 2 ✅ | ✅ |
| [[web/ui-catalog/sindico/S6]] | web | síndico | Central (13 cards + indicadores) | `/api/v1/condominiums/{id}/dashboard/summary` · `/cards/badges` · `wss://.../events` | GET · GET · WS | INS-032 score · XD-021 | INV-089 tenant · INV-122 guard | 3 · 10 (🟡) | 🟡 endpoint agregador inferido |
| [[web/ui-catalog/sindico/S7]] | web | síndico | Plano Diretor — lista | `/api/v1/condominiums/{id}/master-plan/activities` | GET | INS-017 · INS-018 | INV-031 status by event | 3 ✅ | ✅ |
| [[web/ui-catalog/sindico/S8]] | web | síndico | Nova Ação PD | `/api/v1/condominiums/{id}/master-plan/activities` | POST | INS-017 · INS-019 | INV-024 imutabilidade | 3 ✅ | ✅ |
| [[web/ui-catalog/sindico/S9]] | web | síndico | Lista Ações PD (filtros) | `/api/v1/condominiums/{id}/master-plan/activities?filters=...` | GET | INS-017 | INV-089 | 3 ✅ | ✅ |
| [[web/ui-catalog/sindico/S10]] | web | síndico | Detalhe Ação PD + histórico | `/api/v1/condominiums/{id}/master-plan/activities/{aid}` | GET | INS-018 | INV-031 | 3 ✅ | ✅ |
| [[web/ui-catalog/sindico/S11]] | web | síndico | Timeline — lista | `/api/v1/institutional/condominiums/{id}/timeline` | GET | INS-011 · INS-014 | INV-024 append-only · INV-119 DB-level | 2 ✅ | ✅ |
| [[web/ui-catalog/sindico/S12]] | web | síndico | Criar atividade LT | `/api/v1/condominiums/{id}/timeline` · `/api/v1/uploads` | POST · POST mp | INS-011 | INV-024 · INV-032 adendo · INV-119 | 2 · 3 ✅ | ✅ |
| [[web/ui-catalog/sindico/S13]] | web | síndico | Editar LT (nova versão adendo) | `/api/v1/institutional/timeline/{id}/amend` | POST | INS-016 | INV-032 corrects_entry · INV-050 | 3 ✅ | ✅ |
| [[web/ui-catalog/sindico/S14]] | web | síndico | Histórico LT | `/api/v1/institutional/timeline/{id}/history` | GET | INS-011 | INV-024 | 3 ✅ | ✅ |
| [[web/ui-catalog/sindico/S15]] | web | síndico | Eventos — lista + indicadores | `/api/v1/condominiums/{id}/events` | GET | INS-025 | INV-089 | 3 ✅ | ✅ |
| [[web/ui-catalog/sindico/S16]] | web | síndico | Criar evento (18+ tipos) | `/api/v1/condominiums/{id}/events` | POST | INS-025 · INS-028 | INV-089 | 3 ✅ | ✅ |
| [[web/ui-catalog/sindico/S17]] | web | síndico | Lista eventos (submenu) | `/api/v1/condominiums/{id}/events?filter=...` | GET | INS-025 | — | 3 ✅ | ✅ |
| [[web/ui-catalog/sindico/S21]] | web | síndico | Registros de Execução — hub | `/api/v1/commercial/executions?condominium={id}` | GET | COM-020 · COM-021 | INV-041 deal→timeline | 4 ✅ | ✅ |
| [[web/ui-catalog/sindico/S22]] | web | síndico | Detalhe registro execução (5 critérios) | `/api/v1/commercial/executions/{id}` · `/validate` | GET · POST | COM-021 · COM-024 | INV-043 eval única · INV-044 anonimato | 4 ✅ | ✅ |
| [[web/ui-catalog/sindico/S23]] | web | síndico | Comunicados — hub | `/api/v1/condominiums/{id}/announcements` | GET | INS-021 | INV-025 imutável pós-publish | 3 ✅ | ✅ |
| [[web/ui-catalog/sindico/S24]] | web | síndico | Criar comunicado | `/api/v1/condominiums/{id}/announcements` | POST | INS-021 · INS-022 | INV-025 | 3 ✅ | ✅ |
| [[web/ui-catalog/sindico/S25]] | web | síndico | Lista comunicados | `/api/v1/condominiums/{id}/announcements?filter=...` | GET | INS-023 · INS-024 | INV-089 | 3 ✅ | ✅ |
| [[web/ui-catalog/sindico/S26]] | web | síndico | Inbox validações pendentes | `/api/v1/condominiums/{id}/validations?status=pending` | GET | COM-021 · INS-011 | INV-089 · INV-122 guard | 3 · 4 ✅ | 🟡 resumo inferido |
| [[web/ui-catalog/sindico/S27]] | web | síndico | Dashboard agregado (plus) | `/api/v1/condominiums/{id}/dashboard?period=` · `/scores/history` · `/export` | GET · GET · GET | XD-021 · INS-032 | INV-089 · INV-122 | 10 (🔴 endpoint agregador gap) | 🔴 |
| [[mobile/ui-catalog/morador/M1]] | mobile | morador | Painel morador (home) | `/api/v1/institutional/memberships/me/default` · `/institutional/dashboard/resident` | GET · GET | INS-009 · IDN-013 | INV-026 membership única | 10 (🟡 M1) | 🟡 |
| [[mobile/ui-catalog/morador/M2]] | mobile | morador | Selecionar condomínio | `/api/v1/institutional/memberships/me` | GET | IDN-013 · INS-009 | INV-026 | 10 | 🟡 |
| [[mobile/ui-catalog/morador/M3]] | mobile | morador | Buscar condomínio por public_id | `/api/v1/condominiums?public_id={code}` | GET | INS-005 · INS-007 | INV-030 | 10 | 🟡 |
| [[mobile/ui-catalog/morador/M4]] | mobile | morador | Identificar unidade (foto+termo) | `/api/v1/institutional/memberships` · `/api/v1/media/avatars/upload` | POST · POST mp | INS-006 · INS-007 | INV-029 dep ≤2 | 10 | 🟡 |
| [[mobile/ui-catalog/morador/M5]] | mobile | morador | Termo de ciência de vínculo | `/api/v1/institutional/memberships/{id}/consent` | POST | IDN-024 · INS-007 | INV-024 audit · LGPD | 10 | 🟡 |
| [[mobile/ui-catalog/morador/M6]] | mobile | morador | Timeline do condomínio | `/api/v1/institutional/timeline/{cond}?page&size&category` · `/{id}/read` | GET · POST | INS-011 · INS-014 | INV-024 append-only · INV-119 | 10 | 🟡 |
| [[mobile/ui-catalog/morador/M7]] | mobile | morador | Detalhe LT | `/api/v1/institutional/timeline/{id}` | GET | INS-011 | INV-024 | 10 | 🟡 |
| [[mobile/ui-catalog/morador/M8]] | mobile | morador | Plano Diretor (morador) | `/api/v1/condominiums/{id}/master-plan/activities?scope=resident` | GET | INS-017 | INV-089 | 10 | 🟡 |
| [[mobile/ui-catalog/morador/M9]] | mobile | morador | Detalhe Ação PD (morador) | `/api/v1/condominiums/{id}/master-plan/activities/{aid}?scope=resident` | GET | INS-018 | — | 10 | 🟡 |
| [[mobile/ui-catalog/morador/M10]] | mobile | morador | Eventos | `/api/v1/condominiums/{id}/events?scope=resident` | GET | INS-025 · INS-026 | — | 10 | 🟡 |
| [[mobile/ui-catalog/morador/M11]] | mobile | morador | Comunicados (+ ciência) | `/api/v1/condominiums/{id}/announcements` · `/{aid}/read` | GET · POST | INS-021 · INS-023 | INV-089 | 10 | 🟡 |
| [[mobile/ui-catalog/morador/M12]] | mobile | morador | Avaliação bimestral (bloqueante) | `/api/v1/institutional/assessments/me/pending` · `/assessments` | GET · POST | INS-031 | INV-036 reputação · INV-122 | 10 (🔴) | 🔴 |
| [[mobile/ui-catalog/morador/M13]] | mobile | morador | Meu vínculo | `/api/v1/institutional/memberships/me/{condo_id}` · `/lgpd/consents/{id}` | GET · GET | IDN-013 · IDN-033 | INV-026 · LGPD | 10 | 🟡 |
| [[mobile/ui-catalog/morador/M14]] | mobile | morador (titular) | Gerenciar dependentes | `/api/v1/institutional/memberships/{id}/dependents` | GET · POST · DELETE | INS-007 · IDN-033 | INV-029 dep ≤2 | 10 | 🟡 |
| [[mobile/ui-catalog/morador/M15]] | mobile | morador | Encerrar vínculo | `/api/v1/institutional/memberships/{id}/terminate` | POST | INS-008 · IDN-022 | LGPD · INV-091 | 10 | 🟡 |

**Invariantes críticos observáveis em UI** (Governança):

- `INV-021` (Σ fração ≤100%) — form S3/S4 bloqueia + aviso
- `INV-024` (timeline append-only) — UI sempre mostra "editar = novo adendo" (S13)
- `INV-027` (≤15 condos) — botão "novo" desabilitado em S2
- `INV-029` (≤2 dep) — M14 bloqueia adição após 2
- `INV-119` (DB-level immutability) — transparente à UI, mas 403 se tentar PATCH timeline published

---

### 2.2 Connect Me (Sprint 4 ✅ backend; 🟡 UI M1)

| Tela | Stack | Persona | Ação | Endpoint | Método | FR | INV | Sprint | Status |
|---|---|---|---|---|---|---|---|---|---|
| [[web/ui-catalog/sindico/S18]] | web | síndico | Hub Connect Me (quota + listas) | `/api/v1/connect-me/me/summary` · `/connect-me?sender=me` · `?receiver=me` | GET ×3 | COM-006 · COM-012 | INV-045 decrement publish · INV-046 reset anual | 4 ✅ | 🟡 summary inferido |
| [[web/ui-catalog/sindico/S19]] | web | síndico | Criar Connect Me (LGPD) | `/api/v1/companies?q=...` · `/connect-me` | GET · POST | COM-006 · COM-007 | INV-037 unidirecional · INV-045 · INV-048 LGPD log | 4 ✅ | ✅ |
| [[web/ui-catalog/sindico/S20]] | web | síndico | Connect Me recebido (M→S) | `/connect-me?receiver=me&flow=morador_to_sindico` · `/{id}/respond` | GET · POST | COM-009 | INV-048 · INV-037 | 4 ✅ | ✅ |
| [[web/ui-catalog/empresa/E1]] | web | empresa | Painel empresarial | `/api/v1/enterprise/dashboard` · `/active-tenant` | GET · POST | XD-021 | INV-089 · INV-049 | 5 ✅ | 🟡 inferido |
| [[web/ui-catalog/empresa/E2]] | web | empresa | Oportunidades recebidas | `/api/v1/commercial/connect-me/received` | GET | COM-006 | INV-037 · INV-048 (contact oculto) | 4 ✅ | ✅ |
| [[web/ui-catalog/empresa/E3]] | web | empresa | Recusar oportunidade (6 motivos) | `/api/v1/commercial/connect-me/:id/reject` | POST | COM-013 | INV-047 auto-close 48h | 4 ✅ | ✅ |
| [[web/ui-catalog/empresa/E4]] | web | empresa | Manifestar interesse + enviar proposta | `/commercial/connect-me/:id/interested` · `/proposals` · `/proposals/:id/submit` | POST ×3 | COM-007 · COM-015 | INV-048 reveal-log · INV-038 única · INV-039 imutável | 4 ✅ | ✅ |
| [[web/ui-catalog/empresa/E5]] | web | empresa | Minhas propostas (pipeline) | `/api/v1/commercial/proposals?sender=me` | GET | COM-015 · COM-016 | INV-039 · INV-050 adendo | 4 ✅ | ✅ |
| [[web/ui-catalog/empresa/E7]] | web | empresa | Negócios fechados (ciclo) | `/api/v1/commercial/closed-deals/own` · `/:id` | GET ×2 | COM-019 | INV-040 imutável · INV-041 auto-timeline | 4 ✅ | ✅ |
| [[web/ui-catalog/empresa/E14]] | web | empresa | Gestão de agência (delegação) | `/enterprise/agencies` · `/invite` · `/:id/revoke` · `/permissions` | GET · POST · POST · GET | CNT-014 · IDN-025 | INV-049 zero-access · INV-066 escopo | 5 ✅ | ✅ |
| [[web/ui-catalog/empresa/E16]] | web | empresa | Marketing Link (intenção) | `/api/v1/commercial/marketing-link` | POST | COM-011 · CNT-014 | INV-049 · anti-prospecção | 5 ✅ | ✅ |

**Quotas por tier** (INV-045 · INV-046 reset 1/jan BRT):

- trial: 4 envios/ano
- base: 4 envios/ano (hard) · 4 recebidos/ano
- plus: 12 envios
- pro: ilimitado

---

### 2.3 Reputação (Sprint 4 ✅)

| Tela | Stack | Persona | Ação | Endpoint | Método | FR | INV | Sprint | Status |
|---|---|---|---|---|---|---|---|---|---|
| [[web/ui-catalog/sindico/S22]] | web | síndico | Validar execução (5 critérios 1-10) | `/api/v1/commercial/executions/:id/validate` | POST | COM-021 · COM-024 | INV-043 única · INV-044 anonimato · INV-054 recalc | 4 ✅ | ✅ |
| [[web/ui-catalog/empresa/E6]] | web | empresa | Atividade técnica (risco+natureza) | `/api/v1/commercial/technical-activities` | POST | COM-022 | INV-024 audit | 4 ✅ | ✅ |
| [[web/ui-catalog/empresa/E9]] | web | empresa | Registrar execução (estados) | `/api/v1/governance/executions` · `/draft` · `/:id/submit` · `/api/v1/media/upload` | POST · PUT · POST · POST mp | COM-020 | INV-041 auto-timeline on deal | 4 ✅ | ✅ |
| [[web/ui-catalog/empresa/E10]] | web | empresa | Dashboard empresarial (reputação breakdown) | `/api/v1/commercial/reputation/me?breakdown=true` | GET | COM-025 · COM-027 | INV-036 determinística | 4 ✅ | 🟡 inferido |
| [[web/ui-catalog/empresa/E11]] | web | empresa | Avaliações recebidas (pública/privada) | `/commercial/evaluations/received` · `/:id/visibility` | GET · PUT | COM-024 · COM-027 | INV-044 anonimato · INV-043 | 4 ✅ | ✅ |

Reputação é **determinística** (INV-036): mesma entrada → mesma saída. Média ponderada dos 5 critérios + peso por volume de execuções.

---

### 2.4 Conteúdo & Vídeos + LMS Academia (Sprint 5 ✅ · M2/M3)

| Tela | Stack | Persona | Ação | Endpoint | Método | FR | INV | Sprint | Status |
|---|---|---|---|---|---|---|---|---|---|
| [[web/ui-catalog/empresa/E12]] | web | empresa | Perfil institucional (logo, portfolio, vídeos) | `/api/v1/content/videos?empresa_id=me` · `/enterprise/profile` | GET · GET | CNT-002 · COM-003 | INV-056 lock 90d · INV-060 tenant | 5 ✅ | ✅ |
| [[web/ui-catalog/lms/LMS1]] | web | any | Home Academia | `/api/v1/academy/dashboard` · `/paths/recommended` | GET ×2 | CNT-016 | INV-067 parceiro DENY | M2 | 🟡 |
| [[web/ui-catalog/lms/LMS2]] | web | any | Explorar cursos | `/api/v1/academy/courses?filter=...` | GET | CNT-016 | INV-069 ebook tier · ABAC | M2 | 🟡 |
| [[web/ui-catalog/lms/LMS3]] | web | any | Página do curso | `/api/v1/academy/courses/{id}` · `/enroll` | GET · POST | CNT-017 | ABAC tier · INV-067 | M2 | 🟡 |
| [[web/ui-catalog/lms/LMS4]] | web | any | Aula (player) | `/academy/lessons/{id}` · `/progress` · `/complete` | GET · POST · POST | CNT-018 · CNT-010 | INV-070 signed URL TTL · INV-060 | M2 | 🟡 |
| [[web/ui-catalog/lms/LMS5]] | web | any | Conclusão + certificado | `/academy/certificates/by-enrollment/{eid}` · `/certificates/verify/{serial}` (público) | GET × 2 | CNT-019 | INV-068 cert auto 100% | M2 | 🟡 |
| [[web/ui-catalog/lms/LMS6]] | web | any | Treinamentos | `/api/v1/academy/trainings` | GET | CNT-020 | — | M2 | 🟡 |
| [[web/ui-catalog/lms/LMS7]] | web | any | Página do treinamento | `/api/v1/academy/trainings/{id}` | GET | CNT-020 | — | M2 | 🟡 |
| [[web/ui-catalog/lms/LMS8]] | web | admin · enterprise(Pro) · marketing(delegado) | Agenda lives (D-097 — admin + Empresa Pro + agência delegada) | `/academy/lives` GET · `/academy/lives` POST · `/remind` | — | CNT-022 · CNT-024 | CNT-022 D-097 (revoga admin-only) · INV-086 ABAC | M2 | 🟡 |
| [[web/ui-catalog/lms/LMS9]] | web | any | Sala da live (WS) | `/academy/lives/{id}` · `/token` · `/ws/academy/lives/{id}/chat` · `/questions` | GET · POST · WS · POST | CNT-022 · CNT-023 | INV-070 token TTL · INV-093 HMAC | M2 | 🟡 |
| [[web/ui-catalog/lms/LMS10]] | web | any | Replay (30d) | `/academy/lives/{id}/replay` | GET | CNT-023 | INV-056 (lock análogo) | M2 | 🟡 |
| [[web/ui-catalog/lms/LMS11]] | web | syndic/ent/admin | Fórum | `/api/v1/academy/forum/topics` | GET | CNT-021 | ABAC role restriction | M2 | 🟡 |
| [[web/ui-catalog/lms/LMS12]] | web | syndic/ent/admin | Criar tópico | `/api/v1/academy/forum/topics` | POST | CNT-021 | ABAC · INV-086 | M2 | 🟡 |
| [[web/ui-catalog/lms/LMS13]] | web | any | Discussão (comentar) | `/api/v1/academy/forum/topics/{id}` · `/comments` | GET · POST | CNT-021 | — | M2 | 🟡 |
| [[web/ui-catalog/lms/LMS14]] | web | any | Biblioteca (ebooks) | `/api/v1/academy/library` | GET | CNT-025 | INV-069 tier filter | M2 | 🟡 |
| [[web/ui-catalog/lms/LMS15]] | web | any | Página do material | `/api/v1/academy/library/{id}` | GET | CNT-025 | INV-069 | M2 | 🟡 |

Mux integration:

- `POST /api/v1/videos?type=*` (presigned upload)
- `POST /api/v1/webhooks/mux` ⚫ (sem UI — HMAC verify antes de parse — INV-064 · INV-093 · INV-102 body limit 64KB)

---

### 2.5 Assembleia Live (Sprint 7 ✅ backend; Sprint 10 🟡 UI M1)

21 telas web + 5 mobile. Um dos módulos mais complexos — 5 blocos macro.

#### Bloco 1 — Configuração & Criação

| Tela | Stack | Persona | Ação | Endpoint | Método | FR | INV | Sprint | Status |
|---|---|---|---|---|---|---|---|---|---|
| [[web/ui-catalog/assembleia/ASM-DASH]] | web | síndico | Dashboard módulo | `/api/v1/assemblies?condominium={id}` | GET | ASM-001 | INV-075 ≤1 ativa | 7 ✅ | ✅ |
| [[web/ui-catalog/assembleia/ASM-CFG]] | web | síndico | Configurar assembleia | `/api/v1/assemblies` · `/:id` | POST · PATCH · GET | ASM-001 | INV-074 sem edital · INV-118 ≥8d | 7 ✅ | ✅ |
| [[web/ui-catalog/assembleia/ASM-PAUTA]] | web | síndico | Cadastrar pauta (5 tipos) | `/assemblies/:id/agenda` · `/reorder` · `/:item_id` | GET · POST · PATCH · DELETE | ASM-003 · ASM-035 | INV-080 pauta pub = read-only | 7 ✅ | ✅ |
| [[web/ui-catalog/assembleia/ASM-EDITAL]] | web | síndico | Edital (upload/gen/subst) | `/assemblies/:id/edital/upload` · `/generate` · PUT | POST mp · POST · PUT | ASM-002 | INV-118 ≥8d · INV-074 | 7 ✅ | ✅ |
| [[web/ui-catalog/assembleia/ASM-PUB]] | web | síndico | Publicar edital (step-up) | `/assemblies/:id/publish` (header X-Step-Up-Token) · `/notify` | POST · POST | ASM-002 · ASM-005 | INV-074 · INV-110 step-up · INV-118 | 7 ✅ | ✅ |

#### Bloco 2 — Pré-assembleia

| Tela | Stack | Persona | Ação | Endpoint | Método | FR | INV | Sprint | Status |
|---|---|---|---|---|---|---|---|---|---|
| [[web/ui-catalog/assembleia/ASM-CIEN]] / [[mobile/ui-catalog/assembly/ASM-CIEN]] | web · mobile | morador | Ciência pauta (app travado) | `/assemblies/:id/acknowledge` | POST | ASM-006 | INV-086 ABAC · app-gate | 7 ✅ | 🟡 UI mobile M1 |
| [[web/ui-catalog/assembleia/ASM-TERM]] | web | morador | 5 termos obrigatórios (LGPD) | `/assemblies/:id/terms/accept` | POST | ASM-013 · XD-025 | consent versioning · INV-091 | 7 ✅ | 🟡 |
| [[web/ui-catalog/assembleia/ASM-PROC-CAD]] / [[mobile/ui-catalog/assembly/ASM-PROC-CAD]] | web · mobile | morador | Cadastrar procuração | `/assemblies/:id/proxies` | POST | ASM-010 | INV-078 válida +24h · INV-079 revogação 1h | 7 ✅ | 🟡 UI mobile M1 |
| [[web/ui-catalog/assembleia/ASM-PROC-VAL]] | web | administradora | Validar procuração | `/assemblies/:id/proxies/:id/validate` | POST | ASM-010 | INV-077 proxy≠titular | 7 ✅ | ✅ |
| [[web/ui-catalog/assembleia/ASM-ENQP]] | web | morador | Enquete preliminar (s/v jurídico) | `/assemblies/:id/polls` · `/vote` | GET · POST | ASM-009 | — | 7 ✅ | ✅ |
| [[web/ui-catalog/assembleia/ASM-FRAC]] | web | administradora | Upload fração ideal | `/condominiums/:id/units/fractions/bulk` | POST mp | ASM-007 · INS-005 | INV-021 Σ=100 · INV-022 0<f≤100 · INV-023 NUMERIC(5,4) · INV-082 | 7 ✅ | ✅ |
| [[web/ui-catalog/assembleia/ASM-INAD]] | web | administradora | Upload inadimplência (cutoff 1h) | `/condominiums/:id/defaulters/upload` | POST mp | ASM-008 | cutoff validation | 7 ✅ | 🟡 |
| [[web/ui-catalog/assembleia/ASM-SIM]] | web | síndico | Simulador quórum | `/assemblies/:id/quorum/simulate` | GET | ASM-011 | INV-073 Σ fração | 7 ✅ | ✅ |

#### Bloco 3 — Ao vivo

| Tela | Stack | Persona | Ação | Endpoint | Método | FR | INV | Sprint | Status |
|---|---|---|---|---|---|---|---|---|---|
| [[web/ui-catalog/assembleia/ASM-CHKIN]] / [[mobile/ui-catalog/assembly/ASM-CHKIN]] | web · mobile | morador/admin | Check-in (3 modalidades) | `/assemblies/:id/checkin` · `/checkin/assisted` · `/attendance` | POST ×2 · GET | ASM-012 | INV-073 Σ fração presente | 7 ✅ | ✅ backend · 🟡 mobile |
| [[web/ui-catalog/assembleia/ASM-PAINEL-SIND]] | web | síndico | Painel presidente | `/assemblies/:id/start` (step-up) · WS control | POST · WS | ASM-017 · ASM-014 | INV-081 síndico não vota · INV-110 step-up · INV-084 saga | 7 ✅ | ✅ |
| [[web/ui-catalog/assembleia/ASM-TELAO]] | web | público | Telão (2 áreas) | `wss://assemblies/:id/screen` | WS | ASM-018 | <200ms P95 (INV-ASM-live) | 7 ✅ | 🟡 UX M1 |
| [[web/ui-catalog/assembleia/ASM-VOTO]] / [[mobile/ui-catalog/assembly/ASM-VOTO]] | web · mobile | morador | Votar (Sim/Não/Abst ou 1-5) | `/assemblies/:id/agenda/:item/vote` · `/votes` | POST · GET | ASM-014 · ASM-015 | INV-071 voto único · INV-073 frac · INV-083 snapshot · INV-076 timeout=absent · INV-081 | 7 ✅ | 🟡 mobile M1 |
| [[web/ui-catalog/assembleia/ASM-FILA-FALA]] / [[mobile/ui-catalog/assembly/ASM-FILA-FALA]] | web · mobile | morador | Fila de fala 2/3min | `/assemblies/:id/speakers/queue` · `/speakers/grant` | POST · POST | ASM-019 | — | 7 ✅ | 🟡 |
| [[web/ui-catalog/assembleia/ASM-HOML]] | web | síndico | Homologação (step-up) | `/assemblies/:id/agenda/:item/homologate` (X-Step-Up-Token) | POST | ASM-027 | INV-085 homologação separada · INV-072 ata imutável · INV-110 step-up | 7 ✅ | ✅ |

#### Bloco 4 — Encerramento

| Tela | Stack | Persona | Ação | Endpoint | Método | FR | INV | Sprint | Status |
|---|---|---|---|---|---|---|---|---|---|
| [[web/ui-catalog/assembleia/ASM-ENCER]] | web | síndico | Encerrar + gerar ata (step-up) | `/assemblies/:id/close` · `/close/preflight` (X-Step-Up-Token) | POST · GET | ASM-023 · ASM-024 | INV-072 ata imutável · INV-119 DB-level · INV-110 | 7 ✅ | ✅ |

#### Bloco 5 — Pós-assembleia

| Tela | Stack | Persona | Ação | Endpoint | Método | FR | INV | Sprint | Status |
|---|---|---|---|---|---|---|---|---|---|
| [[web/ui-catalog/assembleia/ASM-REL]] | web | síndico | 6 relatórios + score transparência | `/assemblies/:id/reports?type=...` · `/transparency-score` | GET ×2 | ASM-026 · ASM-028 · ASM-030 | INV-072 ata imutável (source) | 7 ✅ | ✅ |

---

### 2.6 Compliance (Sprint 3 ✅ backend audit; Sprint 10 🟡 UI)

| Tela | Stack | Persona | Ação | Endpoint | Método | FR | INV | Sprint | Status |
|---|---|---|---|---|---|---|---|---|---|
| [[web/ui-catalog/compliance/C1]] · [[web/ui-catalog/sindico/S28]] | web | síndico | Painel compliance | `/api/v1/compliance/dashboard` · `/pendencies` · `/condominiums/{id}/compliance/summary` | GET ×3 | XD-025..XD-028 · INS-033 | INV-034 anual · 4 gatilhos | 3 · 10 | 🟡 |
| [[web/ui-catalog/compliance/C2]] · [[web/ui-catalog/sindico/S29]] · [[web/ui-catalog/sindico/S30]] | web | síndico | Declaração anual + termo responsab | `/compliance/declaration/template` · `/status` · `/compliance/responsibility-term/current` · `/condominiums/{id}/compliance/responsibility-term/sign` · `/annual-declaration` | GET ×3 · POST ×2 | INS-033 · XD-025 | INV-034 UNIQUE ano · hash SHA-256 termo · INV-024 audit | 3 ✅ · 10 | 🟡 UI |
| [[web/ui-catalog/compliance/C3]] | web | síndico | Assinaturas obrigatórias usuários | `/api/v1/compliance/signatures/pending` · `/:id/sign` | GET · POST | XD-025 · IDN-034 | INV-024 audit · hash termo | 3 ✅ | 🟡 UI |
| [[web/ui-catalog/compliance/C4]] | web | síndico | Matriz conflito interesse | `/compliance/conflicts` · `/declare` | GET · POST | IDN-034 · XD-027 | INV-035 obrigatório onb | 3 ✅ | 🟡 UI |
| [[web/ui-catalog/compliance/C5]] · [[web/ui-catalog/sindico/S31]] | web | síndico | Trilha auditoria visível | `/compliance/audit-trail` · `/filters` · `/export` · `/condominiums/{id}/compliance/audit` · `/{entry_id}` · `/export` | GET ×3 + POST | XD-009 · XD-010 | INV-033 append-only · INV-090 · INV-120 DB-level · INV-091 retenção 5a | 3 ✅ · 10 | ✅ backend · 🟡 UI |
| [[web/ui-catalog/compliance/C6]] | web | síndico | Imutabilidade e adendos | `/compliance/amendments` | GET · POST | INS-016 · COM-025 | INV-032 adendo · INV-050 · INV-119 | 3 ✅ | 🟡 UI |
| [[web/ui-catalog/compliance/C7]] | web | síndico | Mapa de risco consolidado | `/compliance/risk-map` | GET | XD-029 | — | 3 ✅ | 🟡 UI |
| [[web/ui-catalog/compliance/C8]] | web | síndico | Conformidade contratual | `/compliance/contracts` | GET · POST | XD-025 · COM-019 | INV-040 deal imutável | 3 ✅ | 🟡 UI |
| [[web/ui-catalog/compliance/C9]] | web | síndico | Registro LGPD central | `/compliance/lgpd-logs` · `/{id}` · `/export` | GET ×2 · POST | XD-010 | INV-091 retenção 5a · INV-048 reveal log · INV-109 HMAC keyed | 3 ✅ | 🟡 UI |
| [[web/ui-catalog/compliance/C10]] | web | síndico (principal only) | Score governança (privado) | `/api/v1/condominiums/{id}/governance-score/internal` | GET | INS-032 | INV-044 anonimato? C10 privado `principal` | 3 🟡 backend regra ok · endpoint gap | 🔴 endpoint ausente |
| [[web/ui-catalog/compliance/C11]] | web | síndico | Dossiê exportável PDF | `/compliance/dossier?year=...` | POST | XD-030 · INS-034 | INV-091 retenção · 4 gatilhos | 3 ✅ · 10 UI | 🟡 UI |
| [[web/ui-catalog/empresa/E8]] | web | empresa | Termo execução (LGPD log versionado) | `/commercial/closed-deals/:id/company-sign` · `/terms/execution/current` · `/signature-status` | POST · GET · GET | COM-019 · XD-025 | INV-040 imutável pós dupla assinatura · INV-109 | 4 ✅ | ✅ |
| [[web/ui-catalog/empresa/E15]] | web | empresa | Compliance empresarial anual | `/compliance/enterprise/annual` | GET · POST | COM-002 · XD-026 | INV-034 anual | 3 ✅ | 🟡 UI |

**4 gatilhos bloqueantes** (mensagem padrão: "Para prosseguir, finalize as pendências de compliance."):

1. Encerrar mandato (S2 end-mandate)
2. Gerar relatório final
3. Marcar negócio fechado (opcional)
4. Exportar dossiê (C11)

---

### 2.7 Vizinhança (M2 backlog; spec pronta)

29 telas · 4 jornadas (VS síndico 10 · VE empresa 6 · VM morador 7 · VZ parceiro 6).

| Tela | Stack | Persona | Ação principal | Endpoint | Método | FR | INV | Sprint | Status |
|---|---|---|---|---|---|---|---|---|---|
| [[web/ui-catalog/vizinhanca/VS1]] | web | síndico | Painel vizinhança | `/api/v1/neighborhood/syndic/dashboard` | GET | COM-030 · COM-011 | INV-089 · actor separado | M2 | 🟡 |
| [[web/ui-catalog/vizinhanca/VS2]] | web | síndico | Explorar parceiros | `/api/v1/neighborhood/partners?scope=neighborhood` | GET | COM-030 | — | M2 | 🟡 |
| [[web/ui-catalog/vizinhanca/VS3]] | web | síndico | Detalhe parceiro | `/api/v1/neighborhood/partners/{id}` | GET | COM-030 | — | M2 | 🟡 |
| [[web/ui-catalog/vizinhanca/VS4]] | web | síndico | Detalhe promoção | `/neighborhood/promotions/{id}` | GET | COM-030 | ABAC exclusive-view | M2 | 🟡 |
| [[web/ui-catalog/vizinhanca/VS5]] | web | síndico | Ativar promoção (cupom) | `/neighborhood/promotions/{id}/coupon` · `/coupons/{code}` | POST · GET | COM-030 | INV-051 cooldown 4h · INV-052 formato · INV-053 palavra ≤5 | M2 | 🟡 |
| [[web/ui-catalog/vizinhanca/VS6]] | web | síndico | Indicar parceiro local | `/neighborhood/partners/invite` | POST | COM-011 | IDN-025 invite · Decision 12 | M2 | 🟡 |
| [[web/ui-catalog/vizinhanca/VS7]] | web | síndico | Criar benefício exclusivo | `/neighborhood/partners?available_for_exclusive=true` · `/exclusive-benefits` | GET · POST | COM-030 | ABAC `commercial.exclusive_benefit.view` tenant-match | M2 | 🟡 |
| [[web/ui-catalog/vizinhanca/VS8]] | web | síndico | Benefícios do condomínio | `/neighborhood/exclusive-benefits?condominium={id}` | GET | COM-030 | tenant visibility | M2 | 🟡 |
| [[web/ui-catalog/vizinhanca/VS9]] | web | síndico | Parceiros indicados (status) | `/neighborhood/partners/invited?syndic=me` | GET | COM-011 | — | M2 | 🟡 |
| [[web/ui-catalog/vizinhanca/VS10]] | web | síndico | Histórico ativações | `/neighborhood/coupons?issuer=me` | GET | COM-030 | — | M2 | 🟡 |
| [[web/ui-catalog/vizinhanca/VE1]]..[[web/ui-catalog/vizinhanca/VE6]] | web | empresa | Explorar abertas do bairro | `/neighborhood/promotions?scope=aberta_bairro` etc. | GET · POST | COM-030 | INV-067 (N/A) · sem acesso exclusive | M2 | 🟡 |
| [[web/ui-catalog/vizinhanca/VM1]] / [[mobile/ui-catalog/vizinhanca/VM1]] | web · mobile | morador | Feed bairro | `/api/v1/neighborhood/resident/feed` | GET | COM-030 | ABAC · INV-051 | M2 | 🟡 |
| [[web/ui-catalog/vizinhanca/VM3]] / [[mobile/ui-catalog/vizinhanca/VM3]] | web · mobile | morador | Detalhe promoção | `/neighborhood/promotions/{id}?audience=resident` | GET | COM-030 | ABAC `commercial.exclusive_benefit.view` tenant-match | M2 | 🟡 |
| [[web/ui-catalog/vizinhanca/VM4]] / [[mobile/ui-catalog/vizinhanca/VM4]] | web · mobile | morador | Ativar (cupom) | `/neighborhood/promotions/{id}/coupon` (actor=resident) | POST | COM-030 | INV-051 lock 4h · INV-052 formato | M2 | 🟡 |
| [[web/ui-catalog/vizinhanca/VM5]] / [[mobile/ui-catalog/vizinhanca/VM5]] | web · mobile | morador | Benefícios do condomínio | `/neighborhood/exclusive-benefits?audience=resident` | GET | COM-030 | tenant visibility | M2 | 🟡 |
| [[web/ui-catalog/vizinhanca/VZ1]]..[[web/ui-catalog/vizinhanca/VZ6]] | web | parceiro | Cadastro + perfil + criar promoção + métricas | `/neighborhood/partners/register` · `/promotions` · `/partners/me/metrics` | POST · POST · GET | COM-030 · IDN-025 | INV-067 parceiro sem LMS · Decision 12 | M2 | 🟡 |

---

### 2.8 Banco de Talentos (M2 backlog)

| Tela | Stack | Persona | Ação | Endpoint | Método | FR | INV | Sprint | Status |
|---|---|---|---|---|---|---|---|---|---|
| [[web/ui-catalog/banco-talentos/BT01]] | web | morador | Boas-vindas + status curriculum | `/api/v1/talents/curriculum/me` | GET | CNT-030 · COM-029 | INV-BT-05 único por morador | M2 | 🟡 |
| [[web/ui-catalog/banco-talentos/BT02]] | web | morador | Vídeo apresentação 90s | `/api/v1/videos?type=resume` · `/talents/curriculum/me/video` · `/webhooks/mux` ⚫ | POST · GET | CNT-030 · CNT-031 | INV-BT-01 ≤90s · INV-056 lock 90d · INV-064 HMAC | M2 | 🟡 |
| [[web/ui-catalog/banco-talentos/BT03]] | web | morador | Perfil profissional (6 perg abertas) | `/talents/curriculum/me` | PATCH | COM-029 | INV-BT-05 | M2 | 🟡 |
| [[web/ui-catalog/banco-talentos/BT04]] | web | morador | Áreas interesse (18 cond) | `/talents/curriculum/me/interests` | PATCH | COM-029 | — | M2 | 🟡 |
| [[web/ui-catalog/banco-talentos/BT05]] | web | morador | Dados pessoais | `/talents/curriculum/me/personal` | PATCH | COM-029 · IDN-024 | PII masking · LGPD | M2 | 🟡 |
| [[web/ui-catalog/banco-talentos/BT06]] | web | morador | Disponibilidade | `/talents/curriculum/me/availability` | PATCH | COM-029 | — | M2 | 🟡 |
| [[web/ui-catalog/banco-talentos/BT07]] | web | morador | Pretensão salarial | `/talents/curriculum/me/salary` | PATCH | COM-029 | — | M2 | 🟡 |
| [[web/ui-catalog/banco-talentos/BT08]] | web | morador | Experiência (5 vínculos) | `/talents/curriculum/me/experience` | PUT | COM-029 | D-099 (5 max — revoga D-064) | M2 | 🟡 |
| [[web/ui-catalog/banco-talentos/BT09]] | web | morador | Formação + CNH + NR | `/talents/curriculum/me/education` | PUT | COM-029 | — | M2 | 🟡 |
| [[web/ui-catalog/banco-talentos/BT10]] | web | morador | Consentimento LGPD | `/talents/lgpd/template` · `/talents/curriculum/me/lgpd-consent` | GET · POST | IDN-024 · XD-010 | INV-BT-04 consent · INV-091 retenção | M2 | 🟡 |
| [[web/ui-catalog/banco-talentos/BT11]] | web | morador | Confirmação (submit) | `/talents/curriculum/me/submit` | POST | COM-029 | state machine draft→submitted | M2 | 🟡 |

Empresa busca (Plus/Pro — fora desta entrega, 🟡 spec):

- `GET /api/v1/talents/search?filters=...` — logada em `curriculum_search_logs` (INV-089 privacidade cross-tenant)

---

### 2.9 Marketing & Agências (Sprint 5 ✅ backend; 🟡 UI)

| Tela | Stack | Persona | Ação | Endpoint | Método | FR | INV | Sprint | Status |
|---|---|---|---|---|---|---|---|---|---|
| [[web/ui-catalog/marketing/MK1]] | web | agência | Painel agência | `/api/v1/marketing/dashboard` · `/notifications` | GET ×2 | CNT-014 · IDN-025 | INV-049 zero-access deals/CM | 5 ✅ | 🟡 UI |
| [[web/ui-catalog/marketing/MK2]] | web | agência | Minhas empresas (vínculos) | `/api/v1/marketing/empresas` | GET | CNT-014 | INV-066 escopo | 5 ✅ | 🟡 |
| [[web/ui-catalog/marketing/MK3]] | web | agência | Dashboard empresa ativa (context) | `/marketing/empresas/:id/context` · `/content/videos?empresa_id=:id&limit=5` | GET ×2 | CNT-014 | INV-049 · INV-066 | 5 ✅ | 🟡 |
| [[web/ui-catalog/marketing/MK4]] | web | agência | Novo vídeo (por empresa) | `/content/videos?empresa_id=:id` · `/api/v1/videos?type=institutional` | POST ×2 | CNT-001 · CNT-014 | INV-064 HMAC · INV-056 lock | 5 ✅ | 🟡 |
| [[web/ui-catalog/marketing/MK5]] | web | agência | Lista vídeos (edit/oculta/restora) | `/content/videos?empresa_id=:id` · `/:id` · `/:id/hide` · `/:id/restore` | GET · PUT · POST ×2 | CNT-003 · CNT-014 | INV-056 lock · INV-057 bypass | 5 ✅ | 🟡 |
| [[web/ui-catalog/marketing/MK6]] | web | agência | Analytics vídeos | `/content/videos/analytics?empresa_id=:id` | GET | CNT-034 | INV-060 tenant · INV-012 metrics privacy | 5 ✅ | 🟡 |
| [[web/ui-catalog/marketing/MK7]] | web | agência | Dashboard consolidado agência | `/marketing/dashboard/consolidated` · `/export` | GET · POST | CNT-014 · XD-021 | INV-060 | 5 ✅ | 🟡 |
| [[web/ui-catalog/marketing/MK8]] | web | agência | Marketing Link recebidos | `/commercial/marketing-link/received` · `/:id/attended` · `/:id/view` | GET · POST ×2 | COM-011 · CNT-014 | INV-049 · anti-prospecção | 5 ✅ | 🟡 |
| [[web/ui-catalog/empresa/E14]] | web | empresa | Convidar agência | (ver §2.2) | — | — | — | 5 ✅ | ✅ |
| [[web/ui-catalog/empresa/E16]] | web | empresa | Marketing Link (intenção) | (ver §2.2) | — | — | — | 5 ✅ | ✅ |

---

### 2.10 Onboarding (Sprint 1/2 ✅ backend Zitadel; Sprint 10 🟡 UI M1)

25 telas (2 comuns + 4 morador + 6 síndico + 7 empresa + 4 parceiro + 4 FIN). Todos usam `/api/v1/onboarding/draft` com auto-save Redis TTL 30d.

| Tela | Stack | Persona | Ação | Endpoint | Método | FR | INV | Sprint | Status |
|---|---|---|---|---|---|---|---|---|---|
| [[web/ui-catalog/onboarding/ONB-00-boas-vindas]] | web | any | Selecionar perfil | `/api/v1/onboarding/draft` | POST | IDN-001 | — | 1 ✅ | 🟡 UI M1 |
| [[web/ui-catalog/onboarding/ONB-01-identificacao-inicial]] | web | any | Nome + e-mail | `/api/v1/onboarding/draft` | POST | IDN-001 · IDN-002 | token verif Zitadel 24h | 1 ✅ | 🟡 |
| [[web/ui-catalog/onboarding/ONB-M2]] | web | morador | Identidade (foto, CPF) | `/onboarding/draft` · `/media/avatars/upload` · `/identity/validate-cpf` | PATCH · POST mp · POST | IDN-017 · IDN-018 | INV-002 1 CPF=1 User · INV-029 imutável | 1 ✅ | 🟡 |
| [[web/ui-catalog/onboarding/ONB-M3]] | web | morador | Vínculo condomínio (código 8) | `/condominiums?public_id={code}` · `/onboarding/draft` | GET · PATCH | INS-005 · INS-007 · IDN-018 | INV-030 public_id | 2 ✅ | 🟡 |
| [[web/ui-catalog/onboarding/ONB-M4]] | web | morador | Aceites (3) | `/onboarding/draft` | PATCH | IDN-024 | consent versioning | 1 ✅ | 🟡 |
| [[web/ui-catalog/onboarding/ONB-MFIN]] | web | morador | Confirmação | `/onboarding/finalize` | POST | IDN-018 · IDN-012 | INV-006 FK · INV-004 role enforced | 1 ✅ | 🟡 |
| [[web/ui-catalog/onboarding/ONB-S2]] | web | síndico | Identidade gestão | `/onboarding/draft` | PATCH | IDN-017 | INV-002 · INV-029 | 1 ✅ | 🟡 |
| [[web/ui-catalog/onboarding/ONB-S3]] | web | síndico | Atuação (tipo, qtd condo) | `/onboarding/draft` | PATCH | IDN-017 | INV-027 ≤15 | 1 ✅ | 🟡 |
| [[web/ui-catalog/onboarding/ONB-S4]] | web | síndico | Governança (15 markers) | `/onboarding/draft` | PATCH | INS-040 | — | 1 ✅ | 🟡 |
| [[web/ui-catalog/onboarding/ONB-S5]] | web | síndico N2/N3 | Presença (mini bio + vídeo) | `/onboarding/draft` · `/videos?type=institutional` | PATCH · POST | CNT-001 · INS-039 | INV-056 lock 90d | 5 ✅ | 🟡 |
| [[web/ui-catalog/onboarding/ONB-S6]] | web | síndico | Aceites (3) | `/onboarding/draft` | PATCH | IDN-024 · IDN-034 | INV-035 conflito | 1 ✅ | 🟡 |
| [[web/ui-catalog/onboarding/ONB-SFIN]] | web | síndico | Confirmação + declaração anual | `/onboarding/finalize` | POST | IDN-018 · INS-033 | INV-034 decl anual | 1 · 3 ✅ | 🟡 |
| [[web/ui-catalog/onboarding/ONB-E2]] | web | empresa | Identidade institucional CNPJ | `/identity/validate-cnpj` · `/onboarding/draft` | POST · PATCH | COM-001 · COM-005 | INV-003 1 CNPJ=1 User · INV-029 | 1 · 4 ✅ | 🟡 |
| [[web/ui-catalog/onboarding/ONB-E3]] | web | empresa | Contatos e endereço | `/onboarding/draft` | PATCH | COM-001 | — | 1 ✅ | 🟡 |
| [[web/ui-catalog/onboarding/ONB-E4]] | web | empresa | Atuação (cidades, categorias) | `/onboarding/draft` | PATCH | COM-001 | — | 1 ✅ | 🟡 |
| [[web/ui-catalog/onboarding/ONB-E5]] | web | empresa | Conformidade (25 markers) | `/onboarding/draft` | PATCH | COM-002 | — | 4 ✅ | 🟡 |
| [[web/ui-catalog/onboarding/ONB-E6]] | web | empresa | Conteúdo institucional | `/onboarding/draft` · `/videos?type=institutional` | PATCH · POST | COM-003 · CNT-001 | INV-056 · lint comercial | 5 ✅ | 🟡 |
| [[web/ui-catalog/onboarding/ONB-E7]] | web | empresa | Aceites (2) | `/onboarding/draft` | PATCH | IDN-024 | consent | 1 ✅ | 🟡 |
| [[web/ui-catalog/onboarding/ONB-EFIN]] | web | empresa | Confirmação | `/onboarding/finalize` | POST | IDN-018 · COM-001 | INV-003 | 1 · 4 ✅ | 🟡 |
| [[web/ui-catalog/onboarding/ONB-P2]] | web | parceiro | Identidade (CPF ou CNPJ D-064) | `/onboarding/draft` | PATCH | IDN-001 · COM-030 | Decision 12 | 1 ✅ | 🟡 |
| [[web/ui-catalog/onboarding/ONB-P3]] | web | parceiro | Contato + visual (3 fotos) | `/onboarding/draft` · `/media/upload` | PATCH · POST | COM-030 | — | 1 ✅ | 🟡 |
| [[web/ui-catalog/onboarding/ONB-P4]] | web | parceiro | Aceites (2) | `/onboarding/draft` | PATCH | IDN-024 | — | 1 ✅ | 🟡 |
| [[web/ui-catalog/onboarding/ONB-PFIN]] | web | parceiro | Confirmação | `/onboarding/finalize` | POST | IDN-018 · COM-030 | — | 1 ✅ | 🟡 |

Mobile onboarding (21 telas equivalentes) — idênticos endpoints, diferença apenas na UI.

---

### 2.11 Admin (gap documentado — fora de M1)

| Tela | Stack | Persona | Ação | Endpoint | Método | FR | INV | Sprint | Status |
|---|---|---|---|---|---|---|---|---|---|
| [[web/ui-catalog/admin/GAP-admin-master-sindico]] | web | admin | Painel admin (moderação vídeo/BT/LMS) | `/api/v1/admin/*` | TBD | CNT-027..CNT-029 · D-058 | INV-057 bypass audit · INV-088 · INV-110 MFA admin | M2 · M3 | 🔴 gap |

---

## 3. Matriz inversa — endpoint × telas consumidoras

Referência para backend saber quais UIs dependem de um endpoint.

### 3.1 Institutional BC

| Endpoint | Método | Telas consumidoras | Sub-produto | INV |
|---|---|---|---|---|
| `/api/v1/condominiums` | GET · POST | S2 · S3 · M3 · ONB-M3 · ONB-SFIN · VZ3 | Governança · Onboarding · Vizinhança | INV-027 · INV-030 |
| `/api/v1/condominiums/{id}` | GET · PATCH | S5 · S6 · ONB-M3 | Governança | INV-030 · INV-089 |
| `/api/v1/condominiums/{id}/mandate` | POST | S4 · ONB-SFIN | Governança | INV-028 |
| `/api/v1/condominiums/{id}/end-mandate` | POST | S2 | Governança · Compliance (gate 1) | INV-034 |
| `/api/v1/condominiums/{id}/dashboard/summary` | GET | S6 · S27 · E10 | Governança | INV-089 |
| `/api/v1/condominiums/{id}/master-plan/activities` | GET · POST | S7 · S8 · S9 · M8 | Governança | INV-031 |
| `/api/v1/condominiums/{id}/master-plan/activities/{aid}` | GET · PATCH | S10 · M9 | Governança | INV-031 · INV-024 |
| `/api/v1/condominiums/{id}/timeline` | GET · POST | S11 · S12 · M6 · M7 | Governança · Morador | INV-024 · INV-119 |
| `/api/v1/institutional/timeline/{id}` | GET · POST(/read) | S11 · S13 · S14 · M6 · M7 | Governança · Morador | INV-024 |
| `/api/v1/institutional/timeline/{id}/amend` | POST | S13 | Governança · Compliance (C6) | INV-032 · INV-050 |
| `/api/v1/condominiums/{id}/events` | GET · POST | S15 · S16 · S17 · M10 | Governança | — |
| `/api/v1/condominiums/{id}/announcements` | GET · POST | S23 · S24 · S25 · M11 | Governança | INV-025 |
| `/api/v1/condominiums/{id}/validations` | GET · POST | S26 · S21 · S22 | Governança · Reputação | INV-089 |
| `/api/v1/institutional/memberships/me/*` | GET · PATCH | S2 · M1 · M2 · M13 | Governança · Onboarding | INV-026 |
| `/api/v1/institutional/memberships` | POST · DELETE | M4 · M14 · M15 · ONB-M3 | Morador · Onboarding | INV-029 |
| `/api/v1/institutional/assessments` | GET · POST | M12 | Reputação · Governança | INS-031 |
| `/api/v1/institutional/dashboard/resident` | GET | M1 | Morador | INV-089 |

### 3.2 Commercial BC

| Endpoint | Método | Telas consumidoras | Sub-produto | INV |
|---|---|---|---|---|
| `/api/v1/commercial/connect-me` (POST) · `?sender/receiver` (GET) | POST · GET | S18 · S19 · S20 · E2 · E3 · E4 | Connect Me | INV-037 · INV-038 · INV-045 · INV-048 |
| `/api/v1/commercial/connect-me/:id/respond` | POST | S20 · E3 | Connect Me | INV-047 |
| `/api/v1/commercial/connect-me/:id/interested` | POST | E4 | Connect Me | INV-048 reveal-log |
| `/api/v1/commercial/connect-me/me/summary` | GET | S18 | Connect Me | INV-045 · INV-046 |
| `/api/v1/commercial/proposals` · `/:id/draft` · `/:id/submit` | POST · PUT · POST | E4 · E5 | Connect Me | INV-038 · INV-039 |
| `/api/v1/commercial/closed-deals/*` | GET · POST | E7 · E8 | Connect Me · Compliance | INV-040 · INV-041 |
| `/api/v1/commercial/proposals?sender=me` | GET | E5 | Connect Me | — |
| `/api/v1/commercial/marketing-link` · `/received` · `/:id/attended` · `/view` | POST · GET · POST | E16 · MK8 | Connect Me · Marketing | INV-049 |
| `/api/v1/commercial/evaluations/*` | GET · POST · PUT | S22 · E11 | Reputação | INV-043 · INV-044 |
| `/api/v1/commercial/reputation/*` | GET | E10 | Reputação | INV-036 |
| `/api/v1/commercial/executions` | GET · POST | S21 · S22 · E9 | Reputação | INV-041 · INV-089 |
| `/api/v1/commercial/technical-activities` | POST | E6 | Reputação | — |
| `/api/v1/neighborhood/promotions/*` | GET · POST | VS4 · VS5 · VM3 · VM4 · VE4 · VE5 · VZ3 · VZ4 | Vizinhança | INV-051 · INV-052 · INV-053 |
| `/api/v1/neighborhood/partners/*` | GET · POST | VS2 · VS3 · VS6 · VS9 · VE2 · VE3 · VZ1 · VZ2 | Vizinhança | Decision 12 · INV-067 |
| `/api/v1/neighborhood/coupons/{code}` | GET | VS5 · VM4 | Vizinhança | INV-052 |
| `/api/v1/neighborhood/exclusive-benefits` | GET · POST | VS7 · VS8 · VM5 · VM6 | Vizinhança | ABAC `exclusive_benefit.view` tenant match |

### 3.3 Content BC (inclui LMS)

| Endpoint | Método | Telas | Sub-produto | INV |
|---|---|---|---|---|
| `/api/v1/videos?type=...` | POST (presigned) | BT02 · MK4 · ONB-S5 · ONB-E6 | Vídeo · BT · Onb | INV-056 · INV-064 |
| `/api/v1/content/videos` | GET · PUT | E12 · MK3 · MK5 · MK6 | Vídeo · Marketing | INV-056 · INV-060 · INV-057 bypass |
| `/api/v1/content/videos/:id/hide` · `/restore` | POST | MK5 | Marketing | INV-057 audit |
| `/api/v1/academy/dashboard` · `/paths/recommended` | GET | LMS1 | LMS | INV-067 |
| `/api/v1/academy/courses/*` | GET · POST | LMS2 · LMS3 | LMS | INV-069 tier · ABAC |
| `/api/v1/academy/lessons/*` | GET · POST | LMS4 | LMS | INV-070 signed URL · INV-060 |
| `/api/v1/academy/certificates/*` · `/certificates/verify/{serial}` (público) | GET | LMS5 | LMS | INV-068 auto |
| `/api/v1/academy/lives/*` | GET · POST | LMS8 · LMS9 · LMS10 | LMS | CNT-022 D-097 (admin + Empresa Pro + agência delegada — revoga admin-only D-063/D-076) · INV-070 |
| `/ws/academy/lives/{id}/chat` | WS | LMS9 | LMS | INV-035 WS session · INV-110 ticket |
| `/api/v1/academy/forum/topics/*` | GET · POST | LMS11 · LMS12 · LMS13 | LMS | ABAC role |
| `/api/v1/academy/library*` | GET | LMS14 · LMS15 | LMS | INV-069 tier |
| `/api/v1/talents/curriculum/me*` | GET · PATCH · PUT · POST | BT01..BT11 | BT | INV-BT-01/04/05 · INV-BT lock 90d |
| `/api/v1/talents/lgpd/template` | GET | BT10 | BT | INV-BT-04 · INV-091 |
| `/api/v1/talents/search` | GET | (Empresa Plus/Pro UI TBD) | BT | INV-089 privacy cross-tenant |

### 3.4 Assembly BC

| Endpoint | Método | Telas | Sub-produto | INV |
|---|---|---|---|---|
| `/api/v1/assemblies` | GET · POST | ASM-DASH · ASM-CFG | Assembleia | INV-075 · INV-118 |
| `/api/v1/assemblies/:id` | GET · PATCH | ASM-CFG · ASM-PAUTA | Assembleia | — |
| `/api/v1/assemblies/:id/agenda` · `/reorder` · `/:item_id` | GET · POST · PATCH · DELETE | ASM-PAUTA | Assembleia | INV-080 |
| `/api/v1/assemblies/:id/edital/*` | POST · PUT | ASM-EDITAL | Assembleia | INV-074 · INV-118 |
| `/api/v1/assemblies/:id/publish` (X-Step-Up) · `/notify` | POST | ASM-PUB | Assembleia | INV-074 · INV-110 · INV-118 |
| `/api/v1/assemblies/:id/acknowledge` | POST | ASM-CIEN | Assembleia | app-gate |
| `/api/v1/assemblies/:id/terms/accept` | POST | ASM-TERM | Assembleia · Compliance | LGPD · consent |
| `/api/v1/assemblies/:id/proxies` · `/validate` | POST · POST | ASM-PROC-CAD · ASM-PROC-VAL | Assembleia | INV-077 · INV-078 · INV-079 |
| `/api/v1/assemblies/:id/polls` | GET · POST | ASM-ENQP | Assembleia | — |
| `/api/v1/condominiums/:id/units/fractions/bulk` | POST mp | ASM-FRAC · ONB-S4 (admin) | Assembleia · Gov | INV-021 · INV-022 · INV-023 · INV-082 |
| `/api/v1/assemblies/:id/quorum/simulate` | GET | ASM-SIM | Assembleia | INV-073 |
| `/api/v1/assemblies/:id/checkin` · `/checkin/assisted` · `/attendance` | POST ×2 · GET | ASM-CHKIN | Assembleia | INV-073 |
| `/api/v1/assemblies/:id/start` (X-Step-Up) | POST | ASM-PAINEL-SIND | Assembleia | INV-081 · INV-110 |
| `/api/v1/assemblies/:id/agenda/:item/vote` · `/votes` | POST · GET | ASM-VOTO | Assembleia | INV-071 · INV-073 · INV-076 · INV-083 |
| `/api/v1/assemblies/:id/speakers/*` | POST | ASM-FILA-FALA | Assembleia | — |
| `/api/v1/assemblies/:id/agenda/:item/homologate` (X-Step-Up) | POST | ASM-HOML | Assembleia | INV-072 · INV-085 · INV-110 |
| `/api/v1/assemblies/:id/close` · `/preflight` (X-Step-Up) | POST · GET | ASM-ENCER | Assembleia | INV-072 · INV-110 · INV-119 |
| `/api/v1/assemblies/:id/reports` · `/transparency-score` | GET | ASM-REL | Assembleia | INV-072 |
| `wss://assemblies/:id/screen` | WS | ASM-TELAO | Assembleia | <200ms P95 |

### 3.5 Identity / Auth / Billing BC

| Endpoint | Método | Telas | Sub-produto | INV |
|---|---|---|---|---|
| `/api/v1/auth/login` · `/logout` | GET · POST | S1 · M1 · ONB-00 | Identity | INV-005 1-device · INV-100 cookie · INV-106 regen |
| `/api/v1/auth/resend-verification` | POST | ONB-01 | Identity | IDN-002 · rate limit |
| `/api/v1/onboarding/draft` · `/finalize` | POST · PATCH | ONB-* todas | Onboarding | Redis TTL 30d |
| `/api/v1/identity/validate-cpf` · `/validate-cnpj` | POST | ONB-M2 · ONB-S2 · ONB-E2 | Onboarding | INV-002 · INV-003 · INV-092 PII masked |
| `/api/v1/me/*` (GET · PATCH · DELETE) | ~ | perfil (TBD) | Identity | INV-107 DTO estrito · INV-101 ownership · INV-110 step-up |
| `/api/v1/me/devices/*` | GET · DELETE | device management TBD | Identity | INV-005 · INV-101 · INV-110 |
| `/api/v1/me/export` | GET (step-up) | LGPD export TBD | Identity | IDN-023 · INV-110 |
| `/api/v1/billing/*` (Stripe) | GET · POST | config de billing UI (TBD) | Billing | INV-011 · INV-012 · INV-013 · INV-014 · INV-018 · INV-019 |
| `/api/v1/enterprise/*` | GET · POST | E1 · E14 · E16 | Empresa | INV-049 · INV-066 |
| `/api/v1/marketing/*` | GET · POST | MK1..MK8 · E14 | Marketing | INV-049 · INV-066 |

### 3.6 Compliance / Audit / Cross-domain

| Endpoint | Método | Telas | Sub-produto | INV |
|---|---|---|---|---|
| `/api/v1/compliance/dashboard` · `/pendencies` | GET | C1 · S28 | Compliance | INV-034 · 4 gatilhos |
| `/api/v1/compliance/declaration/*` · `/responsibility-term/*` | GET · POST | C2 · S29 · S30 | Compliance | INV-034 · INV-024 audit |
| `/api/v1/compliance/signatures/*` | GET · POST | C3 | Compliance | XD-025 hash |
| `/api/v1/compliance/conflicts` | GET · POST | C4 | Compliance | INV-035 · XD-027 |
| `/api/v1/compliance/audit-trail` · `/condominiums/{id}/compliance/audit` | GET | C5 · S31 | Compliance · Auditoria | INV-033 · INV-090 · INV-120 DB · INV-091 |
| `/api/v1/compliance/amendments` | GET · POST | C6 | Compliance | INV-032 · INV-050 |
| `/api/v1/compliance/risk-map` | GET | C7 | Compliance | XD-029 |
| `/api/v1/compliance/contracts` | GET · POST | C8 | Compliance | INV-040 |
| `/api/v1/compliance/lgpd-logs*` | GET · POST | C9 | Compliance | INV-091 · INV-048 · INV-109 HMAC |
| `/api/v1/condominiums/{id}/governance-score/internal` | GET | C10 (privado) | Compliance | 🔴 endpoint ausente |
| `/api/v1/compliance/dossier` | POST | C11 | Compliance | INV-091 · XD-030 · 4 gatilhos |

### 3.7 Webhooks sistema (⚫ sem UI por design)

| Endpoint | Método | Origem | INV |
|---|---|---|---|
| `/api/v1/webhooks/stripe` | POST | Stripe | INV-015 UNIQUE event · INV-018 HMAC · INV-102 body limit 64KB · INV-105 DB idempotency |
| `/api/v1/webhooks/mux` | POST | Mux | INV-064 HMAC antes parse · INV-065 saga · INV-102 · INV-105 |
| `/api/v1/webhooks/livekit` | POST | Livekit | INV-084 saga · INV-093 · INV-102 · INV-105 |

---

## 4. Matriz invariantes × UX

Para cada INV crítico, onde o usuário vê/interage (UI enforcement visível ou side-effect percebido).

| INV | Descrição | Tela onde aparece | Comportamento UX |
|---|---|---|---|
| **INV-005** | ≤1 session ativa (1-device) | S1 login · tela conflito device (TBD) | Redirect para "/device-conflict" com opção de encerrar outra sessão |
| **INV-024** | Timeline append-only | S12 · S13 | Botão "editar" abre form de adendo (cria nova entry com `corrects_entry_id`), nunca UPDATE |
| **INV-025** | Announcement imutável pós publish | S24 | Após `state=published`, campos ficam read-only; botão editar → adendo |
| **INV-026** | Membership única por (user, condo) | S2 · M2 · M13 · ONB-M3 | Listar membros exclui duplicatas; M14 (dep) separa em lista diferente |
| **INV-027** | ≤15 condomínios por síndico | S2 | Botão "Cadastrar novo condomínio" desabilitado no 15º · tooltip explica |
| **INV-029** | 1 titular + ≤2 dep por unidade | M14 | Bloqueia adicionar dep além de 2; toast de erro explicativo |
| **INV-030** | `public_id` UNIQUE 8 chars | S5 · M3 · ONB-M3 | Input máscara 8 chars; se já usado em registry → "Condomínio já existe" com link pra pedir convite |
| **INV-034** | Declaração anual única | C2 · S29 · S30 · ONB-SFIN | Se já existe `year=2026`, form volta read-only com "Já submetida em..." |
| **INV-035** | Conflito interesse obrigatório | C4 · ONB-S6 | Bloqueia finalização do onboarding síndico sem declaração |
| **INV-037** | Connect Me unidirecional | S18 · S19 · S20 · E2 | Nenhuma UI de "responder"; recusa/aceite são ações finais sem thread |
| **INV-038** | Proposta única por (CM, company) | E4 | 2º submit → 409 + toast "Você já enviou proposta pra esta oportunidade" |
| **INV-039** | Proposta imutável pós send | E5 | Após enviar, form fica read-only + mostra adendo se houver |
| **INV-040** | Deal imutável pós dupla assinatura | E7 · E8 | Após `company_signed + sindico_signed`, tela em "modo contrato"; ajuste → adendo formal |
| **INV-041** | Deal auto-publica timeline | E8 → S11/M6 | Assinatura dupla dispara entry automática na timeline com tipo `registro_execucao` |
| **INV-043** | Evaluation única por (comp, evaluator, condo) | E11 · S22 | Prevenção: UI não deixa avaliar 2x; rede: 409 |
| **INV-044** | Anonimato do avaliador | E11 | Empresa vê "Avaliador: anônimo" em todas exceto se público |
| **INV-045** | Quota decrementa no publish | S18 · S19 | Banner "Quota: X/Y restantes"; draft não decrementa |
| **INV-046** | Reset anual (1/jan) | S18 | Tooltip "Renova em 1/jan"; resetKey Redis TTL até 31/12 |
| **INV-047** | Auto-close 48h sem interesse | E2 · E4 | Badge "Expira em 12h"; após 48h → estado `expired` |
| **INV-048** | LGPD log em reveal | S19 · S20 · E4 | Modal "Ao manifestar interesse, seus dados serão revelados e registrado em trilha LGPD" |
| **INV-049** | Agência zero-access deals/CM | MK1..MK8 | Tabs "Connect Me" e "Negócios" ocultas por ABAC DENY; ticket 403 se força |
| **INV-051** | Cooldown 4h por (user, loja) | VS5 · VM4 | Botão "Ativar" bloqueado com contador "Disponível em 3h27min" |
| **INV-056** | Vídeo lock 90d | E12 · MK5 · LMS3 · BT02 | Botão "Substituir" desabilitado até `published_at + 90d`; bypass só admin com audit log |
| **INV-057** | Remove pre-90d admin-only | MK5 (admin) | UI oculta p/ não-admin; admin vê botão "Remover (bypass)" + textarea motivo obrigatório |
| **INV-060** | Métricas sem leak tenant | MK3 · MK6 · E10 | Dashboards sempre escopados por empresa; cross-tenant = 403 |
| **INV-064** | HMAC Mux antes parse | ⚫ webhook `/webhooks/mux` | Transparente à UI; implementado middleware |
| **INV-067** | Parceiro vizinhança BLOQ LMS | LMS1 | ABAC DENY + UI oculta botão "Academia" completamente para `local_company` |
| **INV-068** | Cert auto 100% | LMS5 | Ao concluir última aula, modal "Parabéns, seu certificado foi gerado" |
| **INV-071** | Voto único (session, voter) | ASM-VOTO | Após votar, UI mostra "Seu voto: X" read-only; 2º submit → 409 |
| **INV-073** | Quórum por Σ fração | ASM-CHKIN · ASM-SIM · ASM-VOTO | Display "Quórum: 68,3% (50,0% necessário)" em tempo real |
| **INV-074** | Sem edital → sem assembleia | ASM-PUB · ASM-PAINEL-SIND | Botão "Iniciar" disabled até `edital_published_at` não NULL |
| **INV-076** | Timeout → Absent | ASM-VOTO | Timer 15min; se expirar sem voto, UI mostra "Você foi marcado como ausente" |
| **INV-077** | Proxy = voto titular | ASM-VOTO (proxy) | UI mostra "Votando em nome de: Fulano. Valor deve coincidir" |
| **INV-081** | Síndico-presidente não vota | ASM-PAINEL-SIND · ASM-VOTO | UI do presidente não mostra opção "Votar"; apenas conduzir |
| **INV-085** | Homologação separada | ASM-HOML | Tela dedicada pós-ata; sem ela, ata fica `closed` não `homologated` |
| **INV-086** | ABAC ordem estrita | todas | Erros padronizados: 401 sem sessão · 403 `action_not_configured` / `role_denied` / `tenant_mismatch` / `plan_tier_insufficient` |
| **INV-091** | LGPD retenção 5a | C9 · M15 · IDN-022 | M15 mostra "Dados anonimizados após 48h + hard-delete após 5a" |
| **INV-102** | Webhook body limit 64KB | ⚫ webhooks | Server-side apenas |
| **INV-107** | PATCH DTO estrito | M13 · PATCH /me TBD | UI não exibe campos `role`, `is_admin` em form; edição role = fluxo dedicado com MFA |
| **INV-110** | Step-up MFA ≤5min | ASM-PUB · ASM-ENCER · ASM-HOML · E14 revoke · DELETE /me (TBD) · `POST /admin/*` | Modal MFA reautenticação antes de ação; token TTL 5min |
| **INV-118** | Edital ≥8 dias | ASM-CFG · ASM-EDITAL | DateInput validation: `scheduled_date - now ≥ 8 dias`; servidor rejeita 422 `edital_insufficient_notice` |
| **INV-119** | Timeline/ata DB-level immutable | S11..S14 · M6/M7 · ASM-REL | Transparente: UI jamais oferece "delete"; tentar via API → 500 ERROR P0001 (bug UI) |
| **INV-122** | Tenant boundary guard | todas rotas scoped | Cross-tenant deep link → 403 + audit `tenant_boundary_violation` |

---

## 5. Gaps detectados

### 5.1 Telas sem endpoint identificado (🔴 gaps críticos)

Screens que dizem "endpoint inferido" ou não citam rota explícita — precisam de cross-check com backend requirements.

| Tela | Gap | Impacto | Owner |
|---|---|---|---|
| [[web/ui-catalog/sindico/S1]] | `/api/v1/syndic/governance/first-access` · `/acceptance` **inferidos** | Baixo — lógica simples, pode virar flag no `/me` | backend Sprint 10 |
| [[web/ui-catalog/sindico/S6]] | `/condominiums/{id}/dashboard/summary` + `/cards/badges` + `wss://.../events` **agregador não existe** | 🔴 alto — tela principal do síndico depende de N chamadas orquestradas | backend Sprint 10 (BE-015) |
| [[web/ui-catalog/sindico/S27]] | `/dashboard?period=` + `/scores/history` + `/export` **agregadores faltam** | 🔴 alto — Plus+ feature, M1 blocker | backend Sprint 10 |
| [[web/ui-catalog/compliance/C10]] | `/governance-score/internal` **endpoint ausente** — regra 1-10 vs 0-100 conflita | 🔴 alto — score privado síndico principal gate de Plus+ | backend (D-061 pendente) |
| [[web/ui-catalog/sindico/S18]] | `/connect-me/me/summary` **agregador inferido** | Médio — 3 queries separadas funcionariam | backend Sprint 10 |
| [[web/ui-catalog/empresa/E1]] · [[empresa/E10]] | `/enterprise/dashboard` · `/reputation/me?breakdown=true` **agregadores inferidos** | Médio | backend Sprint 10 |
| [[web/ui-catalog/marketing/MK1]] · [[marketing/MK7]] | `/marketing/dashboard` · `/dashboard/consolidated` **módulo `marketing` não existe** no backend — gap §10-marketing-agencias | Médio | backend M2 |
| [[web/ui-catalog/vizinhanca/_moc|web/ui-catalog/vizinhanca/]] | Módulo `neighborhood` **não implementado** (M2 backlog) | 🔴 M2 blocker | backend M2 |
| [[web/ui-catalog/lms/_moc|web/ui-catalog/lms/]] | Módulo `academy/live` **apenas stub** (Sprint 5); Lives provider ADR pendente (Mux Live vs Cloudflare Stream) | 🔴 M2 blocker | backend + ADR |
| [[web/ui-catalog/banco-talentos/_moc|web/ui-catalog/banco-talentos/]] | Módulo `talents` **não implementado** (M2 backlog) | 🔴 M2 blocker | backend M2 |
| [[web/ui-catalog/admin/GAP-admin-master-sindico]] | Painel admin não especificado | Baixo (fora M1) | produto M3 |

### 5.2 Endpoints sem tela mapeada

Endpoints que o backend expõe mas que UI não consome diretamente (alguns são corretos — sistema).

| Endpoint | Motivo | Ação |
|---|---|---|
| `/api/v1/webhooks/stripe` ⚫ | Sistema — Stripe → backend | Correto, sem UI |
| `/api/v1/webhooks/mux` ⚫ | Sistema — Mux → backend | Correto, sem UI |
| `/api/v1/webhooks/livekit` ⚫ | Sistema — Livekit → backend | Correto, sem UI |
| `/api/v1/health` · `/healthz` | Infra | K8s probes, sem UI |
| `/api/v1/admin/abac/invalidate` | Ops interno | Admin tooling |
| `/api/v1/jobs/*` (cron internos) | Sistema (ex: reset quota 1/jan, auto-close 48h) | Sem UI |
| `/api/v1/identity/sync-zitadel` | Job noturno IDN-031 | Sem UI |
| `/api/v1/auth/callback` | OIDC PKCE | Redirect do Zitadel, não navegável |

### 5.3 Invariantes sem UI enforcement visível

INVs que são server-side puras mas o UI idealmente refletiria (para UX clara):

| INV | Descrição | Ação recomendada UI |
|---|---|---|
| **INV-004** | Role nunca vazia — 403 antes UPSERT | Toast amigável ao invés de erro 403 cru |
| **INV-011** | `users.plan_id` aponta Subscription ativa | Banner global "Plano expirado em Xd" quando soft-block |
| **INV-013** | Trial nunca volta pós paid | Settings > plan esconde "voltar ao trial" após conversão |
| **INV-015** | Stripe webhook_event UNIQUE | Sem UX — mas UI pode mostrar "Última cobrança idempotente: OK" em Billing settings |
| **INV-016** | Quota saldo ≥0 · used≤limit | Barra de progresso em todas features com quota (Connect Me, condos) |
| **INV-017** | Trial duration persona-aware | Banner "Trial termina em Xd" com cor conforme persona (síndico 15d · empresa 7d · parceiro 30d) |
| **INV-019** | Soft-block sem grace | Banner "Plano bloqueado desde X. Renove pra reativar" sem countdown |
| **INV-024** | `audit_trail` append-only | C5/S31 já mostra trilha + C6 adendos explica |
| **INV-063** | Grant revogado pós votação | Na prática: vídeo some da UI quando votação fecha — mostrar toast "Votação encerrada, vídeo retornou para acervo empresa" |
| **INV-092** | PII nunca em logs | N/A UI — CI grep |
| **INV-094** | UUIDv7 | N/A UI — infra |
| **INV-097** | Rate limit 20/60rpm | UI deve tratar 429 com retry-after + mensagem "Muitas requisições, aguarde X segundos" |
| **INV-098** | CORS * bloqueado | N/A UI — bootstrap |
| **INV-103** | CORS allowlist específica | N/A UI — deploy config |
| **INV-104** | Hard-delete LGPD grace 48h | M15 mostrar "Dados anonimizados após 48h + hard-delete após 5a" |
| **INV-108** | tenant_id type-driven | N/A UI — compile-time Go |
| **INV-121** | RLS default-on tenant | N/A UI — DB policy |
| **INV-123** | PITR + DR drill mensal | N/A UI — ops |

### 5.4 Gaps top 5 críticos (bloqueadores M1)

1. **Dashboard agregador S6/S27/E10/E1** — 4 endpoints agregadores de dashboard não existem no backend. Alto risco de M1 (Sprint 10) ficar bloqueado.
2. **Score Governança C10 endpoint ausente** — regra 1-10 vs 0-100 não convergida (D-061 pendente), endpoint inexistente.
3. **Módulos `neighborhood`, `talents`, `academy` não implementados** — 55+ telas dependem, M2 bloqueado.
4. **Endpoints de onboarding específicos** — `/identity/validate-cpf` e `/validate-cnpj` inferidos, precisam de ADR (validação local vs Serpro).
5. **Endpoint agendador/cron via UI (reset quota Connect Me, auto-close 48h)** — não expostos; depende de INV-046 (timezone BRT) e INV-047.

---

## 6. Cobertura por sprint

Quantos endpoints/telas de cada sub-produto foram entregues em cada sprint. Fonte: [[backend/tasks/by-sprint/_moc]].

| Sprint | Período | Backend (endpoints cobertos) | Web (telas com build) | Mobile (telas com build) | Objetivo principal |
|---|---|---|---|---|---|
| 0 | 03-23..04-06 | Fundação (PG, Redis, Gin, OIDC) | 0 | 0 | Fundação Clean Arch |
| 1 | 04-07..04-20 | Identity ✅ + Billing ✅ (Stripe, ABAC, trial, quotas — 15 endpoints) | 0 | 0 | Identity/Billing |
| 2 | 04-21..04-27 | Institutional base (Condomínio, Unit, Timeline, PD — 12 endpoints) | 0 | 0 | Inst. base |
| 3 | 04-28..05-04 | Institutional completa (Events, Announcements, Polls, Compliance audit — 18 endpoints) | 0 | 0 | Inst. completa |
| 4 | 05-05..05-11 | Commercial (Connect Me, Proposals, Supplier Vote, Deals, Evaluations — 20 endpoints) | 0 | 0 | Commercial |
| 5 | 05-12..05-18 | Content (Video Mux, Marketing Agencies, delegation — 12 endpoints) | 0 | 0 | Content+Marketing |
| 6 | 05-19..05-25 | Infra (Railway, OTel, CI, Zitadel dev) | 0 | 0 | Infra |
| 7 | 05-26..06-01 | Assembly foundation (21 endpoints + Livekit saga) | 0 | 0 | Assembly |
| 8 | 06-02..06-08 | Security hardening (ABAC matriz 400 testes, rate limit, tenant isolation, HMAC) | 0 | 0 | Hardening |
| 9 | 04-21..04-22 | F1-F33 reconciliação + gates verdes (14 🔴 fechados parciais) | 0 | 0 | Fase 9 dual-check |
| **10 (M1)** | 04-23..05-18 | LGPD workflow + DT-004 + DT-010 + BE-001..BE-015 (aggregators dashboards) | 🔴 **auth+cms+onboarding+morador+síndico core** (bloqueadores) | 🟡 splash+home+timeline+avaliacao+chkin+voto+proc+ciência+tela ~15 | M1 release |

Distribuição de endpoints por sub-produto (estimativa pós Sprint 10):

| Sub-produto | Backend endpoints | Status backend | UI web % | UI mobile % |
|---|---|---|---|---|
| Governança Institucional | ~48 | ✅ | spec 100% · build M1 | spec 100% · build M1 |
| Connect Me | ~18 | ✅ | spec 100% | 0% (M2) |
| Reputação | ~12 | ✅ | spec 100% | 0% (M2) |
| Conteúdo & Vídeos | ~20 | ✅ | spec 100% (E12 + onb) | 0% |
| Assembleia Live | ~30 | ✅ | spec 100% · build M1 parcial | spec 24% (5/21) · build M1 |
| Compliance (C1-C11) | ~18 | ✅ (audit) | spec 100% · build M1 C1-C5 parcial | 0% |
| LMS Academia | ~22 | 🟡 stub | spec 100% · build M2/M3 | 0% |
| Banco de Talentos | ~14 | 🔴 0% | spec 100% · build M2 | 0% |
| Vizinhança | ~18 | 🔴 0% | spec 100% · build M2 | spec 24% (7/29) · build M2 |
| Marketing & Agências | ~12 | ✅ | spec 100% · build 🟡 | 0% |
| Onboarding | ~6 shared | ✅ | spec 100% · build M1 | spec 88% · build M1 |

---

## 7. Fluxos cross-stack (jornadas ponta-a-ponta)

Jornadas críticas que atravessam múltiplos sub-produtos e invariantes. Útil para rastrear "uma ação do usuário" ao longo de todo o stack.

### 7.1 Onboarding do síndico (persona-aware trial 15d)

```
URL /cadastro?role=syndic&plan=syndic_n2
  → ONB-00 boas-vindas
  → ONB-01 identificação (email) → Zitadel envia token
  → Confirma e-mail (24h TTL) → login habilitado
  → ONB-S2 identidade (CPF imutável pós ativação — INV-029)
  → ONB-S3 atuação (tipo síndico, qtd condos) → INV-027 ≤15
  → ONB-S4 governança markers (15 autodeclarados) → INS-040
  → ONB-S5 presença (N2/N3 — vídeo institucional — INV-056 lock 90d)
  → ONB-S6 aceites + conflito interesse → INV-035 · XD-025
  → ONB-SFIN finalize → backend cria User + Subscription trial(15d) + audit
  → Redirect S1 entrada governança
  → S1 aceite termo → S2 listar condos (vazia)
  → Botão "Cadastrar condomínio" → S3 → S4 → S5
  → Subscription trial ativa: pode criar até 15 condos, timeline, PD, comunicados
  → Após 15d sem upgrade: INV-019 soft-block ABAC imediato
```

Endpoints envolvidos: `/api/v1/onboarding/draft` (PATCH ×5) · `/api/v1/onboarding/finalize` · `/api/v1/identity/validate-cpf` · `/api/v1/videos?type=institutional` · `/api/v1/condominiums` · `/api/v1/auth/*`.

INVs aplicados: INV-002 · INV-017 · INV-027 · INV-029 · INV-056 · INV-086 · INV-100 · INV-106 · INV-107.

### 7.2 Onboarding morador com vínculo

```
ONB-00 seleciona Morador
  → ONB-01 nome + email → Zitadel
  → ONB-M2 identidade + CPF → INV-002
  → ONB-M3 público_id condomínio (8 chars) → INV-030 · /api/v1/condominiums?public_id=XXXXXXXX
  → Se unidade já tem titular: M4 flow "pedir como dependente" (INV-029 ≤2)
  → M5 termo ciência vínculo (LGPD versionado)
  → ONB-MFIN finalize → Membership criado, role=resident
  → M1 painel morador → INV-026 membership única
  → M6 timeline → lê INSERT-only timeline (INV-024)
  → A cada 2 meses: M12 avaliação bimestral bloqueante (INS-031)
```

### 7.3 Ciclo Connect Me (síndico → empresa)

```
S18 hub → S19 criar (LGPD consent + select empresa)
  → POST /api/v1/connect-me + decrementa quota (INV-045 · UoW)
  → Empresa: E2 lista recebidos (contato oculto — INV-048)
  → Empresa clica "Tenho interesse" → POST /api/v1/commercial/connect-me/:id/interested
     → Handler UoW: reveal contato + registra em lgpd_logs (INV-048)
  → E4 envia proposta → POST /api/v1/commercial/proposals (INV-038 única)
  → E4 submit → status=sent (INV-039 imutável)
  → Síndico valida em S26 → approve → proposta aceita
  → Síndico marca negócio fechado → Deal criado (COM-019)
  → Empresa+síndico assinam (E8 dupla) → INV-040 imutável
     → Handler emite evento ClosedDealSigned (INV-041)
     → Institutional cria TimelineEntry tipo registro_execucao (dentro UoW)
  → S11/M6 timeline mostra deal publicado
  → Ciclo de execução: E9 registra → S22 valida → E11 avaliação recebida
  → Reputação recalcula (INV-036 determinística · INV-054)
  → 48h sem interesse → INV-047 auto-close por cron
```

### 7.4 Ciclo assembleia Live completa

```
S6 Central → card Assembleia → ASM-DASH
  → ASM-CFG data + modalidade (INV-118 ≥8d)
  → ASM-PAUTA itens (INV-080 read-only pós publish)
  → ASM-EDITAL upload/gerar (INV-118)
  → ASM-PUB publish (X-Step-Up · INV-110) → INV-074 enforcement
  → Moradores: push "Nova assembleia publicada" → app mostra ASM-CIEN (bloqueante)
  → ASM-TERM 5 termos LGPD (versionado hash)
  → ASM-PROC-CAD procuração morador → ASM-PROC-VAL validada por admin
  → ASM-FRAC upload fração (INV-021 Σ=100 · INV-022 · INV-023 NUMERIC(5,4))
  → ASM-INAD upload inadimplência (cutoff 1h)
  → ASM-SIM simulador quórum (INV-073)
  → Dia da assembleia: ASM-CHKIN 3 modalidades → Σ fração presente
  → ASM-PAINEL-SIND (X-Step-Up · INV-110) iniciar → ASM-TELAO ao vivo
  → Para cada item: ASM-VOTO (INV-071 voto único · INV-083 snapshot frac · INV-081 síndico não vota · INV-076 timeout=Absent)
  → ASM-HOML por item (X-Step-Up · INV-085 obrigatória) → status resolvido/adiado/prejudicado
  → ASM-ENCER (X-Step-Up) → ata gerada (INV-072 · INV-119 DB-level)
     → Saga Livekit (INV-084) compensa se falhar (EndRoom)
  → ASM-REL 6 relatórios + score transparência (ASM-028 10 componentes)
  → TimelineEntry automática tipo "assembleia_encerrada"
```

INVs acionados (21): INV-021, INV-022, INV-023, INV-071, INV-072, INV-073, INV-074, INV-075, INV-076, INV-077, INV-078, INV-079, INV-080, INV-081, INV-082, INV-083, INV-084, INV-085, INV-110, INV-118, INV-119.

### 7.5 Upload de vídeo empresa (Marketing Agency delegado)

```
Empresa E14 convida agência → POST /enterprise/agencies/invite
  → IDN-025 invite token (one-shot, TTL 7d)
  → Agência aceita via link (sem auto-registro)
  → Zitadel cria user role=marketing_agency + ABAC delega videos:*, analytics:read (INV-066)
  → Agência MK3 switch-context empresa ativa
  → MK4 novo vídeo → POST /api/v1/videos?type=institutional
     → Backend gera presigned Mux + cria Video draft
     → Agência upload direto Mux (bypass backend — performance)
     → Mux chama /webhooks/mux quando ready (INV-064 HMAC · INV-102 body 64KB · INV-105 DB idempotency)
     → Saga UploadVideo (INV-065): se falha local, cancela Mux upload
     → Video state: draft → moderation (admin) → approved → published
     → INV-056 lock 90d ativado a partir published_at
  → MK6 analytics (INV-060 tenant isolation · INV-012 metrics privacy)
  → Empresa vê em E12 perfil (read-only do conteúdo agência)
  → Agência NUNCA vê CM, propostas, deals (INV-049 zero-access)
```

### 7.6 Encerramento de mandato (gatilho compliance)

```
S2 → botão "Encerrar mandato"
  → POST /api/v1/condominiums/{id}/end-mandate
  → Backend valida 4 gatilhos (C1):
    a) declaração anual do ano corrente submetida? (INV-034)
    b) todos usuários assinaram? (INV-C3)
    c) pendências críticas? (C7 mapa risco)
    d) conflito interesse declarado? (INV-035)
  → Se falhar: 422 "Para prosseguir, finalize as pendências de compliance"
  → Se ok: mandate.end_date = now, membership closed
  → C11 oferece exportar dossiê (PDF assinado digitalmente — XD-030 backlog M3)
  → Card S2 vai pra seção "Encerrados" com botão "Acessar histórico" (R6)
  → Timeline + audit preservados (INV-033 · INV-090 · INV-120)
```

### 7.7 Avaliação bimestral bloqueante (morador)

```
Cron bimestral → cria ManagementAssessmentResponse pendente
  → Mobile/Web: M12 bloqueia acesso a M1/M6/etc até submeter
  → GET /api/v1/institutional/assessments/me/pending → { pending: true }
  → Usuário avalia 3 critérios (comunicacao, organizacao, confianca 1-10)
  → POST /api/v1/institutional/assessments
  → Backend dispara handler recalc reputação síndico (INS-031 + agregador governance_score)
  → M1 destravado
```

---

## 8. Matriz ABAC actions × persona × plan × tela

Subset do XD-007 (ABAC matrix definition). Serve para entender quais ações ficam disponíveis por persona em cada plano, e onde aparecem na UI.

| Action | síndico trial | síndico base | síndico plus | síndico pro | morador | empresa plus | empresa pro | agência | parceiro | admin | Telas-gate |
|---|---|---|---|---|---|---|---|---|---|---|---|
| `timeline.read` | ✅ | ✅ | ✅ | ✅ | ✅ | — | — | — | — | ✅ | S11 · M6 |
| `timeline.create` | ✅ | ✅ | ✅ | ✅ | ❌ | — | — | — | — | ✅ | S12 |
| `timeline.amend` | ✅ | ✅ | ✅ | ✅ | ❌ | — | — | — | — | ✅ | S13 |
| `condominium.create` | ✅ (max 15) | ✅ (max 15) | ✅ (max 15) | ✅ (max 15) | ❌ | — | — | — | — | ✅ | S3 |
| `mandate.end` | ✅ | ✅ | ✅ | ✅ | — | — | — | — | — | ✅ | S2 |
| `dashboard.summary` | ✅ | ✅ | ✅ | ✅ | ✅ (resident view) | ✅ | ✅ | ✅ (empresa ativa) | — | ✅ | S6 · M1 · E1 · MK3 |
| `dashboard.advanced` | ❌ | ❌ | ✅ | ✅ | ❌ | ✅ | ✅ | ✅ | ❌ | ✅ | S27 · E10 · MK7 |
| `connect_me.send` | ✅ (quota 4) | ✅ (quota 4) | ✅ (quota 12) | ✅ ∞ | ❌ | ✅ | ✅ | ❌ INV-049 | ❌ | ✅ | S19 |
| `connect_me.receive` | — | — | — | — | — | ✅ | ✅ | ❌ | ✅ (Decision 12) | — | E2 · VZ1 |
| `proposal.create` | — | — | — | — | — | ✅ | ✅ | ❌ | — | — | E4 |
| `proposal.sign` | ✅ | ✅ | ✅ | ✅ | — | ✅ | ✅ | — | — | — | S22 · E8 |
| `deal.sign` | ✅ | ✅ | ✅ | ✅ | — | ✅ | ✅ | — | — | — | E8 |
| `evaluation.create` | ✅ | ✅ | ✅ | ✅ | — | — | — | — | — | ✅ | S22 |
| `evaluation.visibility.toggle` | — | — | — | — | — | ✅ | ✅ | — | — | ✅ | E11 |
| `reputation.read` | ✅ | ✅ | ✅ | ✅ | ✅ (público) | ✅ | ✅ | ✅ (empresa ativa) | — | ✅ | E10 · Perfil pub |
| `video.upload.institutional` | ❌ | ❌ | ✅ (N2/N3) | ✅ | — | ✅ | ✅ | ✅ (delegado) | ❌ | ✅ | ONB-S5 · E12 · MK4 |
| `video.moderate` | — | — | — | — | — | — | — | — | — | ✅ | Admin gap |
| `video.playback.resident` | — | — | — | — | ✅ (unless grant) | — | — | — | — | ✅ | M6 vídeo LT |
| `lms.course.enroll` | ✅ (Q5 aberta) | ✅ | ✅ | ✅ | ✅ (tier-dep) | ✅ | ✅ | ✅ | ❌ INV-067 | ✅ | LMS3 |
| `lms.certificate.generate` | ❌ (Q5) | ❌ (Q5) | ✅ | ✅ | ✅ (plus+) | ✅ | ✅ | ❌ | ❌ | ✅ | LMS5 |
| `lms.live.schedule` | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ (M2) | ✅ Pro (D-097) | ❌ | ✅ delegado (D-097) | ✅ admin | LMS8 (D-097 revoga admin-only MVP D-063/D-076) |
| `assembly.publish` | ✅ | ✅ | ✅ | ✅ | — | — | — | — | — | ✅ | ASM-PUB (X-Step-Up) |
| `assembly.vote.cast` | — | — | — | — | ✅ (se em unit) | — | — | — | — | ✅ (bypass audit) | ASM-VOTO |
| `assembly.homologate` | ✅ | ✅ | ✅ | ✅ | — | — | — | — | — | ✅ | ASM-HOML |
| `compliance.declaration.sign` | ✅ | ✅ | ✅ | ✅ | — | ✅ (own) | ✅ (own) | — | — | ✅ | C2 · S30 · E15 |
| `compliance.audit.read` | ✅ | ✅ | ✅ | ✅ | ❌ | ❌ | ❌ | ❌ | ❌ | ✅ | C5 · S31 |
| `compliance.dossier.export` | ❌ | ✅ | ✅ | ✅ | — | — | — | — | — | ✅ | C11 |
| `governance_score.read` | ✅ (principal only) | ✅ (principal only) | ✅ (principal only) | ✅ (principal only) | ❌ | — | — | — | — | ✅ | C10 (privado) |
| `neighborhood.promotion.create` | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | ✅ | ✅ | VZ3 |
| `neighborhood.promotion.activate` | ✅ (cooldown 4h) | ✅ | ✅ | ✅ | ✅ | ✅ (abertas) | ✅ (abertas) | ❌ | — | — | VS5 · VM4 · VE5 |
| `neighborhood.exclusive_benefit.create` | ✅ | ✅ | ✅ | ✅ | ❌ | ❌ | ❌ | ❌ | — | ✅ | VS7 |
| `neighborhood.exclusive_benefit.view` | ✅ (próprio) | ✅ | ✅ | ✅ | ✅ (unit.condo==benefit.condo) | ❌ | ❌ | — | — | ✅ | VS8 · VM5 |
| `marketing.agency.manage` | — | — | — | — | — | ✅ | ✅ | — | — | ✅ | E14 |
| `marketing.video.edit` | — | — | — | — | — | ✅ (próprio) | ✅ (próprio) | ✅ (delegado) | — | ✅ | MK5 · E12 |
| `talents.curriculum.publish` | ❌ | ❌ | ❌ | ❌ | ✅ (Plus/Pro) | ❌ | ❌ | ❌ | ❌ | ✅ | BT11 |
| `talents.search` | ❌ | ❌ | ❌ | ❌ | ❌ | ✅ | ✅ (analytics) | ❌ | ❌ | ✅ | BT search TBD |
| `admin.bypass.lock90d` | — | — | — | — | — | — | — | — | — | ✅ (audit INV-088) | — (headless) |

Observações:

- `❌` explícito = ABAC DENY (INV-086 ordem estrita, INV-087 default-deny)
- `—` = persona não se aplica à action (ex: morador não tem `proposal.*`)
- Agência: política `delegated_marketing` (INV-049 · INV-066) — 100+ integration tests garantem zero-access a deals/propostas/CM/timeline

---

## 9. Matriz de segurança & compliance hardening

Requisitos de segurança mapeados às telas/endpoints. Deriva dos findings A-DC-SEC-### e A-DC-PEN-### (Fase 5).

| Hardening | INV | Telas/Endpoints afetados | Verify |
|---|---|---|---|
| Ownership check antes de query (IDOR) | INV-101 | `DELETE /me/*` · `DELETE /institutional/memberships/{id}` (M15) · `PATCH /commercial/evaluations/:id` (E11) | Middleware `OwnershipCheck` + PBT |
| Webhook body limit 64KB | INV-102 | `/webhooks/stripe` · `/webhooks/mux` · `/webhooks/livekit` ⚫ | Middleware `WebhookBodyLimit` + CI grep |
| CORS allowlist específica | INV-103 | startup · todas rotas | `Config.Validate()` startup test |
| LGPD soft-delete grace 48h | INV-104 | M15 · IDN-022 · webhooks Stripe/Mux/Livekit | Saga `AccountDeletionSaga` + integration race test |
| Webhook idempotency DB-first | INV-105 | `/webhooks/*` ⚫ | Schema UNIQUE + chaos test Redis down |
| Session regenerate em transição | INV-106 | S1 login · logout · MFA step-up · role change · password reset · logout multi-device | Integration session fixation test |
| DTO estrito em PATCH (mass-assignment) | INV-107 | `PATCH /me` · `PATCH /institutional/memberships/{id}` (M13) · `PATCH /commercial/proposals/:id` | OpenAPI strict + PBT fuzz + CI grep |
| tenant_id type-driven | INV-108 | todas handlers/repos com tenant | Compile-time (Go types) + CI grep |
| LGPD pseudo keyed HMAC rotating | INV-109 | M15 · C9 (lgpd_logs) · audit anon | Unit + Integration · key rotation 1/jan |
| Step-up MFA ≤5min | INV-110 | ASM-PUB · ASM-ENCER · ASM-HOML · E14 revoke · DELETE /me · GET /me/export · `POST /admin/*` · publish minutes · approve contract ≥ R$10k · transfer mandate · PATCH email/phone | Middleware `RequireStepUp(endpoint)` + Zitadel MFA + per-endpoint integration test |
| Edital ≥8d Lei 4.591/64 | INV-118 | ASM-CFG · ASM-EDITAL · ASM-PUB | Handler validator + DB CHECK + INSERT trigger |
| Timeline/ata DB-level immutable | INV-119 | timeline_entries · assembly_minutes (published/homologated) | Migration REVOKE + trigger `forbid_mutation()` + chaos test |
| Audit log absolutamente imutável | INV-120 | audit_log_entries · LGPD logs · webhook events | Migration REVOKE DELETE/UPDATE + partitioning |
| RLS tenant isolation | INV-121 | todas tabelas com tenant_id (exceto globals com comment) | Migration linter `rls-check.sh` + `pg_policies` query |
| Tenant boundary guard middleware | INV-122 | toda rota com {tenant_id} / {condominium_id} / {company_id} em path | Middleware + CI grep + 500-case fuzz |
| PITR + DR drill mensal | INV-123 | Postgres · pgBackRest · S3/R2 | postgresql.conf archive_mode + alert + cron DR drill + drill-YYYY-MM.md |

### 9.1 Step-up MFA (INV-110) — checklist por endpoint

Todos exigem header `X-Step-Up-Token` com TTL ≤5min:

- `POST /api/v1/assemblies/:id/publish` (ASM-PUB) — publicação edital
- `POST /api/v1/assemblies/:id/start` (ASM-PAINEL-SIND) — iniciar assembleia
- `POST /api/v1/assemblies/:id/close` (ASM-ENCER) — encerrar
- `POST /api/v1/assemblies/:id/agenda/:item/homologate` (ASM-HOML)
- `DELETE /api/v1/me` — account deletion
- `DELETE /api/v1/me/devices/:id` — revogar device
- `GET /api/v1/me/export` — LGPD export
- `PATCH /api/v1/me` (apenas para email/phone/role) — INV-107 ressalva
- `POST /api/v1/enterprise/agencies/:id/revoke` (E14) — revogar agência
- `POST /api/v1/commercial/contracts/:id/approve` quando valor ≥ R$10.000 (COM-017)
- `POST /api/v1/institutional/mandates/:id/transfer` — INS-002
- `POST /api/v1/admin/*` — todas rotas admin (INV-088 audit bypass)

### 9.2 LGPD compliance mapping

| Fluxo | Tela | Endpoint | Log obrigatório |
|---|---|---|---|
| Consentimento versionado | ONB-M4 · ONB-S6 · ONB-E7 · ONB-P4 · M5 · ASM-TERM · BT10 · C3 · E8 | `/onboarding/finalize` · `/terms/accept` · `/lgpd-consent` | `lgpd_logs` insert |
| Reveal Connect Me | S19 · S20 · E4 | `/commercial/connect-me/:id/interested` · `/respond` | `lgpd_logs` insert (INV-048) |
| Direito ao esquecimento | M15 · IDN-022 | `DELETE /api/v1/me` (step-up) | Saga + soft-flag 48h + pseudonymize + hard-delete key 5a |
| Portabilidade | (TBD UI) | `GET /api/v1/me/export` (step-up) | `lgpd_logs` insert + signed URL TTL 24h |
| Audit visibility | C5 · S31 | `/compliance/audit-trail` | Read-only, sem enforcement extra |
| Log registry central | C9 | `/compliance/lgpd-logs` | Append-only (INV-091) + keyed HMAC (INV-109) |

---

## 10. Resumo executivo

### Totais consolidados

- **161 telas web** especificadas em MDs (sindico 31, morador 15, empresa 16, marketing 8, compliance 11, vizinhanca 29, lms 15, banco-talentos 11, assembleia 21, onboarding 23, admin 1 gap)
- **59 telas mobile** (morador 15, sindico 5, assembly 5, onboarding 21, vizinhanca 7, shared 6) — subset das 161 web
- **~240 endpoints distintos** identificados nos MDs (após dedup por path+method)
- **116 invariantes canônicos** (100 v2 + 10 Fase 5 + 6 Fase 9)
- **230+ requirements funcionais** (7 BCs EARS)

### Sub-produto com maior cobertura

**Governança Institucional** — 48 endpoints backend (✅ Sprint 2+3), 39 telas spec entre web/mobile, 100% catalogado. É o núcleo do MVP; todos os outros sub-produtos plugam aqui.

### Sub-produto com menor cobertura

**Admin** — 1 tela apenas, marcada `GAP`. Todos os fluxos admin (moderação vídeo, LMS, BT, auditoria transversal, ABAC editor, ban local, sync Zitadel manual) estão sem spec e sem build. Correto por escopo (M3+), mas hoje é o maior buraco.

**Vice-líder em baixa cobertura**: **Banco de Talentos** — 11 telas spec 100%, mas 0% backend implementado; bloqueia M2.

---

## 11. Links

- [[_moc]]
- [[../CLAUDE]]
- [[../00-product/sub-produtos/_moc]] — catálogo 11 sub-produtos
- [[../01-domain/invariants]] — 116 INVs canônicos
- [[../04-requirements/_moc]] — FRs por BC
- [[../02-architecture/api-design]] — contrato REST canônico
- [[../11-audit/AUDIT]] — findings + dual-check
- [[web/ui-catalog/sindico/_moc]] · [[web/ui-catalog/morador/_moc]] · [[web/ui-catalog/empresa/_moc]] · [[web/ui-catalog/assembleia/_moc]] · [[web/ui-catalog/compliance/_moc]] · [[web/ui-catalog/vizinhanca/_moc]] · [[web/ui-catalog/lms/_moc]] · [[web/ui-catalog/banco-talentos/_moc]] · [[web/ui-catalog/marketing/_moc]] · [[web/ui-catalog/onboarding/_moc]] · [[web/ui-catalog/admin/_moc]]
- [[mobile/ui-catalog/_moc]]
- [[backend/tasks/by-sprint/_moc]]
