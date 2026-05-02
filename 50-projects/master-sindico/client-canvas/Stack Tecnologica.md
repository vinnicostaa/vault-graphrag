---
title: Stack Tecnologica (diagrama do cliente)
type: note
tags: [master-sindico, client-canvas, diagram, mermaid]
project: master-sindico
source: Clients/MasterSindico original .canvas (2026-03-10 a 2026-03-24) — convertido para MD+mermaid em 2026-04-24
created: 2026-04-24
---

# Stack Tecnologica

Diagrama original do cliente convertido de `.canvas` (Obsidian Canvas) para Mermaid. **Visão visual** dos fluxos/arquitetura; conteúdo canônico vive em [[../04-requirements/_moc]] + [[../02-architecture/_moc]].

## Diagrama

```mermaid
graph TD
  g_front["📦 Frontend"]:::group
  SOLID["SolidJS"]
  TANSTACK["TanStack Router + Query + Form"]
  UNO["UnoCSS"]
  AXIOS["Axios"]
  g_back["📦 Backend"]:::group
  FASTIFY["Fastify"]
  DRIZZLE["Drizzle ORM"]
  OSO["Oso + jose (JWT)"]
  AWILIX["Awilix (DI)"]
  ZOD["Zod + drizzle-zod"]
  g_infra["📦 Infraestrutura"]:::group
  PG["PostgreSQL"]
  WS["WebSocket (Assembleia)"]
  S3["Object Storage"]
  g_ext["📦 Serviços Externos"]:::group
  MUX["Mux / Cloudflare Stream"]
  MP["Mercado Pago"]
  SES["AWS SES / SendGrid"]
  SMS["Twilio / Zenvia"]
  CEP["ViaCEP"]
  OAUTH["Google / Apple OAuth"]
  SOLID --> FASTIFY
  FASTIFY --> PG
  FASTIFY --> WS
  FASTIFY --> S3
  FASTIFY -->|Adapter Layer| MUX
  FASTIFY -->|Adapter Layer| MP
  FASTIFY -->|Adapter Layer| SES
  FASTIFY -->|Adapter Layer| SMS
  FASTIFY -->|Adapter Layer| CEP
  FASTIFY -->|Adapter Layer| OAUTH
  classDef group fill:#eef,stroke:#66a,stroke-width:2px,color:#000
```

## Nodes (22)

- **[GROUP]** `g_front` — Frontend
- `SOLID` — SolidJS
- `TANSTACK` — TanStack Router + Query + Form
- `UNO` — UnoCSS
- `AXIOS` — Axios
- **[GROUP]** `g_back` — Backend
- `FASTIFY` — Fastify
- `DRIZZLE` — Drizzle ORM
- `OSO` — Oso + jose (JWT)
- `AWILIX` — Awilix (DI)
- `ZOD` — Zod + drizzle-zod
- **[GROUP]** `g_infra` — Infraestrutura
- `PG` — PostgreSQL
- `WS` — WebSocket (Assembleia)
- `S3` — Object Storage
- **[GROUP]** `g_ext` — Serviços Externos
- `MUX` — Mux / Cloudflare Stream
- `MP` — Mercado Pago
- `SES` — AWS SES / SendGrid
- `SMS` — Twilio / Zenvia
- `CEP` — ViaCEP
- `OAUTH` — Google / Apple OAuth

## Edges (10)

- `SOLID` → `FASTIFY`
- `FASTIFY` → `PG`
- `FASTIFY` → `WS`
- `FASTIFY` → `S3`
- `FASTIFY` → `MUX` — _Adapter Layer_
- `FASTIFY` → `MP` — _Adapter Layer_
- `FASTIFY` → `SES` — _Adapter Layer_
- `FASTIFY` → `SMS` — _Adapter Layer_
- `FASTIFY` → `CEP` — _Adapter Layer_
- `FASTIFY` → `OAUTH` — _Adapter Layer_

## Links

- [[_moc]] — índice dos canvas do cliente
- [[../CLAUDE]] — contrato do projeto
- [[../02-architecture/_moc]]
- [[../04-requirements/_moc]]