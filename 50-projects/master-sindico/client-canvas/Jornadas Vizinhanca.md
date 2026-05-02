---
title: Jornadas Vizinhanca (diagrama do cliente)
type: note
tags: [master-sindico, client-canvas, diagram, mermaid]
project: master-sindico
source: Clients/MasterSindico original .canvas (2026-03-10 a 2026-03-24) — convertido para MD+mermaid em 2026-04-24
created: 2026-04-24
---

# Jornadas Vizinhanca

Diagrama original do cliente convertido de `.canvas` (Obsidian Canvas) para Mermaid. **Visão visual** dos fluxos/arquitetura; conteúdo canônico vive em [[../04-requirements/_moc]] + [[../02-architecture/_moc]].

## Diagrama

```mermaid
graph TD
  g_viz["📦 🏘️ VIZINHANÇA"]:::group
  VS["Jornada Síndico<br/>VS1-VS10"]
  VZ["Jornada Parceiro<br/>VZ1-VZ6"]
  VE["Jornada Empresa<br/>VE1-VE6"]
  VM["Jornada Morador<br/>VM1-VM7"]
  VZ -->|cria promoção| VS
  VZ -->|cria promoção| VM
  VS -->|indica parceiro| VZ
  VE -->|ativa promoção| VZ
  VM -->|ativa promoção| VZ
  VS -->|cria benefício exclusivo| VM
  classDef group fill:#eef,stroke:#66a,stroke-width:2px,color:#000
```

## Nodes (5)

- **[GROUP]** `g_viz` — 🏘️ VIZINHANÇA
- `VS` — Jornada Síndico · VS1-VS10
- `VZ` — Jornada Parceiro · VZ1-VZ6
- `VE` — Jornada Empresa · VE1-VE6
- `VM` — Jornada Morador · VM1-VM7

## Edges (6)

- `VZ` → `VS` — _cria promoção_
- `VZ` → `VM` — _cria promoção_
- `VS` → `VZ` — _indica parceiro_
- `VE` → `VZ` — _ativa promoção_
- `VM` → `VZ` — _ativa promoção_
- `VS` → `VM` — _cria benefício exclusivo_

## Links

- [[_moc]] — índice dos canvas do cliente
- [[../CLAUDE]] — contrato do projeto
- [[../02-architecture/_moc]]
- [[../04-requirements/_moc]]