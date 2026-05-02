---
title: "Adapter Layer (canvas)"
type: canvas
tags: [canvas, obsidian, converted]
source: 50-projects/master-sindico/MasterSindico_extracted/MasterSindico/01 - Arquitetura de Sistema/Adapter Layer.canvas
converted: 2026-04-22
---

# Adapter Layer

> Canvas Obsidian convertido. Estrutura original: JSON com nodes + edges.

## Nodes

- **group**: Adapter Layer
- **group**: Serviços Externos
- **group**: Domínios Internos
- **text**: Governança
- **text**: Connect Me
- **text**: Mídia
- **text**: Billing
- **text**: IAM
- **text**: Notificações
- **text**: VideoAdapter / `IVideoProvider`
- **text**: PaymentAdapter / `IPaymentGateway`
- **text**: EmailAdapter / `IEmailProvider`
- **text**: SMSAdapter / `ISMSProvider`
- **text**: StorageAdapter / `IStorageProvider`
- **text**: CEPAdapter / `ICEPLookup`
- **text**: OAuthAdapter / `IOAuthProvider`
- **text**: Mux / Cloudflare Stream
- **text**: Mercado Pago
- **text**: AWS SES / SendGrid
- **text**: Twilio / Zenvia
- **text**: S3 / R2
- **text**: ViaCEP
- **text**: Google / Apple OAuth

## Edges

- n_media → n_av 
- n_av → n_mux 
- n_bill → n_ap 
- n_ap → n_mp 
- n_notif → n_ae 
- n_ae → n_ses 
- n_notif → n_as 
- n_as → n_twi 
- n_gov → n_ast 
- n_ast → n_s3 
- n_auth → n_ac 
- n_ac → n_cep 
- n_auth → n_ao 
- n_ao → n_oauth 
