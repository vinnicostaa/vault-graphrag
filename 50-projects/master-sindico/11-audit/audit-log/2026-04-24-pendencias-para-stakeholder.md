---
title: Pendências para stakeholder João — 2026-04-24
type: audit
tags:
  - master-sindico
  - stakeholder
  - pendencias
  - decisoes
  - m1
project: master-sindico
created: 2026-04-24T00:00:00.000Z
audience: joão (stakeholder)
---

# Pendências para stakeholder — 2026-04-24

Consolidação de TUDO que precisa de decisão ou ação externa (não pode ser resolvido apenas pelo agente). Varredura sistemática sobre STATE, AUDIT, quebras-detectadas, feature plan e os 5 gap analyses de 2026-04-24.

> **Contexto de prazo**: M1 contratual = 2026-05-08. Web M1 praticamente não iniciado (A-FASE10-DEV-005). Sprint 10 em curso. Fase 11 (9 decisões) + Fase 12 (3 decisões) fecharam 6 Q-### e 2 bloqueadores M1 em 2026-04-24.

## Sumário executivo

- **Decisões de produto ainda em aberto**: 7 (texto da avaliação escondida, fórmula governança, MK-ONBOARDING, paleta OKLCH, ValidationItem ≡ ServiceRecord, tipos de evento 13 vs 18, Empresa Plus E→E cap).
- **Bloqueadores LGPD M1 (humano/comercial)**: 6 dos 7 originais (LGPD-M1-007 fechado via D-101).
- **Bloqueadores técnicos M1 críticos sem dono claro**: 4 (A-FASE10-DEV-005 Web M1, A-FASE10-DEV-003 mobile stack D-048/049/050, A-FASE10-DEV-007 crons sem scheduler, GAP-J-02 admin master-sindico).
- **Escolha macro M1 pendente**: Opção A (postergar para 2026-05-18) vs Opção B (ship 08/05 com known-issues) vs Opção C (remontagem continua).
- **Divergências remanescentes**: 5 telas mobile `plan_requirement: premium` + 32 frontmatters `sub_produto:` não canônico + 3 telas `persona: administradora`.
- **Gaps críticos M1 pós-F11/F12**: 112 endpoints fantasma, 76 ABAC actions M1-críticas órfãs, 3 aggregates ausentes (ValidationItem, ComplianceDeclaration, TermAcceptance — D-105 resolveu ValidationItem por design).

**Recomendação de prioridade HOJE**: resolver a **Opção A/B/C (decisão macro M1)** e **iniciar tratativa comercial/jurídica dos 6 bloqueadores LGPD** (DPO + 3 DPAs + DPIA + política de privacidade). Sem isso tudo o resto escorrega.

---

## Parte 1 — Decisões de produto em aberto (requer João responder)

Estas 7 decisões só podem ser fechadas pelo stakeholder. Impacto direto em implementação M1/M2/M3.

| # | Pendência | Contexto | Opções | Impacto M1 | Urgência |
|---|---|---|---|---|---|
| P-01 | **Texto literal das 3-5 perguntas da avaliação escondida de gestão em eleição** | D-104 fechou a regra (ATIVAR spec M1, impl M3) mas o questionário não foi redigido. `03-subprojects/feature-plan-2026-04-24.md:212`; `03-subprojects/web/ui-catalog/assembleia/ASM-AVAL-ELEICAO.md` | a) João redige; b) produto propõe e João valida | Nenhum (M3 impl) | 🟡 pode esperar Sprint 11 |
| P-02 | **Fórmula exata INV-GOV-001 (Score Governança 1-10)** | D-103 fechou score duplo mas pesos 40/25/15/10/10 ainda são stub; validar com stakeholder. `03-subprojects/web/ui-catalog/sindico/S32-score-governanca.md:108`; `03-subprojects/web/ui-catalog/assembleia/ASM-AVAL-ELEICAO.md:118` | a) Congelar stub 40/25/15/10/10; b) Ajustar pesos; c) Pedir feedback de 2-3 síndicos piloto | Baixo (score calcula mas valores podem sair errados) | 🟠 resolver antes de seed produção |
| P-03 | **MK-ONBOARDING: tela nova ou reusar `_auth/sign-up`?** | Feature plan §2 D-095. Fluxo de convite de agência a ser aceito por empresa. `03-subprojects/feature-plan-2026-04-24.md:214` | a) Criar MK-ONBOARDING próprio; b) Reutilizar sign-up com flag | Baixo (M5 é gated) | 🟢 pode ficar M5 |
| P-04 | **Paleta OKLCH canônica: web `#C49A2A` vs mobile `#C69C3F`** | Gap analysis DS §3; feature plan §9. Divergência real. `03-subprojects/feature-plan-2026-04-24.md:215` | a) Escolher web; b) Escolher mobile; c) Criar novo valor médio | Alto (brand consistency; aparece em todas telas M1) | 🔴 resolver antes Sprint 10 DS |
| P-05 | **Empresa Plus E→E Connect Me: cap numérico ou ilimitado?** | Q8 quebras-detectadas 🟡 aberto. `personas-and-plans`, `functional-matrix` e `commercial.md` Req 19.2 discordam. `_archive/2026-04-23-pre-v2-migration/content/regras-de-negocio/quebras-detectadas.md:130` | a) Ilimitado (como Pro); b) N contratado; c) 20/ano default | Médio M2 (Connect Me) | 🟠 antes do seed Sprint 10 |
| P-06 | **Tipos de evento: 13 canônicos ou 18+ do briefing?** | Gap analysis domínio §C; S16 já registra "consolidar enum". Divergência briefing (Inventario-Telas) vs Enums-Consolidados. `11-audit/audit-log/2026-04-24-gap-analysis-dominio.md` §C | a) Aceitar 13; b) Extender para 18+; c) Os 5 extras são comunicados mal classificados | Médio (UI de criar evento M1) | 🟠 Sprint 10 |
| P-07 | **ValidationItem ≡ ServiceRecord ou aggregates distintos?** | D-105 escolheu Opção B (DDD puro, aggregates distintos + interface). Confirmação: criar stubs ServiceRecord, TechnicalActivity, Adjustment? | a) Confirma D-105 (stubs novos); b) Unificar em 1 aggregate | Médio M1 (S21/S22/S26) | 🟡 D-105 já fechou, apenas confirmação operacional |

---

## Parte 2 — Bloqueadores LGPD M1 (ação humana/comercial)

**6 de 7 bloqueadores LGPD M1 ainda abertos.** O 7º (LGPD-M1-007 race hard-delete) foi fechado em 2026-04-24 via D-101 + ADR-0037. Os demais dependem de **pessoas reais** (jurídico, comercial, DPO humano) — nenhum agente resolve.

Fonte canônica: `AUDIT.md:245-257`.

| ID | Bloqueador | Owner | O que precisa acontecer | Data-alvo M1 |
|---|---|---|---|---|
| **LGPD-M1-001** | DPO nomeado publicamente (`lgpd@mastersindico.com.br`) | comercial/legal team | Nomeação formal + email + publicar em política de privacidade + LGPD Art. 41 | 🔴 M1 bloqueante |
| **LGPD-M1-002** | DPA com **Mux** assinado | comercial/legal team | Contrato bilateral (Data Processing Agreement) | 🔴 M1 bloqueante |
| **LGPD-M1-003** | DPA com **Zitadel** assinado | comercial/legal team | idem; Zitadel é processador de identity (PII crítico) | 🔴 M1 bloqueante |
| **LGPD-M1-004** | DPA com **Twilio** assinado | comercial/legal team | idem; SMS = canal de entrega de códigos | 🔴 M1 bloqueante |
| **LGPD-M1-005** | DPIA-001 executado (Data Protection Impact Assessment) | DPO humano + arquiteto | Avaliar contextos críticos (assembleia, Connect Me, termo responsabilidade) e documentar mitigações | 🔴 M1 bloqueante |
| **LGPD-M1-006** | Política de privacidade versionada publicada | legal + comercial | Redigir + versionar + publicar + linkar em onboarding | 🔴 M1 bloqueante |
| ~~LGPD-M1-007~~ | ~~DELETE /me race resolvido~~ | ✅ fechado 2026-04-24 | soft_delete universal + pseudonimização (D-101 + ADR-0037) | 🟢 |

**Nenhum dos 6 abertos tem dono formal registrado no vault** — todos atribuídos genericamente a "commercial/legal team" ou "DPO + arquiteto". É necessário **nomear responsável específico com data de compromisso**.

---

## Parte 3 — Gaps críticos ainda abertos (técnicos-M1)

Pós Fase 11+12, estes permanecem bloqueantes ou escorregam para M1.1:

### 3.1. Bloqueadores M1 técnicos sem dono
| ID | Gap | Origem | Fonte canônica |
|---|---|---|---|
| **A-FASE10-DEV-005** 🔴 | Web M1 não iniciado — 6 apps apenas scaffolded, sign-in/sign-up retornam `<div>Hello</div>`, zero testes, TanStack/MSW não instalados. Estimativa 15-25 dias; 2 semanas até 08/05 é matematicamente inviável. | `AUDIT.md:419-426` | `03-subprojects/web/tasks` |
| **A-FASE10-DEV-003** 🟠 | Mobile D-048/D-049/D-050 não aplicadas no Flutter — `pubspec.yaml` não tem `bloc_concurrency`, `hydrated_bloc`, `freezed`, `freeRASP`, `hive_flutter`, `local_auth`, `device_info_plus`, `firebase_messaging`, `app_links`. | `AUDIT.md:402-408`; `03-subprojects/mobile/README.md:42-82` | — |
| **A-FASE10-DEV-007** 🟡 | 3 domain files chamam "cron noturno" sem scheduler existir (`connect_me_request.go:158`, `master_plan_activity.go:189`, `poll.go:113`). ADR de scheduling pendente. | `AUDIT.md:438-444` | — |
| **GAP-J-02** 🔴 | 0 telas para role `admin` — 5 capacidades Admin MS órfãs (dashboard financeiro, gestão usuários, moderação, config plataforma, comunicados oficiais). DPO não tem UI para atender LGPD request. | `03-subprojects/web/ui-catalog/admin/GAP-admin-master-sindico.md` | Backlog M2/M3 declarado em D-102 |
| **GAP-J-01** 🔴 | 0 telas para sub-produto `11-administradoras-condominiais`. 5 endpoints REST (`empresa_sindica_handler.go`) sem UI (convite/aceite/revoke). | Gap analysis produto-processos §H.1 | — |
| **GAP-J-04** 🟠 | Runbook `12-docs/runbooks/admin-mfa-lockout.md` citado em `admin-privilegios.md:343,660` mas **não existe**. Pré-requisito de ADR-0026 (MFA admin M1). | — | — |

### 3.2. Bloqueadores QA pré-ship
| ID | Gap | Esforço |
|---|---|---|
| **A-DC-QA-001** | Matriz `09-testing/test-traceability.md` req→test_id não formaliza 99 requisitos | 1-2h |
| **A-DC-QA-060** | `state-machines.md` precisa seção "transição → test_id" por SM | 1h × 10 SMs |
| **A-DC-QA-071** | Inconsistência matriz funcional diz Síndico N1 publica vídeo ❌ vs BIL-017 diz 4/mês — reconciliar (D-096 fechou N1/N2/N3 mas esta inconsistência específica em test-traceability.md:169 está marcada "Pendente") | 30min |
| **A-DC-QA-030** | Agência Marketing ui-catalog subdimensionada (3 telas na era pre-F11; agora 8 — confirmar se já está OK) | verificar |

### 3.3. Aggregates ainda ausentes (pós-D-105)
D-105 resolveu ValidationItem por design (não cria aggregate). Mas restam 12 aggregates declarados em `ubiquitous-language.md` sem arquivo:
- **🔴 M1 crítico**: ComplianceDeclaration (C2 M1), TermAcceptance (onboarding M1)
- **🟠 M2**: VideoVisibilityGrant, LGPDLog, Amendment, SpeechQueue
- **🟡 M3**: Curriculum, NeighborhoodBenefit, ForumTopic, LibraryMaterial, LMSLive, PreliminarySurvey

Fonte: `11-audit/audit-log/2026-04-24-gap-analysis-dominio.md` §B.

### 3.4. Endpoints fantasma e ABAC órfãs (do gap analysis consolidado)
| # | Classe | Quantidade total | Bloqueador M1 |
|---|---|---|---|
| 1 | Endpoints fantasma (telas citam, backend não formaliza) | 333 | **112** |
| 2 | ABAC actions órfãs (usadas em telas, ausentes em matriz canônica) | 221 | **~76** |
| 3 | Componentes DS órfãos | 500 (436 web + 64 mobile) | **9 web** |

Nenhum é bug funcional hoje — **todos são gap de formalização/spec**. Backend roda; problema é que seed da matriz ABAC produção vai derivar de telas ao invés de spec canônica, causando drifts sutis.

---

## Parte 4 — Escolha macro M1 (Opção A/B/C — DECISÃO URGENTE DE JOÃO)

Fonte canônica: `AUDIT.md:284-301`.

### Opção A — Postergar M1 para **2026-05-18** (recomendação técnica)
- Pausa remontagem, fixam-se 14 🔴 (restam 12 depois que LGPD-M1-007 e A-DC-SEC-004 foram fechados) + 6 bloqueadores LGPD, Sprint 10+ executa fixes.
- **Ship M1 firme em 18/05 com AUDIT 🔴 = 0**.
- Custo: ~10 dias úteis + coordenação comercial (LGPD DPAs).
- **Pré-condição**: decisão formal de João HOJE; início imediato das tratativas LGPD.

### Opção B — Ship M1 em **2026-05-08 com known-issues**
- Remontagem migra pro vault (Fase 6) agora; Sprint 10 fecha M1 operacional.
- Known-issues documentados em runbook + hotfix M1.1 até 18/05 cobre 🔴.
- **Risco LGPD**: produção sem DPO/DPAs/DPIA gera exposição regulatória (ANPD pode multar; cliente master-sindico pode exigir DPA antes de aceitar).

### Opção C — Continuar remontagem com fixes inline (não migrar ainda)
- Agentes adicionais fazem fixes nos artefatos v2 (INV-101..110, ADRs 0026-0031, reconciliações QA).
- Migração Fase 6 só quando AUDIT 🔴 = 0 **nos artefatos** da v2.
- Alinhado com "não temos pressa" + "minucioso".

**Data-alvo de decisão**: 2026-04-24 (hoje) para não perder mais 24h.

**Recomendação técnica**: **Opção A**. A web M1 (A-FASE10-DEV-005) sozinha torna ship 08/05 matematicamente inviável. A Opção B exige aceitar risco LGPD. A Opção C é consistente mas não fecha uma data comercial.

---

## Parte 5 — Divergências remanescentes (pós Fase 11+12)

Deveria haver pouquíssima coisa — confirmação da varredura:

### 5.1. N1/N2/N3 em contexto canônico vivo
**0 ocorrências operacionais detectadas** (grep `plan_requirement: n1|n2|n3` = 0; citações remanescentes são "tradução histórica" legítima após D-096). ✅ D-081+D-096 aplicados corretamente.

### 5.2. `plan_requirement: premium` (enum fora do canônico)
5 telas mobile ainda com valor obsoleto — devem virar `plus` ou `plus-or-pro`:
- `03-subprojects/mobile/ui-catalog/assembly/ASM-CHKIN.md:9`
- `03-subprojects/mobile/ui-catalog/assembly/ASM-VOTO.md:9`
- `03-subprojects/mobile/ui-catalog/assembly/ASM-FILA-FALA.md:9`
- `03-subprojects/mobile/ui-catalog/assembly/ASM-CIEN.md:9`
- `03-subprojects/mobile/ui-catalog/assembly/ASM-PROC-CAD.md:9`

Impacto: quebra ABAC se backend rejeitar valor. Fix: 10min sed.

### 5.3. Frontmatter `sub_produto:` não canônico
32 ocorrências em 32 arquivos:
- **23 web**: `sub_produto: onboarding` (não é sub-produto — é fase transversal)
- **6 mobile**: `sub_produto: banco-talentos` (slug correto é `banco-de-talentos`)
- **3 mobile**: `sub_produto: marketing` (slug correto é `marketing-agencias`)

Fix: 15min sed. **Escolha**: João confirma política de reclassificação por persona (ONB-S*→governanca-institucional, ONB-E*→connect-me/banco-de-talentos, ONB-P*→marketing-agencias, ONB-M*→governanca-institucional)?

### 5.4. Persona `administradora` (não canônica)
3 telas com persona não canônica — as 6 canônicas Zitadel são `syndic|resident|enterprise|marketing|local_company|admin`. Administradora é subcategoria de `enterprise`, não role. Fix: migrar para `persona: enterprise` + marker `subcategoria: administradoras_condominios`.

### 5.5. `persona: any/all/todos` ambíguo
18 telas genéricas — ABAC engine não sabe quem pode renderizar. Precisa revisão caso-a-caso para listar roles explícitas.

---

## Parte 6 — Pendências operacionais detectadas em varredura

Itens ⚠️ PENDÊNCIA/TODO identificados em notas específicas (pós grep sistemático):

| Local | Pendência | Contexto |
|---|---|---|
| `04-requirements/functional/assembly.md:524` | **ASM-032** — decidir entre M3 assíncrona vs live — reconciliar matriz cliente vs specs legado | P-INV-007 / D-053 fechou homologação escalonada, mas ASM-032 teste continua "Pendente" |
| `09-testing/test-traceability.md:358` | ASM-032 teste integração marcado "⚠️ Pendente (P-INV-007)" | — |
| `09-testing/test-traceability.md:169-188` | BIL-017 teste dependente de A-DC-QA-071 reconciliado — nota explica mas marca "Pendente" | — |
| `03-subprojects/web/ui-catalog/sindico/S32-score-governanca.md:108` | **Fórmula governance_score (INV-GOV-001)** — stub 40/25/15/10/10 a confirmar com produto | P-02 na seção 1 |
| `03-subprojects/web/ui-catalog/assembleia/ASM-AVAL-ELEICAO.md:118` | **Fórmula de agregação** — média simples vs ponderada a definir | P-01 na seção 1 |
| `03-subprojects/web/ui-catalog/assembleia/ASM-PAUTA.md:83` | Q40 "eleição gera avaliação escondida" não mapeada em `assembly.md` canônico (D-104 fecha em nível de produto mas spec ainda precisa propagar) | Sprint 10 |
| `03-subprojects/feature-plan-2026-04-24.md:212-215` | 4 pendências explícitas: (1) texto perguntas D-104; (2) fórmula INV-GOV-001; (3) MK-ONBOARDING reusa ou cria; (4) paleta OKLCH D-### nova | consolidado em P-01..P-04 |
| `12-docs/runbooks/admin-mfa-lockout.md` | **Ausente** mas citado em `admin-privilegios.md:343,660`; `dr-drill.md` ausente mas citado em sprint-10.md:41 | GAP-J-04 |
| Dívidas mobile (`03-subprojects/mobile/README.md:42-82`) | 15+ deps do Flutter marcadas "⚠️ ausente — dívida M1/M2/M3" (bloc_concurrency, hydrated_bloc, freezed, hive_flutter, local_auth, device_info_plus, firebase_messaging, app_links, freeRASP, sentry_flutter, mux_player_flutter, image_picker/camera, speech_to_text, livekit_client, flutter_windowmanager) | A-FASE10-DEV-003 |

### Findings legado ainda abertos herdados
Registrados em STATE.md:94-100 e STATE.md:486-495 (não revisitados pela Fase 11/12):
- D-001 (legado): cache TTL introspection Zitadel — 🟡 aberto
- D-002 (legado): estratégia testes regressão SDD — 🟡 aberto
- D-030 (legado): `.claude/settings.json` web/app — 🟡 aberto
- DT-003 (legado): circuit breaker Zitadel
- DT-004 (legado): timeout UoW.Run
- DT-006 (legado): `.gitignore` específico
- DT-007 (legado): divergência STRIPE_KEY_PRIV
- DT-010 (pós-fase-10): IAuditLogger LGPD incompleto cross-BC — **pré-M1 LGPD obrigatório** 🟠 aberto (`STATE.md:817-823`)

### Pendente promoção de invariantes órfãos
- 6 INV-BT-* (`01-domain/invariants.md` não tem; só em `00-product/sub-produtos/07`)
- INV-LMS-E (reclassificar como ABAC policy ou promover)
- INV-ASM-events (formalizar como INV-130; decidir se estende INV-119/120)
- INV-ASM-live (reclassificar como SLO, NÃO promover)
- INV-ASM-modalidade-mvp (reclassificar como escopo M1, NÃO promover)

Fonte: `11-audit/audit-log/2026-04-24-gap-analysis-dominio.md` §A.

---

## Parte 7 — Matriz priorizada (top 20 pendências)

Ordenada por impacto M1 × urgência × quem pode resolver.

| # | Pendência | Impacto M1 | Quem resolve | Esforço | Data-alvo | Tipo |
|---|---|---|---|---|---|---|
| 1 | **DECISÃO MACRO M1 Opção A/B/C** | 🔴 bloqueia tudo | João | 30min conversa | **2026-04-24** | Decisão macro |
| 2 | LGPD-M1-001 DPO nomeado publicamente | 🔴 M1 | comercial/legal | dias | antes 05/05 | Humano |
| 3 | LGPD-M1-002/003/004 DPAs (Mux, Zitadel, Twilio) | 🔴 M1 | comercial/legal | 1-3 semanas | antes 05/05 | Humano |
| 4 | LGPD-M1-005 DPIA-001 | 🔴 M1 | DPO+arquiteto | 2-3 dias | antes 05/05 | Humano |
| 5 | LGPD-M1-006 Política privacidade publicada | 🔴 M1 | legal+comercial | 3-5 dias | antes 05/05 | Humano |
| 6 | A-FASE10-DEV-005 Web M1 15-25 dias trabalho | 🔴 M1 | web team | 15-25 dias | depende Opção A | Dev (stakeholder-gated) |
| 7 | P-04 Paleta OKLCH canônica (web vs mobile) | 🔴 alto | João/DS lead | 10min decisão | Sprint 10 dia 1 | Decisão produto |
| 8 | GAP-J-01 Suite telas Administradoras (S-ADM-01..02 + E-ADM-01) | 🔴 M1 se cliente tiver adm | web agent | 2 dias | Sprint 10 | Dev |
| 9 | GAP-J-02 Admin MS: decidir se Gestão usuários + Moderação vídeo entram M2 ou M1 CLI-only | 🔴 parcial M1 | João + arch | 30min | Sprint 10 dia 1 | Decisão produto |
| 10 | GAP-J-03 Consolidar `04-requirements/abac-actions.md` (227 actions) | 🔴 M1 | product agent | 0.5 dia | Sprint 10 | Agente |
| 11 | Criar aggregates M1-críticos: ComplianceDeclaration + TermAcceptance | 🔴 M1 | domain agent | 0.5 dia | Sprint 10 | Agente |
| 12 | Formalizar 112 endpoints 🔴 em `backend/requirements/institutional.md` | 🔴 M1 | backend agent | 1 dia | Sprint 10 | Agente |
| 13 | P-06 Tipos de evento 13 vs 18+ | 🟠 médio | João/produto | 30min | Sprint 10 | Decisão produto |
| 14 | P-05 Empresa Plus E→E cap | 🟠 médio M2 | João | 30min | antes seed M2 | Decisão produto |
| 15 | A-FASE10-DEV-003 Mobile D-048/049/050 aplicar (ou re-decidir adiar M2) | 🟠 M1 | João + mobile lead | decisão 30min + 2-3 dias dev | Sprint 10 | Decisão + dev |
| 16 | DT-010 IAuditLogger LGPD cross-BC (pré-M1 obrigatório) | 🟠 M1 LGPD | backend agent | 1 dia | Sprint 10 | Agente |
| 17 | P-02 Validar fórmula INV-GOV-001 (pesos Score Governança) | 🟠 antes seed | João | 30min | Sprint 10 | Decisão produto |
| 18 | A-DC-QA-071 Reconciliar matriz N1 vídeo vs BIL-017 | 🟠 M1 QA | domain agent | 30min | Sprint 10 | Agente |
| 19 | Fix 5 telas `plan_requirement: premium` + 32 frontmatters não-canônicos + 3 `persona: administradora` | 🟡 cosmético/ABAC | bash script | 25min | Sprint 10 | Agente |
| 20 | Criar runbook stub `admin-mfa-lockout.md` + `dr-drill.md` | 🟡 pré-ADR-0026 | ops agent | 0.5 dia | Sprint 10 | Agente |

### Totais por categoria
- **Decisões macro do stakeholder (João)**: 4 (Opção A/B/C; P-02 fórmula; P-04 paleta; P-06 tipos evento)
- **Decisões comerciais/legais**: 6 (os 6 LGPD-M1-00X)
- **Dev team (bandwidth)**: 2 (Web M1; mobile D-048/049/050)
- **Agentes (resolvível autônomo pós-decisão)**: 8 (endpoints, aggregates, ABAC, QA, frontmatter, runbooks)

---

## Links

- [[../../STATE]] — decisões D-001..D-105
- [[../../AUDIT]] — findings abertos
- [[../../ROADMAP]] — M1 08/05 (ou 18/05 Opção A)
- [[../../03-subprojects/feature-plan-2026-04-24]] — plano mestre front
- [[../../_archive/2026-04-23-pre-v2-migration/content/regras-de-negocio/quebras-detectadas]] — Q-###
- [[2026-04-24-gap-analysis-consolidado]] — top 20 ações Sprint 10
- [[2026-04-24-gap-analysis-endpoints]] — 333 endpoints fantasma
- [[2026-04-24-gap-analysis-design-system]] — 500 componentes órfãos
- [[2026-04-24-gap-analysis-produto-processos]] — 221 ABAC órfãs + GAP-J-01..J-20
- [[2026-04-24-gap-analysis-dominio]] — 13 aggregates declarados sem arquivo
- [[../../03-subprojects/web/ui-catalog/admin/GAP-admin-master-sindico]] — gap admin formal

## Metodologia

Varredura determinística via Read + grep em:
1. `STATE.md` — 72 D-### + DT-### (filtrados por 🟡 aberto / 🔴 bloqueador / aguardando stakeholder)
2. `AUDIT.md` — 14 🔴 Critical + 15 🟠 High + 33 🟡 Medium + 7 bloqueadores LGPD M1
3. `_archive/…/quebras-detectadas.md` — 18 Q-### (5 🔴 resolvidas em 2026-04-24; resto 🟡/🟢)
4. 5 gap analyses de 2026-04-24 (consolidado + endpoints + DS + produto-processos + domínio)
5. `03-subprojects/feature-plan-2026-04-24.md` §9 (4 pendências explícitas)
6. Grep por `⚠️ PENDÊNCIA|TODO|a definir|a confirmar|stub inicial|plan_requirement: premium|sub_produto: onboarding|sub_produto: marketing|sub_produto: banco-talentos|persona: administradora`

Zero invenção — todo item rastreável a path:linha do vault.
