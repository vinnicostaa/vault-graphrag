---
title: Source 07 — Implement multi-tenancy in Spanner
type: source
tags: [research, google, spanner, multi-tenancy, saas, sharding, tenant-isolation]
source: https://docs.cloud.google.com/spanner/docs/implement-multi-tenancy
created: 2026-04-23
updated: 2026-04-23
aliases: [spanner-multi-tenant]
---

# Source 07 — Implement multi-tenancy in Spanner

- **URL primária**: https://docs.cloud.google.com/spanner/docs/implement-multi-tenancy
- **Relacionado**: https://medium.com/google-cloud/implementing-multi-tenancy-in-cloud-spanner-3afe19605d8e (Christoph Bussler, Google Cloud)
- **Data fetch**: 2026-04-23

## Trechos rastreáveis (WebFetch + WebSearch 2026-04-23)

Três padrões principais (docs.cloud.google.com):

### 1. Instance-per-tenant (isolamento máximo)
- Cada tenant = seu próprio Spanner instance + database.
- "Allows the use of separate Google Cloud projects to achieve separate trust boundaries".
- Trade-off: "significant operational overhead—managing numerous instances requires extensive monitoring, maintenance, and scaling efforts".

### 2. Database-per-tenant (isolamento lógico)
- Múltiplos tenants em uma instance, database por tenant. Spanner suporta **até 100 databases por instance**.
- "Complete logical isolation on the database level".
- Todos compartilham topology de replicação e infra subjacente, exceto se usar geo-partitioning.

### 3. Shared database com tenant ID (row-based)
- Todos tenants nas mesmas tables, `tenant_id` como primeiro elemento do primary key.
- "Interleaving tells Spanner to physically co-locate child rows with their parent row" → locality automática.
- "Ensures data locality for tenant-scoped queries and lets Spanner automatically distribute the load across nodes".

### Trade-offs explícitos
- "Instance patterns prevent noisy neighbor problems, while row and database patterns allow tenant workloads to compete for shared resources".
- "Row patterns eliminate per-tenant minimums; instance patterns require 100 processing units minimum".
- Sharding: "sequential UUIDs or composite keys that avoid hotspotting (e.g., tenant_id + timestamp) and avoiding randomly generated UUIDs as primary keys in high-write tables, as they lead to write amplification due to uneven splits".

## Aplicabilidade ao Master Síndico

- **Direta M1**: **ALTÍSSIMA** — conceitualmente transpõe 1:1 pra PostgreSQL:

| Spanner pattern | Postgres equivalente | Decisão M1 |
|---|---|---|
| Instance-per-tenant | DB server separado | NÃO (custo) |
| Database-per-tenant | schema/DB separado | Candidato M3 enterprise |
| Row-based + tenant_id | `tenant_id` + RLS + PK composto | **SIM (default)** |

- **Ações concretas**:
  1. `tenant_id UUID NOT NULL` em toda tabela de domínio.
  2. RLS policy default-deny: `USING (tenant_id = current_setting('app.tenant_id')::uuid)`.
  3. Middleware Gin: `SET LOCAL app.tenant_id = ...` no começo da tx.
  4. PK composto `(tenant_id, ulid)` em tabelas high-write — sem UUIDv4 puro (write amplification).
  5. Índices sempre iniciando com `tenant_id`.
- **ADR obrigatório**: ADR-00XX "Multi-tenancy strategy" referenciando este source como comparativo + justificativa.

## Links relacionados

- [[source-01]] Spanner base.
- [[source-08]] F1 (hierarchical schema — inspira PK composto).
- [[source-12]] Identity Platform multi-tenancy (tenants em nível auth).
- Destilado: [[_destilado#2.1 Multi-tenant]], [[_destilado#2.3 Sharding]].
