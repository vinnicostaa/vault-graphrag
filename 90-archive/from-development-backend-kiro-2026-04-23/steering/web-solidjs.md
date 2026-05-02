---
inclusion: manual
---

# SolidJS — Frontend Web

> Ative este steering quando trabalhar em tarefas que envolvem o frontend web (`Development/web/`).

## Stack

- **SolidJS** + TanStack Router (file-based) + TanStack Query + TanStack Form
- **Rsbuild** (bundler)
- **UnoCSS** + Wind4 preset (Tailwind-compatible)
- **Kobalte** — primitivos acessíveis (Dialog, Select, Tabs, Combobox, etc.)
- **Zod** — validação de formulários
- **Axios** — HTTP client com `withCredentials: true`
- **Bun** — package manager

## Estrutura do Monorepo Web

```
apps/
  auth/          ← Zitadel OIDC flows (login, callback, logout)
  lp/            ← Landing page
  cms/           ← Painel principal (síndico, empresa, morador)
  lms/           ← Academia (pós-lançamento)
  assembly/      ← Assembleia ao vivo (pós-lançamento)
packages/
  ui/            ← Design system (Kobalte + UnoCSS)
  schemas/       ← Zod schemas compartilhados (validação frontend + backend)
  core/          ← SDK stateless (http client, auth helpers)
```

## Regras Absolutas

- `packages/core` é **stateless** — sem signals, sem stores, sem contextos. Apenas funções puras e wrappers de fetch
- Auth context vive em cada app (`apps/cms/src/context/auth.tsx`), não no core
- Toda chamada de API usa `credentials: 'include'` — o cookie httpOnly vai automaticamente
- **Nunca** armazene tokens em `localStorage` ou `sessionStorage`
- Use TanStack Query para server state. Use signals/stores do SolidJS para UI state local
- Formulários: TanStack Form + Zod para validação
- Componentes de UI: Kobalte para primitivos acessíveis

## Design System (imutável)

```css
--primary:    oklch(0.715 0.120 84.2)   /* Gold CTA */
--secondary:  oklch(0.218 0.055 256.1)  /* Navy */
--accent:     oklch(0.871 0.129 82.0)   /* Gold Light */
--background: oklch(0.976 0.003 264.5)  /* Light mode */
--background: oklch(0.183 0.031 263.4)  /* Dark mode */
```

Fontes: **Manrope** (body) · **Michroma** (headings, `letter-spacing: 0.05em`)

**Não altere a paleta de cores sem aprovação explícita.**

## Regras de Visibilidade (críticas)

- Vídeos de empresa **nunca** aparecem para moradores automaticamente
- Liberados apenas durante votação de fornecedor em assembleia quando `autorizar_exibicao_votacao=true`
- Revogados após fim da votação — frontend deve refletir sem cache stale
- Likes e comentários visíveis apenas ao dono do conteúdo

## Padrão de Componente SolidJS

```tsx
// Componente com TanStack Query
import { createQuery } from '@tanstack/solid-query'
import { Show, For } from 'solid-js'

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

## Padrão de Formulário

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
    onSubmit: async ({ value }) => {
      await api.post('/auth/login', value)
    },
    validatorAdapter: zodValidator(),
    validators: { onChange: schema },
  }))

  return (
    <form onSubmit={(e) => { e.preventDefault(); form.handleSubmit() }}>
      <form.Field name="email">
        {(field) => (
          <input
            value={field().state.value}
            onInput={(e) => field().handleChange(e.target.value)}
          />
        )}
      </form.Field>
    </form>
  )
}
```

## HTTP Client

```ts
// packages/core/src/http.ts — stateless, sem contextos
import axios from 'axios'

export const api = axios.create({
  baseURL: import.meta.env.VITE_API_URL,
  withCredentials: true, // cookie httpOnly vai automaticamente
})

// Nunca armazene o token — o cookie é gerenciado pelo browser
```

## Comandos

```bash
# Instalar dependências
bun install

# Rodar todos os apps
bun run dev

# Rodar apenas o CMS
bun run dev --filter=cms

# Build
bun run build

# Typecheck
bun run check-types

# Lint + format (Biome)
bun run check
```

## Regras de PR (web)

- Mesmo padrão do backend: imperativo, capitalizado, sem prefixos
- Prefixo de app quando escopo é claro: `cms: Add TrialBlocker component`
- Sempre incluir `Release Notes:` no final
