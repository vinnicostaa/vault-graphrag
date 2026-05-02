---
title: Backend Master Síndico — README
type: project
tags: [master-sindico, backend, go, clean-architecture]
project: master-sindico
subproject: backend
created: 2026-04-22
---

# Backend — Master Síndico

Coração do sistema. API REST + WebSocket + jobs. Escrito em **Go 1.26** seguindo Clean Architecture + DDD + CQRS.

Repo local: `Development/backend/`.

---

## Stack

| Camada | Tech | Motivo |
|---|---|---|
| Linguagem | Go 1.26 | Type safety, concorrência, binário estático |
| Framework HTTP | Gin (abstraído via `contracts.HTTPRouter`) | Maduro, performante; abstração permite troca |
| Driver PG | pgx/v5 | Performance e suporte a tipos PG nativos |
| Query tipada | sqlc v1.27 | Queries SQL de fato + tipagem Go gerada |
| Migrations | goose v3 | Simples, SQL puro, forward-only |
| Auth | Zitadel OIDC (Authorization Code + PKCE) | Standard, open-source, BeyondCorp-friendly |
| Cache / sessões aux | go-redis/v9 + Redis 7 | ABAC cache, rate limit, pub/sub |
| Log | zerolog | Structured, performance |
| Billing | Stripe via `IPaymentGateway` | Referência global, abstração pra swap |
| Vídeo | Mux via `IVideoProvider` | HLS, webhooks, escala global |
| Live | Livekit via `ILiveProvider` | WebRTC self-hosted |
| Busca | OpenSearch (prod) / PG tsvector (dev) | Full-text escalável |
| SMS | Twilio via `ISMSProvider` | Cobertura BR |
| Email | AWS SES (avaliando Resend) via `IEmailProvider` | Baixo custo + deliverability |
| Push | FCM via `IPushProvider` | Android + iOS |
| Mensageria | NATS JetStream (quando necessário) | Leve, performante |
| Observability | OpenTelemetry + Grafana + Sentry | OSS-friendly |

---

## Estrutura (Clean Arch aplicada)

```
backend/
├── cmd/api/main.go                 # entry point
├── cmd/zitadel-setup/              # bootstrap Zitadel
├── cmd/migrate/                    # runner de migrations
├── internal/
│   ├── core/
│   │   ├── contracts/              # IModule, IUseCase, IRepository, IUnitOfWork, HTTPRouter, Context
│   │   └── errors/                 # AppError + sentinels + ValidationError (sem deps de Gin)
│   ├── modules/
│   │   ├── identity/
│   │   │   ├── domain/             # entidades privadas, VOs, interfaces, events
│   │   │   ├── application/        # use cases CQRS, mappers
│   │   │   └── infrastructure/     # pgx + sqlc, http handlers, Zitadel adapter
│   │   ├── billing/                # Stripe via IPaymentGateway
│   │   ├── institutional/          # condomínio, unidades, timeline imutável
│   │   ├── commercial/             # Connect Me, contratos, reputação
│   │   ├── content/                # Mux, busca, privacidade métricas
│   │   └── assembly/               # WebSocket, Livekit, votação
│   ├── server/                     # GinAdapter, error handler global
│   └── shared/
│       ├── config/                 # Viper + Validate()
│       ├── middleware/             # BeyondCorp stack (ordem rígida)
│       ├── providers/              # auth (Zitadel), cache (Redis), database (pgxpool+UoW+Migrator)
│       └── types/                  # AuthUser, IUserSyncer, DTOs compartilhados
├── pkg/
│   ├── logger/                     # zerolog wrapper
│   ├── utils/                      # UUIDv7
│   ├── money/                      # int64 centavos
│   ├── pagination/                 # cursor-based + PBT
│   └── safecast/                   # Int32/Int16 com clamp
├── test/                           # factories, fixtures
└── .kiro/                          # specs Kiro (canônicas — versão backend)
    ├── specs/master-sindico/
    ├── steering/
    ├── STATE.md
    └── AUDIT.md                    # espelhado em vault/.../11-audit/
```

Detalhes em [[../../02-architecture/clean-arch-mapping]] e [[../../07-quality/CLEAN_ARCH]].

---

## Ordem canônica de implementação

```
1.  Migration SQL + CHECK constraints (numeração particionada por módulo)
    identity 001-099 | billing 100-199 | institutional 200-299
    commercial 300-399 | content 400-499 | assembly 500-599
2.  Query sqlc (colunas explícitas, sem SELECT *)
3.  Domain entity / VO (estado privado, factory NewX/ReconstructX)
4.  Repo interface (domain)
5.  Domain event (se cross-context)
6.  Use case CQRS (command ≠ query, arquivo separado)
7.  Mapper (entidade ↔ DTO)
8.  Repo impl (infrastructure/database)
9.  Provider externo (se SDK envolvido; infrastructure/providers)
10. Handler + rota (contracts.HTTPRouter, nunca Gin cru)
11. Wire-up no module + main
12. Testes (unit + integration + PBT quando crítico)
```

---

## Módulos e status

| Módulo | Status | Responsável |
|---|---|---|
| `identity` | ✅ Produção | Core — auth, sessão, ABAC, 1-device |
| `billing` | ✅ Produção | Planos, Stripe, trial, quotas |
| `institutional` | ✅ Produção | Condomínio, unidades, timeline, compliance |
| `commercial` | ✅ Produção | Connect Me, contratos, reputação |
| `content` | ✅ Produção | Vídeos Mux, busca, trava 90d |
| `assembly` | ✅ Produção | WebSocket, Livekit, votação |
| Cross-domain | 🟡 Em evolução | IAuditLogger (DT-010), eventos |

---

## Comandos locais

```bash
# dev
docker compose up -d              # PG18 + Redis + NATS + Zitadel + MinIO + Livekit
cp .env.example .env              # ajustar vars
go run ./cmd/api                  # sobe API em :8000

# migrations
go run ./cmd/migrate up           # aplica pendentes
go run ./cmd/migrate down         # roll-back (uso cuidadoso; forward-only em prod)
go run ./cmd/migrate status

# sqlc
sqlc generate                     # re-gera queries tipadas
git diff --exit-code internal/**/sqlc  # CI check — generated em dia

# testes
go test -race -count=1 ./...
go test -tags=integration -count=1 ./...
go test -coverprofile=coverage.out ./...
go tool cover -func=coverage.out

# security
gosec -severity=high ./...
govulncheck ./...

# stripe webhook forward (segundo terminal)
stripe listen --forward-to localhost:8000/webhooks/stripe
```

---

## Rotas principais (produção)

```
GET  /health                       liveness (sempre 200)
GET  /health/ready                 readiness (PG + Redis)
GET  /auth/login                   inicia OIDC PKCE (pública)
GET  /auth/callback                recebe code, seta cookie httpOnly
GET  /auth/logout                  deleta cookie + end_session Zitadel
GET  /api/v1/auth/me               AuthUser do contexto
POST /webhooks/stripe              HMAC verify, sem Authenticate
POST /webhooks/mux                 HMAC verify, sem Authenticate (A-022 fix)
GET  /api/v1/condominiums          ABAC + tenant-filter
POST /api/v1/condominiums
... (ver OpenAPI em 02-architecture/openapi/)
```

Rotas OIDC ficam **fora** de `/api/v1` — são redirects de browser, não REST.

---

## Env vars centrais

`.env` gitignored. Principais:

```bash
APP_ENV=development  # development | staging | production
APP_PORT=8000

DB_HOST=localhost
DB_PORT=5432
DB_USER=postgres
DB_PASSWORD=dev
DB_NAME=mastersindico

REDIS_HOST=localhost
REDIS_PORT=6379

ZITADEL_ISSUER=https://zitadel.dev.mastersindico.com.br
ZITADEL_CLIENT_ID=...
ZITADEL_KEY_PATH=./zitadel-key.json

STRIPE_KEY_PRIV=sk_test_...           # sk_live_* em prod
STRIPE_WEBHOOK_SECRET=whsec_...

MUX_TOKEN_ID=...
MUX_TOKEN_SECRET=...
MUX_SIGNING_KEY=...                    # HMAC webhook

LIVEKIT_API_KEY=...
LIVEKIT_API_SECRET=...

OTEL_EXPORTER_OTLP_ENDPOINT=http://localhost:4318

CORS_ALLOWED_ORIGINS=http://localhost:5173  # * bloqueado em prod
```

---

## Requirements específicos deste sub-projeto

- [[requirements]] — requirements que **só** o backend implementa (além dos canônicos)
- [[../../04-requirements/_moc]] — canônicos (backend é implementador principal)

## Tasks específicas

- [[tasks]] — tasks internas do backend
- [[../../05-tasks/by-sprint/_moc]] — sprints canônicos

## Patterns aplicados

- [[patterns]] — detalhamento de patterns neste sub-projeto (DDD, CQRS, UoW, Saga, ABAC engine, Strategy, Factory)
- [[../../07-quality/PATTERNS]] — visão de patterns do projeto

## Architecture específica

- [[architecture]] — diagrama + camadas + decisões específicas
- [[../../02-architecture/_moc]] — arquitetura do projeto

## Security específica

- [[security]] — security posture do backend
- [[../../08-security/_moc]] — security global

## Links

- [[../_moc]] — MOC 03-subprojects
- [[../../CLAUDE]]
- [[../../ROADMAP]]
- [[../../STATE]]
- [[../../11-audit/AUDIT]]
- [[../../02-architecture/clean-arch-mapping]]
- [[../../07-quality/PATTERNS]]
- [[../../../../20-stacks/go/_moc]]
