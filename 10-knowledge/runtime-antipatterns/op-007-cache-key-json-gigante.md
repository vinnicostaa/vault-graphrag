---
title: OP-007 — Cache key com JSON serializado gigante
type: antipattern
tags:
  - antipattern
  - runtime
  - operational
  - cache
  - performance
id: OP-007
severity: Medium
created: 2026-04-24T00:00:00.000Z
updated: 2026-04-24T00:00:00.000Z
aliases:
  - Oversized cache key
doc-consulted: '2026-04-24'
---

# OP-007 — Cache key com JSON serializado gigante

**Severidade**: Medium
**Categoria**: Cache / Performance

## Sintoma

`search:${JSON.stringify(filters)}` — filtros complexos produzem string de 1KB+. Redis aceita até 512MB por key, mas keys enormes são patológicas: ocupam memória desproporcional, degradam throughput de hash lookups, tornam logs ilegíveis.

## Por que é problema

- **Memória**: cada key grande paga overhead (estrutura interna + cópias em replicação).
- **Throughput**: hash/comparação em key de 1KB é mais caro do que hash de 32 chars.
- **Debug ruim**: `KEYS *` ou `SCAN` com keys gigantes é intratável.
- **Cache hit menos provável**: ordem de chaves JSON pode variar entre serializations → cache miss desnecessário.

## Exemplo (anti-pattern)

```ts
// ❌ ERRADO — key cresce com cada novo filtro, ordenação frágil
const key = `search:${JSON.stringify(filters)}`
// filters = { status: 'active', tags: ['a','b','c'], range: {...} } → 400 bytes fácil
```

## Fix preferido — hash determinístico

```ts
import { createHash } from 'crypto'

function stableHash(obj: unknown): string {
  const normalized = JSON.stringify(obj, Object.keys(obj as any).sort())
  return createHash('sha1').update(normalized).digest('hex').slice(0, 16)
}

const key = `search:${indexName}:${stableHash(filters)}:${page}:${limit}`
// → "search:products:4b8fc2a1d3e9...:2:20"  (fixo, curto, ordenável)
```

Importante: **ordenação estável** das chaves JSON — senão `{a:1,b:2}` e `{b:2,a:1}` geram hashes diferentes e explodem cache misses.

## Alternativa

- Bloom filter + lookup por ID primário quando conjunto de keys é enumerável.
- Cache por dimensão (key por campo) e compor resultado em memória.

## Quando tolerar

Ambiente de dev com baixo volume e debug verbose + keys legíveis ajuda inspeção. Em prod, sempre hash.

## Relações

- **Patterns**: Cache-Aside, Content-Addressable Storage (CAS)
- **OPs relacionados**: [[op-005-cache-sem-versionamento]], [[op-006-cache-falha-quebra-fluxo]]

## Fontes

- [[../../40-templates/AGENTS_SPEC]] §4 OP-007
- Redis — [Keys guidelines](https://redis.io/docs/latest/develop/use/keyspace/)
- Kleppmann — *DDIA*, Ch. 3 (Storage & retrieval)

## Links

- [[_moc]]
- [[../database/database-patterns]]
- [[op-005-cache-sem-versionamento]]
- [[../antipatterns/_moc]]
