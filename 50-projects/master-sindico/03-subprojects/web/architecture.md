---
title: Web — Arquitetura
type: spec
tags: [subproject, web, master-sindico, v2, architecture, solidjs, tanstack, rsbuild, unocss, monorepo, feature-sliced]
source: Development/web/CLAUDE.md + AGENTS.md + apps/auth/rsbuild.config.ts + apps/auth/src/index.tsx + uno.config.ts
created: 2026-04-23
updated: 2026-04-23
aliases: [web-architecture]
---

# Web — Arquitetura

Arquitetura do sub-projeto `web/`. Complementa [[README]] detalhando monorepo Bun, tooling Rsbuild/TanStack, file-based routing, state management, API client, WebSocket, device fingerprint, acessibilidade, deploy **Cloudflare Pages** (atualizado 2026-04-27 via [[../../02-architecture/adr/0045-cloudflare-only-stack-deprecate-railway]] / [[../../STATE#D-138]] — Railway descontinuado).

Fontes primárias:
- `Development/web/CLAUDE.md` linhas 11-88 (Workspace Layout + App Architecture) e 113-128 (UnoCSS convenções).
- `Development/web/apps/auth/rsbuild.config.ts` (28 linhas).
- `Development/web/apps/auth/src/index.tsx` (86 linhas — entry canônico).
- `Development/web/apps/auth/uno.config.ts` (212 linhas — design system compartilhado).

---

## 1. Princípios

1. **Monorepo Bun** — 6 apps SolidJS + 2 packages linkados via `workspace "*"`. Sem Turborepo, sem pnpm. Apps importam **TypeScript source** direto dos packages (sem build intermediário).
2. **Feature-sliced** — cada app organiza código por feature (= BC backend quando aplicável). Features não importam entre si; comunicação via `core/api` (HTTP) ou `packages/schemas` (tipos).
3. **Context por app** — auth context, query client, onboarding progress **vivem dentro de cada app**, nunca em package compartilhado. Compartilhar apenas pure functions e schemas Zod.
4. **UI sem lógica de domínio** — componentes de `packages/ui` recebem props tipadas + emitem eventos via callbacks; sem `fetch` interno.
5. **Never trust the frontend** — validação Zod em forms para UX; backend re-valida tudo. ABAC no frontend é só pra esconder/mostrar UI; autorização efetiva sempre no backend.
6. **Mobile-first responsive** — breakpoints UnoCSS default (`sm 640 / md 768 / lg 1024 / xl 1280 / 2xl 1536`).
7. **Accessibility desde o início** — WCAG 2.1 AA obrigatório. Kobalte fornece primitives acessíveis (focus traps, ARIA, keyboard) — **não reimplementar dialog/menu/tooltip à mão**.
8. **Route-based code split** — `autoCodeSplitting: true` do TanStack Router plugin gera chunks por rota.
9. **File-based routing** — `src/routes/` reflete URL. Arquivos com prefixo `-` são ignorados (`routeFileIgnorePrefix: "-"`). Layout routes usam prefixo `_`.
10. **Zero deps de domain** — apenas DTOs vindos do OpenAPI do backend + schemas Zod espelho.

---

## 2. Monorepo Bun Workspaces

### 2.1 Manifesto raiz

```jsonc
// web/package.json
{
  "name": "mastersindico-web",
  "private": true,
  "workspaces": ["packages/*", "apps/*"],
  "scripts": {
    "dev": "bun run --filter '*' --parallel dev",
    "build": "bun run --filter '*' build",
    "check": "bun run --filter '*' check",
    "format": "bun run --filter '*' format"
  }
}
```

### 2.2 Topologia

```
web/ (root)
├── apps/auth      # port 3000 — Zitadel OIDC flows
├── apps/cms       # port 3001 — painel principal multi-persona (sub-roles em rotas internas: agência marketing, administradora condominial, empresa-síndica)
├── apps/admin     # port 3010 — NOVO D-134 2026-04-25; admin owner Master Síndico dedicado, atrás de Cloudflare Access+MFA
├── apps/lms       # port 3002 — Academia
├── apps/forum     # port 3003 — Fórum / Vizinhança
├── apps/assembly  # port 3004 — Assembleia live (WS)
├── apps/lp        # port 3005 — Landing page
├── packages/ui        # Kobalte + CVA + UnoCSS primitives
└── packages/schemas   # Zod schemas espelho
```

### 2.3 Regras de dependência (dependency rule)

```
apps/<any>  →  packages/ui   (workspace "*")
apps/<any>  →  packages/schemas  (workspace "*")
packages/ui  →  packages/schemas  (permitido)
packages/schemas  →  packages/ui  (❌ proibido — schemas não conhece UI)
apps/<A>  →  apps/<B>  (❌ proibido — apps são ilhas; comunicação via backend)
apps/<any>  →  old/  (❌ proibido — old/ é legado gitignored)
```

### 2.4 Versões alinhadas (bun.lock raiz)

Todas as 6 apps usam **versões idênticas** (verificado em `bun.lock`):

| Dep | Versão | Runtime/devDep |
|---|---|---|
| `@kobalte/core` | `^0.13.11` | runtime |
| `@tanstack/solid-router` | `^1.168.17` | runtime |
| `@tanstack/solid-router-devtools` | `^1.166.13` | runtime |
| `axios` | `^1.15.0` | runtime |
| `solid-js` | `^1.9.12` | runtime |
| `zod` | `^4.3.6` | runtime |
| `@rsbuild/core` | `^1.7.5` | dev |
| `@rsbuild/plugin-babel` | `^1.1.2` | dev |
| `@rsbuild/plugin-solid` | `^1.1.1` | dev |
| `@tanstack/router-plugin` | `^1.167.18` | dev |
| `unocss` + `@unocss/postcss` | `^66.6.8` | dev |
| `unocss-preset-primitives` | `^0.0.2-beta.2` | dev |

**Governança de versões**: bump sempre em todas as 6 apps simultaneamente (script a implementar em M1). Divergência de versão = CI fail (task FE-backlog).

---

## 3. Tooling por app

### 3.1 Rsbuild config (`rsbuild.config.ts`)

Idêntico entre apps:

```ts
import { defineConfig } from "@rsbuild/core";
import { pluginBabel } from "@rsbuild/plugin-babel";
import { pluginSolid } from "@rsbuild/plugin-solid";
import { tanstackRouter } from "@tanstack/router-plugin/rspack";

export default defineConfig({
  plugins: [
    pluginBabel({ include: /\.(?:jsx|tsx)$/ }),
    pluginSolid(),
  ],
  tools: {
    rspack: {
      plugins: [
        tanstackRouter({
          target: "solid",
          autoCodeSplitting: true,
          routesDirectory: "./src/routes",
          generatedRouteTree: "./src/routeTree.gen.ts",
          routeFileIgnorePrefix: "-",
          quoteStyle: "single",
        }),
      ],
    },
  },
});
```

Observações:
- **Rspack sob o capô** — Rsbuild = wrapper Rust-based; HMR fast, bundle menor que Webpack.
- **Babel + Solid plugin** — transpile JSX do Solid.
- **TanStack plugin no rspack**, não Rsbuild-level — plugin gera `routeTree.gen.ts` a cada mudança em `src/routes/`.
- `routeFileIgnorePrefix: "-"` — arquivos com `-` inicial são ignorados (útil para `_component.tsx` protótipos).

### 3.2 TypeScript config (raiz + herdado)

```jsonc
// web/tsconfig.json (fonte)
{
  "compilerOptions": {
    "lib": ["ESNext"],
    "target": "ESNext",
    "module": "Preserve",
    "moduleDetection": "force",
    "jsx": "react-jsx",          // Solid compatível via preset
    "allowJs": true,
    "moduleResolution": "bundler",
    "allowImportingTsExtensions": true,
    "verbatimModuleSyntax": true,
    "noEmit": true,
    "strict": true,
    "skipLibCheck": true,
    "noFallthroughCasesInSwitch": true,
    "noUncheckedIndexedAccess": true,   // arrays/objects retornam T | undefined
    "noImplicitOverride": true
  }
}
```

**Strict + `noUncheckedIndexedAccess`** é o diferencial — força tratamento explícito de `undefined` em acesso a índice.

### 3.3 Biome (raiz)

```jsonc
// web/biome.json
{
  "formatter": { "enabled": true, "indentStyle": "space", "lineWidth": 100 },
  "javascript": { "formatter": { "quoteStyle": "double" } },
  "linter": { "enabled": true, "rules": { "recommended": true } },
  "assist": { "actions": { "source": { "organizeImports": "on" } } },
  "vcs": { "enabled": true, "clientKind": "git", "useIgnoreFile": true }
}
```

Apps têm `biome.json` minimalista (`{"extends": "../../biome.json"}` equivalente — 38 bytes).

### 3.4 PostCSS + UnoCSS

```js
// apps/<app>/postcss.config.mjs
import UnoCSS from "@unocss/postcss";
export default { plugins: [UnoCSS()] };
```

`src/main.css` emite no topo:
```css
@unocss preflights;
@unocss;
:root { /* CSS vars tokens */ }
.dark, [data-kb-theme="dark"] { /* dark mode overrides */ }
```

### 3.5 Rotas como código (file-based)

```
apps/auth/src/routes/
├── __root.tsx            # root layout (Outlet + Devtools)
├── _auth.tsx             # layout pathless (prefixo _)
└── _auth/
    ├── sign-in.tsx       # URL: /sign-in (montada sob _auth layout)
    └── sign-up.tsx       # URL: /sign-up
```

Cada rota:

```tsx
import { createFileRoute } from '@tanstack/solid-router';

export const Route = createFileRoute('/_auth/sign-in')({
  component: RouteComponent,
  // loader, validateSearch, beforeLoad, pendingComponent, errorComponent...
});

function RouteComponent() {
  return <div>…</div>;
}
```

`routeTree.gen.ts` é **gerado** — proibido editar à mão. Em caso de corrupção: apagar + `bun run dev` regenera.

---

## 4. Entry point canônico

Todos os apps seguem o mesmo esqueleto (verificado em `apps/auth/src/index.tsx`):

```tsx
import { ColorModeProvider, ColorModeScript, useColorMode } from "@kobalte/core";
import { createRouter, RouterProvider } from "@tanstack/solid-router";
import * as Solid from "solid-js";
import { render } from "solid-js/web";
import { routeTree } from "./routeTree.gen";
import "./main.css";

export function getRouter() {
  const router = createRouter({
    routeTree,
    // context: { auth, queryClient },    // wire futuro
    defaultPreload: "intent",             // prefetch on hover/focus
  });
  return router;
}

declare module "@tanstack/solid-router" {
  interface Register { router: ReturnType<typeof getRouter>; }
}

const router = getRouter();

function ThemeSync() {
  const { colorMode } = useColorMode();
  Solid.createEffect(() => {
    const isDark = colorMode() === "dark";
    document.documentElement.classList.toggle("dark", isDark);
  });
  return null;
}

function InnerApp() {
  Solid.createEffect(() => {
    router.update({ /* context: { auth, queryClient } */ });
  });
  return <RouterProvider router={router} />;
}

const App = () => (
  <>
    <ThemeSync />
    <InnerApp />
    {/* <QueryClientProvider client={queryClient}><AuthProvider>… </AuthProvider></QueryClientProvider> */}
  </>
);

const root = document.getElementById("root");
if (root) {
  render(
    () => (
      <>
        <ColorModeScript />
        <ColorModeProvider>
          <App />
          {/* <Toaster /> */}
        </ColorModeProvider>
      </>
    ),
    root,
  );
}
```

**Pontos não óbvios**:
- `ColorModeScript` precede `ColorModeProvider` para prevenir FOUC em SSR ou primeira renderização; Kobalte injeta script inline detectando preferência.
- `ThemeSync` sincroniza `useColorMode()` → classe `dark` em `<html>` (UnoCSS dark-variant lê daí).
- **AuthProvider + QueryClientProvider estão comentados** em todos os apps — wire em M1 (FE-011, FE-016).
- Router `context` virá populado com `{ auth, queryClient }` para uso em `beforeLoad` / `loader` por rota.

---

## 5. Estado — SolidJS primitives

Hierarquia de estado por escopo:

| Escopo | Mecanismo | Uso |
|---|---|---|
| **UI local (componente)** | `createSignal<T>` + `createMemo` | Toggle, tab ativo, input controlado, drawer aberto |
| **Compartilhado intra-feature** | `createStore<T>` + `produce()` | Form multi-step (onboarding), wizard state, filtros compostos |
| **Transversal ao app** | **Context + `createContext<T>()`** + `useContext` | `AuthContext`, `SessionContext`, `ToastContext`, `ConfirmationDialogContext` |
| **Server state (cache + revalidação)** | **TanStack Solid Query** (`createQuery`, `createMutation`) | Todas as queries HTTP; invalidação por tag |
| **Forms** | **TanStack Solid Form** + Zod validator | Cadastro, onboarding, criar atividade, proposta, voto |
| **Derivado (computed)** | `createMemo` + `createDeferred` | Totais, filtros, mapeamentos |
| **Side effects** | `createEffect`, `onMount`, `onCleanup` | WebSocket reconnect, heartbeat, scroll-to-top |

**Proibições**:
- ❌ Redux, Zustand, Jotai, MobX, RxJS — `solid-js` + TanStack Query cobrem tudo.
- ❌ `createSignal` guardando server data — use `createQuery`.
- ❌ Mutabilidade direta em store — usar `produce()` ou `setStore` com paths.

---

## 6. API client (Axios)

Arquivo canônico por app: `src/core/api/http.ts` (a criar em M1 — FE-010).

```ts
import axios from "axios";
import { z } from "zod";

export const api = axios.create({
  baseURL: import.meta.env.PUBLIC_API_URL,
  withCredentials: true,
  timeout: 10_000,
});

api.interceptors.request.use((cfg) => {
  cfg.headers["X-Request-Id"] = crypto.randomUUID();
  return cfg;
});

api.interceptors.response.use(
  (r) => r,
  (error) => {
    if (error.response?.status === 401) {
      window.location.href = `${import.meta.env.PUBLIC_AUTH_URL}/sign-in?return=${encodeURIComponent(location.href)}`;
    }
    if (error.response?.status === 403 && error.response?.data?.error?.code === "DEVICE_MISMATCH") {
      window.location.href = "/device-mismatch";
    }
    return Promise.reject(error);
  },
);

// Wrapper tipado + validação Zod
export async function get<T>(path: string, schema: z.ZodType<T>): Promise<T> {
  const res = await api.get(path);
  const parsed = z.object({ success: z.literal(true), data: schema }).safeParse(res.data);
  if (!parsed.success) throw new Error(`Response inválido: ${path}`);
  return parsed.data.data;
}
```

**Regra**: **toda resposta backend valida contra Zod** antes de chegar à UI. Mismatch = erro logado (Sentry) + fallback UI. Defesa em profundidade — backend pode regredir.

---

## 7. WebSocket (Assembleia Live)

`apps/assembly` mantém singleton `assembly-ws.ts`:

```ts
export function connectAssembly(id: string) {
  const ws = new WebSocket(`${import.meta.env.PUBLIC_WS_URL}/ws/assembly/${id}`);
  // ReconnectingWebSocket wrapper (backoff exponencial, cap 30s)
  // Mensagens tipadas via Zod schema (messages.schema.ts)
  return ws;
}
```

- Cookie httpOnly **não acompanha WS handshake** em todos os browsers — backend aceita token em query param `?t=<signed_token>` OU valida cookie via upgrade (decisão backend).
- Mensagens: `VOTE_OPEN, VOTE_CAST, VOTE_CLOSED, SPEECH_QUEUE_JOIN, SPEECH_TURN_*, ACK_REGISTERED, ATTENDANCE_CHECKIN`.
- Reconnect automático; buffer de mensagens pendentes durante disconnect (exibe spinner "Reconectando...").

---

## 8. Device fingerprint

Coletor local (a criar M1 — FE-020):

```ts
// src/core/security/fingerprint.ts
export function computeFingerprint(): string {
  const canvas = document.createElement("canvas");
  const ctx = canvas.getContext("2d");
  // desenha texto padrão, lê dataURL, hash SHA-256
  // + navigator.userAgent + screen.width/height + timezone
  return sha256(parts.join("|"));
}
```

Enviado em header `X-Device-Fingerprint` em todas requisições. Backend consolida para IR-011 (1-device policy).

---

## 9. Dark mode

Driven por **CSS variables** em `main.css`:

```css
:root {
  --primary: oklch(0.715 0.120 84.2);   /* Gold CTA */
  --secondary: oklch(0.218 0.055 256.1); /* Navy */
  --background: oklch(0.976 0.003 264.5);
  /* ... */
}

.dark, [data-kb-theme="dark"] {
  --background: oklch(0.183 0.031 263.4);
  --card: oklch(0.218 0.055 256.1);
  /* ... */
}
```

Theme UnoCSS lê as vars: `background: "var(--background)"` etc (ver [[design-system]]).

Toggle: Kobalte `ColorModeProvider` + `useColorMode().setColorMode("dark" | "light" | "system")`. `ThemeSync` (entry) aplica classe `dark` em `<html>`.

---

## 10. Shared UI package (`packages/ui`)

Template canônico é o `Button` (`packages/ui/src/button.tsx`):

```tsx
import * as ButtonPrimitive from "@kobalte/core/button";
import type { PolymorphicProps } from "@kobalte/core/polymorphic";
import { cva, type VariantProps } from "class-variance-authority";
import { splitProps, type JSX, type ValidComponent } from "solid-js";
import { cn } from "./lib/utils";

const buttonVariants = cva(
  "inline-flex items-center justify-center gap-2 ... ring-offset-background transition-colors focus-visible:ring-2 ...",
  {
    variants: {
      variant: { default: "btn-primary", destructive: "btn-destructive", outline: "btn-outline",
                 secondary: "btn-secondary", ghost: "btn-ghost", link: "btn-link" },
      size: { default: "btn-md", sm: "btn-sm", lg: "btn-lg", icon: "btn-icon" },
    },
    defaultVariants: { variant: "default", size: "default" },
  },
);

type ButtonProps<T extends ValidComponent = "button"> = ButtonPrimitive.ButtonRootProps<T> &
  VariantProps<typeof buttonVariants> & { class?: string; children?: JSX.Element };

const Button = <T extends ValidComponent = "button">(
  props: PolymorphicProps<T, ButtonProps<T>>,
) => {
  const [local, others] = splitProps(props as ButtonProps, ["variant", "size", "class"]);
  return (
    <ButtonPrimitive.Root
      class={cn(buttonVariants({ variant: local.variant, size: local.size }), local.class)}
      {...others}
    />
  );
};

export { Button, buttonVariants };
export type { ButtonProps };
```

**Padrão obrigatório** para novo componente compartilhado:
1. **Kobalte primitive** como raiz (`Dialog.Root`, `Select.Root`, `Tabs.Root`, ...).
2. **CVA** para variant × size mapeadas para shortcuts UnoCSS (`btn-*`, `input-*`, `card-*`).
3. **`splitProps`** separando estilos do resto (passa polimórfico via `{...others}`).
4. **`PolymorphicProps<T, ...>`** para suporte à prop `as` (renderiza como `<a>`, `<Link>`, etc).
5. **`cn`** em `packages/ui/src/lib/utils.ts` — space-join mínimo, **sem `clsx` nem `tailwind-merge`** (desnecessário em UnoCSS).
6. **Re-export** de `packages/ui/src/index.ts`: `export * from "./<component>";`.
7. **Adicionar** `"ui": "*"` nas deps do app consumidor (geralmente já está).

### 10.1 Componentes previstos `packages/ui/` (M1-M3)

Atomic design leve:

```
packages/ui/src/
├── atoms/     button (pronto), input, textarea, badge, divider, spinner, avatar, icon, checkbox, radio, switch
├── molecules/ form-field, card, menu-item, tab-strip, breadcrumb, pagination, tooltip-trigger, kbd
└── organisms/ dialog (modal), drawer, toast-host, popover, data-table, combobox, date-picker, upload-dropzone, stepper, command-palette
```

---

## 11. Shared schemas package (`packages/schemas`)

Atualmente **vazio** (stub `console.log`). Estrutura prevista M1:

```
packages/schemas/src/
├── index.ts              # re-export de todos
├── auth.ts               # LoginInput, SessionSchema, DeviceFingerprint
├── profile.ts            # SindicoProfile, MoradorProfile, EmpresaProfile, AgenciaProfile, ParceiroProfile
├── onboarding.ts         # OnboardingStep*, GovernanceMarkers
├── billing.ts            # PlanTier ('trial' | 'base' | 'plus' | 'pro' — Stripe-style universal, SEM slugs)
├── institutional.ts      # CondominiumSchema, UnitSchema, MembershipSchema, TimelineEntrySchema
├── commercial.ts         # ConnectMeSchema, ProposalSchema, ClosedDealSchema, EvaluationSchema
├── content.ts            # VideoSchema, MuxUploadTicket, VisibilityFlags
├── assembly.ts           # AssemblyConfig, VoteSchema, QuorumSchema, MinutesSchema
├── api.ts                # ApiResponse<T>, ApiError, Pagination
└── common.ts             # ULID, CPF, CNPJ, Money (int centavos), FracaoIdeal (decimal string), Email, Phone
```

**Regra**: schemas espelham validação do backend. Se backend muda regex de email → schema aqui muda junto (via PR coordenado). Defesa em profundidade: UX feedback imediato + double-check da response.

**Proibições**:
- ❌ `number` para valores monetários — usa **`z.number().int()` em centavos** ou branded type `Money`.
- ❌ `number` para fração ideal — usa **`z.string().regex(/^\d+\.\d{1,6}$/)` branded `FracaoIdeal`** (nunca float — CLAUDE.md linha 270).

---

## 12. Rotas convenções (TanStack Solid Router)

### 12.1 Organização `src/routes/`

```
apps/cms/src/routes/
├── __root.tsx                               # root (header, footer, global providers)
├── index.tsx                                # / — roteamento pós-login (redireciona por persona ativa)
├── -components/                             # ignorado pelo plugin (prefix -)
│
├── _authenticated.tsx                       # guard layout (pathless _)
├── _authenticated/
│   ├── condominios/
│   │   ├── index.tsx                       # S2/M2 — meus condomínios
│   │   ├── adicionar.tsx                   # M3 — buscar por ID
│   │   └── $condominiumId/
│   │       ├── index.tsx                   # S6/M5 — home do condomínio
│   │       ├── plano-diretor.tsx           # S7-S10 / M8-M9
│   │       ├── timeline.tsx                # S11-S14 / M6-M7
│   │       ├── eventos.tsx                 # S15-S17 / M10
│   │       ├── comunicados.tsx             # S23-S25 / M11
│   │       ├── connect-me.tsx              # S18-S20
│   │       ├── compliance.tsx              # C1-C11
│   │       ├── validacoes.tsx              # S21-S22, S26
│   │       └── dashboard.tsx               # S27
│   ├── empresa/
│   │   ├── index.tsx                       # E1 painel empresarial
│   │   ├── oportunidades.tsx               # E2-E4
│   │   ├── execucao.tsx                    # E5-E9
│   │   ├── dashboard.tsx                   # E10
│   │   ├── perfil.tsx                      # E11
│   │   ├── usuarios.tsx                    # E12
│   │   └── agencia.tsx                     # E13-E16
│   ├── parceiro/                           # VZ1-VZ6
│   ├── agencia/                            # MK1-MK8
│   ├── vizinhanca/                         # VS/VE/VM
│   ├── banco-talentos/                     # TELA 01-11
│   ├── conta/                              # assinatura, dispositivos, perfil
│   └── admin/                              # ★ D-058: views admin GATED por ABAC (role super_admin/moderator/support/financial)
│       ├── tenants.tsx                     # moderação condomínios
│       ├── moderation.tsx                  # vídeos pendentes
│       ├── financial.tsx                   # Stripe reconciliation
│       ├── broadcasts.tsx                  # comunicados globais plataforma
│       ├── audit.tsx                       # audit trail read-only
│       └── _layout.tsx                     # extra guard: requireRole('admin_ms')
│
└── _public.tsx                             # público (não requer auth)
```

### 12.2 Convenções de nome

| Padrão | Significado | Exemplo |
|---|---|---|
| `_name.tsx` | Layout route (pathless, agrupa filhos) | `_authenticated.tsx` |
| `_name/...` | Filhos sob layout `_name` | `_authenticated/condominios/` |
| `$param.tsx` | Rota dinâmica | `$condominiumId.tsx` |
| `-file.tsx` | Ignorado pelo plugin (utilitários locais) | `-components/card.tsx` |
| `__root.tsx` | Root único do app | raiz |
| `index.tsx` | Index da pasta | `/` ou `/condominios/` |

### 12.3 Guards (autorização)

```tsx
// src/routes/_authenticated.tsx
import { createFileRoute, redirect } from '@tanstack/solid-router';

export const Route = createFileRoute('/_authenticated')({
  beforeLoad: async ({ context }) => {
    if (!context.auth.isAuthenticated) {
      throw redirect({ to: '/auth/login', search: { return: location.href } });
    }
  },
  component: AuthenticatedLayout,
});
```

**Admin views** (`/admin/*` em `cms`) adicionam guard extra:

```tsx
// src/routes/_authenticated/admin/_layout.tsx
beforeLoad: async ({ context }) => {
  if (!context.auth.user.roles.includes('admin_ms')) {
    throw redirect({ to: '/' }); // IDOR mitigation: sem 403 revelador, redireciona raiz
  }
},
```

Backend **sempre revalida** — frontend guard é UX only (CLAUDE.md linha 282).

---

## 13. Deploy

### 13.1 Cloudflare Pages por app (canônico após ADR-0045 / D-138)

> **Mudança 2026-04-27**: Railway descontinuado. Cada app deploya em **Cloudflare Pages** via `wrangler.toml` por app. `railway.json` deve ser removido dos 7 apps.

Estrutura por app:

```toml
# apps/auth/wrangler.toml
name = "mastersindico-auth"
compatibility_date = "2026-04-27"
pages_build_output_dir = "dist"

[env.production]
name = "mastersindico-auth"
route = { pattern = "auth.mastersindico.com.br/*", zone_name = "mastersindico.com.br" }

[env.staging]
name = "mastersindico-auth-staging"
route = { pattern = "auth.staging.mastersindico.com.br/*", zone_name = "mastersindico.com.br" }
```

Script no `package.json` por app:

```jsonc
"scripts": {
  "build": "rsbuild build",
  "deploy": "wrangler pages deploy dist --project-name=mastersindico-auth",
  "deploy:staging": "wrangler pages deploy dist --project-name=mastersindico-auth-staging --branch=staging"
}
```

CI/CD (GitHub Actions ou Cloudflare Pages Git integration):
- Push em `main` → build + `wrangler pages deploy --branch=main` (prod)
- Push em qualquer outra branch → preview deploy automático
- PR → preview deploy + comentário com URL

**DT-WEB-001 + NEW-001 RESOLVIDOS** ao migrar — não há mais `railway.json` nem `--filter=web-lp`.

### 13.2 Domínios

| App | Subdomain (prod) | Cloudflare Pages project |
|---|---|---|
| `auth` | `auth.mastersindico.com.br` | `mastersindico-auth` |
| `cms` | `app.mastersindico.com.br` | `mastersindico-cms` |
| `admin` | `admin.mastersindico.com.br` (atrás de Cloudflare Access) | `mastersindico-admin` |
| `lms` | `academia.mastersindico.com.br` | `mastersindico-lms` |
| `forum` | `vizinhanca.mastersindico.com.br` | `mastersindico-forum` |
| `assembly` | `assembleia.mastersindico.com.br` | `mastersindico-assembly` |
| `lp` | `mastersindico.com.br` (apex) | `mastersindico-lp` |

Cookie httpOnly `Domain=.mastersindico.com.br` alcança todos.

### 13.3 CORS backend

Backend (Workers ou Containers — ver [[../../02-architecture/adr/0045-cloudflare-only-stack-deprecate-railway]]) CORS allowlist por ambiente:

```
dev:   http://localhost:3000..3010
prod:  https://auth.mastersindico.com.br, https://app.mastersindico.com.br, https://admin.mastersindico.com.br, ...
```

Cookie dev: sem `Domain` attribute (browsers exigem omissão em `localhost`).

### 13.2 Domínios

| App | Subdomain (prod) |
|---|---|
| `auth` | `auth.mastersindico.com.br` |
| `cms` | `app.mastersindico.com.br` (ou `cms.`) |
| `lms` | `academia.mastersindico.com.br` |
| `forum` | `vizinhanca.mastersindico.com.br` |
| `assembly` | `assembleia.mastersindico.com.br` |
| `lp` | `mastersindico.com.br` (apex) |

Cookie httpOnly `Domain=.mastersindico.com.br` alcança todos.

### 13.3 CORS backend

Backend CORS allowlist por ambiente:

```
dev:   http://localhost:3000..3005
prod:  https://auth.mastersindico.com.br, https://app.mastersindico.com.br, ...
```

Cookie dev: sem `Domain` attribute (browsers exigem omissão em `localhost`).

---

## 14. Observability

- **Sentry Browser SDK** por app com `release` hash de commit.
- **Web Vitals** lib → envia `LCP, CLS, INP, TTFB` para `POST /api/v1/metrics/web-vitals` (body Zod-validated).
- **Console masking** em produção (`import.meta.env.PROD`): PII (CPF, CNPJ, email) nunca em logs.
- **TanStack Router Devtools**: só em dev (`<TanStackRouterDevtools />` no `__root.tsx`).

---

## 15. Testing stack

| Tipo | Ferramenta | Coverage alvo |
|---|---|---|
| Unit (components, hooks, utils) | **Vitest** + `@solidjs/testing-library` + `@testing-library/user-event` | apps/routes 75% · apps/hooks 85% · apps/lib 90% |
| Schema (Zod) | **Vitest** + `fast-check` (PBT) | `packages/schemas` 95% |
| Component library | **Vitest** + `@solidjs/testing-library` | `packages/ui` 90% |
| Integration | **Vitest** + **MSW** (mock de rede) | fluxos críticos |
| E2E | **Playwright** (M2+ via plugin `playwright`) | OIDC + Stripe Checkout + criação condomínio + Connect Me happy path |

Thresholds duros em CI (CLAUDE.md §TDD linha 247-256).

---

## 16. Ligação com artefatos canônicos

- [[../../02-architecture/clean-arch-mapping]] — camadas Clean Arch (backend define o domínio)
- [[../../02-architecture/api-design]] — contratos REST
- [[../../02-architecture/event-driven]] — eventos (frontend só consome via WS/polling)
- [[../../01-domain/bounded-contexts]] — 6 BCs + cross-domain
- [[../../04-requirements/global]] — GR/DR globais
- [[../../08-security/BEYOND_CORP]] — zero-trust posture
- [[../backend/README]] — servidor
- [[../admin-privilegios]] — D-058 transversalidade admin
- [[design-system]] — tokens + shortcuts UnoCSS + Kobalte
- [[patterns]] — SolidJS idiom + CVA + Code Calisthenics TS
- [[ui-catalog]] — 141+ telas × 6 apps
- [[security]] — defesa browser-side

## Links

- [[README]] — overview
- [[_moc]] — MOC do web
- [[../../_moc]] — MOC 03-subprojects
- [[../../CLAUDE]] — contrato agente v2
- [[../../STATE]] — decisões vivas
