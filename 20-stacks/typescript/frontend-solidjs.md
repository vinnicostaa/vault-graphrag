---
title: TypeScript — Frontend SolidJS
type: template
tags: [typescript, solidjs, frontend, tanstack]
source: .kiro/steering/web-solidjs.md
created: 2026-04-22
aliases: [SolidJS stack, Frontend SolidJS]
---

# TypeScript — Frontend SolidJS

Template de padrões pra frontend web em **SolidJS + TanStack + UnoCSS + Kobalte**. Stack escolhido pelo projeto Master Síndico; aplicável a qualquer SPA tipada que queira reatividade fina-granular (signals) em vez de VDOM.

## Stack canônico

- **SolidJS** — reatividade via signals (primitives `createSignal`, `createMemo`, `createEffect`)
- **TanStack Router** — file-based + type-safe + auto code-splitting
- **TanStack Query** — server state (cache, refetch, dedup)
- **TanStack Form** — forms + validator adapter (Zod)
- **Rsbuild** — bundler (sucessor Rspack-based de Vite)
- **UnoCSS** + Wind4 preset — atomic CSS compatível com Tailwind
- **Kobalte** — primitivos acessíveis headless (Dialog, Select, Tabs, Combobox)
- **Zod** — validação de formulários + response parsing
- **Axios** com `withCredentials: true`
- **Bun** — package manager + runtime
- **Biome** — linter + formatter (spaces + double quotes)

## Estrutura de monorepo (Bun workspaces)

```
apps/
  <app-funcional>/   um app por função (auth, admin, marketing, etc.)
packages/
  ui/                design system (primitivos + variantes + tokens)
  schemas/           Zod schemas compartilhados (validação frontend + backend)
  core/              SDK stateless (http client, auth helpers) — opcional
```

**Regra dura**: `packages/core` é **stateless**. Sem signals, sem stores, sem contextos. Só funções puras e wrappers de fetch.

## Regras absolutas

- **Auth context vive em cada app**, não no core. Core é stateless.
- **`credentials: 'include'`** em toda chamada de API — cookie httpOnly vai automaticamente.
- **Nunca** armazenar tokens em `localStorage` ou `sessionStorage`. Ver [[../../10-knowledge/security/security-principles|security-principles]] (storage client-side é inseguro — usar cookie httpOnly + credentials: 'include').
- **TanStack Query pra server state**; signals/stores do SolidJS pra UI state local.
- **Forms: TanStack Form + Zod** via `zodValidator()`.
- **UI primitives: Kobalte** (acessibilidade garantida). Estilo via UnoCSS shortcuts.

## Design System via CSS variables

Paleta definida como CSS variables. Exemplo:

```css
--primary:    oklch(0.715 0.120 84.2);
--secondary:  oklch(0.218 0.055 256.1);
--accent:     oklch(0.871 0.129 82.0);
--background: oklch(0.976 0.003 264.5);  /* light */
--background: oklch(0.183 0.031 263.4);  /* dark */
```

UnoCSS lê as variables no `uno.config.ts` → dark mode é CSS-variable driven, sem JS.

**Paleta é imutável** sem aprovação — tokens de design ficam em `packages/ui` versionado, alterações via PR dedicado. Evita divergência entre apps e regressão visual silenciosa.

## Padrão: componente com TanStack Query

```tsx
import { createQuery } from '@tanstack/solid-query'
import { Show } from 'solid-js'

export function SubscriptionStatus() {
  const query = createQuery(() => ({
    queryKey: ['subscription', 'me'],
    queryFn: () => api.get('/api/v1/subscriptions/me'),
  }))

  return (
    <Show when={!query.isLoading} fallback={<Skeleton />}>
      <Show when={query.data} fallback={<EmptyState />}>
        {(data) => <SubscriptionCard subscription={data()} />}
      </Show>
    </Show>
  )
}
```

Observar:
- `createQuery(() => ({...}))` — factory function; SolidJS precisa disso pra reatividade.
- `query.data` é signal — acessar como `data()` (call).
- `<Show>` em vez de `{cond && ...}` pra preservar reatividade.

## Padrão: formulário com TanStack Form + Zod

```tsx
import { createForm } from '@tanstack/solid-form'
import { zodValidator } from '@tanstack/zod-form-adapter'
import { z } from 'zod'

const schema = z.object({
  email: z.string().email('Email inválido'),
  password: z.string().min(8, 'Mínimo 8 caracteres'),
})

export function SignInForm() {
  const form = createForm(() => ({
    defaultValues: { email: '', password: '' },
    onSubmit: async ({ value }) => { await api.post('/auth/login', value) },
    validatorAdapter: zodValidator(),
    validators: { onChange: schema },
  }))

  return (
    <form onSubmit={(e) => { e.preventDefault(); form.handleSubmit() }}>
      <form.Field name="email">
        {(field) => (
          <input
            value={field().state.value}
            onInput={(e) => field().handleChange(e.currentTarget.value)}
          />
        )}
      </form.Field>
    </form>
  )
}
```

Schema Zod em `packages/schemas/` — reusado no backend. Ver [[../../10-knowledge/security/security-principles|security-principles]] (validação server-side é obrigatória; frontend só melhora UX).

## HTTP client stateless

```ts
// packages/core/src/http.ts
import axios from 'axios'

export const api = axios.create({
  baseURL: import.meta.env.VITE_API_URL,
  withCredentials: true,   // cookie httpOnly vai sozinho
  timeout: 10_000,
})

// NUNCA armazene token manualmente — o cookie é gerenciado pelo browser
```

Interceptor padrão (apps adicionam na init):

```ts
api.interceptors.response.use(
  (r) => r,
  (error) => {
    if (error.response?.status === 401) window.location.href = '/auth/login'
    return Promise.reject(error)
  }
)
```

## Comandos

```bash
bun install                   # deps
bun run dev                   # todos os apps (filter '*' --parallel)
bun run dev --filter=<app>    # app específico
bun run build
bun run check                 # Biome check --write
bun run format                # Biome format --write
bun run check-types           # tsc --noEmit em todos
```

## Gates duros antes de PR

- `bun run check-types` — zero erro strict
- `bun run check` — zero warning Biome
- Build produção limpo (sem warning de chunk gigante)
- Coverage (quando houver Vitest) ≥ 80% em lógica, ≥ 90% em `packages/ui` e `packages/schemas`

## Convenções de PR

- Título imperativo, capitalizado, sem prefixos convencionais (`fix:`, `feat:`).
- Prefixo de app quando escopo claro: `cms: Add X feature`.
- Sempre incluir `Release Notes:` no final.

## Links

- [[../_moc]] — stacks
- [[../../10-knowledge/_moc]] — padrões atemporais
- [[../../10-knowledge/antipatterns/_moc]] — antipatterns catalogados (code smells Fowler)
- [[../../10-knowledge/runtime-antipatterns/_moc]] — antipatterns operacionais (OP-###)
