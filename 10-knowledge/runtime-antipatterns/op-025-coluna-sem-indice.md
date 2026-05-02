---
title: OP-025 — Coluna com query frequente sem índice
type: antipattern
tags:
  - antipattern
  - runtime
  - operational
  - performance
  - database
id: OP-025
severity: Medium
created: 2026-04-24
updated: 2026-04-24
doc-consulted: 2026-04-24
aliases:
  - Missing index
  - Full table scan
---

# OP-025 — Coluna com query frequente sem índice

**Severidade**: Medium (vira High em escala)
**Categoria**: Performance / Database

## Sintoma

`SELECT * FROM orders WHERE user_id = $1` faz **full table scan** porque `user_id` não tem índice. Em dev (100 rows) parece ok; em produção (10M rows) o endpoint trava, locks se acumulam, CPU do DB vai a 100%.

## Por que é problema

- **Latência degradando com o tempo**: table scan fica linearmente pior.
- **Load em cascata**: requests longos seguram conexão; pool esgota; outros endpoints degradam.
- **Custo**: CPU/IO do DB caro; em cloud vira dinheiro.
- **Invisível até a produção escalar**: testes não replicam volume real.

## Exemplo (anti-pattern)

```sql
-- Schema sem índice em user_id (FK não cria índice automaticamente em todos os DBs)
CREATE TABLE orders (
    id         UUID PRIMARY KEY,
    user_id    UUID NOT NULL REFERENCES users(id),
    status     TEXT NOT NULL,
    created_at TIMESTAMPTZ NOT NULL DEFAULT now()
);

-- Query frequente — full scan em produção
SELECT * FROM orders WHERE user_id = $1 AND status = 'active';
```

## Fix preferido — índice composto que cobre o padrão de query

```sql
-- Índice composto para query "orders by user, filtered by status"
CREATE INDEX idx_orders_user_status ON orders(user_id, status)
  WHERE status IN ('active', 'pending');
-- "Partial index" — mais compacto, cobre só queries comuns

-- Para paginação ordenada
CREATE INDEX idx_orders_user_created ON orders(user_id, created_at DESC);
```

**Medir antes e depois**:

```sql
EXPLAIN ANALYZE SELECT * FROM orders WHERE user_id = $1 AND status = 'active';
-- Antes: Seq Scan on orders (cost=0.00..1543.00 rows=50 width=...) (actual rows=12)
-- Depois: Index Scan using idx_orders_user_status (cost=0.42..8.44 rows=1 ...)
```

## Estratégia de index

1. **Colunas sempre em WHERE**: candidatas primárias (especialmente FK em tabela grande).
2. **Índice composto**: ordem importa — colunas mais seletivas primeiro (ex.: `(tenant_id, user_id, status)`).
3. **Partial index** (`WHERE status = 'active'`): reduz tamanho quando só subset é queried.
4. **Covering index** (`INCLUDE` em Postgres): inclui colunas lidas sem tocar heap.
5. **Cuidado com over-indexing**: cada index paga custo em write.

## Alternativa

- **Materialized view** para agregações complexas repetidas.
- **Read replica** especializado para relatórios pesados (separa workload OLTP vs OLAP).
- **Partitioning** para tabelas gigantescas com padrão temporal (`created_at`).

## Quando tolerar

Tabela < 10k rows com crescimento lento e queries raras — full scan é aceitável. Medir antes de indexar (overhead de write é real).

## Relações

- **Patterns**: CQRS (read model com índices otimizados), [[../database/isolation-levels]]
- **OPs relacionados**: [[op-026-lista-sem-paginacao]] (par frequente), [[op-012-fan-out-desnecessario]]

## Fontes

- [[../../40-templates/AGENTS_SPEC]] §4 OP-025
- PostgreSQL — [Indexes](https://www.postgresql.org/docs/current/indexes.html) (consultada 2026-04-24)
- Use The Index, Luke! — [SQL indexing primer](https://use-the-index-luke.com/) (consultada 2026-04-24)
- Kleppmann — *DDIA*, Ch. 3 (Storage & retrieval)

## Links

- [[_moc]]
- [[../database/database-patterns]]
- [[op-026-lista-sem-paginacao]]
- [[../patterns/cqrs]]
