---
title: Web Master Síndico — README
type: spec
tags: [subproject, web, master-sindico, v2, solidjs, typescript, tanstack, rsbuild, unocss, kobalte, bun, frontend]
source: Development/web/CLAUDE.md + AGENTS.md + package.json + apps/*/package.json + uno.config.ts + main.css
created: 2026-04-23
updated: 2026-04-23
aliases: [web-readme, web-overview]
---

# Web — Master Síndico (sub-projeto `web/`)

Portal web responsivo (mobile-first) que cobre todas as personas da plataforma — síndico (`plan_tier ∈ {trial, base, plus, pro}` — ADR-0032/D-081/D-096; nomenclatura legada N1/N2/N3 abolida), morador `base` gratuito + addon Banco de Talentos opt-in (D-126 aboliu "Morador Pagante" 2026-04-25), empresa condominial (Plus/Pro), parceiro vizinhança, agência de marketing (sub-role do produto Conteúdo, rotas internas a `cms`), **admin owner Master Síndico** (D-134 2026-04-25 — **app `apps/admin` DEDICADA**, revoga D-072 que era role transversal), e visitantes (LP). Stack canônica: **SolidJS + TypeScript + TanStack Router/Query/Form + Rsbuild + UnoCSS + Kobalte + Axios + Zod + Biome**, empacotado em **Bun workspaces** com **7 apps + 2 packages** (admin dedicada após D-134; demais sub-roles em rotas internas dos produtos).

Consome a API REST do [[../backend/README|backend Go]] via cookie httpOnly `.mastersindico.com.br` (`credentials: "include"`). WebSocket opcional para Assembleia Live.

Repo alvo: `github.com/mastersindico/web`.

Fontes:
- `Development/web/CLAUDE.md` (Apr 21, 21 KB) — fonte operacional de verdade.
- `Development/web/AGENTS.md` (Apr 22, 30 KB) — guidelines cross-stack.
- `Development/web/package.json` + `Development/web/apps/{auth,cms,lms,forum,assembly,lp}/package.json`.
- `Development/web/apps/auth/{rsbuild.config.ts, uno.config.ts, src/index.tsx, src/main.css, src/routes/}`.
- `Development/web/packages/{ui,schemas}/{package.json, src/}`.
- `Development/web/bun.lock` (107 KB) — versões resolvidas.

> **Nota de reconciliação D-020 (revisão desta Fase 8)**: o legado v2 Fase 2 inferiu "Vite + Solid Router + Tailwind" (README antigo), mas o código real já está em **Rsbuild + TanStack Router + UnoCSS + Kobalte**. Decisão reverte e fixa a stack real como canônica. Registrar em [[../../STATE]] se ainda não registrado.

---

## 1. Stack confirmada (código real)

| Camada | Tech | Versão (bun.lock) | Motivo |
|---|---|---|---|
| Framework | **SolidJS** stable | `^1.9.12` | Reatividade fine-grained (signals), bundle pequeno, sem VDOM, performance nativa |
| Linguagem | **TypeScript** strict | `^5` | Type safety ponta-a-ponta; strict + `noUncheckedIndexedAccess` + `noImplicitOverride` (ver `tsconfig.json` linhas 18-23) |
| Bundler | **Rsbuild** (Rspack) | `@rsbuild/core ^1.7.5` | Rust-based; HMR rápido; Rspack nativo; plugin ecosystem (`@rsbuild/plugin-babel`, `@rsbuild/plugin-solid`) |
| Router | **TanStack Solid Router** file-based | `^1.168.17` | Type-safe routing, file-based + gerador `routeTree.gen.ts` via `@tanstack/router-plugin/rspack`; autoCodeSplitting |
| Data-fetching server state | **TanStack Solid Query** (previsto, comentado no `index.tsx` linhas 57-61) | alinhar com router | Cache, revalidação, optimistic updates; ainda não wirado — backlog FE M1 |
| Forms | **TanStack Solid Form** (previsto; backlog) | — | Declarativo + validação Zod adapter |
| Validation | **Zod** v4 | `^4.3.6` | Schema paralelo ao backend; `packages/schemas` compartilha entre apps; defesa em profundidade |
| HTTP client | **Axios** | `^1.15.0` | Interceptors 401 → redirect auth; `withCredentials: true` default; timeout 10s |
| UI primitives | **Kobalte** | `@kobalte/core ^0.13.11` | Acessibilidade WCAG 2.1 AA nativa; `ColorModeProvider` + `ColorModeScript` para dark mode; headless primitives (Dialog, Select, Tabs, Menu, Tooltip, Popover, Checkbox.Group, RadioGroup, ...) |
| Styling | **UnoCSS** + **@unocss/postcss** | `^66.6.8` | Atomic CSS on-demand; attributify (prefix `un-`, `prefixedOnly`); shortcuts como design system (`btn-primary`, `text-display-h1`, `glass-panel`, `lp-blob-*`); integração Kobalte via `unocss-preset-primitives` |
| Presets UnoCSS | `presetUno + presetWind4 + presetAttributify + presetIcons (prefix 'i-') + presetWebFonts + presetTypography + presetKobalte` | — | Ícones via prefix `i-lucide-*`; fonts Google (Manrope + Michroma + Fira Code) |
| Variants | **class-variance-authority** | `^0.7.1` (em `packages/ui`) | `cva` para variant/size em componentes Kobalte-wrapped (`<Button>` como template) |
| Lint + Format | **Biome** | `2.3.11` schema | Substitui ESLint+Prettier; `organizeImports`, `quoteStyle: "double"`, `lineWidth: 100`, `indentStyle: "space"` (ver `biome.json`) |
| Package manager | **Bun** | `1.3.10+` | Workspaces nativos, scripts paralelos (`--filter '*' --parallel`), install rápido |
| Build packages | **tsdown** | `^0.21.7` | Gera `dist/` opcional para `packages/{ui,schemas}` (apps locais importam **TS source** direto via `workspace "*"`, sem build intermediário) |
| Devtools | **@tanstack/solid-router-devtools** | `^1.166.13` | Bottom-right em dev |
| Auth (OIDC provider) | **Zitadel** (integração via backend) | — | App `auth` faz o handshake frontend do Authorization Code + PKCE; cookie httpOnly `.mastersindico.com.br` emitido pelo backend |
| Deploy | **Cloudflare Pages** (após [[../../02-architecture/adr/0045-cloudflare-only-stack-deprecate-railway]] / [[../../STATE#D-138]] em 2026-04-27) | wrangler latest | `wrangler.toml` por app; rota por subdomínio em zone `mastersindico.com.br`; CI via GitHub Actions ou Pages Git integration. Comando `wrangler pages deploy dist --project-name=mastersindico-<app>`. **Substitui** `railway.json` (Railway descontinuado). |

> Stack **NÃO inclui** (negativa explícita, contra legado Fase 2): Vite, Webpack, Turbopack, React, Radix, Tailwind, Headless UI, Redux, Zustand, clsx, tailwind-merge.

---

## 2. Workspace layout (6 apps + 2 packages)

Estrutura real em `/Development/web/`:

```
web/
├── package.json               # workspaces: ["packages/*", "apps/*"]
├── biome.json                 # fonte de verdade de formatação
├── tsconfig.json              # strict + bundler moduleResolution
├── bun.lock                   # versões travadas
├── index.ts                   # stub "Hello via Bun" (não é entry real)
├── README.md                  # stub bun init (trocar)
├── CLAUDE.md                  # 399 linhas — fonte operacional
├── AGENTS.md                  # 647 linhas — role + cross-stack guidelines
├── apps/                      # 6 apps SolidJS (portas hardcoded, ver §3)
│   ├── auth/                  # 3000 — OIDC flows + login/cadastro
│   ├── cms/                   # 3001 — painel principal multi-persona
│   ├── lms/                   # 3002 — Academia (M3 pós-lançamento)
│   ├── forum/                 # 3003 — Fórum / Vizinhança
│   ├── assembly/              # 3004 — Assembleia live (M3+)
│   └── lp/                    # 3005 — Landing page pública
├── packages/
│   ├── ui/                    # design system Kobalte + CVA + UnoCSS
│   └── schemas/               # Zod schemas compartilhados
├── old/                       # legado (ignore) — auth-context, query-context, __root.tsx antigos; gitignore aplica
│   └── docs/                  # PDFs + analyses antigas (ref histórico)
└── node_modules/              # raiz + apps
```

### 2.1 Esqueleto comum a cada app

Todas as apps compartilham o mesmo scaffold (verificado em `apps/auth/`, `apps/cms/`, ...):

```
apps/<app>/
├── package.json               # dev: rsbuild dev --port NNNN; build: rsbuild build; check/format: biome
├── biome.json                 # herda raiz
├── tsconfig.json              # herda raiz (JSX react-jsx por Solid)
├── rsbuild.config.ts          # pluginBabel + pluginSolid + tanstackRouter plugin
├── uno.config.ts              # 212 linhas idênticas — design system UnoCSS (§4)
├── postcss.config.mjs         # plugins: [UnoCSS()] via @unocss/postcss
├── railway.json               # RAILPACK builder + bun x turbo (inconsistência)
├── README.md                  # stub bun init
├── src/
│   ├── index.tsx              # entry: render + ColorModeProvider + RouterProvider + ThemeSync
│   ├── main.css               # @unocss preflights/@unocss + CSS vars tokens (light+dark) + lp-blob-*
│   ├── env.d.ts               # @rsbuild/core types
│   ├── routeTree.gen.ts       # GERADO pelo plugin — nunca editar
│   └── routes/                # file-based (TanStack)
│       ├── __root.tsx
│       ├── _auth.tsx          # layout pathless (prefixo _)
│       └── _auth/
│           ├── sign-in.tsx
│           └── sign-up.tsx
├── public/                    # favicon.png + logo-{azul,branco,dourado}.png + icon-*.png
└── .tanstack/tmp/             # cache de build do plugin (gitignore)
```

### 2.2 Packages compartilhados

| Package | `main` / `types` | Conteúdo atual | Propósito | Consumo |
|---|---|---|---|---|
| `packages/ui` | `src/index.ts` | `src/index.ts` (re-export `./button`) · `src/button.tsx` (CVA + Kobalte `ButtonPrimitive.Root` + `splitProps` + `PolymorphicProps`) · `src/lib/utils.ts` (`cn` minimalista — space-join) | Design system (atoms inicialmente; molecules/organisms a crescer) | `import { Button } from "ui"` via workspace alias `"ui": "*"` |
| `packages/schemas` | `src/index.ts` | Apenas stub `console.log("Hello via Bun!")` — **vazio** | Zod schemas espelhando validação backend (auth, profile, billing, connect-me, ...) | `import { xxx } from "schemas"` |

**Regra**: apps importam **TypeScript source** direto dos packages via workspace link (`"main": "src/index.ts"`, `"types": "src/index.ts"`, `"exports": { ".": { "types": "./src/index.ts", "import": "./src/index.ts" } }`). `tsdown build` existe para consumidores externos mas apps internos **não dependem** do `dist/`.

---

## 3. 6 apps — papel, porta, persona-alvo

| # | App | Porta dev | Papel primário | Personas-alvo | Marco | Status código |
|---|---|---|---|---|---|---|
| 1 | **`auth`** | 3000 | Zitadel OIDC — entry-point PKCE, callback, sign-in, sign-up, recuperação, OTP, device-mismatch, seleção de perfil inicial | Todas (gateway) | M1 | Scaffold + rotas `_auth/sign-in`, `_auth/sign-up` stub |
| 2 | **`cms`** | 3001 | Painel principal operacional — multi-persona: síndico (governança completa), morador (leitura + participação), empresa (oportunidades/propostas/execução), parceiro vizinhança, agência marketing (sub-role rotas internas), administradora condominial (sub-role D-114 rotas internas Assembly), empresa-síndica vinculada (sub-role D-073 rotas internas) | Síndico · Morador · Empresa · Parceiro · Agência · Administradora · Empresa-Síndica | M1 → M2 | Scaffold; `routes/index.tsx` stub |
| 7 | **`admin`** | 3010 | **NOVA app dedicada (D-134 2026-04-25)** — admin owner Master Síndico (persona `platform_admin` D-116). 5 módulos: Dashboard Financeiro, Gestão Usuários, Moderação, Configuração Plataforma, Broadcast. Atrás de Cloudflare Access + MFA obrigatório (D-117) | Admin owner Master Síndico apenas | M1 esqueleto → M3 completo | A criar |
| 3 | **`lms`** | 3002 | Academia — cursos, treinamentos, lives, fórum, biblioteca; certificados; Stripe paywall | Síndico · Empresa · Morador (parceiro vizinhança bloqueado — D-062) | M3 / pós-lançamento | Scaffold apenas |
| 4 | **`forum`** | 3003 | Fórum / Vizinhança — feed bairro, parceiros locais, cupons, palavra-chave do dia | Morador · Empresa (ver abertas) · Síndico (exclusivas condomínio) · Parceiro (publisher) | M2 | Scaffold apenas |
| 5 | **`assembly`** | 3004 | Assembleia ao vivo — pré-pauta, check-in, voto fracional/simples/fornecedor/eleição, telão 2 áreas, fila de fala, ata, score transparência | Síndico (presidir) · Morador (vota) · Empresa (fornecedor em votação) | M3+ | Scaffold apenas |
| 6 | **`lp`** | 3005 | Landing page pública — hero com blob system (navy → gold), pricing, docs, blog; SEO | Público | M2 | Scaffold + hero blobs + CTA "Continuar" |

**Portas hardcoded** em cada `apps/<app>/package.json` script `dev` (`rsbuild dev --port NNNN`). Mudança exige atualizar CORS backend + redirect URIs Zitadel (CLAUDE.md linhas 24-26).

**⚠️ ATUALIZAÇÃO 2026-04-25 (D-134)**: Admin owner Master Síndico **AGORA É app dedicada** (`apps/admin`) — revoga decisão anterior D-072 que tratava admin como role transversal embutida em `cms`. Trigger da revogação: viabilização Cloudflare Access SSO+MFA (D-117) + benchmark big tech (Stripe/AWS/Vercel — D-113). **Outros sub-roles** (administradora condominial D-114, agência marketing D-102, empresa-síndica D-073, auditor assembly D-132) **continuam em rotas internas dos produtos** — não criam apps separadas. Ver [[../admin-privilegios]] (atualizar) e [[security#5-admin-isolation]].

---

## 4. Contratos com backend

### 4.1 REST — cookie-based

```typescript
import axios from "axios";

const api = axios.create({
  baseURL: import.meta.env.PUBLIC_API_URL, // http://localhost:8000 (dev), https://api.mastersindico.com.br (prod)
  withCredentials: true,                   // cookie httpOnly .mastersindico.com.br vai automaticamente
  timeout: 10_000,
});

api.interceptors.response.use(
  (r) => r,
  (error) => {
    if (error.response?.status === 401) {
      window.location.href = "/auth/login"; // redirect pro app auth (port 3000 em dev, auth.mastersindico.com.br em prod)
    }
    return Promise.reject(error);
  },
);
```

Response format: `ApiResponse<T> = {success: true, data: T}` ou `ApiError = {success: false, error: {code, message}}`. **Validar response com Zod** (via `packages/schemas`) antes de usar em UI — backend pode ter bug (defesa em profundidade).

### 4.2 WebSocket — assembleia

`new WebSocket('wss://api.mastersindico.com.br/ws/assembly/:id')` em `apps/assembly`. Cookie httpOnly **não** acompanha WS; backend valida via query param signed ou via upgrade handshake com cookie (decisão backend). Protocolo de mensagens: `VOTE_OPEN`, `VOTE_CAST`, `VOTE_CLOSED`, `SPEECH_QUEUE_JOIN`, `SPEECH_TURN_*`, `ACK_REGISTERED`, `ATTENDANCE_CHECKIN` (detalhe em [[../backend/README#42]]).

### 4.3 Auth flow (Zitadel OIDC + PKCE)

1. Usuário acessa URL protegida (`cms.mastersindico.com.br/...`) sem cookie → SPA redirect para `auth.mastersindico.com.br/sign-in`.
2. App `auth` inicia fluxo PKCE → redirect backend `/api/v1/auth/login`.
3. Backend gera `code_verifier + code_challenge`, redireciona Zitadel.
4. Zitadel autentica → callback backend `/api/v1/auth/callback?code=...`.
5. Backend troca `code` por tokens, valida `id_token`, cria sessão (1-device check), seta cookie httpOnly `.mastersindico.com.br`, redireciona SPA de origem.
6. SPA chama `GET /api/v1/auth/me` → popula session store; routes guards liberam.
7. Logout: `POST /api/v1/auth/logout` → backend `end_session` Zitadel + clear cookie + `queryClient.clear()` frontend.

Device mismatch (login em 2º device invalida 1º): backend retorna 403 com código `DEVICE_MISMATCH`; SPA mostra tela SYS5 `/device-mismatch`.

### 4.4 WebFetch MCP / Figma

Implementação de telas segue fluxo `Discuss → Plan → Execute → Verify → Ship` (CLAUDE.md §"Workflow SDD"). Antes de codar: `/implement-design <frame>` via plugin Figma, puxar do URL oficial `https://www.figma.com/design/pxJkmn8rzezKnCYXcA7lyL/Master-sindico-APP---UX`.

---

## 5. Contratos entre apps (6 apps, 1 cookie)

- **Cookie compartilhado**: todas as apps rodam sob `*.mastersindico.com.br` → cookie httpOnly `Domain=.mastersindico.com.br` vai para todas automaticamente.
- **Navegação cross-app**: full page redirect (`window.location.href = "..."`), NÃO client-side routing. Ex: `cms.mastersindico.com.br/condominios/XYZ` → deslogado → redirect `auth.mastersindico.com.br/sign-in?return=...`.
- **Dev local**: CORS + cookie sem `Domain` attribute; cada app em `localhost:30NN` com backend em `localhost:8000`; backend CORS allowlist inclui `http://localhost:3000..3005`.
- **Session store local**: cada app mantém seu `session.context.tsx` + `useSession()` hook (regra herdada: "Auth context vive em cada app, não no core" — CLAUDE.md linha 84).
- **Shared state entre apps**: não existe — cada app é isolado. Estado compartilhado = backend (cookie + `/api/v1/auth/me` + `/api/v1/tenant/:id/...`).

---

## 6. Comandos essenciais

Raiz `web/`:

```bash
bun install                           # instala todos os workspaces
bun run dev                           # --filter '*' --parallel — todos os apps nas suas portas
bun run build                         # --filter '*' — build produção de todos os apps + packages
bun run check                         # biome check --write em todos os packages
bun run format                        # biome format --write em todos os packages
```

Por app:

```bash
cd apps/<name>
bun run dev                           # rsbuild dev --port NNNN
bun run build                         # rsbuild build
bun run check                         # biome check --write
bun run format                        # biome format --write
bun run preview                       # rsbuild preview (build + serve)
```

---

## 7. Personas ↔ apps (heatmap)

| Persona / role | `auth` | `cms` | `lms` | `forum` | `assembly` | `lp` |
|---|---|---|---|---|---|---|
| Síndico N1/N2/N3 + Conselho + Subsíndico | ✅ entry | ✅ principal | ✅ acesso | ✅ indicações exclusivas | ✅ presidir | ➖ institucional |
| Morador (titular/dependente; base/pagante) | ✅ entry | ✅ painel/timeline/eventos | ✅ (pagante + N1 free) | ✅ feed bairro/condomínio | ✅ votar/checkin | ➖ institucional |
| Empresa Plus/Pro | ✅ entry | ✅ painel empresarial | ✅ acesso | ✅ explorar abertas | ✅ fornecedor em votação | ➖ institucional |
| Agência Marketing (actor delegado) | ✅ entry | ✅ painel agência (MK1-MK8) | ➖ | ➖ | ❌ | ➖ institucional |
| Parceiro Vizinhança | ✅ entry | ✅ painel parceiro (VZ1-VZ6) | ❌ bloqueado (D-062) | ✅ publisher | ❌ | ➖ institucional |
| **Admin owner Master Síndico** (D-134) | ➖ usa próprio fluxo SSO Cloudflare Access | ➖ NÃO mais embutido em cms | ➖ | ➖ | ➖ | ➖ — usa `apps/admin` dedicada (D-134); 5 módulos: Financeiro, Usuários, Moderação, Config, Broadcast |
| Público (não autenticado) | ➖ login/cadastro | ❌ | ❌ | ❌ | ❌ | ✅ marketing |

---

## 8. Ligação com artefatos canônicos

- [[../../00-product/personas]] — 7 personas + sub-roles ABAC
- [[../../00-product/portfolio-de-produtos]] — 7 sub-produtos
- [[../../01-domain/bounded-contexts]] — 6 BCs (paridade parcial — frontend é cliente dos BCs)
- [[../../04-requirements/global]] — GR-###/DR-### globais
- [[../../04-requirements/functional/_moc]] — funcionais por BC
- [[../../02-architecture/api-design]] — contratos REST canônicos
- [[../backend/README]] — stack servidor
- [[../mobile/README]] — stack mobile (paridade de personas morador/síndico)
- [[../admin-privilegios]] — **D-134 (2026-04-25)** app `apps/admin` dedicada (revoga D-072/D-058 transversal antigo)

---

## 9. Documentos deste sub-projeto

- [[architecture]] — monorepo Bun, feature-sliced, routing TanStack file-based, state Solid primitives, API client Axios, tokens UnoCSS, deploy Railway
- [[patterns]] — SolidJS primitives (signals/stores/resources), atomic design leve, Kobalte + CVA + splitProps, Code Calisthenics TS, TanStack Query/Form, a11y WCAG 2.1 AA, i18n pt-BR
- [[ui-catalog]] — catálogo exaustivo ≥ 141 telas × 6 apps × persona (rota, ações, estados, regras)
- [[design-system]] — Manual IV (ouro #C69C3F + navy #071A33), Michroma + Manrope, tokens OKLCH CSS vars + shortcuts UnoCSS, 20+ componentes
- [[requirements]] — Core Web Vitals, bundle budgets, PWA, a11y, i18n, quotas backend-first, LGPD UI
- [[security]] — cookie httpOnly, CSP strict nonce, XSS (SolidJS auto-escape + proibições innerHTML), CSRF double-submit + SameSite=Lax, CORS allowlist, IDOR 404, admin-isolation ABAC
- [[tasks]] — FE-### seed M1/M2/M3 por app

## Links

- [[_moc]] — MOC do sub-projeto
- [[../../_moc]] — MOC 03-subprojects
- [[../../CLAUDE]] — contrato do agente v2
- [[../../STEERING]] — não-negociáveis
- [[../../ROADMAP]] — marcos M1/M2/M3
- [[../../STATE]] — decisões vivas (D-###)
