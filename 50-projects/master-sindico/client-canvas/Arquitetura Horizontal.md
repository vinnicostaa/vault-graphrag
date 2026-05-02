---
title: Arquitetura Horizontal (diagrama do cliente)
type: note
tags: [master-sindico, client-canvas, diagram, mermaid]
project: master-sindico
source: Clients/MasterSindico original .canvas (2026-03-10 a 2026-03-24) — convertido para MD+mermaid em 2026-04-24
created: 2026-04-24
---

# Arquitetura Horizontal

Diagrama original do cliente convertido de `.canvas` (Obsidian Canvas) para Mermaid. **Visão visual** dos fluxos/arquitetura; conteúdo canônico vive em [[../04-requirements/_moc]] + [[../02-architecture/_moc]].

## Diagrama

```mermaid
graph TD
  g_cross["📦 🛡️ SERVIÇOS CROSS-DOMAIN"]:::group
  COMP["Governança & Compliance<br/>(Dashboard, Matriz, LGPD)"]
  AUDIT["Trilha de Auditoria<br/>(Append-only)"]
  DASH["Indicators & Analytics<br/>(Síndico, Empresa, Agência)"]
  SEC["Segurança<br/>(Fingerprint, Anti-Fraud)"]
  g_ext["📦 🔌 INTEGRAÇÕES (Adapter Layer)"]:::group
  MP["Mercado Pago<br/>Billing"]
  SES["AWS SES / SendGrid<br/>Emails"]
  MUX["Mux / Cloudflare<br/>Video Streaming"]
  S3["AWS S3<br/>Storage"]
  FIRE["Firebase<br/>Push Notifications"]
  CEP["ViaCEP<br/>Geocoding"]
  COMP --> AUDIT
  DASH -->|consome dados| AUDIT
  SEC --> COMP
  g_cross --> g_ext
  classDef group fill:#eef,stroke:#66a,stroke-width:2px,color:#000
```

## Nodes (12)

- **[GROUP]** `g_cross` — 🛡️ SERVIÇOS CROSS-DOMAIN
- `COMP` — Governança & Compliance · (Dashboard, Matriz, LGPD)
- `AUDIT` — Trilha de Auditoria · (Append-only)
- `DASH` — Indicators & Analytics · (Síndico, Empresa, Agência)
- `SEC` — Segurança · (Fingerprint, Anti-Fraud)
- **[GROUP]** `g_ext` — 🔌 INTEGRAÇÕES (Adapter Layer)
- `MP` — Mercado Pago · Billing
- `SES` — AWS SES / SendGrid · Emails
- `MUX` — Mux / Cloudflare · Video Streaming
- `S3` — AWS S3 · Storage
- `FIRE` — Firebase · Push Notifications
- `CEP` — ViaCEP · Geocoding

## Edges (4)

- `COMP` → `AUDIT`
- `DASH` → `AUDIT` — _consome dados_
- `SEC` → `COMP`
- `g_cross` → `g_ext`

## Links

- [[_moc]] — índice dos canvas do cliente
- [[../CLAUDE]] — contrato do projeto
- [[../02-architecture/_moc]]
- [[../04-requirements/_moc]]