---
title: Source 08 — F1 Distributed SQL Database That Scales
type: source
tags: [research, google, f1, spanner, sql, adwords, distributed-sql]
source: https://research.google/pubs/f1-a-distributed-sql-database-that-scales/
created: 2026-04-23
updated: 2026-04-23
aliases: [f1-paper]
---

# Source 08 — F1: A Distributed SQL Database That Scales

- **URL primária**: https://research.google/pubs/f1-a-distributed-sql-database-that-scales/
- **PDF**: https://research.google.com/pubs/archive/41344.pdf
- **Conferência**: VLDB 2013 (SIGMOD 2012 prévia)
- **Autores**: Shute, Vingralek, Samwel, Handy, Whipkey, Rollins, Oancea, Littlefield, Menestrina, Ellner, Cieslewicz, Rae, Stancescu, Apte
- **Data fetch**: 2026-04-23

## Trechos rastreáveis

- "F1 combines high availability, the scalability of NoSQL systems like Bigtable, and the consistency and usability of traditional SQL databases".
- Construído **sobre Spanner** — "provides extremely scalable data storage, synchronous replication, and strong consistency and ordering properties".
- Em produção desde 2012 gerenciando TODO o AdWords: ">100 TB, serves up to hundreds of thousands of requests per second, and runs SQL queries that scan tens of trillions of data rows daily".
- Features: interface SQL + key-value; structured data types; **hierarchical schema model**.
- Latência de commit cross-DC mitigada por "hierarchical schema model with structured data types and through smart application design".

## Aplicabilidade ao Master Síndico

- **Direta**: NENHUMA (F1 é build sobre Spanner, não runnable fora do Google).
- **Inspiracional — Hierarchical schema**:
  - Dados do Master têm hierarquia natural: `tenant → condominio → unidade → morador` + `tenant → assembleia → votos`.
  - Postgres resolve via FK, mas **modelar PK composto** `(tenant_id, condominio_id, unidade_id)` em tabelas-filhas imita a locality F1/Spanner.
  - Queries scoped por condominio usam prefixo de índice — cache hit rate sobe.
- **Por quê relevante mesmo sem adotar**: valida a abordagem row-based + hierarchical que planejamos pro Postgres (source-07). F1 prova que mesmo em escala extrema (100 TB, 100k RPS) a modelagem relacional hierárquica funciona.

## Links relacionados

- [[source-01]] Spanner (base de F1).
- [[source-07]] Multi-tenancy patterns.
- Destilado: [[_destilado#2.2 Consistência]].
