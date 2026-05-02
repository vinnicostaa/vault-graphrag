---
title: Source 01 — Spanner Google's Globally-Distributed Database
type: source
tags: [research, google, spanner, distributed-db, truetime, external-consistency]
source: https://research.google/pubs/spanner-googles-globally-distributed-database-2/
created: 2026-04-23
updated: 2026-04-23
aliases: [spanner-paper]
---

# Source 01 — Spanner: Google's Globally-Distributed Database

- **URL primária**: https://research.google/pubs/spanner-googles-globally-distributed-database-2/
- **PDF canônico**: https://research.google.com/archive/spanner-osdi2012.pdf
- **Conferência**: OSDI 2012; também ACM TOCS Aug 2013 (DOI: 10.1145/2491245)
- **Autores**: Corbett, Dean, Epstein, Fikes, Frost, Furman, Ghemawat, Gubarev, Heiser, Hochschild, Hsieh, Kanthak, Kogan, Li, Lloyd, Melnik, Mwaura, Nagle, Quinlan, Rao, Rolig, Saito, Szymaniak, Taylor, Wang, Woodford
- **Data fetch**: 2026-04-23

## Trechos rastreáveis (fonte: research.google abstract + WebFetch 2026-04-23)

- "Spanner is Google's scalable, multi-version, globally-distributed, and synchronously-replicated database" — research.google.
- "the first system to distribute data at global scale and support externally-consistent distributed transactions".
- **TrueTime**: "a novel time API that exposes clock uncertainty" (TTinterval com uncertainty bounded, tipicamente < 10ms em operação normal).
- Features chave: "nonblocking reads in the past, lock-free read-only transactions, and atomic schema changes".
- Primeiro cliente interno: **F1** (rewrite do backend AdWords), com 5 réplicas nos EUA.

## Aplicabilidade ao Master Síndico

- **Direta**: NENHUMA. Não rodamos planet-scale, não precisamos de TrueTime nem external consistency cross-continent.
- **Inspiracional (M3+ se necessário)**:
  - Conceito de **timestamp commit com uncertainty bound** — aplicável se algum dia implementarmos event sourcing com ordenação global forte.
  - **Synchronous replication** cross-region — inspira stand-by Postgres em outra região BR pra DR.
- **Por quê não agora**: overhead operacional de Spanner/CockroachDB não se justifica pra volume atual e de M1-M3. Postgres single-master + replica + SAGA entre BCs basta.

## Links relacionados

- Próximo: [[source-07]] Multi-tenancy em Spanner (aplicável conceitualmente).
- Próximo: [[source-08]] F1 (construído sobre Spanner).
- Destilado: [[_destilado#2.2 Consistência]].
