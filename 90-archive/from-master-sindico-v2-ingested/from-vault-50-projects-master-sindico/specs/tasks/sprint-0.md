---
title: Sprint 0 — Fundação (concluído)
version: 1.0
status: completed
sprint: 0
period: "2026-03-23 to 2026-04-06"
tags: [sprint-0, foundation, completed]
---

# Sprint 0 — Fundação

**Período**: 2026-03-23 → 2026-04-06
**Status**: ✅ **concluído** (com débitos técnicos registrados em `.kiro/STATE.md`)
**Objetivo**: Estrutura de diretórios, conexões com infra core, Gin + middlewares base, migrations fundamentais, OIDC flow implementado.

## Status final

| Tarefa | Estado | Notas |
|---|---|---|
| 0.1 Monorepo web (bun + biome + tsconfig + turbo) | ✅ | Em `web/` — 6 apps + 2 packages |
| 0.2 Estrutura Go Clean Architecture | ✅ | `cmd/api/main.go` + `internal/core/{contracts,errors}` + `internal/modules/*` + `internal/shared/*` + `pkg/*` |
| 0.3 PostgreSQL + migrations base | ✅ | enums + plans + plan_features + users + sessions + subscriptions — via goose embed.FS |
| 0.4 Redis setup | ✅ | Interface `Cache` + implementação Redis (go-redis/v9) |
| 0.5 Zitadel deploy (ECS Fargate) | ⏳ | **Deferido** para Sprint 6 — hoje usa instância cloud dev |
| 0.6 Core contracts Go | ✅ | `IModule`, `IUseCase[I,O]`, `IRepository[T,ID]`, `IUnitOfWork`, `IDomainEvent`, `AppError` |
| 0.7 Gin plugins base | ✅ | CORS, Recovery, Logger, RequestID, ErrorHandler, GinAdapter. **Rate-limit aplicado só em `/auth`** (DT-005 em STATE.md); Swagger pendente |
| 0.8 CI/CD pipeline | ⏳ | **Deferido** para Sprint 6 (infra) |
| 0.9 AWS Foundation | ⏳ | **Deferido** para Sprint 6 |
| 0.10 Flutter setup | ⏳ | **Deferido** para Sprint 1 junto com auth mobile |
| 0.11 OpenSearch cluster | ⏳ | **Deferido** para Sprint 5 |

## O que foi entregue (código em produção dev)

- Estrutura Clean Architecture completa (`infrastructure → application → domain`)
- Gin abstraído via `contracts.HTTPRouter` (módulos não importam Gin)
- PostgreSQL 18 via Docker Compose local (porta 5432)
- Redis 7 via Docker Compose local (porta 6379) com senha
- NATS JetStream via Docker Compose (porta 4222) — pronto para uso quando necessário
- Migrations: enums (`user_role`, `user_role_type`, `plan_tier`, `user_status`, `subscription_status`) + plans + plan_features + users + sessions + subscriptions
- sqlc v1.30 configurado com nullable overrides (enums nullable como `*string`)
- UnitOfWork com tx injetada via context
- OIDC PKCE via zitadel/oidc/v3 — `/auth/login`, `/auth/callback`, `/auth/logout` funcionando
- `GET /api/v1/auth/me` protegido por middleware Authenticate retornando `AuthUser`
- Health checks: `GET /health` (liveness) + `GET /health/ready` (DB + Redis)
- Fallback `Authorization: Bearer` para mobile (RFC 6750)

## Débitos técnicos criados (ver `.kiro/STATE.md`)

- **DT-001**: `planTierOrder` duplicado em 2 lugares → consolidar em `shared/types/plan_tiers.go`
- **DT-002**: `PlanTier` hardcoded `"trial"` em sync_user_use_case.go → resolver em 1.4 com JOIN real
- **DT-003**: Sem circuit breaker para Zitadel → Sprint 7
- **DT-004**: Sem timeout explícito em UnitOfWork → Sprint 2
- **DT-005**: RateLimiter só em `/auth` → incluir no Sprint 1 junto com ABAC
- **DT-006**: `.gitignore` com pattern `*.json` amplo demais → quick-win

## Decisões arquiteturais estabelecidas (T1–T15)

Ver `requirements/product-overview.md` seção "Decisões Técnicas Implementadas (T1–T15)". Todas as 15 estão **estáveis** e não voltam a debate.

## Lições aprendidas

- **Abstrair Gin cedo foi acerto**: mudança de framework é viável por causa de `contracts.HTTPRouter`. Já evitou 1 refactor quando um bug do Gin apareceu.
- **sqlc > ORM**: determinismo do código gerado + type-safety evitaram várias classes de bug que o legacy teve com Drizzle beta (ver A11 em antipatterns).
- **Fluxo OIDC é complexo**: estruturar cookies PKCE + state + nonce + callback timing pede código mais deliberado. Testes de integração recomendados (deferido para Sprint 1).

## Ponte para Sprint 1

Sprint 1 foca em **Identity & Billing**: ABAC engine (1.2), Stripe adapter (1.3), Plans + Subscriptions + webhook (1.4), Trial logic (1.5), Quotas (1.6). Ver `sprint-1.md`.

Tasks **pré-requisitos obrigatórios** do Sprint 0 (1.0.1–1.0.12) foram concluídas em 2026-04-19. Essas tasks saíram do Sprint 0 para 1 porque bloqueavam a compilação do middleware BeyondCorp.
