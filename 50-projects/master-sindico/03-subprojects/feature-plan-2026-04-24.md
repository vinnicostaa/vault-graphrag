---
title: Feature Plan Frontend — decisões Fase 11/12 (2026-04-24)
type: note
tags:
  - master-sindico
  - feature-plan
  - front
  - web
  - mobile
  - fase-11
  - fase-12
project: master-sindico
created: 2026-04-24T00:00:00.000Z
---

# Feature Plan Frontend — decisões D-094..D-105

**Contexto**: backend iniciou implementação das 12 decisões registradas em 2026-04-24 (Fase 11: D-094..D-102; Fase 12: D-103..D-105). Este plano registra **o que precisa ir ao frontend** (web + mobile) em paralelo ou imediatamente após.

**Pré-condição global**: [[../AUDIT#a-fase10-dev-005]] (web M1 não iniciado) torna escopo M1 inviável em 2026-05-08 com 2 semanas. Assumir **Opção A** (M1 = 2026-05-18) ou shipping backend+mobile-only.

## Sumário executivo

- **Total decisões cobertas**: 12 (D-094..D-105)
- **Telas novas criadas Fase 12**: 3 (ASM-AVAL-ELEICAO web + mobile; S32)
- **Telas com patch**: C10 (Q25 resolvida); demais aguardando sprint 10
- **Componentes DS novos propostos**: 11 (ver §5)
- **Marcos**: M1 (8/18 mai), M2 (20 jun), M3 (20 jul)

## Status por decisão

| D-### | Q/Gap | Impacto no front | Status do front |
|---|---|---|---|
| D-094 | Q2 (quota Morador Base) | M-CONNECTME ou M12 mostra quota 2/ano; modal upgrade ao esgotar | 🟡 patch pendente |
| D-095 | Q4 (agência login) | MK1-MK8 OK; fluxo sign-in específico role marketing em auth app | 🟢 MK1-MK8 prontas; auth pendente |
| D-096 | Q5 (N1/N2/N3 abolidos) | Remover menções N1/N2/N3 de UI; PlanBadge mapping novo | 🟡 Fase 11 limpou v2 requirements; UI catalog pode ter stragglers |
| D-097 | Q28 (Empresa Pro Lives LMS) | LMS8-10 + card E1 empresa; LiveCreationForm component | 🟡 pending |
| D-098 | Q29 (15 condos) | S2 contador "X de 15" + warning; LimitCounter component | 🟡 pending |
| D-099 | Q39 (BT 5 vínculos) | BT08 já atualizada Fase 11; revisar TimelineInput | ✅ BT08 OK |
| D-100 | D-046 (email provider) | Nenhum direto (config backend) | ✅ N/A |
| D-101 | LGPD soft_delete | Tela M-ACCOUNT-DELETE + E-ACCOUNT-DELETE + DeletionTimeline component | 🟡 pending |
| D-102 | Painel agência | MK1-MK8 já prontas — confirmação | ✅ |
| **D-103** | **Q25 score duplo** | **S32 nova (perfil síndico); C10 patch**; ScoreGauge + ScoreHistory + ScoreBreakdown components | ✅ S32 criada / ✅ C10 atualizada |
| **D-104** | **Q40 aval escondida** | **ASM-AVAL-ELEICAO web + mobile nova**; ASM-VOTO patch bloqueio; HiddenAssessmentForm + NavGuard | ✅ 2 telas novas criadas |
| **D-105** | **ValidationItem opção B** | S21/S22/S26 consumo via view; ValidationQueueItem component unificado | 🟡 telas ok, component pending |

## Por decisão — detalhamento

### D-094 — Morador Base Connect Me 2/ano

**Front impacto**:
- Tela M12 (ou nova M-CONNECTME-QUOTA): exibir quota Connect Me remanescente
- Modal "Quota esgotada — upgrade para Paid" quando tentativa de enviar após consumir 2/ano
- Componente `QuotaBadge` (novo) — reutilizável em Connect Me outbound, outras quotas (vídeo, assembleia)

**Ação Sprint 10**: patch em [[web/ui-catalog/morador/M12]] com bloco "Quota Connect Me 2/ano"; adicionar nota sobre QuotaBadge em design-system.

### D-095 — Agência Zitadel + painel admin próprio

**Front impacto**:
- MK1-MK8 já documentadas (cobertura OK do painel)
- Adicionar em `_auth/sign-in`: fluxo que role `marketing` pós-login redireciona para `/mk1` (não para `/cms`)
- Tela nova (opcional) `MK-ONBOARDING`: setup inicial da agência ao aceitar convite de empresa

**Ação Sprint 10**: patch [[web/ui-catalog/marketing/MK1]] com fluxo login; criar MK-ONBOARDING se necessário.

### D-096 — N1/N2/N3 abolidos

**Front impacto**:
- Fase 11 limpou v2 requirements; UI catalog pode ter stragglers
- Componente `PlanBadge` (novo) — mapping: `trial | base | plus | pro` → variants com cor/ícone
- Grep varredura em web/ui-catalog + mobile para zero referências N1/N2/N3 remanescentes

**Ação Sprint 10**: sed global em ui-catalog; criar PlanBadge stub no design-system.

### D-097 — Empresa Pro cria Lives

**Front impacto**:
- Card "Criar Live" em [[web/ui-catalog/empresa/E1]] para role `enterprise` com plan `pro`
- Fluxo LMS: [[web/ui-catalog/lms/LMS8]] `Agenda de Lives` (já existente) + nova tela `LMS-LIVE-CRIAR` (backlog M3)
- Componente `LiveCreationForm` (novo) — reutilizado LMS e painel empresa

**Ação Sprint 10**: patch E1 com card condicional; stub LMS-LIVE-CRIAR.md.

### D-098 — 15 condomínios

**Front impacto**:
- [[web/ui-catalog/sindico/S2]]: exibir "3 de 15 condomínios" com barra progresso
- Alerta aos 12 (80%): "Próximo do limite — considere encerrar mandatos inativos"
- Componente `LimitCounter` (novo)

**Ação Sprint 10**: patch S2.

### D-099 — BT 5 vínculos

**Front impacto**: ✅ [[web/ui-catalog/banco-talentos/BT08]] já atualizada (Fase 11).

### D-101 — soft_delete universal

**Front impacto** (mais carregado):
- Tela nova `M-ACCOUNT-DELETE` (web + mobile): morador solicita exclusão de conta
  - Modal warning com 48h cooldown
  - Aviso "Dados pseudonimizados imediatamente; retenção audit 5 anos"
  - Botão "Cancelar solicitação" durante 48h
- Tela nova `E-ACCOUNT-DELETE`: equivalente para empresa
- Tela síndico S-ENCER-MANDATO: aviso "Dados do mandato preservados 5 anos"
- Componente `DeletionTimeline` (novo) — exibe: solicitado → 48h cooldown → pseudonimizado → retido 5y → hard delete

**Ação Sprint 10 (essencial)**: criar M-ACCOUNT-DELETE + E-ACCOUNT-DELETE.md (stubs mínimos); registrar DeletionTimeline no design-system backlog.

### D-103 — Score duplo

**Front impacto**:
- ✅ [[web/ui-catalog/sindico/S32-score-governanca]] criada
- ✅ [[web/ui-catalog/compliance/C10]] atualizada (Q25 resolvida)
- Criar `web/ui-catalog/morador/M-PERFIL-SINDICO`: morador vê perfil do síndico do condomínio (2 scores públicos)
- Criar `mobile/ui-catalog/sindico/S-PERFIL`: espelho mobile compacto
- Componentes novos:
  - `ScoreGauge` (variants 1-10 / 0-100) — obrigatório M1
  - `ScoreHistory` (linha gráfico) — drill-down
  - `ScoreBreakdown` (lista ponderada) — drill-down

**Ação Sprint 10**:
- Criar M-PERFIL-SINDICO.md web + S-PERFIL mobile (stubs);
- Planejar ScoreGauge no design-system (esforço S, M1);
- Planejar ScoreHistory + ScoreBreakdown (esforço M, M2).

### D-104 — Avaliação escondida em eleição

**Front impacto**:
- ✅ [[web/ui-catalog/assembleia/ASM-AVAL-ELEICAO]] criada
- ✅ [[mobile/ui-catalog/assembly/ASM-AVAL-ELEICAO]] criada
- Patch [[web/ui-catalog/assembleia/ASM-VOTO]]: bloqueio server-side + UI quando pauta é `eleicao_sindico` e morador não avaliou
- Componentes novos:
  - `HiddenAssessmentForm` (novo, crítico M3)
  - `NavGuard` (novo, reutilizável em outros flows obrigatórios)
  - `ConfidentialBanner` (novo, pequeno)

**Ação Sprint 10**: patch ASM-VOTO com bloqueio; planejar components M3.

### D-105 — ValidationItem Opção B DDD puro

**Front impacto**:
- Patch [[web/ui-catalog/sindico/S21]], [[web/ui-catalog/sindico/S22]], [[web/ui-catalog/sindico/S26]]: adicionar nota que API consome view agregada `pending_validations`
- Componente `ValidationQueueItem` (novo, unificado): exibe qualquer um dos 5 tipos (ServiceRecord, TechnicalActivity, PostDealAnnouncement, Proposal, Adjustment) com UI consistente
- Drill-down: clicar em item abre tela específica do tipo ([[web/ui-catalog/empresa/E9]], [[web/ui-catalog/empresa/E6]], etc.)

**Ação Sprint 10**: patch S21/S22/S26; stub ValidationQueueItem.

## 5. Componentes DS novos consolidados (11)

| Componente | Stack | Frequência (telas) | Prioridade | Esforço | M-# |
|---|---|---|---|---|---|
| `QuotaBadge` | web/mobile | 3+ | M1 | S | 1 |
| `PlanBadge` | web/mobile | 30+ | M1 | S | 1 |
| `LimitCounter` | web | 5+ | M1 | S | 1 |
| `ScoreGauge` | web/mobile | 3+ | M1 | M | 1 |
| `ScoreHistory` | web | 1 (drill-down) | M2 | M | 2 |
| `ScoreBreakdown` | web | 2 (drill-down) | M2 | S | 2 |
| `LiveCreationForm` | web | 2 | M3 | L | 3 |
| `DeletionTimeline` | web/mobile | 3 | M2 | M | 2 |
| `HiddenAssessmentForm` | web/mobile | 1 | M3 | L (NavGuard + slider 1-10) | 3 |
| `NavGuard` | web/mobile | multi | M3 | S | 3 |
| `ValidationQueueItem` | web | 3 | M1 | M | 1 |

**Total M1**: 5 components (QuotaBadge, PlanBadge, LimitCounter, ScoreGauge, ValidationQueueItem) + 3 telas patch + 2 telas criar (M-ACCOUNT-DELETE, M-PERFIL-SINDICO)
**Total M2**: 2 components (ScoreHistory, ScoreBreakdown, DeletionTimeline)
**Total M3**: 3 components (LiveCreationForm, HiddenAssessmentForm, NavGuard) + 2 telas (LMS-LIVE-CRIAR, já S32/ASM-AVAL-ELEICAO ok)

## 6. Cronograma sugerido

### Sprint 10 (2026-04-24 → 2026-05-18, M1 Opção A)

**Semana 1 (até 2026-05-04)**: telas + patches
- Dia 1-2: criar M-ACCOUNT-DELETE + E-ACCOUNT-DELETE + M-PERFIL-SINDICO + S-PERFIL mobile (stubs)
- Dia 3-4: patches S2, S21-S26, E1, M12, ASM-VOTO, LMS8
- Dia 5: sed cleanup N1/N2/N3 em ui-catalog; validar Fase 11 fechada

**Semana 2 (até 2026-05-11)**: design-system components
- QuotaBadge, PlanBadge, LimitCounter (3 componentes atômicos, esforço S)
- ScoreGauge (esforço M — 2 variantes + animação)
- ValidationQueueItem (esforço M — adapter de 5 DTOs)

**Semana 3 (até 2026-05-18)**: M1 ship
- Polish + integration tests das telas M1
- Dual-check final antes de ship

### M2 (até 2026-06-20)
- DeletionTimeline, ScoreHistory, ScoreBreakdown
- M-PERFIL-SINDICO full (com drill-downs)
- Connect Me full (Morador, Síndico), Conteúdo/Vídeos

### M3 (até 2026-07-20)
- HiddenAssessmentForm + NavGuard
- Assembly live completa (Livekit + todas telas ASM-*)
- LMS completa + LiveCreationForm

## 7. Riscos

| Risco | Probabilidade | Impacto | Mitigação |
|---|---|---|---|
| Web M1 não conseguir implementar 5 components + 3 telas novas em 2 semanas | 🟡 médio | Alto (não ship M1) | Reduzir escopo M1: adiar QuotaBadge + LimitCounter para M2 |
| `ScoreGauge` animation complexity > S | 🟡 médio | Baixo | Stub inicial sem animação; melhorar em M2 |
| `NavGuard` conflito com TanStack Router history | 🟡 baixo | Médio M3 | Prototipar cedo em M2 |
| Backend não expor endpoint `/scores` combinado em tempo | 🟡 baixo | Médio | S32 consome 2 endpoints separados (governance + compliance) como fallback |

## 8. Links

- [[../STATE#D-094]] até [[../STATE#D-105]] — decisões de origem
- [[traceability]] — matriz end-to-end
- [[web/ui-catalog/_moc]] — ui-catalog web
- [[mobile/ui-catalog/_moc]] — ui-catalog mobile
- [[web/design-system]] — tokens e components web
- [[mobile/design-system]] — tokens e components mobile
- [[../11-audit/audit-log/2026-04-24-gap-analysis-consolidado]] — gaps correlatos (pré-decisões)

## 9. Pendências para próxima revisão

- Confirmar com produto texto literal das 3-5 perguntas da avaliação escondida (D-104)
- Validar fórmula INV-GOV-001 (pesos 40/25/15/10/10) com stakeholder
- Decidir se `MK-ONBOARDING` (convite agência) é tela nova ou reusa `_auth/sign-up`
- Validar divergência OKLCH paleta web (`#C49A2A`) vs mobile (`#C69C3F`) — abrir D-### específica
