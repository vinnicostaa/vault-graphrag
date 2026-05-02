---
title: Vault Index — automation-agents
type: moc
created: 2026-04-22T00:00:00.000Z
updated: '2026-04-24'
tags:
  - moc
  - root
aliases:
  - Vault Index
---

# Vault Index

Raiz do grafo de conhecimento do projeto **automation-agents** — ferramenta multi-agente genérica que usa Obsidian como cérebro persistente dos agentes Planner / Worker / Dual Reviewer / Orchestrator.

> **Contrato do agente**: [[CLAUDE]] (leitura obrigatória em toda sessão).

---

## Navegação por área

| Área | Pasta | MOC | Função |
|---|---|---|---|
| **Raiz** | `/` | este arquivo + [[CLAUDE]] | Índice + contrato do agente |
| Meta & governança | `00-meta/` | [[00-meta/_moc]] | vault-guide, STATE/AUDIT/CHARTER meta |
| Conhecimento técnico | `10-knowledge/` | [[10-knowledge/_moc]] | Arquitetura, padrões, princípios, metodologia, antipatterns, segurança, testes, banco, observabilidade, **providers** |
| Templates por stack | `20-stacks/` | [[20-stacks/_moc]] | Go, TypeScript, Rust, Flutter, PostgreSQL |
| Sistema multi-agente | `30-agents/` | [[30-agents/_moc]] | Planner, Worker, Reviewer, Orchestrator, memória, sessão, inbox, **playbooks**, skills |
| Templates replicáveis | `40-templates/` | [[40-templates/_moc]] | CLAUDE.md.template, project-scaffold, SPECs, STATE/AUDIT/CHARTER templates |
| Projetos concretos | `50-projects/` | [[50-projects/_moc]] | Inventário vivo de projetos — instanciar novos via [[30-agents/playbooks/onboarding-novo-projeto]] |
| Fontes brutas | `60-sources/` | [[60-sources/_moc]] | refactoring.guru, cleancoder, spec-kit, get-shit-done, fernando-franco-valle (ingestão) |
| Arquivo | `90-archive/` | [[90-archive/_moc]] | Versões antigas, rascunhos consolidados |

---

## Contratos obrigatórios do agente

- [[CLAUDE]] — contrato raiz (estrutura, convenções, constituição técnica, research-loop, dual-check, anti-alucinação)
- [[30-agents/autonomous-execution]] — modo autônomo
- [[30-agents/mcp-integration]] — uso de MCPs (Context7, Obsidian, Postgres, GitHub, AWS docs)
- [[30-agents/playbooks/_moc]] — playbooks obrigatórios:
  - [[30-agents/playbooks/research-loop]] — ciclo de pesquisa antes de decidir
  - [[30-agents/playbooks/dual-check]] — validação por 2o agente
  - [[30-agents/playbooks/dependency-audit]] — auditoria periódica de deps
  - [[30-agents/playbooks/cve-scan]] — resposta a CVE
  - [[30-agents/playbooks/onboarding-novo-projeto]] — handshake agente↔dev pra novo projeto
  - [[30-agents/playbooks/plan-and-execute]] — SDD Plan fase operacionalizada
  - [[30-agents/playbooks/rollback]] — reverter mudança

---

## Convenções do vault

1. **Flat over deep** — pastas no máximo em 3 níveis. Navegação primária via `[[backlinks]]` + MOCs.
2. **Atomic notes** — cada nota cobre **um** conceito. Se crescer demais, split + link.
3. **Frontmatter sempre** — `title`, `type`, `tags`, `created`, `source?`, `updated?`, `aliases?`.
4. **Prefixos numéricos** (Johnny.Decimal) pra ordenação estável das pastas raiz e dentro de projetos.
5. **MOCs em `_moc.md`** (underscore prefix agrupa no topo na listagem). Raiz é `_index.md`.
6. **Idioma**: pt-BR no corpo das notas. Frontmatter e identifiers técnicos em inglês. Termos de domínio específicos de cada projeto (em `50-projects/<slug>/01-domain/ubiquitous-language.md`) preservam o idioma original quando carregam semântica legal ou cultural.
7. **Knowledge em 10-**, **Projeto específico em 50-**. Nunca misturar.

---

## Estrutura canônica de projeto (50-projects/<slug>/)

Cada projeto instancia [[40-templates/project-scaffold/_moc]]:

```
50-projects/<slug>/
├── CLAUDE.md                       # contrato do agente
├── _moc.md · overview · legacy-ref · plugins
├── STATE.md · AUDIT.md · SESSION_CHARTER.md · STEERING.md · ROADMAP.md · DoR-DoD.md
├── 00-product/   (vision, personas, business-model, glossary)
├── 01-domain/    (bounded-contexts, ubiquitous-language, business-rules, domain-rules, impl-rules, invariants)
├── 02-architecture/ (system-overview, clean-arch-mapping, patterns, beyond-corp, openapi/, adr/)
├── 03-subprojects/ (backend/, web/, mobile/ — cada um com README + architecture + patterns + requirements + tasks + security)
├── 04-requirements/ (global + functional por módulo + non-functional + compliance)
├── 05-tasks/        (global + by-sprint + by-module + backlog)
├── 06-execution/    (STEPS zero→prod + dev/QA/deploy/rollback/incident playbooks)
├── 07-quality/      (CLEAN_CODE · CLEAN_ARCH · CODE_CALISTHENICS · PATTERNS)
├── 08-security/     (BEYOND_CORP · threat-model · cve-process · OWASP · LGPD)
├── 09-testing/      (test-strategy + unit/integration/e2e/load/security)
├── 10-schedule/     (cronograma, milestones, sprint-calendar)
├── 11-audit/        (AUDIT · audit-log/ · dual-check-log · postmortems)
└── 12-docs/         (README, how-to/, runbooks/, adr-index, changelog)
```

Ver [[40-templates/project-scaffold/_moc]] pro esqueleto replicável.

---

## Como este vault é Graph-RAG

Ver [[10-knowledge/methodology/graph-rag]]. Resumo: estrutura de pastas + `[[links]]` formam o grafo simbólico; plugin Smart Connections adiciona similaridade vetorial por cima. Consultas combinam ambos.

Segue padrão Karpathy LLM Wiki: compilar conhecimento uma vez em markdown interconectado, Obsidian como visualizador, LLM como mantenedor. Ver [[10-knowledge/methodology/_moc]].

---

## Status do vault

- **Criado**: 2026-04-22
- **2026-04-22**: normalização 100% `.md` + CLAUDE.md raiz + 7 playbooks + overlay canônico `50-projects/master-sindico/`
- **2026-04-23**: criação de `10-knowledge/providers/` (stripe, mux, zitadel, livekit, railway) + destilação de SDD (`methodology/sdd-maturity-checklist`, `sdd-adoption-gaps`) + sincronização clone→vault real via MCP
- **Projetos ativos**: ver [[50-projects/_moc]] para inventário vivo.
- **Novos projetos**: instanciar via [[30-agents/playbooks/onboarding-novo-projeto]].

---

## Aviso sobre o clone filesystem

Existe um clone em filesystem (`automation-agents/vault/` no disco) que NÃO sincroniza bi-direcionalmente com este vault real. Agente usa exclusivamente MCP Obsidian para leitura/escrita. O clone é read-only pro agente.

## Links externos

- [[00-meta/vault-guide]] — como operar este vault no dia a dia
- [[CLAUDE]] — contrato raiz do agente
