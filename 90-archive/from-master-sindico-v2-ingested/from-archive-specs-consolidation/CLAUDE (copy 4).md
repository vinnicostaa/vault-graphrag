# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

MasterSindico — a condominium management SaaS platform. Turborepo monorepo with a Fastify API backend and SolidJS frontend, sharing Zod schemas via a workspace package.

## Commands

### Development
```bash
bun install                          # Install dependencies
docker compose -f apps/api/docker-compose.yml up -d  # Start PostgreSQL (port 5433) + Redis (port 6380)
bun run dev                          # Start all apps (API + Web) via Turborepo
bun run dev --filter=api             # Start only the API
bun run dev --filter=web             # Start only the Web
```

### Build & Typecheck
```bash
bun run build                        # Build all packages
bun run build:api                    # Build API only (uses SWC)
bun run build:web                    # Build Web only (uses RSBuild)
bun run check-types                  # Typecheck all packages
cd apps/api && bun run typecheck     # Typecheck API only
```

### Linting & Formatting
Biome is used for both linting and formatting (not ESLint/Prettier):
```bash
cd apps/web && bun run check         # biome check --write
cd apps/web && bun run format        # biome format --write
```
Style: spaces for indentation, double quotes for JS/TS.

### Database (Drizzle ORM)
Run from `apps/api/`:
```bash
bunx drizzle-kit generate            # Generate migration from schema changes
bunx drizzle-kit migrate             # Apply pending migrations
bunx drizzle-kit push                # Push schema directly (dev only)
bunx drizzle-kit studio              # Open Drizzle Studio GUI
```
- Schema files: `apps/api/src/infrastructure/database/drizzle/schema/`
- Migrations output: `apps/api/src/infrastructure/database/drizzle/migrations/`
- Casing convention: `snake_case` (configured in `drizzle.config.ts`)

## Architecture

### Monorepo Structure
- **`apps/api`** — Fastify 5 + TypeScript backend, compiled with SWC, runs with tsx in dev
- **`apps/web`** — SolidJS frontend with TanStack Router (file-based routing), built with RSBuild + UnoCSS
- **`packages/schemas`** — Shared Zod v4 schemas (`mastersindico-schemas`), used by both apps for validation and OpenAPI generation

### API — Modular DDD Architecture

Each feature module follows this structure:
```
modules/<feature>/
├── application/use-cases/     # Use cases (one class per operation)
├── domain/services/           # Domain logic and business rules
├── infrastructure/
│   ├── database/drizzle/repositories/  # Drizzle repository implementations
│   ├── http/routes/           # Fastify route definitions
│   ├── jobs/                  # Scheduled cron tasks (toad-scheduler)
│   └── providers/             # External service adapters
└── <feature>.module.ts        # Module registration (DI + routes + jobs)
```

**Module interface**: Every module implements `Module` with `register(app)` (DI container + routes) and optional `bootstrap(app)` (cron jobs, startup tasks).

**Dependency injection**: Awilix via `@fastify/awilix`. Lifetimes:
- `SINGLETON` — stateless services, providers (e.g., JwtTokenService, OAuthProvider)
- `SCOPED` — request-bound: repositories, use cases, domain services that depend on repos/logger

**Unit of Work**: `DrizzleUnitOfWork` wraps database transactions with `AsyncLocalStorage` for scoped transaction propagation. Repositories receive the transaction context automatically.

**Plugin registration order** (in `app.ts`): Awilix → Logger → Middleware → Redis → CORS → Helmet → Rate Limit → Swagger → Raw Body → Multipart → Cookie → Scheduler → Error Handler → Business Modules.

**Current modules**: Auth, User, Profile, Onboarding, Billing. Infrastructure modules: Drizzle, Cache (Redis), Email (Resend primary, Gmail SMTP fallback).

**Environment config**: All env vars validated with Zod at startup (`apps/api/src/shared/config/index.ts`). Invalid vars cause immediate process exit.

### Web — SolidJS with File-based Routing

**Route layouts** (TanStack Router convention):
- `_public/` — Public pages (landing, terms)
- `_auth/` — Auth pages (sign-in, sign-up, verify-email, forgot-password, reset-password)
- `_private/` — Authenticated pages (home, onboarding)

**State management**: TanStack Query for server state, SolidJS Context for auth state, TanStack Form + Zod adapter for forms.

**Styling**: UnoCSS with Wind4 preset (Tailwind-compatible utilities), Kobalte for accessible headless components, custom theme via CSS variables.

**API client**: Axios with `withCredentials: true` for cookie-based session auth.

## Key Conventions

- Language in code comments and log messages is Portuguese (pt-BR)
- API routes are versioned: `/v1/<module>/...`
- Schemas shared between frontend and backend go in `packages/schemas`, not duplicated
- Database tables use `snake_case`; TypeScript uses `camelCase`
- Auth is cookie/session-based (not token-in-header); sessions stored in DB + cached in Redis
- Node.js >=25 and Bun as package manager are required
