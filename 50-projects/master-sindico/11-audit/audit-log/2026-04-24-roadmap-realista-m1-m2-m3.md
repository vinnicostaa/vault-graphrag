---
title: Roadmap realista M1/M2/M3 — pós-análise 2026-04-24
type: audit
tags:
  - master-sindico
  - roadmap
  - milestones
  - planning
project: master-sindico
date: '2026-04-24'
---

# Roadmap realista M1/M2/M3 — pós-análise 2026-04-24

## Sumário executivo

- **M1 em 2026-05-08 é INVIÁVEL** — web praticamente não iniciado (5/6 apps placeholders; deps tanstack/solid-query, msw, vitest não instalados; cms/auth são stubs; finding A-FASE10-DEV-005 🔴). Gap analysis consolidado §6 já carrega a mesma premissa na pré-condição global.
- **Recomendação primária**: **Opção A — postergar M1 para 2026-05-18** (+10 dias úteis) e manter um escopo reduzido e realista (auth + cms morador M1-M5 + síndico S1-S6 + lp + 5 DS M1), NÃO as 181 telas web do catálogo.
- **Fallback**: **Opção C — split M1.0 (backend + mobile-only, 2026-05-08) + M1.1 (web, 2026-05-22)**. Permite marcar “M1 contratual entregue” formalmente se o contrato aceitar entrega parcial.
- **M2 (2026-06-20)** e **M3 (2026-07-20)** ficam com datas originais, mas com ajuste de escopo: Connect Me full + Reputação + Vizinhança continuam M2; Assembly + LMS + BT onboarding continuam M3. Admin plataforma, LMS full e i18n vão para M4+.
- **Confiança na entrega M1 (Opção A)**: 🟡 MÉDIA (≈60%). Com Opção C a confiança sobe para 🟢 ≈80% em M1.0 e 🟡 ≈55% em M1.1.

---

## Premissas

- Stakeholder **João** disponível para decisões (resolução das ~8 Q-### abertas em 2-3 dias úteis — ver [[2026-04-24-pendencias-para-stakeholder]] se existir, senão será criado).
- Front-end: ~3 pessoas × 40h = 120h/semana (assumido; validar com João).
- Backend continua ritmo Sprint 9: fechou com 12/14 BeyondCorp, gosec 0, govulncheck 0, 16 A-### fechados, **33 features entregues em 10h autônomo** — evidência de maturidade.
- Mobile: criado 2026-04-21; assembly já codado (M3), mas timeline/home M1 placeholder (A-FASE10-DEV-004). Pivot viável.
- Infra: Railway + Zitadel + Stripe + Mux + Livekit operacionais (Sprint 9 fechou).
- Autonomia técnica autorizada: agente decide D-### menores sem travar para João.
- Sprint 10 rodando; 20 ações priorizadas no gap analysis consolidado §6 (~14 dia-pessoa) cabem em 2 semanas com 3-4 agentes paralelos.

---

## M1 — MVP Contratual

### Data recomendada
**2026-05-18** (Opção A) — postergar 10 dias úteis do original 2026-05-08.

### Motivação (evidências)
- **A-FASE10-DEV-005 🔴 bloqueador M1**: 5/6 apps web são placeholders; `auth` e `cms` são stubs; `tanstack/solid-query`, `msw`, `vitest` não estão instalados. 15-25 dias de trabalho restante em web, 14 dias disponíveis. Matemática não fecha.
- Gap analysis consolidado §6 pré-condição global: “assumir Opção A (M1 = 2026-05-18) ou shipping M1 backend+mobile-only com web em onda M1.1”.
- **112 endpoints fantasma CRITICAL M1** precisam ser formalizados em `backend/requirements/institutional.md` antes de web consumir — finder 2026-04-24-gap-analysis-endpoints.md.

### Escopo acordado (realista)

#### Backend (Sprint 10 + possível Sprint 11)
- [X] Identity (Sprint 1) — entregue
- [X] Billing com Stripe (Sprint 1) — entregue
- [X] Institutional base (Sprints 2+3) — entregue
- [ ] Unit.fracao_ideal + Unit.unit_type migration (pendente M1; spec existe em 01-domain/aggregates/Unit.md)
- [ ] DT-010 IAuditLogger cross-BC completo (ADR-0037 + D-101; linka em A-FASE10-DEV-008)
- [ ] **112 endpoints CRITICAL M1 formalizados** em `backend/requirements/institutional.md` (§1 gap analysis; D-095 namespace web canônico)
- [ ] **221 ABAC actions** consolidadas em `04-requirements/abac-actions.md` (§2 gap analysis)
- [ ] A-023, A-024, A-039 (findings abertos Sprint 10)
- [ ] LGPD: soft_delete jobs + audit trail retenção 5y (D-101 + ADR-0037)
- [ ] Unit.fracao_ideal validação INV-CONDO-003 (soma = 1.00 em mandato ativo)

#### Web (onda única se Opção A; split se Opção C)
- [ ] App `auth` funcional: login OIDC Zitadel, callback, 1-device enforcement, session refresh
- [ ] App `cms` — **morador M1-M5 + síndico S1-S6** (11 telas reais; NÃO as 181 do catálogo)
- [ ] App `lp` completa: hero (existe) + pricing + sobre + contato + LGPD/ToS públicos
- [ ] **5 componentes DS M1**: PageHeader, FilterBar, Form+FormField, QuotaBadge, ScoreGauge (versão simples, número)
- [ ] Vitest + MSW + Playwright instalados e 1 smoke test por app
- [ ] Coverage ≥ 75% nos 3 apps M1 (auth, cms, lp)
- [ ] Storybook com 10 top componentes catalogados

#### Mobile (pivot: abandona assembly, foca morador)
- [ ] Splash + auth OIDC + 1-device
- [ ] Home morador (tela M1)
- [ ] Timeline read-only (tela M6)
- [ ] D-048/049/050 aplicadas: bloc_concurrency, hydrated_bloc, freezed, freeRASP report-only
- [ ] Coverage domain ≥ 95%, bloc ≥ 90% (padrão reputado Sprint 9)
- [ ] Assembly mobile **arquivado para M3** (código não descartado, só saído do Sprint 10)

### Fora de escopo M1 (empurrados para M2+)
- Telas S7+, M7+, E1+, MK1+, C1+ (web)
- LMS, BT, Vizinhança, Assembleia, Forum
- Score duplo (governança + compliance) UI completo — S32 pode ficar stub mostrando número simples; backend calcula
- Avaliação escondida em eleição (D-104 é M3)
- Painel Admin plataforma MS (GAP-J-02 — M4)
- BT onboarding 11 telas (M3)
- 4-6 telas Administradora Condominial (GAP-J-01 — pode ir M2 se houver folga)

### Dependências críticas M1 (hard gates)
| Dep | Prazo | Responsável |
|---|---|---|
| Stakeholder resolve Q-### pendentes | 2026-04-28 | João |
| DPO nomeado | 2026-05-05 | João |
| DPAs Mux/Zitadel/Twilio/Stripe assinados | 2026-05-10 | João + jurídico |
| Política privacidade + ToS publicados | 2026-05-12 | João + jurídico |
| 112 endpoints formalizados | 2026-05-01 | backend agent |
| 5 componentes DS M1 prontos | 2026-05-05 | web team |
| `04-requirements/abac-actions.md` consolidado | 2026-04-27 | product agent |

### Riscos M1
| Risco | Probabilidade | Impacto | Mitigação |
|---|---|---|---|
| Web não entrega 11 telas + auth em 3 semanas | 🟠 alto | M1 slippa para M1.1 | Opção C pré-aprovada como fallback; shipping parcial |
| Decisões de produto demoradas | 🟡 médio | Bloqueia implementação | Autonomia D-### menores + batch de decisões 2026-04-28 |
| LGPD DPAs atrasam | 🟡 médio | Bloqueia shipping regulamentar | Soft-launch interno (moradores cientes) |
| Sprint 10 backend introduz bugs | 🟡 médio | Retrabalho | QA reforçado; integration tests + dual-check em D-102/103 |
| Paleta OKLCH web vs mobile divergente | 🟢 baixo | Retrabalho visual pequeno | D-096 canônica decidida em 2026-04-25 |

### Critério de aceite M1 (DoR/DoD — ver DoR-DoD.md)
- Síndico cria conta → escolhe plano → trial passes → Stripe recorrência real → Zitadel produção + DNS `mastersindico.com.br`
- AUDIT 🔴🟡 = 0
- Coverage global ≥ 85% (backend), 75% (web M1), 95% domain mobile
- Todos gates CI verdes (gosec, govulncheck, staticcheck, lint, type-check)
- LGPD: DPAs assinados + política publicada + soft_delete funcional + audit trail 5y

---

## M1.1 (se Opção C — fallback)

### Data
**2026-05-22** (+14 dias após M1.0 backend+mobile)

### Escopo
- Web completo conforme M1 acima
- Validação integrada E2E entre backend + web + mobile

### Quando acionar
Se em 2026-05-02 (T-6 do M1 original) web ainda tiver menos de 3 telas M1 reais funcionando, acionar Opção C formalmente.

---

## M2 — Connect Me + Conteúdo + Reputação

### Data
**2026-06-20** (mantida)

### Escopo

#### Backend (features já existem no código; falta formalização + consolidação)
- Commercial full: Connect Me 4 direções (S→E, M→S, E→E+/Pro, S→Parceiro), Proposta, ClosedDeal, ServiceRecord, TechnicalActivity
- Content: Video Mux, MarketingAgencyLink, Paywall 25%, trava 90d vídeo institucional
- Reputação: motor score governança + compliance + reputação Bronze→Diamante (D-051, determinístico)
- Vizinhança: parcial (backend commercial isolado de assembly)
- **140 endpoints HIGH M2** formalizados
- OpenSearch ou tsvector migration (D-### pendente — decidir antes 2026-05-25)
- Moderação manual workflow

#### Web
- **Telas S7-S31** (restantes síndico, ~24 telas)
- **Telas E1-E16** (empresa, 16 telas)
- **Telas MK1-MK8** (marketing agências, 8 telas)
- **Telas C1-C11** (compliance síndico, 11 telas)
- App `forum` (Vizinhança) — novo app
- Telas S32 completa + drill-down breakdown scores
- **Componentes DS M2**: ScoreHistory, ScoreBreakdown, DeletionTimeline, ValidationQueueItem, ConfirmContextModal, VoiceTextarea, ExportButton, BadgeStatus unificado, FileUpload unificado
- Coverage ≥ 85% todos apps M1+M2

#### Mobile
- Connect Me (direção M→S)
- Vizinhança VM1-VM7 (7 telas mobile)
- Avaliação bimestral M12
- Eventos M10 + Comunicados M11
- Coverage mantém 95% domain / 90% bloc

### Dependências M2
- M1 fechado e operando em produção ≥ 7 dias
- Backend reputação score determinístico implementado e auditado
- OpenSearch ou tsvector decisão confirmada (D-### pendente)
- Mux produção com paywall validado

### Riscos M2
- **Connect Me 4 direções é complexidade alta** — arquitetura comum mas UX diverge; risco de retrabalho se não houver spec compartilhada (verificar `00-product/sub-produtos/connect-me.md`).
- **OpenSearch vs tsvector**: decisão tardia pode custar 3-5 dias.
- **Moderação manual**: capacidade operacional (quem modera?) — definir com João.

---

## M3 — Assembleia + LMS + Polish

### Data
**2026-07-20** (mantida)

### Escopo

#### Backend
- Assembly live completo com Livekit (já implementado; falta formalização + quórum fracional por `Unit.fracao_ideal` + ata imutável + edital PDF + proxy/procuração + fila de fala)
- LMS module novo (0% implementado) — thin: 5 áreas, 3 trilhas iniciais, certificados
- Banco de Talentos module novo (0% implementado) — vídeo-currículo 90s + favoritos + tags
- **Avaliação escondida eleição** (D-104) + `hidden_assessments` table
- **81 endpoints MEDIUM M3** formalizados

#### Web
- App `lms` completa (LMS1-LMS15)
- App `assembly` completa: ASM-CFG, PAUTA, EDITAL, PUB, TELAO, VOTO, HOML, ENCER, REL + ASM-AVAL-ELEICAO (D-104)
- Telas BT (empresa busca, favoritos, tags)
- **Componentes DS M3**: HiddenAssessmentForm, NavGuard, LiveCreationForm, SpeakerQueueDialog, Telão viewer
- A11y WCAG 2.1 AA polish
- SLO 99.5% rotas críticas

#### Mobile
- ASM-CHKIN + ASM-VOTO + ASM-AVAL-ELEICAO (reaproveitando assembly arquivado em Sprint 10)
- LMS básico
- BT01-BT11 (onboarding BT, 11 telas)
- BiometricAuth (para check-in assembleia)

### Dependências M3
- M2 fechado e operando em produção ≥ 7 dias
- Livekit WebSocket <200ms P95 validado em staging
- Mux Live streaming validado
- LMS aggregates novos (Course, Module, Lesson, Enrollment, Certificate, ForumTopic) criados em `01-domain/aggregates/`
- 4 eventos LMS órfãos resolvidos (CourseCompleted, VideoWatched, TopicReplied, CertificateIssued)
- 7 INVs órfãos formalizados (INV-ASM-015 voto secreto por item, INV-LMS-003 trilha progresso, INV-BT-010 trava 90d vídeo, etc)

### Riscos M3
- **Livekit custo + latência** em escala — plano B: WebRTC custom (alto esforço, arrisca M3)
- **LMS do zero em 30 dias** — escopo thin obrigatório; se escopo inflar, corta trilhas
- **Assembly + LMS em paralelo** — 2 frentes novas simultâneas é arriscado; priorizar Assembly (contratual)

---

## Pós-M3 — M4 e além

- **M4 (Q3 2026)**: Admin plataforma MS (GAP-J-02), validação Receita Federal, compliance NR-1/ABRACS, ML moderação, breach notification 72h automática
- **M5 (Q4 2026)**: Empresa-síndica multi-condomínio, Agência multi-empresa, vizinhança geográfica avançada, assembleia híbrida
- **M6 (2027)**: AI copiloto síndico, FCM priorização ML, mobile feature parity + offline-first
- **Transversais contínuos**: i18n, WCAG 2.1 AA polish, SBOM + supply chain, NATS JetStream produção, Passkeys WebAuthn admin

---

## Riscos transversais

| Risco | Probabilidade | Impacto | Mitigação |
|---|---|---|---|
| Web não entrega M1 em 3 semanas mesmo com Opção A | 🟠 alto | M1 slippa | Opção C pré-aprovada (M1.0 + M1.1) |
| LGPD DPAs atrasam | 🟡 médio | M1 bloqueado | Soft-launch interno + comunicação clara a usuários |
| Stakeholder indisponível | 🟢 baixo (autonomia OK) | Atrasa decisões | Agente decide D-### menores |
| Backend Sprint 10 introduz bugs pós-implementação | 🟡 médio | Retrabalho | QA reforçado + integration tests + dual-check |
| Escopo inflando (“queria também...”) | 🟠 alto | Derruba M2/M3 | ROADMAP congelado + CR process |
| Time mobile não consegue pivot limpo (abandonar assembly) | 🟡 médio | M1 mobile fica incompleto | Assembly NÃO é deletado, só arquivado; retoma M3 |
| 500 componentes DS órfãos sem resolução | 🟡 médio | Retrabalho web M2 | D-094 aliases + 5 componentes M1 como âncora |

---

## Ações concretas hoje (2026-04-24)

### Para João (stakeholder)
1. **Decidir Opção A / B / C** — impacta data-alvo de tudo (urgente, T+0)
2. **Responder ~5 Q-### ainda em aberto** (listados em [[2026-04-24-pendencias-para-stakeholder]] se existir)
3. **Confirmar capacidade do time web** (~3 pessoas × 40h?)
4. **Acionar jurídico** para DPAs + política + DPO (janela 2026-04-24 a 2026-05-12)

### Para o agente (técnico)
5. **Formalizar 112 endpoints** em `backend/requirements/institutional.md` (1 dia)
6. **Consolidar `04-requirements/abac-actions.md`** (0.5 dia — 221 actions)
7. **Criar aggregates faltantes**: ValidationItem, ComplianceDeclaration, TermAcceptance (0.5 dia)
8. **Rename 32 frontmatters** `sub_produto:` não-canônico (15 min)
9. **Rename 5 `plan_requirement: premium`** → canônico (10 min)
10. **Stub `admin-mfa-lockout.md`** em `12-docs/runbooks/` (0.25 dia)

### Para web team
11. **Iniciar app `auth` real** (OIDC Zitadel — hoje é stub)
12. **Instalar deps**: tanstack/solid-query, msw, vitest, playwright, storybook
13. **Iniciar 5 telas M1 reais**: M1-M3 síndico S1-S3 (prova de conceito)

### Para mobile team
14. **Pivot: arquivar assembly**, iniciar home morador + timeline read
15. **Aplicar D-048/049/050** (bloc_concurrency, hydrated_bloc, freezed, freeRASP report-only)

### Para backend team
16. **Sprint 10 prioridade**: 112 endpoints + Unit migration + IAuditLogger cross-BC
17. **Não tomar dívida nova** até A-023, A-024, A-039 fecharem

---

## Resumo datas

| Marco | Data original | Data recomendada | Confiança |
|---|---|---|---|
| M1 (Opção A) | 2026-05-08 | **2026-05-18** | 🟡 60% |
| M1.0 (Opção C, fallback) | — | 2026-05-08 | 🟢 80% |
| M1.1 (Opção C, fallback) | — | 2026-05-22 | 🟡 55% |
| M2 | 2026-06-20 | **2026-06-20** (mantida) | 🟡 65% |
| M3 | 2026-07-20 | **2026-07-20** (mantida) | 🟡 55% |

---

## Links

- [[../STATE]]
- [[../AUDIT]]
- [[../ROADMAP]]
- [[../10-schedule/milestones]]
- [[../10-schedule/cronograma-macro]]
- [[../10-schedule/calendar]]
- [[../03-subprojects/feature-plan-2026-04-24]]
- [[../03-subprojects/traceability]]
- [[../04-requirements/matrix-functional]]
- [[2026-04-24-gap-analysis-consolidado]]
- [[2026-04-24-gap-analysis-endpoints]]
- [[2026-04-24-gap-analysis-dominio]]
- [[2026-04-24-gap-analysis-produto-processos]]
- [[2026-04-24-gap-analysis-design-system]]
