---
title: Multi-tenant — Mapa de consolidação 2026-04-24
type: audit
tags:
  - audit
  - multi-tenant
  - consolidation
  - master-sindico
date: 2026-04-24T00:00:00.000Z
sessions:
  - systematic-review-2026-04-24
status: pending-decisions
---

# Multi-tenant — Mapa de Consolidação

> Investigação realizada via subagente Opus em 2026-04-24 a pedido do João: *"a parte do multi-tenant precisa ser revisada com o que ja tem no vault"*. Vault: `/home/vinni/Documentos/Obsidian Vault/automation-agents/vault/`.

## TL;DR

23+ documentos vivos sobre multi-tenant no vault. **6 conflitos confirmados** entre arquivos canônicos. **3 docs satélites referenciados mas inexistentes**. Documento canônico `02-architecture/topology-multitenant.md` é sólido mas não cobre: deletion-by-tenant, runbook cross-tenant test, observabilidade por tenant, tenant kinds formal. Recomendação: manter `topology-multitenant.md` como hub canônico e expandir com 6 seções novas + criar 4 satélites.

## 6 Conflitos confirmados (🔁)

### 🔁 C1 — Representação de `tenant_id`
- ADR-0021 + topology-multitenant: `tenant_id BYTEA NOT NULL` (ULID 16 bytes)
- ADR-0029: tipos Go `CondominiumID/CompanyID/UserID/NeighborhoodID` envolvem `uuid.UUID`
- ADR-0009 update: PG18 UUIDv7 nativo (não ULID)
- Backend real: coluna chama `condominium_id` (não `tenant_id`); coluna `tenant_kind` discriminadora **não existe**

### 🔁 C2 — RLS opcional (ADR-0021) vs default-on (ADR-0034)
- ADR-0021 §Decisão regra 4 (accepted 2026-04-23): "RLS policy ativada em **tabelas sensíveis**"
- ADR-0034 §Parte A (accepted 2026-04-23 — mesma data): "RLS é **default-on obrigatório**"
- topology-multitenant §1 reconcilia mas §10 ainda lista pendência stale
- Backend: RLS não implementado ainda — Sprint 10 task

### 🔁 C3 — Tenant resolution flow
- ADR-0029: path param é autoridade quando há `:id`; header é hint only
- topology-multitenant §5.1: JWT claim `org_id` + membership lookup
- backend/security.md §13.1: `AuthUser.CondominiumIDs[]` populado via JOIN — sem path/JWT direto
- ADR-0034 Parte B: TenantBoundaryGuard compara path vs ctx.tenant_id

### 🔁 C4 — Admin bypass (4 mecanismos diferentes)
- ADR-0034 Parte C: PG role `ms_admin_role`
- topology-multitenant §6.3: `SET LOCAL app.bypass_rls = 'true'`
- BEYOND_CORP §3.2: ABAC `is_admin=true` primeiro check
- backend/security §5.2: `role=="admin"` hardcoded no engine

### 🔁 C5 — TenantBoundaryGuard middleware ausente do código
- ADR-0034 + INV-122: declaram obrigatório
- backend/architecture §5 + backend/security §3: stack listado **NÃO inclui**
- Sprint 10 task: implementar

### 🔁 C6 — Tenant kinds (mais profundo)
- ADR-0029: 4 kinds (`condominium | company | user | neighborhood`)
- bounded-contexts §1.1: 2 kinds (condominium + corporate/company)
- ADR-0039 (IAM Cloudflare): redefine em `Member + Group + Membership + Role + PG + Quota + Plan`
- Backend real: 1 kind (`condominium_id`)

## Docs satélites referenciados mas inexistentes

- `01-domain/patterns/tenant-isolation-row-based.md` (referenciado em `01-domain/patterns/_moc.md`)
- `06-execution/playbooks/rls-debug.md` (referenciado em ADR-0034)
- `06-execution/playbooks/dr-drill.md` (referenciado em INV-123 — verificar)

## 9 Decisões pendentes (D-MT-###) propostas

| ID | Decisão | Owner | Prazo |
|---|---|---|---|
| **D-MT-1** | Criar matriz `(role, kind, action) → permitido` em `08-security/cross-tenant-roles-matrix.md` | arquiteto + pentester | M1 |
| **D-MT-2** 🔴 | Reconciliar tenant kinds: 4 (ADR-0029) vs 2 (legado) vs ADR-0039 redefine | arquiteto + product | **pré-Sprint 10** (crítico — mexer schema depois é caro) |
| **D-MT-3** | Criar `01-domain/patterns/tenant-isolation-row-based.md` com pattern Go canônico | tech lead backend | M1 |
| **D-MT-4** | Criar `06-execution/playbooks/rls-debug.md` (suporte sem bypass audit) | SRE | M1 (junto com migration RLS) |
| **D-MT-5** | Observability: estratégia cardinality `tenant_id` em métricas Prometheus | SRE | M2 |
| **D-MT-6** | Backup/restore por tenant — runbook PITR single-tenant | SRE + DPO | M2 |
| **D-MT-7** | Deletion LGPD por tenant (offboarding com timeline imutável) | DPO + arquiteto | M2 |
| **D-MT-8** | Tenant resolution flow oficial (resolve C3 — diagrama header→JWT→path→membership) | arquiteto | pré-Sprint 10 |
| **D-MT-9** | Atualizar topology-multitenant §10 ("RLS por tabela" stale) — apontar ADR-0034 default-on | arquiteto | imediato (texto stale) |

## Plano de edição (23 ações)

### Imediatos (texto stale, low-risk — aplicáveis sem dual-check)
1. `topology-multitenant.md §10`: remover bullet "RLS por tabela: decidir..." → substituir por "✅ Decidido em ADR-0034"
2. `adr/0021-multi-tenant-row-based.md` §Decisão regra 4: substituir "RLS policy ativada em tabelas sensíveis" → "RLS default-on (atualizado por ADR-0034)"
3. `adr/0021-multi-tenant-row-based.md` §Schema template: substituir "RLS opcional" → "RLS default-on (ver ADR-0034)"
4. `12-docs/adr-index.md`: ajustar entrada "ADR-0004 — Multi-tenant estrito" → ADR-0021 (vestígio renumeração pre-v2)
5. `01-domain/patterns/_moc.md`: substituir "A criar — tenant-isolation-row-based.md" por entrada ativa após criar a nota

### Estruturais (criar seções em topology-multitenant.md — exigem D-MT-2/8 fechadas)
6-11. Adicionar §1.2 Tenant Kinds, §5 Tenant Resolution Chain, §6.5 Admin Bypass (escolher 1 mecanismo), §7.5 Observability, §10 Backup/restore, §11 Deletion LGPD

### Criar novos arquivos
12-17. tenant-isolation-row-based.md, rls-debug.md, cross-tenant-roles-matrix.md, cross-tenant-pbt.md, tenant-deletion-lgpd.md, tenant-restore-pitr.md

### Cross-link
18-21. Atualizar bounded-contexts.md, backend/security §3, backend/architecture §5, BEYOND_CORP §3.3 cache key, CLAUDE.md

### Arquivamento
22-23. Confirmar headers `_archive/` + `90-archive/`; documentar `Clients/MasterSindico/` como external legacy reference

## Aguardando decisão João

- D-MT-2 (kinds 4 vs 2 vs ADR-0039) é **crítica antes de Sprint 10**.
- D-MT-8 (tenant resolution flow) também precisa decisão antes de Sprint 10.
- D-MT-1 (matriz roles), D-MT-3 (pattern doc), D-MT-4 (rls-debug runbook): podem entrar em Sprint 10 se aprovados.
- D-MT-5/6/7: M2.

## Vizinhos

- [[../../02-architecture/topology-multitenant]]
- [[../../02-architecture/adr/0021-multi-tenant-row-based]]
- [[../../02-architecture/adr/0029-tenant-id-typed]]
- [[../../02-architecture/adr/0034-postgres-rls-default-on-tenant-guard]]
- [[../../02-architecture/adr/0039-iam-cloudflare-style-master-sindico]]
- [[../../STATE#D-125]]
- [[../AUDIT]]
