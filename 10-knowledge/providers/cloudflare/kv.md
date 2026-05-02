---
title: Cloudflare KV — Global eventually-consistent key-value
type: note
tags:
  - provider
  - cloudflare
  - kv
  - key-value
  - cache
  - edge
category: edge-storage
doc-oficial: https://developers.cloudflare.com/kv/
doc-consulted: 2026-04-24
created: 2026-04-24
updated: 2026-04-24
aliases:
  - KV
  - CF KV
  - Workers KV
---

# Cloudflare KV

**Categoria**: Global eventually-consistent key-value store acessível via binding em Workers.

**Intenção**: dado pequeno, com leitura frequentíssima, tolerante a staleness de ~60s globalmente. Ideal para config, feature flags, session metadata leve, cache de resposta.

## Quando usar

1. **Feature flags / remote config** — toggle rápido sem deploy.
2. **Cache de resposta API** com TTL curto (layer L2 global entre Cache API L1 e origem).
3. **Session metadata leve** (user preferences, A/B test assignment).
4. **Lookup de IDs → metadata** (ex: shortener slug → destino).
5. **Geofencing / country-based config**.

## Quando **NÃO** usar

- **Qualquer dado que precise consistência forte** (autz, billing state, counters) — ver [[../../runtime-antipatterns/op-008-autorizacao-apenas-cache-hit|OP-008]].
- **Dado volumoso por key** (> 25 MB) — limit.
- **High-write rate** — KV otimizado para read-heavy; escrever 1000/s por key vira custo + latência alta.
- **Strict session store** — usar [[durable-objects|Durable Objects]] ou D1.
- **Transações multi-key** — sem suporte.

## Padrão canônico

```toml
# wrangler.toml
[[kv_namespaces]]
binding = "CONFIG"
id = "abc123..."
```

```ts
interface Env { CONFIG: KVNamespace }

// Read (type hints opcionais)
const value = await env.CONFIG.get('feature:new-ui', { type: 'json' })
const text  = await env.CONFIG.get('welcome-msg', { type: 'text' })
const bin   = await env.CONFIG.get('blob', { type: 'arrayBuffer' })

// Read com metadata
const { value, metadata } = await env.CONFIG.getWithMetadata<{ rolloutPercent: number }>('feature:x')

// Write
await env.CONFIG.put('feature:new-ui', JSON.stringify(true), {
  expirationTtl: 3600,                    // TTL em segundos
  metadata: { updatedBy: 'admin-123' }
})

// Write com expiration absoluto
await env.CONFIG.put('promo-code:XMAS', '20%off', {
  expiration: Math.floor(Date.now() / 1000) + 86400 * 7
})

// Delete
await env.CONFIG.delete('old-key')

// List (paginado)
const list = await env.CONFIG.list({ prefix: 'feature:', limit: 1000 })
for (const entry of list.keys) {
  console.log(entry.name, entry.expiration, entry.metadata)
}
// list.list_complete = true quando não há mais cursors
```

## CLI

```bash
# Criar namespace
npx wrangler kv namespace create my-config
# → id para wrangler.toml

# Put / get / delete (ótimo para setup inicial)
npx wrangler kv key put "feature:new-ui" "true" --binding=CONFIG
npx wrangler kv key get "feature:new-ui" --binding=CONFIG
npx wrangler kv key delete "feature:new-ui" --binding=CONFIG

# Bulk via JSON
npx wrangler kv bulk put ./flags.json --binding=CONFIG
```

## Limites (consultada 2026-04-24)

| Limite | Valor |
|---|---|
| Value size | 25 MB |
| Key length | 512 bytes |
| Metadata per key | 1024 bytes |
| Namespaces por conta | 1000 |
| Keys por namespace | Ilimitado |
| Writes por namespace | 1/sec por key (soft — bursts aceitos) |
| **Propagation global** | ~60s eventual consistency |

**Consistência**: write no PoP A → propagação para outros PoPs leva até 60s. Nunca assumir consistência forte.

## Preço (consultada 2026-04-24)

| Dimensão | Free | Paid |
|---|---|---|
| Reads | 100k/dia | $0.50 / 1M |
| Writes | 1k/dia | $5.00 / 1M |
| Deletes | 1k/dia | $5.00 / 1M |
| Lists | 1k/dia | $5.00 / 1M |
| Storage | 1 GB | $0.50 / GB-mo |

**Bias em leitura**: reads 10× mais baratos que writes. KV é otimizado pra read-heavy.

## Antipatterns específicos

- **Session store com KV** — eventual consistency 60s = usuário que loga em PoP A pode não aparecer em PoP B por 1min. Use [[durable-objects]] ou cookie signed.
- **Contador atômico via `get → increment → put`** — race condition ([[../../runtime-antipatterns/op-001-race-condition-contador|OP-001]]). Use DO.
- **Autorização baseada em KV stale** — ver [[../../runtime-antipatterns/op-008-autorizacao-apenas-cache-hit|OP-008]]. Incluir hash de permissões na key, ou re-validar em fonte de verdade.
- **Hot writes na mesma key** — 1 write/sec soft limit; burst curto OK, sustained high-write-rate degrada. Use DO ou Queues.
- **Usar como cache de HTTP response** — pagará 2× (KV + Cache API). Prefira Cache API (grátis) pra response cache; KV para "config que muda raramente".
- **List recursiva enorme** — cada 1k keys é 1 list op. Se lista tem 1M keys, são 1000 list ops ($5 cada 1M). Shardear ou manter index paralelo.
- **Key com JSON gigante na key** — ver [[../../runtime-antipatterns/op-007-cache-key-json-gigante|OP-007]]. Hash determinístico.

## Relações

- **Combina com**: [[workers]] (binding), Cache API (L1 + L2 tier pattern — ver [[patterns]] §2)
- **Alternativas**:
  - **[[durable-objects]]** — quando precisa consistência forte ou state stateful
  - **[[d1]]** — quando dado é relacional ou precisa queries
  - **Redis externo** — quando precisa pub/sub, streams, ou consistência forte com baixa latência

## Fontes

- [KV docs](https://developers.cloudflare.com/kv/) (consultada 2026-04-24)
- [KV Limits](https://developers.cloudflare.com/kv/platform/limits/) (consultada 2026-04-24)
- [KV Pricing](https://developers.cloudflare.com/kv/platform/pricing/) (consultada 2026-04-24)
- [How KV works](https://developers.cloudflare.com/kv/concepts/how-kv-works/) (consultada 2026-04-24)

## Links

- [[_moc]]
- [[workers]]
- [[patterns]] §2 (Cache API + KV tier)
- [[../../runtime-antipatterns/op-005-cache-sem-versionamento]]
- [[../../runtime-antipatterns/op-008-autorizacao-apenas-cache-hit]]
