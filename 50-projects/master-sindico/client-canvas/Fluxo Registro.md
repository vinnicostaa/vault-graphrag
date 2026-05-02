---
title: Fluxo Registro (diagrama do cliente)
type: note
tags: [master-sindico, client-canvas, diagram, mermaid]
project: master-sindico
source: Clients/MasterSindico original .canvas (2026-03-10 a 2026-03-24) — convertido para MD+mermaid em 2026-04-24
created: 2026-04-24
---

# Fluxo Registro

Diagrama original do cliente convertido de `.canvas` (Obsidian Canvas) para Mermaid. **Visão visual** dos fluxos/arquitetura; conteúdo canônico vive em [[../04-requirements/_moc]] + [[../02-architecture/_moc]].

## Diagrama

```mermaid
graph TD
  U1["Usuário<br/>POST /v1/auth/register"]
  R1["User Registry<br/>Valida/Hash/Cria User"]
  E1["Email Service<br/>Envia verificação"]
  U2["Usuário<br/>Acessa link email"]
  R2["User Registry<br/>Ativa conta (Active)"]
  O1["Onboarding<br/>Fluxo por persona"]
  U1 --> R1
  R1 --> E1
  E1 --> U2
  U2 --> R2
  R2 --> O1
  classDef group fill:#eef,stroke:#66a,stroke-width:2px,color:#000
```

## Nodes (6)

- `U1` — Usuário · POST /v1/auth/register
- `R1` — User Registry · Valida/Hash/Cria User
- `E1` — Email Service · Envia verificação
- `U2` — Usuário · Acessa link email
- `R2` — User Registry · Ativa conta (Active)
- `O1` — Onboarding · Fluxo por persona

## Edges (5)

- `U1` → `R1`
- `R1` → `E1`
- `E1` → `U2`
- `U2` → `R2`
- `R2` → `O1`

## Links

- [[_moc]] — índice dos canvas do cliente
- [[../CLAUDE]] — contrato do projeto
- [[../02-architecture/_moc]]
- [[../04-requirements/_moc]]