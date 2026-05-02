# My Síndico — API

Backend da plataforma My Síndico. Construído com **Fastify 5** seguindo **Domain-Driven Design (DDD)** com separação estrita entre camadas e injeção de dependência via Awilix.

## Tech Stack

| Categoria | Tecnologia |
|---|---|
| **Framework** | Fastify 5 |
| **Linguagem** | TypeScript 5.9 |
| **ORM** | Drizzle ORM 1.0-beta |
| **Banco** | PostgreSQL |
| **Cache** | Redis (@fastify/redis) |
| **Auth** | JWT (jose) + Argon2 + OAuth (Arctic/Google PKCE) |
| **Autorização** | ABAC via CASL + IAuthorizationService |
| **Pagamentos** | Stripe SDK (subscriptions, webhooks, refunds) |
| **Validação** | Zod v4 + drizzle-zod |
| **DI** | Awilix + @fastify/awilix |
| **API Docs** | Swagger + Scalar API Reference |
| **Tarefas** | toad-scheduler + @fastify/schedule |
| **Build** | SWC (prod) / tsx (dev) |
| **Email** | Resend + Nodemailer (Gmail) |

## Arquitetura

```
src/
├── app.ts                     → Bootstrap (plugins, hooks, modules)
├── server.ts                  → Entrypoint (listen)
├── core/
│   ├── contracts/             → IUseCase, TransactionalUseCase, IAuthorizationService
│   └── errors/                → DomainError, BusinessError, ForbiddenError, etc.
├── infrastructure/
│   ├── database/drizzle/      → Schema, migrations, unit-of-work
│   │   └── schema/
│   │       ├── auth/          → users, accounts, sessions, verifications
│   │       ├── billing/       → plans, subscriptions, transactions, invoices, etc.
│   │       ├── profiles/      → 5 tabelas de perfil
│   │       ├── buildings/     → Condomínios
│   │       ├── categories/    → Categorias de serviço (Manual Cap. 6)
│   │       └── videos/        → Vídeos e metadata
│   ├── plugins/               → CORS, Helmet, Rate-limit, Cookie, etc.
│   └── providers/             → Cache module, Email module
├── modules/                   → 5 módulos de negócio (DDD)
│   ├── auth/                  → Autenticação, sessões, OAuth
│   ├── billing/               → Planos, subscriptions, permissões, quotas, Stripe
│   ├── profile/               → 5 tipos de perfil + CRUD
│   ├── onboarding/            → Criação de perfil pós-registro
│   └── user/                  → Entidade base User
└── shared/
    ├── hooks/                 → authenticate, authorize, requirePermission
    ├── infrastructure/auth/   → CaslAuthorizationAdapter
    ├── providers/             → ICacheProvider
    └── types/                 → ability.types, awilix.d.ts, fastify.d.ts
```

### Dependency Rule

```
Domain ← Application ← Infrastructure ← Shared
  (dependências SEMPRE apontam para dentro, nunca ao contrário)
```

Cada módulo segue a mesma estrutura DDD:

```
modules/<modulo>/
├── domain/
│   ├── entities/       → Entidades (rich domain models)
│   ├── enums/          → Enums de domínio
│   ├── repositories/   → Interfaces (Ports)
│   ├── services/       → Domain Services
│   ├── value-objects/   → Value Objects
│   └── interfaces/     → DTOs de domínio
├── application/
│   ├── use-cases/      → Orquestradores de negócio
│   └── mappers/        → Entity → DTO
├── infrastructure/
│   ├── database/       → Implementação dos repos (Drizzle)
│   ├── http/routes/    → Rotas Fastify
│   └── providers/      → Integrações externas
└── <modulo>.module.ts  → Registro de DI (Awilix)
```

## Fluxo de Autorização (ABAC)

```
Request
  → authenticate hook     (popula request.user via JWT)
  → authorize hook        (build CASL ability + registra CaslAdapter no DI)
  → requirePermission     (check grosso: plano permite esta feature?)
  → handler/controller
  → Use Case              (this.authorize() — check fino: ownership/conditions)
```

| Camada | Onde | O que checa |
|---|---|---|
| **Grossa** | `requirePermission("read", "Profile")` | O plano do user permite acessar este recurso? |
| **Fina** | `this.authorize("update", entityInstance)` | O user pode agir nesta entidade específica? (conditions/ownership) |

## Schema do Banco de Dados

| Grupo | Tabelas |
|---|---|
| **auth** | users, local_accounts, oauth_accounts, sessions, verifications |
| **billing** | plans, subscriptions, transactions, invoices, payment_methods, permissions, plan_permissions, plan_quotas, feature_usages |
| **profiles** | residents, syndics, enterprises, local_companies, marketings |
| **buildings** | buildings, units |
| **categories** | categories, service_categories |
| **videos** | videos, video_metadata, video_views, video_uploads, video_likes |

## Módulos de Negócio

Cada módulo tem seu próprio README com documentação detalhada:

| Módulo | Descrição | README |
|---|---|---|
| **auth** | Autenticação completa (email + OAuth + sessions) | `modules/auth/README.md` |
| **billing** | Planos, subscriptions, ABAC, quotas, Stripe | `modules/billing/README.md` |
| **profile** | 5 tipos de perfil com regras específicas por persona | `modules/profile/README.md` |
| **onboarding** | Fluxo de criação de perfil pós-registro | `modules/onboarding/README.md` |
| **user** | Entidade base User com role e email | `modules/user/README.md` |

## Scripts

```bash
# Desenvolvimento (hot-reload)
bun run dev

# Build de produção (SWC)
bun run build

# Type-checking
bun run typecheck

# Produção
bun run start
```

## Banco de Dados

```bash
# Gerar migration a partir do schema
bunx drizzle-kit generate

# Aplicar migrations
bunx drizzle-kit migrate

# Visualizar DB no browser
bunx drizzle-kit studio
```

## API Docs

Com o servidor rodando: [http://localhost:8000/docs](http://localhost:8000/docs)

Gerada automaticamente via Swagger + Scalar a partir dos schemas Zod.
