---
title: ADR-0034 — PostgreSQL RLS default-on + TenantBoundaryGuard middleware
type: adr
tags: [adr, master-sindico, v2, multi-tenant, security, rls, postgres, beyond-corp]
source: Q-024 + Q-030 + G-034 consolidado 2026-04-23 + beyond-corp research R4
created: 2026-04-23
updated: 2026-04-23
status: accepted
deciders: arquiteto, pentester, BeyondCorp architect
consulted: Full-stack senior, QA
---

# ADR-0034 — PostgreSQL RLS default-on + TenantBoundaryGuard middleware

## Contexto

[[0021-multi-tenant-row-based]] definiu topologia row-based com `tenant_id` + PK composto. RLS declarado **opcional como defense-in-depth**. Hoje RLS cobre ~5 tabelas sensíveis; o resto depende de convenção "toda query em sqlc tem `WHERE tenant_id = $1`".

Q-024 e Q-030 identificaram furos concretos:

1. **Cross-tenant leak via `tenant_id` em path** — rotas como `GET /tenants/{id}/timeline` aceitam `tenant_id` em path. User A manda request com tenant_id de B no path; handler (sem middleware comparador) injeta na query sqlc; query retorna timeline de B. INV-089/108 declaram isolation mas nenhum middleware `ctx.tenant_id == path.tenant_id` existe.

2. **Search tsvector cross-tenant** — queries full-text em Banco de Talentos/Connect Me podem ser construídas sem filtro `tenant_id` se o developer esquecer. Handler com bug em 1 endpoint = vazamento amplo.

3. **Defesa monocamada frágil** — INV-089 depende só de sqlc query. Handler custom (fora de sqlc), query dinâmica, ou injection = bypass.

Pesquisa beyond-corp R4 e Spanner auth-per-row convergem: **inverter o default de RLS** — opt-in quando global (plans, country codes), opt-out explícito em migration.

## Decisão

### Parte A — RLS default-on para toda tabela com `tenant_id`

Todas as tabelas de domínio que carregam `tenant_id BYTEA NOT NULL` ganham:

```sql
ALTER TABLE <module>.<entity> ENABLE ROW LEVEL SECURITY;

CREATE POLICY tenant_isolation ON <module>.<entity>
  AS RESTRICTIVE
  FOR ALL
  TO app_user
  USING (tenant_id = current_setting('app.tenant_id', true)::bytea)
  WITH CHECK (tenant_id = current_setting('app.tenant_id', true)::bytea);
```

`AS RESTRICTIVE` garante que policy é AND-composta com outras policies (não OR). `WITH CHECK` garante INSERT/UPDATE também respeitam tenant atual.

Tabelas globais (plans, country_codes, zitadel_roles) permanecem sem RLS — marcação explícita em migration com comment `-- GLOBAL (no RLS by design, reviewed in ADR-0034)`.

### Parte B — Middleware seta `SET LOCAL app.tenant_id` + comparador `TenantBoundaryGuard`

Em cada request HTTP/worker:

```
1. Auth middleware resolve ctx.tenant_id via JWT claim + membership lookup.
2. TenantBoundaryGuard: se rota tem {tenant_id} em path/query,
   compara path/query vs ctx.tenant_id — mismatch = 403 + audit.
3. Transaction wrapper executa SET LOCAL app.tenant_id = <ctx.tenant_id>::bytea
   ANTES de qualquer query de domínio.
4. Policy RLS valida tenant_id em toda leitura/escrita.
```

`TenantBoundaryGuard` é middleware obrigatório em todas as rotas que declaram `tenant_id` em path/query. Rotas globais (plans, login, webhooks) explicitamente isentas via tag `// global-route`.

### Parte C — Policy para admin bypass legítimo

Admin Master Síndico (role `ms_admin_role`) precisa ver cross-tenant para moderação. Solução: policy separada:

```sql
CREATE POLICY tenant_isolation ON <module>.<entity>
  AS RESTRICTIVE
  FOR ALL
  TO app_user
  USING (tenant_id = current_setting('app.tenant_id', true)::bytea);

CREATE POLICY admin_bypass ON <module>.<entity>
  AS PERMISSIVE
  FOR ALL
  TO ms_admin_role
  USING (true);  -- admin vê tudo; bypass é auditado app-level
```

`ms_admin_role` é role separado usado apenas em sessões admin MFA step-up verificadas. App distingue pool `app_user` (default) vs `ms_admin_role` (elevated) conforme session flag.

### Parte D — Migration enforcer

Nova regra em CI: migrations que criam tabela com coluna `tenant_id` **sem** `ENABLE ROW LEVEL SECURITY + CREATE POLICY` falham linter. Script `09-testing/migration-linter/rls-check.sh`.

## Alternativas consideradas

1. **RLS só app-level em vez de DB** — é o estado atual com falha; rejeitado.
2. **Sem RLS, só type-driven tenant_id (INV-108)** — é complementar, não substituto. Type system protege de assinaturas erradas; RLS protege de query errada. Ambas são necessárias.
3. **Schema-per-tenant** — explorada em ADR-0021 e rejeitada (10k+ condomínios = explosão de schemas). Mantida a decisão.
4. **pg_bouncer com `SET tenant_id` via prepared statements** — alternativa de performance; complexa de operar. Retomar se RLS impactar > 10% latência p95.

## Consequências

### Positivas

- Handler com bug em `WHERE tenant_id` = bloqueado pela policy (query retorna 0 rows em vez de vazar).
- Query dinâmica, raw SQL, ou injection = RLS filtra.
- Cross-tenant leak via path (Q-024) = bloqueado pelo TenantBoundaryGuard antes do handler.
- Alinhamento com BeyondCorp + Spanner defense-in-depth.

### Negativas

- Overhead RLS: Postgres avalia policy a cada row. Tests mostram < 3% overhead com policy simples em índice; aceitável.
- Toda conexão do pool precisa `SET LOCAL app.tenant_id` antes da primeira query. Middleware garante; esqueceu = RLS retorna 0 rows (comportamento seguro default).
- Testes de desenvolvimento precisam de `SET app.tenant_id` manual (via fixture `testfixtures`).
- Debugging com `psql` como superuser bypassa RLS — positivo para suporte, doc explicita.

## Implementação

- **Esta semana (pré-Sprint 10)**:
  - Migration `20260424100000_rls_default_on.up.sql` adiciona ENABLE RLS + CREATE POLICY em todas tabelas com `tenant_id`. `.down.sql` desfaz.
  - Middleware Go `TenantBoundaryGuard(pathParam, queryParam string)` + tests.
  - CI linter `rls-check.sh` bloqueia migrations sem RLS.
  - INV-121 (RLS default-on) + INV-122 (TenantBoundaryGuard) adicionados.
- **Sprint 10**:
  - Pool Postgres separa `app_user` e `ms_admin_role` via sessão MFA step-up.
  - PBT cross-tenant: fuzz 500 cenários `∀ user_A ∈ tenant_A, ∀ resource_B ∈ tenant_B → access denied`.
- **Runbook**: `06-execution/playbooks/rls-debug.md` — como rodar queries cross-tenant em suporte.

## Referências

- Q-024, Q-030 — [[../../11-audit/audit-log/2026-04-23-quebras-negocio-systematic]]
- G-034 — [[../../11-audit/audit-log/2026-04-23-ops-bigtech-gaps]]
- beyond-corp R4 — [[../../13-research/beyond-corp/_aplicacoes-master-sindico]]
- [[0021-multi-tenant-row-based]] (ADR fundadora, complementada)
- [[0029-tenant-id-typed]] (INV-108 type-driven — complementar)
- INV-089 (tenant isolation) promovida para DB-level
- Novos INV-121 (RLS default-on), INV-122 (TenantBoundaryGuard)
- D-089 — STATE

## Vizinhos

- [[0021-multi-tenant-row-based]]
- [[0029-tenant-id-typed]]
- [[../topology-multitenant]]
- [[../../08-security/BEYOND_CORP]]
- [[../../01-domain/invariants]]
