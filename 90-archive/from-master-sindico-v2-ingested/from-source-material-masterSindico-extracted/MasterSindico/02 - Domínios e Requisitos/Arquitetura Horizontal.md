---
title: "Arquitetura Horizontal (canvas)"
type: canvas
tags: [canvas, obsidian, converted]
source: 50-projects/master-sindico/MasterSindico_extracted/MasterSindico/02 - Domínios e Requisitos/Arquitetura Horizontal.canvas
converted: 2026-04-22
---

# Arquitetura Horizontal

> Canvas Obsidian convertido. Estrutura original: JSON com nodes + edges.

## Nodes

- **group**: 🛡️ SERVIÇOS CROSS-DOMAIN
- **text**: Governança & Compliance / (Dashboard, Matriz, LGPD)
- **text**: Trilha de Auditoria / (Append-only)
- **text**: Indicators & Analytics / (Síndico, Empresa, Agência)
- **text**: Segurança / (Fingerprint, Anti-Fraud)
- **group**: 🔌 INTEGRAÇÕES (Adapter Layer)
- **text**: Mercado Pago / Billing
- **text**: AWS SES / SendGrid / Emails
- **text**: Mux / Cloudflare / Video Streaming
- **text**: AWS S3 / Storage
- **text**: Firebase / Push Notifications
- **text**: ViaCEP / Geocoding

## Edges

- COMP → AUDIT 
- DASH → AUDIT (consome dados)
- SEC → COMP 
- g_cross → g_ext 
