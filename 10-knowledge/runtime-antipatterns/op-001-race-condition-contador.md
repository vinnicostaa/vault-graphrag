---
title: OP-001 — Race condition em contador / quota
type: antipattern
tags:
  - antipattern
  - runtime
  - operational
  - concurrency
id: OP-001
severity: Critical
created: 2026-04-24T00:00:00.000Z
updated: 2026-04-24T00:00:00.000Z
aliases:
  - Race condition counter
  - Lost update
doc-consulted: '2026-04-24'
---

# OP-001 — Race condition em contador / quota

**Severidade**: Critical
**Categoria**: Concorrência

## Sintoma

Padrão `read → compute → write` sem atomicidade. Duas threads/requests concorrentes leem o mesmo valor, incrementam localmente, gravam — um dos incrementos some ("lost update"). Em quota de plano, usuário consome mais do que tem direito; em contador financeiro, dinheiro somem/duplica.

## Por que é problema

- **Seg.**: quota pode ser burlada por concorrência (enviar N mensagens simultâneas explode `quota_remaining`).
- **Dados**: divergência silenciosa entre valor visto e valor real.
- **Difícil reproduzir**: só ocorre sob carga, escapa de testes unitários.

## Exemplo (anti-pattern)

```go
// ❌ ERRADO — dois requests concorrentes vão zerar o mesmo count
current, _ := repo.GetQuota(ctx, userID)
if current.Remaining <= 0 { return ErrQuotaExceeded }
current.Remaining--
repo.Save(ctx, current)
```

## Fix preferido — atomic increment no banco

```sql
-- PostgreSQL: UPSERT + RETURNING garante atomicidade
INSERT INTO quotas (user_id, remaining) VALUES ($1, $initial)
ON CONFLICT (user_id) DO UPDATE
  SET remaining = quotas.remaining - 1
  WHERE quotas.remaining > 0
RETURNING remaining;
-- Linha vazia → quota exaurida; linha retornada → decremento ok.
```

NoSQL equivalente: `UpdateExpression SET remaining = remaining - 1 IF remaining > 0` (DynamoDB) ou `$inc` + condição (MongoDB).

## Alternativa

- **Distributed lock** (Redis `SET key val NX PX <ttl>`) quando a regra envolve mais de 1 tabela/decisão que não cabe em único SQL atômico.
- **Optimistic locking** via `version` column + retry com backoff.

## Nunca

- `SELECT FOR UPDATE` em hot path sem justificativa explícita — contende locks e degrada throughput.

## Quando tolerar

Contador aproximado (analytics de views, likes onde drift de 1-2 unidades não importa) pode usar write assíncrono via queue. Explicitar no código + PR.

## Relações

- **Patterns que resolvem**: [[../patterns/transaction-patterns]] (UoW + isolation levels), [[../patterns/transaction-patterns#saga]] (compensação quando contador cruza serviço)
- **OPs relacionados**: [[op-002-check-then-act]] (mesma família), [[op-004-sem-retry-strategy]] (retry em falha de UPSERT)

## Fontes

- [[../../40-templates/AGENTS_SPEC]] §4 OP-001
- PostgreSQL docs — [Concurrency Control](https://www.postgresql.org/docs/current/mvcc.html)
- Martin Kleppmann — *Designing Data-Intensive Applications*, Ch. 7 (Transactions)

## Links

- [[_moc]]
- [[../antipatterns/_moc]] — code smells clássicos (distinto)
- [[../database/isolation-levels]] — isolamento que previne lost update
- [[../patterns/transaction-patterns]]
