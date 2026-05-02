---
title: Master Canvas (diagrama do cliente)
type: note
tags: [master-sindico, client-canvas, diagram, mermaid]
project: master-sindico
source: Clients/MasterSindico original .canvas (2026-03-10 a 2026-03-24) — convertido para MD+mermaid em 2026-04-24
created: 2026-04-24
---

# Master Canvas

Diagrama original do cliente convertido de `.canvas` (Obsidian Canvas) para Mermaid. **Visão visual** dos fluxos/arquitetura; conteúdo canônico vive em [[../04-requirements/_moc]] + [[../02-architecture/_moc]].

## Diagrama

```mermaid
graph TD
  hub["hub"]
  arch["arch"]
  id["id"]
  com["com"]
  cont["cont"]
  asm["asm"]
  cross["cross"]
  hub --> id
  hub --> com
  hub --> cont
  hub --> asm
  hub --> cross
  arch --> hub
  classDef group fill:#eef,stroke:#66a,stroke-width:2px,color:#000
```

## Nodes (7)

- `hub` — 
- `arch` — 
- `id` — 
- `com` — 
- `cont` — 
- `asm` — 
- `cross` — 

## Edges (6)

- `hub` → `id`
- `hub` → `com`
- `hub` → `cont`
- `hub` → `asm`
- `hub` → `cross`
- `arch` → `hub`

## Links

- [[_moc]] — índice dos canvas do cliente
- [[../CLAUDE]] — contrato do projeto
- [[../02-architecture/_moc]]
- [[../04-requirements/_moc]]