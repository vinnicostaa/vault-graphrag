---
title: Modelo ABAC (diagrama do cliente)
type: note
tags: [master-sindico, client-canvas, diagram, mermaid]
project: master-sindico
source: Clients/MasterSindico original .canvas (2026-03-10 a 2026-03-24) — convertido para MD+mermaid em 2026-04-24
created: 2026-04-24
---

# Modelo ABAC

Diagrama original do cliente convertido de `.canvas` (Obsidian Canvas) para Mermaid. **Visão visual** dos fluxos/arquitetura; conteúdo canônico vive em [[../04-requirements/_moc]] + [[../02-architecture/_moc]].

## Diagrama

```mermaid
graph TD
  U["Usuário<br/>userId · role · plan"]
  T["Tenant<br/>tenantId · type"]
  A["Ação<br/>action · resource"]
  C["Contexto<br/>quota · time"]
  P["Policy Engine<br/>Oso"]
  R["Resultado<br/>allowed · features"]
  U --> P
  T --> P
  A --> P
  C --> P
  P --> R
  classDef group fill:#eef,stroke:#66a,stroke-width:2px,color:#000
```

## Nodes (6)

- `U` — Usuário · userId · role · plan
- `T` — Tenant · tenantId · type
- `A` — Ação · action · resource
- `C` — Contexto · quota · time
- `P` — Policy Engine · Oso
- `R` — Resultado · allowed · features

## Edges (5)

- `U` → `P`
- `T` → `P`
- `A` → `P`
- `C` → `P`
- `P` → `R`

## Links

- [[_moc]] — índice dos canvas do cliente
- [[../CLAUDE]] — contrato do projeto
- [[../02-architecture/_moc]]
- [[../04-requirements/_moc]]