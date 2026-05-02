---
title: Cloudflare Durable Objects — Stateful coordinators on edge
type: note
tags:
  - provider
  - cloudflare
  - durable-objects
  - state
  - coordination
  - websocket
  - edge
category: edge-state
doc-oficial: https://developers.cloudflare.com/durable-objects/
doc-consulted: 2026-04-24
created: 2026-04-24
updated: 2026-04-24
aliases:
  - Durable Objects
  - DOs
  - CF DOs
---

# Cloudflare Durable Objects (DOs)

**Categoria**: Stateful single-threaded coordinators on edge.

**Intenção**: isolate único por chave, com storage transacional embarcado. Resolve problemas onde KV não serve (consistency forte), D1 não serve (coordenação por chave com baixa latência), e Workers puros não servem (estado entre requests).

## Quando usar

1. **Coordenação com consistência forte por chave** — rate limiter por usuário, contador atômico, lock distribuído.
2. **WebSocket rooms** — chat, collaborative editing, live leaderboard.
3. **State machine por entidade** — 1 DO por jogo/pedido/sessão com transições guardadas.
4. **Counters / analytics realtime** com agregação por dimensão (1 DO por tenant para contagem consistente).
5. **Per-session server state** quando cookie + JWT não bastam.

## Quando **NÃO** usar

- **Estado compartilhado global sem chave natural** — viola premissa "1 DO por chave". Use D1 + eventual consistency.
- **Carga imprevisível numa única chave** (hotspot) — single-threaded = gargalo. Sharding manual ou DO sharder pattern.
- **Conteúdo muito grande por DO** — cada DO tem ~128MB memory; storage per-DO limit 10GB.
- **Quando eventual consistency ~60s serve** — use [[kv]] que é mais barato.

## Modelo mental

```
┌─────────────────────────────────────┐
│  DurableObjectNamespace             │
│                                     │
│  ┌──────────┐  ┌──────────┐         │
│  │ DO #room1│  │ DO #room2│  ...    │
│  │ (isolate)│  │ (isolate)│         │
│  │ state +  │  │ state +  │         │
│  │ storage  │  │ storage  │         │
│  └──────────┘  └──────────┘         │
└─────────────────────────────────────┘
```

- **1 classe de DO** (ex: `ChatRoom`) definida no Worker.
- **N instâncias** (1 por chave: `room-abc`, `room-xyz`).
- Cada instância vive em **1 location** (escolhido por CF baseado em first access; "jurisdictional" restringe a EU/US).
- **Single-threaded per instance** — sem race conditions internas.
- **Storage** (`state.storage`) é transacional embarcado; durável entre restarts.
- **Hibernation**: isolate adormece quando inativo; acorda no próximo request (ms).

## Padrão canônico

### Definir DO

```ts
// src/rate_limiter.ts
export class RateLimiter {
  constructor(private state: DurableObjectState, private env: Env) {}

  async fetch(request: Request): Promise<Response> {
    const now = Date.now()
    const windowMs = 60_000
    const limit = 60

    // Transactional — state.storage garante atomicidade
    return await this.state.blockConcurrencyWhile(async () => {
      const bucket = (await this.state.storage.get<number[]>('requests')) ?? []
      const filtered = bucket.filter((t) => now - t < windowMs)

      if (filtered.length >= limit) {
        return new Response('rate limit exceeded', {
          status: 429,
          headers: { 'Retry-After': '60' }
        })
      }
      filtered.push(now)
      await this.state.storage.put('requests', filtered)
      return new Response('ok', { status: 200 })
    })
  }
}
```

### Binding + migration

```toml
# wrangler.toml
[[durable_objects.bindings]]
name = "RATE_LIMITER"
class_name = "RateLimiter"

# Migration — obrigatória ao criar/renomear/remover classes
[[migrations]]
tag = "v1"
new_classes = ["RateLimiter"]
```

### Usar no Worker

```ts
export { RateLimiter } from './rate_limiter'   // tem que exportar!

export default {
  async fetch(req: Request, env: Env): Promise<Response> {
    const userId = new URL(req.url).searchParams.get('userId') ?? 'anonymous'
    const id = env.RATE_LIMITER.idFromName(userId)       // chave determinística
    const stub = env.RATE_LIMITER.get(id)                // handle
    return stub.fetch(req)                               // delega ao DO
  }
}
```

### WebSocket + Hibernation API

```ts
export class ChatRoom {
  constructor(private state: DurableObjectState) {}

  async fetch(request: Request): Promise<Response> {
    const upgrade = request.headers.get('Upgrade')
    if (upgrade !== 'websocket') return new Response('expected WebSocket', { status: 426 })

    const pair = new WebSocketPair()
    const [client, server] = Object.values(pair)

    // Hibernation API — DO adormece com WS aberto; wake-up em mensagem
    this.state.acceptWebSocket(server)
    return new Response(null, { status: 101, webSocket: client })
  }

  // Callback quando mensagem chega (wake-up)
  async webSocketMessage(ws: WebSocket, message: string | ArrayBuffer) {
    // Broadcast to all active sockets
    for (const conn of this.state.getWebSockets()) {
      conn.send(message)
    }
    // Persist replay (optional)
  }

  async webSocketClose(ws: WebSocket, code: number, reason: string, wasClean: boolean) {
    // Cleanup
  }
}
```

Hibernation mata o custo de manter milhares de rooms idle.

## Limites (consultada 2026-04-24)

| Limite | Valor |
|---|---|
| Memory per DO | 128 MB |
| Storage per DO | 10 GB |
| Storage operation throughput | ~1000 ops/s por DO |
| Object lifespan | Indefinido (persiste para sempre até explicit delete) |
| WebSocket per DO | 32k (com Hibernation) |
| DO classes por Worker | 10 |
| DO instances | Ilimitado |

## Preço (consultada 2026-04-24)

| Dimensão | Valor |
|---|---|
| Base | $5/mo (pre-requisito) |
| Requests | $0.15 / 1M (após 1M incluso) |
| Duration (active) | $12.50 / 1M GB-s |
| Storage | $0.20 / GB-mo |
| Storage reads | $0.20 / 1M |
| Storage writes | $1.00 / 1M |

## Antipatterns específicos

- **Single "global" DO com todas as keys** — hotspot. Sharde: `env.RL.idFromName(hash(userId) % 32)` → 32 shards.
- **Acessar mesmo DO de muitos Workers concorrentes sem `blockConcurrencyWhile`** — requests podem interleaving. Use-o ou `state.storage.transaction()`.
- **Armazenar binário grande em storage** — 10 GB limit + performance. Delegue para R2 e guarde apenas ref no storage.
- **Não declarar migration em `wrangler.toml`** — deploy falha. Sempre `[[migrations]]` ao criar/renomear.
- **DO para coisa que KV serve** — paga $5/mo base sem motivo. Use KV + Cache API primeiro.
- **Location lock em EU/US estrito para escala global** — tráfego do outro lado paga latência. Use jurisdictional só quando compliance exige.
- **Esquecer `export class` no index** — runtime não acha a classe → erro obscuro em prod.

## Padrões

- **Sharded counter**: N DOs por "slot" + agregação periódica → throughput ∝ N.
- **Alarm** (`state.storage.setAlarm(timestamp)`) — timer durável que dispara `alarm()` callback. Útil para agendar cleanup, rate-limit expiry, timeouts.
- **Jurisdictional** para compliance: `env.DO.newUniqueId({ jurisdiction: 'eu' })` — força a instância a ficar em PoPs EU.

## Relações

- **Combina com**: [[workers]] (binding); [[kv]] (DO para stateful, KV para config; tier L1/L2)
- **Alternativas**:
  - [[kv]] quando consistency eventual basta
  - [[d1]] quando queries relacionais importam
  - Redis externo quando pub/sub + consistency forte + baixa latência regional importa

## Fontes

- [Durable Objects docs](https://developers.cloudflare.com/durable-objects/) (consultada 2026-04-24)
- [DO Best Practices](https://developers.cloudflare.com/durable-objects/best-practices/) (consultada 2026-04-24)
- [DO Pricing](https://developers.cloudflare.com/durable-objects/platform/pricing/) (consultada 2026-04-24)
- [DO Limits](https://developers.cloudflare.com/durable-objects/platform/limits/) (consultada 2026-04-24)
- [WebSocket Hibernation](https://developers.cloudflare.com/durable-objects/examples/websocket-hibernation-server/) (consultada 2026-04-24)

## Links

- [[_moc]]
- [[workers]]
- [[kv]]
- [[patterns]] §4 (DO + WebSocket pattern)
- [[../../runtime-antipatterns/op-001-race-condition-contador]]
