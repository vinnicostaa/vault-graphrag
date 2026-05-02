---
title: Modelo Planos (diagrama do cliente)
type: note
tags: [master-sindico, client-canvas, diagram, mermaid]
project: master-sindico
source: Clients/MasterSindico original .canvas (2026-03-10 a 2026-03-24) — convertido para MD+mermaid em 2026-04-24
created: 2026-04-24
---

# Modelo Planos

Diagrama original do cliente convertido de `.canvas` (Obsidian Canvas) para Mermaid. **Visão visual** dos fluxos/arquitetura; conteúdo canônico vive em [[../04-requirements/_moc]] + [[../02-architecture/_moc]].

## Diagrama

```mermaid
graph TD
  B["Base<br/>Gratuito"]
  PRO["Pro<br/>Mensal/Anual"]
  T["Trial<br/>15d/7d/30d"]
  PLUS["Plus<br/>Mensal/Anual"]
  T -->|expira sem pagamento| B
  T -->|paga| PLUS
  T -->|paga| PRO
  B -->|upgrade| PLUS
  B -->|upgrade| PRO
  PLUS -->|upgrade| PRO
  PRO -->|downgrade| PLUS
  PLUS -->|downgrade| B
  PRO -->|expira| B
  PLUS -->|expira| B
  classDef group fill:#eef,stroke:#66a,stroke-width:2px,color:#000
```

## Nodes (4)

- `B` — Base · Gratuito
- `PRO` — Pro · Mensal/Anual
- `T` — Trial · 15d/7d/30d
- `PLUS` — Plus · Mensal/Anual

## Edges (10)

- `T` → `B` — _expira sem pagamento_
- `T` → `PLUS` — _paga_
- `T` → `PRO` — _paga_
- `B` → `PLUS` — _upgrade_
- `B` → `PRO` — _upgrade_
- `PLUS` → `PRO` — _upgrade_
- `PRO` → `PLUS` — _downgrade_
- `PLUS` → `B` — _downgrade_
- `PRO` → `B` — _expira_
- `PLUS` → `B` — _expira_

## Links

- [[_moc]] — índice dos canvas do cliente
- [[../CLAUDE]] — contrato do projeto
- [[../02-architecture/_moc]]
- [[../04-requirements/_moc]]