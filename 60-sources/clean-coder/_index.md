---
title: blog.cleancoder — Uncle Bob — índice local
type: source
tags: [source, clean-architecture, uncle-bob, solid, software-craftsmanship]
source_url: https://blog.cleancoder.com
ingested: 2026-04-22
---

# blog.cleancoder — Uncle Bob

Blog de **Robert C. Martin (Uncle Bob)**. Fonte primária sobre **Clean Architecture**, **SOLID**, **software craftsmanship**, **TDD**, **component design**. Posts de 2011 até 2023.

## Conteúdo principal

Tópicos dominantes (cronológico inverso):
- **Functional programming em Clojure** — posts finais (2021-2023): Functional Classes, Functional Duplications, Spacewar
- **Type systems** — On Types, More On Types (2021)
- **Architecture** — Clean Architecture (2012 original), Screaming Architecture, Component Cohesion, SOLID Relevance
- **Practices** — TDD, Pairing Guidelines, Code Calisthenics, Loops (Loopy)
- **Professionalism** — The Disinvitation, Conference Conduct, Developer Certification
- **Design** — REPL Driven Design, if-else-switch, Roots

## Como o agente consulta

**Retrieval de princípios** — ao aplicar SOLID ou Clean Arch em task:
1. Busca posts referentes (ex: "Single Responsibility", "Dependency Inversion")
2. Cita postura canônica do Uncle Bob
3. Linka em PR description

**Destilação**: conforme princípios forem aplicados, criar notas atômicas em:
- `[[../../10-knowledge/principles/_moc|10-knowledge/principles/]]` — SOLID detalhado, Clean Code rules
- `[[../../10-knowledge/architecture/_moc|10-knowledge/architecture/]]` — Clean Arch, Component Cohesion, Screaming Architecture

## Estrutura canônica (pós reorg 2026-04-22)

```
clean-coder/
├── _index.md
├── _toc.md           # índice curado por tema ← começar aqui
├── raw/              # 171 posts achatados: yyyy-mm-dd-<slug>.md
└── _notes/           # staging pre-promoção pro 10-knowledge/
```

Arquivos renomeados no formato `yyyy-mm-dd-<slug>.md` (ex: `2012-08-13-the-clean-architecture.md`). Atribuição preservada no corpo. Profundidade anterior do blog (blog.cleancoder.com/uncle-bob/YYYY/MM/DD/) colapsada — data vive no nome.

## Status de ingestão

- ✅ 173 HTMLs convertidos via pandoc (`-t gfm-raw_html`) — 171 posts + 2 índices
- ✅ Reorganizado 2026-04-22: achatado em `raw/yyyy-mm-dd-<slug>.md`, ruído de footer removido (mesmo valor, nomes limpos)
- 📋 A destilar: ver [[_toc#Prioridade de destilação]]
- ⚠️ Alguns posts preservam navegação/footer do blog — conteúdo útil está no miolo

## Licença / atribuição

Blog público, posts do Uncle Bob. Citação **sempre** com atribuição direta ao autor + link pro post original. **Não republicar** conteúdo íntegro em docs públicas.

## Posts prioritários pra destilar

1. **2012 — The Clean Architecture** — base do conceito
2. **2020 — Solid Relevance** — SOLID ainda vale?
3. **2021 — if-else-switch** — antipattern conhecido
4. **2013 — Screaming Architecture** — arquitetura deve "gritar" o domínio
5. **2014 — FP Basics: Iteration** — comparação FP × OO

## Links

- [[_toc]] — **índice curado por tema (entrada principal)**
- [[_notes/_moc]] — staging de destilados
- [[../_moc]] — sources
- [[../../10-knowledge/principles/_moc]] — destino SOLID / Clean Code
- [[../../10-knowledge/architecture/_moc]] — destino Clean Arch / Component Design
- Site original: https://blog.cleancoder.com
