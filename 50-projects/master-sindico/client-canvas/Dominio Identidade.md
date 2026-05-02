---
title: Dominio Identidade (diagrama do cliente)
type: note
tags: [master-sindico, client-canvas, diagram, mermaid]
project: master-sindico
source: Clients/MasterSindico original .canvas (2026-03-10 a 2026-03-24) — convertido para MD+mermaid em 2026-04-24
created: 2026-04-24
---

# Dominio Identidade

Diagrama original do cliente convertido de `.canvas` (Obsidian Canvas) para Mermaid. **Visão visual** dos fluxos/arquitetura; conteúdo canônico vive em [[../04-requirements/_moc]] + [[../02-architecture/_moc]].

## Diagrama

```mermaid
graph TD
  g_id["📦 🔐 IDENTIDADE"]:::group
  AUTH["Autenticação<br/>JWT · OAuth"]
  ABAC["Autorização<br/>ABAC · Roles"]
  REG["Registro<br/>4 tipos de usuário"]
  ONBOARD["Onboarding<br/>4-7 telas por persona"]
  BILLING["Billing<br/>Trial · Base · Plus · Pro"]
  FEATURE["Feature Access<br/>Matriz de features"]
  PROFILE["Perfis<br/>Síndico · Empresa"]
  AUTH --> ABAC
  ABAC --> FEATURE
  REG --> ONBOARD
  ONBOARD --> PROFILE
  BILLING --> FEATURE
  classDef group fill:#eef,stroke:#66a,stroke-width:2px,color:#000
```

## Nodes (8)

- **[GROUP]** `g_id` — 🔐 IDENTIDADE
- `AUTH` — Autenticação · JWT · OAuth
- `ABAC` — Autorização · ABAC · Roles
- `REG` — Registro · 4 tipos de usuário
- `ONBOARD` — Onboarding · 4-7 telas por persona
- `BILLING` — Billing · Trial · Base · Plus · Pro
- `FEATURE` — Feature Access · Matriz de features
- `PROFILE` — Perfis · Síndico · Empresa

## Edges (5)

- `AUTH` → `ABAC`
- `ABAC` → `FEATURE`
- `REG` → `ONBOARD`
- `ONBOARD` → `PROFILE`
- `BILLING` → `FEATURE`

## Links

- [[_moc]] — índice dos canvas do cliente
- [[../CLAUDE]] — contrato do projeto
- [[../02-architecture/_moc]]
- [[../04-requirements/_moc]]