---
title: TAO — Facebook's Distributed Data Store for the Social Graph
type: source
tags: [research, meta, tao, graph, cache, write-through, eventual-consistency, mysql, source]
source: https://engineering.fb.com/2013/06/25/core-infra/tao-the-power-of-the-graph/
created: 2026-04-23
updated: 2026-04-23
aliases: [source-04-tao]
---

# source-04 — TAO: The Power of the Graph

- **URL (blog)**: https://engineering.fb.com/2013/06/25/core-infra/tao-the-power-of-the-graph/
- **URL (paper USENIX ATC 2013)**: https://www.usenix.org/system/files/conference/atc13/atc13-bronson.pdf
- **Publicado**: 2013-06-25 (blog) / 2013-06 (paper, USENIX ATC)
- **Autor/Canal**: Bronson et al. (Facebook) / Engineering at Meta
- **Tipo**: paper acadêmico + post de engenharia

## Sinopse

TAO é o datastore de grafo social que substituiu o cache memcached client-side no Facebook. Modelo: **objects** (nós tipados) + **associations** (arestas tipadas). Write-through cache em duas camadas sobre MySQL, eventual consistency entre regiões.

## Pontos-chave

1. **Modelo de dados**: `objects` (nós com ID + tipo + payload) e `associations` (arestas direcionadas com tipo, timestamp, payload). Associations agrupadas por origem + tipo, permitindo queries tipo "últimos K amigos de X".
2. **Arquitetura em tiers** (por região geográfica):
   - **Follower tier** — recebe requisições dos clientes, serve a maioria dos reads do cache.
   - **Leader tier** — fala com MySQL e mantém consistência entre followers da região.
   - **MySQL** — persistente, shardado.
3. **Write-through cache** — todo write vai ao cache (leader) e ao MySQL; invalidation/refresh propaga para followers da mesma região, e assincronamente para leaders de outras regiões.
4. **Consistência**:
   - **Read-after-write** dentro de um tier com no máximo 1 falha.
   - **Eventual consistency** entre regiões, lag <1s tipicamente.
5. **Escala** (em 2013): 1B+ reads/s, milhões de writes/s, centenas de milhares de shards.
6. **Multi-region master-slave por shard**: um shard tem master region; writes vão pro master que replica pro resto.

## Aplicabilidade ao Master Síndico

Detalhado em `_destilado.md` §3.

**Aplicações diretas**:
- **Write-through pattern pro ABAC cache (Redis)** — consistência de autorização não tolera janela; toda mudança de permissão invalida/reescreve cache sincronamente.
- **Invalidation granular por chave** — shard-by-tenant + key pattern `tenant:{id}:user:{id}:perm_fp:{hash}`.

**Não aplicar em M1**:
- Two-tier caching multi-region (overkill BR-only).
- MySQL sharding manual (Postgres single-instance atende M1; read replica no M2).

**Preparar arquitetura pra futuro**:
- Separar domain ↔ repository ↔ cache decorator (clean architecture §2). Decorator implementa write-through e pode ser reconfigurado pra leader-follower sem mudar domain.

## Citações

> "TAO servers satisfy most read requests from a write-through cache while orchestrating the execution of writes and maintaining cache consistency among all TAO clusters."

> "Objects and associations in TAO are eventually consistent with replication lag usually less than one second."

## Confiança / datas

- Data blog confirmada via fetch (2013-06-25) — **alta confiança**.
- Paper USENIX 2013 canônico — **alta confiança**.
- Fonte primária. Conceitos e números (1B reads/s) citados tanto no blog quanto no paper.
