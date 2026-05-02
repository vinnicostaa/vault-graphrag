---
title: MOC — GSD Agents Library
type: moc
tags: [moc, agents, gsd, external-reference, library]
source_collection: https://github.com/gsd-build/get-shit-done
source_date: 2026-04-22
created: 2026-04-24
updated: 2026-04-24
---

# GSD Agents Library

Catálogo destilado dos **33 agentes especializados** do repositório [`gsd-build/get-shit-done`](https://github.com/gsd-build/get-shit-done) — uma das referências canônicas de arquitetura multi-agente para desenvolvimento com LLM (ver [[../../10-knowledge/methodology/gsd-get-shit-done]] e [[../../60-sources/get-shit-done/_toc]]).

> **Regra zero desta biblioteca**: esta pasta **informa** o pipeline do vault; **não substitui** [[../planner/_moc|planner]], [[../worker/_moc|worker]], [[../reviewer/_moc|reviewer]], [[../orchestrator/_moc|orchestrator]]. A correspondência vive em [[pipeline-mapping]].

> **Staleness**: este catálogo é congelado no *commit/date* indicado nos frontmatters dos arquivos. Para re-sincronizar com upstream, ver [[../playbooks/gsd-resync]].

---

## Organização

**Híbrido** (ver [[../../00-meta/STATE|D-vault-004]] para rationale):

- **33 reference-cards curtos** (`card-gsd-<name>.md`) — 1 por agent, 40-60 linhas, destilando intenção + I/O + ferramentas + padrão-chave replicável.
- **6 famílias** (`family-<name>.md`) — agrupam cards por função no ciclo GSD; sintetizam padrões que emergem do grupo.
- Material bruto preservado canonicamente em `[[../../60-sources/get-shit-done/raw/agents/]]` (EN).

## 6 famílias (fiel ao TOC original)

| Família | N | Entry | Foco |
|---|---:|---|---|
| Research & Análise | 10 | [[family-research]] | Investigar, mapear, sintetizar |
| Planning & Roadmap | 5 | [[family-planning]] | Decompor, ordenar, alocar |
| Executor & Code | 4 | [[family-executor]] | Implementar, consertar, revisar |
| Verifier & Audit | 8 | [[family-audit]] | Verificar, auditar, certificar |
| Debug | 2 | [[family-debug]] | Investigar falhas post-mortem |
| Docs | 4 | [[family-docs]] | Classificar, sintetizar, verificar, escrever |

**Total**: 33 agents.

## Pipeline crosswalk

A tabela completa de correspondência Planner / Worker / Reviewer / Orchestrator × GSD agents vive em [[pipeline-mapping]].

## Todos os cards (A-Z)

Cards ficam nesta pasta com prefixo `card-gsd-`. Lista alfabética:

- [[card-gsd-advisor-researcher]] · [[card-gsd-ai-researcher]] · [[card-gsd-assumptions-analyzer]]
- [[card-gsd-code-fixer]] · [[card-gsd-code-reviewer]] · [[card-gsd-codebase-mapper]]
- [[card-gsd-debug-session-manager]] · [[card-gsd-debugger]]
- [[card-gsd-doc-classifier]] · [[card-gsd-doc-synthesizer]] · [[card-gsd-doc-verifier]] · [[card-gsd-doc-writer]]
- [[card-gsd-domain-researcher]]
- [[card-gsd-eval-auditor]] · [[card-gsd-eval-planner]] · [[card-gsd-executor]]
- [[card-gsd-framework-selector]]
- [[card-gsd-integration-checker]] · [[card-gsd-intel-updater]]
- [[card-gsd-nyquist-auditor]]
- [[card-gsd-pattern-mapper]] · [[card-gsd-phase-researcher]] · [[card-gsd-plan-checker]] · [[card-gsd-planner]] · [[card-gsd-project-researcher]]
- [[card-gsd-research-synthesizer]] · [[card-gsd-roadmapper]]
- [[card-gsd-security-auditor]]
- [[card-gsd-ui-auditor]] · [[card-gsd-ui-checker]] · [[card-gsd-ui-researcher]]
- [[card-gsd-user-profiler]]
- [[card-gsd-verifier]]

## Frontmatter canônico dos cards

```yaml
---
title: gsd-<name>
type: agent-contract
tags: [agent, gsd, <family>, external-reference]
source: https://github.com/gsd-build/get-shit-done/blob/main/.claude/agents/gsd-<name>.md
source_raw: 60-sources/get-shit-done/raw/agents/gsd-<name>.md
source_date: 2026-04-22   # data do clone; fix em gsd-resync
source_commit: <sha>       # commit do upstream no clone (pendente — capturar na 1ª resync)
created: 2026-04-24
updated: 2026-04-24
aliases: [<name>]
---
```

## Vizinhos no grafo

- [[../_moc]] — pipeline do vault
- [[../playbooks/_moc]] — inclui [[../playbooks/gsd-resync]]
- [[../planner/_moc]] · [[../worker/_moc]] · [[../reviewer/_moc]] · [[../orchestrator/_moc]] — pastas operacionais do pipeline (gsd-library informa mas não substitui)
- [[../../10-knowledge/methodology/gsd-get-shit-done]] — ergonomia de entrega (conceito)
- [[../../60-sources/get-shit-done/_toc]] — TOC autoritativo da fonte original
- [[../../00-meta/STATE]] — D-vault-004 registra esta biblioteca
