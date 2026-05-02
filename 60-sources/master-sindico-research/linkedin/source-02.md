---
title: source-02 — Introducing Espresso, LinkedIn's distributed document store
type: source
tags: [research, linkedin, espresso, nosql, distributed-database, change-capture]
source: https://engineering.linkedin.com/espresso/introducing-espresso-linkedins-hot-new-distributed-document-store
created: 2026-04-23
updated: 2026-04-23
aliases: [espresso, linkedin-espresso]
---

# source-02 — Espresso: distributed document store

**URL**: https://engineering.linkedin.com/espresso/introducing-espresso-linkedins-hot-new-distributed-document-store
**Fonte primária**: sim (LinkedIn Engineering)
**Complementar**: https://dbdb.io/db/espresso, paper SIGMOD 2013

## Por que essa fonte importa

Referência canônica de "quando largar RDBMS monolítico por NoSQL documento" — trade-offs explícitos. Útil como **contraponto** pra nossa decisão de manter Postgres em M1/M2/M3.

## Pontos extraídos

### Motivação
- Stacks anteriores LinkedIn: Oracle RDBMS (caro, difícil escalar) + Voldemort KV (eventual consistency, modelo limitado).
- Nenhum atendia "primary store strongly consistent, read/write, com stream de change capture timeline-consistent".
- Dor operacional: schema changes exigiam DBA manual; shards provisionados à mão; failover entre datacenters com downtime; licenças Oracle caras.

### Features técnicas
- Modelo hierárquico: database → table → collection → document.
- Documentos serializados em **Avro binary**.
- **Secondary indexing sincronizado** com mutação base — read-after-write consistente.
- **Change capture** via Databus: transporta transações em commit order pra replicação cross-DC e consumers externos.
- Storage engine **pluggable**: produção usa MySQL/InnoDB por baixo (ironia — RDBMS como storage engine de NoSQL).
- Cluster management via Apache Helix.
- Router HTTP stateless distribui requests.
- SCN (System Change Number) per-partition ordena eventos.

### Escala
- ~30 aplicações LinkedIn rodam sobre Espresso.
- Mais de uma dezena de clusters em produção.
- Milhões de registros/s em peak.
- Centenas de TB (fonte, sem contar replicas).

### Uso pra grafo social
- **Não explicitado neste artigo**. Artigos posteriores (LIquid) mostram grafo como camada **acima** do Espresso com materialized views dedicadas. Espresso não é graph-native; é o storage genérico.

## Aplicabilidade master-sindico

Detalhada em [[_destilado#2-social-graph--espresso--não-aplicável-em-m1m2]].

- **NÃO migrar pra NoSQL doc store em M1/M2**. Critério: LinkedIn migrou quando RDBMS monolítico **não conseguiu escalar** (centenas de milhões de members). Nosso cenário M1: 1-10k contas, Postgres domina com folga.
- **Pattern aproveitável**: change capture stream (Databus) = outbox + logical replication do Postgres. Mesma ideia com ordens de magnitude menos código.
- **Lição negativa**: até o Espresso usa MySQL/InnoDB como storage engine debaixo. O RDBMS não morreu; o que morreu foi a interface SQL monolítica pra acesso. Nossa interface é REST + gRPC sobre Postgres — sem o acoplamento que eles precisaram quebrar.

## Quando releer

- Quando Postgres master-sindico mostrar **gargalo real** (slow query sustained p99 > 500ms após índices, contention > 30%).
- Quando grafo condomínio↔empresa↔síndico ultrapassar 10M edges.
- Quando precisarmos de cross-DC replication com failover < 1min.

## Vizinhos

- [[_destilado]]
- [[source-01]] — The Log (Databus é conceitualmente um log de change capture)
- [[source-10]] — Endorsements migraram de SQL pra GraphDB, mesmo padrão de "RDBMS → store especializado"
