---
title: Cloudflare Workers — Edge serverless (V8 isolates)
type: note
tags:
  - provider
  - cloudflare
  - workers
  - edge
  - serverless
  - compute
category: edge-compute
doc-oficial: https://developers.cloudflare.com/workers/
doc-consulted: 2026-04-24
created: 2026-04-24
updated: 2026-04-24
aliases:
  - Workers
  - CF Workers
---

# Cloudflare Workers

**Categoria**: Edge serverless compute.

**Intenção**: executar JavaScript/TypeScript/WASM em 330+ PoPs globais, com cold start <50ms e cobrança por request.

## Quando usar

1. **API HTTP com latência baixa global** — resposta <50ms em qualquer região.
2. **Middleware / BFF edge** — auth, rate limit, A/B test, rewrite antes do origem.
3. **Webhook handlers** (Stripe, GitHub) — resposta rápida + fan-out para Queues/DOs.
4. **Transformação leve** — manipular responses de APIs terceiras, API gateway, protocol translation.
5. **Cron jobs globais** via `scheduled` trigger (alternativa a AWS EventBridge).

## Quando **NÃO** usar

- **CPU > 30s por request** (paid tier limit) → Railway/ECS/Lambda.
- **Stateful longa-duração** (> 10s em memória entre requests) → Durable Objects ou serviço próprio.
- **Workloads com libs binárias Node nativas** (sharp, canvas, puppeteer completo) → Lambda containers ou Railway.
- **GPU/ML training** → GPU infra dedicada.

## Padrão canônico

`wrangler.toml`:
```toml
name = "my-api"
main = "src/index.ts"
compatibility_date = "2026-04-24"
compatibility_flags = ["nodejs_compat"]

[observability]
enabled = true
head_sampling_rate = 1.0

[[routes]]
pattern = "api.example.com/*"
custom_domain = true
```

`src/index.ts` com Hono (framework preferido):
```ts
import { Hono } from 'hono'

interface Env {
  API_TOKEN: string            // secret
  LOG_LEVEL: string            // var
}

const app = new Hono<{ Bindings: Env }>()

app.get('/health', (c) => c.json({ ok: true }))

app.get('/users/:id', async (c) => {
  const id = c.req.param('id')
  // ... lógica
  return c.json({ id })
})

// Scheduled handler (cron)
export default {
  fetch: app.fetch,
  async scheduled(event: ScheduledEvent, env: Env): Promise<void> {
    // Rodar a cada cron definido em wrangler.toml [triggers.crons]
    console.log('tick', event.scheduledTime)
  }
}
```

## Limites/quotas reais (consultada 2026-04-24)

| Limite | Free | Paid ($5/mo) |
|---|---|---|
| Requests | 100k/dia | 10M incluso + $0.30/M extra |
| CPU time / req | 10ms | 30s |
| Memory / isolate | 128MB | 128MB |
| Script size | 1MB | 10MB (compressed) |
| Subrequests / req | 50 | 1000 |
| Environment vars | 64 | 128 |
| Bindings por tipo | 1000 | 1000 |
| Workers / conta | 100 | 500 |

**Gotchas**:
- **CPU time ≠ wall time**: await em I/O não consome CPU. 30s CPU = 30s *fazendo cálculo*. Wall time pode ser muito mais (limite implícito ~5min).
- **Memory 128MB fixo**: não dá pra aumentar. Grandes payloads devem ser streaming ou offload (R2).
- **Subrequests 50/1000**: chama para APIs externas + fetch interno contam. Excede → erro.
- **Log rate**: free tem retenção curta; paid + `[observability]` dá 7 dias query-able.

## Antipatterns específicos

- **Bloquear event loop com CPU puro** (hash pesado, crypto custom) — use WASM ou offload para Queue consumer.
- **Cold start pessimism** — <50ms típico; para SLO <10ms determinístico usar paid + "always warm" (cron ping cada 1min).
- **Reinstanciar lib pesada em cada request** — bindings e clientes leves na top-level do module (são reusados entre requests do mesmo isolate).
- **Armazenar estado em variável global** — isolate pode morrer; use KV/DO/storage.

## Bindings disponíveis

Declarados em `wrangler.toml`, acessados via `env.NAME`:

- `[[r2_buckets]]` — [[r2|R2 Bucket]]
- `[[kv_namespaces]]` — [[kv|KV Namespace]]
- `[[d1_databases]]` — [[d1|D1 Database]]
- `[[durable_objects.bindings]]` — [[durable-objects|Durable Objects]]
- `[[queues.producers]]` / `[[queues.consumers]]` — Queues (lazy)
- `[[services]]` — Service Bindings (Worker-to-Worker direto, zero latência)
- `[[vars]]` — public env vars
- `wrangler secret put` — secrets encriptados

## MCP / Tooling

- **`bindings.mcp.cloudflare.com`** — explorar bindings programaticamente.
- **`observability.mcp.cloudflare.com`** — logs + analytics em tempo real.
- **`docs.mcp.cloudflare.com`** — referência da API.
- **Wrangler tail**: `npx wrangler tail` — stream de logs local.

## Relações

- **Combina com**: todos os bindings CF ([[r2]], [[kv]], [[d1]], [[durable-objects]])
- **Complementa**: [[pages]] (Pages = Workers + static); [[tunnel]] (Worker → origem atrás de Tunnel)
- **Alternativas**: [[../railway/_moc|Railway]] (compute long-running); AWS Lambda (regional, maior limite)

## Fontes

- [Workers docs](https://developers.cloudflare.com/workers/) (consultada 2026-04-24)
- [Workers Limits](https://developers.cloudflare.com/workers/platform/limits/) (consultada 2026-04-24)
- [Workers Pricing](https://developers.cloudflare.com/workers/platform/pricing/) (consultada 2026-04-24)
- [V8 isolates — Kenton Varda blog](https://blog.cloudflare.com/cloud-computing-without-containers/) (consultada 2026-04-24)
- [Hono (framework preferido)](https://hono.dev) (consultada 2026-04-24)

## Links

- [[_moc]]
- [[overview]]
- [[integration-guide]]
- [[patterns]]
- [[antipatterns]]
