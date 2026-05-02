---
title: Web — Patterns SolidJS/TS
type: guide
tags: [subproject, web, master-sindico, v2, patterns, solidjs, typescript, kobalte, tanstack, code-calisthenics, a11y]
source: Development/web/CLAUDE.md §Code Calisthenics + §TDD + AGENTS.md §Filosofia + packages/ui/src/button.tsx
created: 2026-04-23
updated: 2026-04-23
aliases: [web-patterns]
---

# Web — Patterns SolidJS/TS

Patterns idiomáticos do sub-projeto `web/`. Complementa [[architecture]] e [[design-system]]. Fonte primária: `Development/web/CLAUDE.md` §Code Calisthenics (linhas 304-318) + §TDD + `packages/ui/src/button.tsx` como template.

---

## 1. SolidJS primitives — quando usar cada uma

| Primitive | Quando | Exemplo |
|---|---|---|
| `createSignal<T>()` | UI state local, reativo | `const [open, setOpen] = createSignal(false);` |
| `createStore<T>()` | Objeto/array com mutações parciais | `const [form, setForm] = createStore({ nome: '', email: '' });` |
| `createMemo<T>()` | Valor derivado (cache automático) | `const total = createMemo(() => items().reduce((s,i) => s+i.value, 0));` |
| `createResource<T>()` | Fetch simples (async dependente de signals) | `const [user] = createResource(() => userId(), fetchUser);` |
| `createEffect()` | Side effect reativo | `createEffect(() => localStorage.setItem('theme', theme()));` |
| `onMount()` | 1x após mount | `onMount(() => analytics.track('page_view'));` |
| `onCleanup()` | Teardown (dispose) | `onCleanup(() => ws.close());` |
| `createDeferred()` | Computação pesada postergada | busca com throttle |
| `createContext<T>()` + `useContext` | Transversal no app | `AuthContext`, `SessionContext` |
| `For` / `Show` / `Switch` / `Match` | Render lists/condicionais | `<Show when={user()} fallback={<Spinner/>}>…</Show>` |
| `createQuery` (TanStack) | Server state (read) | queries HTTP com cache/revalidação |
| `createMutation` (TanStack) | Server state (write) | POST/PUT/DELETE com optimistic updates |

**Proibições** (CLAUDE.md + SolidJS docs):
- ❌ `useState`, `useEffect`, `useRef`, `useMemo` — isso é React. Errado.
- ❌ `.value` para ler signal — é **função**: `count()`, não `count.value`.
- ❌ Destructuring em props de componente — **quebra reatividade**. Use `props.x` direto ou `splitProps`.
- ❌ Spreading signals diretamente — `{...signal()}` recria a cada tick. Use `mergeProps`.

---

## 2. Component library (atomic design leve)

### 2.1 Hierarquia em `packages/ui/src/`

```
atoms/     button, input, textarea, checkbox, radio, switch, badge, spinner, divider, avatar, icon
molecules/ form-field (label+input+helper+error), card, menu-item, tab-strip, breadcrumb, pagination, tooltip
organisms/ dialog (modal), drawer, toast-host, popover, data-table, combobox, date-picker, upload-dropzone, stepper, command-palette
```

Telas (screens) vivem em `apps/<app>/src/routes/**` — **não** em `packages/ui`. Package `ui` é só primitive/atoms/molecules/organisms reutilizáveis.

### 2.2 Convenções de props

- **Máx 5 props** por componente. Mais que isso = agrupa em `config: {...}` ou usa composition (slots / `children`).
- **Callbacks prefixo `on`**: `onSubmit`, `onClose`, `onChange`, `onSelect`.
- **`children: JSX.Element`** suportado — composition sobre configuração.
- **`class?: string`** aceito para override externo (passado via `cn()`).
- **`as?: ValidComponent`** via Kobalte `PolymorphicProps` — permite renderizar como `<a>`, `<Link>`, `<button>` sem duplicação.

### 2.3 Template canônico (já no código)

`packages/ui/src/button.tsx` é a referência:

```tsx
const buttonVariants = cva(
  "inline-flex items-center justify-center gap-2 whitespace-nowrap rounded-md ...",
  {
    variants: {
      variant: { default: "btn-primary", destructive: "btn-destructive", outline: "btn-outline",
                 secondary: "btn-secondary", ghost: "btn-ghost", link: "btn-link" },
      size:    { default: "btn-md", sm: "btn-sm", lg: "btn-lg", icon: "btn-icon" },
    },
    defaultVariants: { variant: "default", size: "default" },
  },
);
```

**Regras ao criar novo componente**:
1. **Kobalte primitive** como raiz (`Dialog.Root`, `Tabs.Root`, ...) — acessibilidade built-in.
2. **CVA** para variant × size, mapeadas para **shortcuts UnoCSS** (não utilitários crus — reusa design system).
3. **`splitProps`** para separar estilo vs outros:
   ```tsx
   const [local, others] = splitProps(props, ["variant", "size", "class"]);
   ```
4. **`PolymorphicProps<T, ...>`** para `as`.
5. **`cn(...)`** (space-join em `packages/ui/src/lib/utils.ts`) — **sem** `clsx`/`tailwind-merge`.
6. Re-exportar em `packages/ui/src/index.ts`.

---

## 3. UnoCSS — attributify + shortcuts

### 3.1 Attributify prefixada

```tsx
// ✅ correto
<div un-flex un-items-center un-gap-4>...</div>

// ❌ errado — attributify config usa prefixedOnly: true
<div flex items-center gap-4>...</div>
```

### 3.2 Shortcuts são o design system — USE, não reinvente

| Categoria | Shortcut | Componente equivalente |
|---|---|---|
| Button | `btn-base`, `btn-primary`, `btn-secondary`, `btn-outline`, `btn-ghost`, `btn-link` × sizes `btn-sm/md/lg/icon` | `<Button>` |
| Tipografia | `text-display-h1`, `text-display-h2`, `text-display-h3` (Michroma); `text-body`, `text-small`, `text-muted` (Manrope) | — |
| Layout | `flex-center`, `flex-between`, `glass-panel`, `bg-grid` | — |
| LP Hero | `lp-blob`, `lp-section`, `lp-heading`, `lp-subheading`, `lp-cta`, `lp-cta-outline` | scope apps/lp |

**Proibição**: nunca hardcode hex (`bg-[#C69C3F]`). Usa token: `bg-primary`, `bg-secondary`, `border-border`, `text-muted-foreground`.

### 3.3 Ícones (presetIcons)

```tsx
<i class="i-lucide-home" />                           {/* inline */}
<i class="i-lucide-check text-2xl text-success" />    {/* com utility */}
```

Catalog: Lucide (`i-lucide-*`) é o default. Outros sets disponíveis via `presetIcons`.

### 3.4 Fonts

- `font-sans` = **Manrope** (body, 400-800, italic habilitado)
- `font-display` = **Michroma** (headings, single weight 400, `letter-spacing: 0.05em` aplicado em `h1-h6` via CSS global)
- `font-mono` = Fira Code / Fira Mono (código inline quando precisa)

Carregadas via `presetWebFonts` do Google. Preload recomendado em `<head>` do `index.html` (FE-003).

---

## 4. TanStack patterns

### 4.1 Router — file-based + loaders

```tsx
// src/routes/_authenticated/condominios/$condominiumId.tsx
import { createFileRoute } from '@tanstack/solid-router';
import { api, get } from '../../../core/api/http';
import { CondominiumSchema } from 'schemas';

export const Route = createFileRoute('/_authenticated/condominios/$condominiumId')({
  loader: async ({ params }) => get(`/api/v1/condominios/${params.condominiumId}`, CondominiumSchema),
  pendingComponent: CondominiumSkeleton,
  errorComponent: ({ error }) => <ErrorBanner error={error} />,
  component: CondominiumPage,
});

function CondominiumPage() {
  const condominium = Route.useLoaderData();         // já validado Zod
  return <div>{condominium().name}</div>;
}
```

- **`loader`** roda antes de render; bloqueia a navegação até resolver (com `pendingComponent` entre).
- **`beforeLoad`** é o lugar para auth guards (redirect se não autorizado).
- **`validateSearch`** com Zod para query params tipados.

### 4.2 Query — TanStack Solid Query

```tsx
import { useQuery } from '@tanstack/solid-query';

function Timeline() {
  const condominiumId = useCondominiumId();
  const query = useQuery(() => ({
    queryKey: ['timeline', condominiumId()],
    queryFn: () => get(`/api/v1/condominios/${condominiumId()}/timeline`, TimelineSchema),
    staleTime: 30_000,
  }));
  return (
    <Show when={query.data} fallback={<TimelineSkeleton />}>
      {(data) => <TimelineList entries={data().entries} />}
    </Show>
  );
}
```

- **Query key** começa pelo nome do resource + identificadores relevantes (tenant, filtros).
- **Invalidation** via `queryClient.invalidateQueries({ queryKey: ['timeline'] })` após mutation.
- **Optimistic updates** em mutation `onMutate` + rollback em `onError`.

### 4.3 Forms — TanStack Solid Form + Zod

```tsx
import { createForm } from '@tanstack/solid-form';
import { zodValidator } from '@tanstack/zod-form-adapter';
import { SindicoOnboardingStep3Schema } from 'schemas';

function GovernanceMarkersForm() {
  const form = createForm(() => ({
    defaultValues: { markers: [] as string[] },
    validatorAdapter: zodValidator(),
    onSubmit: async ({ value }) => {
      await api.patch('/api/v1/onboarding/progress', { step: 3, data: value });
    },
  }));

  return (
    <form.Provider>
      <form.Field
        name="markers"
        validators={{ onChange: SindicoOnboardingStep3Schema.shape.markers }}
      >
        {(field) => <CheckboxGroup {...field().state} onChange={field().handleChange} />}
      </form.Field>
      <Button type="submit">Salvar</Button>
    </form.Provider>
  );
}
```

---

## 5. Accessibility (WCAG 2.1 AA — obrigatório)

### 5.1 Regras duras

- **Keyboard navigation**: toda ação reachable via Tab; `Enter`/`Space` ativam; `Esc` fecha modals. Kobalte cuida disso — **não reimplementar**.
- **Focus management**: `Dialog` de Kobalte trap focus dentro; retorna ao trigger no close. Usa `autofocus` com critério (primeiro campo editável).
- **ARIA**: role, aria-label, aria-describedby, aria-invalid em forms. `<Label>` sempre amarrado com `for=/id=`.
- **Contraste** mínimo **4.5:1 (texto normal) / 3:1 (texto grande 18px+ bold)**. Tokens do [[design-system]] respeitam.
- **Screen reader**: anúncios de mudança de estado via `aria-live="polite"` (toasts, status loaders).
- **Semantic HTML**: `<nav>`, `<main>`, `<header>`, `<footer>`, `<article>`, `<section>`. Não usar `<div>` quando semântica existe.

### 5.2 Teste de a11y

- Vitest + `jest-axe` em suite unitária: toda tela roda `axe(render(Component))` — zero violations esperadas.
- Manual: testar com leitor de tela (NVDA Windows / VoiceOver macOS) em fluxos críticos (login, votação, Connect Me envio).

---

## 6. Loading / Empty / Error / Success — 4 estados obrigatórios

Toda tela que faz fetch deve ter os 4 estados tratados explicitamente:

```tsx
function ConnectMeList() {
  const query = useQuery(() => ({ queryKey: ['connect-me'], queryFn: fetchConnectMeList }));

  return (
    <Switch>
      <Match when={query.isPending}>
        <SkeletonList rows={5} />                          {/* LOADING */}
      </Match>
      <Match when={query.isError}>
        <ErrorBanner error={query.error} onRetry={() => query.refetch()} />  {/* ERROR */}
      </Match>
      <Match when={query.data?.length === 0}>
        <EmptyState
          icon="i-lucide-inbox"
          title="Nenhuma solicitação ainda"
          description="Quando empresas responderem ao seu Connect Me, aparecem aqui."
          action={<Button onClick={...}>Criar novo Connect Me</Button>}
        />                                                   {/* EMPTY */}
      </Match>
      <Match when={query.data}>
        <ConnectMeItems items={query.data} />                 {/* SUCCESS */}
      </Match>
    </Switch>
  );
}
```

**Quinto estado (permission-denied)** quando aplicável: `<AccessDenied reason="Plano atual não permite esta ação" cta="Upgrade" />`.

---

## 7. Error boundaries + toast

```tsx
// __root.tsx (por app)
<ErrorBoundary fallback={<GlobalErrorFallback />}>
  <Outlet />
</ErrorBoundary>
```

- **Global**: no root — captura erros não tratados, mostra fallback amigável, reporta a Sentry.
- **Por rota**: `errorComponent` no `createFileRoute` — captura erros do loader.
- **Toast**: feedback de sucesso/erro em ações (Kobalte Toast). Host único `<Toaster />` em `__root.tsx`. Acessível (role="status").

---

## 8. Code Calisthenics adaptado TypeScript/SolidJS

Adaptação das 9 regras do `code-calisthenics.md` (backend-Go) para TypeScript/SolidJS, conforme CLAUDE.md §"Code Calisthenics" (linhas 304-318):

### 8.1 Regra 1 — **1 nível de indent por função/componente**

```tsx
// ❌ ruim
function handle(event: Event) {
  if (event.type === 'click') {
    if (state() === 'ready') {
      if (user()?.canAct) {
        doIt();
      }
    }
  }
}

// ✅ bom
function handle(event: Event) {
  if (event.type !== 'click') return;
  if (state() !== 'ready') return;
  if (!user()?.canAct) return;
  doIt();
}
```

### 8.2 Regra 2 — **Sem `else`**

```tsx
// ❌ if/else
const label = active() ? 'Ativo' : 'Inativo';   // ✅ ternário é OK quando único

// ❌ else if encadeado
if (status === 'pending') return <Pending />;
else if (status === 'active') return <Active />;
else if (status === 'closed') return <Closed />;
else return <Unknown />;

// ✅ map/dispatch
const components = { pending: Pending, active: Active, closed: Closed };
const Component = components[status] ?? Unknown;
return <Component />;
```

### 8.3 Regra 3 — **Envolver primitivos com branded types**

```ts
// primitive obsession
function fetchUser(id: string) { ... }  // ❌ string é vago

// branded
type UserId = string & { __brand: 'UserId' };
function fetchUser(id: UserId) { ... }   // ✅ compilador força

// via Zod
export const UserIdSchema = z.string().uuid().brand<'UserId'>();
export type UserId = z.infer<typeof UserIdSchema>;
```

Candidatos a branded: `UserId`, `TenantId`, `CondominiumId`, `UnitId`, `VoteOptionId`, `CPF`, `CNPJ`, `Email`, `Phone`, `Money` (int centavos), `FracaoIdeal` (string decimal).

### 8.4 Regra 4 — **First-class collections**

```ts
// ❌ array primitivo exposto
const votes: Vote[] = [...];
votes.push(newVote);

// ✅ collection imutável com invariantes
type VoteCollection = {
  readonly votes: ReadonlyArray<Vote>;
  add(v: Vote): VoteCollection;
  winner(): OptionId | null;
  count(): number;
};

function makeVoteCollection(votes: Vote[] = []): VoteCollection {
  return {
    votes: Object.freeze([...votes]),
    add: (v) => makeVoteCollection([...votes, v]),
    winner: () => /* ... */,
    count: () => votes.length,
  };
}
```

### 8.5 Regra 5 — **Um ponto por linha (Lei de Demeter)**

```ts
// ❌ user.profile.address.city
const city = user().profile?.address?.city;

// ✅ método no agregado
const city = user().cityName();   // user retorna cityName já resolvido
```

### 8.6 Regra 6 — **Não abreviar**

```ts
// ❌
const sub = subs.filter(s => s.exp > now);
const q = qs.get('id');
const condo = condos[0];

// ✅
const activeSubscriptions = subscriptions.filter(s => s.expiresAt > now);
const searchParam = searchParams.get('id');
const condominium = condominiums[0];

// Exceções idiomáticas permitidas:
// - e (event callback)
// - i (índice de loop curto)
// - ctx (context)
// - cn (class-join helper — convenção externa consolidada)
```

### 8.7 Regra 7 — **Componentes pequenos**

Máx ~**100 linhas JSX**. Acima: extrair sub-componentes. Exceção: tela inteira de formulário multi-step (~150 linhas se cada field é trivial).

### 8.8 Regra 8 — **Máx 3-5 props**

```tsx
// ❌ 8 props
<CondominiumCard
  id={...} name={...} address={...} cep={...} unitsCount={...}
  activeSindico={...} plan={...} onSelect={...} onDelete={...}
/>

// ✅ consolidar
<CondominiumCard
  condominium={condominium}        // um objeto tipado
  actions={{ onSelect, onDelete }}
/>
```

### 8.9 Regra 9 — **Sem setters/getters**

SolidJS já força via signals/stores. Mutação direta em store é via `setStore(path, value)` — acessor controlado. Nunca `obj.foo = bar` em objeto reativo.

---

## 9. TanStack Query patterns

### 9.1 Query keys hierárquicos

```ts
['timeline', condominiumId()]                                    // list
['timeline', condominiumId(), entryId]                           // detail
['timeline', condominiumId(), { filters: { type: 'evento' }}]    // filtered list
```

### 9.2 Invalidação via tag

```ts
// mutation cria entrada
const mutation = createMutation(() => ({
  mutationFn: (data) => api.post(`/api/v1/timeline`, data),
  onSuccess: () => queryClient.invalidateQueries({ queryKey: ['timeline', condominiumId()] }),
}));
```

### 9.3 Optimistic updates (com rollback)

```ts
const mutation = createMutation(() => ({
  mutationFn: (data) => api.post('/api/v1/events', data),
  onMutate: async (newEvent) => {
    await queryClient.cancelQueries({ queryKey: ['events'] });
    const previous = queryClient.getQueryData(['events']);
    queryClient.setQueryData(['events'], (old: Event[]) => [...old, { ...newEvent, id: 'temp-id' }]);
    return { previous };
  },
  onError: (err, _, ctx) => {
    queryClient.setQueryData(['events'], ctx?.previous);
    toast.error('Falha ao criar evento. Tente novamente.');
  },
  onSettled: () => queryClient.invalidateQueries({ queryKey: ['events'] }),
}));
```

### 9.4 Logout = reset total

```ts
async function logout() {
  await api.post('/api/v1/auth/logout');
  queryClient.clear();                               // limpa cache
  sessionStore.reset();                              // limpa store local
  window.location.href = `${AUTH_URL}/sign-in`;      // full redirect
}
```

---

## 10. i18n

MVP pt-BR only. Preparo para EN (M3+):

- **Biblioteca**: `@solid-primitives/i18n` (backlog M3) ou manual dictionary por app.
- **Termos preservados em pt-BR** (não traduzir mesmo para EN): "Síndico", "Condomínio", "Assembleia", "Plano Diretor", "Fração Ideal", "Connect Me" (é brand), "Outorgante/Outorgado", "Morador", "Edital", "Pauta".
- **Copy**: sempre em pt-BR, formalidade média (usa "você", não "tu"/"vos"), voz ativa.
- **Accessibility labels**: também em pt-BR (`aria-label="Abrir menu"`).
- **Error messages**: humanizadas, sem jargão técnico ("Não foi possível carregar. Tente novamente." em vez de "HTTP 500").

---

## 11. Anti-patterns (proibidos)

| Anti-pattern | Motivo | Alternativa |
|---|---|---|
| `innerHTML`, `dangerouslySetInnerHTML` | XSS vector | Render texto via `{variable}` (auto-escape); markdown via lib segura (`marked` + DOMPurify) |
| `eval`, `new Function`, `setTimeout(string)` | Code execution | Closures e funções tipadas |
| Token em `localStorage`/`sessionStorage` | XSS leva tudo | Cookie httpOnly (único caminho) |
| Calcular valor monetário no frontend | Inconsistência com backend | **Backend retorna, frontend exibe** |
| Calcular quota (Connect Me, vídeos) no frontend | Mesma razão | Backend retorna `remainingQuota` |
| Validação ABAC no frontend para **decidir** ação | Ele é cliente, não guard | Usa pra esconder UI; backend re-valida |
| Abrir WS sem fechar em cleanup | Leak de memória/sockets | `onCleanup(() => ws.close())` |
| `createSignal` armazenando server data | Fora do ciclo cache | `createQuery` |
| Props destructuring direto em componente | Quebra reatividade Solid | `props.x` ou `splitProps` |
| Usar `class` + `className` misturados | SolidJS usa `class` (não `className`) | `class` sempre |
| Hardcode hex (`bg-[#C69C3F]`) | Quebra dark mode + tokens | Tokens `bg-primary`, `text-muted-foreground` |
| `else if` encadeado | Code Calisthenics rule 2 | Map/dispatch |

---

## 12. Testing patterns

### 12.1 Test harness

```ts
// test/helpers/render.tsx
import { render } from '@solidjs/testing-library';
import { QueryClient, QueryClientProvider } from '@tanstack/solid-query';
import { Router, RouterProvider } from '@tanstack/solid-router';

export function renderWithProviders(ui: () => JSX.Element) {
  const queryClient = new QueryClient({ defaultOptions: { queries: { retry: false } } });
  return render(() => (
    <QueryClientProvider client={queryClient}>
      <ColorModeProvider>{ui()}</ColorModeProvider>
    </QueryClientProvider>
  ));
}
```

### 12.2 Padrão teste de form

```ts
import { describe, it, expect } from 'vitest';
import { userEvent } from '@testing-library/user-event';

describe('LoginForm', () => {
  it('bloqueia submit com email inválido', async () => {
    const { getByLabelText, getByRole } = renderWithProviders(() => <LoginForm />);
    const user = userEvent.setup();

    await user.type(getByLabelText('Email'), 'invalido');
    await user.click(getByRole('button', { name: /entrar/i }));

    expect(getByRole('alert')).toHaveTextContent(/email inválido/i);
  });
});
```

### 12.3 MSW para fetching

```ts
// test/msw.ts
import { setupServer } from 'msw/node';
import { http, HttpResponse } from 'msw';

export const server = setupServer(
  http.get('/api/v1/auth/me', () => HttpResponse.json({ success: true, data: sessionFixture })),
  http.post('/api/v1/auth/login', () => HttpResponse.json({ success: false, error: { code: 'INVALID_CREDENTIALS' } }, { status: 401 })),
);
```

### 12.4 Property-based para regras críticas

```ts
import fc from 'fast-check';

it('fração ideal nunca soma acima de 1.0', () => {
  fc.assert(
    fc.property(
      fc.array(fc.float({ min: 0.0001, max: 1, noNaN: true }), { minLength: 1, maxLength: 200 }),
      (fractions) => {
        const normalized = normalizeFractions(fractions);
        return normalized.reduce((a, b) => a + b, 0) <= 1.0 + Number.EPSILON;
      },
    ),
  );
});
```

Regras críticas que **exigem** property-based (CLAUDE.md linhas 262-270):
- Quotas Connect Me (morador Base 2/ano, Pagante 4/ano) — frontend reflete.
- Trial expiração — frontend confia no `trial_ends_at`.
- Visibilidade vídeo empresa — respeita flag `autorizar_exibicao_votacao`.
- Fração ideal em UI de votação — exibe DECIMAL recebido (nunca soma local com `number`).

---

## 13. Performance patterns

### 13.1 Lazy loading por rota

`autoCodeSplitting: true` do TanStack plugin cuida automaticamente. Cada rota vira chunk separado.

### 13.2 Image optimization

```tsx
<img
  src="/logo-dourado.avif"
  fallback="/logo-dourado.webp"
  srcSet="/logo-dourado-1x.avif 1x, /logo-dourado-2x.avif 2x"
  loading="lazy"                    // abaixo da fold
  decoding="async"
  width="188" height="40"           // evita CLS
  alt="Master Síndico"
/>
```

### 13.3 `createResource` com throttle em searches

```tsx
const searchTerm = createSignal('');
const throttled = throttleSignal(searchTerm, 300);
const [results] = createResource(throttled, fetchSearchResults);
```

### 13.4 Virtualização em listas longas

Uso de `@tanstack/solid-virtual` quando lista > 100 itens (timeline, notificações, histórico audit).

---

## 14. Antipatterns SolidJS-específicos

```tsx
// ❌ destructuring props — PERDE reatividade
function User({ name, email }: Props) { return <span>{name}</span>; }

// ✅ acessa direto
function User(props: Props) { return <span>{props.name}</span>; }

// ❌ ler signal fora de reactive scope
const count = countSignal();                          // one-shot, não reage
console.log(count);

// ✅ dentro de effect/memo/JSX
createEffect(() => console.log(countSignal()));
return <span>{countSignal()}</span>;

// ❌ spread em JSX de signal
<Comp {...userSignal()} />                            // recria obj a cada tick

// ✅ mergeProps ou prop individual
<Comp name={userSignal().name} email={userSignal().email} />

// ❌ for...of dentro de JSX
return <ul>{items.map(i => <li>{i}</li>)}</ul>;       // funciona mas não tracks

// ✅ <For>
return <ul><For each={items()}>{(i) => <li>{i}</li>}</For></ul>;
```

---

## 15. Security patterns (resumo — detalhe em [[security]])

- `credentials: "include"` em **toda** chamada HTTP.
- Validar Zod **toda response** antes de usar (defesa em profundidade).
- Escapar strings user-provided — **nunca** `innerHTML` (SolidJS auto-escape com `{variable}`).
- `queryClient.clear()` no logout + redirect.
- `dangerouslySetInnerHTML` **não existe em SolidJS** mas equivalente `innerHTML` é **proibido**. Plugin `security-guidance` bloqueia.
- PII em logs console: masked (`masked(cpf)`) — `LOG_LEVEL` via `import.meta.env.PROD`.

---

## Links

- [[README]] — overview
- [[architecture]] — monorepo + routing + state
- [[design-system]] — tokens + shortcuts
- [[security]] — defesa browser
- [[ui-catalog]] — telas
- [[_moc]] — MOC do web
