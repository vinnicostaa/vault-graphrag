---
title: ADR-0021 — Multi-tenant row-based + PK composto + RLS opcional
type: adr
tags: [adr, architecture, multi-tenant, postgres, rls, master-sindico, v2]
status: accepted
date: 2026-04-23
created: 2026-04-23
updated: 2026-04-23
---

# ADR-0021 — Multi-tenant row-based + PK composto (tenant_id, ulid) + RLS opcional

## Status

accepted — 2026-04-23

## Contexto

Master Síndico hospeda N condomínios no mesmo cluster Postgres. Precisamos:

- Isolamento entre tenants (cross-tenant proibido por padrão).
- Custo operacional baixo (não queremos N instâncias).
- Preparação para sharding futuro sem refactor.
- Defense in depth: múltiplas barreiras.

Research Google Spanner ([[../../13-research/google-arch/_destilado]] §2.1) formaliza 3 padrões: instance-per-tenant, database-per-tenant, row-based. Spanner docs: "Row patterns eliminate per-tenant minimums".

## Decisão

**Shared DB + `tenant_id` row-based + PK composto `(tenant_id, id)` + PostgreSQL RLS opcional em tabelas sensíveis**.

### Regras

1. Toda tabela de domínio carrega `tenant_id BYTEA NOT NULL`.
2. **PK composto**: `PRIMARY KEY (tenant_id, id)` onde `id` é ULID ([[0009-uuidv7-ulid-pk]]).
3. Todo índice secundário começa por `tenant_id` (locality + prep sharding).
4. RLS policy ativada em tabelas sensíveis (votes, minutes, memberships, invoices, timeline_entries).
5. Middleware seta `SET LOCAL app.tenant_id = ...` na abertura de toda tx.
6. Tabelas globais (users, plans, companies_global_registry, moderation_rules) **sem** `tenant_id` e RLS off — documentadas em lista explícita.

### Schema template

```sql
CREATE TABLE <module>.<entity> (
    tenant_id   BYTEA NOT NULL,
    id          BYTEA NOT NULL,
    -- campos específicos
    created_at  TIMESTAMPTZ NOT NULL DEFAULT now(),
    updated_at  TIMESTAMPTZ NOT NULL DEFAULT now(),
    PRIMARY KEY (tenant_id, id)
);
CREATE INDEX ON <module>.<entity> (tenant_id, created_at DESC);
```

### RLS

```sql
ALTER TABLE <table> ENABLE ROW LEVEL SECURITY;
CREATE POLICY tenant_isolation ON <table>
    USING (tenant_id = current_setting('app.tenant_id', true)::bytea);
```

## Consequências

### Positivas

- Custo operacional baixo (1 DB).
- Preparado para sharding via hash(tenant_id) M4+ sem refactor.
- Defense in depth: ABAC + RLS + tenant_id em query + audit.
- Locality B-tree via PK composto.

### Negativas

- Noisy neighbor possível — tenant grande pode sobrecarregar. Mitigação M3+: connection pool limits por tenant tier.
- Exige disciplina: **toda** query filtra tenant_id. Enforcement via linter sqlc + RLS.
- Não é isolamento físico (compliance enterprise M4+ pode pedir option dedicada).

## Enforcement

1. Linter sqlc custom — rejeita queries que tocam tabela com `tenant_id` sem WHERE.
2. RLS como segunda barreira.
3. Cross-tenant test harness obrigatório (100+ casos matrix).
4. Middleware `TenantIsolation` seta `SET LOCAL` e falha se tenant_id ausente.

## Alternativas rejeitadas

Ver [[../topology-multitenant]] §2 (detalhado): instance-per-tenant, database-per-tenant, schema-per-tenant, pool, row-based sem PK composto.

## Referências

- [[../../13-research/google-arch/_destilado]] §2.1, §2.2, §2.3
- [[../../13-research/beyond-corp/_destilado]] §3.6
- [[../../13-research/linkedin/_destilado]] §2 (grafo raso em PG)
- [[../topology-multitenant]] (canônico)
- [[0009-uuidv7-ulid-pk]]
- [[0012-abac-redis-cache]]
