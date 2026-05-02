---
title: Source 18 — Research at Google Publications portal
type: source
tags: [research, google, publications, index, meta]
source: https://research.google/pubs/
created: 2026-04-23
updated: 2026-04-23
aliases: [research-google-pubs]
---

# Source 18 — Publications — Google Research (portal)

- **URL primária**: https://research.google/pubs/
- **Papers relacionados citados pelos outros sources neste research**: https://research.google/pubs/borg-omega-and-kubernetes/, https://research.google/pubs/spanner-googles-globally-distributed-database-2/, https://research.google/pubs/f1-a-distributed-sql-database-that-scales/, etc.
- **Data fetch**: 2026-04-23

## Conteúdo rastreável

- Portal oficial de publicações do Google Research.
- Areas: Distributed Systems and Parallel Computing, Data Management, Software Systems, Machine Intelligence, Security, etc.
- "Google publishes hundreds of papers each year".

## Aplicabilidade ao Master Síndico

Este source funciona como **meta-fonte**: referência pra futuras pesquisas conforme o projeto evoluir (M2/M3+). Papers extras a monitorar sem destilar agora:

| Paper | URL | Quando destilar |
|---|---|---|
| MapReduce (2004) | research.google/pubs (paper clássico) | Não — superseded por Spark/Flink/Dataflow |
| Chubby (2006) | https://research.google/pubs/the-chubby-lock-service-for-loosely-coupled-distributed-systems/ | [[source-19]] já tem stub |
| Google File System (2003) | research.google archive | Não — superseded por Colossus [[source-06]] |
| MegaStore (2011) | research.google archive | Se adotar cross-region sync Postgres (M3+) |
| Photon (2013) — stream joins | research.google/pubs | Se stream joins virarem core (M3+) |
| Mesa (2014) — near-real-time warehousing | research.google/pubs | Alternativa a BigQuery/ClickHouse pra dashboards ads-like (M3+) |
| Sibyl (2014) — large-scale ML | research.google/pubs | Se Master Síndico adotar ML em produção |

## Links relacionados

- Todos os outros sources deste research/google-arch.
- Destilado: [[_destilado#8 Pub/Sub Research at Google]].
