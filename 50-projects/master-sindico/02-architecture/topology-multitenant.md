---
title: Topologia multi-tenant — row-based + PK composto
type: guide
tags: [architecture, multi-tenant, postgres, rls, master-sindico, v2]
source: 13-research/google-arch + 13-research/beyond-corp + 13-research/linkedin
created: 2026-04-23
updated: 2026-04-23
aliases: [multi-tenancy, tenant-isolation]
---

# Topologia multi-tenant — row-based + PK composto

Escolha canônica de topologia multi-tenant para o Master Síndico e as razões. Decisão fundadora em [[adr/0021-multi-tenant-row-based]].

> Herança direta: [[../13-research/google-arch/_destilado]] §2.1 (Spanner multi-tenancy) + [[../13-research/beyond-corp/_destilado]] §3.6 (defesa em profundidade) + [[../13-research/linkedin/_destilado]] §2 (grafo raso cabe em Postgres).

---

## 1. Escolha canônica

**Row-based shared DB + `tenant_id` em PK composto `(tenant_id, ulid)` + PostgreSQL Row-Level Security (RLS) default-on como defense-in-depth.**

> **Revisão Fase 9 (Q-024 + Q-030 + G-034)**: RLS é **default-on obrigatório** em toda tabela com `tenant_id` ([[adr/0034-postgres-rls-default-on-tenant-guard]]). Opt-out explícito só para tabelas globais. Middleware `TenantBoundaryGuard` compara `path.tenant_id == ctx.tenant_id` em rotas com `{tenant_id}` no path (INV-122). Linter `rls-check.sh` bloqueia migration que cria tabela com `tenant_id` sem `ENABLE ROW LEVEL SECURITY + CREATE POLICY`.

- **1 PostgreSQL cluster**, 1 database, N schemas lógicos (um por módulo apenas para organização de migrations — não isolamento).
- Toda tabela de domínio carrega `tenant_id BYTEA NOT NULL` (ULID bytes).
- **PK composto**: `(tenant_id, id)` onde `id` é ULID/UUIDv7 time-ordered.
- **Todo índice** começa por `tenant_id` (locality, cache B-tree, prep para sharding físico).
- **RLS policy default-on** por tabela (Fase 9): `AS RESTRICTIVE ... USING (tenant_id = current_setting('app.tenant_id', true)::bytea) WITH CHECK (tenant_id = current_setting('app.tenant_id', true)::bytea)`. Admin bypass via policy `admin_bypass PERMISSIVE TO ms_admin_role USING (true)` em sessão MFA step-up. Ver ADR-0034.
- Middleware HTTP/worker seta `SET LOCAL app.tenant_id = ...` no início de cada tx.

### 1.1 Conceito de "tenant" no master-sindico

- `tenant_id` ≡ `condominium_id` para dados de governança/assembly/content.
- `tenant_id` também engloba **tenant corporativo** quando uma administradora gerencia múltiplos condomínios — ver [[adr/0021-multi-tenant-row-based]] §trade-offs.
- Dados **cross-tenant legítimos** (catálogo global de empresas no Connect Me, plans Stripe) ficam em tabelas **globais** sem `tenant_id` e com `RLS OFF`; mudança explicita-se em migration.

---

## 2. Alternativas consideradas e rejeitadas

### 2.1 Instance-per-tenant (1 PostgreSQL por condomínio)

- ❌ **Custo operacional proibitivo**. Centenas/milhares de condomínios M2+ ⇒ N instâncias para provisionar, backup, migrar.
- ❌ Migrações schema: N vezes, sincronização frágil.
- ❌ Sem global view sem ETL.
- ✅ Isolamento total (blast radius = 1 tenant).

**Quando reconsiderar**: cliente enterprise regulado (grande administradora com compliance SOC2 exigindo isolamento físico) — M3+ opção "dedicated DB tier". Source: [[../13-research/google-arch/_destilado]] §2.1.

### 2.2 Database-per-tenant (1 DB por condomínio, mesma instância)

- ❌ Migrações: N databases, ainda frágil.
- ❌ Connection pooling explode (1 pool por DB).
- ✅ Isolamento lógico mais forte que row-based.

**Quando reconsiderar**: M3 "enterprise tier" — administradoras com 50+ condomínios que pagam premium por DB próprio dentro da mesma infra. Source: [[../13-research/google-arch/_destilado]] §2.1.

### 2.3 Schema-per-tenant

- ❌ PostgreSQL não escala schemas indefinidamente (catalog bloat a partir de ~5k schemas).
- ❌ Migrações: N schemas × M tabelas.
- ✅ Isolamento lógico razoável.

Rejeitado sem reserva — Postgres não é bom com muitos schemas.

### 2.4 Pool pattern (N schemas reutilizados com mapping tenant→schema)

- ❌ Complexidade operacional para pouca vantagem vs row-based.
- Rejeitado.

### 2.5 Row-based puro (sem PK composto)

- Alternativa "barata": só `WHERE tenant_id = $1` em toda query, PK = `id`.
- ❌ Sem locality em B-tree: rows de tenants diferentes interleaveadas.
- ❌ Prepara mal para sharding físico futuro.
- ✅ Simples de implementar.

**Descartado**: complexidade marginal de PK composto é baixa e trava-se o futuro sem ela.

---

## 3. Trade-offs da escolha row-based + PK composto

| Aspecto | Ganho | Custo |
|---|---|---|
| Isolamento | ABAC + RLS + tenant_id em todo índice + cross-tenant tests | Não é isolamento físico; breach de auth ainda pode ler DB |
| Operacional | 1 DB, 1 set de migrations, 1 pool | Requer disciplina: **toda query** tem tenant_id |
| Custo $ | Baixo (shared resources) | — |
| Performance | Locality B-tree com PK composto | Tenant grande pode "noisy-neighbor" outros — mitigação: connection pool limits por tenant em M3+ |
| Escalabilidade | Suficiente para ~1M condomínios no mesmo cluster | Além disso: particionamento por hash(tenant_id) ou shards físicos |
| Complexidade código | Middleware `SET LOCAL app.tenant_id` + cross-tenant test harness | Baixa |
| Migração para outro pattern | PK composto já tem chave de sharding pronta | Restantes |

**Citação herdada** ([[../13-research/google-arch/_destilado]] §2.1, fonte Spanner docs):
> "Resource isolation: instance patterns prevent noisy neighbor problems, while row and database patterns allow tenant workloads to compete for shared resources. Row patterns eliminate per-tenant minimums."

---

## 4. Discriminator Column Pattern

Toda tabela de domínio segue o template:

```sql
CREATE TABLE <module>.<entity> (
    tenant_id   BYTEA NOT NULL,                      -- discriminator
    id          BYTEA NOT NULL,                      -- ULID bytes (16 bytes)
    -- campos específicos
    created_at  TIMESTAMPTZ NOT NULL DEFAULT now(),
    updated_at  TIMESTAMPTZ NOT NULL DEFAULT now(),
    PRIMARY KEY (tenant_id, id)
);

-- índices sempre começam por tenant_id
CREATE INDEX ON <module>.<entity> (tenant_id, created_at DESC);
CREATE INDEX ON <module>.<entity> (tenant_id, <fk>);

-- RLS opcional (ativar em tabelas sensíveis)
ALTER TABLE <module>.<entity> ENABLE ROW LEVEL SECURITY;
CREATE POLICY tenant_isolation ON <module>.<entity>
    USING (tenant_id = current_setting('app.tenant_id', true)::bytea);
```

### Exceções — tabelas globais (sem tenant_id)

- `identity.users` — User tem identidade única (CPF/CNPJ); uma mesma pessoa é síndico no condo A e morador no B. Escopo por tenant é no `memberships`.
- `billing.plans` — catálogo Stripe.
- `commercial.companies_global_registry` — CNPJ é nacional; empresa pode atender múltiplos condomínios.
- `content.moderation_rules` — regras globais (política de plataforma).

Essas tabelas são **read-mostly** e têm **write restrito a admin master** (ABAC).

---

## 5. Subdomain routing (opcional, M2+)

Duas opções:

### 5.1 Path-based (M1 default)

- `GET https://app.mastersindico.com.br/api/v1/condominiums/:id/...`
- `tenant_id` vem do JWT (Zitadel claim `org_id` → membership lookup → tenant atual do user nessa request).

### 5.2 Subdomain-based (M2+ opcional para administradoras)

- `https://<slug>.mastersindico.com.br/api/v1/...` onde `<slug>` é identificador do condomínio ou administradora.
- DNS wildcard `*.mastersindico.com.br` → CDN → API.
- Middleware resolve slug → tenant_id antes da auth.
- Vantagem: branding + marketing + isolamento visual.
- Custo: DNS wildcard TLS (Let's Encrypt wildcard), complexidade de roteamento.

**Decisão M1**: path-based. Subdomain só entra M2+ se cliente enterprise pedir (exemplo: administradora quer `minhaadm.mastersindico.com.br`).

---

## 6. Cross-tenant queries — proibidas por padrão

### 6.1 Regra dura

Toda query que toca tabela com `tenant_id` DEVE ter `WHERE tenant_id = $1`. Violação = falha no CI.

### 6.2 Enforcement

1. **Linter sqlc custom** — scanner grep nas queries `.sql` que contêm `tenant_id` coluna mas não filtro no WHERE. Rejeita no CI.
2. **RLS policy ativa em tabelas sensíveis** — segunda barreira mesmo que dev esqueça.
3. **Middleware `SET LOCAL app.tenant_id`** — definido na entrada da tx; se não for setado, RLS retorna 0 rows ou erro.
4. **Cross-tenant test harness** — matriz ABAC com 100+ casos (usuário do tenant A tenta ler recurso do tenant B; deve resultar em 404/403).

### 6.3 Admin master — exceção controlada

- Role `platform_admin` pode ler cross-tenant **via endpoints específicos** (`/admin/v1/...`), com step-up MFA.
- RLS não se aplica quando `SET LOCAL app.bypass_rls = 'true'` é explicitamente setado — **apenas em `/admin/v1/*` autenticado**.
- Logs de acesso cross-tenant gravam em `lgpd_logs` (audit trail WORM) por 5 anos.

### 6.4 Exceção: write-only cross-tenant (global catalog)

Tabelas sem `tenant_id` (plans, companies_global_registry) são permitidas mas:
- Write restringido a admin.
- Read liberado a todos os tenants (catálogo público).

---

## 7. Testes sistemáticos de isolamento

Herança [[../CLAUDE]] §3.11 — "toda query escopada por condomínio tem `WHERE condominium_id = $tenant_id`; cross-tenant testado sistematicamente".

### Suites obrigatórias em `09-testing/`

1. **Tenant Boundary Matrix** (PBT — property-based test):
   - Given: 10 tenants aleatórios × 50 ações × 100 endpoints.
   - When: user do tenant A chama endpoint com ID de entidade do tenant B.
   - Then: 404 (não existe pra você) ou 403 (explicitamente negado).
2. **RLS policy tests** — valida que `SET LOCAL app.tenant_id = 'X'` filtra corretamente; `SET LOCAL app.bypass_rls = 'true'` bypassa (apenas via código admin).
3. **Migration consistency** — toda migration que cria tabela é reviewed para ter `tenant_id` (exceto a lista de exceções globais).

---

## 8. Roadmap de escala

| Fase | Tenants | Estratégia |
|---|---|---|
| **M1 (2026-05-08)** | 1–50 | Row-based + PK composto + RLS em tabelas sensíveis. 1 cluster Railway. |
| **M2 (2026-Q3)** | 50–500 | Adicionar read replicas + connection pool tuning por tenant. |
| **M3 (2027 Q1)** | 500–5k | Particionamento Postgres nativo por `hash(tenant_id)` em tabelas grandes (timeline_entries, events). Read replicas regionais SP/RJ. |
| **M4+ (2027 H2+)** | 5k–50k | Opção "dedicated DB tier" para enterprise (database-per-tenant). Sharding físico por hash(tenant_id) usando Citus ou migração para CockroachDB se transacional cross-shard crítica. |
| **M5+ (visionário)** | 50k+ | Re-avaliar Spanner/CockroachDB distributed SQL ou arquitetura microserviço com DB per BC. |

Fonte herdada: [[../13-research/google-arch/_destilado]] §2.3 (ULID/UUIDv7 evita hotspotting), §9 (matriz de aplicabilidade).

---

## 9. Defense in depth — barreiras múltiplas

```
Request → [ABAC middleware]      (matrix ordered: admin_bypass → role → action → tenant → scope)
       → [TenantIsolation mw]    (SET LOCAL app.tenant_id = ...)
       → [Handler]
       → [Use Case]              (use case adiciona tenant_id em todo aggregate criado)
       → [Repo]                  (sqlc queries parametrizadas com tenant_id)
       → [PostgreSQL RLS]        (policy filtra por app.tenant_id)
       → [Index locality]        (PK composto (tenant_id, id) localiza páginas)
       → [Audit trail]           (lgpd_logs append-only 5 anos)
```

Cada barreira é redundante. **Se atacante comprometer ABAC, RLS ainda bloqueia**. [[../13-research/beyond-corp/_destilado]] §1.2 (princípio "assume breach").

---

## 10. ⚠️ Pendências

- **Citus vs Postgres particionamento nativo** — avaliar em ADR M3 quando tabelas cruzarem ~100M rows.
- ✅ ~~**RLS por tabela**~~ — **DECIDIDO em [[adr/0034-postgres-rls-default-on-tenant-guard|ADR-0034]]**: RLS default-on em **toda tabela com `tenant_id`**. Opt-out explícito só para tabelas globais. Atualizado 2026-04-24 (D-MT-9). Ver [[../11-audit/audit-log/2026-04-24-multitenant-consolidation-map]] para 6 conflitos detectados + 9 D-MT-### pendentes.
- **`tenant_id` em Redis keys** — padrão `tenant:{id}:...`; validar que todo cache_key inclui prefixo. Linter a escrever.

---

## 11. Vizinhos

- [[adr/0021-multi-tenant-row-based]]
- [[adr/0009-uuidv7-ulid-pk]]
- [[adr/0012-abac-redis-cache]]
- [[clean-arch-mapping]]
- [[c4-components]] — todo aggregate carrega tenant_id
- [[../13-research/google-arch/_destilado]]
- [[../13-research/beyond-corp/_destilado]]
