---
title: Gap Analysis — Endpoints (telas vs backend)
type: audit
tags:
  - master-sindico
  - gap-analysis
  - endpoints
  - audit
created: 2026-04-24T00:00:00.000Z
---

# Gap Analysis — Endpoints (telas vs backend requirements)

> Análise sistemática cruzando 245 telas (192 web + 59 mobile) contra 7 bounded contexts backend (`identity, billing, institutional, commercial, content, assembly, cross-domain`). Objetivo: detectar endpoints fantasma (bloqueadores M1), APIs órfãs, endpoints inferidos não-validados e divergências web×mobile.

## Sumário

- **Telas analisadas**: 245 arquivos `.md` em ui-catalog (192 web / 59 mobile)
- **Endpoints únicos citados em telas** (método+path, query-less): **340**
- **Endpoints formalizados em backend/requirements**: **169**
- **Gaps A — fantasmas (tela cita, backend não formaliza)**: **333** (🔴 CRITICAL 112 / 🟠 HIGH 140 / 🟡 MEDIUM 81 / 🟢 LOW 0)
- **Gaps B — backend sem tela consumidora**: **162** (webhooks+admin+infra legítimos + possíveis órfãos)
- **Endpoints marcados como 'inferido' nas telas (C)**: **51** menções com path em **33** telas
- **Divergências web×mobile (D)**: **45** telas apresentam diferenças de endpoint entre plataformas

### Leitura rápida

- **~98% dos endpoints citados pelas telas NÃO estão em nenhum `backend/requirements/*.md`**. As 7 specs de BC hoje enumeram só o _núcleo_ (endpoints canônicos do M1 identity/billing); tudo que é leitura, derivado ou feature M2+ fica implícito. Não é 'API faltando no código' (vários handlers existem em `internal/modules/*`), mas é **falta de formalização REST**, o que quebra o rastreio FR-UI↔FR-BE e deixa o DoR fraco para Sprint 10.
- Priorize pelo **ranking de severidade** (ROADMAP.md): só `🔴 CRITICAL` já bloqueia o M1 (2026-05-08).
- Este relatório é derivado de scripts determinísticos (ver seção Metodologia). Dados brutos em `/tmp/gap-analysis/` desta sessão — regeneráveis.

---

## A. Endpoints fantasma (telas precisam, backend não formaliza)

Severidade derivada do ROADMAP:

- 🔴 CRITICAL → M1 (identity + billing + institutional base; onboarding; unidades; timeline; plano diretor; dashboard morador/síndico; anúncios institucionais)
- 🟠 HIGH → M2 (commercial/Connect-Me; content/vídeos; reputação; vizinhança; moderação)
- 🟡 MEDIUM → M3 (assembleia; LMS thin; enquetes)
- 🟢 LOW → infra/admin

### 🔴 CRITICAL — 112 endpoints

| Método | Path | Telas consumidoras | Sub-produto | Ação recomendada |
|---|---|---|---|---|
| GET | `/api/v1/assemblies/{id}/timeline` | [[../../03-subprojects/web/ui-catalog/assembleia/ASM-REL|ASM-REL]] | assembleia | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/assemblies/{id}/transparency-score` |
| GET | `/api/v1/condominiums` | [[../../03-subprojects/web/ui-catalog/sindico/S2|S2]], [[../../03-subprojects/web/ui-catalog/vizinhanca/VZ3|VZ3]] | governanca-institucional, vizinhanca | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/condominiums/{id}/units` |
| POST | `/api/v1/condominiums/draft` | [[../../03-subprojects/web/ui-catalog/sindico/S3|S3]] | governanca-institucional | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/condominiums/{id}/units` |
| GET | `/api/v1/condominiums/fractions/template` | [[../../03-subprojects/web/ui-catalog/assembleia/ASM-FRAC|ASM-FRAC]] | assembleia | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/condominiums/{id}/units` |
| POST | `/api/v1/condominiums/{draft_id}/finalize` | [[../../03-subprojects/web/ui-catalog/sindico/S5|S5]] | governanca-institucional | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/condominiums/{id}/units` |
| POST | `/api/v1/condominiums/{draft_id}/mandate` | [[../../03-subprojects/web/ui-catalog/sindico/S4|S4]] | governanca-institucional | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/condominiums/{id}/units` |
| POST | `/api/v1/condominiums/{id}/amendments` | [[../../03-subprojects/web/ui-catalog/sindico/S10|S10]] | governanca-institucional | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/condominiums/{id}/units` |
| GET | `/api/v1/condominiums/{id}/announcements` | [[../../03-subprojects/web/ui-catalog/sindico/S23|S23]], [[../../03-subprojects/web/ui-catalog/sindico/S25|S25]] | governanca-institucional | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/condominiums/{id}/units` |
| POST | `/api/v1/condominiums/{id}/announcements` | [[../../03-subprojects/web/ui-catalog/sindico/S24|S24]] | governanca-institucional | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/condominiums/{id}/units` |
| GET | `/api/v1/condominiums/{id}/announcements/summary` | [[../../03-subprojects/web/ui-catalog/sindico/S23|S23]] | governanca-institucional | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/condominiums/{id}/units` |
| POST | `/api/v1/condominiums/{id}/announcements/{aid}/reject` | [[../../03-subprojects/web/ui-catalog/sindico/S25|S25]] | governanca-institucional | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/condominiums/{id}/units` |
| POST | `/api/v1/condominiums/{id}/announcements/{aid}/request-adjustment` | [[../../03-subprojects/web/ui-catalog/sindico/S25|S25]] | governanca-institucional | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/condominiums/{id}/units` |
| POST | `/api/v1/condominiums/{id}/announcements/{aid}/validate` | [[../../03-subprojects/web/ui-catalog/sindico/S25|S25]] | governanca-institucional | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/condominiums/{id}/units` |
| GET | `/api/v1/condominiums/{id}/assemblies` | [[../../03-subprojects/web/ui-catalog/assembleia/ASM-DASH|ASM-DASH]] | assembleia | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/condominiums/{id}/units` |
| POST | `/api/v1/condominiums/{id}/assemblies` | [[../../03-subprojects/web/ui-catalog/assembleia/ASM-DASH|ASM-DASH]] | assembleia | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/condominiums/{id}/units` |
| GET | `/api/v1/condominiums/{id}/assemblies/kpis` | [[../../03-subprojects/web/ui-catalog/assembleia/ASM-DASH|ASM-DASH]] | assembleia | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/condominiums/{id}/units` |
| GET | `/api/v1/condominiums/{id}/cards/badges` | [[../../03-subprojects/web/ui-catalog/sindico/S6|S6]] | governanca-institucional | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/condominiums/{id}/units` |
| POST | `/api/v1/condominiums/{id}/compliance/annual-declaration` | [[../../03-subprojects/web/ui-catalog/sindico/S30|S30]] | governanca-institucional | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/condominiums/{id}/units` |
| GET | `/api/v1/condominiums/{id}/compliance/annual-declaration/status` | [[../../03-subprojects/web/ui-catalog/sindico/S30|S30]] | governanca-institucional | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/condominiums/{id}/units` |
| GET | `/api/v1/condominiums/{id}/compliance/audit` | [[../../03-subprojects/web/ui-catalog/sindico/S31|S31]] | governanca-institucional | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/condominiums/{id}/units` |
| GET | `/api/v1/condominiums/{id}/compliance/audit/export` | [[../../03-subprojects/web/ui-catalog/sindico/S31|S31]] | governanca-institucional | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/condominiums/{id}/units` |
| GET | `/api/v1/condominiums/{id}/compliance/audit/{entry_id}` | [[../../03-subprojects/web/ui-catalog/sindico/S31|S31]] | governanca-institucional | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/condominiums/{id}/units` |
| POST | `/api/v1/condominiums/{id}/compliance/responsibility-term/sign` | [[../../03-subprojects/web/ui-catalog/sindico/S29|S29]] | governanca-institucional | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/condominiums/{id}/units` |
| GET | `/api/v1/condominiums/{id}/compliance/summary` | [[../../03-subprojects/web/ui-catalog/sindico/S28|S28]] | governanca-institucional | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/condominiums/{id}/units` |
| GET | `/api/v1/condominiums/{id}/dashboard` | [[../../03-subprojects/web/ui-catalog/sindico/S27|S27]] | governanca-institucional | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/condominiums/{id}/units` |
| GET | `/api/v1/condominiums/{id}/dashboard/export` | [[../../03-subprojects/web/ui-catalog/sindico/S27|S27]] | governanca-institucional | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/condominiums/{id}/units` |
| GET | `/api/v1/condominiums/{id}/dashboard/summary` | [[../../03-subprojects/web/ui-catalog/sindico/S6|S6]] | governanca-institucional | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/condominiums/{id}/units` |
| POST | `/api/v1/condominiums/{id}/end-mandate` | [[../../03-subprojects/web/ui-catalog/sindico/S2|S2]] | governanca-institucional | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/condominiums/{id}/units` |
| GET | `/api/v1/condominiums/{id}/events` | [[../../03-subprojects/web/ui-catalog/sindico/S15|S15]], [[../../03-subprojects/web/ui-catalog/sindico/S17|S17]] | governanca-institucional | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/condominiums/{id}/units` |
| POST | `/api/v1/condominiums/{id}/events` | [[../../03-subprojects/web/ui-catalog/sindico/S16|S16]] | governanca-institucional | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/condominiums/{id}/units` |
| GET | `/api/v1/condominiums/{id}/events/export` | [[../../03-subprojects/web/ui-catalog/sindico/S17|S17]] | governanca-institucional | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/condominiums/{id}/units` |
| GET | `/api/v1/condominiums/{id}/fractions/imports/{import_id}` | [[../../03-subprojects/web/ui-catalog/assembleia/ASM-FRAC|ASM-FRAC]] | assembleia | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/condominiums/{id}/units` |
| POST | `/api/v1/condominiums/{id}/fractions/imports/{import_id}/commit` | [[../../03-subprojects/web/ui-catalog/assembleia/ASM-FRAC|ASM-FRAC]] | assembleia | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/condominiums/{id}/units` |
| POST | `/api/v1/condominiums/{id}/fractions/upload` | [[../../03-subprojects/web/ui-catalog/assembleia/ASM-FRAC|ASM-FRAC]] | assembleia | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/condominiums/{id}/units` |
| POST | `/api/v1/condominiums/{id}/invite-shared` | [[../../03-subprojects/web/ui-catalog/sindico/S5|S5]] | governanca-institucional | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/condominiums/{id}/units` |
| GET | `/api/v1/condominiums/{id}/master-plan/actions` | [[../../03-subprojects/web/ui-catalog/sindico/S7|S7]], [[../../03-subprojects/web/ui-catalog/sindico/S9|S9]] | governanca-institucional | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/condominiums/{id}/master-plan/export` |
| POST | `/api/v1/condominiums/{id}/master-plan/actions` | [[../../03-subprojects/web/ui-catalog/sindico/S8|S8]] | governanca-institucional | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/condominiums/{id}/master-plan/batch-advance` |
| GET | `/api/v1/condominiums/{id}/master-plan/actions/{action_id}` | [[../../03-subprojects/web/ui-catalog/sindico/S10|S10]] | governanca-institucional | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/condominiums/{id}/master-plan/export` |
| PATCH | `/api/v1/condominiums/{id}/master-plan/actions/{action_id}` | [[../../03-subprojects/web/ui-catalog/sindico/S10|S10]] | governanca-institucional | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/condominiums/{id}` |
| POST | `/api/v1/condominiums/{id}/master-plan/actions/{action_id}/hide` | [[../../03-subprojects/web/ui-catalog/sindico/S10|S10]] | governanca-institucional | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/condominiums/{id}/master-plan/batch-advance` |
| GET | `/api/v1/condominiums/{id}/master-plan/actions/{action_id}/history` | [[../../03-subprojects/web/ui-catalog/sindico/S10|S10]] | governanca-institucional | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/condominiums/{id}/master-plan/export` |
| GET | `/api/v1/condominiums/{id}/master-plan/summary` | [[../../03-subprojects/web/ui-catalog/sindico/S7|S7]] | governanca-institucional | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/condominiums/{id}/master-plan/export` |
| POST | `/api/v1/condominiums/{id}/memberships/request` | [[../../03-subprojects/web/ui-catalog/onboarding/ONB-M3|ONB-M3]] | governanca-institucional | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/condominiums/{id}/units` |
| GET | `/api/v1/condominiums/{id}/scores/history` | [[../../03-subprojects/web/ui-catalog/sindico/S27|S27]] | governanca-institucional | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/condominiums/{id}/units` |
| GET | `/api/v1/condominiums/{id}/service-records` | [[../../03-subprojects/web/ui-catalog/sindico/S21|S21]] | governanca-institucional | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/condominiums/{id}/units` |
| POST | `/api/v1/condominiums/{id}/service-records/external-link` | [[../../03-subprojects/web/ui-catalog/sindico/S21|S21]] | governanca-institucional | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/condominiums/{id}/units` |
| GET | `/api/v1/condominiums/{id}/service-records/{record_id}` | [[../../03-subprojects/web/ui-catalog/sindico/S22|S22]] | governanca-institucional | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/condominiums/{id}/units` |
| POST | `/api/v1/condominiums/{id}/service-records/{record_id}/evaluation` | [[../../03-subprojects/web/ui-catalog/sindico/S22|S22]] | governanca-institucional | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/condominiums/{id}/units` |
| POST | `/api/v1/condominiums/{id}/service-records/{record_id}/reject` | [[../../03-subprojects/web/ui-catalog/sindico/S22|S22]] | governanca-institucional | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/condominiums/{id}/units` |
| POST | `/api/v1/condominiums/{id}/service-records/{record_id}/validate` | [[../../03-subprojects/web/ui-catalog/sindico/S22|S22]] | governanca-institucional | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/condominiums/{id}/units` |
| POST | `/api/v1/condominiums/{id}/timeline` | [[../../03-subprojects/web/ui-catalog/sindico/S12|S12]] | governanca-institucional | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/condominiums/{id}/timeline/{type}` |
| GET | `/api/v1/condominiums/{id}/timeline/{entry_id}` | [[../../03-subprojects/web/ui-catalog/sindico/S13|S13]] | governanca-institucional | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/condominiums/{id}/timeline/{entry_id}/chain` |
| GET | `/api/v1/condominiums/{id}/timeline/{entry_id}/diff` | [[../../03-subprojects/web/ui-catalog/sindico/S14|S14]] | governanca-institucional | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/condominiums/{id}/timeline/{entry_id}/chain` |
| GET | `/api/v1/condominiums/{id}/timeline/{entry_id}/versions` | [[../../03-subprojects/web/ui-catalog/sindico/S14|S14]] | governanca-institucional | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/condominiums/{id}/timeline/{entry_id}/chain` |
| POST | `/api/v1/condominiums/{id}/timeline/{entry_id}/versions` | [[../../03-subprojects/web/ui-catalog/sindico/S13|S13]] | governanca-institucional | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/condominiums/{id}/timeline/{type}` |
| GET | `/api/v1/condominiums/{id}/validations` | [[../../03-subprojects/web/ui-catalog/sindico/S26|S26]] | governanca-institucional | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/condominiums/{id}/units` |
| GET | `/api/v1/condominiums/{id}/validations/summary` | [[../../03-subprojects/web/ui-catalog/sindico/S26|S26]] | governanca-institucional | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/condominiums/{id}/units` |
| GET | `/api/v1/governance/executions` | [[../../03-subprojects/web/ui-catalog/empresa/E10|E10]] | connect-me | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/videos/{id}/playback` |
| POST | `/api/v1/governance/executions` | [[../../03-subprojects/web/ui-catalog/empresa/E9|E9]] | connect-me | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/videos/{id}/view-progress` |
| PUT | `/api/v1/governance/executions/draft` | [[../../03-subprojects/web/ui-catalog/empresa/E9|E9]] | connect-me | Formalizar em `backend/requirements/*.md` ou mover UI para backlog |
| POST | `/api/v1/governance/executions/{id}/submit` | [[../../03-subprojects/web/ui-catalog/empresa/E9|E9]] | connect-me | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/videos/{id}/view-progress` |
| POST | `/api/v1/governance/technical-activities` | [[../../03-subprojects/web/ui-catalog/empresa/E6|E6]] | connect-me | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/videos/{id}/view-progress` |
| PUT | `/api/v1/governance/technical-activities/draft` | [[../../03-subprojects/web/ui-catalog/empresa/E6|E6]] | connect-me | Formalizar em `backend/requirements/*.md` ou mover UI para backlog |
| POST | `/api/v1/governance/technical-activities/{id}/submit` | [[../../03-subprojects/web/ui-catalog/empresa/E6|E6]] | connect-me | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/videos/{id}/view-progress` |
| GET | `/api/v1/institutional/announcements/{cond}` | [[../../03-subprojects/mobile/ui-catalog/morador/M11|m:M11]] | governanca-institucional | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/videos/{id}/playback` |
| GET | `/api/v1/institutional/announcements/{id}` | [[../../03-subprojects/mobile/ui-catalog/morador/M11|m:M11]] | governanca-institucional | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/videos/{id}/playback` |
| POST | `/api/v1/institutional/announcements/{id}/acknowledge` | [[../../03-subprojects/web/ui-catalog/morador/M11|M11]] | governanca-institucional | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/videos/{id}/view-progress` |
| POST | `/api/v1/institutional/announcements/{id}/read` | [[../../03-subprojects/mobile/ui-catalog/morador/M11|m:M11]] | governanca-institucional | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/videos/{id}/view-progress` |
| POST | `/api/v1/institutional/assessments` | [[../../03-subprojects/web/ui-catalog/morador/M12|M12]] | governanca-institucional | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/videos/{id}/view-progress` |
| GET | `/api/v1/institutional/assessments/me/pending` | [[../../03-subprojects/web/ui-catalog/morador/M12|M12]] | governanca-institucional | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/videos/{id}/playback` |
| GET | `/api/v1/institutional/condominiums/lookup` | [[../../03-subprojects/web/ui-catalog/morador/M3|M3]] | governanca-institucional | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/videos/{id}/playback` |
| GET | `/api/v1/institutional/condominiums/{id}/announcements` | [[../../03-subprojects/web/ui-catalog/morador/M11|M11]] | governanca-institucional | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/videos/{id}/playback` |
| GET | `/api/v1/institutional/condominiums/{id}/events` | [[../../03-subprojects/web/ui-catalog/morador/M10|M10]] | governanca-institucional | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/videos/{id}/playback` |
| GET | `/api/v1/institutional/condominiums/{id}/master-plan` | [[../../03-subprojects/web/ui-catalog/morador/M8|M8]] | governanca-institucional | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/videos/{id}/playback` |
| GET | `/api/v1/institutional/condominiums/{id}/timeline` | [[../../03-subprojects/web/ui-catalog/morador/M6|M6]] | governanca-institucional | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/videos/{id}/playback` |
| GET | `/api/v1/institutional/dashboard/resident` | [[../../03-subprojects/mobile/ui-catalog/morador/M1|m:M1]], [[../../03-subprojects/web/ui-catalog/morador/M1|M1]] | governanca-institucional | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/videos/{id}/playback` |
| POST | `/api/v1/institutional/events/{id}/acknowledge` | [[../../03-subprojects/web/ui-catalog/morador/M10|M10]] | governanca-institucional | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/videos/{id}/view-progress` |
| DELETE | `/api/v1/institutional/events/{id}/cancel-participation` | [[../../03-subprojects/web/ui-catalog/morador/M10|M10]] | governanca-institucional | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/proxies/{id}` |
| POST | `/api/v1/institutional/events/{id}/confirm-participation` | [[../../03-subprojects/web/ui-catalog/morador/M10|M10]] | governanca-institucional | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/videos/{id}/view-progress` |
| POST | `/api/v1/institutional/invites/{id}/resend` | [[../../03-subprojects/web/ui-catalog/morador/M14|M14]] | governanca-institucional | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/videos/{id}/view-progress` |
| GET | `/api/v1/institutional/lgpd/consents/{id}` | [[../../03-subprojects/web/ui-catalog/morador/M13|M13]] | governanca-institucional | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/videos/{id}/playback` |
| GET | `/api/v1/institutional/master-plan/{id}` | [[../../03-subprojects/web/ui-catalog/morador/M9|M9]] | governanca-institucional | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/videos/{id}/playback` |
| GET | `/api/v1/institutional/master-plan/{id}/timeline` | [[../../03-subprojects/web/ui-catalog/morador/M9|M9]] | governanca-institucional | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/videos/{id}/playback` |
| POST | `/api/v1/institutional/memberships` | [[../../03-subprojects/web/ui-catalog/morador/M5|M5]] | governanca-institucional | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/videos/{id}/view-progress` |
| POST | `/api/v1/institutional/memberships/draft` | [[../../03-subprojects/web/ui-catalog/morador/M4|M4]] | governanca-institucional | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/videos/{id}/view-progress` |
| GET | `/api/v1/institutional/memberships/me` | [[../../03-subprojects/mobile/ui-catalog/morador/M2|m:M2]], [[../../03-subprojects/web/ui-catalog/morador/M2|M2]] | governanca-institucional | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/videos/{id}/playback` |
| GET | `/api/v1/institutional/memberships/me/default` | [[../../03-subprojects/mobile/ui-catalog/morador/M1|m:M1]], [[../../03-subprojects/web/ui-catalog/morador/M1|M1]] | governanca-institucional | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/videos/{id}/playback` |
| PATCH | `/api/v1/institutional/memberships/me/default` | [[../../03-subprojects/mobile/ui-catalog/morador/M1|m:M1]], [[../../03-subprojects/mobile/ui-catalog/morador/M2|m:M2]] | governanca-institucional | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/partners/{id}/palavra-do-dia` |
| GET | `/api/v1/institutional/memberships/me/{condo_id}` | [[../../03-subprojects/web/ui-catalog/morador/M13|M13]] | governanca-institucional | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/videos/{id}/playback` |
| POST | `/api/v1/institutional/memberships/{dependent_id}/revoke` | [[../../03-subprojects/web/ui-catalog/morador/M14|M14]] | governanca-institucional | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/videos/{id}/view-progress` |
| GET | `/api/v1/institutional/memberships/{id}/cascade-preview` | [[../../03-subprojects/web/ui-catalog/morador/M15|M15]] | governanca-institucional | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/videos/{id}/playback` |
| GET | `/api/v1/institutional/memberships/{id}/dependents` | [[../../03-subprojects/web/ui-catalog/morador/M14|M14]] | governanca-institucional | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/videos/{id}/playback` |
| POST | `/api/v1/institutional/memberships/{id}/dependents/invite` | [[../../03-subprojects/web/ui-catalog/morador/M14|M14]] | governanca-institucional | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/videos/{id}/view-progress` |
| POST | `/api/v1/institutional/memberships/{id}/revoke` | [[../../03-subprojects/web/ui-catalog/morador/M15|M15]] | governanca-institucional | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/videos/{id}/view-progress` |
| POST | `/api/v1/institutional/memberships/{id}/select` | [[../../03-subprojects/web/ui-catalog/morador/M2|M2]] | governanca-institucional | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/videos/{id}/view-progress` |
| POST | `/api/v1/institutional/polls/{poll_id}/responses` | [[../../03-subprojects/web/ui-catalog/morador/M7|M7]] | governanca-institucional | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/videos/{id}/view-progress` |
| GET | `/api/v1/institutional/timeline/{cond}` | [[../../03-subprojects/mobile/ui-catalog/morador/M6|m:M6]] | governanca-institucional | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/videos/{id}/playback` |
| GET | `/api/v1/institutional/timeline/{id}` | [[../../03-subprojects/mobile/ui-catalog/morador/M6|m:M6]], [[../../03-subprojects/web/ui-catalog/morador/M7|M7]] | governanca-institucional | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/videos/{id}/playback` |
| GET | `/api/v1/institutional/timeline/{id}/addenda` | [[../../03-subprojects/web/ui-catalog/morador/M7|M7]] | governanca-institucional | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/videos/{id}/playback` |
| POST | `/api/v1/institutional/timeline/{id}/read` | [[../../03-subprojects/mobile/ui-catalog/morador/M6|m:M6]], [[../../03-subprojects/web/ui-catalog/morador/M6|M6]] | governanca-institucional | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/videos/{id}/view-progress` |
| GET | `/api/v1/institutional/units/check` | [[../../03-subprojects/web/ui-catalog/morador/M4|M4]] | governanca-institucional | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/videos/{id}/playback` |
| POST | `/api/v1/media/avatars/upload` | [[../../03-subprojects/web/ui-catalog/onboarding/ONB-M2|ONB-M2]] | governanca-institucional | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/videos/{id}/view-progress` |
| POST | `/api/v1/media/upload` | [[../../03-subprojects/web/ui-catalog/empresa/E12|E12]], [[../../03-subprojects/web/ui-catalog/empresa/E9|E9]] | connect-me | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/videos/{id}/view-progress` |
| GET | `/api/v1/neighborhood/syndic/dashboard` | [[../../03-subprojects/web/ui-catalog/vizinhanca/VS1|VS1]] | vizinhanca | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/neighborhood/feed` |
| POST | `/api/v1/onboarding/complete` | [[../../03-subprojects/web/ui-catalog/onboarding/ONB-EFIN|ONB-EFIN]], [[../../03-subprojects/web/ui-catalog/onboarding/ONB-MFIN|ONB-MFIN]], [[../../03-subprojects/web/ui-catalog/onboarding/ONB-PFIN|ONB-PFIN]], [[../../03-subprojects/web/ui-catalog/onboarding/ONB-SFIN|ONB-SFIN]] | governanca-institucional | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/videos/{id}/view-progress` |
| PATCH | `/api/v1/onboarding/draft` | [[../../03-subprojects/web/ui-catalog/onboarding/ONB-E2|ONB-E2]], [[../../03-subprojects/web/ui-catalog/onboarding/ONB-E3|ONB-E3]], [[../../03-subprojects/web/ui-catalog/onboarding/ONB-E4|ONB-E4]], [[../../03-subprojects/web/ui-catalog/onboarding/ONB-E5|ONB-E5]], [[../../03-subprojects/web/ui-catalog/onboarding/ONB-E6|ONB-E6]], +8 | governanca-institucional | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/partners/{id}/palavra-do-dia` |
| POST | `/api/v1/onboarding/draft` | [[../../03-subprojects/web/ui-catalog/onboarding/ONB-00-boas-vindas|ONB-00-boas-vindas]], [[../../03-subprojects/web/ui-catalog/onboarding/ONB-01-identificacao-inicial|ONB-01-identificacao-inicial]] | governanca-institucional | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/videos/{id}/view-progress` |
| POST | `/api/v1/onboarding/terms/accept` | [[../../03-subprojects/web/ui-catalog/onboarding/ONB-E7|ONB-E7]], [[../../03-subprojects/web/ui-catalog/onboarding/ONB-M4|ONB-M4]], [[../../03-subprojects/web/ui-catalog/onboarding/ONB-P4|ONB-P4]], [[../../03-subprojects/web/ui-catalog/onboarding/ONB-S6|ONB-S6]] | governanca-institucional | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/videos/{id}/view-progress` |
| POST | `/api/v1/syndic/governance/acceptance` | [[../../03-subprojects/mobile/ui-catalog/sindico/S1|m:S1]], [[../../03-subprojects/web/ui-catalog/sindico/S1|S1]] | governanca-institucional | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/videos/{id}/view-progress` |
| GET | `/api/v1/syndic/governance/first-access` | [[../../03-subprojects/mobile/ui-catalog/sindico/S1|m:S1]], [[../../03-subprojects/web/ui-catalog/sindico/S1|S1]] | governanca-institucional | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/videos/{id}/playback` |
| GET | `/api/v1/utils/cep/{cep}` | [[../../03-subprojects/web/ui-catalog/sindico/S3|S3]], [[../../03-subprojects/web/ui-catalog/vizinhanca/VZ1|VZ1]] | governanca-institucional, vizinhanca | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/videos/{id}/playback` |
| GET | `/api/v1/utils/cep/{cep}/neighborhood` | [[../../03-subprojects/web/ui-catalog/banco-talentos/BT05|BT05]] | banco-talentos | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/videos/{id}/playback` |

### 🟠 HIGH — 140 endpoints

| Método | Path | Telas consumidoras | Sub-produto | Ação recomendada |
|---|---|---|---|---|
| GET | `/api/v1/categories` | [[../../03-subprojects/web/ui-catalog/onboarding/ONB-E4|ONB-E4]] | governanca-institucional | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/videos/{id}/playback` |
| GET | `/api/v1/commercial/closed-deals/own` | [[../../03-subprojects/web/ui-catalog/empresa/E7|E7]] | connect-me | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/videos/{id}/playback` |
| GET | `/api/v1/commercial/closed-deals/{id}` | [[../../03-subprojects/web/ui-catalog/empresa/E7|E7]] | connect-me | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/videos/{id}/playback` |
| POST | `/api/v1/commercial/closed-deals/{id}/company-sign` | [[../../03-subprojects/web/ui-catalog/empresa/E8|E8]] | connect-me | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/videos/{id}/view-progress` |
| GET | `/api/v1/commercial/closed-deals/{id}/signature-status` | [[../../03-subprojects/web/ui-catalog/empresa/E8|E8]] | connect-me | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/videos/{id}/playback` |
| GET | `/api/v1/commercial/connect-me/received` | [[../../03-subprojects/web/ui-catalog/empresa/E2|E2]] | connect-me | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/videos/{id}/playback` |
| POST | `/api/v1/commercial/connect-me/{id}/decline` | [[../../03-subprojects/web/ui-catalog/empresa/E3|E3]] | connect-me | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/videos/{id}/view-progress` |
| POST | `/api/v1/commercial/connect-me/{id}/interested` | [[../../03-subprojects/web/ui-catalog/empresa/E4|E4]] | connect-me | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/videos/{id}/view-progress` |
| POST | `/api/v1/commercial/deals/{id}/contract` | [[../../03-subprojects/web/ui-catalog/compliance/C8|C8]] | governanca-institucional | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/videos/{id}/view-progress` |
| GET | `/api/v1/commercial/evaluations/received` | [[../../03-subprojects/web/ui-catalog/empresa/E11|E11]] | connect-me | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/videos/{id}/playback` |
| PUT | `/api/v1/commercial/evaluations/{id}/visibility` | [[../../03-subprojects/web/ui-catalog/empresa/E11|E11]] | connect-me | Formalizar em `backend/requirements/*.md` ou mover UI para backlog |
| POST | `/api/v1/commercial/marketing-link` | [[../../03-subprojects/web/ui-catalog/empresa/E16|E16]] | connect-me | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/videos/{id}/view-progress` |
| GET | `/api/v1/commercial/marketing-link/received` | [[../../03-subprojects/web/ui-catalog/marketing/MK8|MK8]] | connect-me-agencia | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/videos/{id}/playback` |
| GET | `/api/v1/commercial/marketing-link/{id}` | [[../../03-subprojects/web/ui-catalog/empresa/E16|E16]] | connect-me | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/videos/{id}/playback` |
| POST | `/api/v1/commercial/marketing-link/{id}/attended` | [[../../03-subprojects/web/ui-catalog/marketing/MK8|MK8]] | connect-me-agencia | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/videos/{id}/view-progress` |
| POST | `/api/v1/commercial/marketing-link/{id}/view` | [[../../03-subprojects/web/ui-catalog/marketing/MK8|MK8]] | connect-me-agencia | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/videos/{id}/view-progress` |
| POST | `/api/v1/commercial/proposals` | [[../../03-subprojects/web/ui-catalog/empresa/E4|E4]] | connect-me | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/videos/{id}/view-progress` |
| GET | `/api/v1/commercial/proposals/own` | [[../../03-subprojects/web/ui-catalog/empresa/E5|E5]] | connect-me | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/videos/{id}/playback` |
| PUT | `/api/v1/commercial/proposals/{id}/draft` | [[../../03-subprojects/web/ui-catalog/empresa/E4|E4]] | connect-me | Formalizar em `backend/requirements/*.md` ou mover UI para backlog |
| POST | `/api/v1/commercial/proposals/{id}/submit` | [[../../03-subprojects/web/ui-catalog/empresa/E4|E4]] | connect-me | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/videos/{id}/view-progress` |
| GET | `/api/v1/commercial/terms/execution/current` | [[../../03-subprojects/web/ui-catalog/empresa/E8|E8]] | connect-me | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/videos/{id}/playback` |
| GET | `/api/v1/companies` | [[../../03-subprojects/web/ui-catalog/sindico/S19|S19]] | governanca-institucional | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/companies/{id}/reputation-breakdown` |
| GET | `/api/v1/compliance/annual-declaration/current` | [[../../03-subprojects/web/ui-catalog/sindico/S30|S30]] | governanca-institucional | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/videos/{id}/playback` |
| GET | `/api/v1/compliance/audit-trail` | [[../../03-subprojects/web/ui-catalog/compliance/C5|C5]] | governanca-institucional | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/videos/{id}/playback` |
| POST | `/api/v1/compliance/audit-trail/export` | [[../../03-subprojects/web/ui-catalog/compliance/C5|C5]] | governanca-institucional | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/videos/{id}/view-progress` |
| GET | `/api/v1/compliance/audit-trail/filters` | [[../../03-subprojects/web/ui-catalog/compliance/C5|C5]] | governanca-institucional | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/videos/{id}/playback` |
| GET | `/api/v1/compliance/conflict-of-interest` | [[../../03-subprojects/web/ui-catalog/compliance/C4|C4]] | governanca-institucional | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/videos/{id}/playback` |
| POST | `/api/v1/compliance/conflict-of-interest` | [[../../03-subprojects/web/ui-catalog/compliance/C4|C4]] | governanca-institucional | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/videos/{id}/view-progress` |
| GET | `/api/v1/compliance/conflict-of-interest/history` | [[../../03-subprojects/web/ui-catalog/compliance/C4|C4]] | governanca-institucional | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/videos/{id}/playback` |
| GET | `/api/v1/compliance/contracts` | [[../../03-subprojects/web/ui-catalog/compliance/C8|C8]] | governanca-institucional | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/videos/{id}/playback` |
| GET | `/api/v1/compliance/contracts/{id}` | [[../../03-subprojects/web/ui-catalog/compliance/C8|C8]] | governanca-institucional | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/videos/{id}/playback` |
| GET | `/api/v1/compliance/dashboard` | [[../../03-subprojects/web/ui-catalog/compliance/C1|C1]] | governanca-institucional | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/videos/{id}/playback` |
| POST | `/api/v1/compliance/declaration` | [[../../03-subprojects/web/ui-catalog/compliance/C2|C2]] | governanca-institucional | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/videos/{id}/view-progress` |
| GET | `/api/v1/compliance/declaration/status` | [[../../03-subprojects/web/ui-catalog/compliance/C2|C2]] | governanca-institucional | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/videos/{id}/playback` |
| GET | `/api/v1/compliance/declaration/template` | [[../../03-subprojects/web/ui-catalog/compliance/C2|C2]] | governanca-institucional | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/videos/{id}/playback` |
| GET | `/api/v1/compliance/dossier/eligibility` | [[../../03-subprojects/web/ui-catalog/compliance/C11|C11]] | governanca-institucional | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/videos/{id}/playback` |
| POST | `/api/v1/compliance/dossier/generate` | [[../../03-subprojects/web/ui-catalog/compliance/C11|C11]] | governanca-institucional | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/videos/{id}/view-progress` |
| GET | `/api/v1/compliance/dossier/history` | [[../../03-subprojects/web/ui-catalog/compliance/C11|C11]] | governanca-institucional | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/videos/{id}/playback` |
| GET | `/api/v1/compliance/dossier/jobs/{id}` | [[../../03-subprojects/web/ui-catalog/compliance/C11|C11]] | governanca-institucional | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/videos/{id}/playback` |
| GET | `/api/v1/compliance/lgpd-logs` | [[../../03-subprojects/web/ui-catalog/compliance/C9|C9]] | governanca-institucional | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/videos/{id}/playback` |
| POST | `/api/v1/compliance/lgpd-logs/export` | [[../../03-subprojects/web/ui-catalog/compliance/C9|C9]] | governanca-institucional | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/videos/{id}/view-progress` |
| GET | `/api/v1/compliance/lgpd-logs/{id}` | [[../../03-subprojects/web/ui-catalog/compliance/C9|C9]] | governanca-institucional | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/videos/{id}/playback` |
| GET | `/api/v1/compliance/locked-records` | [[../../03-subprojects/web/ui-catalog/compliance/C6|C6]] | governanca-institucional | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/videos/{id}/playback` |
| GET | `/api/v1/compliance/pendencies` | [[../../03-subprojects/web/ui-catalog/compliance/C1|C1]] | governanca-institucional | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/videos/{id}/playback` |
| GET | `/api/v1/compliance/records/{id}/addenda` | [[../../03-subprojects/web/ui-catalog/compliance/C6|C6]] | governanca-institucional | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/videos/{id}/playback` |
| POST | `/api/v1/compliance/records/{id}/addenda` | [[../../03-subprojects/web/ui-catalog/compliance/C6|C6]] | governanca-institucional | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/videos/{id}/view-progress` |
| GET | `/api/v1/compliance/responsibility-term/current` | [[../../03-subprojects/web/ui-catalog/sindico/S29|S29]] | governanca-institucional | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/videos/{id}/playback` |
| GET | `/api/v1/compliance/risk-map` | [[../../03-subprojects/web/ui-catalog/compliance/C7|C7]] | governanca-institucional | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/videos/{id}/playback` |
| GET | `/api/v1/compliance/risk-map/overdue` | [[../../03-subprojects/web/ui-catalog/compliance/C7|C7]] | governanca-institucional | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/videos/{id}/playback` |
| GET | `/api/v1/compliance/score` | [[../../03-subprojects/web/ui-catalog/compliance/C10|C10]] | governanca-institucional | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/videos/{id}/playback` |
| GET | `/api/v1/compliance/score/history` | [[../../03-subprojects/web/ui-catalog/compliance/C10|C10]] | governanca-institucional | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/videos/{id}/playback` |
| GET | `/api/v1/compliance/signatures` | [[../../03-subprojects/web/ui-catalog/compliance/C3|C3]] | governanca-institucional | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/videos/{id}/playback` |
| POST | `/api/v1/compliance/signatures/sign` | [[../../03-subprojects/web/ui-catalog/compliance/C3|C3]] | governanca-institucional | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/videos/{id}/view-progress` |
| POST | `/api/v1/compliance/signatures/{user_id}/resend` | [[../../03-subprojects/web/ui-catalog/compliance/C3|C3]] | governanca-institucional | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/videos/{id}/view-progress` |
| GET | `/api/v1/connect-me` | [[../../03-subprojects/web/ui-catalog/sindico/S18|S18]], [[../../03-subprojects/web/ui-catalog/sindico/S20|S20]] | governanca-institucional | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/videos/{id}/playback` |
| POST | `/api/v1/connect-me` | [[../../03-subprojects/web/ui-catalog/sindico/S19|S19]] | governanca-institucional | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/connect-me/{id}/interest` |
| GET | `/api/v1/connect-me/me/summary` | [[../../03-subprojects/web/ui-catalog/sindico/S18|S18]] | governanca-institucional | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/videos/{id}/playback` |
| POST | `/api/v1/connect-me/{id}/respond` | [[../../03-subprojects/web/ui-catalog/sindico/S20|S20]] | governanca-institucional | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/connect-me/{id}/interest` |
| GET | `/api/v1/content/videos` | [[../../03-subprojects/web/ui-catalog/empresa/E12|E12]], [[../../03-subprojects/web/ui-catalog/marketing/MK3|MK3]], [[../../03-subprojects/web/ui-catalog/marketing/MK5|MK5]] | connect-me, connect-me-agencia | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/videos/{id}/playback` |
| POST | `/api/v1/content/videos` | [[../../03-subprojects/web/ui-catalog/marketing/MK4|MK4]] | connect-me-agencia | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/videos/{id}/view-progress` |
| PUT | `/api/v1/content/videos/draft` | [[../../03-subprojects/web/ui-catalog/marketing/MK4|MK4]] | connect-me-agencia | Formalizar em `backend/requirements/*.md` ou mover UI para backlog |
| PUT | `/api/v1/content/videos/{id}` | [[../../03-subprojects/web/ui-catalog/marketing/MK5|MK5]] | connect-me-agencia | Formalizar em `backend/requirements/*.md` ou mover UI para backlog |
| POST | `/api/v1/content/videos/{id}/hide` | [[../../03-subprojects/web/ui-catalog/marketing/MK5|MK5]] | connect-me-agencia | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/videos/{id}/view-progress` |
| POST | `/api/v1/content/videos/{id}/restore` | [[../../03-subprojects/web/ui-catalog/marketing/MK5|MK5]] | connect-me-agencia | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/videos/{id}/view-progress` |
| POST | `/api/v1/enterprise/active-tenant` | [[../../03-subprojects/web/ui-catalog/empresa/E1|E1]] | connect-me | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/videos/{id}/view-progress` |
| GET | `/api/v1/enterprise/agencies` | [[../../03-subprojects/web/ui-catalog/empresa/E14|E14]] | connect-me | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/videos/{id}/playback` |
| POST | `/api/v1/enterprise/agencies/invite` | [[../../03-subprojects/web/ui-catalog/empresa/E14|E14]] | connect-me | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/videos/{id}/view-progress` |
| GET | `/api/v1/enterprise/agencies/{id}/permissions` | [[../../03-subprojects/web/ui-catalog/empresa/E14|E14]] | connect-me | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/videos/{id}/playback` |
| POST | `/api/v1/enterprise/agencies/{id}/revoke` | [[../../03-subprojects/web/ui-catalog/empresa/E14|E14]] | connect-me | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/videos/{id}/view-progress` |
| GET | `/api/v1/enterprise/compliance/addendums` | [[../../03-subprojects/web/ui-catalog/empresa/E15|E15]] | connect-me | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/videos/{id}/playback` |
| POST | `/api/v1/enterprise/compliance/declarations/{type}/sign` | [[../../03-subprojects/web/ui-catalog/empresa/E15|E15]] | connect-me | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/videos/{id}/view-progress` |
| GET | `/api/v1/enterprise/compliance/lgpd-logs` | [[../../03-subprojects/web/ui-catalog/empresa/E15|E15]] | connect-me | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/videos/{id}/playback` |
| GET | `/api/v1/enterprise/compliance/status` | [[../../03-subprojects/web/ui-catalog/empresa/E15|E15]] | connect-me | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/videos/{id}/playback` |
| GET | `/api/v1/enterprise/compliance/warranties` | [[../../03-subprojects/web/ui-catalog/empresa/E15|E15]] | connect-me | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/videos/{id}/playback` |
| GET | `/api/v1/enterprise/dashboard` | [[../../03-subprojects/web/ui-catalog/empresa/E1|E1]] | connect-me | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/videos/{id}/playback` |
| POST | `/api/v1/enterprise/dashboard/export` | [[../../03-subprojects/web/ui-catalog/empresa/E10|E10]] | connect-me | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/videos/{id}/view-progress` |
| GET | `/api/v1/enterprise/dashboard/metrics` | [[../../03-subprojects/web/ui-catalog/empresa/E10|E10]] | connect-me | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/videos/{id}/playback` |
| GET | `/api/v1/enterprise/profile` | [[../../03-subprojects/web/ui-catalog/empresa/E12|E12]] | connect-me | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/videos/{id}/playback` |
| PUT | `/api/v1/enterprise/profile` | [[../../03-subprojects/web/ui-catalog/empresa/E12|E12]] | connect-me | Formalizar em `backend/requirements/*.md` ou mover UI para backlog |
| GET | `/api/v1/enterprise/users` | [[../../03-subprojects/web/ui-catalog/empresa/E13|E13]] | connect-me | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/videos/{id}/playback` |
| POST | `/api/v1/enterprise/users/invite` | [[../../03-subprojects/web/ui-catalog/empresa/E13|E13]] | connect-me | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/videos/{id}/view-progress` |
| DELETE | `/api/v1/enterprise/users/{id}` | [[../../03-subprojects/web/ui-catalog/empresa/E13|E13]] | connect-me | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/proxies/{id}` |
| POST | `/api/v1/enterprise/users/{id}/resend-invite` | [[../../03-subprojects/web/ui-catalog/empresa/E13|E13]] | connect-me | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/videos/{id}/view-progress` |
| PUT | `/api/v1/enterprise/users/{id}/role` | [[../../03-subprojects/web/ui-catalog/empresa/E13|E13]] | connect-me | Formalizar em `backend/requirements/*.md` ou mover UI para backlog |
| GET | `/api/v1/marketing/dashboard` | [[../../03-subprojects/web/ui-catalog/marketing/MK1|MK1]] | connect-me-agencia | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/videos/{id}/playback` |
| GET | `/api/v1/marketing/dashboard/consolidated` | [[../../03-subprojects/web/ui-catalog/marketing/MK7|MK7]] | connect-me-agencia | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/videos/{id}/playback` |
| POST | `/api/v1/marketing/dashboard/export` | [[../../03-subprojects/web/ui-catalog/marketing/MK7|MK7]] | connect-me-agencia | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/videos/{id}/view-progress` |
| GET | `/api/v1/marketing/empresas` | [[../../03-subprojects/web/ui-catalog/marketing/MK2|MK2]] | connect-me-agencia | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/videos/{id}/playback` |
| GET | `/api/v1/marketing/empresas/{id}/context` | [[../../03-subprojects/web/ui-catalog/marketing/MK3|MK3]] | connect-me-agencia | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/videos/{id}/playback` |
| GET | `/api/v1/marketing/empresas/{id}/metrics` | [[../../03-subprojects/web/ui-catalog/marketing/MK6|MK6]] | connect-me-agencia | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/videos/{id}/playback` |
| GET | `/api/v1/marketing/empresas/{id}/metrics/timeseries` | [[../../03-subprojects/web/ui-catalog/marketing/MK6|MK6]] | connect-me-agencia | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/videos/{id}/playback` |
| GET | `/api/v1/marketing/notifications` | [[../../03-subprojects/web/ui-catalog/marketing/MK1|MK1]] | connect-me-agencia | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/videos/{id}/playback` |
| GET | `/api/v1/neighborhood/activations` | [[../../03-subprojects/web/ui-catalog/vizinhanca/VE6|VE6]], [[../../03-subprojects/web/ui-catalog/vizinhanca/VM7|VM7]], [[../../03-subprojects/web/ui-catalog/vizinhanca/VS10|VS10]] | vizinhanca | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/neighborhood/feed` |
| GET | `/api/v1/neighborhood/coupons/{code}` | [[../../03-subprojects/web/ui-catalog/vizinhanca/VS5|VS5]] | vizinhanca | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/neighborhood/feed` |
| GET | `/api/v1/neighborhood/exclusive-benefits` | [[../../03-subprojects/web/ui-catalog/vizinhanca/VM5|VM5]], [[../../03-subprojects/web/ui-catalog/vizinhanca/VS8|VS8]] | vizinhanca | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/neighborhood/feed` |
| POST | `/api/v1/neighborhood/exclusive-benefits` | [[../../03-subprojects/web/ui-catalog/vizinhanca/VS7|VS7]] | vizinhanca | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/videos/{id}/view-progress` |
| PATCH | `/api/v1/neighborhood/exclusive-benefits/{id}` | [[../../03-subprojects/web/ui-catalog/vizinhanca/VS8|VS8]] | vizinhanca | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/partners/{id}/palavra-do-dia` |
| POST | `/api/v1/neighborhood/exclusive-benefits/{id}/end` | [[../../03-subprojects/web/ui-catalog/vizinhanca/VS8|VS8]] | vizinhanca | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/videos/{id}/view-progress` |
| GET | `/api/v1/neighborhood/partners` | [[../../03-subprojects/web/ui-catalog/vizinhanca/VE2|VE2]], [[../../03-subprojects/web/ui-catalog/vizinhanca/VS7|VS7]] | vizinhanca | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/neighborhood/feed` |
| POST | `/api/v1/neighborhood/partners` | [[../../03-subprojects/web/ui-catalog/vizinhanca/VZ1|VZ1]] | vizinhanca | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/videos/{id}/view-progress` |
| GET | `/api/v1/neighborhood/partners/filters` | [[../../03-subprojects/web/ui-catalog/vizinhanca/VS2|VS2]] | vizinhanca | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/neighborhood/feed` |
| POST | `/api/v1/neighborhood/partners/invite` | [[../../03-subprojects/web/ui-catalog/vizinhanca/VS6|VS6]] | vizinhanca | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/videos/{id}/view-progress` |
| POST | `/api/v1/neighborhood/partners/invite/{id}/resend` | [[../../03-subprojects/web/ui-catalog/vizinhanca/VS9|VS9]] | vizinhanca | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/videos/{id}/view-progress` |
| GET | `/api/v1/neighborhood/partners/invited` | [[../../03-subprojects/web/ui-catalog/vizinhanca/VS9|VS9]] | vizinhanca | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/neighborhood/feed` |
| GET | `/api/v1/neighborhood/partners/me` | [[../../03-subprojects/web/ui-catalog/vizinhanca/VZ2|VZ2]] | vizinhanca | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/neighborhood/feed` |
| PATCH | `/api/v1/neighborhood/partners/me` | [[../../03-subprojects/web/ui-catalog/vizinhanca/VZ2|VZ2]] | vizinhanca | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/partners/{id}/palavra-do-dia` |
| GET | `/api/v1/neighborhood/partners/me/metrics` | [[../../03-subprojects/web/ui-catalog/vizinhanca/VZ5|VZ5]] | vizinhanca | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/neighborhood/feed` |
| GET | `/api/v1/neighborhood/partners/me/metrics/engagement-weekly` | [[../../03-subprojects/web/ui-catalog/vizinhanca/VZ5|VZ5]] | vizinhanca | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/neighborhood/feed` |
| GET | `/api/v1/neighborhood/partners/me/metrics/top-promotions` | [[../../03-subprojects/web/ui-catalog/vizinhanca/VZ5|VZ5]] | vizinhanca | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/neighborhood/feed` |
| GET | `/api/v1/neighborhood/partners/recommended` | [[../../03-subprojects/web/ui-catalog/vizinhanca/VM6|VM6]] | vizinhanca | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/neighborhood/feed` |
| GET | `/api/v1/neighborhood/partners/{id}` | [[../../03-subprojects/web/ui-catalog/vizinhanca/VE3|VE3]], [[../../03-subprojects/web/ui-catalog/vizinhanca/VM2|VM2]], [[../../03-subprojects/web/ui-catalog/vizinhanca/VS3|VS3]] | vizinhanca | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/neighborhood/feed` |
| POST | `/api/v1/neighborhood/partners/{id}/contact-click` | [[../../03-subprojects/web/ui-catalog/vizinhanca/VM2|VM2]], [[../../03-subprojects/web/ui-catalog/vizinhanca/VS3|VS3]] | vizinhanca | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/videos/{id}/view-progress` |
| POST | `/api/v1/neighborhood/partners/{id}/recommend` | [[../../03-subprojects/web/ui-catalog/vizinhanca/VS3|VS3]], [[../../03-subprojects/web/ui-catalog/vizinhanca/VS9|VS9]] | vizinhanca | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/videos/{id}/view-progress` |
| GET | `/api/v1/neighborhood/promotions` | [[../../03-subprojects/web/ui-catalog/vizinhanca/VZ4|VZ4]], [[../../03-subprojects/web/ui-catalog/vizinhanca/VZ6|VZ6]] | vizinhanca | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/neighborhood/feed` |
| POST | `/api/v1/neighborhood/promotions` | [[../../03-subprojects/web/ui-catalog/vizinhanca/VZ3|VZ3]] | vizinhanca | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/videos/{id}/view-progress` |
| GET | `/api/v1/neighborhood/promotions/{id}` | [[../../03-subprojects/web/ui-catalog/vizinhanca/VE4|VE4]], [[../../03-subprojects/web/ui-catalog/vizinhanca/VM3|VM3]], [[../../03-subprojects/web/ui-catalog/vizinhanca/VS4|VS4]] | vizinhanca | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/neighborhood/feed` |
| PATCH | `/api/v1/neighborhood/promotions/{id}` | [[../../03-subprojects/web/ui-catalog/vizinhanca/VZ4|VZ4]] | vizinhanca | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/partners/{id}/palavra-do-dia` |
| POST | `/api/v1/neighborhood/promotions/{id}/activate` | [[../../03-subprojects/web/ui-catalog/vizinhanca/VE4|VE4]], [[../../03-subprojects/web/ui-catalog/vizinhanca/VS4|VS4]] | vizinhanca | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/videos/{id}/view-progress` |
| POST | `/api/v1/neighborhood/promotions/{id}/coupon` | [[../../03-subprojects/web/ui-catalog/vizinhanca/VS5|VS5]] | vizinhanca | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/videos/{id}/view-progress` |
| POST | `/api/v1/neighborhood/promotions/{id}/end` | [[../../03-subprojects/web/ui-catalog/vizinhanca/VZ4|VZ4]] | vizinhanca | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/videos/{id}/view-progress` |
| GET | `/api/v1/neighborhood/syndic/metrics` | [[../../03-subprojects/web/ui-catalog/vizinhanca/VS10|VS10]] | vizinhanca | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/neighborhood/feed` |
| POST | `/api/v1/reputation/assessment` | [[../../03-subprojects/mobile/ui-catalog/morador/M12|m:M12]] | governanca-institucional | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/videos/{id}/view-progress` |
| GET | `/api/v1/reputation/assessment/pending` | [[../../03-subprojects/mobile/ui-catalog/morador/M12|m:M12]] | governanca-institucional | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/videos/{id}/playback` |
| GET | `/api/v1/talents/curriculum/me` | [[../../03-subprojects/web/ui-catalog/banco-talentos/BT01|BT01]], [[../../03-subprojects/web/ui-catalog/banco-talentos/BT11|BT11]] | banco-talentos | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/talents` |
| GET | `/api/v1/talents/curriculum/me/areas` | [[../../03-subprojects/web/ui-catalog/banco-talentos/BT04|BT04]] | banco-talentos | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/talents` |
| PUT | `/api/v1/talents/curriculum/me/areas` | [[../../03-subprojects/web/ui-catalog/banco-talentos/BT04|BT04]] | banco-talentos | Formalizar em `backend/requirements/*.md` ou mover UI para backlog |
| PATCH | `/api/v1/talents/curriculum/me/availability` | [[../../03-subprojects/web/ui-catalog/banco-talentos/BT06|BT06]] | banco-talentos | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/partners/{id}/palavra-do-dia` |
| PUT | `/api/v1/talents/curriculum/me/education` | [[../../03-subprojects/web/ui-catalog/banco-talentos/BT09|BT09]] | banco-talentos | Formalizar em `backend/requirements/*.md` ou mover UI para backlog |
| GET | `/api/v1/talents/curriculum/me/experiences` | [[../../03-subprojects/web/ui-catalog/banco-talentos/BT08|BT08]] | banco-talentos | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/talents` |
| PATCH | `/api/v1/talents/curriculum/me/personal` | [[../../03-subprojects/web/ui-catalog/banco-talentos/BT05|BT05]] | banco-talentos | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/partners/{id}/palavra-do-dia` |
| GET | `/api/v1/talents/curriculum/me/profile` | [[../../03-subprojects/web/ui-catalog/banco-talentos/BT03|BT03]] | banco-talentos | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/talents` |
| PATCH | `/api/v1/talents/curriculum/me/profile` | [[../../03-subprojects/web/ui-catalog/banco-talentos/BT03|BT03]] | banco-talentos | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/partners/{id}/palavra-do-dia` |
| PATCH | `/api/v1/talents/curriculum/me/salary` | [[../../03-subprojects/web/ui-catalog/banco-talentos/BT07|BT07]] | banco-talentos | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/partners/{id}/palavra-do-dia` |
| PATCH | `/api/v1/talents/curriculum/me/self-report` | [[../../03-subprojects/web/ui-catalog/banco-talentos/BT09|BT09]] | banco-talentos | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/partners/{id}/palavra-do-dia` |
| POST | `/api/v1/talents/curriculum/me/submit` | [[../../03-subprojects/web/ui-catalog/banco-talentos/BT11|BT11]] | banco-talentos | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/videos/{id}/view-progress` |
| GET | `/api/v1/talents/curriculum/me/video` | [[../../03-subprojects/web/ui-catalog/banco-talentos/BT02|BT02]] | banco-talentos | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/talents` |
| GET | `/api/v1/talents/lgpd/template` | [[../../03-subprojects/web/ui-catalog/banco-talentos/BT10|BT10]] | banco-talentos | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/talents` |
| GET | `/api/v1/videos/institutional/lock-status` | [[../../03-subprojects/web/ui-catalog/onboarding/ONB-S5|ONB-S5]] | governanca-institucional | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/videos/{id}/playback` |
| POST | `/api/v1/videos/institutional/upload` | [[../../03-subprojects/web/ui-catalog/onboarding/ONB-E6|ONB-E6]], [[../../03-subprojects/web/ui-catalog/onboarding/ONB-S5|ONB-S5]] | governanca-institucional | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/videos/{id}/view-progress` |
| POST | `/api/v1/videos/technical/upload` | [[../../03-subprojects/web/ui-catalog/onboarding/ONB-E6|ONB-E6]] | governanca-institucional | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/videos/{id}/view-progress` |

### 🟡 MEDIUM — 81 endpoints

| Método | Path | Telas consumidoras | Sub-produto | Ação recomendada |
|---|---|---|---|---|
| GET | `/api/v1/academy/certificates/by-enrollment/{eid}` | [[../../03-subprojects/web/ui-catalog/lms/LMS5|LMS5]] | academy/lms | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/videos/{id}/playback` |
| POST | `/api/v1/academy/courses/{cid}/lessons/{lid}/complete` | [[../../03-subprojects/web/ui-catalog/lms/LMS4|LMS4]] | academy/lms | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/videos/{id}/view-progress` |
| GET | `/api/v1/academy/courses/{id}` | [[../../03-subprojects/web/ui-catalog/lms/LMS3|LMS3]] | academy/lms | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/videos/{id}/playback` |
| POST | `/api/v1/academy/courses/{id}/enroll` | [[../../03-subprojects/web/ui-catalog/lms/LMS3|LMS3]] | academy/lms | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/videos/{id}/view-progress` |
| GET | `/api/v1/academy/dashboard` | [[../../03-subprojects/web/ui-catalog/lms/LMS1|LMS1]] | academy/lms | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/videos/{id}/playback` |
| GET | `/api/v1/academy/forum` | [[../../03-subprojects/web/ui-catalog/lms/LMS11|LMS11]] | academy/lms | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/videos/{id}/playback` |
| GET | `/api/v1/academy/forum/{id}` | [[../../03-subprojects/web/ui-catalog/lms/LMS13|LMS13]] | academy/lms | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/videos/{id}/playback` |
| POST | `/api/v1/academy/forum/{id}/like` | [[../../03-subprojects/web/ui-catalog/lms/LMS13|LMS13]] | academy/lms | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/videos/{id}/view-progress` |
| POST | `/api/v1/academy/forum/{id}/reply` | [[../../03-subprojects/web/ui-catalog/lms/LMS13|LMS13]] | academy/lms | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/videos/{id}/view-progress` |
| POST | `/api/v1/academy/forum/{id}/report` | [[../../03-subprojects/web/ui-catalog/lms/LMS13|LMS13]] | academy/lms | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/videos/{id}/view-progress` |
| POST | `/api/v1/academy/forum/{id}/resolve` | [[../../03-subprojects/web/ui-catalog/lms/LMS13|LMS13]] | academy/lms | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/videos/{id}/view-progress` |
| POST | `/api/v1/academy/forum/{id}/subscribe` | [[../../03-subprojects/web/ui-catalog/lms/LMS13|LMS13]] | academy/lms | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/videos/{id}/view-progress` |
| GET | `/api/v1/academy/lessons/{id}` | [[../../03-subprojects/web/ui-catalog/lms/LMS4|LMS4]] | academy/lms | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/videos/{id}/playback` |
| GET | `/api/v1/academy/lives` | [[../../03-subprojects/web/ui-catalog/lms/LMS8|LMS8]] | academy/lms | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/videos/{id}/playback` |
| GET | `/api/v1/academy/lives/replays` | [[../../03-subprojects/web/ui-catalog/lms/LMS10|LMS10]] | academy/lms | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/videos/{id}/playback` |
| GET | `/api/v1/academy/lives/{id}` | [[../../03-subprojects/web/ui-catalog/lms/LMS9|LMS9]] | academy/lms | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/videos/{id}/playback` |
| POST | `/api/v1/academy/lives/{id}/questions` | [[../../03-subprojects/web/ui-catalog/lms/LMS9|LMS9]] | academy/lms | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/videos/{id}/view-progress` |
| POST | `/api/v1/academy/lives/{id}/remind` | [[../../03-subprojects/web/ui-catalog/lms/LMS8|LMS8]] | academy/lms | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/videos/{id}/view-progress` |
| GET | `/api/v1/academy/lives/{id}/replay` | [[../../03-subprojects/web/ui-catalog/lms/LMS10|LMS10]] | academy/lms | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/videos/{id}/playback` |
| GET | `/api/v1/academy/paths/recommended` | [[../../03-subprojects/web/ui-catalog/lms/LMS1|LMS1]] | academy/lms | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/videos/{id}/playback` |
| GET | `/api/v1/academy/trainings` | [[../../03-subprojects/web/ui-catalog/lms/LMS6|LMS6]] | academy/lms | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/videos/{id}/playback` |
| GET | `/api/v1/academy/trainings/{id}` | [[../../03-subprojects/web/ui-catalog/lms/LMS7|LMS7]] | academy/lms | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/videos/{id}/playback` |
| POST | `/api/v1/academy/trainings/{id}/complete` | [[../../03-subprojects/web/ui-catalog/lms/LMS7|LMS7]] | academy/lms | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/videos/{id}/view-progress` |
| PATCH | `/api/v1/assemblies/{id}` | [[../../03-subprojects/web/ui-catalog/assembleia/ASM-CFG|ASM-CFG]] | assembleia | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/partners/{id}/palavra-do-dia` |
| GET | `/api/v1/assemblies/{id}/agenda` | [[../../03-subprojects/web/ui-catalog/assembleia/ASM-PAUTA|ASM-PAUTA]] | assembleia | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/assemblies/{id}/transparency-score` |
| POST | `/api/v1/assemblies/{id}/agenda` | [[../../03-subprojects/web/ui-catalog/assembleia/ASM-PAUTA|ASM-PAUTA]] | assembleia | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/assemblies/{id}/agenda/publish` |
| PATCH | `/api/v1/assemblies/{id}/agenda/reorder` | [[../../03-subprojects/web/ui-catalog/assembleia/ASM-PAUTA|ASM-PAUTA]] | assembleia | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/partners/{id}/palavra-do-dia` |
| DELETE | `/api/v1/assemblies/{id}/agenda/{item_id}` | [[../../03-subprojects/web/ui-catalog/assembleia/ASM-PAUTA|ASM-PAUTA]] | assembleia | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/proxies/{id}` |
| POST | `/api/v1/assemblies/{id}/agenda/{item}/close-vote` | [[../../03-subprojects/web/ui-catalog/assembleia/ASM-PAINEL-SIND|ASM-PAINEL-SIND]] | assembleia | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/assemblies/{id}/agenda/publish` |
| POST | `/api/v1/assemblies/{id}/agenda/{item}/homologate` | [[../../03-subprojects/web/ui-catalog/assembleia/ASM-HOML|ASM-HOML]], [[../../03-subprojects/web/ui-catalog/assembleia/ASM-PAINEL-SIND|ASM-PAINEL-SIND]] | assembleia | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/assemblies/{id}/agenda/publish` |
| POST | `/api/v1/assemblies/{id}/agenda/{item}/open` | [[../../03-subprojects/web/ui-catalog/assembleia/ASM-PAINEL-SIND|ASM-PAINEL-SIND]] | assembleia | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/assemblies/{id}/agenda/publish` |
| POST | `/api/v1/assemblies/{id}/agenda/{item}/open-vote` | [[../../03-subprojects/web/ui-catalog/assembleia/ASM-PAINEL-SIND|ASM-PAINEL-SIND]] | assembleia | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/assemblies/{id}/agenda/publish` |
| POST | `/api/v1/assemblies/{id}/agenda/{item}/vote` | [[../../03-subprojects/web/ui-catalog/assembleia/ASM-VOTO|ASM-VOTO]] | assembleia | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/assemblies/{id}/agenda/publish` |
| GET | `/api/v1/assemblies/{id}/agenda/{item}/votes` | [[../../03-subprojects/web/ui-catalog/assembleia/ASM-VOTO|ASM-VOTO]] | assembleia | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/assemblies/{id}/transparency-score` |
| GET | `/api/v1/assemblies/{id}/attendance` | [[../../03-subprojects/web/ui-catalog/assembleia/ASM-CHKIN|ASM-CHKIN]] | assembleia | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/assemblies/{id}/transparency-score` |
| POST | `/api/v1/assemblies/{id}/board` | [[../../03-subprojects/web/ui-catalog/assembleia/ASM-PAINEL-SIND|ASM-PAINEL-SIND]] | assembleia | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/assemblies/{id}/speech-queue/{id}/approve` |
| POST | `/api/v1/assemblies/{id}/checkin` | [[../../03-subprojects/web/ui-catalog/assembleia/ASM-CHKIN|ASM-CHKIN]] | assembleia | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/assemblies/{id}/speech-queue/{id}/approve` |
| POST | `/api/v1/assemblies/{id}/checkin/assisted` | [[../../03-subprojects/web/ui-catalog/assembleia/ASM-CHKIN|ASM-CHKIN]] | assembleia | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/assemblies/{id}/speech-queue/{id}/approve` |
| POST | `/api/v1/assemblies/{id}/cience` | [[../../03-subprojects/web/ui-catalog/assembleia/ASM-CIEN|ASM-CIEN]] | assembleia | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/assemblies/{id}/speech-queue/{id}/approve` |
| POST | `/api/v1/assemblies/{id}/cience/remind` | [[../../03-subprojects/web/ui-catalog/assembleia/ASM-CIEN|ASM-CIEN]] | assembleia | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/assemblies/{id}/speech-queue/{id}/approve` |
| GET | `/api/v1/assemblies/{id}/cience/report` | [[../../03-subprojects/web/ui-catalog/assembleia/ASM-CIEN|ASM-CIEN]] | assembleia | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/assemblies/{id}/transparency-score` |
| GET | `/api/v1/assemblies/{id}/close/preflight` | [[../../03-subprojects/web/ui-catalog/assembleia/ASM-ENCER|ASM-ENCER]] | assembleia | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/assemblies/{id}/transparency-score` |
| POST | `/api/v1/assemblies/{id}/delinquency/imports/{id}/commit` | [[../../03-subprojects/web/ui-catalog/assembleia/ASM-INAD|ASM-INAD]] | assembleia | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/assemblies/{id}/speech-queue/{id}/approve` |
| POST | `/api/v1/assemblies/{id}/delinquency/upload` | [[../../03-subprojects/web/ui-catalog/assembleia/ASM-INAD|ASM-INAD]] | assembleia | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/assemblies/{id}/speech-queue/{id}/approve` |
| POST | `/api/v1/assemblies/{id}/delinquency/{unit_id}/override` | [[../../03-subprojects/web/ui-catalog/assembleia/ASM-INAD|ASM-INAD]] | assembleia | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/assemblies/{id}/speech-queue/{id}/approve` |
| PUT | `/api/v1/assemblies/{id}/edital` | [[../../03-subprojects/web/ui-catalog/assembleia/ASM-EDITAL|ASM-EDITAL]] | assembleia | Formalizar em `backend/requirements/*.md` ou mover UI para backlog |
| POST | `/api/v1/assemblies/{id}/edital/generate` | [[../../03-subprojects/web/ui-catalog/assembleia/ASM-EDITAL|ASM-EDITAL]] | assembleia | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/assemblies/{id}/edital` |
| POST | `/api/v1/assemblies/{id}/edital/upload` | [[../../03-subprojects/web/ui-catalog/assembleia/ASM-EDITAL|ASM-EDITAL]] | assembleia | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/assemblies/{id}/edital` |
| POST | `/api/v1/assemblies/{id}/minutes/publish` | [[../../03-subprojects/web/ui-catalog/assembleia/ASM-ENCER|ASM-ENCER]] | assembleia | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/assemblies/{id}/speech-queue/{id}/approve` |
| POST | `/api/v1/assemblies/{id}/notify` | [[../../03-subprojects/web/ui-catalog/assembleia/ASM-PUB|ASM-PUB]] | assembleia | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/assemblies/{id}/speech-queue/{id}/approve` |
| GET | `/api/v1/assemblies/{id}/poll/stats` | [[../../03-subprojects/web/ui-catalog/assembleia/ASM-ENQP|ASM-ENQP]] | assembleia | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/assemblies/{id}/transparency-score` |
| POST | `/api/v1/assemblies/{id}/poll/{item_id}/answer` | [[../../03-subprojects/web/ui-catalog/assembleia/ASM-ENQP|ASM-ENQP]] | assembleia | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/assemblies/{id}/speech-queue/{id}/approve` |
| POST | `/api/v1/assemblies/{id}/presentation/show` | [[../../03-subprojects/web/ui-catalog/assembleia/ASM-TELAO|ASM-TELAO]] | assembleia | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/assemblies/{id}/speech-queue/{id}/approve` |
| POST | `/api/v1/assemblies/{id}/presentation/upload` | [[../../03-subprojects/web/ui-catalog/assembleia/ASM-TELAO|ASM-TELAO]] | assembleia | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/assemblies/{id}/speech-queue/{id}/approve` |
| GET | `/api/v1/assemblies/{id}/proxies` | [[../../03-subprojects/web/ui-catalog/assembleia/ASM-PROC-CAD|ASM-PROC-CAD]], [[../../03-subprojects/web/ui-catalog/assembleia/ASM-PROC-VAL|ASM-PROC-VAL]] | assembleia | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/assemblies/{id}/transparency-score` |
| POST | `/api/v1/assemblies/{id}/proxies` | [[../../03-subprojects/web/ui-catalog/assembleia/ASM-PROC-CAD|ASM-PROC-CAD]] | assembleia | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/assemblies/{id}/speech-queue/{id}/approve` |
| DELETE | `/api/v1/assemblies/{id}/proxies/{proxy_id}` | [[../../03-subprojects/web/ui-catalog/assembleia/ASM-PROC-CAD|ASM-PROC-CAD]] | assembleia | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/proxies/{id}` |
| POST | `/api/v1/assemblies/{id}/proxies/{proxy_id}/approve` | [[../../03-subprojects/web/ui-catalog/assembleia/ASM-PROC-VAL|ASM-PROC-VAL]] | assembleia | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/assemblies/{id}/speech-queue/{id}/approve` |
| POST | `/api/v1/assemblies/{id}/proxies/{proxy_id}/reject` | [[../../03-subprojects/web/ui-catalog/assembleia/ASM-PROC-VAL|ASM-PROC-VAL]] | assembleia | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/assemblies/{id}/speech-queue/{id}/approve` |
| POST | `/api/v1/assemblies/{id}/publish` | [[../../03-subprojects/web/ui-catalog/assembleia/ASM-PUB|ASM-PUB]] | assembleia | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/assemblies/{id}/speech-queue/{id}/approve` |
| POST | `/api/v1/assemblies/{id}/queue/lower` | [[../../03-subprojects/web/ui-catalog/assembleia/ASM-FILA-FALA|ASM-FILA-FALA]] | assembleia | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/assemblies/{id}/speech-queue/{id}/approve` |
| POST | `/api/v1/assemblies/{id}/queue/raise` | [[../../03-subprojects/web/ui-catalog/assembleia/ASM-FILA-FALA|ASM-FILA-FALA]] | assembleia | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/assemblies/{id}/speech-queue/{id}/approve` |
| POST | `/api/v1/assemblies/{id}/queue/{user_id}/call` | [[../../03-subprojects/web/ui-catalog/assembleia/ASM-FILA-FALA|ASM-FILA-FALA]] | assembleia | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/assemblies/{id}/speech-queue/{id}/approve` |
| POST | `/api/v1/assemblies/{id}/queue/{user_id}/end` | [[../../03-subprojects/web/ui-catalog/assembleia/ASM-FILA-FALA|ASM-FILA-FALA]] | assembleia | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/assemblies/{id}/speech-queue/{id}/approve` |
| GET | `/api/v1/assemblies/{id}/quorum-simulation` | [[../../03-subprojects/web/ui-catalog/assembleia/ASM-SIM|ASM-SIM]] | assembleia | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/assemblies/{id}/transparency-score` |
| GET | `/api/v1/assemblies/{id}/reports/attendance` | [[../../03-subprojects/web/ui-catalog/assembleia/ASM-REL|ASM-REL]] | assembleia | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/assemblies/{id}/reports/{type}` |
| GET | `/api/v1/assemblies/{id}/reports/audit` | [[../../03-subprojects/web/ui-catalog/assembleia/ASM-REL|ASM-REL]] | assembleia | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/assemblies/{id}/reports/{type}` |
| GET | `/api/v1/assemblies/{id}/reports/decisions` | [[../../03-subprojects/web/ui-catalog/assembleia/ASM-REL|ASM-REL]] | assembleia | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/assemblies/{id}/reports/{type}` |
| GET | `/api/v1/assemblies/{id}/reports/proxies` | [[../../03-subprojects/web/ui-catalog/assembleia/ASM-REL|ASM-REL]] | assembleia | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/assemblies/{id}/reports/{type}` |
| GET | `/api/v1/assemblies/{id}/reports/summary` | [[../../03-subprojects/web/ui-catalog/assembleia/ASM-REL|ASM-REL]] | assembleia | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/assemblies/{id}/reports/{type}` |
| GET | `/api/v1/assemblies/{id}/reports/votes` | [[../../03-subprojects/web/ui-catalog/assembleia/ASM-REL|ASM-REL]] | assembleia | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/assemblies/{id}/reports/{type}` |
| POST | `/api/v1/assemblies/{id}/start` | [[../../03-subprojects/web/ui-catalog/assembleia/ASM-PAINEL-SIND|ASM-PAINEL-SIND]] | assembleia | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/assemblies/{id}/speech-queue/{id}/approve` |
| GET | `/api/v1/assemblies/{id}/terms` | [[../../03-subprojects/web/ui-catalog/assembleia/ASM-TERM|ASM-TERM]] | assembleia | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/assemblies/{id}/transparency-score` |
| POST | `/api/v1/assemblies/{id}/terms/accept` | [[../../03-subprojects/web/ui-catalog/assembleia/ASM-TERM|ASM-TERM]] | assembleia | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/assemblies/{id}/speech-queue/{id}/approve` |
| POST | `/api/v1/assembly/{id}/check-in` | [[../../03-subprojects/mobile/ui-catalog/assembly/ASM-CHKIN|m:ASM-CHKIN]] | assembleia | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/videos/{id}/view-progress` |
| GET | `/api/v1/cep/resolve` | [[../../03-subprojects/web/ui-catalog/onboarding/ONB-M3|ONB-M3]] | governanca-institucional | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/videos/{id}/playback` |
| POST | `/api/v1/identity/validate-cnpj` | [[../../03-subprojects/web/ui-catalog/onboarding/ONB-E2|ONB-E2]] | governanca-institucional | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/videos/{id}/view-progress` |
| POST | `/api/v1/identity/validate-cpf` | [[../../03-subprojects/web/ui-catalog/onboarding/ONB-M2|ONB-M2]] | governanca-institucional | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/videos/{id}/view-progress` |
| POST | `/api/v1/portfolios/upload` | [[../../03-subprojects/web/ui-catalog/onboarding/ONB-E6|ONB-E6]], [[../../03-subprojects/web/ui-catalog/onboarding/ONB-P3|ONB-P3]] | governanca-institucional | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/videos/{id}/view-progress` |
| POST | `/api/v1/storage/sign` | [[../../03-subprojects/web/ui-catalog/vizinhanca/VZ1|VZ1]] | vizinhanca | Formalizar em `backend/requirements/*.md`; análogo existente: `/api/v1/videos/{id}/view-progress` |
| WS | `/ws/assemblies/{id}` | [[../../03-subprojects/web/ui-catalog/assembleia/ASM-TELAO|ASM-TELAO]] | assembleia | Formalizar em `backend/requirements/*.md`; análogo existente: `/ws/assemblies/{id}/voting` |

---

## B. Endpoints backend sem tela consumidora

162 endpoints formalizados em backend não foram citados por nenhuma tela. Classificados:

### B.1 Webhooks (legítimo — sem UI) — 3

| Método | Path | BC | Conclusão |
|---|---|---|---|
| POST | `/webhooks/mux` | content,legacy-monolith | Legítimo (provider externo) |
| POST | `/webhooks/stripe` | billing,legacy-monolith | Legítimo (provider externo) |
| POST | `/webhooks/{provider}` | cross-domain | Legítimo (provider externo) |

### B.2 Admin (legítimo — console admin, fora do ui-catalog web/mobile de clientes) — 24

| Método | Path | BC | Conclusão |
|---|---|---|---|
| DELETE | `/admin/v1/plans/{id}` | billing | Legítimo (admin console próprio — `ui-catalog/admin/` subdoc) |
| GET | `/admin/v1/audit-trail` | cross-domain | Legítimo (admin console próprio — `ui-catalog/admin/` subdoc) |
| GET | `/admin/v1/disputes` | billing | Legítimo (admin console próprio — `ui-catalog/admin/` subdoc) |
| GET | `/admin/v1/feature-flags` | cross-domain | Legítimo (admin console próprio — `ui-catalog/admin/` subdoc) |
| GET | `/admin/v1/lgpd-logs` | cross-domain | Legítimo (admin console próprio — `ui-catalog/admin/` subdoc) |
| GET | `/admin/v1/metrics/financial` | billing | Legítimo (admin console próprio — `ui-catalog/admin/` subdoc) |
| GET | `/admin/v1/outbox-status` | cross-domain | Legítimo (admin console próprio — `ui-catalog/admin/` subdoc) |
| GET | `/admin/v1/system-health` | cross-domain | Legítimo (admin console próprio — `ui-catalog/admin/` subdoc) |
| PATCH | `/admin/plans/{id}` | billing | Legítimo (admin console próprio — `ui-catalog/admin/` subdoc) |
| PATCH | `/admin/v1/feature-flags/{key}` | cross-domain | Legítimo (admin console próprio — `ui-catalog/admin/` subdoc) |
| PATCH | `/admin/v1/plans/{id}` | billing | Legítimo (admin console próprio — `ui-catalog/admin/` subdoc) |
| POST | `/admin/broadcasts` | content | Legítimo (admin console próprio — `ui-catalog/admin/` subdoc) |
| POST | `/admin/plans` | billing | Legítimo (admin console próprio — `ui-catalog/admin/` subdoc) |
| POST | `/admin/refunds/{pending}/approve` | billing | Legítimo (admin console próprio — `ui-catalog/admin/` subdoc) |
| POST | `/admin/subscriptions/{id}/refund` | billing | Legítimo (admin console próprio — `ui-catalog/admin/` subdoc) |
| POST | `/admin/users/{id}/quota-override` | billing | Legítimo (admin console próprio — `ui-catalog/admin/` subdoc) |
| POST | `/admin/v1/broadcasts` | content | Legítimo (admin console próprio — `ui-catalog/admin/` subdoc) |
| POST | `/admin/v1/plans` | billing | Legítimo (admin console próprio — `ui-catalog/admin/` subdoc) |
| POST | `/admin/v1/refunds/{pending_id}/approve` | billing | Legítimo (admin console próprio — `ui-catalog/admin/` subdoc) |
| POST | `/admin/v1/subscriptions/{id}/pause` | billing | Legítimo (admin console próprio — `ui-catalog/admin/` subdoc) |
| POST | `/admin/v1/subscriptions/{id}/refund` | billing | Legítimo (admin console próprio — `ui-catalog/admin/` subdoc) |
| POST | `/admin/v1/users/{id}/quota-override` | billing | Legítimo (admin console próprio — `ui-catalog/admin/` subdoc) |
| POST | `/admin/v1/videos/{id}/moderate` | content | Legítimo (admin console próprio — `ui-catalog/admin/` subdoc) |
| POST | `/admin/videos/{id}/moderate` | content | Legítimo (admin console próprio — `ui-catalog/admin/` subdoc) |

### B.3 Infra (health/metrics — legítimo) — 4

| Método | Path | BC | Conclusão |
|---|---|---|---|
| GET | `/healthz` | cross-domain | Legítimo (observability) |
| GET | `/metrics` | cross-domain | Legítimo (observability) |
| GET | `/readyz` | cross-domain | Legítimo (observability) |
| GET | `/version` | cross-domain | Legítimo (observability) |

### B.4 Feature APIs sem tela (SUSPEITO — possível endpoint morto ou tela faltante) — 131

| Método | Path | BC | Conclusão |
|---|---|---|---|
| DELETE | `/api/v1/me` | identity | **Revisar**: ou criar tela ou mover para backlog |
| DELETE | `/api/v1/me/consents/{type}` | identity | **Revisar**: ou criar tela ou mover para backlog |
| DELETE | `/api/v1/me/devices/{id}` | identity | **Revisar**: ou criar tela ou mover para backlog |
| DELETE | `/api/v1/me/sessions/{id}` | identity | **Revisar**: ou criar tela ou mover para backlog |
| DELETE | `/api/v1/proxies/{id}` | assembly | **Revisar**: ou criar tela ou mover para backlog |
| GET | `/api/v1/announcements/{id}/read-stats` | institutional | **Revisar**: ou criar tela ou mover para backlog |
| GET | `/api/v1/assemblies/{id}/minutes` | assembly | **Revisar**: ou criar tela ou mover para backlog |
| GET | `/api/v1/assemblies/{id}/president` | assembly | **Revisar**: ou criar tela ou mover para backlog |
| GET | `/api/v1/assemblies/{id}/reports/{type}` | assembly | **Revisar**: ou criar tela ou mover para backlog |
| GET | `/api/v1/assemblies/{id}/simulator` | assembly | **Revisar**: ou criar tela ou mover para backlog |
| GET | `/api/v1/assemblies/{id}/transparency-score` | assembly | **Revisar**: ou criar tela ou mover para backlog |
| GET | `/api/v1/auth/zitadel/jwks` | identity | **Revisar**: ou criar tela ou mover para backlog |
| GET | `/api/v1/certificates/{id}` | content | **Revisar**: ou criar tela ou mover para backlog |
| GET | `/api/v1/companies/search` | commercial | **Revisar**: ou criar tela ou mover para backlog |
| GET | `/api/v1/companies/{id}` | commercial | **Revisar**: ou criar tela ou mover para backlog |
| GET | `/api/v1/companies/{id}/dashboard` | commercial | **Revisar**: ou criar tela ou mover para backlog |
| GET | `/api/v1/companies/{id}/reputation-breakdown` | commercial | **Revisar**: ou criar tela ou mover para backlog |
| GET | `/api/v1/condominiums/{id}` | institutional | **Revisar**: ou criar tela ou mover para backlog |
| GET | `/api/v1/condominiums/{id}/dossier` | institutional | **Revisar**: ou criar tela ou mover para backlog |
| GET | `/api/v1/condominiums/{id}/governance-score` | institutional | **Revisar**: ou criar tela ou mover para backlog |
| GET | `/api/v1/condominiums/{id}/master-plan` | institutional | **Revisar**: ou criar tela ou mover para backlog |
| GET | `/api/v1/condominiums/{id}/master-plan/export` | institutional | **Revisar**: ou criar tela ou mover para backlog |
| GET | `/api/v1/condominiums/{id}/public` | institutional | **Revisar**: ou criar tela ou mover para backlog |
| GET | `/api/v1/condominiums/{id}/timeline/{entry_id}/chain` | institutional | **Revisar**: ou criar tela ou mover para backlog |
| GET | `/api/v1/condominiums/{id}/transparency` | institutional | **Revisar**: ou criar tela ou mover para backlog |
| GET | `/api/v1/condominiums/{id}/units` | institutional | **Revisar**: ou criar tela ou mover para backlog |
| GET | `/api/v1/digital-contents/{id}/download` | content | **Revisar**: ou criar tela ou mover para backlog |
| GET | `/api/v1/governance-assessments` | institutional | **Revisar**: ou criar tela ou mover para backlog |
| GET | `/api/v1/me` | identity | **Revisar**: ou criar tela ou mover para backlog |
| GET | `/api/v1/me/contexts` | identity | **Revisar**: ou criar tela ou mover para backlog |
| GET | `/api/v1/me/devices` | identity | **Revisar**: ou criar tela ou mover para backlog |
| GET | `/api/v1/me/export` | identity | **Revisar**: ou criar tela ou mover para backlog |
| GET | `/api/v1/me/invoices` | billing | **Revisar**: ou criar tela ou mover para backlog |
| GET | `/api/v1/me/quotas` | billing | **Revisar**: ou criar tela ou mover para backlog |
| GET | `/api/v1/me/subscription` | billing | **Revisar**: ou criar tela ou mover para backlog |
| GET | `/api/v1/memberships/{id}` | identity | **Revisar**: ou criar tela ou mover para backlog |
| GET | `/api/v1/partners/{id}` | commercial | **Revisar**: ou criar tela ou mover para backlog |
| GET | `/api/v1/plans` | billing | **Revisar**: ou criar tela ou mover para backlog |
| GET | `/api/v1/podcast/episodes` | content | **Revisar**: ou criar tela ou mover para backlog |
| GET | `/api/v1/search` | content | **Revisar**: ou criar tela ou mover para backlog |
| GET | `/api/v1/talents` | commercial | **Revisar**: ou criar tela ou mover para backlog |
| GET | `/api/v1/videos/{id}/analytics` | content | **Revisar**: ou criar tela ou mover para backlog |
| GET | `/api/v1/videos/{id}/comments` | content | **Revisar**: ou criar tela ou mover para backlog |
| GET | `/api/v1/videos/{id}/playback` | content | **Revisar**: ou criar tela ou mover para backlog |
| GET | `/auth/callback` | identity | **Revisar**: ou criar tela ou mover para backlog |
| GET | `/auth/me` | legacy-monolith | **Revisar**: ou criar tela ou mover para backlog |
| GET | `/certificates/verify/{qr_code}` | content | **Revisar**: ou criar tela ou mover para backlog |
| PATCH | `/api/v1/companies/{id}` | commercial | **Revisar**: ou criar tela ou mover para backlog |
| PATCH | `/api/v1/condominiums/{id}` | institutional | **Revisar**: ou criar tela ou mover para backlog |
| PATCH | `/api/v1/me` | identity | **Revisar**: ou criar tela ou mover para backlog |
| PATCH | `/api/v1/memberships/{id}` | identity,institutional | **Revisar**: ou criar tela ou mover para backlog |
| PATCH | `/api/v1/partners/{id}/palavra-do-dia` | commercial | **Revisar**: ou criar tela ou mover para backlog |
| POST | `/api/v1/announcements` | institutional | **Revisar**: ou criar tela ou mover para backlog |
| POST | `/api/v1/announcements/{id}/publish` | institutional | **Revisar**: ou criar tela ou mover para backlog |
| POST | `/api/v1/assemblies/{id}/acknowledge` | assembly | **Revisar**: ou criar tela ou mover para backlog |
| POST | `/api/v1/assemblies/{id}/agenda-items` | assembly | **Revisar**: ou criar tela ou mover para backlog |
| POST | `/api/v1/assemblies/{id}/agenda-items/{item}/close-voting` | assembly | **Revisar**: ou criar tela ou mover para backlog |
| POST | `/api/v1/assemblies/{id}/agenda-items/{item}/open-voting` | assembly | **Revisar**: ou criar tela ou mover para backlog |
| POST | `/api/v1/assemblies/{id}/agenda-items/{item}/votes` | assembly | **Revisar**: ou criar tela ou mover para backlog |
| POST | `/api/v1/assemblies/{id}/agenda/publish` | assembly | **Revisar**: ou criar tela ou mover para backlog |
| POST | `/api/v1/assemblies/{id}/check-in` | assembly | **Revisar**: ou criar tela ou mover para backlog |
| POST | `/api/v1/assemblies/{id}/edital` | assembly | **Revisar**: ou criar tela ou mover para backlog |
| POST | `/api/v1/assemblies/{id}/homologate-vote` | assembly | **Revisar**: ou criar tela ou mover para backlog |
| POST | `/api/v1/assemblies/{id}/live-room` | assembly | **Revisar**: ou criar tela ou mover para backlog |
| POST | `/api/v1/assemblies/{id}/offline-votes-sync` | assembly | **Revisar**: ou criar tela ou mover para backlog |
| POST | `/api/v1/assemblies/{id}/speech-queue` | assembly | **Revisar**: ou criar tela ou mover para backlog |
| POST | `/api/v1/assemblies/{id}/speech-queue/{id}/approve` | assembly | **Revisar**: ou criar tela ou mover para backlog |
| POST | `/api/v1/billing/checkout` | billing | **Revisar**: ou criar tela ou mover para backlog |
| POST | `/api/v1/billing/portal` | billing | **Revisar**: ou criar tela ou mover para backlog |
| POST | `/api/v1/companies` | commercial | **Revisar**: ou criar tela ou mover para backlog |
| POST | `/api/v1/company-evaluations` | commercial | **Revisar**: ou criar tela ou mover para backlog |
| POST | `/api/v1/compliance-declarations` | institutional | **Revisar**: ou criar tela ou mover para backlog |
| POST | `/api/v1/condominiums` | institutional | **Revisar**: ou criar tela ou mover para backlog |
| POST | `/api/v1/condominiums/{id}/close-mandate` | institutional | **Revisar**: ou criar tela ou mover para backlog |
| POST | `/api/v1/condominiums/{id}/co-admin-invite` | identity | **Revisar**: ou criar tela ou mover para backlog |
| POST | `/api/v1/condominiums/{id}/master-plan/batch-advance` | institutional | **Revisar**: ou criar tela ou mover para backlog |
| POST | `/api/v1/condominiums/{id}/timeline/{type}` | institutional | **Revisar**: ou criar tela ou mover para backlog |
| POST | `/api/v1/condominiums/{id}/transfer-mandate` | institutional | **Revisar**: ou criar tela ou mover para backlog |
| POST | `/api/v1/condominiums/{id}/units` | institutional | **Revisar**: ou criar tela ou mover para backlog |
| POST | `/api/v1/connect-me/company-to-company` | commercial | **Revisar**: ou criar tela ou mover para backlog |
| POST | `/api/v1/connect-me/draft` | commercial | **Revisar**: ou criar tela ou mover para backlog |
| POST | `/api/v1/connect-me/resident-to-syndic` | commercial | **Revisar**: ou criar tela ou mover para backlog |
| POST | `/api/v1/connect-me/syndic-to-company` | commercial | **Revisar**: ou criar tela ou mover para backlog |
| POST | `/api/v1/connect-me/syndic-to-partner` | commercial | **Revisar**: ou criar tela ou mover para backlog |
| POST | `/api/v1/connect-me/{id}/interest` | commercial | **Revisar**: ou criar tela ou mover para backlog |
| POST | `/api/v1/courses` | content | **Revisar**: ou criar tela ou mover para backlog |
| POST | `/api/v1/courses/{id}/modules` | content | **Revisar**: ou criar tela ou mover para backlog |
| POST | `/api/v1/courses/{id}/modules/{mid}/lessons` | content | **Revisar**: ou criar tela ou mover para backlog |
| POST | `/api/v1/deals/{id}/sign` | commercial | **Revisar**: ou criar tela ou mover para backlog |
| POST | `/api/v1/digital-contents` | content | **Revisar**: ou criar tela ou mover para backlog |
| POST | `/api/v1/enrollments` | content | **Revisar**: ou criar tela ou mover para backlog |
| POST | `/api/v1/events` | institutional | **Revisar**: ou criar tela ou mover para backlog |
| POST | `/api/v1/events/{id}/acknowledgments` | institutional | **Revisar**: ou criar tela ou mover para backlog |
| POST | `/api/v1/events/{id}/confirmations` | institutional | **Revisar**: ou criar tela ou mover para backlog |
| POST | `/api/v1/execution-records` | commercial | **Revisar**: ou criar tela ou mover para backlog |
| POST | `/api/v1/execution-records/{id}/validate` | commercial | **Revisar**: ou criar tela ou mover para backlog |
| POST | `/api/v1/forum/posts` | content | **Revisar**: ou criar tela ou mover para backlog |
| POST | `/api/v1/forum/posts/{id}/report` | content | **Revisar**: ou criar tela ou mover para backlog |
| POST | `/api/v1/forum/topics` | content | **Revisar**: ou criar tela ou mover para backlog |
| POST | `/api/v1/governance-assessments/{id}/respond` | institutional | **Revisar**: ou criar tela ou mover para backlog |
| POST | `/api/v1/harassment-reports` | commercial | **Revisar**: ou criar tela ou mover para backlog |
| POST | `/api/v1/lessons/{id}/quiz-response` | content | **Revisar**: ou criar tela ou mover para backlog |
| POST | `/api/v1/lives` | content | **Revisar**: ou criar tela ou mover para backlog |
| POST | `/api/v1/marketing-links` | commercial | **Revisar**: ou criar tela ou mover para backlog |
| POST | `/api/v1/me/consents` | identity | **Revisar**: ou criar tela ou mover para backlog |
| POST | `/api/v1/partners` | commercial | **Revisar**: ou criar tela ou mover para backlog |
| POST | `/api/v1/partners/{id}/benefits` | commercial | **Revisar**: ou criar tela ou mover para backlog |
| POST | `/api/v1/partners/{id}/coupons/generate` | commercial | **Revisar**: ou criar tela ou mover para backlog |
| POST | `/api/v1/polls` | institutional | **Revisar**: ou criar tela ou mover para backlog |
| POST | `/api/v1/polls/{id}/close` | institutional | **Revisar**: ou criar tela ou mover para backlog |
| POST | `/api/v1/polls/{id}/responses` | institutional | **Revisar**: ou criar tela ou mover para backlog |
| POST | `/api/v1/proposals` | commercial | **Revisar**: ou criar tela ou mover para backlog |
| POST | `/api/v1/proposals/{id}/amend` | commercial | **Revisar**: ou criar tela ou mover para backlog |
| POST | `/api/v1/proposals/{id}/send` | commercial | **Revisar**: ou criar tela ou mover para backlog |
| POST | `/api/v1/proxies` | assembly | **Revisar**: ou criar tela ou mover para backlog |
| POST | `/api/v1/reserve-fund-contributions` | institutional | **Revisar**: ou criar tela ou mover para backlog |
| POST | `/api/v1/risk-maps` | institutional | **Revisar**: ou criar tela ou mover para backlog |
| POST | `/api/v1/supplier-vote-sessions` | commercial | **Revisar**: ou criar tela ou mover para backlog |
| POST | `/api/v1/supplier-vote-sessions/{id}/votes` | commercial | **Revisar**: ou criar tela ou mover para backlog |
| POST | `/api/v1/videos/{id}/comments` | content | **Revisar**: ou criar tela ou mover para backlog |
| POST | `/api/v1/videos/{id}/likes` | content | **Revisar**: ou criar tela ou mover para backlog |
| POST | `/api/v1/videos/{id}/view-progress` | content | **Revisar**: ou criar tela ou mover para backlog |
| POST | `/auth/login` | identity | **Revisar**: ou criar tela ou mover para backlog |
| POST | `/auth/logout` | identity | **Revisar**: ou criar tela ou mover para backlog |
| POST | `/auth/password-reset-request` | identity | **Revisar**: ou criar tela ou mover para backlog |
| POST | `/auth/resend-verification` | identity | **Revisar**: ou criar tela ou mover para backlog |
| POST | `/auth/step-up` | identity | **Revisar**: ou criar tela ou mover para backlog |
| POST | `/signup` | identity | **Revisar**: ou criar tela ou mover para backlog |
| WS | `/ws/assemblies/{id}/projection` | assembly | **Revisar**: ou criar tela ou mover para backlog |
| WS | `/ws/assemblies/{id}/speech` | assembly | **Revisar**: ou criar tela ou mover para backlog |
| WS | `/ws/assemblies/{id}/voting` | assembly | **Revisar**: ou criar tela ou mover para backlog |

---

## C. Endpoints 'inferido' em telas

51 menções com path identificável em 33 telas que admitem NÃO ter validação com backend (marca 'inferido' na seção `Integração com backend`). Cada um deve virar FR-BE-### formal **ou** ser removido/movido para backlog.

| Tela | Endpoint inferido | Sub-produto | Ação recomendada |
|---|---|---|---|
| [[../../03-subprojects/web/ui-catalog/empresa/E1|web:E1]] | `/api/v1/enterprise/dashboard` | connect-me | Revisar — sub-produto não-identificado automaticamente |
| [[../../03-subprojects/web/ui-catalog/empresa/E1|web:E1]] | `/api/v1/enterprise/active-tenant` | connect-me | Revisar — sub-produto não-identificado automaticamente |
| [[../../03-subprojects/web/ui-catalog/morador/M1|web:M1]] | `/api/v1/institutional/memberships/me/default` | governanca-institucional | 🔴 Formalizar em `backend/requirements/institutional.md` (M1) |
| [[../../03-subprojects/web/ui-catalog/morador/M1|web:M1]] | `/api/v1/institutional/dashboard/resident` | governanca-institucional | 🔴 Formalizar em `backend/requirements/institutional.md` (M1) |
| [[../../03-subprojects/web/ui-catalog/morador/M15|web:M15]] | `/api/v1/institutional/memberships/{id}/cascade-preview` | governanca-institucional | 🔴 Formalizar em `backend/requirements/institutional.md` (M1) |
| [[../../03-subprojects/web/ui-catalog/morador/M2|web:M2]] | `/api/v1/institutional/memberships/{id}/select` | governanca-institucional | 🔴 Formalizar em `backend/requirements/institutional.md` (M1) |
| [[../../03-subprojects/web/ui-catalog/morador/M4|web:M4]] | `/api/v1/institutional/units/check` | governanca-institucional | 🔴 Formalizar em `backend/requirements/institutional.md` (M1) |
| [[../../03-subprojects/web/ui-catalog/morador/M6|web:M6]] | `/api/v1/institutional/timeline/{id}/read` | governanca-institucional | 🔴 Formalizar em `backend/requirements/institutional.md` (M1) |
| [[../../03-subprojects/web/ui-catalog/sindico/S1|web:S1]] | `/api/v1/syndic/governance/first-access` | governanca-institucional | 🔴 Formalizar em `backend/requirements/institutional.md` (M1) |
| [[../../03-subprojects/web/ui-catalog/sindico/S1|web:S1]] | `/api/v1/syndic/governance/acceptance` | governanca-institucional | 🔴 Formalizar em `backend/requirements/institutional.md` (M1) |
| [[../../03-subprojects/web/ui-catalog/sindico/S10|web:S10]] | `/api/v1/condominiums/{id}/master-plan/actions/{action_id}` | governanca-institucional | 🔴 Formalizar em `backend/requirements/institutional.md` (M1) |
| [[../../03-subprojects/web/ui-catalog/sindico/S10|web:S10]] | `/api/v1/condominiums/{id}/master-plan/actions/{action_id}/history` | governanca-institucional | 🔴 Formalizar em `backend/requirements/institutional.md` (M1) |
| [[../../03-subprojects/web/ui-catalog/sindico/S11|web:S11]] | `/api/v1/condominiums/{id}/timeline` | governanca-institucional | 🔴 Formalizar em `backend/requirements/institutional.md` (M1) |
| [[../../03-subprojects/web/ui-catalog/sindico/S12|web:S12]] | `/api/v1/uploads` | governanca-institucional | 🟠 Formalizar em cross-domain (utilitário compartilhado) |
| [[../../03-subprojects/web/ui-catalog/sindico/S12|web:S12]] | `/api/v1/condominiums/{id}/timeline` | governanca-institucional | 🔴 Formalizar em `backend/requirements/institutional.md` (M1) |
| [[../../03-subprojects/web/ui-catalog/sindico/S13|web:S13]] | `/api/v1/condominiums/{id}/timeline/{entry_id}` | governanca-institucional | 🔴 Formalizar em `backend/requirements/institutional.md` (M1) |
| [[../../03-subprojects/web/ui-catalog/sindico/S13|web:S13]] | `/api/v1/condominiums/{id}/timeline/{entry_id}/versions` | governanca-institucional | 🔴 Formalizar em `backend/requirements/institutional.md` (M1) |
| [[../../03-subprojects/web/ui-catalog/sindico/S14|web:S14]] | `/api/v1/condominiums/{id}/timeline/{entry_id}/versions` | governanca-institucional | 🔴 Formalizar em `backend/requirements/institutional.md` (M1) |
| [[../../03-subprojects/web/ui-catalog/sindico/S14|web:S14]] | `/api/v1/condominiums/{id}/timeline/{entry_id}/diff` | governanca-institucional | 🔴 Formalizar em `backend/requirements/institutional.md` (M1) |
| [[../../03-subprojects/web/ui-catalog/sindico/S15|web:S15]] | `/api/v1/condominiums/{id}/events` | governanca-institucional | 🔴 Formalizar em `backend/requirements/institutional.md` (M1) |
| [[../../03-subprojects/web/ui-catalog/sindico/S16|web:S16]] | `/api/v1/condominiums/{id}/events` | governanca-institucional | 🔴 Formalizar em `backend/requirements/institutional.md` (M1) |
| [[../../03-subprojects/web/ui-catalog/sindico/S17|web:S17]] | `/api/v1/condominiums/{id}/events/export` | governanca-institucional | 🔴 Formalizar em `backend/requirements/institutional.md` (M1) |
| [[../../03-subprojects/web/ui-catalog/sindico/S18|web:S18]] | `/api/v1/connect-me/me/summary` | governanca-institucional | 🟠 Formalizar em `backend/requirements/commercial.md` (M2) |
| [[../../03-subprojects/web/ui-catalog/sindico/S2|web:S2]] | `/api/v1/condominiums` | governanca-institucional | 🔴 Formalizar em `backend/requirements/institutional.md` (M1) |
| [[../../03-subprojects/web/ui-catalog/sindico/S2|web:S2]] | `/api/v1/condominiums/{id}/end-mandate` | governanca-institucional | 🔴 Formalizar em `backend/requirements/institutional.md` (M1) |
| [[../../03-subprojects/web/ui-catalog/sindico/S21|web:S21]] | `/api/v1/condominiums/{id}/service-records` | governanca-institucional | 🔴 Formalizar em `backend/requirements/institutional.md` (M1) |
| [[../../03-subprojects/web/ui-catalog/sindico/S21|web:S21]] | `/api/v1/condominiums/{id}/service-records/external-link` | governanca-institucional | 🔴 Formalizar em `backend/requirements/institutional.md` (M1) |
| [[../../03-subprojects/web/ui-catalog/sindico/S23|web:S23]] | `/api/v1/condominiums/{id}/announcements/summary` | governanca-institucional | 🔴 Formalizar em `backend/requirements/institutional.md` (M1) |
| [[../../03-subprojects/web/ui-catalog/sindico/S25|web:S25]] | `/api/v1/condominiums/{id}/announcements/{aid}/request-adjustment` | governanca-institucional | 🔴 Formalizar em `backend/requirements/institutional.md` (M1) |
| [[../../03-subprojects/web/ui-catalog/sindico/S26|web:S26]] | `/api/v1/condominiums/{id}/validations/summary` | governanca-institucional | 🔴 Formalizar em `backend/requirements/institutional.md` (M1) |
| [[../../03-subprojects/web/ui-catalog/sindico/S27|web:S27]] | `/api/v1/condominiums/{id}/dashboard` | governanca-institucional | 🔴 Formalizar em `backend/requirements/institutional.md` (M1) |
| [[../../03-subprojects/web/ui-catalog/sindico/S27|web:S27]] | `/api/v1/condominiums/{id}/scores/history` | governanca-institucional | 🔴 Formalizar em `backend/requirements/institutional.md` (M1) |
| [[../../03-subprojects/web/ui-catalog/sindico/S27|web:S27]] | `/api/v1/condominiums/{id}/dashboard/export` | governanca-institucional | 🔴 Formalizar em `backend/requirements/institutional.md` (M1) |
| [[../../03-subprojects/web/ui-catalog/sindico/S28|web:S28]] | `/api/v1/condominiums/{id}/compliance/summary` | governanca-institucional | 🔴 Formalizar em `backend/requirements/institutional.md` (M1) |
| [[../../03-subprojects/web/ui-catalog/sindico/S29|web:S29]] | `/api/v1/compliance/responsibility-term/current` | governanca-institucional | Revisar — sub-produto não-identificado automaticamente |
| [[../../03-subprojects/web/ui-catalog/sindico/S3|web:S3]] | `/api/v1/utils/cep/{cep}` | governanca-institucional | 🟠 Formalizar em cross-domain (utilitário compartilhado) |
| [[../../03-subprojects/web/ui-catalog/sindico/S3|web:S3]] | `/api/v1/condominiums/draft` | governanca-institucional | 🔴 Formalizar em `backend/requirements/institutional.md` (M1) |
| [[../../03-subprojects/web/ui-catalog/sindico/S3|web:S3]] | `/api/v1/uploads` | governanca-institucional | 🟠 Formalizar em cross-domain (utilitário compartilhado) |
| [[../../03-subprojects/web/ui-catalog/sindico/S30|web:S30]] | `/api/v1/compliance/annual-declaration/current` | governanca-institucional | Revisar — sub-produto não-identificado automaticamente |
| [[../../03-subprojects/web/ui-catalog/sindico/S31|web:S31]] | `/api/v1/condominiums/{id}/compliance/audit/{entry_id}` | governanca-institucional | 🔴 Formalizar em `backend/requirements/institutional.md` (M1) |
| [[../../03-subprojects/web/ui-catalog/sindico/S31|web:S31]] | `/api/v1/condominiums/{id}/compliance/audit/export` | governanca-institucional | 🔴 Formalizar em `backend/requirements/institutional.md` (M1) |
| [[../../03-subprojects/web/ui-catalog/sindico/S4|web:S4]] | `/api/v1/condominiums/{draft_id}/mandate` | governanca-institucional | 🔴 Formalizar em `backend/requirements/institutional.md` (M1) |
| [[../../03-subprojects/web/ui-catalog/sindico/S4|web:S4]] | `/api/v1/utils/stt` | governanca-institucional | 🟠 Formalizar em cross-domain (utilitário compartilhado) |
| [[../../03-subprojects/web/ui-catalog/sindico/S5|web:S5]] | `/api/v1/condominiums/{draft_id}/finalize` | governanca-institucional | 🔴 Formalizar em `backend/requirements/institutional.md` (M1) |
| [[../../03-subprojects/web/ui-catalog/sindico/S5|web:S5]] | `/api/v1/condominiums/{id}/invite-shared` | governanca-institucional | 🔴 Formalizar em `backend/requirements/institutional.md` (M1) |
| [[../../03-subprojects/web/ui-catalog/sindico/S6|web:S6]] | `/api/v1/condominiums/{id}/dashboard/summary` | governanca-institucional | 🔴 Formalizar em `backend/requirements/institutional.md` (M1) |
| [[../../03-subprojects/web/ui-catalog/sindico/S6|web:S6]] | `/api/v1/condominiums/{id}/cards/badges` | governanca-institucional | 🔴 Formalizar em `backend/requirements/institutional.md` (M1) |
| [[../../03-subprojects/web/ui-catalog/sindico/S7|web:S7]] | `/api/v1/condominiums/{id}/master-plan/actions` | governanca-institucional | 🔴 Formalizar em `backend/requirements/institutional.md` (M1) |
| [[../../03-subprojects/web/ui-catalog/sindico/S7|web:S7]] | `/api/v1/condominiums/{id}/master-plan/summary` | governanca-institucional | 🔴 Formalizar em `backend/requirements/institutional.md` (M1) |
| [[../../03-subprojects/web/ui-catalog/sindico/S8|web:S8]] | `/api/v1/condominiums/{id}/master-plan/actions` | governanca-institucional | 🔴 Formalizar em `backend/requirements/institutional.md` (M1) |
| [[../../03-subprojects/web/ui-catalog/sindico/S9|web:S9]] | `/api/v1/condominiums/{id}/master-plan/actions` | governanca-institucional | 🔴 Formalizar em `backend/requirements/institutional.md` (M1) |

**Agregado por sub-produto (telas com ≥1 endpoint inferido)**:

| Sub-produto | Menções inferido |
|---|---|
| governanca-institucional | 49 |
| connect-me | 2 |

---

## D. Inconsistências web × mobile

Telas em ambas as plataformas mas listando endpoints diferentes. Total: **45** telas divergentes (de 46 com overlap).

| Tela | Apenas web | Apenas mobile | Compartilhado | Conclusão |
|---|---|---|---|---|
| [[../../03-subprojects/web/ui-catalog/morador/M1|w:M1]] / [[../../03-subprojects/mobile/ui-catalog/morador/M1|m:M1]] | — | `PATCH /api/v1/institutional/memberships/me/default` | 2 | Web subdocumentou — completar `Integração com backend` |
| [[../../03-subprojects/web/ui-catalog/morador/M10|w:M10]] / [[../../03-subprojects/mobile/ui-catalog/morador/M10|m:M10]] | `DELETE /api/v1/institutional/events/{id}/cancel-participation`<br>`GET /api/v1/institutional/condominiums/{id}/events`<br>`POST /api/v1/institutional/events/{id}/acknowledge`<br>`POST /api/v1/institutional/events/{id}/confirm-participation` | — | 0 | Mobile subdocumentou — completar `Integração com backend` |
| [[../../03-subprojects/web/ui-catalog/morador/M11|w:M11]] / [[../../03-subprojects/mobile/ui-catalog/morador/M11|m:M11]] | `GET /api/v1/institutional/condominiums/{id}/announcements`<br>`POST /api/v1/institutional/announcements/{id}/acknowledge` | `GET /api/v1/institutional/announcements/{cond}`<br>`GET /api/v1/institutional/announcements/{id}`<br>`POST /api/v1/institutional/announcements/{id}/read` | 0 | Divergência real de contrato — alinhar (padrão: web canônico) |
| [[../../03-subprojects/web/ui-catalog/morador/M12|w:M12]] / [[../../03-subprojects/mobile/ui-catalog/morador/M12|m:M12]] | `GET /api/v1/institutional/assessments/me/pending`<br>`POST /api/v1/institutional/assessments` | `GET /api/v1/reputation/assessment/pending`<br>`POST /api/v1/reputation/assessment` | 0 | Divergência real de contrato — alinhar (padrão: web canônico) |
| [[../../03-subprojects/web/ui-catalog/morador/M13|w:M13]] / [[../../03-subprojects/mobile/ui-catalog/morador/M13|m:M13]] | `GET /api/v1/institutional/lgpd/consents/{id}`<br>`GET /api/v1/institutional/memberships/me/{condo_id}` | — | 0 | Mobile subdocumentou — completar `Integração com backend` |
| [[../../03-subprojects/web/ui-catalog/morador/M14|w:M14]] / [[../../03-subprojects/mobile/ui-catalog/morador/M14|m:M14]] | `GET /api/v1/institutional/memberships/{id}/dependents`<br>`POST /api/v1/institutional/invites/{id}/resend`<br>`POST /api/v1/institutional/memberships/{dependent_id}/revoke`<br>`POST /api/v1/institutional/memberships/{id}/dependents/invite` | — | 0 | Mobile subdocumentou — completar `Integração com backend` |
| [[../../03-subprojects/web/ui-catalog/morador/M15|w:M15]] / [[../../03-subprojects/mobile/ui-catalog/morador/M15|m:M15]] | `GET /api/v1/institutional/memberships/{id}/cascade-preview`<br>`POST /api/v1/institutional/memberships/{id}/revoke` | — | 0 | Mobile subdocumentou — completar `Integração com backend` |
| [[../../03-subprojects/web/ui-catalog/morador/M2|w:M2]] / [[../../03-subprojects/mobile/ui-catalog/morador/M2|m:M2]] | `POST /api/v1/institutional/memberships/{id}/select` | `PATCH /api/v1/institutional/memberships/me/default` | 1 | Divergência real de contrato — alinhar (padrão: web canônico) |
| [[../../03-subprojects/web/ui-catalog/morador/M3|w:M3]] / [[../../03-subprojects/mobile/ui-catalog/morador/M3|m:M3]] | `GET /api/v1/institutional/condominiums/lookup` | — | 0 | Mobile subdocumentou — completar `Integração com backend` |
| [[../../03-subprojects/web/ui-catalog/morador/M4|w:M4]] / [[../../03-subprojects/mobile/ui-catalog/morador/M4|m:M4]] | `GET /api/v1/institutional/units/check`<br>`POST /api/v1/institutional/memberships/draft` | — | 0 | Mobile subdocumentou — completar `Integração com backend` |
| [[../../03-subprojects/web/ui-catalog/morador/M5|w:M5]] / [[../../03-subprojects/mobile/ui-catalog/morador/M5|m:M5]] | `POST /api/v1/institutional/memberships` | — | 0 | Mobile subdocumentou — completar `Integração com backend` |
| [[../../03-subprojects/web/ui-catalog/morador/M6|w:M6]] / [[../../03-subprojects/mobile/ui-catalog/morador/M6|m:M6]] | `GET /api/v1/institutional/condominiums/{id}/timeline` | `GET /api/v1/institutional/timeline/{cond}?page&size&category`<br>`GET /api/v1/institutional/timeline/{id}` | 1 | Divergência real de contrato — alinhar (padrão: web canônico) |
| [[../../03-subprojects/web/ui-catalog/morador/M7|w:M7]] / [[../../03-subprojects/mobile/ui-catalog/morador/M7|m:M7]] | `GET /api/v1/institutional/timeline/{id}`<br>`GET /api/v1/institutional/timeline/{id}/addenda`<br>`POST /api/v1/institutional/polls/{poll_id}/responses` | — | 0 | Mobile subdocumentou — completar `Integração com backend` |
| [[../../03-subprojects/web/ui-catalog/morador/M8|w:M8]] / [[../../03-subprojects/mobile/ui-catalog/morador/M8|m:M8]] | `GET /api/v1/institutional/condominiums/{id}/master-plan` | — | 0 | Mobile subdocumentou — completar `Integração com backend` |
| [[../../03-subprojects/web/ui-catalog/morador/M9|w:M9]] / [[../../03-subprojects/mobile/ui-catalog/morador/M9|m:M9]] | `GET /api/v1/institutional/master-plan/{id}`<br>`GET /api/v1/institutional/master-plan/{id}/timeline` | — | 0 | Mobile subdocumentou — completar `Integração com backend` |
| [[../../03-subprojects/web/ui-catalog/onboarding/ONB-E2|w:ONB-E2]] / [[../../03-subprojects/mobile/ui-catalog/onboarding/ONB-E2|m:ONB-E2]] | `PATCH /api/v1/onboarding/draft`<br>`POST /api/v1/identity/validate-cnpj` | — | 0 | Mobile subdocumentou — completar `Integração com backend` |
| [[../../03-subprojects/web/ui-catalog/onboarding/ONB-E3|w:ONB-E3]] / [[../../03-subprojects/mobile/ui-catalog/onboarding/ONB-E3|m:ONB-E3]] | `PATCH /api/v1/onboarding/draft` | — | 0 | Mobile subdocumentou — completar `Integração com backend` |
| [[../../03-subprojects/web/ui-catalog/onboarding/ONB-E4|w:ONB-E4]] / [[../../03-subprojects/mobile/ui-catalog/onboarding/ONB-E4|m:ONB-E4]] | `GET /api/v1/categories`<br>`PATCH /api/v1/onboarding/draft` | — | 0 | Mobile subdocumentou — completar `Integração com backend` |
| [[../../03-subprojects/web/ui-catalog/onboarding/ONB-E5|w:ONB-E5]] / [[../../03-subprojects/mobile/ui-catalog/onboarding/ONB-E5|m:ONB-E5]] | `PATCH /api/v1/onboarding/draft` | — | 0 | Mobile subdocumentou — completar `Integração com backend` |
| [[../../03-subprojects/web/ui-catalog/onboarding/ONB-E6|w:ONB-E6]] / [[../../03-subprojects/mobile/ui-catalog/onboarding/ONB-E6|m:ONB-E6]] | `PATCH /api/v1/onboarding/draft`<br>`POST /api/v1/portfolios/upload`<br>`POST /api/v1/videos/institutional/upload`<br>`POST /api/v1/videos/technical/upload` | — | 0 | Mobile subdocumentou — completar `Integração com backend` |
| [[../../03-subprojects/web/ui-catalog/onboarding/ONB-E7|w:ONB-E7]] / [[../../03-subprojects/mobile/ui-catalog/onboarding/ONB-E7|m:ONB-E7]] | `POST /api/v1/onboarding/terms/accept` | — | 0 | Mobile subdocumentou — completar `Integração com backend` |
| [[../../03-subprojects/web/ui-catalog/onboarding/ONB-M2|w:ONB-M2]] / [[../../03-subprojects/mobile/ui-catalog/onboarding/ONB-M2|m:ONB-M2]] | `PATCH /api/v1/onboarding/draft`<br>`POST /api/v1/identity/validate-cpf`<br>`POST /api/v1/media/avatars/upload` | — | 0 | Mobile subdocumentou — completar `Integração com backend` |
| [[../../03-subprojects/web/ui-catalog/onboarding/ONB-M3|w:ONB-M3]] / [[../../03-subprojects/mobile/ui-catalog/onboarding/ONB-M3|m:ONB-M3]] | `GET /api/v1/cep/resolve`<br>`GET /api/v1/condominiums/search`<br>`PATCH /api/v1/onboarding/draft`<br>`POST /api/v1/condominiums/{id}/memberships/request` | — | 0 | Mobile subdocumentou — completar `Integração com backend` |
| [[../../03-subprojects/web/ui-catalog/onboarding/ONB-M4|w:ONB-M4]] / [[../../03-subprojects/mobile/ui-catalog/onboarding/ONB-M4|m:ONB-M4]] | `POST /api/v1/onboarding/terms/accept` | — | 0 | Mobile subdocumentou — completar `Integração com backend` |
| [[../../03-subprojects/web/ui-catalog/onboarding/ONB-MFIN|w:ONB-MFIN]] / [[../../03-subprojects/mobile/ui-catalog/onboarding/ONB-MFIN|m:ONB-MFIN]] | `POST /api/v1/onboarding/complete` | — | 0 | Mobile subdocumentou — completar `Integração com backend` |
| [[../../03-subprojects/web/ui-catalog/onboarding/ONB-P2|w:ONB-P2]] / [[../../03-subprojects/mobile/ui-catalog/onboarding/ONB-P2|m:ONB-P2]] | `PATCH /api/v1/onboarding/draft` | — | 0 | Mobile subdocumentou — completar `Integração com backend` |
| [[../../03-subprojects/web/ui-catalog/onboarding/ONB-P3|w:ONB-P3]] / [[../../03-subprojects/mobile/ui-catalog/onboarding/ONB-P3|m:ONB-P3]] | `PATCH /api/v1/onboarding/draft`<br>`POST /api/v1/portfolios/upload` | — | 0 | Mobile subdocumentou — completar `Integração com backend` |
| [[../../03-subprojects/web/ui-catalog/onboarding/ONB-P4|w:ONB-P4]] / [[../../03-subprojects/mobile/ui-catalog/onboarding/ONB-P4|m:ONB-P4]] | `POST /api/v1/onboarding/terms/accept` | — | 0 | Mobile subdocumentou — completar `Integração com backend` |
| [[../../03-subprojects/web/ui-catalog/onboarding/ONB-S2|w:ONB-S2]] / [[../../03-subprojects/mobile/ui-catalog/onboarding/ONB-S2|m:ONB-S2]] | `PATCH /api/v1/onboarding/draft` | — | 0 | Mobile subdocumentou — completar `Integração com backend` |
| [[../../03-subprojects/web/ui-catalog/onboarding/ONB-S3|w:ONB-S3]] / [[../../03-subprojects/mobile/ui-catalog/onboarding/ONB-S3|m:ONB-S3]] | `PATCH /api/v1/onboarding/draft` | — | 0 | Mobile subdocumentou — completar `Integração com backend` |
| [[../../03-subprojects/web/ui-catalog/onboarding/ONB-S4|w:ONB-S4]] / [[../../03-subprojects/mobile/ui-catalog/onboarding/ONB-S4|m:ONB-S4]] | `PATCH /api/v1/onboarding/draft` | — | 0 | Mobile subdocumentou — completar `Integração com backend` |
| [[../../03-subprojects/web/ui-catalog/onboarding/ONB-S5|w:ONB-S5]] / [[../../03-subprojects/mobile/ui-catalog/onboarding/ONB-S5|m:ONB-S5]] | `GET /api/v1/videos/institutional/lock-status`<br>`PATCH /api/v1/onboarding/draft`<br>`POST /api/v1/videos/institutional/upload` | — | 0 | Mobile subdocumentou — completar `Integração com backend` |
| [[../../03-subprojects/web/ui-catalog/onboarding/ONB-S6|w:ONB-S6]] / [[../../03-subprojects/mobile/ui-catalog/onboarding/ONB-S6|m:ONB-S6]] | `POST /api/v1/onboarding/terms/accept` | — | 0 | Mobile subdocumentou — completar `Integração com backend` |
| [[../../03-subprojects/web/ui-catalog/onboarding/ONB-SFIN|w:ONB-SFIN]] / [[../../03-subprojects/mobile/ui-catalog/onboarding/ONB-SFIN|m:ONB-SFIN]] | `POST /api/v1/onboarding/complete` | — | 0 | Mobile subdocumentou — completar `Integração com backend` |
| [[../../03-subprojects/web/ui-catalog/sindico/S1|w:S1]] / [[../../03-subprojects/mobile/ui-catalog/sindico/S1|m:S1]] | `GET /api/v1/syndic/governance/first-access (inferido`<br>`POST /api/v1/syndic/governance/acceptance (inferido` | `GET /api/v1/syndic/governance/first-access`<br>`POST /api/v1/syndic/governance/acceptance` | 0 | Divergência real de contrato — alinhar (padrão: web canônico) |
| [[../../03-subprojects/web/ui-catalog/sindico/S2|w:S2]] / [[../../03-subprojects/mobile/ui-catalog/sindico/S2|m:S2]] | `GET /api/v1/condominiums?owner=me&status=active`<br>`POST /api/v1/condominiums/{id}/end-mandate (inferido` | — | 0 | Mobile subdocumentou — completar `Integração com backend` |
| [[../../03-subprojects/web/ui-catalog/sindico/S6|w:S6]] / [[../../03-subprojects/mobile/ui-catalog/sindico/S6|m:S6]] | `GET /api/v1/condominiums/{id}/cards/badges (inferido`<br>`GET /api/v1/condominiums/{id}/dashboard/summary (inferido` | — | 0 | Mobile subdocumentou — completar `Integração com backend` |
| [[../../03-subprojects/web/ui-catalog/sindico/S7|w:S7]] / [[../../03-subprojects/mobile/ui-catalog/sindico/S7|m:S7]] | `GET /api/v1/condominiums/{id}/master-plan/actions (inferido`<br>`GET /api/v1/condominiums/{id}/master-plan/summary (inferido` | — | 0 | Mobile subdocumentou — completar `Integração com backend` |
| [[../../03-subprojects/web/ui-catalog/sindico/S8|w:S8]] / [[../../03-subprojects/mobile/ui-catalog/sindico/S8|m:S8]] | `POST /api/v1/condominiums/{id}/master-plan/actions (inferido` | — | 0 | Mobile subdocumentou — completar `Integração com backend` |
| [[../../03-subprojects/web/ui-catalog/vizinhanca/VM1|w:VM1]] / [[../../03-subprojects/mobile/ui-catalog/vizinhanca/VM1|m:VM1]] | `GET /api/v1/neighborhood/feed?audience=resident` | — | 0 | Mobile subdocumentou — completar `Integração com backend` |
| [[../../03-subprojects/web/ui-catalog/vizinhanca/VM2|w:VM2]] / [[../../03-subprojects/mobile/ui-catalog/vizinhanca/VM2|m:VM2]] | `GET /api/v1/neighborhood/partners/{id}?audience=resident`<br>`POST /api/v1/neighborhood/partners/{id}/contact-click` | — | 0 | Mobile subdocumentou — completar `Integração com backend` |
| [[../../03-subprojects/web/ui-catalog/vizinhanca/VM3|w:VM3]] / [[../../03-subprojects/mobile/ui-catalog/vizinhanca/VM3|m:VM3]] | `GET /api/v1/neighborhood/promotions/{id}?audience=resident` | — | 0 | Mobile subdocumentou — completar `Integração com backend` |
| [[../../03-subprojects/web/ui-catalog/vizinhanca/VM5|w:VM5]] / [[../../03-subprojects/mobile/ui-catalog/vizinhanca/VM5|m:VM5]] | `GET /api/v1/neighborhood/exclusive-benefits?condominium_id=mine` | — | 0 | Mobile subdocumentou — completar `Integração com backend` |
| [[../../03-subprojects/web/ui-catalog/vizinhanca/VM6|w:VM6]] / [[../../03-subprojects/mobile/ui-catalog/vizinhanca/VM6|m:VM6]] | `GET /api/v1/neighborhood/partners/recommended?condominium_id=mine` | — | 0 | Mobile subdocumentou — completar `Integração com backend` |
| [[../../03-subprojects/web/ui-catalog/vizinhanca/VM7|w:VM7]] / [[../../03-subprojects/mobile/ui-catalog/vizinhanca/VM7|m:VM7]] | `GET /api/v1/neighborhood/activations?actor=resident&actor_id=me` | — | 0 | Mobile subdocumentou — completar `Integração com backend` |

---

## E. Top 20 gaps acionáveis priorizados (bloqueadores M1)

20 endpoints CRITICAL com MAIOR número de telas consumidoras — consertar primeiro produz maior desbloqueio.

1. **`PATCH /api/v1/onboarding/draft`** — consumido por **13** tela(s) ([[../../03-subprojects/web/ui-catalog/onboarding/ONB-E2|ONB-E2]], [[../../03-subprojects/web/ui-catalog/onboarding/ONB-E3|ONB-E3]], [[../../03-subprojects/web/ui-catalog/onboarding/ONB-E4|ONB-E4]], [[../../03-subprojects/web/ui-catalog/onboarding/ONB-E5|ONB-E5]], [[../../03-subprojects/web/ui-catalog/onboarding/ONB-E6|ONB-E6]], +8).
    - Ação: adicionar linha na tabela `Endpoints REST expostos` em `backend/requirements/institutional.md` + FR-BE-### + entrada em `traceability.md`.
2. **`POST /api/v1/onboarding/complete`** — consumido por **4** tela(s) ([[../../03-subprojects/web/ui-catalog/onboarding/ONB-EFIN|ONB-EFIN]], [[../../03-subprojects/web/ui-catalog/onboarding/ONB-MFIN|ONB-MFIN]], [[../../03-subprojects/web/ui-catalog/onboarding/ONB-PFIN|ONB-PFIN]], [[../../03-subprojects/web/ui-catalog/onboarding/ONB-SFIN|ONB-SFIN]]).
    - Ação: adicionar linha na tabela `Endpoints REST expostos` em `backend/requirements/institutional.md` + FR-BE-### + entrada em `traceability.md`.
3. **`POST /api/v1/onboarding/terms/accept`** — consumido por **4** tela(s) ([[../../03-subprojects/web/ui-catalog/onboarding/ONB-E7|ONB-E7]], [[../../03-subprojects/web/ui-catalog/onboarding/ONB-M4|ONB-M4]], [[../../03-subprojects/web/ui-catalog/onboarding/ONB-P4|ONB-P4]], [[../../03-subprojects/web/ui-catalog/onboarding/ONB-S6|ONB-S6]]).
    - Ação: adicionar linha na tabela `Endpoints REST expostos` em `backend/requirements/institutional.md` + FR-BE-### + entrada em `traceability.md`.
4. **`GET /api/v1/condominiums`** — consumido por **2** tela(s) ([[../../03-subprojects/web/ui-catalog/sindico/S2|S2]], [[../../03-subprojects/web/ui-catalog/vizinhanca/VZ3|VZ3]]).
    - Ação: adicionar linha na tabela `Endpoints REST expostos` em `backend/requirements/institutional.md` + FR-BE-### + entrada em `traceability.md`.
5. **`GET /api/v1/condominiums/{id}/announcements`** — consumido por **2** tela(s) ([[../../03-subprojects/web/ui-catalog/sindico/S23|S23]], [[../../03-subprojects/web/ui-catalog/sindico/S25|S25]]).
    - Ação: adicionar linha na tabela `Endpoints REST expostos` em `backend/requirements/institutional.md` + FR-BE-### + entrada em `traceability.md`.
6. **`GET /api/v1/condominiums/{id}/events`** — consumido por **2** tela(s) ([[../../03-subprojects/web/ui-catalog/sindico/S15|S15]], [[../../03-subprojects/web/ui-catalog/sindico/S17|S17]]).
    - Ação: adicionar linha na tabela `Endpoints REST expostos` em `backend/requirements/institutional.md` + FR-BE-### + entrada em `traceability.md`.
7. **`GET /api/v1/condominiums/{id}/master-plan/actions`** — consumido por **2** tela(s) ([[../../03-subprojects/web/ui-catalog/sindico/S7|S7]], [[../../03-subprojects/web/ui-catalog/sindico/S9|S9]]).
    - Ação: adicionar linha na tabela `Endpoints REST expostos` em `backend/requirements/institutional.md` + FR-BE-### + entrada em `traceability.md`.
8. **`GET /api/v1/institutional/dashboard/resident`** — consumido por **2** tela(s) ([[../../03-subprojects/mobile/ui-catalog/morador/M1|m:M1]], [[../../03-subprojects/web/ui-catalog/morador/M1|M1]]).
    - Ação: adicionar linha na tabela `Endpoints REST expostos` em `backend/requirements/institutional.md` + FR-BE-### + entrada em `traceability.md`.
9. **`GET /api/v1/institutional/memberships/me`** — consumido por **2** tela(s) ([[../../03-subprojects/mobile/ui-catalog/morador/M2|m:M2]], [[../../03-subprojects/web/ui-catalog/morador/M2|M2]]).
    - Ação: adicionar linha na tabela `Endpoints REST expostos` em `backend/requirements/institutional.md` + FR-BE-### + entrada em `traceability.md`.
10. **`GET /api/v1/institutional/memberships/me/default`** — consumido por **2** tela(s) ([[../../03-subprojects/mobile/ui-catalog/morador/M1|m:M1]], [[../../03-subprojects/web/ui-catalog/morador/M1|M1]]).
    - Ação: adicionar linha na tabela `Endpoints REST expostos` em `backend/requirements/institutional.md` + FR-BE-### + entrada em `traceability.md`.
11. **`PATCH /api/v1/institutional/memberships/me/default`** — consumido por **2** tela(s) ([[../../03-subprojects/mobile/ui-catalog/morador/M1|m:M1]], [[../../03-subprojects/mobile/ui-catalog/morador/M2|m:M2]]).
    - Ação: adicionar linha na tabela `Endpoints REST expostos` em `backend/requirements/institutional.md` + FR-BE-### + entrada em `traceability.md`.
12. **`GET /api/v1/institutional/timeline/{id}`** — consumido por **2** tela(s) ([[../../03-subprojects/mobile/ui-catalog/morador/M6|m:M6]], [[../../03-subprojects/web/ui-catalog/morador/M7|M7]]).
    - Ação: adicionar linha na tabela `Endpoints REST expostos` em `backend/requirements/institutional.md` + FR-BE-### + entrada em `traceability.md`.
13. **`POST /api/v1/institutional/timeline/{id}/read`** — consumido por **2** tela(s) ([[../../03-subprojects/mobile/ui-catalog/morador/M6|m:M6]], [[../../03-subprojects/web/ui-catalog/morador/M6|M6]]).
    - Ação: adicionar linha na tabela `Endpoints REST expostos` em `backend/requirements/institutional.md` + FR-BE-### + entrada em `traceability.md`.
14. **`POST /api/v1/media/upload`** — consumido por **2** tela(s) ([[../../03-subprojects/web/ui-catalog/empresa/E12|E12]], [[../../03-subprojects/web/ui-catalog/empresa/E9|E9]]).
    - Ação: adicionar linha na tabela `Endpoints REST expostos` em `backend/requirements/institutional.md` + FR-BE-### + entrada em `traceability.md`.
15. **`POST /api/v1/onboarding/draft`** — consumido por **2** tela(s) ([[../../03-subprojects/web/ui-catalog/onboarding/ONB-00-boas-vindas|ONB-00-boas-vindas]], [[../../03-subprojects/web/ui-catalog/onboarding/ONB-01-identificacao-inicial|ONB-01-identificacao-inicial]]).
    - Ação: adicionar linha na tabela `Endpoints REST expostos` em `backend/requirements/institutional.md` + FR-BE-### + entrada em `traceability.md`.
16. **`POST /api/v1/syndic/governance/acceptance`** — consumido por **2** tela(s) ([[../../03-subprojects/mobile/ui-catalog/sindico/S1|m:S1]], [[../../03-subprojects/web/ui-catalog/sindico/S1|S1]]).
    - Ação: adicionar linha na tabela `Endpoints REST expostos` em `backend/requirements/institutional.md` + FR-BE-### + entrada em `traceability.md`.
17. **`GET /api/v1/syndic/governance/first-access`** — consumido por **2** tela(s) ([[../../03-subprojects/mobile/ui-catalog/sindico/S1|m:S1]], [[../../03-subprojects/web/ui-catalog/sindico/S1|S1]]).
    - Ação: adicionar linha na tabela `Endpoints REST expostos` em `backend/requirements/institutional.md` + FR-BE-### + entrada em `traceability.md`.
18. **`GET /api/v1/utils/cep/{cep}`** — consumido por **2** tela(s) ([[../../03-subprojects/web/ui-catalog/sindico/S3|S3]], [[../../03-subprojects/web/ui-catalog/vizinhanca/VZ1|VZ1]]).
    - Ação: adicionar linha na tabela `Endpoints REST expostos` em `backend/requirements/institutional.md` + FR-BE-### + entrada em `traceability.md`.
19. **`GET /api/v1/assemblies/{id}/timeline`** — consumido por **1** tela(s) ([[../../03-subprojects/web/ui-catalog/assembleia/ASM-REL|ASM-REL]]).
    - Ação: adicionar linha na tabela `Endpoints REST expostos` em `backend/requirements/institutional.md` + FR-BE-### + entrada em `traceability.md`.
20. **`POST /api/v1/condominiums/draft`** — consumido por **1** tela(s) ([[../../03-subprojects/web/ui-catalog/sindico/S3|S3]]).
    - Ação: adicionar linha na tabela `Endpoints REST expostos` em `backend/requirements/institutional.md` + FR-BE-### + entrada em `traceability.md`.

---

## F. Recomendações de processo

1. **Formalizar catálogo REST completo no backend** — criar seção `## Endpoints REST expostos (catálogo completo)` em cada `backend/requirements/<bc>.md`, migrando os 340 endpoints citados para tabelas formais com FR-BE-### correspondente. Meta realista: 🔴 CRITICAL completos até 2026-05-01 (T-7 do M1).
2. **Zerar endpoints 'inferido' até T-5 do M1 (2026-05-03)** — todo endpoint inferido é risco de handler ausente em produção. Se não dá pra formalizar no M1, move pra backlog e remove da tela.
3. **Unificar contratos web×mobile** — adotar convenção: **web é canônico**; mobile só diverge quando há feature mobile-only (push, offline-queue). Abrir tickets para as 45 telas divergentes (~3h cada; pode paralelizar).
4. **Expandir `traceability.md`** — matriz atual cobre ~20 endpoints. Alvo: cobrir 340 com colunas `(UI screen, FR-UI, endpoint, FR-BE, status)`.
5. **Revisar 131 feature-APIs órfãs** — sanity-check: criar tela, mover para backlog, ou descontinuar. APIs mortas são pentest risk + custo de manutenção.
6. **DoD de tela reforçado** — toda PR que mexe em tela exige que endpoint esteja em `backend/requirements/<bc>.md`; CI opcional com script diff que usa a mesma lógica deste relatório.
7. **Automatização** — o script usado aqui pode virar `tools/gap-endpoints.py` rodando em pre-merge; produz delta toda PR.

---

## Metodologia

**Extração UI** (340 endpoints únicos):

- Varredura em `03-subprojects/web/ui-catalog/**/*.md` (192) + `03-subprojects/mobile/ui-catalog/**/*.md` (59).
- Regex captura: (a) tabelas markdown com coluna de método HTTP + coluna de path iniciando em `/api|/auth|/webhooks|/signup|/ws`; (b) menções inline `METHOD /path` no corpo.
- Normalização: `:param` → `{param}`; strip `?query`; `//` → `/`; tira `.`/`,`/`;`/`)` do final.

**Extração backend** (169 endpoints únicos):

- Varredura em `03-subprojects/backend/requirements/*.md` (7 BCs) + `03-subprojects/backend/requirements.md` (monolítico legado).
- Captura tabelas `## Endpoints REST expostos` + menções inline em FR-BE-###.
- Mesma normalização.

**Cruzamento**:

- Match exato `(método, path-normalizado)` → lista A e B.
- Detecção de method-mismatch: path exato mas método diferente (5 casos identificados).
- Detecção de near-path: mesmo path-key ignorando nomes de params (6 casos).
- Severidade atribuída por prefixo de path mapeando ao escopo do ROADMAP (identity/billing/institutional/events/timeline/master-plan = M1).

**Inferido (C)**:

- Grep case-insensitive por 'inferid' nas telas; extrai path na mesma linha quando presente.

**Inconsistências web×mobile (D)**:

- Match por `(subfolder, screen_id)` entre ui-catalog web e mobile; diff dos conjuntos de endpoints.

**Dados brutos** (regeneráveis via script inline):

- `/tmp/gap-analysis/ui-endpoints-clean.tsv`, `/tmp/gap-analysis/backend-endpoints-unique.tsv`
- `/tmp/gap-analysis/A-phantom-final.json`, `/tmp/gap-analysis/B-orphan-classified.json`
- `/tmp/gap-analysis/C-inferido.tsv`, `/tmp/gap-analysis/D-inconsistencies.json`

---

## Links

- [[../AUDIT]]
- [[../../03-subprojects/traceability]]
- [[../../03-subprojects/backend/requirements/_moc]]
- [[../../ROADMAP]]
- [[../../03-subprojects/web/ui-catalog/_moc|Web UI Catalog _moc]]
- [[../../03-subprojects/mobile/ui-catalog/_moc|Mobile UI Catalog _moc]]
- [[../_moc]]
- [[../../_moc]]
- [[../../CLAUDE]]
