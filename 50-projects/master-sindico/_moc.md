---
title: MOC — Master Síndico v2 (Remontagem)
type: moc
tags: [moc, project, master-sindico, v2, rebuild]
created: 2026-04-23
updated: 2026-04-23
---

# Master Síndico v2 — Remontagem

Workspace de remontagem do projeto Master Síndico (SaaS governança condominial BR). Organização **Johnny.Decimal** com paridade ao scaffold canônico do vault. Migra para `vault/50-projects/master-sindico/` na Fase 6 do plano.

> **Entry point**: [[CLAUDE]]. Leitura obrigatória: CLAUDE → [[STATE]] → [[AUDIT]] → [[SESSION_CHARTER]].

---

## 🆕 Entrando agora? Comece aqui

- **[[notation-conventions]]** — **glossário das siglas usadas no vault** (D-###, A-###, DT-###, ADR, INV, BR, GR, FR, M1/M2/M3, backlog, severidades 🔴🟠🟡🟢). Se você é não-técnico ou está topando uma sigla pela primeira vez, leia esta página antes de tudo.

## Governança viva (raiz)

- [[CLAUDE]] — contrato do agente nesta pasta
- [[STATE]] — decisões (D-###) + dívidas (DT-###)
- [[AUDIT]] — findings (A-###); espelho em [[11-audit/AUDIT]]
- [[SESSION_CHARTER]] — ordens da sessão atual
- [[STEERING]] — não-negociáveis do projeto
- [[ROADMAP]] — marcos M1/M2/M3
- [[DoR-DoD]] — Definition of Ready / Done
- [[notation-conventions]] — convenções de notação (siglas + severidades)
- [[_ingestion-log]] — trilha de auditoria do material absorvido do legado

## Navegação zero → prod

- [[00-product/_moc]] — visão, personas, business model, glossário, portfolio-de-produtos
- [[01-domain/_moc]] — bounded contexts, ubiquitous language, aggregates, invariants, events, state machines
- [[02-architecture/_moc]] — C4 (contexts/containers/components), clean-arch, patterns, multi-tenant, API, event-driven, observability, ADRs, research-inspirations
- [[03-subprojects/_moc]] — backend, web, mobile (cada um com arquitetura + patterns + requirements + security + tasks + ui-catalog se web/mobile)
- [[04-requirements/_moc]] — global, functional por BC, non-functional, compliance-LGPD, matrix-funcional, registration-data
- [[05-tasks/_moc]] — backlog, by-sprint, by-module
- [[06-execution/_moc]] — STEPS zero→prod + dev/QA/deploy/rollback/incident + playbooks
- [[07-quality/_moc]] — CLEAN_CODE, CLEAN_ARCH, CODE_CALISTHENICS, PATTERNS
- [[08-security/_moc]] — BEYOND_CORP, threat-model, OWASP, CVE process, LGPD, pentest-checklist, beyond-corp-references
- [[09-testing/_moc]] — test strategy, unit/integration/e2e/load/security, coverage thresholds
- [[10-schedule/_moc]] — cronograma macro, milestones, calendar
- [[11-audit/_moc]] — AUDIT, audit-log, dual-check-log, postmortems
- [[12-docs/_moc]] — README, how-to, runbooks, ADR-index, changelog

## Research + ingestão (transitórios; desaparecem na migração)

- [[13-research/_moc]] — Beyond-Corp + big techs (Google/Netflix/YouTube/Instagram/TikTok/Meet/LinkedIn) + condominial BR
- [[90-ingested/_moc]] — material-fonte absorvido do legado (archive + source-material + vault/50-projects/master-sindico)

## Como navegar por tarefa

| Quero | Vá para |
|---|---|
| Entender o produto | [[00-product/vision]] → [[00-product/personas]] → [[00-product/portfolio-de-produtos]] |
| Ver arquitetura macro | [[02-architecture/c4-context]] → [[02-architecture/c4-containers]] |
| Ver decisões arquiteturais | [[02-architecture/adr/_moc]] + [[STATE]] D-### |
| Revisar segurança | [[08-security/BEYOND_CORP]] + [[08-security/threat-model]] + [[08-security/pentest-checklist]] |
| Ver requisitos | [[04-requirements/_moc]] — por BC em `functional/` |
| Ver research aplicado | [[13-research/_destilado]] + [[02-architecture/research-inspirations]] |
| Ver o que herdamos | [[_ingestion-log]] + [[90-ingested/INDEX]] |
| Estratégia de testes | [[09-testing/test-strategy]] |

## Status atual

Data: 2026-04-23. Fase 0 concluída (estrutura criada + governança raiz escrita). Próximo: Fase 1b (ingestão minuciosa continuada) + Fase 2 (research web amplo) em paralelo. Ver [[STATE]] D-001 e [[SESSION_CHARTER]].

## Vizinhos no grafo

- Plano completo: `/home/vinni/.claude/plans/squishy-drifting-avalanche.md`
- Vault raiz: `../vault/CLAUDE.md`
- Legado vivo: `../vault/50-projects/master-sindico/_moc.md`
- Knowledge transversal: `../vault/10-knowledge/_moc.md`
- Playbooks: `../vault/30-agents/playbooks/_moc.md`
- Templates: `../vault/40-templates/project-scaffold/`
