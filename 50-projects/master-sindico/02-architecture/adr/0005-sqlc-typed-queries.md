---
title: ADR-0005 — sqlc para queries tipadas
type: adr
tags: [adr, database, sqlc, postgres, master-sindico, v2]
status: accepted
date: 2026-04-23
created: 2026-04-23
updated: 2026-04-23
---

# ADR-0005 — sqlc para queries tipadas

## Status

accepted — 2026-04-23 (herdado de legado)

## Contexto

Queries SQL escritas à mão em Go com `database/sql` genérico → perde tipo, erros em runtime. ORMs (GORM, Ent) escondem SQL e geram queries ineficientes. Precisamos: SQL explícito, tipos Go gerados, migrations controladas.

## Decisão

**sqlc** gera queries Go tipadas a partir de arquivos `.sql` com anotações `-- name:`.

```sql
-- name: GetVoteByAgendaAndVoter :one
SELECT tenant_id, id, agenda_item_id, voter_id, option, weight, cast_at
FROM assembly.votes
WHERE tenant_id = $1 AND agenda_item_id = $2 AND voter_id = $3;
```

Gera `func (q *Queries) GetVoteByAgendaAndVoter(ctx, args) (Vote, error)` com tipos Go.

### Regras

- **Arquivos `.sql` em `migrations/<module>/sql/`**, um por entidade.
- **SELECT *** proibido (A-021 histórico) — listar colunas.
- **Mapper explícito** entre tipo sqlc-gerado e entidade de domínio (row ≠ entity).
- **sqlc.yaml** versionado; geração reproduzível.

## Consequências

### Positivas

- Tipos Go gerados sem reflexão → zero overhead runtime.
- SQL explícito — DBA pode revisar; EXPLAIN ANALYZE direto.
- Erros de tipo em compile time.
- Integração pgx/v5 oficial.

### Negativas

- Complexa feature SQL (CTEs recursivas, window functions) pode quebrar parser sqlc — fallback `db.QueryContext` manual.
- Necessita `sqlc generate` no pipeline (plugin IDE ou make target).
- Não faz migration management (→ goose, ver [[0007-goose-migrations-partitioned]]).

## Alternativas consideradas

1. **pgx direto + manual `Scan`** — verboso, mais suscetível a erro.
2. **GORM** — ORM pesado, gera queries ruins (N+1), esconde SQL.
3. **Ent (Facebook)** — schema-first estilo grafo; overhead cognitivo alto; comunidade menor.
4. **Gorm v2 + manual query** — híbrido; sqlc é melhor pra esse caso.

## Referências

- [[../clean-arch-mapping]] §4 (database layer)
- [[0006-pgx-v5]]
- [[0007-goose-migrations-partitioned]]
- [[../patterns]] §2 (Repository)
