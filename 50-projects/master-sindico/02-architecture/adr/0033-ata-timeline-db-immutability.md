---
title: ADR-0033 — Ata e Timeline imutáveis via REVOKE + trigger DB (não só app-level)
type: adr
tags: [adr, master-sindico, v2, assembly, institutional, security, db, postgres, immutability]
source: Q-020 consolidado 2026-04-23 + Fase 9 research review
created: 2026-04-23
updated: 2026-04-23
status: accepted
deciders: arquiteto, pentester, DPO
consulted: QA, Full-stack senior
---

# ADR-0033 — Ata (Minutes) e Timeline imutáveis via REVOKE + trigger DB

## Contexto

INV-024 (Timeline imutável após publicação) e INV-072 (Minutes imutável após `published` e `homologated`) hoje são **enforcement app-level** (handler rejeita UPDATE/DELETE) + **trigger PG opcional**. Problema encontrado em auditoria Q-020:

- DBA malicioso, incident de SQL injection em endpoint não-coberto, ou RCE em container Go → `UPDATE timeline_entries SET ... WHERE id = X` passa direto. Nenhuma barreira DB.
- Ata adulterada = **nulidade jurídica** da assembleia (Art. 1.354 CC + Lei 4.591/64). Síndico pode perder mandato; condomínio fica sem decisão válida; contratos gerados pela assembleia são anuláveis.
- Timeline adulterada = **fraude processual** — síndico pode apagar atividade mal-registrada; conselho fiscal não consegue auditar.
- Pesquisa big-tech: AWS CloudTrail, GCP Cloud Audit Logs e sistemas fiscais BR (SPED) usam **append-only enforced by DB** + log file integrity (hash-chain) como camada mínima.

Enforcement app-level sem DB constraint é o **padrão de falha #1** identificado na reanálise research (junto com Q-002 cupom, Q-003 quota, Q-030 search).

## Decisão

**Elevar INV-024 e INV-072 a enforcement DB-level obrigatório**, em duas camadas:

### Camada 1 — REVOKE de DML destrutivo no role da aplicação

```sql
-- role usado pelo backend Go em runtime
REVOKE UPDATE, DELETE, TRUNCATE ON
  institutional.timeline_entries,
  assembly.assembly_minutes,
  assembly.assembly_votes,
  cross_domain.audit_log_entries
FROM app_user;

-- INSERT permanece (append-only)
GRANT INSERT ON ... TO app_user;
```

`app_user` é o role usado pelo pool `pgx`. Mesmo que um handler bugue e tente UPDATE, Postgres retorna `ERROR: permission denied`. Backend Go trata como 500 e alerta via OTel.

### Camada 2 — Trigger `BEFORE UPDATE OR DELETE` fail-stop

```sql
CREATE OR REPLACE FUNCTION institutional.forbid_mutation()
RETURNS trigger LANGUAGE plpgsql AS $$
BEGIN
  RAISE EXCEPTION 'append_only_violation: %.% is immutable post-publish',
    TG_TABLE_SCHEMA, TG_TABLE_NAME
    USING ERRCODE = 'P0001', HINT = 'INV-024/072/090';
END $$;

CREATE TRIGGER timeline_entries_immutable
  BEFORE UPDATE OR DELETE ON institutional.timeline_entries
  FOR EACH ROW WHEN (OLD.published_at IS NOT NULL)
  EXECUTE FUNCTION institutional.forbid_mutation();

CREATE TRIGGER assembly_minutes_immutable
  BEFORE UPDATE OR DELETE ON assembly.assembly_minutes
  FOR EACH ROW WHEN (OLD.status IN ('published', 'homologated'))
  EXECUTE FUNCTION institutional.forbid_mutation();

CREATE TRIGGER audit_entries_immutable
  BEFORE UPDATE OR DELETE ON cross_domain.audit_log_entries
  FOR EACH ROW EXECUTE FUNCTION institutional.forbid_mutation();
```

Trigger permanece mesmo se algum dia app_user ganhar UPDATE acidental (ex: superuser esqueceu). Defense-in-depth.

### Camada 3 — Admin bypass controlado via role separado

Admin DML sobre atas/timeline só via função stored procedure `assembly.admin_timeline_correction(...)` que exige:
- Role `ms_admin_role` (separado de `app_user`).
- Chamada wraping em `BEGIN; SELECT assembly.admin_timeline_correction(...); COMMIT;` deixando row em `audit_log_entries` com `kind='admin_timeline_override'` + `prev_hash`.
- Runbook 4-eyes (D-054 extendido): admin abre ticket, outro admin aprova no stored proc via `approved_by`.
- Correções **sempre virão como TimelineEntry NOVA** (não editar antiga) — stored proc só permite override em casos extremos documentados (ex: vazamento de CPF em timeline, LGPD Art.18 VI).

### Camada 4 — Hash-chain (referência futura)

`audit_log_entries` ganha `prev_hash TEXT` + trigger calcula `row_hash = sha256(prev_hash || payload_canonicalized)`. CLI offline `audit-verify` re-valida cadeia. **Plano M2** — cobre G-006 (beyond-corp R3).

## Alternativas consideradas

1. **Só trigger, sem REVOKE** — rejeitada: trigger pode ser disable-ado via `SET session_replication_role = replica` por quem tem permissão. REVOKE é barreira mais dura.
2. **Só REVOKE, sem trigger** — rejeitada: superuser (para migrations) bypassa REVOKE. Trigger pega esse caso.
3. **Enforcement só app-level com defensive code** — rejeitada: é exatamente o estado atual que o Q-020 identificou como falha. App bugga; DB é a última barreira.
4. **Tabela event-sourced separada (só INSERT nunca UPDATE)** — considerada mas excessiva pra MVP. Append-only tradicional com REVOKE + trigger cobre 95% dos casos.

## Consequências

### Positivas

- Ata falsificada exige comprometer role `ms_admin_role` + bypass 4-eyes + trigger + audit trail. Barra drasticamente elevada.
- Fraude processual fica provável detectável (audit entry com `admin_timeline_override` é anomalia óbvia).
- Compliance Lei 4.591/64 + Art. 1.354 CC reforçado: ata juridicamente válida.
- Reforça BeyondCorp zero-trust: "don't trust the app, enforce at data layer".

### Negativas

- Correções de ata legítimas (typo, nome incorreto) passam a exigir stored proc + 4-eyes. **Aceitável** — ata publicada sem correção é exatamente o que a Lei exige; typos viram adendo (TimelineEntry nova).
- Migrations que precisam fazer UPDATE em tabelas append-only (backfill, schema change) exigem `DROP TRIGGER` + restaurar depois. Runbook `06-execution/playbooks/schema-migration-append-only.md` documenta.
- `pg_dump` + restore em ambiente dev herda grant/trigger; precisa `--no-privileges` em dev local para remover.

## Implementação

- **Sprint 10 (pré-M1)**: criar migration `20260424000000_ata_timeline_immutability_db_level.up.sql` com REVOKE + trigger + function. Adicionar seed do `ms_admin_role` + stored proc override. `.down.sql` reverte.
- **Backend Go**: ajustar handlers pra tratar `pq.ErrorCode == 'P0001'` como 500 + alert (nunca deveria ocorrer em happy path).
- **Runbook**: `06-execution/playbooks/ata-override-4eyes.md` — stored proc + ticket + 2-admin approval.
- **Teste**: property-based `∀ minutes.status IN {published, homologated} → UPDATE/DELETE falha`.

## Referências

- Q-020 — [[../../11-audit/audit-log/2026-04-23-quebras-negocio-systematic#Q-020]]
- INV-024, INV-072, INV-090 — [[../../01-domain/invariants]]
- Novos: INV-119 (ata DB-level), INV-120 (REVOKE DML)
- D-088 — STATE
- Lei 4.591/64 Art. 24 + Código Civil Art. 1.354
- AWS CloudTrail Log File Integrity Validation (modelo de hash-chain)
- D-054 (4-eyes policy extended para admin DML overrides)

## Vizinhos

- [[0030-saga-orchestration-default]]
- [[0026-admin-mfa-m1]]
- [[0028-lgpd-keyed-hmac]]
- [[../../01-domain/invariants]]
- [[../../01-domain/business-rules]]
