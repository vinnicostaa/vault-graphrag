---
title: MOC — Master Síndico
type: moc
tags: [moc, project, master-sindico]
created: 2026-04-22
updated: 2026-04-22
project: master-sindico
---

# Master Síndico

Plataforma SaaS de **governança condominial** (Brasil). Cliente-referência do `automation-agents` — primeiro projeto atendido pelo pipeline multi-agente.

> **Entry point do agente**: [[CLAUDE]]. **Leitura obrigatória** por sessão: CLAUDE → [[STATE]] → [[11-audit/AUDIT]] → [[SESSION_CHARTER]].

---

## Estrutura canônica (nova, 2026-04-22)

Organização **Johnny.Decimal** com overlay canônico sobre o material legado (specs Kiro, contextos, content, client-vault). Cada pasta tem `_moc.md`.

### Governança viva (raiz)
- [[CLAUDE]] — contrato do agente neste projeto
- [[STATE]] — decisões (D-###) + dívidas (DT-###) + bloqueadores (B-###)
- [[AUDIT]] — findings (A-###) — espelho canônico em [[11-audit/AUDIT]]
- [[SESSION_CHARTER]] — ordens da sessão atual
- [[STEERING]] — não-negociáveis do projeto
- [[ROADMAP]] — marcos M1/M2/M3
- [[DoR-DoD]] — Definition of Ready / Done
- [[overview]] — visão resumida (legado, mantido por compat)
- [[legacy-reference]] — como usar TypeScript legacy sem copiar
- [[plugins]] — plugins Claude Code habilitados

### Navegação zero → prod
- [[00-product/_moc]] — visão, personas, business model, glossário
- [[01-domain/_moc]] — bounded contexts, ubiquitous language, business+domain+implementation rules, invariants
- [[02-architecture/_moc]] — system overview, clean-arch mapping, patterns, openapi, ADRs
- [[03-subprojects/_moc]] — backend, web, mobile (cada um com seu README + architecture + patterns + requirements + tasks + security)
- [[04-requirements/_moc]] — global, functional por módulo, non-functional, compliance
- [[05-tasks/_moc]] — global, by-sprint, by-module, backlog
- [[06-execution/_moc]] — STEPS (zero→prod), dev/QA/deploy/rollback/incident
- [[07-quality/_moc]] — CLEAN_ARCH, CLEAN_CODE, CODE_CALISTHENICS, PATTERNS
- [[08-security/_moc]] — BEYOND_CORP, threat-model, CVE process, OWASP, LGPD
- [[09-testing/_moc]] — test strategy, unit/integration/e2e/load/security
- [[10-schedule/_moc]] — cronograma macro, milestones, calendar
- [[11-audit/_moc]] — AUDIT, audit-log, dual-check-log, postmortems
- [[12-docs/_moc]] — README, how-to, runbooks, ADR-index, changelog

### Material de referência (fonte)
- [[specs/README]] + `specs/{requirements,design,tasks,reference}/` — specs Kiro canônicas (migradas pro overlay canônico)
- `contextos/` — análises técnicas + manuais legados
- `legacy-docs/` — docs operacionais do legado TS (cópias em `_archive/legacy-docs-copies/`)
- `client-vault/` — Vault Obsidian externo do cliente (visão original)
- `MasterSindico_extracted/` — extração do zip original
- `_archive/` — histórico sanitizado (ver [[_archive/MOVE_LOG_2026-04-22]] e [[_archive/content-moved-2026-04-22]])
- `.claude/` — skills e settings Claude Code locais

> Pasta `content/` removida em 2026-04-22 — itens valiosos promovidos ao overlay canônico; históricos em `_archive/content-monolitico-2026-03/` e `_archive/content-discovery-2026-03/`. Ver [[_archive/content-moved-2026-04-22]].
> Pasta `plans/` movida para `../../90-archive/2026-04-22-master-sindico-dups/plans/` (idêntica por hash a `legacy-docs/`).

---

## Como navegar por tarefa

| Quero | Vá pra |
|---|---|
| Entender o produto | [[00-product/vision]] → [[00-product/personas]] |
| Implementar feature nova | [[06-execution/STEPS]] → [[07-quality/PATTERNS]] → [[CLAUDE]] |
| Revisar segurança | [[08-security/BEYOND_CORP]] + [[08-security/threat-model]] |
| Escrever teste | [[09-testing/test-strategy]] |
| Criar PR | [[DoR-DoD]] → [[07-quality/CLEAN_CODE#checklist-do-review]] |
| Ver o que estou bloqueando | [[11-audit/AUDIT]] + [[STATE]] |
| Debugger bug novo | [[07-quality/CLEAN_CODE]] + [[11-audit/AUDIT]] (checar A-### relacionado) |

---

## Status atual (2026-04-22)

- **Sprint 9 ✅ entregue autônoma** (F1-F33; SESSION_CHARTER §4)
- **Sprint 10 em curso** — finalização M1 (frontend/mobile + polish + deploy prep)
- **M1 prazo**: 2026-05-08
- **AUDIT**: 🔴 A-023, A-024 abertos (Sprint 10); 🟡 vazio
- **STATE**: D-030 em aberto (configs `.claude/settings.json` pra web/app)
- **Gates (snapshot)**: build ✅ · vet ✅ · test -race ✅ · integration ✅ · gosec -sev high 0 ✅ · govulncheck 0 ✅

---

## Vizinhos no grafo

- [[../_moc]] — pasta 50-projects
- [[../../CLAUDE]] — contrato raiz do vault
- [[../../30-agents/playbooks/_moc]] — playbooks obrigatórios
- [[../../40-templates/_moc]] — templates que este projeto instancia
- [[../../20-stacks/go/_moc]] — stack Go padrões
- [[../../10-knowledge/_moc]] — conhecimento técnico atemporal
