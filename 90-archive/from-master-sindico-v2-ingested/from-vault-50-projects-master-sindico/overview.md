---
title: Master Síndico — Overview
type: project
tags: [master-sindico, overview]
source: .kiro/steering/project-overview.md
created: 2026-04-22
project: master-sindico
---

# Master Síndico — Overview

## Propósito

Plataforma SaaS de **governança condominial** (Brasil). Conecta síndicos, moradores, empresas do mercado condominial, agências de marketing e comércio local. **Não** é ERP, app de comunicação ou sistema financeiro — é governança aplicada:

- Critério na contratação (via reputação Bronze → Diamante)
- Memória institucional auditável (timeline imutável)
- Ecossistema unificado (perfis, vídeos, cursos, contato direto)

**Invariante-chave**: identidade é única (CPF/CNPJ), mas contextos de operação são múltiplos — o mesmo usuário pode ser síndico no Condomínio X, morador no Y, e ter empresa cadastrada.

## Stack

| Camada | Tech |
|---|---|
| Backend | Go 1.26 + Gin (abstraído via `contracts.HTTPRouter`) + pgx/v5 + go-redis/v9 + zerolog + goose + sqlc |
| Auth | Zitadel (OIDC Authorization Code + PKCE) — cookie httpOnly `.mastersindico.com.br` + fallback Bearer mobile |
| Billing | Stripe via `IPaymentGateway` — SDK nunca no domínio |
| Vídeo | Mux via `IVideoProvider` |
| Busca | OpenSearch |
| SMS | Twilio via `ISMSProvider` |
| Email | AWS SES via `IEmailProvider` |
| Push | FCM via `IPushProvider` |
| Mensageria | NATS JetStream (quando necessário) |
| Infra | AWS ECS Fargate + RDS PG18 + ElastiCache Redis 7 + S3 + CloudFront + Route53 + CloudWatch |

## Arquitetura

**Clean Architecture + DDD + CQRS + Vertical Slices.** Dependências apontam para dentro: `infrastructure → application → domain`. **BeyondCorp adaptado**: Zero Trust, ABAC, tenant isolation estrito, 1 dispositivo por conta.

```
cmd/api/main.go
internal/
  core/contracts/     IModule, IUseCase, IRepository, IUnitOfWork, HTTPRouter, Context
  core/errors/        AppError + sentinels + ValidationError (sem deps de Gin)
  modules/{ctx}/
    domain/           entidades privadas, value objects, interfaces, eventos
    application/      use cases CQRS, mappers
    infrastructure/   database (pgx+sqlc), http/routes, providers externos, jobs
  server/             GinAdapter, error handler global, response helpers
  shared/
    config/           Viper + Validate()
    middleware/       BeyondCorp stack (ordem rígida)
    providers/        auth (Zitadel), cache (Redis), database (pgxpool+UoW+Migrator)
    types/            AuthUser, IUserSyncer, DTOs compartilhados
pkg/
  logger/             zerolog wrapper
  utils/              UUIDv7
  money/              int64 centavos
```

## Bounded Contexts

| Módulo | Responsabilidade | Status |
|---|---|---|
| `identity` | Usuários, sessões, autenticação Zitadel, 1-device policy | Implementado |
| `billing` | Planos, assinaturas, Stripe, trial por persona, quotas | Sprint 1 |
| `institutional` | Condomínio, unidades, timeline imutável, plano diretor | Sprint 2 |
| `commercial` | Connect Me (4 fluxos), contratos, marketplace, banco de talentos | Sprint 4 |
| `content` | Vídeos Mux (trava 90 dias), busca OpenSearch, privacidade de métricas | Sprint 5 |
| `assembly` | Assembleias live (WebSocket), votações, atas, fila de fala | Sprint 7 |

## Marcos de Entrega

| Marco | Data | Escopo |
|---|---|---|
| **Marco 1** | **2026-05-08** | Identity + Billing + Institutional base (MVP contratual) |
| **Marco 2** | 2026-06-20 | Commercial + Content |
| **Marco 3** | 2026-07-20 | Assembly + LMS + Polish |

Priorização **sempre** protege Marco 1. Decisões em aberto → ver STATE (a ingerir).

## Rotas implementadas hoje

```
GET /health              liveness (sempre 200)
GET /health/ready        readiness (PG + Redis)
GET /auth/login          inicia OIDC PKCE (pública)
GET /auth/callback       recebe code, seta cookie httpOnly (pública)
GET /auth/logout         deleta cookie + end_session Zitadel (pública)
GET /api/v1/auth/me      AuthUser do contexto (protegida)
```

Rotas OIDC ficam **fora** de `/api/v1` — são redirects de browser, não REST.

## Env vars centrais

Em `backend/.env` (gitignored):
- `DB_*` — PostgreSQL local (docker-compose)
- `REDIS_*` — Redis local (docker-compose)
- `ZITADEL_*` — instância cloud dev + service account key.json
- `STRIPE_KEY_PRIV` — `sk_test_*` em dev, `sk_live_*` só em prod
- `STRIPE_WEBHOOK_SECRET` — `whsec_*`

## Links

- [[_moc]] — MOC do projeto
- [[legacy-reference]] — TypeScript legacy como referência
- [[plugins]] — plugins Claude Code habilitados
- [[../../20-stacks/go/_moc]] — stack Go padrões
- [[../../10-knowledge/architecture/clean-architecture]] *(a criar)*
