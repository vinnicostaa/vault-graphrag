---
title: ADR-0009 — ULID / UUIDv7 como PK de todas tabelas de domínio
type: adr
tags: [adr, database, ulid, uuid, master-sindico, v2]
status: accepted
date: 2026-04-23
created: 2026-04-23
updated: 2026-04-23
---

# ADR-0009 — ULID / UUIDv7 como PK de todas tabelas de domínio

## Status

accepted — 2026-04-23 (update 2026-04-23 — ver seção abaixo)

## 2026-04-23 update — UUIDv7 nativo PG18 vira default em tabelas novas

Pós-[[../../11-audit/dual-check-log/2026-04-23-decisoes-arquiteturais]] (dual-check arch Fase 5) reviu que **PostgreSQL 18** (GA esperado Q3 2025, já disponível em RC) traz **`uuidv7()` nativo** como função interna, eliminando a necessidade de extensão `pg_uuidv7` ou função SQL custom.

**Mudança**:

- **Tabelas novas** (criadas em migrations pós-2026-05-01) usam `DEFAULT uuidv7()` no PK (PG18 nativo), armazenadas como `UUID` nativo do Postgres (16 bytes com benefício do tipo nativo sobre `BYTEA`).
- **Tabelas existentes** (legado + v2 pre-migration) continuam com ULID `BYTEA` — não há necessidade de migrar (custo alto, benefício marginal).
- **Geração server-side** continua permitida quando cross-region ou quando client precisa do ID antes do INSERT (ex: pre-seed cache).

**Impacto**:

- Linter CI passa a aceitar ambos `DEFAULT uuidv7()` (UUID nativo) **e** `BYTEA` ULID como PK válidos — rejeita apenas `UUIDv4` puro / `BIGSERIAL` para domain.
- API/JSON serialization: UUIDv7 é canônico `xxxxxxxx-xxxx-7xxx-xxxx-xxxxxxxxxxxx` (hyphen format) — clients parseiam nativamente.
- [[../../01-domain/invariants#INV-094]] (DomainEvent.id UUIDv7) fica alinhado automaticamente para eventos novos.

**Finding cobre**: reclassificação discutida em dual-check arch não gerou `A-DC-###` crítico — é refino de decisão.

## Contexto

PKs auto-increment (BIGSERIAL) quebram em sharding futuro e leak volume (N+1 aponta competidores). UUIDv4 puro perde locality em B-tree → write amplification em tabelas de alta escrita ([[../../13-research/google-arch/_destilado]] §2.3: "avoiding randomly generated UUIDs as primary keys in high-write tables, as they lead to write amplification"). Time-ordered IDs dão locality sem comprometer unicidade distribuída.

## Decisão

**ULID** (Crockford base32, 128 bits, time-ordered primeiros 48 bits) como PK de toda tabela de domínio.

- Armazenamento em PG: `BYTEA` (16 bytes) + CHECK `length = 16`. Economiza 12 bytes por row vs TEXT 26 chars.
- API/JSON: string Crockford base32 26 chars (lexicographicamente ordenável).
- Geração server-side em Go via lib `oklog/ulid/v2`.
- **UUIDv7** aceito como alternativa equivalente (mesma propriedade time-ordered); padronizamos ULID para não misturar.

### PK composto multi-tenant

`PRIMARY KEY (tenant_id BYTEA, id BYTEA)` em toda tabela de domínio — ver [[0021-multi-tenant-row-based]] e [[../topology-multitenant]].

## Consequências

### Positivas

- Locality B-tree: rows recentes ficam em páginas vizinhas → cache hit melhor.
- Ordenação natural por tempo (sem precisar `created_at DESC`).
- Preparado para sharding hash por tenant_id (time-ordered dentro de cada shard).
- Seguro para expor em URL (não-sequencial, não leak volume).

### Negativas

- Debug "olho humano" mais difícil que BIGINT (mas ferramentas resolvem).
- Custo marginal de geração (~100ns Go).
- Clientes que esperam UUID canônico hyphen-based precisam adapter.

## Regras

- **Nunca usar UUIDv4** em novas tabelas — linter CI bloqueia migration que cria coluna `UUID DEFAULT gen_random_uuid()` sem `v7`.
- **BIGSERIAL permitido** apenas em tabelas puramente técnicas (outbox id pode ser ULID; rate_limit counter pode ser BIGSERIAL).

## Alternativas consideradas

1. **UUIDv4** — write amplification em grandes tabelas; ver research.
2. **UUIDv7** — equivalente a ULID; padronizar um para clareza.
3. **Snowflake ID (Twitter-style)** — requer coordenação de node ID; ULID evita.
4. **BIGSERIAL** — leak volume, quebra em sharding.
5. **xid** — 12 bytes, elegante, mas menos conhecido.

## Referências

- [[../../13-research/google-arch/_destilado]] §2.3
- [[0021-multi-tenant-row-based]]
- [[0029-tenant-id-typed]] — tenant IDs tipados usam UUID nativo (PG18) em novos tipos.
- [[../topology-multitenant]] §1
- [[../../11-audit/dual-check-log/2026-04-23-decisoes-arquiteturais]] — update 2026-04-23 (PG18 nativo).
- PostgreSQL 18 UUID functions: https://www.postgresql.org/docs/18/functions-uuid.html
- RFC 9562 UUIDv7 (draft): https://datatracker.ietf.org/doc/html/rfc9562#section-5.7
- https://github.com/oklog/ulid
