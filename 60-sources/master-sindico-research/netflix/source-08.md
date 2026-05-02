---
title: Netflix Polyglot Persistence — Cassandra, Elasticsearch, MySQL
type: source
tags: [polyglot-persistence, cassandra, elasticsearch, mysql, database-per-service, research]
source: https://www.infoq.com/articles/polyglot-persistence-microservices/
created: 2026-04-23
updated: 2026-04-23
aliases: [netflix-polyglot-persistence]
---

# Netflix Polyglot Persistence

> Fontes: InfoQ, QCon SF 2017/2021 talk "Polyglot Persistence Powering Microservices", Netflix Marken annotation service — destilado 2026-04-23.

## Princípio

"Não existe silver bullet" — cada microserviço escolhe o datastore certo pro seu padrão de acesso. Netflix's Cloud Database Engineering oferece polyglot persistence **como serviço**.

## Stack Netflix (roles)

| Datastore | Papel |
|---|---|
| **Cassandra** | Source of truth, writes multi-região (viewing history de 300M+ usuários) |
| **Elasticsearch** | Search, annotations, log analytics |
| **Dynomite (+Redis)** | Cache quente, estruturas complexas |
| **EVCache** | Cache massivo chave-valor |
| **MySQL** | Transações com forte ACID (billing, signup) |
| **Apache Iceberg** | Data lakehouse, analytics |
| **Kafka** | Event streaming (não DB, mas parte do stack de dados) |

## Lições aprendidas

1. **Database per service** — cada microserviço dono do seu schema, sem cross-DB queries.
2. **Source of truth ≠ cache ≠ search index** — Cassandra guarda, EVCache serve quente, ES indexa. Três coisas diferentes.
3. **Benchmark antes de escolher** — Netflix criou **NDBench** (Netflix Data Bench) open source pra testar Cassandra/ES/etc.
4. **Custo operacional cresce com nº de datastores** — cada um tem backup, upgrade, monitoring, tuning. Não adicionar levianamente.

## Anti-padrões

- **"Novo serviço = novo DB"** sem justificativa → explosão operacional.
- **Shared DB entre microserviços** → volta ao monolito distribuído.
- **Usar Cassandra pra tudo** quando MySQL resolve → complexidade desnecessária.
- **Usar ES como source of truth** → ES não é DB transacional, vai te morder.

## Aplicabilidade ao Master Síndico

### Stack realista pra M1-M3

| Datastore | Papel no master-sindico | Justificativa |
|---|---|---|
| **PostgreSQL** | Source of truth pra **tudo** no M1 | ACID, JSONB, full-text BR básica, transactions cross-aggregate — mais barato operar que 5 DBs |
| **Redis** | Cache + rate limit + session + pub/sub pubsub light | Single node M1; cluster M2+ |
| **S3/R2** | Object storage (upload docs, backup) | Obrigatório; barato |
| **Mux** | Vídeo (storage + delivery + playback) | SaaS — não operamos |
| **Meilisearch** ou **Typesense** | Search (Connect Me empresas, biblioteca docs) — M2+ | Postgres full-text cobre M1; search dedicado quando > 10k empresas |

### NÃO adotar (respeitando YAGNI)

- Cassandra: só faz sentido com writes > 10k/s sustained; não é nosso caso.
- Elasticsearch: muito pesado pra operar; Meilisearch/Typesense mais simples.
- Iceberg / data lake: overkill pra analytics básico; Postgres + Metabase/Grafana resolve.

### Evolução prevista

- M1: **só Postgres + Redis + S3**.
- M2: adicionar search dedicado quando Postgres FTS ficar lento.
- M3+: considerar read replicas Postgres por tenant se multi-região BR necessário.

## Links

- [[source-07]] — cache complementa persistence
- [[source-09]] — observability
- [[_destilado]]
