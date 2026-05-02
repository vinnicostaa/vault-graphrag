---
title: "Source 04 — Vitess: sharding MySQL em escala YouTube"
type: source
tags: [research, youtube, vitess, mysql, sharding, database, scaling]
source: https://vitess.io/docs/overview/whatisvitess/
created: 2026-04-23
updated: 2026-04-23
aliases: [vitess]
---

# Source 04 — Vitess

## Metadados

- **Docs oficiais**: https://vitess.io/docs/overview/whatisvitess/
- **Repo**: https://github.com/vitessio/vitess
- **Origem**: YouTube (2010+); CNCF Graduated
- **Apresentação original**: LISA '12 (USENIX) — Jiang Wu, Anthony Yeh
- **Linguagem**: Go
- **Licença**: Apache 2.0

## O que é

Sistema de clustering/sharding pra MySQL que faz o MySQL se comportar como um banco distribuído, sem mudar drasticamente a aplicação.

## Por que YouTube construiu

Pre-Vitess (até ~2010), YouTube sofria:
- Replicas MySQL não acompanhavam escrita → replication lag.
- Sharding manual custoso, frágil, downtime longo (semanas).
- Uma connection exhaustion derrubava fleet inteira.
- Queries caras em produção = SRE pager storm.

## Arquitetura

```
app → VTGate (proxy stateless) → VTTablet (sidecar) → MySQL
                ↓
        Topology Service (etcd/ZooKeeper)
                ↑
           VTctld (control plane)
```

- **VTGate**: proxy stateless — roteia queries pro shard correto, parse SQL, sanitize, connection pooling.
- **VTTablet**: sidecar junto ao MySQL — rate limiting, query killer, backup, replication management.
- **VTctld**: admin/control plane.
- **Topology Service**: etcd ou ZooKeeper — guarda shard map, schema, leaders.

## Números declarados (YouTube interno)

- 10.000+ MySQL instances em 20+ cells.
- 50M+ peak QPS.
- VTGate handles 90% of traffic statelessly.
- Resharding time: de semanas pré-Vitess → 4h com Vitess.
- 99.99% uptime.

## Features-chave

- **Sharding dinâmico** — reshard sem downtime significativo.
- **Query safety** — kill query > threshold, blacklist, transaction timeout.
- **Connection pooling** no VTTablet reduz connection count no MySQL.
- **VReplication** — copy/filter/transform entre shards.
- **Prometheus + Grafana** metrics nativos.
- **Schema migrations online** (gh-ost-like integrado).

## Relevância para Master Síndico

### Aplicável? **Não no MVP. Talvez em 5 anos.**

Master-sindico em M1 (maio/26) tem:
- 1 tenant piloto (1 condomínio).
- Dezenas a centenas de usuários.
- Volume de eventos (assembleia, vídeo upload) baixo.
- **Postgres single-instance + read replica** atende com folga.

### Quando considerar

- Vitess (ou Postgres equivalente: **Citus**, **CockroachDB**, **Yugabyte**) entra na conversa quando:
  - \>1TB de dados quentes.
  - >10k QPS sustained.
  - Multi-region active-active requirement.
- Master-sindico provavelmente vai bater **pgbouncer + partitioning + read replica** por muitos anos antes de precisar sharding.

### Lição aplicável agora

1. **Connection pooling desde dia 1** — usar pgbouncer/pgpool. Lição direta: connection exhaustion é dos primeiros problemas em escala.
2. **Query killer / timeout** — todo statement deve ter timeout (statement_timeout no Postgres). Lição: "queries caras em prod" é vetor de incidente.
3. **Tenant como sharding key mental** desde o schema — mesmo sem shard físico, modelar tenant_id em toda tabela. Quando precisar shardar, o caminho está aberto.
4. **Online schema migrations** — usar gh-ost/pg-online-schema-change equivalente; evitar migration com lock longo.
5. **Metrics nativos** — Postgres + pgBouncer exporter + Prometheus desde o início.

## O que NÃO aplicar (agora)

- Vitess propriamente dito — é pra MySQL; master-sindico é Postgres (decisão legado).
- Topology Service dedicado — etcd/ZooKeeper separado é overhead injustificado.

## Links internos

- [[_destilado]]
- [[../_moc]]
