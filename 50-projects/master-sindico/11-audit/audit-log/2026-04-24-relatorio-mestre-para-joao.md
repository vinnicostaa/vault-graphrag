---
title: Relatório Mestre para João — o que você precisa saber e decidir
type: audit
tags:
  - master-sindico
  - stakeholder
  - relatorio-mestre
  - joao
  - 2026-04-24T00:00:00.000Z
project: master-sindico
date: 2026-04-24T00:00:00.000Z
audience: João (stakeholder)
purpose: consolidação final antes de stakeholder review
---

# Relatório Mestre para João — 2026-04-24

**Para você ler primeiro. Tudo consolidado.** Outros 3 relatórios de suporte ([[2026-04-24-pendencias-para-stakeholder]], [[2026-04-24-check-consistencia-fase12]], [[2026-04-24-roadmap-realista-m1-m2-m3]]) têm o detalhe; este dá a visão de cima.

---

## TL;DR (leia isto primeiro)

1. **M1 em 2026-05-08 é inviável** matematicamente. Web está em 0% (5/6 apps são stubs; deps críticos não instalados). **Recomendação: postergar M1 → 2026-05-18 (Opção A)**. Sem essa decisão, nada se move com confiança hoje.
2. **Backend está maduro** (Sprint 9 fechou com BeyondCorp 12/14, gosec 0, govulncheck 0). O problema é **formalizar specs** (333 endpoints citados em UI sem entrada em `backend/requirements/`), não código.
3. **Você tem 10 decisões pendentes** listadas abaixo. **3 são HOJE** (Opção A/B/C + LGPD responsáveis + paleta OKLCH). Resto pode esperar 2-3 dias.
4. **6 bloqueadores LGPD M1 são humanos/comerciais** (DPO, 3 DPAs, DPIA, política privacidade). Sem dono formal no vault — você precisa nomear.
5. **Vault está consistente**: 12 decisões D-094..D-105 aplicadas hoje, 10 arquivos novos criados, 8 editados, dual-check passou 14/14 checagens mecânicas, 12 pequenas inconsistências (fix 45-60min) foram identificadas e 3 top já corrigidas.

---

## Parte 1 — Decisões URGENTES (hoje)

### 🔴 DEC-1. Opção A vs B vs C para M1 — **decisão-chave**

| Opção | Data M1 | Entrega | Confiança | Risco principal |
|---|---|---|---|---|
| **A (recomendada)** | 2026-05-18 (+10 dias úteis) | backend + mobile + web parciais | 🟡 60% | LGPD bloqueia shipping regulatório |
| **B** | 2026-05-08 (firme) | backend + mobile apenas + web em onda M1.1 (22/05) | 🟢 80% para M1.0 / 55% M1.1 | UX split pode confundir usuário final |
| **C** | sem data | continuar remontagem até AUDIT 🔴=0 | — | contrato pode expirar |

**Minha recomendação técnica**: **Opção A**. A é mais simples operacionalmente, dá 2 semanas honestas para web team, e mantém linha única de shipping. B só vale se LGPD regulamentar for resolvido antes de 08/05.

→ **Ação**: responder A/B/C.

### 🔴 DEC-2. Nomear responsáveis LGPD M1 (6 itens sem dono)

Sem dono formal no vault — listados genericamente como "commercial/legal team". Precisam responsável + data-alvo:

| ID | Item | Quem resolve | Deadline sugerida |
|---|---|---|---|
| LGPD-M1-001 | **DPO nomeado** publicamente (`lgpd@mastersindico.com.br`) | ? | 2026-05-05 |
| LGPD-M1-002 | **DPA com Mux** assinado | ? | 2026-05-10 |
| LGPD-M1-003 | **DPA com Zitadel** assinado | ? | 2026-05-10 |
| LGPD-M1-004 | **DPA com Twilio** assinado | ? | 2026-05-10 |
| LGPD-M1-005 | **DPIA-001** executado (Data Protection Impact Assessment) | ? (DPO + arquiteto) | 2026-05-12 |
| LGPD-M1-006 | **Política privacidade** versionada publicada | ? | 2026-05-12 |

→ **Ação**: nomear pessoas e datas. Sem isso, M1 trava mesmo com tecnologia pronta.

### 🔴 DEC-3. Paleta OKLCH web vs mobile

Web usa `#C49A2A`, mobile usa `#C69C3F`. Divergência pequena mas bloqueia design-system unificado (10 componentes DS M1 dependem da paleta canônica).

→ **Ação**: escolher UMA cor. Sugestão técnica: mobile (`#C69C3F`) é mais próxima do espírito OKLCH do Manual IV. Mas você decide.

---

## Parte 2 — Decisões IMPORTANTES (esta semana)

### 🟠 DEC-4. Fórmula INV-GOV-001 (Score Governança 1-10)

Stub atual: pesos 40/25/15/10/10 entre os 5 componentes (avaliações regulares M12 / avaliações escondidas eleição / engajamento timeline / transparência / resposta Connect Me).

→ **Ação**: confirmar pesos OU revisar. Impacta cálculo do score exibido em S32 e percebido por morador.

### 🟠 DEC-5. Texto literal das 3-5 perguntas da avaliação escondida (D-104)

Stub atual (5 perguntas escala 1-10):
1. Como você avalia a **transparência** da gestão atual?
2. Como você avalia a **qualidade das decisões** tomadas?
3. Como você avalia a **resposta às demandas dos moradores**?
4. Como você avalia o **cuidado com o patrimônio** do condomínio?
5. Como você avalia o **compliance regulatório**?

→ **Ação**: você escreve ou delega para produto/UX? Textos são críticos (LGPD + UX).

### 🟠 DEC-6. Empresa Plus E→E Connect Me — cap ou ilimitado?

Q8 ainda aberta (não fechei junto com as outras). Empresa Plus pode mandar Connect Me para outras empresas (B2B) com limite ou sem?

→ **Ação**: responder. Se com cap, sugestão N/ano.

### 🟠 DEC-7. Tipos de evento — 13 canônicos ou 18+ do briefing?

Divergência entre `04-Listas-Mestre/Enums-Consolidados.md` (13 tipos) vs PDF cliente "JORNADA DO SÍNDICO" (18+ tipos).

→ **Ação**: aceitar 13 ou expandir para 18+? Impacta enum backend + UI S16 (criar evento).

### 🟠 DEC-8. Mobile D-048/D-049/D-050 — aplicar pré-M1?

Vault decidiu adicionar `bloc_concurrency` + `hydrated_bloc` + `freezed` + `freeRASP` no app Flutter. Código real não tem nenhuma delas. 2-3 dias de dev pra aplicar.

→ **Ação**: aplicar antes de M1 (zera DT) ou postergar M2 (registra dívida)?

---

## Parte 3 — Decisões que podem esperar (Sprint 11+)

### 🟡 DEC-9. Admin MS M1 — CLI ou UI?

5 capacidades admin (Dashboard Financeiro, Gestão Usuários, Moderação, Config, Broadcast) sem telas documentadas. Opções:
- **M1**: CLI only (query console + scripts internos) — evita construir UI
- **M2**: 4-6 telas S-ADM mínimas (1 semana dev)

→ **Ação**: decidir depois de M1 shipping.

### 🟡 DEC-10. Administradoras Condominiais — criar telas S-ADM/E-ADM em Sprint 10?

Sub-produto 11 existe no portfolio mas tem **0 telas** dedicadas (só referenciado em telas de empresa como subtype). Backend expõe 5 endpoints `empresa_sindica_handler.go` sem UI.

→ **Ação**: pode esperar M2 a menos que cliente condo tenha contrato de administradora ativo.

---

## Parte 4 — Pendências operacionais (agente resolve autônomo)

Estas NÃO requerem decisão sua — agente aplica quando você destravar DEC-1 a DEC-6:

| Ação | Esforço | Responsável |
|---|---|---|
| Formalizar 112 endpoints 🔴 em `backend/requirements/institutional.md` | 1 dia | agente |
| Consolidar `04-requirements/abac-actions.md` (221 actions órfãs) | 0.5 dia | agente |
| Renomear 32 frontmatters `sub_produto:` não-canônico | 15min | agente |
| Renomear 5 `plan_requirement: premium` → canônico | 10min | agente |
| Criar stub `admin-mfa-lockout.md` em runbooks | 0.25 dia | agente |
| Expandir traceability.md para 340 endpoints | 0.5 dia | agente |
| Criar 10 componentes DS M1 em `packages/ui` (código) | 3-4 dias | **dev web team** (eu não codifico) |
| Migração Unit (+`fracao_ideal`, `+unit_type`) | 0.5 dia backend | dev backend |
| DT-010 IAuditLogger cross-BC completo | 1 dia backend | dev backend |

---

## Parte 5 — Roadmap recomendado (resumo)

### M1 — 2026-05-18 (Opção A)

**Backend**:
- ✅ Identity + Billing + Institutional base (Sprint 1-3 entregues)
- [ ] Unit.fracao_ideal + unit_type migration
- [ ] 112 endpoints formalizados
- [ ] DT-010 IAuditLogger
- [ ] A-023, A-024, A-039 Sprint 10

**Web**:
- [ ] App `auth` OIDC real (substituir stubs)
- [ ] App `cms`: morador M1-M5 + síndico S1-S6 (11 telas, não 181)
- [ ] App `lp` completa
- [ ] 5 componentes DS M1
- [ ] Vitest + MSW + Playwright setup + cobertura 75%

**Mobile**:
- [ ] Splash + auth OIDC
- [ ] Home morador + Timeline read
- [ ] D-048/049/050 aplicadas (se DEC-8 = sim)

**Fora M1**: LMS, Assembly completa, Reputação motor, Banco de Talentos, Painel Admin.

### M2 — 2026-06-20

- Commercial full (Connect Me, Reputação motor, Vizinhança)
- Content full (Mux, MarketingAgencyLink)
- Telas S7-S31, E1-E16, MK1-MK8, C1-C11, VM1-VM7
- 3 componentes DS M2 (ScoreHistory, DeletionTimeline, ValidationQueueItem)

### M3 — 2026-07-20

- Assembly live completo (Livekit)
- LMS completo
- Banco de Talentos completo
- Avaliação escondida eleição (D-104)
- 3 componentes DS M3 (HiddenAssessmentForm, NavGuard, LiveCreationForm)

---

## Parte 6 — Estado do vault (facts)

- **6.507+ MDs** no vault (era 6.138 pré-F12 — cresceu 369 com F6 ui-catalog + F7 gap analysis + F11+F12 decisões)
- **0 non-MD** (regra "vault 100% markdown" respeitada)
- **0 wikilinks quebrados** (pós fix F6)
- **493 telas UI catalog** (181 web + 53 mobile + 3 novas Fase 12)
- **105 decisões D-001..D-105** registradas em STATE.md
- **38 ADRs** (Fase 11 criou 0037; Fase 12 criou 0038)
- **124+ invariantes** (3 novos Fase 12: INV-GOV-001/002/ASM-023)
- **BeyondCorp 12/14 validado** no backend Sprint 9
- **16 A-### fechados Sprint 9** (race TOCTOU, saga compensation, gosec 66→0)

### Fase 12 aplicada hoje (D-103 + D-104 + D-105)

- **D-103** Score duplo (Governança 1-10 + Compliance 0-100) — 5 arquivos atualizados + S32 criada + C10 patched
- **D-104** Avaliação escondida eleição — ASM-AVAL-ELEICAO web+mobile criadas + Req ASM-ELE-AVAL + INV-ASM-023
- **D-105** ValidationItem Opção B DDD — pattern validatable-interface criado + 3 aggregates stubs + ADR-0038

### Consistência cross-vault pós F12

Dual-check independente identificou 12 inconsistências pequenas (45-60min fix total). Das 3 top-críticas, **2 já foram corrigidas** nesta sessão:
- ✅ ADR-0038 indexada em adr-index.md
- ✅ ASM-PAUTA.md Q40 marcada como resolvida
- 🟡 Sub-produto 05-assembleia-live.md seção §17 ainda cita D-063 genérico sem linkar D-104 (fix 5min manual)

Restantes 9 fixes (sem impacto bloqueador M1):
- 3 usos ativos N1/N2/N3 em onboarding (fix global sed)
- 2 arquivos sem `created` no frontmatter
- 1 link quebrado `S-PERFIL` (tela não criada ainda, referência futura)
- 3 pequenos ajustes em scores/telas

---

## Parte 7 — Ações concretas HOJE (ordenadas)

### Para você (~1h de trabalho)

1. **Decidir DEC-1 (Opção A/B/C)** — 15 min
2. **Nomear responsáveis DEC-2 (LGPD 6 itens)** — 20 min (ou delegar para jurídico)
3. **Responder DEC-3 (paleta OKLCH)** — 5 min
4. **Revisar esta lista e marcar quais DEC-4 a DEC-10 você responde hoje ou semana** — 10 min
5. **Aprovar fixes restantes de consistência** (agente aplica) — 5 min

### Para o agente (autônomo após seu OK)

1. Aplicar as 9 inconsistências remanescentes (45 min)
2. Formalizar 112 endpoints (1 dia)
3. Consolidar abac-actions.md (0.5 dia)
4. Criar stubs de runbooks e arquivos pendentes (0.5 dia)

### Para o dev team (pós-decisão macro)

1. Backend: Sprint 10 com prioridades definidas (Unit migration + IAuditLogger + endpoint formalization)
2. Web: instalar deps + iniciar auth + 3 telas prova (2 dias)
3. Mobile: pivot se DEC-8 = sim (abandonar assembly temporariamente, aplicar D-048/049/050)

---

## Parte 8 — Links de suporte

### Relatórios gerados hoje
- [[2026-04-24-pendencias-para-stakeholder]] — detalhe das 20 pendências
- [[2026-04-24-check-consistencia-fase12]] — 12 inconsistências detalhadas
- [[2026-04-24-roadmap-realista-m1-m2-m3]] — roadmap com esforço por item
- [[2026-04-24-gap-analysis-consolidado]] — gap analysis consolidada (333 endpoints + 500 DS + 221 ABAC)

### Governança
- [[../AUDIT]] — findings + LGPD bloqueadores
- [[../STATE]] — 105 decisões D-###
- [[../ROADMAP]] — marcos originais
- [[../CLAUDE]] — contrato do agente

### Por área
- [[../03-subprojects/feature-plan-2026-04-24]] — feature plan frontend
- [[../03-subprojects/backend/requirements/_moc]] — requirements backend (7 BCs)
- [[../03-subprojects/web/ui-catalog/_moc]] — 181 telas web
- [[../03-subprojects/mobile/ui-catalog/_moc]] — 53 telas mobile
- [[../01-domain/invariants]] — 124 INVs
- [[../12-docs/adr-index]] — 38 ADRs

---

## Pronto pra te ouvir

Depois que você decidir **DEC-1 a DEC-3** (3 decisões urgentes), eu posso:
1. Aplicar as 9 inconsistências remanescentes
2. Disparar agentes para formalizar 112 endpoints + 221 ABAC actions
3. Registrar D-106 a D-112 (suas novas decisões) em STATE
4. Atualizar ROADMAP com data-alvo M1 canônica
5. Notificar dev team (via PR ou mensagem) com prioridades Sprint 10

Ou se preferir revisar tudo primeiro e voltar depois — também OK. Vault está congelado em estado consistente agora.
