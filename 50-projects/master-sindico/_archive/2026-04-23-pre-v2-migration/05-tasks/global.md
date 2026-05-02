---
title: Global Tasks — Master Síndico
type: project
tags: [master-sindico, tasks, global, cross-cutting]
project: master-sindico
created: 2026-04-22
---

# Global Tasks — Master Síndico

Tasks **transversais** que atravessam sub-projetos ou módulos. Canônico.

---

## Operacionais recorrentes

| ID | Título | Frequência | Dono | Notas |
|---|---|---|---|---|
| G-001 | Dep-audit completo | Semanal (ou por release) | agente | [[../../../30-agents/playbooks/dependency-audit]] |
| G-002 | CVE scan | Contínuo (CI) + semanal manual | agente | [[../../../30-agents/playbooks/cve-scan]] |
| G-003 | Sprint-auditor fim-de-sprint | Por sprint | agente | skill em `.claude/skills/` |
| G-004 | Review do STEERING e STATE | Por sprint | dev humano | detectar drift |
| G-005 | Backup do DB + snapshot | Diário | Railway managed | monitor |
| G-006 | Rotação de secrets | Trimestral | dev | Stripe keys, Zitadel keys, Twilio tokens |
| G-007 | Pen-test | Semestral interno; anual externo | dev (com agente auxiliar) | post-M3 |
| G-008 | LGPD compliance review | Trimestral | DPO + dev | checklist |
| G-009 | Runbooks review | Trimestral | dev | [[../12-docs/runbooks/_moc]] |

---

## Preparação pro Marco

### Pré-M1 (até 2026-05-08)

- [x] ~~Sprint 0-9 backend entregue~~
- [ ] Frontend M1: onboarding síndico + dashboard condomínio + unidades (Sprint 10)
- [ ] Mobile M1: auth + home morador (Sprint 10)
- [ ] Deploy Railway prod configurado
- [ ] Smoke tests E2E Playwright mínimos (Sprint 10)
- [ ] LGPD base: consent + delete endpoint + DPO (Sprint 10)
- [ ] Zitadel managed em Railway + DNS prod (9.19)
- [ ] Stripe live mode + webhook configurado
- [ ] Sentry configurado em 3 sub-projetos
- [ ] Observability: OTel + dashboards mínimos
- [ ] Changelog M1 preparado
- [ ] AUDIT zerada 🔴 antes do deploy

### Pré-M2 (até 2026-06-20)

- [ ] Commercial + Content telas (Sprint 11)
- [ ] Saga Mux/Livekit validados em staging com chaos test
- [ ] Moderação de conteúdo UI (admin)
- [ ] OpenSearch prod
- [ ] Cache Redis ABAC com invalidação monitorada
- [ ] Load test 1000 concurrent users

### Pré-M3 (até 2026-07-20)

- [ ] Assembly telas completas
- [ ] Livekit produção
- [ ] PDF signing (atas)
- [ ] LMS telas
- [ ] Polish + a11y WCAG AA
- [ ] SLOs definidos + dashboards
- [ ] Runbooks completos

---

## Cross-subproject (requerem backend + web + mobile)

| ID | Título | Módulos | Sub-projetos |
|---|---|---|---|
| C-001 | OpenAPI types generator | backend, web, mobile | [[../02-architecture/openapi/_moc]] |
| C-002 | Design system v1 | web, mobile | Tokens compartilhados |
| C-003 | i18n infra | backend, web, mobile | Pt-BR first; EN prep |
| C-004 | Push notification orchestration | backend, mobile | Tópicos por tenant + role |
| C-005 | Deep-linking / Universal links | web, mobile | `mastersindico://` + `.mastersindico.com.br` |
| C-006 | Logout idempotente cross-device | backend, web, mobile | Sincronizar logout global |

---

## Dívidas técnicas (DT-###)

Ver [[../STATE]] §Dívidas Técnicas Registradas. Resumo das abertas:

- DT-001..DT-009 (ver STATE)
- DT-010: `IAuditLogger` cross-módulo (promoção de A-007)

---

## AUDIT aberto (A-###) que vira task

Ver [[../11-audit/AUDIT]] seção 🔴 Aberto. Resumo atual:

- A-023: device change audit log comparativo (Sprint 10)
- A-024: `rawBodyReader` pra interface pública Context (Sprint 10)

---

## Como criar task nova

1. Identificar módulo ou módulo cross
2. Se cross: aqui
3. Se single-module: [[by-module/_moc]]
4. Se sprint-específica: [[by-sprint/_moc]] ou backlog
5. Usar formato T-<ID> com DoR/DoD (ver [[../DoR-DoD]])

## Links

- [[_moc]]
- [[by-module/_moc]]
- [[by-sprint/_moc]]
- [[backlog]]
- [[../STATE]]
- [[../11-audit/AUDIT]]
- [[../ROADMAP]]
- [[../DoR-DoD]]
