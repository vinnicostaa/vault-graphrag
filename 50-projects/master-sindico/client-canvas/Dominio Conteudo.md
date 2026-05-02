---
title: Dominio Conteudo (diagrama do cliente)
type: note
tags: [master-sindico, client-canvas, diagram, mermaid]
project: master-sindico
source: Clients/MasterSindico original .canvas (2026-03-10 a 2026-03-24) — convertido para MD+mermaid em 2026-04-24
created: 2026-04-24
---

# Dominio Conteudo

Diagrama original do cliente convertido de `.canvas` (Obsidian Canvas) para Mermaid. **Visão visual** dos fluxos/arquitetura; conteúdo canônico vive em [[../04-requirements/_moc]] + [[../02-architecture/_moc]].

## Diagrama

```mermaid
graph TD
  g_cont["📦 📚 CONTEÚDO"]:::group
  VIDEO["Vídeos Institucionais<br/>Upload · Player"]
  AGENCY["Agência Marketing<br/>Gestão delegada"]
  SEARCH["Busca & Discovery<br/>Full-text"]
  LMS["Academia LMS<br/>5 áreas"]
  GOV["📜 Governança"]
  VIDEO -->|conteúdo para| SEARCH
  AGENCY -->|gerencia| VIDEO
  SEARCH -->|descobre| VIDEO
  SEARCH -->|descobre| LMS
  LMS -->|integra| GOV
  classDef group fill:#eef,stroke:#66a,stroke-width:2px,color:#000
```

## Nodes (6)

- **[GROUP]** `g_cont` — 📚 CONTEÚDO
- `VIDEO` — Vídeos Institucionais · Upload · Player
- `AGENCY` — Agência Marketing · Gestão delegada
- `SEARCH` — Busca & Discovery · Full-text
- `LMS` — Academia LMS · 5 áreas
- `GOV` — 📜 Governança

## Edges (5)

- `VIDEO` → `SEARCH` — _conteúdo para_
- `AGENCY` → `VIDEO` — _gerencia_
- `SEARCH` → `VIDEO` — _descobre_
- `SEARCH` → `LMS` — _descobre_
- `LMS` → `GOV` — _integra_

## Links

- [[_moc]] — índice dos canvas do cliente
- [[../CLAUDE]] — contrato do projeto
- [[../02-architecture/_moc]]
- [[../04-requirements/_moc]]