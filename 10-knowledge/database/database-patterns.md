---
title: Database — padrões universais
type: concept
tags: [database, patterns, schema]
source: .kiro/steering/database.md
created: 2026-04-22
---

# Database — padrões universais

Regras que valem em qualquer banco relacional. Implementação em banco específico em `../../20-stacks/<banco>/`.

## Regras absolutas

- **PKs: UUIDv7** — ordenável por tempo + aleatório. Gerado na aplicação, não no banco.
- **Nomenclatura**: `snake_case` em tudo (tabelas, colunas, índices, constraints, enums).
- **Timestamps padrão em toda tabela**: `created_at`, `updated_at`, `deleted_at` (soft delete).
- **Tabelas imutáveis por design** — sem `deleted_at`, sem UPDATE: `timeline_entries`, `audit_trail`, `votes`, `closed_deals` após assinatura. Enforcement via **trigger** ou constraint (`CHECK (id IS NOT NULL) NOT VALID` — workaround pra bloquear updates).
- **Valores monetários**: inteiro de 64 bits (centavos). **Nunca** `FLOAT`, `DOUBLE`, `NUMERIC` pra dinheiro. Ver AP em [[../antipatterns/_moc]].
- **Valores fracionários com regra de negócio** (fração ideal, proporção de voto): `DECIMAL/NUMERIC(10,6)`. **Nunca** `FLOAT`.
- **Migrations nunca destrutivas em produção**: sem `DROP TABLE`, sem `DROP COLUMN`, sem `ALTER COLUMN` que perde dados. Alternativa: deprecar → migrar dados → drop em migration futura.
- **CHECK constraints obrigatórios** pra enums e invariantes simples. "Validação só na aplicação" cai em produção — banco é última linha de defesa.
- **Sem ORM** — driver nativo + query builder ou codegen (sqlc, Prisma, Drizzle). ORM esconde query plan.

## Numeração de migrations por módulo

Em projeto com múltiplos bounded contexts, particionar faixas numéricas:

```
00001-099 → identity (users, sessions, enums)
00100-199 → billing  (plans, subscriptions, features)
00200-299 → institutional
00300-399 → commercial
00400-499 → content
00500-599 → assembly
```

Evita colisão quando múltiplas PRs criam migration no mesmo sprint.

## Índices obrigatórios

Pra toda nova tabela:

```sql
-- Soft delete
CREATE INDEX idx_{table}_deleted_at ON {table}(deleted_at) WHERE deleted_at IS NULL;

-- Tenant isolation
CREATE INDEX idx_{table}_{tenant_col} ON {table}({tenant_col}) WHERE deleted_at IS NULL;

-- Todas as FKs
CREATE INDEX idx_{table}_{fk_column} ON {table}({fk_column});

-- Colunas em WHERE/ORDER BY frequentes
CREATE INDEX idx_{table}_status ON {table}(status) WHERE deleted_at IS NULL;
CREATE INDEX idx_{table}_created_at ON {table}(created_at DESC) WHERE deleted_at IS NULL;
```

Antes de criar índice em migration: rodar `EXPLAIN (ANALYZE, BUFFERS)` via MCP de banco com hypothetical indexes → confirma que índice é usado e reduz custo.

## Nomenclatura canônica

| Tipo | Padrão | Exemplo |
|---|---|---|
| Tabela | `snake_case` plural | `subscriptions`, `timeline_entries` |
| Coluna | `snake_case` | `condominium_id`, `created_at` |
| PK | `id` (UUID) | `id UUID PRIMARY KEY` |
| FK | `{tabela_singular}_id` | `user_id`, `plan_id` |
| Índice | `idx_{tabela}_{coluna(s)}` | `idx_users_email` |
| Unique constraint | `uq_{tabela}_{coluna(s)}` | `uq_users_email` |
| Check constraint | `chk_{tabela}_{regra}` | `chk_subscriptions_price_positive` |
| Enum | `snake_case` singular | `subscription_status`, `user_role` |
| Trigger | `trg_{tabela}_{ação}` | `trg_subscriptions_update_timestamp` |

## Tenant isolation no schema

Toda query multi-tenant **filtra por `tenant_id` do contexto**:

```sql
SELECT * FROM subscriptions
WHERE tenant_id = $1
  AND user_id = $2
  AND deleted_at IS NULL;
```

**Nunca** aceitar `tenant_id` do request body — sempre do claim de auth no contexto. Cross-tenant apenas via grant explícito (Connect Me, validações).

## Connection pool config

Defaults de drivers **não** são adequados pra produção. Configurar explicitamente:

- **MaxConns**: `CPU_cores × 2 + 4` típico (ex: 8 cores = 20)
- **MinConns**: 4
- **MaxConnLifetime**: 15min (evita conn ruins acumulando)
- **MaxConnIdleTime**: 5min
- **HealthCheckPeriod**: 1min

## Soft delete patterns

Consistência em todas as queries:

```sql
-- LEITURA: sempre filtra
SELECT ... WHERE deleted_at IS NULL;

-- DELETE: nunca DELETE direto, sempre UPDATE
UPDATE <table> SET deleted_at = NOW() WHERE id = $1;

-- ÍNDICE: partial index pula tombstones
CREATE INDEX ... WHERE deleted_at IS NULL;
```

Exceção: tabelas **imutáveis por design** (audit, timeline, votes) não têm `deleted_at` — dado nunca é removido, mesmo logicamente.

## UPSERT com conflict handling

Padrão pra idempotência em escrita:

```sql
INSERT INTO subscriptions (id, user_id, ..., stripe_subscription_id)
VALUES ($1, $2, ..., $N)
ON CONFLICT (stripe_subscription_id)
DO UPDATE SET
    status = EXCLUDED.status,
    current_period_end = EXCLUDED.current_period_end,
    updated_at = NOW()
RETURNING *;
```

Elimina OP-002 (check-then-act race) e simplifica lógica.

## Links

- [[_moc]]
- [[../../20-stacks/postgres/postgresql-conventions]] — PG-specific (sqlc, goose, pgx pool, sqlc.yaml)
- [[../patterns/transaction-patterns]] — UoW, Saga, locks
- [[../security/security-principles]] — tenant isolation, validação server-side
- [[../runtime-antipatterns/_moc]] — OP-001 (race), OP-002 (check-then-act), OP-012 (fan-out), OP-025 (índice ausente)
