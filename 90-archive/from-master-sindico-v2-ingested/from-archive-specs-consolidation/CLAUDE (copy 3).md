# CLAUDE.md

Guia para Claude Code operando no sub-projeto **`web/`** do Master Síndico.

- **GitHub**: https://github.com/mastersindico/web
- **Stack**: SolidJS · TanStack Router/Query/Form · Rsbuild · UnoCSS · Kobalte · Axios · Zod · Biome · Bun workspaces
- **Papel no ecossistema**: consumidor da API Go em `../backend/` e fonte da experiência web dos síndicos, moradores, empresas, agências e parceiros.

O `../CLAUDE.md` cobre contexto cross-repo (produto, personas, money-in-cents, Zitadel, tenant isolation). Este arquivo é a **fonte de verdade operacional do web**. `AGENTS.md` ao lado contém a definição completa do papel sênior + guidelines cross-stack.

## Workspace Layout

Bun workspaces (não Turborepo, não pnpm). Seis apps SolidJS + dois packages compartilhados, linkados via `workspace "*"`:

| Path                 | Role                                          | Dev port |
|----------------------|-----------------------------------------------|----------|
| `apps/auth`          | Zitadel OIDC flows (PKCE entry point)         | 3000     |
| `apps/cms`           | Painel principal (síndico, empresa, morador)  | 3001     |
| `apps/lms`           | Academia (pós-lançamento)                     | 3002     |
| `apps/forum`         | Fórum / Vizinhança                            | 3003     |
| `apps/assembly`      | Assembleia ao vivo (pós-lançamento)           | 3004     |
| `apps/lp`            | Landing page                                  | 3005     |
| `packages/ui`        | Design system (Kobalte + CVA + UnoCSS)        | —        |
| `packages/schemas`   | Zod schemas compartilhados                    | —        |

Portas são hardcoded em cada app's `package.json` `dev` script (`rsbuild dev --port NNNN`). Mudar porta requer atualizar CORS do backend + redirect URIs do Zitadel — **nunca fazer casualmente**.

A pasta `old/` é gitignored, sobra da iteração anterior (`auth-context`, `query-context`). **Não importar de lá**.

## Comandos essenciais

Rodar da raiz do `web/`:

```bash
bun install                  # instala todos os workspaces
bun run dev                  # --filter '*' --parallel — todos os apps nas suas portas
bun run build                # --filter '*' — todos os apps + packages
bun run check                # biome check --write em todos os packages
bun run format               # biome format --write em todos os packages
```

Trabalho em um app específico:

```bash
cd apps/<name>
bun run dev
bun run build
bun run check
bun run format
bun run preview              # servir build de produção local
```

Cada app tem seu `biome.json`, `rsbuild.config.ts`, `uno.config.ts`, `postcss.config.mjs`, `railway.json`. Raiz `biome.json` é fonte de verdade de formatação.

## Fontes de verdade (`.kiro` e steering)

O `web/` **não tem `.kiro/` próprio**. As specs do produto vivem em `../backend/.kiro/specs/master-sindico/` (fonte única para os 3 stacks). Convenções técnicas transversais vivem em `../backend/.kiro/steering/`:

| Steering | Aplicável ao web? | Como aplicar |
|---|---|---|
| `sdd-workflow.md` | ✅ sim | Ciclo 5 fases (Discuss → Plan → Execute → Verify → Ship) adaptado — ver seção abaixo |
| `mcp-integration.md` | ✅ sim | Context7 para docs oficiais (SolidJS, TanStack, Zod, Kobalte, UnoCSS, Axios, Biome) |
| `plugins.md` | ✅ sim | `typescript-lsp`, `figma`, `code-review`, `feature-dev`, `superpowers`, `security-guidance`, `commit-commands` |
| `security.md` | ✅ sim | "Never trust the frontend" é princípio **desta pasta** — reforço aqui embaixo |
| `testing.md` | ⚠️ parcial | TDD adaptado ao SolidJS (Vitest + `@solidjs/testing-library`); thresholds diferentes (ver abaixo) |
| `do-dont.md` | ⚠️ parcial | Várias regras aplicam; divergências documentadas aqui |
| `transaction-patterns.md` | ❌ não | Transações são problema do backend |
| `code-calisthenics.md` | ⚠️ adaptado | 9 regras adaptadas para TypeScript/SolidJS (ver abaixo) |
| `go-patterns.md`, `database.md` | ❌ não | Específicos de Go |

Spec do produto, requisitos e decisões: `../backend/.kiro/specs/master-sindico/requirements/` (auth, billing, institutional, commercial, content, assembly, cross-domain, personas-and-plans, functional-matrix, talent-bank). Quando implementar uma tela, leia o requirement correspondente **antes** de codar.

## App Architecture (o que não é óbvio de um arquivo)

Cada app segue o mesmo esqueleto — pegue `apps/auth` como referência:

- **Entry point**: `src/index.tsx`. Monta SolidJS em `#root`, envolve com `ColorModeProvider` + `ColorModeScript` (Kobalte) — efeito `ThemeSync` alterna classe `dark` em `<html>`. Instância do TanStack Router em `RouterProvider`.
- **Routing**: TanStack Solid Router em modo **file-based**. Rotas em `src/routes/`. `src/routeTree.gen.ts` é **gerado** por `@tanstack/router-plugin/rspack` em build/dev — **nunca editar à mão**, é regenerado do filesystem.
  - Config do plugin (em `rsbuild.config.ts`): `target: "solid"`, `autoCodeSplitting: true`, `routeFileIgnorePrefix: "-"`, `quoteStyle: "single"`.
  - Layout routes usam prefixo underscore: `_auth/sign-in.tsx` renderiza sob o layout pathless `_auth.tsx`.
- **Styling**: UnoCSS compilado via `@unocss/postcss`. `src/main.css` emite `@unocss preflights;` + `@unocss;` + variáveis CSS no `:root` (design tokens). Tema em `uno.config.ts` lê essas variáveis — dark mode é CSS-variable driven.
- **Env vars**: declaradas em `src/env.d.ts`, expostas por Rsbuild como `import.meta.env.*`. Raiz `.env` é referência dev.

Apps atualmente **não têm auth/query providers** wireados — `index.tsx` mostra `AuthProvider` / `QueryClientProvider` comentados. Ao implementar auth, o context vive **dentro de cada app** (regra herdada: "Auth context vive em cada app, não no core"), nunca em package compartilhado.

## Shared Packages

`packages/ui` e `packages/schemas` declaram `"main": "src/index.ts"` e `"types": "src/index.ts"` — apps importam direto do TypeScript source via workspace link, sem build intermediário. Os scripts `tsdown build`/`dev` existem para produzir `dist/` quando um consumidor externo quiser, mas **apps locais não dependem disso**.

### `packages/ui`

Primitivos baseados em Kobalte + CVA. `Button` (`src/button.tsx`) é o template:

- `cva` para variant/size mapeadas para shortcuts UnoCSS (`btn-primary`, `btn-sm`…)
- Kobalte `PolymorphicProps` para suporte à prop `as`
- `splitProps` separando estilos da raiz Kobalte
- `cn` em `src/lib/utils.ts` — espaço-join mínimo, sem `clsx`/`tailwind-merge`

Ao criar novo componente compartilhado:
1. Seguir o pattern do `Button` (CVA + Kobalte + `splitProps`)
2. Re-exportar de `packages/ui/src/index.ts`
3. Adicionar `"ui": "*"` nas deps do app que usar

### `packages/schemas`

Zod schemas compartilhados entre apps. Importar de `"schemas"` (workspace alias), **nunca** por path relativo.

- Um arquivo por domínio: `auth.ts`, `profile.ts`, `billing.ts`, etc
- Re-exportar tudo de `src/index.ts`
- Schemas **espelham os tipos validados no backend** — se o backend valida `email` com regex X, o Zod schema usa a mesma regra
- **Never trust the frontend** — schemas daqui são para UX (feedback imediato), backend revalida tudo

## UnoCSS — convenções

`uno.config.ts` é quase idêntico entre apps. Pontos que afetam markup:

- **Attributify** com `prefix: "un-"` e `prefixedOnly: true` — atributo é `<div un-flex un-items-center>`, **nunca** `<div flex items-center>`
- **Icons** com prefixo `i-` (`presetIcons`): `<i class="i-lucide-home" />`
- **Shortcuts** são o design system — **reuse** em vez de utilitários crus:
  - Buttons: `btn-base`, `btn-primary`, `btn-secondary`, `btn-outline`, `btn-ghost`, `btn-link` × sizes `btn-sm | btn-md | btn-lg | btn-icon`
  - Typography: `text-display-h1..h3` (Michroma), `text-body`, `text-small`, `text-muted`
  - Layout: `flex-center`, `flex-between`, `glass-panel`, `bg-grid`
  - LP: `lp-blob`, `lp-section`, `lp-heading`, `lp-subheading`, `lp-cta`, `lp-cta-outline`
- **Fonts**: `font-sans` = Manrope (body), `font-display` = Michroma (headings, `letter-spacing: 0.05em`). Loaded via `presetWebFonts` do Google
- **Color tokens**: CSS variables em `src/main.css` — `--primary`, `--secondary`, `--accent`, `--surface`, `--surface-variant`, `--destructive`, `--success`, `--warning`, `--border`, `--input`, `--ring`. Paleta `lp.*` separada para hero. **Nunca** hardcode hex

Paleta é **imutável** (decisão de produto). Mudança requer aprovação explícita.

## Workflow SDD adaptado ao frontend

O ciclo de 5 fases do backend (`../backend/.kiro/steering/sdd-workflow.md`) aplica aqui com ajustes:

```
Discuss    →  Plan           →  Execute    →  Verify              →  Ship
requisito    + <verify>         TDD             tsc + biome +         PR +
+ design     + arquivos         Vitest/Testing  build + E2E           Release Notes
+ Figma      + docs oficiais    Library         + screenshots
```

### Fase Discuss (antes de qualquer código)

1. Ler requisito em `../backend/.kiro/specs/master-sindico/requirements/*.md`
2. Ler design correspondente em `design/*.md` (ex: `design/api-design.md` para contratos)
3. **Puxar o frame Figma** da tela via plugin `figma` — `/implement-design <frame>`. URL oficial: https://www.figma.com/design/pxJkmn8rzezKnCYXcA7lyL/Master-sindico-APP---UX
4. Conferir se a rota do backend existe em `internal/modules/{ctx}/infrastructure/http/routes/` — se não, **parar** e implementar backend primeiro
5. Ler código existente do app (outras telas similares para pegar pattern)
6. Checar `../backend/.kiro/STATE.md` para bloqueadores e decisões abertas

### Fase Plan (declare antes de codar)

```xml
<plan task="Onboarding Síndico - tela 3: Governance Markers">
  <files>
    apps/cms/src/routes/_onboarding/syndic/governance-markers.tsx     [novo]
    packages/ui/src/checkbox-group.tsx                                 [novo se não existir]
    packages/schemas/src/onboarding.ts                                 [editar — add 15 markers schema]
  </files>

  <docs-oficiais-via-context7>
    - solidjs/solid — topic: "reactivity createResource"
    - TanStack Solid Form — topic: "arrayField with Zod validator"
    - Kobalte Checkbox.Group
  </docs-oficiais-via-context7>

  <figma>
    Frame "Onboarding / Síndico / 3-Governance" — validar tokens, spacing, estados
  </figma>

  <verify>
    - Vitest: TestOnboardingSyndicGovernance_AllMarkersSelectable
    - Vitest: TestOnboardingSyndicGovernance_DisclaimerVisible
    - Vitest: TestOnboardingSyndicGovernance_RequiredFieldsBlockSubmit
    - Visual: screenshot matches Figma em light + dark
    - bun run check (biome) sem warning
    - tsc --noEmit sem erro
  </verify>

  <api-integration>
    PATCH /api/v1/onboarding/progress { step: 3, data: {...15 markers} }
    Uso: TanStack Query mutation + credentials: "include"
  </api-integration>
</plan>
```

### Fase Execute — TDD + ordem

1. **RED**: Vitest test descrevendo o comportamento (`render()` + `userEvent`)
2. **GREEN**: implementa até teste passar (sem over-engineer)
3. **REFACTOR**: simplifica, extrai helpers, elimina duplicação

Ordem por feature:
1. Schema Zod em `packages/schemas/`
2. Componente atômico em `packages/ui/` se reusável
3. Rota/tela em `apps/<app>/src/routes/`
4. Hook de data fetch (TanStack Query) em `apps/<app>/src/hooks/`
5. Integração com auth context se aplicável
6. Testes (por camada)

### Fase Verify

```bash
bun run check             # biome check --write
cd apps/<app> && bun run build    # typecheck + rsbuild build
cd apps/<app> && bun test          # Vitest (quando configurado)
# Opcional: E2E via Playwright MCP quando Sprint 5+
```

**Gates duros**:
- tsc sem erro (modo strict — SolidJS é TypeScript-first)
- biome sem erro nem warning
- Vitest com coverage ≥ **80%** em lógica de apps, ≥ **90%** em `packages/ui` e `packages/schemas`
- Build produção limpo (sem warning de chunks grandes)

### Fase Ship

Commit pt-BR via plugin `commit-commands`:

```
cms: Add onboarding step 3 - governance markers

Release Notes:
- Added 15 governance markers checkbox field for syndic onboarding step 3
```

## TDD & Testing

### Stack

- **Vitest** — runner (Rsbuild tem integração nativa)
- **`@solidjs/testing-library`** — render + queries
- **`@testing-library/user-event`** — simulação de interação
- **`msw`** — mock de endpoints quando testar fetching (sem mockar função diretamente — mocka a rede)

Adicionar em `apps/<app>/package.json`:

```jsonc
"devDependencies": {
  "vitest": "^2",
  "@solidjs/testing-library": "^0.8",
  "@testing-library/user-event": "^14",
  "msw": "^2",
  "jsdom": "^25"
}
```

### Coverage thresholds por tipo

| Tipo | Alvo |
|---|---|
| `packages/schemas/` (Zod) | **95%** — validação é domínio |
| `packages/ui/` (componentes) | **90%** — design system |
| `apps/*/src/hooks/` (TanStack Query, lógica) | **85%** |
| `apps/*/src/routes/` (telas) | **75%** — interação coberta por integration |
| `apps/*/src/lib/` (utils) | **90%** |
| **Global** | **≥ 80%** |

### Padrões obrigatórios

- Teste **antes** do código (Red-Green-Refactor)
- Toda forma tem teste de validação (campo obrigatório, formato inválido, submit bloqueado)
- Toda chamada a API tem teste com MSW (sucesso + 401 + 403 + 500 + timeout)
- Zero uso de `any` em teste — use generics de `testing-library`
- Screen reader: toda tela tem teste que valida ARIA labels ou roles

### Regras críticas que DEVEM ter teste de propriedade (fast-check)

- Quotas de Connect Me (morador Base: 2/ano, Pagante: 4/ano) — frontend **reflete** backend, nunca calcula
- Trial expiração — frontend **confia** no `trial_ends_at` do backend
- Visibilidade de vídeo de empresa — frontend **respeita** flag `autorizar_exibicao_votacao`
- Fração ideal em UI de votação — exibe como `DECIMAL` recebido do backend, **nunca** faz soma local com `number`

## Segurança frontend

**Never trust the frontend** é regra dura do `../backend/.kiro/steering/security.md`. A responsabilidade desta pasta é:

### Nunca fazer

- ❌ Armazenar token em `localStorage`, `sessionStorage`, `IndexedDB`, cookies não-httpOnly
- ❌ `dangerouslySetInnerHTML` — nunca, para nada. Plugin `security-guidance` bloqueia
- ❌ Validar autorização no frontend (ABAC) — só usa pra mostrar/esconder UI. Backend sempre reavalia
- ❌ Calcular valores monetários/quotas/prazos — frontend exibe o que backend retornou
- ❌ Confiar em query params, hash, URL state para decisão de segurança
- ❌ Hardcodar secret/key/token em código (biome + security-guidance pegam)
- ❌ Fazer request a domínio arbitrário via user input (SSRF indireto)
- ❌ Rodar código de string user-provided (`eval`, `new Function`, `setTimeout(string)`)

### Sempre fazer

- ✅ `credentials: "include"` em toda chamada de API — cookie httpOnly `.mastersindico.com.br` vai sozinho
- ✅ Validar com Zod toda entrada de formulário **antes** de enviar (UX)
- ✅ Validar resposta do backend com Zod (defesa em profundidade — API pode ter bug)
- ✅ Escapar strings vindas do usuário antes de renderizar (SolidJS faz por default se usar `{variable}`, não faz se usar `innerHTML`)
- ✅ CSP via headers no deploy Railway (configurar em `deploy-config.md`)
- ✅ Esconder dados sensíveis em logs de console (masks de CPF/CNPJ, token nunca)
- ✅ Revogar cache ao fazer logout — `queryClient.clear()` + redirect Zitadel

### LGPD no frontend

- Modal de consentimento antes de qualquer ação que revela dados pessoais (Connect Me)
- Banner de cookies/tracking conforme regra do produto
- Zero coleta de dados analytics sem opt-in explícito
- Logs do console **nunca** contêm PII em produção (`LOG_LEVEL` controla via `import.meta.env.PROD`)

## Code Calisthenics adaptado a TypeScript/SolidJS

As 9 regras de `../backend/.kiro/steering/code-calisthenics.md` adaptadas:

1. **1 nível de indent por função** — early return, componentes pequenos, extrair children em helpers
2. **Sem `else`** — ternário (`cond ? a : b`) ou guard clause; sem `else if` encadeado (use Strategy/Map)
3. **Envolver primitivos com branded types** — `type UserId = string & { __brand: "UserId" }`; Zod `z.brand<"UserId">()`
4. **First-class collections** — `type VoteCollection = { readonly votes: Vote[]; add(v: Vote): VoteCollection; winner(): Option<OptionId> }` — imutável
5. **Um ponto por linha** — `user.profile.address.city` viola Demeter; método no agregado (`user.cityName()`)
6. **Não abreviar** — `subscription`, não `sub`; `condominium`, não `condo`. Exceções idiomáticas: `e` (event), `i` (índice em loop curto), `ctx` (contexto)
7. **Componentes pequenos** — máx ~100 linhas JSX; acima: extrair sub-componentes
8. **Máx 3-5 props em componente** — mais: `props: { title; description; actions; children }` consolidado, ou component composition
9. **Sem setters/getters** — SolidJS já força signals; mutação via `setState`/`setStore` controlada

Plugin `code-review` valida no PR. `security-guidance` pega violações de XSS automaticamente.

## Plugins Claude Code relevantes ao web

Habilitados via `../backend/.claude/settings.json` (workspace-level) ou em `~/.claude/settings.json` (user-level):

| Plugin | Uso no web |
|---|---|
| `typescript-lsp` | ✅ obrigatório — diagnostics em tempo real em `.tsx`/`.ts` |
| `figma` | ✅ sempre que implementar tela — `/implement-design <frame>` |
| `security-guidance` | ✅ pre-tool hook detecta `innerHTML`, `dangerouslySetInnerHTML`, `eval` |
| `code-review` | ✅ no PR — avalia com scoring de confiança |
| `feature-dev` | ✅ agents de exploration/design/review na fase Discuss/Plan/Verify |
| `superpowers` | ✅ `/brainstorm`, `/write-plan`, TDD skills red/green |
| `commit-commands` | ✅ `/commit` pt-BR |
| `claude-md-management` | ✅ mantém este CLAUDE.md e AGENTS.md coerentes |
| `context7` | ✅ docs oficiais: SolidJS, TanStack, Zod, Kobalte, UnoCSS, Axios, Biome, Rsbuild |
| `playwright` | ⏳ Sprint 5+ — E2E do fluxo OIDC + Stripe Checkout + criação de condomínio |
| `ui-ux-pro-max` (skill community) | 🟡 opcional — `/plugin marketplace add nextlevelbuilder/ui-ux-pro-max-skill` + install; orienta design tokens mas não conhece SolidJS/UnoCSS diretamente |

## MCP e credenciais

Os MCPs usados pelo agente (Context7, Postgres, GitHub, AWS Docs, Obsidian) estão configurados em `../backend/.mcp.json` + `../backend/.claude/settings.json`. Quando abrir Claude Code em `web/` especificamente:

- Se workspace root é `backend/`, MCPs carregam. Ótimo.
- Se workspace root é `web/`, precisa replicar `.mcp.json` + `.claude/settings.json` aqui (ou abrir Claude Code de `../Development/` root para ambos ficarem ativos)

Env vars relevantes ficam em `../backend/.env` (gitignored). Para credenciais **específicas do web** (ex: `FIGMA_ACCESS_TOKEN` no fluxo de design-to-code), já estão no mesmo `.env`.

## Consumo da API backend

Todo endpoint do backend em `http://localhost:8000/api/v1/*` (dev) ou `https://api.mastersindico.com.br/api/v1/*` (prod). Acessado via:

```typescript
import axios from "axios";

const api = axios.create({
  baseURL: import.meta.env.PUBLIC_API_URL,  // http://localhost:8000 em dev
  withCredentials: true,                     // cookie httpOnly vai automaticamente
  timeout: 10_000,
});

api.interceptors.response.use(
  (r) => r,
  (error) => {
    if (error.response?.status === 401) {
      window.location.href = "/auth/login";
    }
    return Promise.reject(error);
  }
);
```

Response format padrão (backend):

```typescript
type ApiResponse<T> = { success: true; data: T };
type ApiError = { success: false; error: { code: string; message: string } };
```

Validar response com Zod (`packages/schemas/`) **antes** de usar em UI — backend pode ter bug.

## Convenções finais

- **Idioma**: UI copy, comentários, commit messages em **pt-BR**. Identificadores e filenames em **inglês**
- **Imports**: absolutos via alias configurado em `tsconfig.json` quando disponível; relativos se curtos
- **Never trust the frontend** — espelho reverso: toda regra de negócio vive no backend; frontend exibe
- **Visibilidade real** — vídeos de empresa escondidos de moradores exceto em votação de fornecedor ativa (`autorizar_exibicao_votacao=true`); frontend não cacheia lista stale após votação fechar
- **Paleta imutável** — ver `main.css`. Mudança = aprovação explícita
- **Context por app** — auth context, query client, onboarding progress — cada app tem o seu. Compartilhar **só** pure functions via `packages/core` (quando criado) ou diretamente `packages/schemas`/`packages/ui`

## Troubleshooting

| Sintoma | Provável causa |
|---|---|
| `routeTree.gen.ts` quebrado em build | Editou à mão — apagar e rodar `rsbuild dev` para regenerar |
| Tema não troca dark/light | `ThemeSync` effect não montado; verifique ordem em `index.tsx` |
| `bun run dev` erro de porta | Porta hardcoded em `package.json` está em uso — `lsof -i :3000` |
| Biome reclama de import order | `bun run check` reorganiza automaticamente |
| `un-flex` não aplica | Esqueceu `prefixedOnly: true` no `uno.config.ts`? Ou escreveu `flex` sem prefix |
| Workspace não resolve `"ui"` | `bun install` na raiz resolve; se persiste, rebuild `packages/ui/dist/` |
| Cookie não vai para `/api/v1/*` em dev | Cookie tem `Domain=.mastersindico.com.br`; dev em `localhost` precisa config separada no backend (CORS + cookie sem Domain attribute) |
