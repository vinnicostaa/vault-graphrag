---
title: Agente 2 — pendentes web fechados (2026-04-27 noite)
type: audit
tags:
  - audit
  - master-sindico
  - '2026-04-27'
  - agente-2
  - web
  - pkce
  - lgpd
  - uno-preset
  - schemas
project: master-sindico
subproject: web
created: '2026-04-27'
updated: '2026-04-27'
---

# Agente 2 — pendentes web fechados (2026-04-27 noite)

Pass de execução autônoma sobre os pendentes que ficaram após review do
Agente 1 ([[2026-04-27-master-report]] / [[2026-04-27-agente-2-systematic-pass]]).
Foco exclusivo no `web/` (não mexer em backend/mobile).

## Dual-check feito ANTES de tocar código (Context7)

| Lib | Veredito relevante |
|---|---|
| **UnoCSS** `/unocss/unocss` | Pattern oficial é `Preset` (não `UserConfig`) — `import { Preset } from "unocss"` com `name`, `theme`, `shortcuts`. Apps fazem `defineConfig({ presets: [..., masterSindicoPreset()] })`. Fechou A-006-bis. |
| **TanStack Router** `/tanstack/router` | Campo `state` em `ToOptions` guarda em history API (não vai no URL) — alternativa canônica a search params para dados sensíveis. Fortaleceu solução A-002. |
| **Rsbuild** `/web-infra-dev/rsbuild` | Resolve tsconfig paths automaticamente; problema A-006-bis era schema do export, não bundler. Confirmou diagnóstico. |
| **SolidJS** `/solidjs/solid` | `window.location.assign()` é padrão para OIDC redirect (full reload sai do SPA). Confirmou refactor A-003. |

## Pendentes fechados (todos verde — biome + tests + build admin)

### 🔴 NEW-001 — railway.json 6 apps

7 `railway.json` reescritos com `bun run --filter=<APP> build/preview` (sem `bun x turbo`, sem `--filter=web-lp`). Deploy prod desbloqueado. Fecha **DT-WEB-001** completamente.

### 🟡 NEW-002 — tsconfig divergência

**Falso positivo confirmado**: diff entre admin e 6 apps base era apenas comentários `/* modules */ /* type checking */`. Funcional idêntico (todos `noUnusedLocals: true`, `paths`, `noUncheckedSideEffectImports: true`). Marcar reclassificado.

### 🟡 NEW-003 — ServerErrorPage requestId dangling

`apps/auth/src/lib/api.ts`: novo request interceptor gera `X-Request-Id` UUID por request via `crypto.randomUUID()`; `extractRequestId(error)` helper exportado. `apps/auth/src/index.tsx` `defaultErrorComponent` propaga ao `ServerErrorPage`. Em prod usuário vê referência real — não "—".

### 🟡 NEW-004 — biome `noUnknownProperty: off` amplo

Removido. Permanece apenas `noUnknownAtRules: off` em `**/main.css` (escopo correto para `@unocss preflights;`).

### 🟡 CVE moderate (postcss + follow-redirects)

`bun update follow-redirects postcss` → `follow-redirects 1.16.0`, `postcss 8.5.12`. Zero CVEs moderate restantes.

### 🟠 A-006-bis — uno preset compartilhado

**Solução**: `packages/ui/src/master-sindico-preset.mjs` (JS puro com JSDoc tipo) exportando `masterSindicoPreset(): Preset` no pattern oficial UnoCSS. Cada app importa via `import { masterSindicoPreset } from "ui/preset"` (workspace export `./preset`). Drift zerado, mudança de design system passa a ser **um arquivo**.

**Por que `.mjs` e não `.ts`?** `@unocss/postcss` usa `unconfig + bundle-require` que tem fricção com TS cross-workspace (`Missing semicolon` parser error). `.mjs` puro JS bypassa o issue. Funciona em Bun, Rsbuild, esbuild e Node.

**Validado**: `apps/admin` build OK (198.3 KB / 78.5 KB gzip — CSS reduziu de 66.9KB para 47.3KB com shortcuts compartilhados consolidados).

Fecha **DT-WEB-004** definitivamente.

### 🔴 A-WEB-2026-04-27-002 — OTP em search params (LGPD)

`apps/auth/src/lib/otp-flow.ts` novo: `saveOtpFlow / readOtpFlow / clearOtpFlow` com TTL 5min em `sessionStorage` (escopo aba, zera ao fechar). 7 cases de teste em `otp-flow.test.ts` cobrindo TTL/limpeza/JSON inválido/chaves separadas.

Refatorados:
- `forgot-password/code.tsx` → `saveOtpFlow("ms.recover", ...)` em vez de `navigate({ search: { code } })`
- `forgot-password/new-password.tsx` → `readOtpFlow("ms.recover")`; redireciona para `/forgot-password` se expirou
- `verify-email.tsx` → `saveOtpFlow("ms.signup-verify", ...)` (com `meta.role + meta.plan`)
- `set-password.tsx` → `readOtpFlow("ms.signup-verify")`; mantém `role + plan` em search (não-sensíveis)

Code OTP NUNCA mais transita pelo URL. Vault security.md §5 + STEERING #20 atendidos.

### 🔴 A-WEB-2026-04-27-003 — PKCE gap

`auth-context.signIn(email, password)` REMOVIDO. Nova assinatura: `signIn(options?: { idpHint?, returnTo? }): void` que faz `window.location.assign(${PUBLIC_API_URL}/auth/login?...)`. Backend já tinha `/auth/login` (gera code_verifier em cookie `gorilla/securecookie` + redirect Zitadel) e `/auth/callback` + `/api/v1/auth/exchange-code` — só faltava o frontend usar.

`sign-in.tsx` reescrito: removido form email+password local; agora mostra apenas:
- Tabs Entrar/Cadastrar
- Headline "Bem-vindo de volta"
- Botão grande "Entrar com Master Síndico" → `signIn()`
- Link "Esqueci minha senha"
- 3 botões SSO (Google/Microsoft/Apple) → `signIn({ idpHint })`

Credenciais NUNCA mais chegam ao frontend. STEERING #18 + D-001 atendidos.

### 🟠 Code Calisthenics #7 — `sign-up.tsx` 577 linhas

Quebrado em `apps/auth/src/components/signup-forms/`:

| Arquivo | Linhas |
|---|---|
| `types.ts` (FormCallbacks) | 11 |
| `resident-form.tsx` | 90 |
| `syndic-form.tsx` | 105 |
| `enterprise-form.tsx` | 127 |
| `local-company-form.tsx` | 125 |
| `index.ts` (barrel) | 5 |
| `sign-up.tsx` (wrapper + Switch) | **140** (de 577) |

Cada form abaixo do threshold ~100 linhas (vault `code-calisthenics.md` regra #7). Sign-up vira composição declarativa.

### 🟡 A-WEB-2026-04-27-021 — schemas billing + institutional

`packages/schemas/src/billing.ts` novo (170 linhas) com:
- `SubscriptionStatusSchema` (7 valores enum DB)
- `MoneyCentsSchema` (int64, sem float)
- `FeatureKeySchema` (7 features canônicas)
- `FeatureUsageSchema` + `canConsumeQuota` helper (snapshot quota — A-FASE8-FU-001)
- `SubscriptionSchema` + `PlanSchema`
- Princípios canônicos refletidos: tier universal Stripe-style D-080/D-081 (sem slug por persona); trial persona-aware GR-027; quota snapshot; money em centavos; quotaLimit `null` = ilimitado D-111

`packages/schemas/src/institutional.ts` novo (180 linhas) com:
- `CondominiumTypeSchema` (6 tipos, espelha enum `condominium_type`)
- `CepSchema` + `AddressSchema`
- `CondominiumSchema` (CNPJ opcional, totalUnits, fracaoIdealTotal)
- `UnitTypeSchema` (6 valores alinhados com migration `00211_add_unit_type_and_fracao_ideal.sql`)
- `FracaoIdealSchema` (string decimal NUMERIC(10,4), 0 < x ≤ 100, vírgula→ponto, max 4 casas)
- `UnitSchema` + helper `isUnitHabitable`
- `MembershipStatusSchema` + `MembershipEndReasonSchema` + `MembershipRoleTypeSchema`
- `MembershipSchema` + helper `isMembershipActive`
- `UserSummarySchema` (cross-context)

41 novos test cases cobrindo enums, ranges, normalizações BR (vírgula CEP), boundary, UUID v7 strict format.

### 🟡 A-015 + A-011 — `.env.example` + docs

`web/.env.example` novo: template completo com seções (Aplicação, API/WS, Zitadel OIDC, Sentry) + comentários explicativos (vault security.md §secret management). PUBLIC_* prefix doc.

`web/CLAUDE.md` atualizado: nova seção **"⚠️ `bun test` ≠ `bun run test`"** com exemplo correto vs errado, referência DT-WEB-017.

## Métricas finais (smoke completo — todos verde)

| Métrica | Antes (manhã) | Depois |
|---|---|---|
| `bun run check` | 6 errors + 14 warnings | **0 / 0 / 1 info** |
| Tests web (Vitest) | 80 | **175** (+95 novos) |
| ↳ schemas | 52 | **93** (+41 billing+institutional) |
| ↳ ui | 13 | **58** (+45 Button/Input/Card/OtpInput/Tabs) |
| ↳ auth | 15 | **24** (+9 otp-flow + ServerErrorPage prod-mode) |
| Apps web | 6 | **7** (+ admin) |
| `apps/admin` build | quebrado | **198.3 KB / 78.5 KB gzip** ✅ |
| CSS bundle (admin) | 66.9 KB | **47.3 KB** (preset compartilhado consolidou) |
| `uno.config.ts` linhas (todos apps) | ~261 × 7 = 1827 | **84 × 7 = 588** (preset extraído, drift zerado) |
| `sign-up.tsx` linhas | 577 | **140** (4 forms extraídos) |
| Findings 🔴 web pendentes | 3 | **0** ✅ |
| CVE moderate | 2 | **0** ✅ |

## Pendente após esta sessão

- **A-017** (gap testes auth para 12 telas) — incremental, deferido para próxima feature
- **A-019** (UserRoleSchema vs platform_admin) — falso positivo confirmado, sem ação
- Coverage **packages/ui 24% → ainda abaixo de 90%** mesmo com +45 tests (alguns componentes têm muitas linhas) — a fechar threshold em pass dedicado

## Vizinhos

- [[2026-04-27-agente-2-systematic-pass]] — pass anterior (Lotes 1-7)
- [[2026-04-27-consistency-pass]] — vault patches do Agente 1 (manhã)
- [[../web/.audit/MASTER-REPORT-2026-04-27]] — review do Agente 1 (filesystem)
- [[../../STATE]]
- [[../../03-subprojects/web/tasks]]
