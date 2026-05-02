---
title: Agente 1 — fixes apps/auth + packages (Zod 4 + http-client + isAuthenticating)
type: audit
tags:
  - audit
  - master-sindico
  - '2026-04-28'
  - agente-1
  - web
  - auth
  - zod4
  - schemas
  - ui
  - lgpd
  - pkce
project: master-sindico
subproject: web
created: '2026-04-28'
updated: '2026-04-28'
---

# Agente 1 — fixes apps/auth + packages (2026-04-28)

Sessão autônoma de revisão sistemática do trabalho do Agente 2 (fixes 2026-04-27 noite) seguida de auditoria linha-a-linha do `web/` com agentes paralelos e dual-check, e aplicação de correções nos packages e em `apps/auth/`.

Foco da sessão (ordem do João): `web/` 100% — packages PRIMEIRO (com Context7 latest), depois `apps/auth → lms → cms → assembly → forum → lp → admin`.

## Vereditos do 2º agente verificador (validação dos findings críticos do round 1)

| Finding original | Veredito 2º agente | Ação |
|---|---|---|
| 1 — Backend A-035 sqlc não regenerado | **CONFIRMADO** | Bloqueador real do quórum por fração ideal. Backend (fora foco web) — sinalizado |
| 2 — ADRs 0046-0048 ausentes no vault | **FALSO POSITIVO da premissa** + GAP REAL | Audit log Agente 2 NÃO menciona 0046-0048; alegação veio de auto-memória `now.md` enganosa. Gaps reais: 0036, 0043, 0046+. Decisão de migrar Stripe→MP/Mux→CF Stream/LiveKit→CF Calls não foi formalizada |
| 3 — MoneyCentsSchema z.number() | CONFIRMADO (severidade baixa-média) | Aplicado `.max(MAX_SAFE_INTEGER)` |
| 4 — readOtpFlow fora de onMount | **FALSO POSITIVO** (em SolidJS é idiomático CSR) | Mantido — Context7 SolidJS docs confirmam pattern |
| 5 — PUBLIC_USE_MOCKS ausente env.d.ts | CONFIRMADO | Adicionado `readonly PUBLIC_USE_MOCKS?: "true" \| "false"` |

## Bugs CRÍTICOS adicionais descobertos via tsc (não vistos pelos agentes anteriores — princípio "nunca inventar, validar com Read+build")

🔴 `apps/auth/src/routes/_auth/forgot-password/index.tsx`: `<Input>` (linha 74) e `<Button>` (linha 93) usados sem import. Build quebrava silenciosamente (a route só era exercitada em runtime). FIX: `import { Button, Input } from "ui"`.

🔴 `apps/auth/src/routes/_auth/callback.tsx:74` e `apps/auth/src/routes/_auth/logout.tsx:39`: `await auth.refresh()` — método não existe em `AuthContextType`. Nome correto é `refreshSession` (linha 72 do contexto). FIX: rename ambas chamadas.

## Lotes de correção aplicados (apps/auth + packages)

### LOTE 1 — Biome warnings pré-existentes (3 fixes)
- `apps/auth/src/__tests__/msw/handlers.ts:99` concat → template literal
- `apps/auth/src/api/fixtures/users.ts:72` `noNonNullAssertion` justificado via `biome-ignore` (índice hardcoded em array literal de tamanho fixo)
- `apps/auth/src/api/mock-client.ts` removido `MOCK_SESSIONS` import unused + `requestPasswordReset(input)` → `(_input)` (param não usado)

### LOTE 2 — Bugs CRÍTICOS de build (3 fixes)
- `forgot-password/index.tsx`: imports `{ Button, Input } from "ui"` adicionados
- `callback.tsx`, `logout.tsx`: `auth.refresh()` → `auth.refreshSession()`
- `apps/auth/src/env.d.ts`: declaração `PUBLIC_USE_MOCKS?: "true" | "false"` com JSDoc apontando vault
- `packages/schemas/src/billing.ts MoneyCentsSchema`: `.max(Number.MAX_SAFE_INTEGER, "...")` defesa IEEE-754

### LOTE 3 — Zod 4 modernização packages/schemas
- `common.ts`:
  - `EmailSchema` agora usa `z.email()` (Zod 4 top-level) em vez de `z.string().email()` legacy
  - `UuidV7Schema` agora valida ESTRITAMENTE v7: `z.uuid({ version: "v7", error: "ID inválido (UUID v7 obrigatório)" })`. Antes `z.string().uuid()` aceitava v1-v8 — gap real de defesa em profundidade
  - `CepSchema` adicionado como canônico (era duplicado em `auth.ts` local + `institutional.ts` export)
- `auth.ts`:
  - Removido `CepSchema` local; importado de `common`
  - `AuthSessionSchema.user.id`: `z.string().uuid()` → `UuidV7Schema` (reutilização)
  - `AuthSessionSchema.user.avatarUrl`: `z.string().url().optional().nullable()` → `z.url().nullish()`
  - `expiresAt`: `z.string().datetime()` → `z.iso.datetime()`
  - **Novo**: `SignUpResponseSchema` (defesa de contrato — backend signup response)
- `institutional.ts`:
  - `CepSchema` re-exportado para backward compat (canônico em `common`)
  - 8 `z.string().datetime()` → `z.iso.datetime()`
  - `z.string().url()` → `z.url()` em `UserSummarySchema.avatarUrl`
- `billing.ts`:
  - 7 `z.string().datetime()` → `z.iso.datetime()`
  - `MoneyCentsSchema`: `z.number().int()` → `z.int()` (Zod 4 alias top-level)

### LOTE 4 — packages/ui Button + apps/auth findings 🔴/🟠
- `packages/ui/src/button.tsx`:
  - Base CVA agora usa shortcut `btn-base` do preset compartilhado (mudanças design system propagam) + classes Button-específicas (gap-2, whitespace-nowrap, text-sm, [&_svg]:*)
  - Variant `info: "btn-info"` adicionado (preset já tinha shortcut, faltava expor chave TS)
  - JSDoc pt-BR documentando WHY de cada decisão
- `apps/auth/src/api/http-client.ts`:
  - Import `axios` + `SignUpResponseSchema`
  - `signUp` valida response com `SignUpResponseSchema.safeParse`; erro explícito se shape divergente (defesa em profundidade)
  - `fetchSession` distingue 401 (`null`, sessão expirada) de network failures/5xx (re-throw — TanStack Query mostra estado `isError`). Antes engolia tudo no `catch` silencioso, fazendo network blip virar logout fantasma
- `apps/auth/src/lib/api.ts`:
  - Interceptor 401 redirect global REMOVIDO (rotas públicas como `/sign-up` e `/forgot-password` saltavam para `/sign-in` indevidamente em qualquer 401 do backend)
  - Mantido apenas `403 NEW_DEVICE` redirect (1-device-policy IDN-026/D-135 precisa de redirect global)
  - `extractRequestId` agora acessa `AxiosHeaders` por chave (`error.config?.headers?.[REQUEST_ID_HEADER]`) em vez de `.get?.()` — mais robusto em testes onde `error.config` pode ser `undefined`
- `apps/auth/src/contexts/auth-context.tsx`:
  - `isAuthenticating` agora é signal próprio (`createSignal(false)`) com semântica correta — `true` ao iniciar `signIn`, `false` após mock invalidate; em prod fica `true` até o full-reload concluir. Antes refletia `signOutMutation.isPending` (bug oculto pela ausência de consumers, mas contrato público enganoso)
- `apps/auth/src/api/mock-client.ts`:
  - `loadSessionFromStorage` usa `AuthSessionSchema.safeParse` em vez de `as AuthSession` cego (paridade defensiva com `httpApiClient`)
- `apps/auth/src/routes/__root.tsx`:
  - `TanStackRouterDevtools` com guard `!import.meta.env.PROD` via `<Show>` — Rsbuild build-time elimina dev-tools do bundle de produção

## Validação após cada lote
- `bun run check` (Biome): 50 files / 0 warnings em apps/auth; 0 warnings em todos os 7 apps web (auth, cms, lms, forum, assembly, lp, admin)
- `bunx tsc --noEmit` em `apps/auth`: 0 erros
- `packages/schemas` Vitest: 93/93 tests passam
- `packages/ui` Vitest: 58/58 tests passam

## Pendentes (próximos lotes)

### 🟠 LGPD email em search params (refactor maior — lote dedicado)
- `apps/auth/src/routes/_auth/sign-up.tsx:68` `navigate({ to: "/verify-email", search: { email, role, plan } })` — email transita pela URL (LGPD art. 5º + STEERING #20 + vault `08-security/security.md §5`)
- `apps/auth/src/routes/_auth/forgot-password/index.tsx:33-46` mesmo
- `apps/auth/src/routes/_auth/verify-email.tsx:10` mesmo (search schema declara email)
- Solução: estender `otp-flow.ts` com chave `"ms.signup-init"` / `"ms.recover-init"` na etapa 1; etapas seguintes leem do sessionStorage. Manter `role + plan` em search (não-sensíveis)

### 🟡 Findings menores
- `ROLE_LABEL` duplicado em `sign-in.tsx` + `sign-up.tsx` (mover para `packages/schemas` ou util compartilhado)
- `logout.tsx:40` `toast.success` enganoso quando backend falha
- MSW handlers stale (`POST /auth/sign-in` morto pós-PKCE; sem handler para `GET /me` autenticado)
- Signup forms mensagem genérica `"Verifique os campos."` — deveria propagar issues do `parsed.error.flatten()`

### 🔵 Brand types (lote dedicado — muda tipo TS)
- `CpfSchema.brand<"Cpf">()`, `CnpjSchema.brand<"Cnpj">()`, `CepSchema.brand<"Cep">()`, `FracaoIdealSchema.brand<"FracaoIdeal">()` — Code Calisthenics #3 mandate. Audit consumers em apps antes de aplicar

### 🟣 Backend (fora foco web atual)
- A-035 `unit.sql` não atualizado com `unit_type`/`fracao_ideal`; `sqlc generate` não rodou após migration 00211 — quórum por fração ideal quebra em prod
- ADRs 0036, 0043, 0046, 0047, 0048 ausentes — decidir se faltam decisões formalizadas (Stripe→MP, Mux→CF Stream, LiveKit→CF Calls não estão acceptas, ADR-0004/0010/0011 ainda canônicos)
- `backend/internal/modules/content/infrastructure/providers/i_live_stream_provider.go` deveria estar em `domain/providers/` (DIP)

### 🔵 Auditorias web restantes
- packages/ui Atoms ausentes (FE-013): Textarea, Checkbox, Radio/RadioGroup, Switch, Badge, Spinner, Avatar, Select/Combobox, Dialog/Modal
- packages/ui sugestões cosméticas: `definePreset` factory wrapper, `TabsIndicator` opcional, migração `container.querySelector` → `screen.getByRole/getByText`
- apps/lms, cms, assembly, forum, lp, admin (ordem definida pelo João)

## Memórias permanentes salvas (sessão 2026-04-28)

Sessão capturou 13 diretrizes do João como memórias persistentes:
- `feedback_dual_check_sempre.md` — 2º agente + docs latest + vault + big-tech
- `feedback_vault_graphrag_canonico.md` — vault MCP é fonte canônica
- `feedback_findings_corrigir_imediatamente.md` — corrigir + reportar Telegram
- `feedback_bun_add_nunca_package.md` — `bun add` para deps
- `feedback_analise_completa_vault_e_projeto.md` — todas pastas (excl. node_modules/lock/.git)
- `feedback_agentes_paralelos_review.md` — 2-3 agentes background com escopos disjuntos
- `feedback_ordem_revisao_apps.md` — packages → auth → lms → cms → assembly → resto
- `feedback_nao_inventar.md` — verificar fonte real antes de afirmar
- `feedback_design_system_consistencia.md` — vault tokens > packages/ui > main.css > Figma
- `feedback_referencias_design_por_app.md` — Stripe→admin, Meet→assembly, YouTube→LMS, IG/TikTok→cms, LinkedIn→forum
- `feedback_figma_incompleto_referencias.md` — fallback big-tech adaptado a tokens vault
- `feedback_route_params_ingles_docs_ptbr.md` — paths inglês, docs/JSDoc pt-BR
- `feedback_padrao_excelencia_multidisciplinar.md` — full-stack + pentester + arquiteto + QA + dev exigente

Telegram workflow atualizado: update curto a cada etapa + relatório completo ao concluir lote.

## Vizinhos / referências

- [[2026-04-27-agente-2-pendentes-fechados]] — pass anterior do Agente 2 (PKCE, OTP, schemas, preset, sign-up split)
- [[../../STATE]] — D-138 Cloudflare-only stack (ADR-0045)
- [[../../11-audit/AUDIT]] — findings A-### globais
- [[../../03-subprojects/web/tasks]] — backlog FE-### + DT-WEB-###
- [[../../03-subprojects/web/REVIEW-2026-04-27-AGENTE-1]] — review prévia Lotes 2+3
- [[../../02-architecture/adr/0045-cloudflare-only-stack-deprecate-railway]]
