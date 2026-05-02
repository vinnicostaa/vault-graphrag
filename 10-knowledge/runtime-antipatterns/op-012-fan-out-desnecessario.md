---
title: OP-012 — Fan-out desnecessário (N queries quando 1 basta)
type: antipattern
tags:
  - antipattern
  - runtime
  - operational
  - performance
  - database
id: OP-012
severity: Medium
created: 2026-04-24
updated: 2026-04-24
doc-consulted: 2026-04-24
aliases:
  - Needless parallel queries
  - Shotgun queries
---

# OP-012 — Fan-out desnecessário (N queries quando 1 basta)

**Severidade**: Medium (performance)
**Categoria**: Acoplamento / Performance

## Sintoma

Em vez de 1 query direcionada, código dispara N queries em paralelo (`Promise.all([...])`) e N-1 delas retornam vazio. Overhead de rede, CPU DB, pool de conexões — pagos sem ganho.

## Por que é problema

- **Latência**: mesmo em paralelo, cada query paga RTT, parsing, plano.
- **Pool exaustion**: N queries × M requests concorrentes esgotam connection pool rapidamente.
- **Observabilidade ruim**: logs/traces enchem de queries vazias, difícil distinguir sinal de ruído.

## Exemplo (anti-pattern)

```ts
// ❌ ERRADO — busca em 5 tabelas; 4 retornam null
const [asAdmin, asCustomer, asVendor, asPartner, asSupport] = await Promise.all([
  adminRepo.findByID(id),
  customerRepo.findByID(id),
  vendorRepo.findByID(id),
  partnerRepo.findByID(id),
  supportRepo.findByID(id),
])
const profile = asAdmin || asCustomer || asVendor || asPartner || asSupport
```

## Fix preferido — coluna discriminadora ou UNION direcionado

### Opção 1: coluna discriminadora na tabela de usuários

```sql
-- Schema
ALTER TABLE users ADD COLUMN profile_type TEXT NOT NULL
  CHECK (profile_type IN ('admin','customer','vendor','partner','support'));
CREATE INDEX idx_users_profile_type ON users(profile_type, id);
```

```ts
const user = await userRepo.findByID(id)
const profile = await profileRepoFor(user.profileType).findByID(id)   // 1 query direcionada
```

### Opção 2: UNION ALL quando múltiplas tabelas são legado

```sql
SELECT 'admin'    AS type, id, name FROM admins    WHERE id = $1
UNION ALL
SELECT 'customer' AS type, id, name FROM customers WHERE id = $1
-- ... (executor decide com base em presence de índices)
LIMIT 1;
```

## Alternativa

- **Dataloader** pattern quando o fan-out é em loop `for (const id of ids) { ... }` — batching + cache de request.
- **CQRS read model** projetando os perfis numa única view materializada.

## Quando tolerar

Perfis realmente independentes e com probabilidade ≥ 50% em cada. Se cada um é 20% de hit, consolidar.

## Relações

- **Patterns**: [[../patterns/cqrs]] (read model otimizado), Dataloader (batch + cache)
- **OPs relacionados**: [[op-025-coluna-sem-indice]] (N queries custam mais sem índice), [[op-026-lista-sem-paginacao]]

## Fontes

- [[../../40-templates/AGENTS_SPEC]] §4 OP-012
- PostgreSQL — [EXPLAIN ANALYZE](https://www.postgresql.org/docs/current/using-explain.html) (consultada 2026-04-24)
- Facebook dataloader — [github](https://github.com/graphql/dataloader) (consultada 2026-04-24)

## Links

- [[_moc]]
- [[../database/database-patterns]]
- [[../patterns/cqrs]]
- [[op-025-coluna-sem-indice]]
