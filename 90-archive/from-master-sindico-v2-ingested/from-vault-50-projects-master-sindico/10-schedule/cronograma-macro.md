---
title: Cronograma Macro — Master Síndico
type: project
tags: [master-sindico, schedule, cronograma, milestones]
project: master-sindico
created: 2026-04-22
---

# Cronograma Macro

Operacionalização do [[../ROADMAP]] em calendar semanal. Sprints de 1 semana (seg-dom). Sprint-auditor domingo.

---

## 2026 — Visão anual

```
Q1 (Jan-Mar): Discovery + Sprints 0-7 backend
Q2 (Abr-Jun): Sprints 8-13 — M1 (08/05) + M2 (20/06)
Q3 (Jul-Set): Sprints 14-23 — M3 (20/07) + escala + observability
Q4 (Out-Dez): Escala nacional + compliance + SOC 2 prep
```

---

## Abril 2026 (atual)

| Semana | Sprint | Foco | Status |
|---|---|---|---|
| 01-07/Abr | Sprint 6 (início Q2) | Integrações cross-module | ✅ |
| 08-14/Abr | Sprint 7 | Assembly + WebSocket + observability | ✅ |
| 15-21/Abr | Sprint 8 | Security hardening | ✅ |
| **22-28/Abr** | **Sprint 9→10** | Sprint 9 autônoma (F1-F33 entregue em 22/Abr); Sprint 10 em curso — finalização M1 | **⏳ em curso** |

---

## Maio 2026 — entrega M1

| Semana | Sprint | Foco | Marco |
|---|---|---|---|
| 29/Abr-04/Mai | Sprint 10 fim | Polish + deploy staging | - |
| **08/Mai** | — | **🟢 M1 — MVP contratual em prod** | **M1** |
| 06-11/Mai | Sprint 11 | Commercial + Content telas | - |
| 13-18/Mai | Sprint 12 | Commercial + Content UI completa | - |
| 20-25/Mai | Sprint 13 | Saga validation + chaos | - |
| 27/Mai-01/Jun | Sprint 14 | Polish M2 | - |

---

## Junho 2026 — entrega M2

| Semana | Sprint | Foco | Marco |
|---|---|---|---|
| 03-08/Jun | Sprint 15 | Content + Mux prod | - |
| 10-15/Jun | Sprint 16 | Deploy staging M2 + testes | - |
| 17-19/Jun | Sprint 17 | Final gates | - |
| **20/Jun** | — | **🟡 M2 — Ecossistema comercial em prod** | **M2** |
| 23-29/Jun | Sprint 18 | Assembly setup | - |

---

## Julho 2026 — entrega M3

| Semana | Sprint | Foco | Marco |
|---|---|---|---|
| 01-06/Jul | Sprint 19 | Livekit + assembly telas | - |
| 08-13/Jul | Sprint 20 | LMS telas + PDF signing | - |
| 15-19/Jul | Sprint 21 | Polish + a11y + SLOs | - |
| **20/Jul** | — | **🔵 M3 — Governança plena em prod** | **M3** |
| 22-27/Jul | Sprint 22 | Post-M3 retro + escala planning | - |

---

## Q3 (Jul-Set) — escala nacional

Focos:
- Observability madura (Grafana + alertas + runbooks completos)
- Load test 10k concurrent users (k6)
- Plano de sharding geográfico
- Migração Railway → AWS ECS (se escala > 500 condomínios)
- LGPD audit externo
- Sentry prod + error budget tracking

---

## Q4 (Out-Dez) — compliance + preparação SOC 2

- Pen-test externo
- SOC 2 Type 1 prep
- Auditoria LGPD formal
- Disaster recovery test
- Documentação completa para compliance

---

## Cadência semanal

| Dia | Atividade |
|---|---|
| Seg | Sprint planning — leitura STATE + AUDIT + ROADMAP; definir tasks do sprint |
| Ter-Qui | Execução |
| Sex | Polish, review, staging deploy |
| Sáb | Buffer (feature crítica ou folga) |
| Dom | Sprint-auditor roda; AUDIT atualizado; prep próximo sprint |

---

## Code freeze

Freeze = sem merge em `main` por período definido.

- **M1/M2/M3 pré-deploy**: 48h freeze antes do deploy prod
- **Fim de ano** (21-Dez → 2-Jan): freeze opcional se time descansa
- **Incident response ativo**: freeze automático

---

## Contingência

Se sprint atrasa >2 dias do planejado:
- Revisar escopo (remover "nice-to-have")
- Avaliar extensão do sprint (1 dia extra, max)
- Se maior: re-planejar marco (D-### no STATE)

## Links

- [[_moc]]
- [[milestones]]
- [[sprint-calendar]]
- [[../ROADMAP]]
- [[../05-tasks/by-sprint/_moc]]
- [[../SESSION_CHARTER]]
- [[../client-vault/00 - Visão Geral/Roadmap 2026]]
