---
title: INDEX — Material-fonte absorvido do legado
type: note
tags: [ingestion, index, audit-trail, master-sindico, v2]
created: 2026-04-23
updated: 2026-04-23
---

# INDEX — Material-fonte absorvido do legado

Índice cruzado de cada arquivo do legado que foi copiado pra `90-ingested/`. Origem, destino, status da destilação, nota.

> Regra dura: **nada é apagado** do legado antes da Fase 6 do plano. Aqui documentamos a rota.

---

## Consolidados da Fase 1b (2026-04-23)

Dois relatórios agentes-Explore persistidos em disco foram destilados integralmente pra arquivos canônicos:

| Consolidado | Escopo | Origem (JSON) | Arquivo canônico |
|---|---|---|---|
| Providers & fluxos | Stripe/Zitadel/Mux/Livekit/Twilio/SES|Resend/FCM/OpenSearch/NATS/S3/CloudFront/Route53/CloudWatch/Grafana/Sentry/OTel/Railway/ECS — interfaces, fluxos outbound, webhooks inbound, crons, saga/UoW, secrets, falhas conhecidas; 42 domain events cross-BC; 27 webhooks; 22 crons; 5 pipelines end-to-end | `/home/vinni/.claude/projects/-home-vinni-Documentos-Projetos-Privados-automation-agents/1d72f407-cd66-4ed6-8ee6-c838b924ac35/tool-results/toolu_01XSALYmMs5xjrsWkhNWwDEg.json` (91.7 KB / 2528 linhas no texto extraído) | [[_consolidado-providers-fluxos]] |
| Specs Kiro + legado + ANALISE_* | 46 arquivos primários, 103 requisitos EARS por BC (IDN/BIL/INT/COM/CTT/ASM/XDM), 18 NFRs, 20 ADRs + 46 tabelas data-model, 9 sprints de tasks, AP-001..AP-012, legacy summary TS→Go, divergências CLAUDE/README, 276 arquivos TS scored (ANALISE_TECNICA_COMPLETA_V2), mapa requisito↔task↔código | `/home/vinni/.claude/projects/-home-vinni-Documentos-Projetos-Privados-automation-agents/1d72f407-cd66-4ed6-8ee6-c838b924ac35/tool-results/toolu_01C8SgQjHnruKN61WnsvfFXK.json` (72.4 KB / 843 linhas no texto extraído) | [[_consolidado-specs-legado]] |

> Esses dois consolidados são a **entrada canônica da Fase 3** (destilação pra 00-product → 12-docs). Preservam fidelidade total ao relatório-fonte — copiar com fidelidade, não resumir.

---

## Status legenda

- **PENDENTE**: arquivo identificado mas ainda não copiado pra cá.
- **COPIADO**: arquivo presente em `90-ingested/`, sem destilação feita ainda.
- **DESTILADO**: conteúdo útil extraído pra artefatos canônicos da v2 (00-12).
- **DESTILADO-1B**: conteúdo destilado via relatórios Fase 1b em `_consolidado-*.md` (ponte rumo à Fase 3).
- **DUPLICATA**: arquivado sem destilação porque é 100% duplicata de outra fonte (ex: MasterSindico_extracted/ vs client-vault/).
- **IGNORADO**: não será absorvido (ex: configs de tooling, non-md binaries, 60-sources já destiladas).

---

## Fonte 1 — `vault/50-projects/master-sindico/**`

| Origem | Destino v2 | Status | Nota |
|---|---|---|---|
| `STATE.md` | raiz da v2 `STATE.md` (incorporado) | DESTILADO (Fase 0) | D-### ativos herdados; ver STATE.md §"Herança do legado" |
| `AUDIT.md` + `11-audit/AUDIT.md` | raiz `AUDIT.md` | DESTILADO (Fase 0) | A-### Sprint 10 herdados |
| `ROADMAP.md` | raiz `ROADMAP.md` | DESTILADO (Fase 0) | M1/M2/M3 preservados com nota de revisão |
| `SESSION_CHARTER.md` | raiz `SESSION_CHARTER.md` | DESTILADO (Fase 0) | ordens desta sessão |
| `STEERING.md` | raiz `STEERING.md` | DESTILADO (Fase 0) | não-negociáveis + 7 novos pra multi-produto |
| `DoR-DoD.md` | raiz `DoR-DoD.md` | DESTILADO (Fase 0) | adaptado pra remontagem |
| `CLAUDE.md` | raiz `CLAUDE.md` | DESTILADO (Fase 0) | contrato v2 + herança |
| `_moc.md` | raiz `_moc.md` | DESTILADO (Fase 0) | navegação canônica |
| `overview.md` | 00-product/vision.md (futuro) | PENDENTE | Fase 3 |
| `plugins.md` | 12-docs/plugins.md (futuro) | PENDENTE | Fase 4 |
| `legacy-reference.md` | 02-architecture/legacy-reference.md (futuro) | PENDENTE | Fase 3 |
| `00-product/*` | 00-product/ | PENDENTE | Fase 3 — destilar vision, personas, business-model, glossary + novo portfolio-de-produtos |
| `01-domain/*` | 01-domain/ | PENDENTE | Fase 3 — bounded-contexts, context-map, ubiquitous-language, invariants, aggregates/, events, rules, state-machines |
| `02-architecture/*` | 02-architecture/ | PENDENTE | Fase 3 — expandir com C4, multi-tenant topology, research-inspirations |
| `03-subprojects/*` | 03-subprojects/ | PENDENTE | Fase 3 — backend/web/mobile + ui-catalog + design-system |
| `04-requirements/*` | 04-requirements/ | PENDENTE | Fase 3 — global, functional/*, non-functional, compliance-lgpd, matrix-functional, registration-data |
| `05-tasks/*` | 05-tasks/ | PENDENTE | Fase 4 |
| `06-execution/*` | 06-execution/ | PENDENTE | Fase 4 |
| `07-quality/*` | 07-quality/ | PENDENTE | Fase 4 |
| `08-security/*` | 08-security/ | PENDENTE | Fase 3 — BEYOND_CORP + threat-model + owasp + cve-process + lgpd + pentest-checklist |
| `09-testing/*` | 09-testing/ | PENDENTE | Fase 4 |
| `10-schedule/*` | 10-schedule/ | PENDENTE | Fase 4 |
| `11-audit/*` | 11-audit/ | PENDENTE | Fase 4 |
| `12-docs/*` | 12-docs/ | PENDENTE | Fase 4 |
| `client-vault/**` (53 arquivos, 372K) | 90-ingested/from-vault-50-projects-master-sindico/client-vault/ | PENDENTE — COPIAR | Fonte cliente; ler linha a linha na Fase 1b; destilar pra 00-product + 01-domain/ubiquitous-language |
| `contextos/**` (28 arquivos, 2.1MB) | 90-ingested/from-vault-50-projects-master-sindico/contextos/ | COPIADO (parcial DESTILADO-1B → [[_consolidado-providers-fluxos]] pra stripe-context + MUX_VIDEO_CONTEXT) | telas.md, Manual IV, Matriz Funcional, DADOS PARA CADASTRO, ONBOARDING, PROFILE_MODULE_DOCS, novo escopo-7, stripe-context, MUX_VIDEO_CONTEXT, DATABASE_VALIDATION — Fase 1b cobriu integrações Stripe/Mux; resto pra 04-requirements + 03-subprojects/web/ui-catalog + 02-architecture/providers (Fase 3) |
| `legacy-docs/**` (16 arquivos, 1.1MB) | 90-ingested/from-vault-50-projects-master-sindico/legacy-docs/ | COPIADO (parcial DESTILADO-1B → [[_consolidado-specs-legado]]) | Plans antigos, arquiteturas, análises requirements — Fase 1b cobriu master-sindico-implementation-plan (primeiras 200 linhas) e architecture-refined; triagem + destilado completo pra Fase 3 |
| `specs/**` (Kiro) | 90-ingested/from-vault-50-projects-master-sindico/specs/ | COPIADO + DESTILADO-1B → [[_consolidado-specs-legado]] | requirements.md (26K), design.md (24K), tasks.md (24K), reference/antipatterns-to-avoid (AP-###), reference/legacy-summary — **Fase 1b consolidou integral**: 103 reqs EARS por BC, 18 NFRs, 20 ADRs, 46 tabelas, 9 sprints, 12 APs, legacy summary. Destilação final pra 04-requirements + 01-domain + 02-architecture/adr fica pra Fase 3. |
| `plans/**` (se existir) | 90-ingested/from-vault-50-projects-master-sindico/plans/ | PENDENTE — COPIAR | Plans arquiteturais antigos |
| `MasterSindico_extracted/**` (47 arquivos, 352K) | 90-ingested/from-source-material-masterSindico-extracted/ | DUPLICATA | 100% duplicata de client-vault (confirmado relatório Fase 1). Arquivar como completude |
| `.claude/` | — | IGNORADO | Config local Claude Code |
| `.kiro/` | — | IGNORADO | Config Kiro tooling |
| `_archive/**` (interno) | 90-ingested/from-vault-50-projects-master-sindico/archive-interno/ | PENDENTE — COPIAR | Histórico sanitizado (MOVE_LOG, content-moved, content-monolitico-2026-03/, content-discovery-2026-03/) — jornadas arquivadas são fonte canônica de M1-M13/S1-S60/E1-E16 |

## Fonte 2 — `_archive/2026-04-22-specs-consolidation/`

| Origem | Destino v2 | Status | Nota |
|---|---|---|---|
| `master-sindico-implementation-plan.md` (27K, 845 linhas) | 90-ingested/from-archive-specs-consolidation/ | COPIADO (parcial DESTILADO-1B → [[_consolidado-specs-legado]]) | 8 fases detalhadas: schemas, use cases, rotas. Destilar em 05-tasks/backlog + 10-schedule + ADRs |
| `architecture-refined.md` (22K, 653 linhas) | idem | COPIADO (parcial DESTILADO-1B → [[_consolidado-specs-legado]]) | Desacoplamento providers, PermissionService, separação módulos. ADR-0004 |
| `ARCHITECTURE.md` (17K, 347 linhas) | idem | COPIADO | Flutter-centric; parcial em 03-subprojects/mobile/architecture.md |
| `AGENTS.md` (30K) | idem | COPIADO | Clean Arch + CQRS + error handling + BeyondCorp — destilar em 07-quality + 02-architecture/patterns |
| `ANALISE_COMPLETA.md` (77K) | idem | COPIADO (parcial DESTILADO-1B → [[_consolidado-specs-legado]] §F e §J) | Linha-por-linha 130 arquivos TS; AP-001..AP-020 — destilar em 07-quality/PATTERNS + 08-security |
| `CODING_MANUAL.md` (15K) | idem | COPIADO | Flutter clean-arch. Parcial em 03-subprojects/mobile/patterns |
| `CLAUDE (copy 1..4).md` + `CLAUDE.md` | idem | COPIADO + DIFF DESTILADO-1B → [[_consolidado-specs-legado]] §H | Divergências catalogadas em §H; canônico já no raiz da v2 |
| `README (copy 1..5).md` + `README.md` (19K) | idem | COPIADO + DIFF DESTILADO-1B → [[_consolidado-specs-legado]] §H | Divergências catalogadas em §H; canônico em 12-docs/README (Fase 4) |
| `anotacoes.md` (195B) | idem | PENDENTE — COPIAR | Curto; arquivar bruto |
| `System Architecture - API Design.md` (0 B) | idem | PENDENTE — COPIAR | Arquivo vazio; arquivar |
| `System Architecture - Identity.md` (0 B) | idem | PENDENTE — COPIAR | Arquivo vazio; arquivar |

## Fonte 3 — `_archive/2026-04-22-vault-ingestion/`

| Origem | Destino v2 | Status | Nota |
|---|---|---|---|
| `sources/blog.cleancoder/` | — | IGNORADO | Já destilado em vault/10-knowledge + 60-sources |
| `sources/fernandofrancovalle.com/` | — | IGNORADO | Idem |
| `sources/get-shit-done/` | — | IGNORADO | Idem |
| `sources/refactoring.guru/` | — | IGNORADO | Idem |
| `sources/spec-kit/` | — | IGNORADO | Idem |
| `obsidian-config/.obsidian/` | — | IGNORADO | Config Obsidian |
| `obsidian-config/.smart-env/` | — | IGNORADO | Config Smart Connect |
| `non-md-binaries/Michroma-Regular.ttf` | — | IGNORADO | Fonte já referenciada em design-system |
| `non-md-binaries/scheduled_tasks.lock` | — | IGNORADO | Lock file |

## Fonte 4 — `_source-material/2026-04-23-raw-source/from-master-sindico/contextos/`

| Origem | Destino v2 | Status | Nota |
|---|---|---|---|
| `ANALISE_COMPLETA_API.md` (48K) | 90-ingested/from-source-material-contextos/ | PENDENTE — COPIAR | Análise package.json + arquitetura TS |
| `ANALISE_COMPLETA_LINHA_POR_LINHA.md` (155K, 19/276) | idem | PENDENTE — COPIAR | Análise incompleta; grep dirigido |
| `analise_completa.md` (23K) | idem | PENDENTE — COPIAR | Stack (Bun, Fastify, Drizzle, Stripe) |
| `ANALISE_COMPLETA.md` (180K) | idem | PENDENTE — COPIAR | Versão mais longa; já destilada em Fase 1 |
| `ANALISE_COMPLETA_MODELAGEM.md` (30K) | idem | PENDENTE — COPIAR | Modelagem vs escopo SOW 85%. Destilar em 01-domain/aggregates |
| `ANALISE_TECNICA_COMPLETA_V2.md` (247K, 276 arquivos) | idem | PENDENTE — COPIAR | V2 linha-por-linha. Score por módulo → 07-quality/PATTERNS |
| `ANALISE_TECNICA.md` (34K) | idem | PENDENTE — COPIAR | Análise resumida |
| `analise_integrada.md` (16K) | idem | PENDENTE — COPIAR | Código vs documentação, gaps do legado |
| `structured_dump.md` (456K, TS concatenado) | idem | PENDENTE — COPIAR | Dump bruto; grep dirigido por regras de negócio + TODOs |

## Fonte 5 — `_source-material/2026-04-23-raw-source/from-master-sindico/MasterSindico_extracted/`

| Origem | Destino v2 | Status | Nota |
|---|---|---|---|
| `MasterSindico/**` (47 arquivos, 352K) | 90-ingested/from-source-material-masterSindico-extracted/MasterSindico/ | DUPLICATA | 100% duplicata de client-vault; arquivar por completude |

---

## Próximos passos (Fase 1b)

1. Copiar (via `cp -r`) o material-fonte PENDENTE — COPIAR pra `90-ingested/`. Script de uma vez, preservando estrutura.
2. Ler integralmente os 2 relatórios Fase 1 persistidos em disco (providers-fluxos + specs-Kiro) via Read paginado — alimenta Fase 3.
3. Disparar agentes Explore pra client-vault linha a linha + cópias CLAUDE/README diff semântico + structured_dump grep dirigido.
4. Atualizar este INDEX.md conforme destilação avança (PENDENTE → COPIADO → DESTILADO).
