---
title: Cloudflare Pages — Static + Edge Functions
type: note
tags:
  - provider
  - cloudflare
  - pages
  - static-site
  - edge
category: edge-deploy
doc-oficial: https://developers.cloudflare.com/pages/
doc-consulted: 2026-04-24
created: 2026-04-24
updated: 2026-04-24
aliases:
  - Pages
  - CF Pages
  - Workers Pages
---

# Cloudflare Pages

**Categoria**: Edge deploy para static sites + edge functions. Desde 2024, convergido com Workers (branding "Workers Pages"); Pages é essencialmente Workers com Git integration + static asset serving built-in.

## Quando usar

1. **Static site com Git deploy** (SPA, Astro, Next.js static, Hugo, VitePress, Docusaurus).
2. **Fullstack app** com static + edge functions (Next.js SSR on Workers, Remix on Workers, SvelteKit).
3. **Preview deployments** por PR (automático com Git integration).
4. **Custom domain + cert automático** sem configurar CDN separadamente.

## Quando **NÃO** usar

- **Build time > 20min** → Railway ou self-hosted CI.
- **Output > 25MB por arquivo** (limit hard) ou > 20k arquivos → split ou R2 direct.
- **Next.js com todas features** (ISR on-demand complexo, middleware com side-effects pesados) → Vercel às vezes tem mais paridade oficial.
- **App precisa rodar em container** (Python backend completo) → Railway/Render.

## Padrão canônico

### Modo 1: Git integration (recomendado)

Dashboard: `Pages → Create project → Connect to Git → <org>/<repo>`.

Configurar no dashboard:
- **Build command**: `npm run build` / `pnpm build` / etc.
- **Build output dir**: `dist/` / `.next/` / `build/`
- **Environment vars**: (equivalente a `wrangler.toml [vars]`)
- **Compatibility flags** (quando usar edge functions): `nodejs_compat`

Deploy automático em push. Preview URL por PR.

### Modo 2: Direct upload via wrangler

```bash
npx wrangler pages deploy dist/ --project-name=my-app
```

Pra CI sem Git integration.

### Edge Functions (dentro do projeto)

```
my-app/
├── dist/                        # build output (static)
├── functions/                   # edge functions (cada arquivo vira Worker)
│   ├── api/
│   │   └── users/[id].ts        # /api/users/:id
│   └── _middleware.ts           # roda em toda request
└── _routes.json                 # opcional — allowlist
```

`functions/api/users/[id].ts`:
```ts
export const onRequestGet: PagesFunction = async ({ params, env, request }) => {
  return Response.json({ id: params.id })
}

export const onRequestPost: PagesFunction = async ({ request, env }) => {
  const body = await request.json()
  return Response.json({ created: body })
}
```

## `_routes.json`

Define quais paths vão para funções vs static:

```json
{
  "version": 1,
  "include": ["/api/*"],
  "exclude": ["/static/*"]
}
```

Otimização crítica: paths em `exclude` nunca invocam function → latência de static asset puro.

## Limites (consultada 2026-04-24)

| Limite | Free | Paid ($20/mo Pages tier) |
|---|---|---|
| Sites | Ilimitado | Ilimitado |
| Builds | 500/mo | 5000/mo |
| Build time | 20 min | 20 min |
| Concurrent builds | 1 | 5 |
| Files | 20k | 20k |
| File size | 25MiB | 25MiB |
| Custom domains | 100/projeto | 100/projeto |
| Bandwidth | Ilimitado (eyeball) | Ilimitado (eyeball) |

**Functions** seguem limites de Workers (10ms free / 30s paid CPU).

## Antipatterns específicos

- **Build muito grande bate 20k files limit** — gzip não ajuda; consolidar assets, mover binários pro R2.
- **Framework SSR puro Node** (não-edge compatible) — Pages é edge; APIs Node nativas falham. Usar `nodejs_compat` flag + verificar lib por lib.
- **Usar Pages para app que precisa Worker dedicado** — quando você tem 50+ edge functions, considerar migrar para Workers standalone (mais controle sobre bindings/config).
- **Misturar Pages + custom Worker na mesma rota** — confusão; escolher um. Service Bindings Pages → Worker está em beta.

## Migration path: Next.js

- **Static export** (`next build && next export`): use `dist/` como output — Pages serve igual a qualquer SPA.
- **SSR / App Router**: usar `@cloudflare/next-on-pages` (`npm i -D @cloudflare/next-on-pages`). Gera output compatível com Pages Functions. Compatibilidade amplia a cada release; casos complexos (ISR on-demand, middleware com side-effects) podem falhar.
- **Alternativa**: `wrangler.toml` + `nodejs_compat` + deploy Next.js como Worker puro (sem Pages). Mais controle, mais config manual.

## Relações

- **Combina com**: [[workers]] (functions compartilham runtime); [[r2]] (assets grandes); [[access]] (proteger preview deploys com auth)
- **Complementa**: [[tunnel]] (origin atrás de Tunnel se Pages precisa fallback a servidor interno)
- **Alternativas**: [[../railway/_moc|Railway]] (deploy backend + DB unificado); Vercel; Netlify

## Fontes

- [Pages docs](https://developers.cloudflare.com/pages/) (consultada 2026-04-24)
- [Pages Functions](https://developers.cloudflare.com/pages/functions/) (consultada 2026-04-24)
- [Pages Limits](https://developers.cloudflare.com/pages/platform/limits/) (consultada 2026-04-24)
- [@cloudflare/next-on-pages](https://github.com/cloudflare/next-on-pages) (consultada 2026-04-24)

## Links

- [[_moc]]
- [[workers]]
- [[integration-guide]]
- [[../railway/_moc]] — alternativa PaaS
