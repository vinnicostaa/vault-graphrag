---
title: Source 17 — Dremel A Decade of Interactive SQL Analysis at Web Scale
type: source
tags: [research, google, dremel, bigquery, analytics, columnar, olap]
source: https://research.google/pubs/dremel-a-decade-of-interactive-sql-analysis-at-web-scale/
created: 2026-04-23
updated: 2026-04-23
aliases: [dremel-paper]
---

# Source 17 — Dremel: A Decade of Interactive SQL Analysis at Web Scale

- **URL primária**: https://research.google/pubs/dremel-a-decade-of-interactive-sql-analysis-at-web-scale/
- **PDF**: https://www.vldb.org/pvldb/vol13/p3461-melnik.pdf
- **Conferência**: VLDB 2020 (retrospectiva 10 anos após Dremel VLDB 2010).
- **Data fetch**: 2026-04-23

## Trechos rastreáveis (WebFetch 2026-04-23)

### Princípios fundantes Dremel (hoje na base do BigQuery)
- **Disaggregated storage and compute** — separa storage (Colossus) de compute (Dremel workers).
- **In situ analysis** — analisa dado onde está; evita movimentação.
- **Columnar storage for semistructured data** — nested/non-relational data em formato colunar (Capacitor é o formato atual).

### Componentes BigQuery
- Storage: **Colossus** (columnar + compressão).
- Query engine: **Dremel** — multi-level serving tree; queries quebradas e reagrupadas.
- Infra: **Jupiter** (fabric de rede), **Borg** (orquestração).

### Impacto
- "The three ideas leveraged in the original Dremel paper became building blocks in many serverless data analytics systems".
- BigQuery GA em 2011.

## Aplicabilidade ao Master Síndico

### M1/M2
- **NÃO necessário**. Reports dentro do Postgres com CTE + materialized views + partial indexes cobrem até milhões de eventos.
- **Padrão desde já**: **separar schemas OLTP vs OLAP** no mesmo Postgres (`app.*` vs `analytics.*`). Views materializadas refresh scheduled. Facilita migração futura sem refactor de código.

### M3+
- Se volume de eventos analíticos cruzar ~10M/dia ou queries bloquearem OLTP:
  - **Opção 1**: BigQuery (se estivermos em GCP).
  - **Opção 2**: **ClickHouse** — open-source, columnar, disaggregated. Equivalente ideológico mais acessível.
  - **Opção 3**: DuckDB no edge pra reports locais do síndico (ex: PWA mobile).
- Seguir princípios Dremel mesmo em qualquer escolha: storage-compute separados, columnar, in-situ.

## Links relacionados

- [[source-06]] Colossus (storage do BigQuery).
- [[source-15]] Borg (orquestração do BigQuery).
- Destilado: [[_destilado#7 BigQuery]].
