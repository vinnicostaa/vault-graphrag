---
title: Netflix EVCache / Dynomite — cache distribuído em escala
type: source
tags: [cache, evcache, dynomite, memcached, redis, replication, kafka, research]
source: https://www.infoq.com/articles/netflix-global-cache/
created: 2026-04-23
updated: 2026-04-23
aliases: [netflix-evcache]
---

# Netflix EVCache / Dynomite: cache em escala

> Fontes: InfoQ "Building Global Caching System at Netflix", GitHub Netflix/EVCache wiki, Netflix Tech Blog cache warming — destilado 2026-04-23.

## Números EVCache

- ~**2 trilhões de items**, **14,3 PB** de dados
- **200 clusters**, **22.000 instâncias**
- **400M ops/s** (sim, por segundo)
- **30M eventos de replicação globais** (multi-região)

## Arquitetura

- Client-server sobre **Memcached** + **spymemcached** + extstore (offload pra SSD).
- Shard via **Ketama consistent hashing** — add/remove nós sem rehash completo.
- Replicação multi-zona dentro da região (leitura local, escrita replicada).
- **Global replication**: mutations emitidas pro **Kafka**; Replication Relay → Replication Proxy em outras regiões → cache destino.

## Otimizações relevantes (aplicáveis em qualquer cache)

- **Batch compression Zstandard** → 35% redução de bandwidth.
- **DNS service discovery (Eureka DNS)** em vez de NLB → cortou 50% do custo de transfer (45 GB/s → 100 MB/s).
- Envio de **metadata (chave + versão)** via Kafka, valor buscado só no destino → reduz load no broker.

## Dynomite (diferente do EVCache)

- Camada distribuída **sobre Redis** (podia ser sobre Memcached, mas Netflix usa Redis).
- Oferece **peer-to-peer replication** multi-região com consistência eventual.
- Lida com operações Redis (list, hash, sorted set) que Memcached não suporta.
- Hoje: mantido mas foco de investimento é EVCache; Dynomite usado onde Redis semantics são necessárias.

## Cache warming (insight crítico)

- EVCache é **Tier-1** — muitos serviços dependem dele como único storage.
- Scale up → precisa **warm cache** do cluster novo antes de receber tráfego.
- Netflix fez tooling próprio pra copiar hot keys de cluster old → new em background.
- Sem warming: cold start → miss rate → sobrecarrega origem → cascata.

## Padrões canônicos

1. **Read-through** — miss busca na origem, popula cache.
2. **Write-through / write-behind** — escrita atualiza cache (síncrono ou assíncrono).
3. **Cache-aside** — app controla; mais flexível mas mais bug-prone.
4. **TTL + stale-while-revalidate** — retorna stale, refresca em background.

## Aplicabilidade ao Master Síndico

- Stack hoje: **Postgres** + provavelmente **Redis** (cache + rate limit + session).
- **NÃO usar EVCache** — overkill, não temos multi-região.
- **Redis simples single-instance** basta pra M1-M2. Replicação Redis Sentinel em M3 se precisar HA.

### O que aplicar do playbook Netflix

- **TTL baixo em dados que podem ser stale** (ranking Connect Me: 5min; perfil empresa: 1h).
- **Cache warming no deploy** — script que popula Redis com keys "hot" pós-deploy.
- **Compression (Zstandard) em valores grandes** — se cachear payload JSON > 1KB.
- **Cache-aside pattern** — app lê cache, miss busca DB, popula cache.
- **Observar hit rate** — métrica obrigatória; queda súbita indica invalidation bug.

### Quando Dynomite faria sentido

- Nunca no nosso horizonte. Redis puro ou Redis Cluster cobre.
- Se precisar cross-região, usar PostgreSQL logical replication + Redis local por região.

## Links

- [[source-06]] — CDN (camada acima do cache)
- [[source-08]] — polyglot persistence
- [[_destilado]]
