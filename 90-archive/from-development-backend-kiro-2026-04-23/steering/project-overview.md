---
inclusion: always
---

# Master Síndico — Visão Geral do Projeto

## Propósito

Plataforma SaaS de **governança condominial** que conecta síndicos, moradores, empresas do mercado condominial, agências de marketing e comércio local em um ecossistema integrado. Não é ERP, não é app de comunicação, não é sistema financeiro — é governança aplicada: critério na contratação, memória institucional auditável, reputação baseada em evidência (Bronze → Diamante) e ecossistema unificado de perfis, vídeos, cursos e contato direto.

Identidade é única (um CPF/CNPJ), contextos de operação são múltiplos: o mesmo usuário pode ser síndico no Condomínio X, morador no Y, e ter empresa cadastrada.

## Stack

- **Backend**: Go 1.26 + Gin (abstraído via `contracts.HTTPRouter`) + pgx/v5 + go-redis/v9 + zerolog + goose + sqlc
- **Auth**: Zitadel (OIDC Authorization Code + PKCE) + cookie httpOnly `.mastersindico.com.br` + fallback `Authorization: Bearer` para mobile
- **Billing**: Stripe (via `IPaymentGateway` — SDK nunca no domínio)
- **Vídeo**: Mux (via `IVideoProvider`)
- **Busca**: OpenSearch
- **SMS**: Twilio (via `ISMSProvider`)
- **Email**: AWS SES (via `IEmailProvider`)
- **Push**: FCM (via `IPushProvider`)
- **Mensageria**: NATS JetStream (quando necessário)
- **Infra**: AWS ECS Fargate + RDS PostgreSQL 18 + ElastiCache Redis 7 + S3 + CloudFront + Route53 + CloudWatch

## Arquitetura

Clean Architecture + DDD + CQRS + Vertical Slices. Dependências apontam para dentro: `infrastructure → application → domain`. BeyondCorp adaptado: Zero Trust, ABAC, tenant isolation estrito, 1 dispositivo por conta.

```
cmd/api/main.go                    ← bootstrap, DI wire-up, graceful shutdown
internal/
  core/contracts/                  ← IModule, IUseCase, IRepository, IUnitOfWork, HTTPRouter, Context
  core/errors/                     ← AppError + sentinels + ValidationError (sem deps de Gin)
  modules/{context}/
    domain/                        ← entidades privadas, value objects, interfaces, eventos
    application/                   ← use cases CQRS, mappers
    infrastructure/                ← database (pgx+sqlc), http/routes, providers externos, jobs
  server/                          ← GinAdapter, error handler global, response helpers
  shared/
    config/                        ← Viper + Validate()
    middleware/                    ← BeyondCorp stack (ordem rígida)
    providers/                     ← auth (Zitadel), cache (Redis), database (pgxpool+UoW+Migrator)
    types/                         ← AuthUser, IUserSyncer, DTOs compartilhados
pkg/
  logger/                          ← zerolog wrapper (interface, sem SetGlobalLevel)
  utils/                           ← UUIDv7
  money/                           ← int64 centavos (planejado)
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
| **Marco 2** | 2026-06-20 | Commercial + Content (Connect Me, vídeos, marketplace) |
| **Marco 3** | 2026-07-20 | Assembly + LMS + Polish (observabilidade, performance) |

Priorização sempre protege o **Marco 1** (08/05) — ver `.kiro/STATE.md` para decisões em aberto.

## Rotas Implementadas (hoje)

```
GET  /health              ← liveness (sempre 200)
GET  /health/ready        ← readiness (PostgreSQL + Redis)
GET  /auth/login          ← inicia OIDC PKCE, redireciona para Zitadel (pública)
GET  /auth/callback       ← recebe code, seta cookie httpOnly, redireciona (pública)
GET  /auth/logout         ← deleta cookie + end_session no Zitadel (pública)
GET  /api/v1/auth/me      ← retorna AuthUser do contexto (protegida)
```

Rotas OIDC ficam **fora** de `/api/v1` — são redirects de browser, não REST.

## Fontes de Verdade

- **Decisões de produto e regras de negócio**: `.kiro/specs/master-sindico/requirements/`
- **Arquitetura, contratos de interface, data model**: `.kiro/specs/master-sindico/design/`
- **Plano de implementação (sprint + tasks)**: `.kiro/specs/master-sindico/tasks/`
- **Padrões transversais (Go, DB, security, testing, MCPs)**: `.kiro/steering/*.md`
- **Histórico do full-stack legacy**: `.kiro/specs/master-sindico/reference/`
- **Bloqueadores e decisões vivas**: `.kiro/STATE.md`

## Variáveis de Ambiente

Ver `backend/.env` para valores de desenvolvimento. **Nunca** commitar secrets. Chaves centrais:
- `DB_*` — PostgreSQL local (docker-compose)
- `REDIS_*` — Redis local (docker-compose)
- `ZITADEL_*` — instância cloud dev + service account key.json
- `STRIPE_KEY_PRIV` — `sk_test_*` em dev (`sk_live_*` apenas em prod)
- `STRIPE_WEBHOOK_SECRET` — secret do webhook Stripe (`whsec_*`)
