---
title: Ingestion Log — Master Síndico v2
type: note
tags: [ingestion, log, audit-trail, master-sindico, v2]
created: 2026-04-23
updated: 2026-04-23
---

# Ingestion Log — Master Síndico v2

Trilha de auditoria: cada arquivo do legado que é absorvido pela v2, para onde foi destilado, status da destilação. Índice cruzado em `90-ingested/INDEX.md`.

**Regra**: nada do legado é apagado. Aqui registra a rota. O apagamento só acontece na Fase 6 com confirmação explícita do usuário.

---

## Fontes do legado

1. **Vault ativo** — `../vault/50-projects/master-sindico/` (overlay canônico 00-12 + client-vault + contextos + specs Kiro + legacy-docs + plans + _archive/ interno).
2. **Archive** — `../_archive/2026-04-22-specs-consolidation/` (implementation-plan, architecture-refined, ARCHITECTURE, AGENTS, ANALISE_COMPLETA, CODING_MANUAL, múltiplas cópias CLAUDE/README).
3. **Archive** — `../_archive/2026-04-22-vault-ingestion/` (sources spec-kit/clean-coder/refactoring-guru/get-shit-done/fernando-franco-valle + obsidian-config + non-md-binaries).
4. **Source-material** — `../_source-material/2026-04-23-raw-source/from-master-sindico/contextos/` (ANALISE_*.md, structured_dump.md).
5. **Source-material** — `../_source-material/2026-04-23-raw-source/from-master-sindico/MasterSindico_extracted/` (dump Pandoc do zip original do cliente).

---

## Relatórios Fase 1 desta sessão (destilados prontos pra consumir durante Fase 3)

| ID | Origem | Tamanho | Destino previsto |
|---|---|---|---|
| R1 | Legado arquitetural (Explore agent 1) | inline | 02-architecture/patterns.md + adr/ + 07-quality/ |
| R2 | Domínio/requisitos/client-vault (Explore agent 2) | inline | 00-product/*, 01-domain/*, 04-requirements/* |
| R3 | Governança viva (Explore agent 3) | inline | STATE.md + AUDIT.md + ROADMAP.md + 11-audit/ |
| R4 | Telas/jornadas/design system (Explore agent 4) | inline | 03-subprojects/web/ui-catalog.md + design-system.md + 04-requirements/matrix-functional.md + registration-data.md |
| R5 | Providers/fluxos (Explore agent 5) | 91.7K persistido em `/home/vinni/.claude/projects/-home-vinni-Documentos-Projetos-Privados-automation-agents/1d72f407-cd66-4ed6-8ee6-c838b924ac35/tool-results/toolu_01XSALYmMs5xjrsWkhNWwDEg.json` | 02-architecture/event-driven.md + 04-requirements/functional/*.md (por provider) |
| R6 | Specs Kiro + legacy-docs + ANALISE_* (Explore agent 6) | 72.4K persistido em `/home/vinni/.claude/projects/-home-vinni-Documentos-Projetos-Privados-automation-agents/1d72f407-cd66-4ed6-8ee6-c838b924ac35/tool-results/toolu_01C8SgQjHnruKN61WnsvfFXK.json` | 04-requirements/ + 01-domain/ + 02-architecture/adr/ + 90-ingested/INDEX.md |
| R7 | Knowledge transversal vault (Explore agent 7) | inline | linkar em 02-architecture/patterns.md + 07-quality/* + 08-security/* + 09-testing/* |

---

## Buckets de ingestão

### Bucket A — Vault ativo `vault/50-projects/master-sindico/`

Ainda a processar arquivo-por-arquivo na Fase 3:
- 00-product/* → 00-product/
- 01-domain/* → 01-domain/
- 02-architecture/* → 02-architecture/
- 03-subprojects/* → 03-subprojects/
- 04-requirements/* → 04-requirements/
- 05-tasks/* → 05-tasks/
- 06-execution/* → 06-execution/
- 07-quality/* → 07-quality/
- 08-security/* → 08-security/
- 09-testing/* → 09-testing/
- 10-schedule/* → 10-schedule/
- 11-audit/* → 11-audit/
- 12-docs/* → 12-docs/
- STATE.md / AUDIT.md / ROADMAP.md / SESSION_CHARTER.md / STEERING.md / DoR-DoD.md / CLAUDE.md / _moc.md / overview.md / plugins.md → raiz + atualização dos arquivos já criados
- legacy-reference.md → 02-architecture/legacy-reference.md (ou 90-ingested/ se só histórico)
- client-vault/** → 90-ingested/from-vault-50-projects-master-sindico/client-vault/ (fonte cliente preservada); destilado vai pra 00-product/ + 01-domain/
- contextos/** → 90-ingested/from-vault-50-projects-master-sindico/contextos/; destilado vai pra 04-requirements/ + 03-subprojects/web/ui-catalog
- legacy-docs/** → 90-ingested/from-vault-50-projects-master-sindico/legacy-docs/; destilado parcial pra 02-architecture/adr/ + 04-requirements/
- specs/** (Kiro) → 90-ingested/from-vault-50-projects-master-sindico/specs/; destilado vai pra 04-requirements/ + 01-domain/
- MasterSindico_extracted/** → redundante com client-vault (relatório Fase 1 confirmou). Arquivar em 90-ingested/ e não destilar separadamente.
- .claude/, .kiro/ → config local do projeto; preservar em 90-ingested/ (não canônico pra v2).
- _archive/** (interno do projeto) → 90-ingested/from-vault-50-projects-master-sindico/archive-interno/

### Bucket B — `_archive/2026-04-22-specs-consolidation/`

- master-sindico-implementation-plan.md (8 fases, 845 linhas) → 90-ingested/from-archive-specs-consolidation/ + destilado em 05-tasks/backlog.md + 10-schedule/cronograma-macro.md + 02-architecture/adr/0001.
- architecture-refined.md (653 linhas) → 90-ingested/ + destilado em 02-architecture/patterns.md + adr/0004 (Stripe abstração) + adr/0003.
- ARCHITECTURE.md (347 linhas) → 90-ingested/ + destilado parcial (Flutter-específico limitado) em 03-subprojects/mobile/architecture.md.
- AGENTS.md (30K) → 90-ingested/ + padrões destilados em 07-quality/ e 02-architecture/patterns.md.
- ANALISE_COMPLETA.md (77K) → 90-ingested/ + AP-###s destilados em 07-quality/PATTERNS.md + 08-security/.
- CODING_MANUAL.md (15K) → 90-ingested/ + destilado parcial pra 03-subprojects/mobile/patterns.md.
- CLAUDE (copy 1..4).md + CLAUDE.md → 90-ingested/ com diff semântico em _DIFF.md; canônico já consolidado em nosso CLAUDE.md.
- README (copy 1..5).md + README.md → 90-ingested/ com diff; canônico em 12-docs/README.md.
- anotacoes.md (195B) → 90-ingested/ (arquivar; sem destilação separada).
- System Architecture - API Design.md / Identity.md (ambos 0 bytes) → 90-ingested/ (arquivar como vazios).

### Bucket C — `_archive/2026-04-22-vault-ingestion/`

- sources/{blog.cleancoder, fernandofrancovalle.com, get-shit-done, refactoring.guru, spec-kit}/** → **não absorver pra v2**. Essas fontes já estão destiladas em `vault/10-knowledge/` e `vault/60-sources/` (relatório Fase 1c confirmou). Linkar quando relevante.
- obsidian-config/{.obsidian, .smart-env} → ignorar (config tooling, não produto).
- non-md-binaries/ → ignorar (font Michroma.ttf + scheduled_tasks.lock; a fonte já está referenciada em design-system).

### Bucket D — `_source-material/2026-04-23-raw-source/from-master-sindico/contextos/`

- ANALISE_COMPLETA_API.md (48K) → 90-ingested/from-source-material-contextos/ + amostragem destilada em 03-subprojects/backend/architecture.md.
- ANALISE_COMPLETA_LINHA_POR_LINHA.md (155K, 19/276 arquivos) → 90-ingested/ (grep dirigido, não ler linearmente).
- analise_completa.md (23K) → 90-ingested/ + destilado em 03-subprojects/backend/architecture.md.
- ANALISE_COMPLETA.md (180K) → 90-ingested/ + destilado já em Fase 1.
- ANALISE_COMPLETA_MODELAGEM.md (30K) → 90-ingested/ + destilado em 01-domain/aggregates/.
- ANALISE_TECNICA_COMPLETA_V2.md (247K, 276 arquivos) → 90-ingested/ + score/gaps por módulo destilados em 07-quality/PATTERNS.md.
- ANALISE_TECNICA.md (34K) → 90-ingested/ + parcial em 02-architecture/patterns.md.
- analise_integrada.md (16K) → 90-ingested/ + destilado.
- structured_dump.md (456K TS concatenado) → 90-ingested/ + grep dirigido (regras de negócio, TODOs, schemas Drizzle, nomes de rotas) + conclusões em 90-ingested/INDEX.md.

### Bucket E — `_source-material/2026-04-23-raw-source/from-master-sindico/MasterSindico_extracted/`

- 47 arquivos (352K) — 100% duplicata do client-vault confirmado no relatório Fase 1. Arquivar em 90-ingested/from-source-material-masterSindico-extracted/ sem destilação separada.

---

## Protocolo de destilação

Para cada arquivo do legado:

1. **Classificar**: é fonte-cliente (client-vault), doc-destilado (legacy-reference, overview), análise técnica (ANALISE_*), config (obsidian, kiro), dump bruto (structured_dump), ou duplicata?
2. **Decidir destino**: 
   - **Destilar** em artefato canônico da v2 (00-12) — se tem conteúdo único e aplicável.
   - **Arquivar bruto** em 90-ingested/ com entrada em INDEX.md — se é fonte-cliente ou dump.
   - **Ignorar** — se é config tooling ou 100% duplicata.
3. **Registrar**: linha em 90-ingested/INDEX.md com origem + destino + status + nota.
4. **Respeitar fonte original**: quando citar, indicar `source: <caminho-absoluto-legado>` no frontmatter.

---

## Status agregado

- **Fase 0**: ✅ Setup da v2 concluído.
- **Fase 1**: ✅ Relatórios destilados (7 agentes Explore em 2 rodadas).
- **Fase 1b (ingestão minuciosa)**: ⏳ a executar — ler R5/R6 persistidos + client-vault completo + cópias CLAUDE/README diff + structured_dump grep.
- **Fase 2 (research web)**: ⏳ a iniciar — disparar agentes paralelos por vertical.
- **Fase 3+**: conforme plano.
