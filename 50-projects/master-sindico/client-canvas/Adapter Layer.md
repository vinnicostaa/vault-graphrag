---
title: Adapter Layer (diagrama do cliente)
type: note
tags: [master-sindico, client-canvas, diagram, mermaid]
project: master-sindico
source: Clients/MasterSindico original .canvas (2026-03-10 a 2026-03-24) — convertido para MD+mermaid em 2026-04-24
created: 2026-04-24
---

# Adapter Layer

Diagrama original do cliente convertido de `.canvas` (Obsidian Canvas) para Mermaid. **Visão visual** dos fluxos/arquitetura; conteúdo canônico vive em [[../04-requirements/_moc]] + [[../02-architecture/_moc]].

## Diagrama

```mermaid
graph TD
  g_adapter["📦 Adapter Layer"]:::group
  g_ext["📦 Serviços Externos"]:::group
  g_internal["📦 Domínios Internos"]:::group
  n_gov["Governança"]
  n_cm["Connect Me"]
  n_media["Mídia"]
  n_bill["Billing"]
  n_auth["IAM"]
  n_notif["Notificações"]
  n_av["VideoAdapter<br/>IVideoProvider"]
  n_ap["PaymentAdapter<br/>IPaymentGateway"]
  n_ae["EmailAdapter<br/>IEmailProvider"]
  n_as["SMSAdapter<br/>ISMSProvider"]
  n_ast["StorageAdapter<br/>IStorageProvider"]
  n_ac["CEPAdapter<br/>ICEPLookup"]
  n_ao["OAuthAdapter<br/>IOAuthProvider"]
  n_mux["Mux / Cloudflare Stream"]
  n_mp["Mercado Pago"]
  n_ses["AWS SES / SendGrid"]
  n_twi["Twilio / Zenvia"]
  n_s3["S3 / R2"]
  n_cep["ViaCEP"]
  n_oauth["Google / Apple OAuth"]
  n_media --> n_av
  n_av --> n_mux
  n_bill --> n_ap
  n_ap --> n_mp
  n_notif --> n_ae
  n_ae --> n_ses
  n_notif --> n_as
  n_as --> n_twi
  n_gov --> n_ast
  n_ast --> n_s3
  n_auth --> n_ac
  n_ac --> n_cep
  n_auth --> n_ao
  n_ao --> n_oauth
  classDef group fill:#eef,stroke:#66a,stroke-width:2px,color:#000
```

## Nodes (23)

- **[GROUP]** `g_adapter` — Adapter Layer
- **[GROUP]** `g_ext` — Serviços Externos
- **[GROUP]** `g_internal` — Domínios Internos
- `n_gov` — Governança
- `n_cm` — Connect Me
- `n_media` — Mídia
- `n_bill` — Billing
- `n_auth` — IAM
- `n_notif` — Notificações
- `n_av` — VideoAdapter · `IVideoProvider`
- `n_ap` — PaymentAdapter · `IPaymentGateway`
- `n_ae` — EmailAdapter · `IEmailProvider`
- `n_as` — SMSAdapter · `ISMSProvider`
- `n_ast` — StorageAdapter · `IStorageProvider`
- `n_ac` — CEPAdapter · `ICEPLookup`
- `n_ao` — OAuthAdapter · `IOAuthProvider`
- `n_mux` — Mux / Cloudflare Stream
- `n_mp` — Mercado Pago
- `n_ses` — AWS SES / SendGrid
- `n_twi` — Twilio / Zenvia
- `n_s3` — S3 / R2
- `n_cep` — ViaCEP
- `n_oauth` — Google / Apple OAuth

## Edges (14)

- `n_media` → `n_av`
- `n_av` → `n_mux`
- `n_bill` → `n_ap`
- `n_ap` → `n_mp`
- `n_notif` → `n_ae`
- `n_ae` → `n_ses`
- `n_notif` → `n_as`
- `n_as` → `n_twi`
- `n_gov` → `n_ast`
- `n_ast` → `n_s3`
- `n_auth` → `n_ac`
- `n_ac` → `n_cep`
- `n_auth` → `n_ao`
- `n_ao` → `n_oauth`

## Links

- [[_moc]] — índice dos canvas do cliente
- [[../CLAUDE]] — contrato do projeto
- [[../02-architecture/_moc]]
- [[../04-requirements/_moc]]