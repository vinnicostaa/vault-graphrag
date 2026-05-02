---
title: Cronograma Macro — Master Síndico v2
type: guide
tags: [schedule, cronograma, milestones, master-sindico, v2]
source: 90-ingested/from-vault-50-projects-master-sindico/10-schedule/cronograma-macro.md + ROADMAP v2
created: 2026-04-23
updated: 2026-04-23
---

# Cronograma Macro — Master Síndico v2

Operacionaliza [[../ROADMAP]] em calendário semanal. Sprints de 1-2 semanas; sprint-auditor domingo; deploy prod nos marcos M1/M2/M3.

**Data-alvo vigente**:
- 🟢 **M1 — MVP Contratual**: 2026-05-08 (sexta)
- 🟡 **M2 — Connect Me + Conteúdo**: 2026-06-20 (sábado)
- 🔵 **M3 — Assembleia + LMS + Polish**: 2026-07-20 (segunda)

---

## 2026 — Visão anual

```
Q1 (Jan-Mar): Discovery + Sprints 0-7 backend  ✅ concluído
Q2 (Abr-Jun): Sprints 8-13 — M1 (08/05) + M2 (20/06)
Q3 (Jul-Set): Sprints 14-23 — M3 (20/07) + escala + observability
Q4 (Out-Dez): Escala nacional + compliance + SOC 2 prep
```

---

## Abril 2026 — atual (Sprint 10 em curso)

| Semana | Sprint | Foco | Status |
|---|---|---|---|
| 01-07/Abr | Sprint 6 (início Q2) | Integrações cross-module | ✅ |
| 08-14/Abr | Sprint 7 | Assembly + WebSocket + observability | ✅ |
| 15-21/Abr | Sprint 8 | Security hardening | ✅ |
| 22-28/Abr | Sprint 9→10 | Sprint 9 autônoma (F1-F33 entregue 22/Abr); Sprint 10 em curso | ⏳ |
| **29/Abr-04/Mai** | **Sprint 10 fim** | **Polish + staging + deploy prep M1** | **⏳** |

---

## Maio 2026 — entrega M1

### Sprint 10 (continuação): 2026-04-28 → 2026-05-04

Foco: frontend web + mobile + AUDIT A-020..A-024 + LGPD base + runbooks. Ver [[../05-tasks/by-sprint/sprint-10]].

| Dia | Atividade |
|---|---|
| Seg 28/Abr | Sprint planning mid-sprint |
| Ter 29/Abr | Polish frontend web |
| Qua 30/Abr | Polish mobile + LGPD endpoints |
| **Qui 01/Mai** | **Holiday BR (Dia Trabalho); buffer** |
| Sex 02/Mai | Staging final + smoke |
| Sáb 03/Mai | Buffer |
| Dom 04/Mai | **Sprint-auditor** + freeze prep |

### Code freeze: 2026-05-05 → 2026-05-06

Só fix crítico. Sem merge de feature.

### Deploy M1

| Dia | Atividade |
|---|---|
| Qua 06/Mai | Deploy staging final |
| Qui 07/Mai | Smoke em staging; checklist go/no-go |
| **Sex 08/Mai** | **🟢 M1 ao vivo — promote staging → prod** |
| Sáb-Dom 09-10/Mai | Monitor 48h pós-deploy |

### Sprint 11: 2026-05-06 → 2026-05-18

Kick-off M2 (Connect Me UI + Reputação dashboard + vídeos upload/player + moderação base).

| Semana | Foco |
|---|---|
| 06-11/Mai | Connect Me 2 fluxos UI (Síndico→Empresa + Morador→Síndico) |
| 13-18/Mai | Reputação dashboard + vídeo player HLS + moderação backend |

Ver [[../05-tasks/by-sprint/sprint-11]].

### Sprint 12: 2026-05-13 → 2026-05-25

Vizinhança + moderação admin UI + Connect Me fluxos avançados + OpenSearch prod.

| Semana | Foco |
|---|---|
| 13-18/Mai | Vizinhança + OpenSearch |
| 20-25/Mai | Connect Me E↔E + S→P + Moderação UI |

Ver [[../05-tasks/by-sprint/sprint-12]].

### Sprint 13: 2026-05-20 → 2026-06-01

Chaos validation + load test + deploy M2 prep.

| Semana | Foco |
|---|---|
| 20-25/Mai | Chaos tests (Mux/Livekit/Stripe) |
| 27/Mai-01/Jun | Load test 1000 users + polish |

Ver [[../05-tasks/by-sprint/sprint-13]].

---

## Junho 2026 — entrega M2

### Sprint 14 pré-M2: 2026-06-03 → 2026-06-15

Finalização M2 telas + testes e2e.

| Semana | Foco |
|---|---|
| 03-08/Jun | Content + Mux prod + reputação V2 |
| 10-15/Jun | Deploy staging M2 + testes end-to-end |

### Code freeze: 2026-06-17 → 2026-06-19

### Deploy M2

| Dia | Atividade |
|---|---|
| Qua 17/Jun | Deploy staging final |
| Qui-Sex 18-19/Jun | Smoke + monitor |
| **Sáb 20/Jun** | **🟡 M2 ao vivo** |
| Dom-Seg 21-22/Jun | Monitor 48h |

### Sprint 14: 2026-06-24 → 2026-07-06

Kick-off M3 (Assembleia live UI + LMS thin + Banco Talentos).

Ver [[../05-tasks/by-sprint/sprint-14]].

---

## Julho 2026 — entrega M3

### Sprint 15: 2026-07-01 → 2026-07-13

Procuração + ata + simulador + enquetes preliminares + compliance anual UI.

Ver [[../05-tasks/by-sprint/sprint-15]].

### Sprint 16: 2026-07-08 → 2026-07-20

a11y WCAG 2.1 AA + 6 relatórios + score transparência + deploy M3.

| Semana | Foco |
|---|---|
| 08-13/Jul | a11y audit + fix |
| 15-19/Jul | Relatórios + telão 2 áreas + polish |

### Code freeze: 2026-07-18 → 2026-07-19

### Deploy M3

| Dia | Atividade |
|---|---|
| Sáb 19/Jul | Deploy staging final |
| **Dom 20/Jul** | **🔵 M3 ao vivo** |
| Seg-Ter 21-22/Jul | Monitor 48h |

### Sprint 17+: 2026-07-22 → ...

Post-M3 retro + escala planning.

---

## Q3 (Jul-Set) — escala nacional

Pós-M3:
- Observability madura (Grafana + alertas + runbooks completos)
- Load test 10k concurrent users (k6)
- Plano de sharding geográfico
- Migração Railway → AWS ECS (se escala > 500 condomínios) — fechar D-024
- LGPD audit externo
- Sentry prod + error budget tracking
- SLO 99.9% upgrade

---

## Q4 (Out-Dez) — compliance + preparação SOC 2

- Pen-test externo
- SOC 2 Type 1 prep
- Auditoria LGPD formal
- Disaster recovery test
- Documentação completa para compliance
- Moderação ML pilot (D-014)

---

## Cadência semanal fixa

| Dia | Atividade |
|---|---|
| Seg | Sprint planning — leitura STATE + AUDIT + ROADMAP; definir tasks do sprint |
| Ter-Qui | Execução |
| Sex | Polish, review, staging deploy |
| Sáb | Buffer (feature crítica ou folga) |
| Dom | Sprint-auditor roda; AUDIT atualizado; prep próximo sprint |

---

## Code freeze windows

Freeze = sem merge em `main` por período definido.

- **M1/M2/M3 pré-deploy**: 48h freeze antes do deploy prod
- **Fim de ano** (21-Dez → 2-Jan): freeze opcional se time descansa
- **Incident response ativo**: freeze automático até RESOLVED

---

## Contingência

Se sprint atrasa >2 dias do planejado:
1. Revisar escopo (remover "nice-to-have")
2. Avaliar extensão do sprint (1 dia extra, max)
3. Se maior: re-planejar marco (D-### no STATE)
4. Comms cliente obrigatório se slide afeta M1/M2/M3

Prioridade: **protege M1**. M2/M3 podem slidar 1 semana se M1 apertar.

---

## Links

- [[_moc]]
- [[milestones]]
- [[calendar]]
- [[../ROADMAP]]
- [[../05-tasks/by-sprint/_moc]]
- [[../SESSION_CHARTER]]
- Herança: [[../90-ingested/from-vault-50-projects-master-sindico/10-schedule/cronograma-macro]]
