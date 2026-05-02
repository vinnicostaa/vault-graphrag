---
title: Source 13 — Monarch Google Planet-Scale In-Memory Time Series Database
type: source
tags: [research, google, monarch, time-series, monitoring, prometheus]
source: https://research.google/pubs/monarch-googles-planet-scale-in-memory-time-series-database/
created: 2026-04-23
updated: 2026-04-23
aliases: [monarch-paper]
---

# Source 13 — Monarch: Google's Planet-Scale In-Memory Time Series Database

- **URL primária**: https://research.google/pubs/monarch-googles-planet-scale-in-memory-time-series-database/
- **PDF**: https://www.vldb.org/pvldb/vol13/p3181-adams.pdf
- **Conferência**: VLDB 2020 (PVLDB vol.13 n.12)
- **Data fetch**: 2026-04-23

## Trechos rastreáveis (WebFetch 2026-04-23)

### Escala e arquitetura
- "Ingests terabytes of time series data into memory and serves millions of queries every second".
- Jul/2019: ~950 bilhões de time series; ~750 TB de memória; 2.2 TB/s ingestion; ~6M queries/s.
- "Regionalized architecture for reliability and scalability, and global query and configuration planes that integrate the regions into a unified system".

### Multi-tenant design
- Roda como multi-tenant service suportando várias aplicações Google simultaneamente.
- Monitora availability, correctness, performance, load de "billion-user-scale applications".

### Data model
- "Flexible configuration, an expressive relational data model, and powerful queries".
- **Targets** associam cada time series à source entity (VM, process).
- Schemas suportando distribution-typed time series (histogramas).

### Production status
- Em operação desde 2010.
- Paper destaca "important lessons learned from a decade's experience of developing and running Monarch as a service in Google".

## Aplicabilidade ao Master Síndico

- **Direta literal**: NÃO — Monarch é interno Google.
- **Inspiracional para nossa stack observability**:
  - **Regional + global query plane** → replicável com **Thanos ou Grafana Mimir** sobre Prometheus (M2+).
  - **Multi-tenant labels** → usar `tenant_id` como label Prometheus, mas **com cardinalidade controlada** (não `user_id`).
  - **Distribution-typed** → Prometheus tem histograms nativos; usar pra latências (source-04 SLI).
- **Stack recomendada**:
  - **M1**: Prometheus single-instance + Grafana.
  - **M2**: Thanos ou Mimir quando tiver 3+ ambientes/regiões.
  - **M3+**: considerar VictoriaMetrics ou Mimir gerenciado.
- **Padrões Monarch que viram regras**:
  1. Alertar em **SLO burn rate**, não em CPU/RAM.
  2. **Cardinalidade** controlada — limitar labels que expandem séries.
  3. **Push vs scrape** — Prometheus scrape é default; pra workers efêmeros usar Pushgateway ou OTel collector em modo push.

## Links relacionados

- [[source-03]] Dapper (traces, complementa métricas Monarch).
- [[source-04]] SLOs (inputs pra alertas).
- Destilado: [[_destilado#4.2 Monarch]].
