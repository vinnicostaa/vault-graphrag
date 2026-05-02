---
title: MOC — Master Síndico
type: moc
tags: [moc, project, master-sindico]
project: master-sindico
created: 2026-04-22
updated: 2026-04-23
---

# Master Síndico

Plataforma SaaS de **governança condominial** (Brasil). Cliente-referência do `automation-agents` — primeiro projeto atendido pelo pipeline multi-agente.

> **Entry point do agente**: [[CLAUDE]]. **Leitura obrigatória** por sessão: CLAUDE → [[STATE]] → [[11-audit/AUDIT]] → [[SESSION_CHARTER]].

---

## Estrutura canônica (nova, 2026-04-22; limpeza 2026-04-23)

Organização **Johnny.Decimal** com overlay canônico sobre o material legado. Cada pasta tem `_moc.md`.

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
- [[06-execution/_moc]] — STEPS (zero→prod), dev/QA/deploy/rollback/incident · [[06-execution/sdd-gsd-aplicado|SDD+GSD aplicado]]
- [[07-quality/_moc]] — CLEAN_ARCH, CLEAN_CODE, CODE_CALISTHENICS, PATTERNS
- [[08-security/_moc]] — BEYOND_CORP, threat-model, CVE process, OWASP, LGPD
- [[09-testing/_moc]] — test strategy, unit/integration/e2e/load/security
- [[10-schedule/_moc]] — cronograma macro, milestones, calendar
- [[11-audit/_moc]] — AUDIT, audit-log, dual-check-log, postmortems
- [[12-docs/_moc]] — README, how-to, runbooks, ADR-index, changelog

### Material de referência (fonte)
- [[specs/README]] + `specs/{requirements,design,tasks,reference}/` — specs Kiro canônicas (migradas pro overlay canônico)
- `content/` — docs-fonte do produto (requirements.md 206KB, design.md 84KB, tasks.md 68KB, jornadas, regras-de-negocio) · [[content/sdd-gsd-aplicado|sdd-gsd-aplicado (histórico)]]
- `contextos/` — **apenas material cliente após triagem 2026-04-23**: `DADOS PARA CADASTRO.md`, `Manual Identidade Visual - Master Síndico.md`, `Matriz Funcional.md`, `novo escopo-7.md`. Demais análises técnicas legado movidas pra `_source-material/`
- `client-vault/` — Vault Obsidian externo do cliente (visão original; preservado — linguagem ubíqua)
- `_archive/` — placeholders vazios + duplicatas arquivadas
- `.claude/` — skills e settings Claude Code locais

### Material movido em 2026-04-23 (fora do vault, preservado em `_source-material/`)

Regra dura: nada apagado, só movido. Destino: `/home/vinni/Documentos/Projetos/Privados/automation-agents/_source-material/2026-04-23-raw-source/from-master-sindico/`.

- ~~`legacy-docs/`~~ — 72 arquivos (duplicatas ou legado TS)
- ~~`plans/`~~ — 2 arquivos (`architecture-refined.md`, `master-sindico-implementation-plan.md`; superseded por [[ROADMAP]] + [[10-schedule/_moc]])
- ~~`MasterSindico_extracted/`~~ — 47 arquivos (duplicata Pandoc de `client-vault/`)
- Parte de `contextos/` — 23 arquivos (structured_dump, 8 ANALISE_*, CONTEXT_BRAIN, context, DATABASE_VALIDATION, MUX_VIDEO_CONTEXT, Manual Técnico, ONBOARDING, PROFILE_MODULE_DOCS, TECHNICAL_ANALYSIS, format_js, modelagem, profiles-context_ts, stripe-context, stripe-helper, telas, telas_txt, extracts/)

Índice completo em `_source-material/2026-04-23-raw-source/README-reference/INDEX.md`.

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
| Integrar provider | [[../../10-knowledge/providers/_moc]] (Stripe, Mux, Zitadel, Livekit, Railway) |

---

## Status atual (2026-04-23)

- **Sprint 9 ✅ entregue autônoma** (F1-F33; SESSION_CHARTER §4)
- **Sprint 10 em curso** — finalização M1 (frontend/mobile + polish + deploy prep)
- **M1 prazo**: 2026-05-08
- **AUDIT**: 🔴 A-023, A-024 abertos (Sprint 10); 🟡 vazio
- **STATE**: D-030 em aberto (configs `.claude/settings.json` pra web/app)
- **Gates (snapshot)**: build ✅ · vet ✅ · test -race ✅ · integration ✅ · gosec -sev high 0 ✅ · govulncheck 0 ✅
- **Limpeza 2026-04-23**: ~144 arquivos legados movidos pra `_source-material/` (fora do vault); vault ativo focado em regra-de-negócio + linguagem ubíqua

---

## Vizinhos no grafo

- [[../_moc]] — pasta 50-projects
- [[../../CLAUDE]] — contrato raiz do vault
- [[../../30-agents/playbooks/_moc]] — playbooks obrigatórios
- [[../../40-templates/_moc]] — templates que este projeto instancia
- [[../../20-stacks/go/_moc]] — stack Go padrões
- [[../../10-knowledge/_moc]] — conhecimento técnico atemporal
- [[../../10-knowledge/providers/_moc]] — providers globais (Stripe, Mux, Zitadel, Livekit, Railway)
