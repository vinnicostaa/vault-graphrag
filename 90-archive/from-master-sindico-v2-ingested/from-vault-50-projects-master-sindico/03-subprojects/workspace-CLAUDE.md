---
title: CLAUDE.md — Workspace multi-repo (raiz)
type: agent-contract
tags: [master-sindico, workspace, backend, web, mobile, fullstack-legacy, claude-contract]
project: master-sindico
subproject: workspace
source: _archive/2026-04-22-specs-consolidation/CLAUDE.md
ingested: 2026-04-22
---

# CLAUDE.md — Workspace multi-repo (raiz)

> Contrato do agente para o **workspace git raiz** que agrega `backend/` (Go), `web/` (SolidJS), `app/` (Flutter) e `full-stack/` (legacy Fastify). Cobre tudo que **atravessa** sub-projetos — ordens específicas de cada um ficam em seus próprios `CLAUDE.md` ([[backend/README|backend]], [[web/CLAUDE|web]], [[mobile/CLAUDE|mobile]]).

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Workspace Layout

This directory is a **multi-project workspace**, not a single monorepo. Each sub-project is an independent git repository with its own build tooling, dependencies, and its own `CLAUDE.md` / `AGENTS.md` / `README.md`. Read the relevant sub-project doc before editing — the root file here only covers what spans them.

GitHub organization: **https://github.com/mastersindico** (3 repos públicos-privados).

| Folder | GitHub repo | Stack | Own CLAUDE.md |
|---|---|---|---|
| `backend/` | [mastersindico/backend](https://github.com/mastersindico/backend) | Go 1.26, Gin, PostgreSQL 18 (pgx + sqlc), Redis 7, Zitadel, Stripe, goose | ✅ |
| `web/` | [mastersindico/web](https://github.com/mastersindico/web) | SolidJS, TanStack Router, Rsbuild, UnoCSS, Kobalte, Axios, Zod, Biome (Bun workspaces) | — (see `AGENTS.md`) |
| `app/` | [mastersindico/app](https://github.com/mastersindico/app) | Flutter/Dart (SDK ^3.11.4) | ✅ |
| `full-stack/` | [mastersindico/platform](https://github.com/mastersindico/platform) — **Legacy / superseded** (Fastify + SolidJS + Drizzle em Turborepo) | Bun, Fastify 5, Drizzle, SolidJS | ✅ |
| `enterprise-catergories-subcategories-enum.schema.ts` | Loose reference schema at the root | — | — |

`backend.code-workspace` opens `backend`, `web`, and `full-stack` together in VS Code.

**Important:** The `full-stack/` project is an older TypeScript iteration of the same product. The active backend is now in `backend/` (Go). Do not port code or conventions between them without explicit direction — they have different architectures (Drizzle ORM vs sqlc, Awilix DI vs manual wiring, Fastify routes vs Gin abstracted via `contracts.HTTPRouter`).

## Product Context (shared across sub-projects)

**Master Síndico** is a condominium-governance SaaS for Brazil. It connects síndicos, residents, market companies, and local commerce around governance, reputation (Bronze → Diamante), and content. Personas/plans drive feature access via an ABAC matrix (role × plan_tier × action).

Language conventions that hold everywhere:
- User-facing copy, code comments, log messages, and commit text are **pt-BR**. Identifiers and filenames are English.
- Money is always integer cents (`int64` in Go). Never floats.
- IDs are UUIDv7 (see `backend/pkg/utils/id.go`).
- Auth is **Zitadel OIDC** in the Go backend (cookie for web, `Authorization: Bearer` for mobile). The legacy `full-stack/` uses its own JWT + OAuth implementation — do not cross-reference.

## Common Commands by Sub-Project

Run each from inside its own folder. These are the ones you'll reach for repeatedly; deeper details live in each sub-project's own docs.

### `backend/` (Go)
```bash
docker compose up -d                 # Postgres 18 (5432), Redis 7 (6379), NATS JetStream (4222)
go run ./cmd/api                     # API on :8000; migrations run automatically in dev
go build ./...                       # must compile clean before moving on
go vet ./...
sqlc generate                        # after editing any internal/modules/*/infrastructure/database/queries/*.sql
```
Generated sqlc code in `internal/modules/<module>/infrastructure/database/generated/` is **not hand-edited**. Migrations live in `internal/shared/providers/database/migrations/` and are embedded via `embed.FS` (goose v3). Migration numbering is partitioned by module: identity 001–099, billing 100–199, institutional 200–299, commercial 300–399, content 400–499, assembly 500–599. Never destructive in production (no `DROP TABLE`, no `DROP COLUMN`).

### `web/` (Bun workspaces, six SolidJS apps + two packages)
```bash
bun install
bun run dev                          # runs every app in parallel (--filter '*' --parallel)
bun run build
bun run check                        # biome check --write across all packages
bun run format
```
App-to-port map (hardcoded in each app's `package.json` `dev` script):

| App    | Port | Notes |
|--------|------|-------|
| auth     | 3000 | PKCE entry point |
| cms      | 3001 | |
| lms      | 3002 | |
| forum    | 3003 | |
| assembly | 3004 | |
| lp       | 3005 | Landing page |

Shared packages are `ui` and `schemas` (both built with `tsdown`, linted with Biome). Apps depend on them via workspace `"*"` — rebuild the package's `dist/` if types aren't resolving in an app. Tooling is **Biome** (not ESLint/Prettier); style is spaces + double quotes.

### `app/` (Flutter — renomeada de `mobile/` em 2026-04-21 para casar com o repo `mastersindico/app`)
```bash
flutter pub get
flutter run                          # picks connected device/emulator
flutter test
flutter analyze                      # uses analysis_options.yaml
dart run build_runner build --delete-conflicting-outputs  # sempre que mexer em @injectable/@lazySingleton
```
Projeto tem Clean Architecture + Feature First completamente fio-por-fio — auth end-to-end, DI via `get_it`+`injectable`, router via `go_router`, tema, testes com `mocktail`+`bloc_test`. Ver `app/CLAUDE.md`, `app/ARCHITECTURE.md`, `app/CODING_MANUAL.md` — autoritativos sobre qualquer menção mobile neste arquivo raiz. Auth alvo: Zitadel PKCE; tokens em `flutter_secure_storage`; backend recebe `Authorization: Bearer`.

### `full-stack/` (legacy, Turborepo + Bun)
Only touch if explicitly asked. Its `CLAUDE.md` has the full command set; the headline ones:
```bash
docker compose -f apps/api/docker-compose.yml up -d   # Postgres on 5433, Redis on 6380 (different ports from backend/!)
bun run dev                                           # api + web
cd apps/api && bunx drizzle-kit generate|migrate|push|studio
```

## Architectural Invariants (Go backend — read this before changing anything there)

The `backend/CLAUDE.md` is authoritative. The non-obvious ones worth surfacing here:

- **Dependency rule:** `infrastructure → application → domain`. `domain/` imports nothing from the other two. `shared/middleware/` never imports from `modules/` — it crosses the boundary via `shared/types/` (`AuthUser`, `IUserSyncer`, `SyncUserCommand/Output`).
- **Handlers never import Gin.** Everything goes through `contracts.HTTPRouter` / `contracts.HandlerFunc`. The Gin adapter lives in `internal/server/`.
- **Errors never get compared by string.** Use `AppError` sentinels in `core/errors/` with `errors.Is` / `errors.As`. Handlers propagate via `ctx.SetError(err)`; the global `ErrorHandler` writes the response.
- **Middleware order is fixed:** `Recovery → RequestID → Logger → CORS → ErrorHandler → RateLimiter → Authenticate → RequirePermission → Handler`. `RegisterModules()` passes modules a router with `Authenticate` already applied — OIDC routes (`/auth/login`, `/auth/callback`, `/auth/logout`) are mounted *outside* `/api/v1` because they are browser redirects, not REST endpoints.
- **sqlc nullable enums** are overridden to `*string` (see `sqlc.yaml`) so handlers don't deal with `pgtype.UUID` or `NullUserRoleType`.
- **Naming:** full words, not abbreviations (`unitOfWork`, not `uow`; `condominium`, not `condo`). One use case per file. Compile-time interface assertions: `var _ contracts.IUseCase[I, O] = (*MyUseCase)(nil)`.

## Spec-Driven Workflow

The backend (and eventually the other sub-projects) follow SDD via `.kiro/specs/master-sindico/` — `requirements.md`, `design.md`, `tasks.md`. For any non-trivial backend change, read the task + its requirement + its design before touching code, then verify with `go build ./...` and `go vet ./...` before moving on.
