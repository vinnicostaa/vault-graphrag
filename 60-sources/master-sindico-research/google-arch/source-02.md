---
title: Source 02 — Bigtable A Distributed Storage System for Structured Data
type: source
tags: [research, google, bigtable, nosql, wide-column, sstable]
source: https://research.google.com/archive/bigtable.html
created: 2026-04-23
updated: 2026-04-23
aliases: [bigtable-paper]
---

# Source 02 — Bigtable: A Distributed Storage System for Structured Data

- **URL primária**: https://research.google.com/archive/bigtable.html
- **PDF canônico**: https://research.google.com/archive/bigtable-osdi06.pdf
- **Conferência**: OSDI 2006
- **Autores**: Chang, Dean, Ghemawat, Hsieh, Wallach, Burrows, Chandra, Fikes, Gruber
- **Produto derivado**: Google Cloud Bigtable — https://cloud.google.com/bigtable
- **Data fetch**: 2026-04-23

## Trechos rastreáveis

- "Bigtable is a distributed storage system for managing structured data that is designed to scale to a very large size: petabytes of data across thousands of commodity servers" (research.google archive).
- Modelo: "sparse, distributed multi-dimensional sorted map" indexado por `(row_key, column_key, timestamp) → byte_array`.
- Construído sobre: **Colossus (sucessor GFS) + Chubby (lock service) + SSTable (storage log-structured tipo LevelDB)**.
- Cloud Bigtable em Apr/2024: "manages over 10 Exabytes of data and serves more than 7 billion requests per second" (cloud.google.com/bigtable).
- SSTable: "persistent, ordered, immutable key-value mapping made up of a sequence of blocks, each 64KB" (summaries acadêmicas consistentes).
- Inspirou: Apache HBase, Apache Cassandra.

## Aplicabilidade ao Master Síndico

- **Direta**: NENHUMA. Condomínios BR não geram petabytes; modelo wide-column não encaixa em domínio governança (entidades bem estruturadas, relacionais).
- **Inspiracional**:
  - **Imutabilidade SSTable** → inspira event-sourcing com append-only event log + snapshots + compaction (M2+ talvez).
  - **Hierarchy metadata (root tablet → metadata → tablets)** → padrão de navegação por árvore que pode ser útil em directory service de unidades/apartamentos.
- **Por quê não agora**: PostgreSQL + schema relacional atende. Bigtable/Cassandra só se precisarmos de write-heavy com latência p99 < 10ms em centenas de milhares de RPS — não é o caso.

## Links relacionados

- [[source-06]] Colossus (storage backend de Bigtable).
- [[source-19]] Chubby (coordination de Bigtable).
- Destilado: [[_destilado#2.4 Colossus / Bigtable / GFS]].
