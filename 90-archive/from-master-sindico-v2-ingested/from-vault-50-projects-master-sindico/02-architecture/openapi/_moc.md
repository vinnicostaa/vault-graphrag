---
title: MOC — OpenAPI contracts Master Síndico
type: moc
tags: [moc, master-sindico, openapi, api-contract]
project: master-sindico
created: 2026-04-22
---

# OpenAPI — Contratos de API (3.1)

Arquivos `.yaml` OpenAPI 3.1 por módulo, **gerados** (via `oapi-codegen` reverso) ou **escritos manualmente** e checados contra handlers.

## Notas canônicas (a criar por módulo)

- [[identity]] — /auth, /api/v1/auth/me, /api/v1/users
- [[billing]] — /api/v1/subscriptions, /api/v1/plans, /api/v1/feature-usage, /webhooks/stripe
- [[institutional]] — /api/v1/condominiums, /api/v1/units, /api/v1/memberships, /api/v1/timeline
- [[commercial]] — /api/v1/companies, /api/v1/connect-me, /api/v1/contracts, /api/v1/marketplace
- [[content]] — /api/v1/videos, /webhooks/mux, /api/v1/search
- [[assembly]] — /api/v1/assemblies, /api/v1/agenda-items, /api/v1/votes, /api/v1/sessions/live

## Convenções

### Paths
- Prefixo `/api/v1` (exceto `/auth/*` e `/webhooks/*` públicos)
- kebab-case em recursos
- verbos HTTP corretos (não `/api/v1/user/create` — é `POST /api/v1/users`)

### Respostas
- 2xx: payload tipado
- 4xx: `ValidationError { code, message, fields?[] }` ou `AppError { code, message }`
- 5xx: nunca detalhar interno; `{ code: "internal_error" }`

### Headers
- `Authorization: Bearer <token>` (mobile)
- `Cookie: session=<httpOnly>` (web)
- `X-Device-Fingerprint` (todos)
- `X-Request-ID` (correlação; gerar se ausente)

### Paginação
- `?limit=20&cursor=<opaque>` (cursor-based; sem offset)
- Resposta: `{ items[], next_cursor?, total? }`

### Versionamento
- Breaking change → `/api/v2` e deprecation de `/api/v1`
- Non-breaking: adicionar campos optional

## Geração e validação

```bash
# Gerar OpenAPI a partir do código (experimental):
go run ./cmd/gen-openapi > 02-architecture/openapi/current.yaml

# Validar contra handlers via teste de contrato:
spectral lint 02-architecture/openapi/*.yaml
```

## Vizinhos

- [[../_moc]]
- [[../../03-subprojects/backend/README]]
- [[../../04-requirements/_moc]]
- [[../../09-testing/_moc]] — testes de contrato
