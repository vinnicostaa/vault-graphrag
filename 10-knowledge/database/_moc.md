---
title: MOC — Database
type: moc
tags:
  - moc
  - database
created: 2026-04-22T00:00:00.000Z
updated: '2026-04-24'
---

# Database

Padrões cross-linguagem e cross-banco pra modelagem, queries, migrations. Implementações concretas por banco em `../../20-stacks/<banco>/`.

## Notas

- [[database-patterns]] — padrões universais (PK, naming, soft delete, imutabilidade, numeração de migration)
- [[isolation-levels]] — 4 níveis ANSI (Read Uncommitted → Serializable), MVCC Postgres, quando subir isolation vs `SELECT FOR UPDATE` (D-vault-009)
- [[sharding-strategies]] — hash/range/geography/directory; shard keys; tools (Citus, Vitess, pgcat); quando evitar (D-vault-009)

## A criar (conforme ingestão)

- indexing-strategy — quando criar índice, como validar com EXPLAIN
- query-patterns — pagination, cursor-based, search
- schema-versioning — migrations idempotentes e não-destrutivas
- data-retention — LGPD/GDPR retention policies aplicadas ao banco

## Vizinhos

- [[../_moc]] — knowledge
- [[../../20-stacks/postgres/_moc]] — implementação PostgreSQL (MOC-stub criado D-vault-011)
- [[../patterns/transaction-patterns]] — ACID, Saga, locks (usa `isolation-levels` como base)
- [[../architecture/cqrs]] — read model pode precisar sharding
- [[../security/_moc]] — tenant isolation, audit trail, validação server-side
- [[../../00-meta/STATE]] — D-vault-009 (isolation + sharding)
