---
title: Design — Índice
version: 1.0
status: stable
---

# Design — Master Síndico

Documentos de design arquitetural e técnico. Esta pasta contém **como** implementar o que foi definido em `requirements/`.

## Navegação

| Arquivo | Tópico |
|---|---|
| `architectural-principles.md` | BeyondCorp + Clean Arch + DDD + CQRS — fundamentos |
| `stack-rationale.md` | Por que cada tech foi escolhida (Go, Gin, pgx, sqlc, Zitadel, Stripe, etc) |
| `backend-structure.md` | Layout de diretórios + Module Pattern |
| `middleware-stack.md` | BeyondCorp: Recovery → RequestID → Logger → CORS → ErrorHandler → RateLimiter → Authenticate → ABAC → Handler |
| `authentication-flow.md` | OIDC PKCE + cookie multi-subdomain + 1-device |
| `data-model.md` | Schema PostgreSQL por bounded context |
| `api-design.md` | Contratos REST por módulo |
| `adapter-layer.md` | IPaymentGateway, IVideoProvider, IEmailProvider, ISMSProvider, IStorageProvider, ISearchProvider, ICEPLookup |
| `opensearch-indices.md` | 7 índices + searchable/filterable |
| `websocket-architecture.md` | Assembly real-time com gorilla/websocket + Redis pub/sub |
| `frontend-mobile.md` | SolidJS monorepo + Flutter Clean Architecture |
| [`design-assets.md`](design-assets.md) | **Novo** — URL do Figma oficial, design system (paleta, tipografia), workflow design→código |
| [`deploy-config.md`](deploy-config.md) | **Novo** — Railway (primário) + plano AWS (futuro). URL do projeto, env vars, CI/CD, backup |

## Estado atual

Durante a transição de `design.md` monolítico para estes arquivos, o monolítico ainda é **fonte canônica**. Os arquivos desta pasta vão crescer incrementalmente. A cada Sprint que toca um tema específico, o arquivo correspondente é atualizado/criado.

## Convenções

- **Schema de banco**: em `data-model.md`. Migrations reais em `internal/shared/providers/database/migrations/`.
- **Contratos HTTP**: em `api-design.md`. Swagger/OpenAPI gerado via anotações Go quando habilitado (Sprint 0.7).
- **Design decisions**: cada arquivo começa com "Por que esta abordagem" — justifica com trade-offs, não só como.
- **Referência a steering**: quando um tema é detalhado em `.kiro/steering/*.md`, link em vez de duplicar.

## Invariantes transversais ao design

- `infrastructure → application → domain` (dependency rule)
- Framework-agnostic no core: handlers usam `contracts.HandlerFunc`, não `gin.HandlerFunc`
- Providers externos isolados via interface (`IPaymentGateway`, etc)
- Value objects imutáveis, entidades com estado privado
- CQRS: Command ≠ Query, arquivos separados
- Money como `int64` centavos; fração ideal como `DECIMAL(10,6)`
- Toda query de condomínio: `WHERE condominium_id = $tenantID`
