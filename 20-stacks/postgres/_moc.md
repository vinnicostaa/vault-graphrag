---
title: MOC — PostgreSQL stack
type: moc
tags: [moc, stack, postgres, postgresql, database]
created: 2026-04-24
updated: 2026-04-24
---

# PostgreSQL stack

MOC da stack PostgreSQL (Postgres 16+, recomendado 18+). Convenções idiomáticas e operacionais.

## Notas existentes

- [[postgresql-conventions]] — convenções de nomes, PKs, FKs, indexes, soft delete, particionamento (destilado pré-sessão)

## A criar (sob demanda)

- `pgx-v5-patterns.md` — pool, prepared statements, `BeginTxFunc`
- `sqlc-patterns.md` — overrides, type mapping, queries compostas
- `migrations-goose.md` — expand-contract, rollback strategies
- `advisory-locks.md` — patterns de lock lógico

## Relacionados em knowledge global

- [[../../10-knowledge/patterns/transaction-patterns]] — UoW + Saga
- [[../../10-knowledge/database/isolation-levels]] — ANSI levels + Postgres default
- [[../../10-knowledge/database/sharding-strategies]] — quando shardar; alternativas (Citus, pgcat)
- [[../../10-knowledge/database/database-patterns]]
- [[../../10-knowledge/providers/railway/_moc]] — Postgres gerenciado em PaaS

## Vizinhos

- [[../_moc]] — stacks raiz
- [[../../10-knowledge/database/_moc]]
