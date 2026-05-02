---
title: "Stack Tecnologica (canvas)"
type: canvas
tags: [canvas, obsidian, converted]
source: 50-projects/master-sindico/client-vault/00 - Visão Geral/Stack Tecnologica.canvas
converted: 2026-04-22
---

# Stack Tecnologica

> Canvas Obsidian convertido. Estrutura original: JSON com nodes + edges.

## Nodes

- **group**: Frontend
- **text**: SolidJS
- **text**: TanStack Router + Query + Form
- **text**: UnoCSS
- **text**: Axios
- **group**: Backend
- **text**: Fastify
- **text**: Drizzle ORM
- **text**: Oso + jose (JWT)
- **text**: Awilix (DI)
- **text**: Zod + drizzle-zod
- **group**: Infraestrutura
- **text**: PostgreSQL
- **text**: WebSocket (Assembleia)
- **text**: Object Storage
- **group**: Serviços Externos
- **text**: Mux / Cloudflare Stream
- **text**: Mercado Pago
- **text**: AWS SES / SendGrid
- **text**: Twilio / Zenvia
- **text**: ViaCEP
- **text**: Google / Apple OAuth

## Edges

- SOLID → FASTIFY 
- FASTIFY → PG 
- FASTIFY → WS 
- FASTIFY → S3 
- FASTIFY → MUX (Adapter Layer)
- FASTIFY → MP (Adapter Layer)
- FASTIFY → SES (Adapter Layer)
- FASTIFY → SMS (Adapter Layer)
- FASTIFY → CEP (Adapter Layer)
- FASTIFY → OAUTH (Adapter Layer)
