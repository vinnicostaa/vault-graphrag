---
title: Cloudflare — Patterns (cross-produto)
type: pattern
tags:
  - provider
  - cloudflare
  - patterns
  - edge
doc-consulted: 2026-04-24
created: 2026-04-24
updated: 2026-04-24
aliases:
  - CF patterns
  - Cloudflare patterns
---

# Cloudflare — Patterns (cross-produto)

Este arquivo cobre **apenas padrões que atravessam 2+ produtos Cloudflare**. Gotchas específicos de 1 produto vão na nota atômica do produto (ajuste #3 do dual-check).

## 1. Binding injection + tipagem forte

**Problema**: Worker precisa consumir múltiplos produtos (R2 + KV + D1 + DO). Cada um tem SDK próprio; gerenciar clients manualmente vira boilerplate.

**Pattern**: declarar todos em `wrangler.toml` → `@cloudflare/workers-types` gera tipos → code recebe `env` tipado.

```ts
// src/bindings.ts — interface centralizada
export interface Bindings {
  // Vars
  API_BASE_URL: string
  LOG_LEVEL: 'debug' | 'info' | 'warn' | 'error'
  // Secrets
  STRIPE_WEBHOOK_SECRET: string
  // Bindings
  UPLOADS: R2Bucket
  CACHE: KVNamespace
  DB: D1Database
  RATE_LIMITER: DurableObjectNamespace
  EVENTS_OUT: Queue<OutboundEvent>
}

// Hono (ou qualquer framework)
import { Hono } from 'hono'
const app = new Hono<{ Bindings: Bindings }>()

app.get('/users/:id', async (c) => {
  const row = await c.env.DB.prepare('SELECT ... WHERE id = ?')
                   .bind(c.req.param('id'))
                   .first()
  await c.env.CACHE.put(`user:${c.req.param('id')}`, JSON.stringify(row), { expirationTtl: 300 })
  return c.json(row)
})
```

**Vantagens**: zero auth/network para bindings (canal local no PoP), rotação de credenciais não toca código, compilador pega erros.

## 2. Edge cache + KV combo (2 tiers)

**Problema**: precisa cache global com latência sub-10ms mas valores mudam raramente.

**Pattern**: Cache API (PoP-local) como L1 + KV (eventual consistency global) como L2.

```ts
async function getConfig(env: Bindings, key: string): Promise<Config> {
  const cacheUrl = new URL('https://cache.internal/config/' + key)
  const cache = caches.default

  // L1: cache do PoP atual
  let resp = await cache.match(cacheUrl)
  if (resp) return resp.json()

  // L2: KV (eventual consistency ~60s globalmente)
  const cached = await env.CACHE.get(key, { type: 'json' })
  if (cached) {
    // Popula L1 pro PoP atual; TTL curto
    ctx.waitUntil(cache.put(cacheUrl, new Response(JSON.stringify(cached), {
      headers: { 'Cache-Control': 'max-age=60' }
    })))
    return cached
  }

  // L3: origem (D1 ou API externa)
  const fresh = await fetchFromOrigin(key)
  ctx.waitUntil(env.CACHE.put(key, JSON.stringify(fresh), { expirationTtl: 600 }))
  ctx.waitUntil(cache.put(cacheUrl, new Response(JSON.stringify(fresh), {
    headers: { 'Cache-Control': 'max-age=60' }
  })))
  return fresh
}
```

**Por que 2 tiers**: L1 evita hit em KV (~5-30ms). L2 evita hit em origem (~50-500ms). Cada tier com TTL mais curto que o anterior.

Referência runtime: [[../../runtime-antipatterns/op-005-cache-sem-versionamento|OP-005]] (regra de versionamento vale aqui também).

## 3. Wrangler env promotion (dev → staging → prod)

**Problema**: evitar config drift entre ambientes.

**Pattern**: único `wrangler.toml` com sections `[env.staging]`, `[env.prod]`. Deploy seleciona env com flag.

```bash
# Dev local
npx wrangler dev

# Staging (auto em PR)
npx wrangler deploy --env staging

# Prod (protegido; main branch ou tag)
npx wrangler deploy --env prod
```

Secrets são **por env**: `wrangler secret put KEY --env staging` vs `--env prod`.

Nunca reusar bucket/DB entre envs:
```toml
[[env.staging.r2_buckets]]
binding = "UPLOADS"
bucket_name = "my-app-uploads-staging"  # distinto de prod

[[env.prod.r2_buckets]]
binding = "UPLOADS"
bucket_name = "my-app-uploads-prod"
```

## 4. Durable Objects + WebSocket com state

**Problema**: coordenar WebSockets (chat room, collaborative edit, live leaderboard) sem database polling.

**Pattern**: 1 DO por "room" — único isolate por chave, mantém WebSocket conexões + estado em memória.

```ts
export class ChatRoom {
  private sessions: Set<WebSocket> = new Set()

  constructor(private state: DurableObjectState, private env: Bindings) {}

  async fetch(request: Request): Promise<Response> {
    const upgradeHeader = request.headers.get('Upgrade')
    if (upgradeHeader !== 'websocket') return new Response('Expected WebSocket', { status: 426 })

    const pair = new WebSocketPair()
    const [client, server] = Object.values(pair)
    server.accept()
    this.sessions.add(server)

    server.addEventListener('message', async (evt) => {
      // Broadcast to all in this room
      for (const ws of this.sessions) {
        if (ws.readyState === WebSocket.OPEN) ws.send(evt.data)
      }
      // Persist for replay
      const messages = (await this.state.storage.get<any[]>('messages')) ?? []
      messages.push({ data: evt.data, at: Date.now() })
      await this.state.storage.put('messages', messages.slice(-100))  // keep last 100
    })

    server.addEventListener('close', () => this.sessions.delete(server))
    return new Response(null, { status: 101, webSocket: client })
  }
}

// Worker rota para o DO correto por nome da sala
export default {
  async fetch(req: Request, env: Bindings): Promise<Response> {
    const url = new URL(req.url)
    const roomName = url.pathname.split('/')[2]
    const id = env.CHAT_ROOM.idFromName(roomName)
    return env.CHAT_ROOM.get(id).fetch(req)
  }
}
```

**Trade-offs**:
- DO é **single-threaded per instance** — sem race conditions internas, mas hotspot possível se 1 sala escala muito.
- State in-memory perde em restart; usar `state.storage` para persistência durável.
- Hibernation API (`state.acceptWebSocket()`) reduz custo quando inativo.

## 5. R2 + signed URL para uploads direct-to-bucket

**Problema**: upload de vídeo/imagem grande passando pelo Worker consome tempo CPU (30s limit paid).

**Pattern**: Worker gera signed URL presigned; cliente faz PUT direto no bucket.

```ts
app.post('/uploads/presign', async (c) => {
  const { filename, contentType } = await c.req.json<{ filename: string; contentType: string }>()
  const key = `uploads/${crypto.randomUUID()}/${sanitize(filename)}`

  // R2 signed URL via S3-compat API (aws4fetch ou sdk)
  const url = await c.env.UPLOADS.createMultipartUpload(key, {
    httpMetadata: { contentType },
    customMetadata: { uploaderId: c.get('userId') }
  })
  return c.json({ uploadUrl: url, key })
})

app.post('/uploads/:key/complete', async (c) => {
  // Cliente avisa que terminou; registra no DB
  await c.env.DB.prepare('INSERT INTO uploads ...').bind(c.req.param('key')).run()
  return c.json({ ok: true })
})
```

Browser faz PUT direto em `uploadUrl` → não passa pelo Worker → sem limite CPU.

**Gotcha**: configurar CORS no bucket R2 via dashboard ou `wrangler r2 bucket cors put`.

## 6. Queues + DLQ para processamento assíncrono

**Problema**: webhook dispara trabalho que não cabe em 30s CPU do Worker handler.

**Pattern**: handler enfileira; consumer processa off-thread com retry automático e DLQ.

```toml
# wrangler.toml
[[queues.producers]]
binding = "EVENTS_OUT"
queue = "events"

[[queues.consumers]]
queue = "events"
max_batch_size = 10
max_batch_timeout = 5
max_retries = 3
dead_letter_queue = "events-dlq"
```

```ts
// Producer (handler de webhook)
app.post('/webhooks/stripe', async (c) => {
  const body = await c.req.text()
  // ... validar signature (OP-023)
  await c.env.EVENTS_OUT.send({ type: 'stripe.webhook', body })
  return c.json({ received: true })
})

// Consumer (função exportada queue)
export default {
  async fetch(...) { /* ... */ },
  async queue(batch: MessageBatch<Event>, env: Bindings): Promise<void> {
    for (const msg of batch.messages) {
      try {
        await processEvent(msg.body, env)
        msg.ack()
      } catch (err) {
        if (isRetryable(err)) msg.retry()  // retorna pra queue com backoff
        else msg.ack()                      // não-retentável — confirma e manda pro DLQ manualmente via .ack e log
      }
    }
  }
}
```

Combina com [[../../runtime-antipatterns/op-004-sem-retry-strategy|OP-004]] (retry strategy) e [[../../runtime-antipatterns/op-003-webhook-sem-idempotencia|OP-003]] (idempotência).

## 7. Zero Trust — Access + Tunnel para app interno

**Problema**: expor app interno (admin, grafana, staging) sem abrir porta na internet ou configurar VPN.

**Pattern**: Tunnel (cloudflared) reverse-proxy + Access aplicado sobre o hostname.

```
admin.company.com
        │
        ▼
Cloudflare Access policy — "user@company.com AND MFA"
        │  ← OAuth/OIDC
        ▼
Cloudflare Tunnel (cloudflared daemon em VM interna)
        │  ← TLS persistent connection
        ▼
localhost:3000 (admin app)
```

Ver [[access]] e [[tunnel]] (Onda 3) + [[../../security/beyond-corp]] para modelo conceitual.

## 8. Observability unificada

**Pattern**: logs via [MCP observability.mcp.cloudflare.com](https://observability.mcp.cloudflare.com/mcp), métricas no dashboard, traces via AI Gateway (se LLM) ou OTel exporter customizado.

Para Workers com logs estruturados:

```ts
function logger(env: Bindings) {
  return {
    info: (msg: string, fields: Record<string, unknown> = {}) => {
      console.log(JSON.stringify({ level: 'info', msg, ...fields, ts: Date.now() }))
    },
    error: (msg: string, err: Error, fields: Record<string, unknown> = {}) => {
      console.error(JSON.stringify({ level: 'error', msg, err: err.message, stack: err.stack, ...fields, ts: Date.now() }))
    }
  }
}
```

Com `observability.enabled = true`, logs indexáveis no dashboard via SQL-like queries.

## Fontes

- [Bindings — Workers](https://developers.cloudflare.com/workers/runtime-apis/bindings/) (consultada 2026-04-24)
- [Cache API](https://developers.cloudflare.com/workers/runtime-apis/cache/) (consultada 2026-04-24)
- [Wrangler Environments](https://developers.cloudflare.com/workers/configuration/environments/) (consultada 2026-04-24)
- [Durable Objects — WebSockets](https://developers.cloudflare.com/durable-objects/examples/websocket-hibernation-server/) (consultada 2026-04-24)
- [R2 Presigned URLs](https://developers.cloudflare.com/r2/api/s3/presigned-urls/) (consultada 2026-04-24)
- [Queues Consumer](https://developers.cloudflare.com/queues/configuration/javascript-apis/) (consultada 2026-04-24)
- [Zero Trust — Access + Tunnel](https://developers.cloudflare.com/cloudflare-one/connections/connect-networks/) (consultada 2026-04-24)

## Links

- [[_moc]]
- [[integration-guide]]
- [[antipatterns]]
- [[../../runtime-antipatterns/_moc]] — referências OP-003/004/005/023
- [[../../security/beyond-corp]]
