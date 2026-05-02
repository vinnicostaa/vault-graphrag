---
title: ADR-0007 — Goose migrations particionadas por módulo
type: adr
tags: [adr, database, migrations, goose, master-sindico, v2]
status: accepted
date: 2026-04-23
created: 2026-04-23
updated: 2026-04-23
---

# ADR-0007 — Goose migrations particionadas por módulo

## Status

accepted — 2026-04-23 (herdado de legado)

## Contexto

Migrations SQL precisam: versioning, up/down, CI-safe, reviewable em PR, não pisar entre módulos. Com 7 BCs escritos em paralelo, numeração global causa conflitos de merge.

## Decisão

**pressly/goose v3** como migration tool. Migrations particionadas **por módulo**:

```
migrations/
├── identity/
│   ├── 0001_create_users.sql
│   ├── 0002_create_sessions.sql
│   └── 0003_create_devices.sql
├── billing/
│   ├── 0001_create_plans.sql
│   └── 0002_create_subscriptions.sql
├── institutional/
│   ├── 0001_create_condominiums.sql
│   ├── 0002_create_units.sql
│   └── 0003_create_timeline_entries.sql
├── commercial/
├── content/
├── assembly/
└── shared/                   # outbox_events, saga_instances, processed_events, rate_limit
    ├── 0001_create_outbox.sql
    └── 0002_create_saga_instances.sql
```

Tabela de tracking: 1 por módulo (`<module>_migrations_version`).

Runner CLI: `cmd/migrate/main.go --module=institutional up|down|redo|status`.

## Consequências

### Positivas

- PRs concorrentes em BCs diferentes não colidem em numeração.
- Rollback granular por módulo.
- Dev local roda migrations só do módulo que está tocando.

### Negativas

- Complexidade extra: pipeline CI precisa rodar `migrate all` em ordem definida (shared → identity → billing → ...).
- Migration cross-module é rara mas precisa convenção (coloca em `shared/` ou no módulo "downstream").

## Regras

1. **Expand-contract** obrigatório: migration destrutiva (DROP COLUMN, NOT NULL em tabela populada) passa por 3 releases (expand → migrate data → contract).
2. **Zero-downtime** — migration não bloqueia tabela grande; `CREATE INDEX CONCURRENTLY`, `NOT VALID` + `VALIDATE` patterns.
3. **CHECK constraints** em coluna (fração ideal, status enum) em vez de validação só app.
4. **FK cascata** documentada — REFERENCES com ON DELETE CASCADE somente se semanticamente faz sentido.

## Alternativas consideradas

1. **golang-migrate** — bom, mas menos flexível na organização por módulo.
2. **Atlas** (Ariga) — schema-as-code; muita curva; ótimo para M3+.
3. **Flyway** — JVM dependency; descartado.
4. **Liquibase** — XML/YAML — verboso.

## Referências

- [[../topology-multitenant]] §4 (schema template)
- [[../c4-components]] §1 (estrutura canônica)
- Histórico: legado já usa goose, paridade.
