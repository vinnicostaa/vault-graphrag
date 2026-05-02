---
title: Sharding Strategies
type: concept
tags: [database, sharding, scaling, distributed]
created: 2026-04-24
updated: 2026-04-24
aliases: [horizontal partitioning, database sharding, shard key]
---

# Sharding Strategies

Conceito: dividir um dataset em múltiplos nós de banco (**shards**) para escalar horizontalmente quando um único nó não aguenta mais carga, volume ou latência. Cada shard é um DB independente; a aplicação (ou um proxy) decide para qual shard cada query vai.

## Por que shardar

Vertical scaling (subir o server) tem limites duros:

- RAM e IOPS de uma única máquina
- Custo sub-linear: dobrar capacidade custa >2x
- Ponto único de falha
- Backup/restore que demora horas

Sinais de que a hora chegou: tabelas na casa de **bilhões de linhas**, DB > 10TB com queries críticas degradando mesmo após tuning, write throughput saturando um nó primário, latência p99 inaceitável por contenção de lock/IO.

Antes de shardar, esgotar: índices corretos, particionamento nativo (seção final), read replicas, caching, query rewrite, archiving de dados frios.

## Estratégias

### Hash-based

`shard_id = hash(key) % N`. Distribuição uniforme, zero hotspot **se** a key tem alta cardinalidade. Contra: range query vira scatter-gather; rebalancear ao adicionar shard invalida quase todo mapping (mitigado por **consistent hashing**).

Caso típico: multi-tenant SaaS com milhões de tenants pequenos.

### Range-based

Shards por intervalo contíguo (`A–M`, `N–Z`, ou por faixa temporal). Range query é eficiente (fica em um shard). Contra: hotspot fácil (novos registros sempre vão pro último shard em range cronológico).

Caso típico: dados time-series, arquivamento por ano.

### Geography-based

Shard por região geográfica (US-East, EU-West, APAC). Resolve latência (usuário perto do dado) e residency compliance (GDPR, LGPD). Contra: queries cross-region são caras; usuário que se muda força migração.

Caso típico: plataforma global com regulação de residência.

### Directory-based

Tabela lookup `tenant_id → shard_id`. Flexibilidade máxima — mover tenant individual, co-localizar tenants relacionados, shards heterogêneos. Contra: lookup é hop extra em cada query; a directory vira ponto crítico (precisa de HA e cache).

Caso típico: B2B com tenants de tamanhos muito heterogêneos (alguns 100x maiores).

## Shard key

Decisão mais difícil e mais permanente. Critérios:

- **Imutável** — trocar shard key = migrar a linha. `user_id` sim; `email` não
- **Distribuição uniforme** — evitar hotspot. `created_at` é péssimo (escrita sempre no último shard)
- **Alinhada com queries dominantes** — query comum deve resolver em um shard (`tenant_id` funciona se 90% das queries filtram por tenant)
- **Evita cross-shard joins** — se `order.user_id` e `user.id` sharded pela mesma key, join fica local

Anti-padrões: auto-increment ID puro, timestamp puro, valor com baixa cardinalidade (ex: `status`, `country` em dataset global).

## Problemas

- **Cross-shard query** — precisa scatter (consultar todos) + gather (agregar). Latência do pior shard domina. Mitigar com materialized view por shard ou agregação assíncrona
- **Transação distribuída** — 2PC é lento e frágil. Preferir single-shard transaction + **Saga** entre shards
- **Rebalance** — adicionar shard requer mover dados sem downtime. Consistent hashing reduz movimento; directory-based permite migração incremental
- **Schema change** — migration precisa rodar em todo shard; versões divergentes durante rollout exigem backward-compat
- **Unique constraint global** — `UNIQUE(email)` não existe natively across shards. Precisa de serviço externo (Redis com SETNX) ou garantir unicidade pela shard key

## Tools

- **Citus** — extensão Postgres que transforma N Postgres em cluster sharded; distributed tables + reference tables; distribuído pelo Microsoft/Azure
- **Vitess** — sharding para MySQL criado pelo YouTube; usado em larga escala (Slack, Square, GitHub)
- **MongoDB native** — sharding built-in via config server + mongos router
- **pgcat** — connection pooler com suporte a shard routing para Postgres
- **Spanner / CockroachDB / TiDB** — NewSQL com sharding automático; evita decisão manual ao custo de operação mais complexa

## Quando evitar

- Postgres bem tunado aguenta **dezenas de TB** com particionamento nativo e hardware moderno
- Se read heavy: **read replicas** primeiro
- Se problema é uma tabela gigante: **declarative partitioning** (PARTITION BY RANGE / LIST / HASH) divide a tabela sem divider o DB
- Sharding adiciona complexidade operacional permanente: backup, migration, observability e on-call ficam N vezes mais complexos

Regra: shardar é decisão **irreversível na prática**. Só depois de esgotar alternativas e com dados de capacity planning.

## Alternativa: partitioning nativo

`CREATE TABLE ... PARTITION BY RANGE (created_at)` em Postgres mantém **uma tabela lógica** dividida fisicamente em N partições no mesmo DB. Benefícios:

- Query planner faz **partition pruning** automático (varre só partição relevante)
- Drop de partição antiga é `O(1)` vs `DELETE` pesado
- Sem complexidade de sharding: mesmo DB, mesma transação, mesmo backup

Cobre a maioria dos casos que parecem exigir sharding. Tentar primeiro.

## Links

- [[_moc]]
- [[../_moc]]
- [[database-patterns]]
