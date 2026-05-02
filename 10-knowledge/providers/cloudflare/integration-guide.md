---
title: Cloudflare — Integration Guide (Wrangler + bindings + deploy)
type: guide
tags:
  - provider
  - cloudflare
  - integration
  - wrangler
  - deploy
doc-consulted: 2026-04-24
created: 2026-04-24
updated: 2026-04-24
aliases:
  - CF Integration
  - Wrangler setup
---

# Cloudflare — Integration Guide

Setup end-to-end: do zero à produção. Usa **wrangler v3+** como CLI canônica (substitui `@cloudflare/wrangler` legado). Cobre Workers + Pages; produtos stateful (R2/D1/KV/DO) entram via bindings.

## 1. Pré-requisitos

- **Node.js 18+** (wrangler roda em Node; compatibilidade Bun é parcial para certos comandos).
- Conta Cloudflare (free tier é suficiente para desenvolvimento inteiro).
- `wrangler login` ou `CLOUDFLARE_API_TOKEN` + `CLOUDFLARE_ACCOUNT_ID` em env.

## 2. Setup inicial

```bash
# Instalação
npm install -D wrangler
# OU global (desaconselhado — prende versão):
npm install -g wrangler

# Login (abre browser)
npx wrangler login
# OU via token:
export CLOUDFLARE_API_TOKEN=<token-com-scope-minimo>
export CLOUDFLARE_ACCOUNT_ID=<account-id>

# Bootstrap de Worker
npx wrangler init my-worker
#   ? Would you like to use TypeScript? Yes
#   ? Would you like to create a Worker at src/index.ts? Yes
#   ? Would you like us to write your first test? No
```

Resultado: `wrangler.toml` + `src/index.ts` + `tsconfig.json` + `package.json`.

## 3. `wrangler.toml` canônico (ou `wrangler.jsonc`)

```toml
name = "my-worker"
main = "src/index.ts"
compatibility_date = "2026-04-24"           # pin de runtime behavior
compatibility_flags = ["nodejs_compat"]     # se precisa Node APIs

# Observability — OBRIGATÓRIO em prod (desativa free metrics se false)
[observability]
enabled = true
head_sampling_rate = 1.0                    # 100% sampling em dev; reduzir em prod

# Vars públicas
[vars]
API_BASE_URL = "https://api.example.com"
LOG_LEVEL = "info"

# Secrets (NÃO colocar aqui — usar `wrangler secret put`)

# Bindings — cada produto consumido
[[r2_buckets]]
binding = "UPLOADS"
bucket_name = "my-app-uploads"

[[kv_namespaces]]
binding = "CACHE"
id = "abc123..."

[[d1_databases]]
binding = "DB"
database_name = "my-app"
database_id = "xyz789..."

[[durable_objects.bindings]]
name = "RATE_LIMITER"
class_name = "RateLimiter"

[[queues.producers]]
binding = "EVENTS_OUT"
queue = "events"

[[queues.consumers]]
queue = "events"
max_batch_size = 10

# Route (prod) OU custom_domain
[[routes]]
pattern = "api.example.com/*"
custom_domain = true
```

### Ambientes (dev / staging / prod)

```toml
# wrangler.toml — top-level é default/dev

[env.staging]
vars = { LOG_LEVEL = "debug" }
[[env.staging.r2_buckets]]
binding = "UPLOADS"
bucket_name = "my-app-uploads-staging"

[env.prod]
vars = { LOG_LEVEL = "warn" }
[[env.prod.r2_buckets]]
binding = "UPLOADS"
bucket_name = "my-app-uploads-prod"
```

Deploy:
```bash
npx wrangler deploy                         # dev (default)
npx wrangler deploy --env staging
npx wrangler deploy --env prod
```

## 4. Secrets management

**Nunca** commitar secret em `wrangler.toml` — só vars públicas vão lá.

```bash
# Set secret (lê stdin — safe em CI com pipe)
npx wrangler secret put STRIPE_WEBHOOK_SECRET
# → prompta valor; sobe direto pra conta, não fica no git

# Set secret em env específico
npx wrangler secret put STRIPE_WEBHOOK_SECRET --env prod

# List secrets (nomes; valores nunca expostos)
npx wrangler secret list --env prod

# Delete
npx wrangler secret delete OLD_KEY --env prod
```

Em código (`src/index.ts`):
```ts
interface Env {
  API_BASE_URL: string                // vars
  STRIPE_WEBHOOK_SECRET: string       // secret — mesmo tipo em TS
  UPLOADS: R2Bucket                   // binding
  DB: D1Database
}
```

## 5. Local dev

```bash
npx wrangler dev                       # server local em 127.0.0.1:8787
npx wrangler dev --remote              # usa Cloudflare network (bindings reais)
npx wrangler dev --local-protocol=https --local-ip=0.0.0.0  # testar device físico
```

**Local mode**: bindings simulados (D1 via `.wrangler/state/d1/`, KV em memória, R2 local).
**Remote mode**: bindings reais, dev direto na edge — ótimo para testar integrações (WAF, Access, etc.).

## 6. D1 — migrations

```bash
# Criar DB
npx wrangler d1 create my-app
# → retorna database_id; copiar para wrangler.toml

# Criar migration
npx wrangler d1 migrations create my-app create_users_table
# → cria migrations/0001_create_users_table.sql

# Aplicar (local)
npx wrangler d1 migrations apply my-app

# Aplicar (remote prod)
npx wrangler d1 migrations apply my-app --remote --env prod

# Executar SQL ad-hoc
npx wrangler d1 execute my-app --command "SELECT COUNT(*) FROM users"
```

Schema versionado como arquivo SQL numerado (mesmo padrão de Go-migrate, knex, Flyway).

## 7. R2 — upload via CLI

```bash
npx wrangler r2 bucket create my-app-uploads
npx wrangler r2 object put my-app-uploads/path/to/file.jpg --file=./local.jpg
npx wrangler r2 object get my-app-uploads/path/to/file.jpg --file=./out.jpg
```

No código: `env.UPLOADS.put('key', body)`, `env.UPLOADS.get('key')`.

## 8. Durable Objects — deploy

DO tem **migration** obrigatória em mudanças de `class_name`:

```toml
[[durable_objects.bindings]]
name = "RATE_LIMITER"
class_name = "RateLimiter"

[[migrations]]
tag = "v1"
new_classes = ["RateLimiter"]

# Quando renomear classe:
[[migrations]]
tag = "v2"
renamed_classes = [{ from = "RateLimiter", to = "RateLimiterV2" }]
```

Runtime `class` em TypeScript:

```ts
export class RateLimiter {
  constructor(private state: DurableObjectState, private env: Env) {}

  async fetch(request: Request): Promise<Response> {
    const count = (await this.state.storage.get<number>('count')) ?? 0
    await this.state.storage.put('count', count + 1)
    return new Response(JSON.stringify({ count: count + 1 }))
  }
}

export default {
  async fetch(req: Request, env: Env): Promise<Response> {
    const id = env.RATE_LIMITER.idFromName(req.headers.get('CF-Connecting-IP')!)
    const stub = env.RATE_LIMITER.get(id)
    return stub.fetch(req)
  }
}
```

## 9. CI/CD (GitHub Actions canônico)

```yaml
# .github/workflows/deploy.yml
name: Deploy Worker
on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 20
      - run: npm ci
      - run: npm test
      - name: Deploy to staging
        if: github.ref != 'refs/heads/main'
        run: npx wrangler deploy --env staging
        env:
          CLOUDFLARE_API_TOKEN: ${{ secrets.CLOUDFLARE_API_TOKEN }}
          CLOUDFLARE_ACCOUNT_ID: ${{ secrets.CLOUDFLARE_ACCOUNT_ID }}
      - name: Deploy to prod
        if: github.ref == 'refs/heads/main'
        run: npx wrangler deploy --env prod
        env:
          CLOUDFLARE_API_TOKEN: ${{ secrets.CLOUDFLARE_API_TOKEN }}
          CLOUDFLARE_ACCOUNT_ID: ${{ secrets.CLOUDFLARE_ACCOUNT_ID }}
```

**Regra de token**: API token com scope mínimo — para Workers deploy, `Workers Scripts:Edit`, `Account:Read`, + produtos usados (D1, R2, KV, etc.). Nunca "All Resources".

## 10. Pages (static + functions)

Pages tem 2 modos:
1. **Via CLI** (`wrangler pages`): `npx wrangler pages deploy dist/`.
2. **Git integration** (recomendado): conectar repo; build rodado pela Cloudflare. Edge functions em `functions/` viram Workers automaticamente.

```
my-pages-app/
├── dist/                      # build output
├── functions/
│   └── api/
│       └── [[path]].ts        # [[catch-all]] handler
└── _routes.json               # allowlist de rotas dinâmicas
```

```ts
// functions/api/users/[id].ts
export async function onRequestGet({ params, env }) {
  return new Response(`user ${params.id}`)
}
```

## 11. Observabilidade

- **Default logs**: `wrangler tail` (stream de logs em tempo real — tier free tem retenção curta).
- **Paid**: habilitar **Workers Logs** ($1/mo por Worker) — 7 dias retenção, query SQL-like no dashboard.
- **Metrics**: dashboard default (req count, error rate, CPU time, duration).
- **Tracing**: OpenTelemetry via **workers-otel** (comunidade) ou AI Gateway para traces de LLM calls.
- **Audit**: Audit Logs API (disponível via [MCP auditlogs.mcp.cloudflare.com](https://auditlogs.mcp.cloudflare.com/mcp)).

## 12. Checklist pre-prod

- [ ] `wrangler.toml` tem `compatibility_date` explícito (não "dev").
- [ ] `observability.enabled = true` em env prod.
- [ ] API token tem scope mínimo; não é "All".
- [ ] Secrets (nunca vars) para credenciais.
- [ ] Todos os ambientes (dev/staging/prod) com buckets/DBs distintos.
- [ ] Route ou custom_domain configurado para prod.
- [ ] Migration de DO declarada (se usa DOs).
- [ ] CI rodando `wrangler deploy --env staging` em PRs e `--env prod` em main.
- [ ] Logs / alertas configurados.

## Fontes

- [Wrangler docs](https://developers.cloudflare.com/workers/wrangler/) (consultada 2026-04-24)
- [Workers Configuration (wrangler.toml)](https://developers.cloudflare.com/workers/wrangler/configuration/) (consultada 2026-04-24)
- [Environments and bindings](https://developers.cloudflare.com/workers/configuration/environments/) (consultada 2026-04-24)
- [D1 migrations](https://developers.cloudflare.com/d1/reference/migrations/) (consultada 2026-04-24)
- [Durable Objects migrations](https://developers.cloudflare.com/durable-objects/reference/durable-objects-migrations/) (consultada 2026-04-24)
- [API Tokens (least privilege)](https://developers.cloudflare.com/fundamentals/api/get-started/create-token/) (consultada 2026-04-24)

## Links

- [[_moc]]
- [[overview]]
- [[patterns]]
- [[workers]]
- [[pages]]
