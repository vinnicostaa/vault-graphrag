---
title: >-
  Pass sistemático Agente 2 — apps/admin scaffold + A-035 foundation + Flutter
  D-048/049 + 11 fixes web (2026-04-27)
type: audit
tags:
  - audit
  - master-sindico
  - '2026-04-27'
  - agente-2
  - lote
  - scaffold
  - be-001
  - a-035
  - d-134
  - d-048
  - d-049
  - biome
  - a11y
  - frontend
  - backend
  - flutter
project: master-sindico
created: '2026-04-27'
updated: '2026-04-27'
aliases:
  - agente-2-2026-04-27
  - lote-1-7-execucao
---

# Pass sistemático Agente 2 — 2026-04-27

> **Sessão autônoma** disparada pelo João sobre 3 repos (`backend/`, `web/`, `app/`) + vault. Identificação **Agente 2** (Agente 1 deixou findings em `web/.audit/FINDINGS-2026-04-27.md` e atualizou parcialmente o vault — patches de consistency em [[2026-04-27-consistency-pass]]).
>
> Premissa: profundidade > velocidade; dual-check obrigatório; nada via FS no vault — toda mutação vault via MCP.

## Contexto absorvido

- **Findings Agente 1**: 26 itens (3 🔴 M1 + 10 🟠 + 10 🟡 + 3 🟢) em `Development/web/.audit/FINDINGS-2026-04-27.md`. Stack web 100 % alinhada com docs oficiais (Vitest 4 + Zod 4 + TanStack 1.168 + Kobalte 0.13 + SolidJS 1.9 + MSW 2.13 + Bun 1.3). 80 testes Vitest verdes. `bun run check` falhava com 6 errors a11y + 14 warnings + 1 info.
- **Vault canônico**: STATE D-094..D-139 absorvidos; STEERING #18 PKCE / #20 PII / #25 coverage / #27 sem else; M1 firme 2026-05-18 (D-106). [[STATE]] de 2.142 linhas, [[AUDIT]] de 791 linhas, [[CLAUDE]] do projeto e [[notation-conventions]] todos lidos.
- **Filesystem 3 repos**: backend `development` ativo (10+ commits Sprint 9-10, .kiro deletado pós-migração canônica vault); web `development` (5 commits + WIP grande); app `development` (2 commits + WIP feature `assembly` adiantada).

## Dual-check antes de mutar

| Tema | Fonte 1 | Fonte 2 (dual) | Veredito |
|---|---|---|---|
| **`UserRoleSchema`** (A-WEB-019) | Schema web (6 valores) | Backend `00001_create_enums.sql` ENUM `user_role` (6 valores idênticos) | ✅ alinhado — finding **falso positivo**. `platform_admin` no vault refere-se a sub-roles ABAC dentro do app admin (super_admin/moderator/...), não substitui role Zitadel `admin`. |
| **`A-035 fração ideal range`** | functional/institutional.md `NUMERIC(5,4)` 0<x≤1.0000 | sprint-10 T-UNIT-01 `NUMERIC(10,4)` 0<x≤100 | Aplicado **sprint-10 (mais recente, percentual mais legível)** — consistente com convenção brasileira de fração ideal em % |
| **`unit_type` enum values** | backend/README.md (6 valores `residencial/comercial/vaga_garagem/...`) | backend/requirements/institutional.md VO (6 valores diferentes `apartamento/casa/...`) | Aplicado **backend/README + tasks BE-001** (6 valores `residencial/comercial/vaga_garagem/deposito/sala_comercial/loja`) — vault tem inconsistência interna; abrir DT-### para reconciliar requirements/institutional.md depois |
| **Hero carousel `createEffect+setInterval`** (A-WEB-007) | Findings Agente 1 | Código real `auth-hero-carousel.tsx` | Já estava com `onMount + onCleanup` correto antes desta sessão — verify-email + forgot-password/code é que tinham o anti-pattern (corrigidos) |

## Lotes executados

### Lote 1 — Quick wins web (10 fixes / 30 min)

Aplicados em batch (sed para 6 apps idênticos byte-a-byte + Edits específicos):

| Finding | Fix | Arquivos |
|---|---|---|
| A-WEB-005 (DT-WEB-003) | `var(--color-border)` → `var(--border)`; `var(--color-muted-foreground)` → `var(--muted-foreground)` | 6× `apps/*/src/main.css` (md5 antes/depois conferido) |
| A-WEB-004 (DT-WEB-002) | shortcut `btn-destructive` definido | 6× `apps/*/uno.config.ts` |
| A-WEB-016 | `interface ImportMetaEnv` type-safe (PUBLIC_API_URL/WS_URL/AUTH_URL/ENV/SENTRY_DSN/RELEASE_TAG) | 6× `apps/*/src/env.d.ts` |
| A-WEB-023 | "Bem vindo a master sindico" → "Bem-vindo à Master Síndico" | `auth-hero-carousel.tsx` |
| A-WEB-014 | "Bem-vindo á Master Síndico" → "Bem-vindo à Master Síndico" | `lp/index.tsx`, `cms/index.tsx` |
| A-WEB-008 | `ServerErrorPage` prod-safe (oculta msg crua em prod, mostra em dev/test; novo prop `requestId`) | `auth/components/error-pages.tsx` |
| A-WEB-009 | `AuthProvider.fetchSession` valida `AuthSessionSchema.safeParse` + console.error em divergência | `auth/contexts/auth-context.tsx` |
| A-WEB-007 | `createEffect+setInterval` → `onMount(setInterval; onCleanup(clearInterval))` | `verify-email.tsx`, `forgot-password/code.tsx` |
| A-WEB-020 | `cn` re-exportado de `packages/ui/src/index.ts` | — |
| A-WEB-022 | AGENTS.md packages/core fix + admin path documentado | `web/AGENTS.md` |
| **A-WEB-019** | **🟢 Falso positivo** — schema 6 valores casa com backend ENUM | (mantido) |

### Lote 2 — A11y biome (DoD #1) (45 min)

| Mudança | Arquivo |
|---|---|
| `tabIndex={0}` no span ativo + `tabIndex={-1}` no Link inativo (ARIA tablist correto) | `sign-in.tsx`, `sign-up.tsx` |
| `<div role="group">` → `<fieldset>` + `<legend class="sr-only">` | `sign-in.tsx` (SSO buttons) |
| `<div onMouseEnter/onFocusIn>` → `<section aria-roledescription="carousel" aria-label>` | `auth-hero-carousel.tsx` |
| `} catch (err) {` → `} catch {` (correctness/noUnusedVariables) | `verify-email.tsx`, `forgot-password/code.tsx` |
| Biome overrides root: `**/routeTree.gen.ts` excluído de lint+format; `**/main.css` com `noUnknownAtRules: off` | `web/biome.json` |
| Test novo `vi.stubEnv("PUBLIC_ENV", "prod")` valida dual-behavior do ServerErrorPage | `auth/__tests__/error-pages.test.tsx` |

**Resultado** `bun run check` (web inteiro, 9 packages): **0 errors / 0 warnings / 1 info**. Down from 6 errors + 14 warnings + 1 info.

**Tests**: 52 schemas + 13 ui + **16 auth** (1 novo) = 81 verdes (antes: 80).

### Lote 4 — apps/admin scaffold (D-134, fecha DT-WEB-023) (1h)

Estrutura completa em `web/apps/admin/`:

```
apps/admin/
├── package.json (port 3010, name "admin", scripts dev/build/check/format/preview/test)
├── biome.json (extends root)
├── tsconfig.json (strict + noUnused* + paths)
├── postcss.config.mjs (UnoCSS)
├── rsbuild.config.ts (TanStack Router plugin)
├── railway.json (sem turbo — fecha DT-WEB-001 nesta app)
├── vitest.config.ts (copia auth)
├── .gitignore
├── README.md (princípios + 5 áreas + "não fazer")
├── public/ (favicon + logo-dourado + icon-dourado)
└── src/
    ├── env.d.ts (typed)
    ├── main.css (tokens OKLCH compartilhados)
    ├── routeTree.gen.ts (stub mínimo; rsbuild dev regenera)
    ├── index.tsx (ColorModeProvider initialColorMode="dark")
    └── routes/
        ├── __root.tsx
        ├── index.tsx (redirect → /dashboard)
        ├── _admin.tsx (layout: banner Modo Admin, nav 5 áreas, footer LGPD)
        └── _admin/dashboard.tsx (placeholder 4 KPIs + roadmap)
```

Princípios aplicados:
- **3 camadas de defesa coexistem** (Cloudflare Access SSO+MFA + ABAC backend + frontend espelho com TODO documentado para fase backend `/api/v1/admin/me`).
- **Banner persistente "Modo Admin"** evita confusão com `cms` em produção.
- **`initialColorMode="dark"`** desde o scaffold (não cai no bug A-WEB-013 dos 5 apps base).
- **Sem role primária `platform_admin` no Zitadel** — explicado no README; sub-roles são atributos ABAC.

`bun install` + `bun run check`: zero erros (após adicionar `.gitignore` que biome 2.3.11 exige).

### Lote 5 — Backend A-035 / BE-001 / T-UNIT-01 (1.5h)

Foundation entregue, refactor mecânico de queries/mappers/use cases postergado.

**Value Objects criados** em `internal/modules/institutional/domain/value_objects/`:

- **`unit_type.go`** + **`unit_type_test.go`** — enum 6 valores (canônico backend/README.md), `ParseUnitType`, `IsValid`, `IsHabitable`, `String`, `DefaultUnitType=residencial`.
- **`fracao_ideal.go`** + **`fracao_ideal_test.go`** — VO baseado em `math/big.Rat` (zero float drift), parsing exato (`"3.4567"` ou `"3,4567"`), validação 0<x≤100 + max 4 casas decimais, métodos `String/Numeric/Equal/Cmp/IsZero/IsWithinCondominiumLimit/SumFracoes`, `MarshalText`/`UnmarshalText` para integração com SQL drivers/JSON.

**Migration** `00211_add_unit_type_and_fracao_ideal.sql`:

- `CREATE TYPE unit_type AS ENUM (...)` (6 valores)
- `ALTER TABLE units ADD unit_type unit_type NOT NULL DEFAULT 'residencial'` (retrofit seguro)
- `ALTER TABLE units ADD fracao_ideal NUMERIC(10,4) NULL CHECK (fracao_ideal IS NULL OR (fracao_ideal > 0 AND fracao_ideal <= 100))`
- `CREATE INDEX idx_units_condominium_unit_type ON units(condominium_id, unit_type) WHERE active`
- **Trigger** `units_check_fracao_sum_trg`: BEFORE INSERT/UPDATE → soma frações ativas do condomínio (excluindo row em edição) → se total novo > 100 → `RAISE EXCEPTION ... USING ERRCODE = '23514'` (check_violation, mapeável em handler para `422 fraction_sum_exceeded`)
- Down completo (drop trigger → function → index → columns → type)

**Entity** `unit.go` ampliada **mantendo backward compat**:

- Campos novos privados `unitType vo.UnitType`, `fracaoIdeal vo.FracaoIdeal`
- `NewUnit` (legado) → delega para `NewUnitWithDetails(... DefaultUnitType, FracaoIdeal{})` — não quebra 4 call sites
- `NewUnitWithDetails` — construtor canônico pós-BE-001 com validação `unitType.IsValid()`
- `ReconstructUnit` (legado) → delega para `ReconstructUnitFull`
- `ReconstructUnitFull` — versão com unit_type + fração para mappers pós-migration
- Métodos novos: `UnitType()`, `FracaoIdeal()`, `HasFracaoIdeal()`, `ChangeUnitType(t)`, `SetFracaoIdeal(f)`, `Deactivate()`
- Erro novo `ErrInvalidUnitType` para mapeamento via `errors.Is`

**Tests** `unit_test.go` extendido com 7 casos novos (DefaultUnitType, NewUnitWithDetails accepts/rejects, ChangeUnitType valid/invalid, SetFracaoIdeal round-trip, Deactivate, ReconstructUnitFull).

**Resultados**: `go build ./...` ✅ silent. `go vet ./...` ✅ silent. `go test ./internal/modules/institutional/domain/...` ✅ entities + services + value_objects todos OK.

**Follow-up técnico (não bloqueia M1, mecânico)**:
1. `sqlc generate` após editar `infrastructure/database/queries/unit.sql` (incluir `unit_type, fracao_ideal` em SELECT/INSERT)
2. Atualizar `infrastructure/database/unit_repository_impl.go` mapper para `ReconstructUnitFull`
3. Atualizar `application/use_cases/create_unit_use_case.go` para receber `UnitType` + `FracaoIdeal` opcionais
4. Atualizar `infrastructure/http/routes/unit_handler.go` para parsing dos campos no DTO + mapping `errcode=23514` → `422 fraction_sum_exceeded`

### Lote 6 — Flutter D-048 + D-049 (15 min)

`pubspec.yaml`:
- **D-048**: `bloc_concurrency ^0.3.0` (transformers droppable/concurrent/restartable/sequential), `hydrated_bloc ^10.1.1` (persistência local; `path_provider ^2.1.5` para Documents dir)
- **D-049**: `freezed_annotation ^3.1.0` + `json_annotation ^4.9.0` em deps, `freezed ^3.1.0` + `json_serializable ^6.9.0` em dev_deps

`fvm flutter pub get` ✅ 11 deps changed.
`fvm flutter analyze` ✅ No issues found (35.6s).
`fvm flutter test` ✅ 27 tests passed.

**Adoção incremental**: refactor `domain/entities/*` para `@freezed` e blocs para usar `concurrent`/`droppable` transformers fica para próximas features (não regredir AS-IS funcional).

### Lote 7 — vault updates

Em curso. Esta nota + atualizações em [[../../AUDIT]], [[../../STATE]], [[../../03-subprojects/web/_moc]], [[../../03-subprojects/web/tasks]], [[../../03-subprojects/web/AUDIT-2026-04-27]], [[../../03-subprojects/backend/tasks]], [[../../03-subprojects/mobile/_moc]] (relativas a `app/`).

## Dois 🔴 críticos M1 que requerem João/backend (não podem ser resolvidos só no frontend)

### A-WEB-002 — OTP code via search params (LGPD)

`forgot-password/code` → `forgot-password/new-password` passa `?code=XXXXXX` no URL; mesmo padrão em `verify-email` → `set-password`. Vaza em histórico browser, header `Referer`, logs de proxy/CDN, screenshots compartilhados.

**Mitigação proposta** (requer backend D-### nova + ADR):

1. Backend `/api/v1/auth/confirm-reset-code` retorna `reset_token` (JWT/opaco TTL 5min)
2. Frontend `forgot-password/code` armazena em `sessionStorage` ou navigation state e passa para `forgot-password/new-password` invisível na URL
3. Schema `ResetPasswordSchema` evolui: aceita `reset_token` em vez de `code`

### A-WEB-003 — PKCE gap em auth-context

`auth-context.signIn` faz `POST /api/v1/auth/sign-in { email, password }` — não é PKCE. STEERING #18 + D-001 exigem PKCE. `oidc-client-ts ^3.5.0` instalado mas não importado.

**Cenários a confirmar com backend**:

1. Backend tem 2 fluxos (local creds dev + OIDC PKCE prod) e frontend usa o errado
2. `/sign-in` é stub temporário e será removido — fluxo real é redirect para Zitadel hosted login
3. Backend implementa hybrid (ROPC + OIDC) — RFC 6749 deprecou ROPC

Fix depende da resposta do backend. Abrir D-### nova com decisão registrada.

## Métricas da sessão (Agente 2)

| Métrica | Antes | Depois |
|---|---|---|
| `bun run check` (web) | 6 errors + 14 warnings + 1 info | **0 errors + 0 warnings + 1 info** |
| Tests web | 80 (52+13+15) | **81** (52+13+16) |
| Apps web | 6 (auth+cms+lms+forum+assembly+lp) | **7** (+ admin) |
| Backend Unit entity | 4 fields | **6 fields** (+unit_type, +fracao_ideal) |
| Backend VOs institutional | 1 (Address) | **3** (+UnitType, +FracaoIdeal) |
| Backend tests institutional | (cached) | + 7 entity + 6 fracao_ideal + 4 unit_type = **17 novos** |
| Migrations backend | 39 | **40** (+ 00211) |
| Flutter deps | 16 | **22** (+6: bloc_concurrency, hydrated_bloc, freezed_*, json_*, path_provider) |
| Flutter tests | 27 | 27 (sem regressão) |
| `flutter analyze` | (não rodado) | **No issues found** |

## D-### / DT-### a fechar / abrir no STATE

Sugeridas neste pass (a registrar formalmente):

| ID | Status | Descrição |
|---|---|---|
| **DT-WEB-002** | 🟢 fechar | btn-destructive shortcut definido (Lote 1 A-WEB-004) |
| **DT-WEB-003** | 🟢 fechar | scrollbar var-name corrigida em 6 main.css (Lote 1 A-WEB-005) |
| **DT-WEB-016** | 🟢 fechar | biome a11y errors apps/auth → 0 (Lote 2) |
| **DT-WEB-019** | 🟢 fechar | AuthSession Zod validation aplicado (Lote 1 A-WEB-009) |
| **DT-WEB-021** | 🟢 fechar | ServerErrorPage prod-safe (Lote 1 A-WEB-008) |
| **DT-WEB-022** | 🟢 fechar | createEffect+setInterval refactor (Lote 1 A-WEB-007) |
| **DT-WEB-023** | 🟢 fechar | apps/admin scaffolded (Lote 4 D-134) |
| **A-035 / BE-001** | 🟢 partial close | foundation pronta (VOs + migration + entity + tests). Mecânico restante (sqlc + mapper + use case + handler) leva ~1-2h |
| **A-WEB-019** | 🟢 falso positivo | UserRoleSchema alinhado com backend ENUM canônico |
| **A-WEB-002** | 🟠 mantido aberto | requer ADR + decisão backend para `reset_token` |
| **A-WEB-003** | 🟠 mantido aberto | requer ADR + decisão backend para PKCE flow |
| **A-FASE10-DEV-003** | 🟢 fechar (parcial) | D-048+D-049 deps Flutter aplicadas; adoção incremental |

## Vizinhos

- [[../../web/.audit/FINDINGS-2026-04-27|Findings Agente 1]] (filesystem do projeto, não vault)
- [[2026-04-27-consistency-pass]] — vault patches do dia (Agente 1)
- [[../AUDIT]]
- [[../../STATE]]
- [[../../STEERING]]
- [[../../03-subprojects/web/AUDIT-2026-04-27]]
- [[../../03-subprojects/backend/tasks]]
- [[../../03-subprojects/mobile/_moc]]
- [[../../05-tasks/by-sprint/sprint-10]]
