# My Síndico — Web

Frontend da plataforma My Síndico. SPA construída com **SolidJS** + **Rsbuild**, usando o ecossistema TanStack para roteamento, data-fetching e formulários, e UnoCSS para estilização.

## Tech Stack

| Categoria | Tecnologia |
|---|---|
| **Framework** | SolidJS 1.9 |
| **Bundler** | Rsbuild |
| **Roteamento** | TanStack Solid Router (file-based) |
| **Data Fetching** | TanStack Solid Query |
| **Forms** | TanStack Solid Form + Zod adapter |
| **Estilização** | UnoCSS (attributify + icons + typography + web-fonts) |
| **Componentes** | Kobalte (headless UI) + CVA (variantes) |
| **HTTP Client** | Axios |
| **Notificações** | solid-sonner |
| **OTP** | @corvu/otp-field |
| **Validação** | Zod v4 (via `mastersindico-schemas`) |
| **Linting** | Biome |

## Estrutura

```
src/
├── index.tsx              → Entrypoint (Router + QueryClient + Providers)
├── main.css               → Estilos globais
├── routeTree.gen.ts       → Gerado automaticamente pelo TanStack Router
├── components/
│   ├── auth/              → Componentes de autenticação (login, register, etc.)
│   ├── content/           → Componentes de conteúdo
│   ├── form/              → Componentes de formulário reutilizáveis
│   ├── handlers/          → Error/loading handlers
│   ├── layouts/           → Layouts base
│   ├── profile/           → Componentes de perfil
│   ├── shared/            → Componentes genéricos
│   └── ui/                → Design system base (button, input, card, etc.)
├── context/               → Providers (auth, theme, etc.)
├── hooks/                 → Custom hooks (useAuth, etc.)
├── lib/                   → Axios instance, helpers
└── routes/                → File-based routing
    ├── __root.tsx          → Layout raiz
    ├── _auth/              → Rotas de autenticação (login, register, verify, etc.)
    ├── _private/           → Rotas protegidas (requer autenticação)
    └── _public/            → Rotas públicas
```

## Roteamento

O roteamento é **file-based** via TanStack Solid Router. O `routeTree.gen.ts` é gerado automaticamente pelo plugin do router.

### Route Groups

| Grupo | Prefixo | Descrição |
|---|---|---|
| `_auth` | `/auth/*` | Login, cadastro, verificação, reset de senha |
| `_private` | `/*` | Dashboard, perfil, configurações (requer JWT) |
| `_public` | `/*` | Páginas públicas (landing, busca) |

## Comunicação com a API

```
Web (SolidJS) ←→ Axios ←→ API (Fastify :8000)
         ↖___________________↗
              mastersindico-schemas (Zod)
```

- Schemas de request/response compartilhados via `mastersindico-schemas`
- Validação client-side e server-side usando os mesmos Zod schemas
- TanStack Query gerencia cache, refetch e estados de loading/error

## Identidade Visual

Conforme o **Manual de Identidade Visual — Master Síndico**:
- Tipografia principal: **Michroma** (display) + fontes web do UnoCSS
- Paleta de cores definida no manual de branding
- Arquivo da fonte disponível em `contextos/Michroma-Regular.ttf`

## Scripts

```bash
# Desenvolvimento (hot-reload) — http://localhost:3000
bun run dev

# Build de produção
bun run build

# Preview do build local
bun run preview

# Lint e format
bun run check
bun run format
```
