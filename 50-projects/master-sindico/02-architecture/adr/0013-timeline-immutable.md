---
title: ADR-0013 — Timeline imutável (INSERT-only, sem soft-delete)
type: adr
tags: [adr, institutional, timeline, immutable, master-sindico, v2]
status: accepted
date: 2026-04-23
created: 2026-04-23
updated: 2026-04-23
---

# ADR-0013 — Timeline imutável

## Status

accepted — 2026-04-23

## Contexto

Governança condominial exige **memória institucional inquestionável** para prova em MP, contestação de assembleia, disputa judicial, continuidade entre gestões. ERPs concorrentes (Superlógica, Igora) armazenam eventos em silos financeiros mutáveis. Stories ephemeral do Instagram ([[../../13-research/instagram/_destilado]] §2) vai **contra** o valor core.

## Decisão

`timeline_entries` é **append-only**: sem UPDATE, sem DELETE, sem coluna `deleted_at`. Correção vira nova entry com `correction_of: <original_id>`.

### Schema

```sql
CREATE TABLE institutional.timeline_entries (
    tenant_id       BYTEA NOT NULL,
    id              BYTEA NOT NULL,
    condominium_id  BYTEA NOT NULL,
    event_type      TEXT NOT NULL,
    actor_id        BYTEA NOT NULL,
    actor_role      TEXT NOT NULL,
    payload         JSONB NOT NULL,
    correction_of   BYTEA,                -- se entry corrige outra, aponta
    occurred_at     TIMESTAMPTZ NOT NULL,
    created_at      TIMESTAMPTZ NOT NULL DEFAULT now(),
    hash            BYTEA NOT NULL,        -- SHA-256 do payload canonicalizado
    prev_hash       BYTEA,                 -- hash da entry anterior (hash-chain)
    PRIMARY KEY (tenant_id, id),
    FOREIGN KEY (tenant_id, correction_of) REFERENCES institutional.timeline_entries(tenant_id, id)
);

CREATE INDEX ON institutional.timeline_entries (tenant_id, condominium_id, occurred_at DESC);
CREATE INDEX ON institutional.timeline_entries (tenant_id, event_type);

-- trigger bloqueia UPDATE/DELETE
CREATE RULE no_update_timeline AS ON UPDATE TO institutional.timeline_entries DO INSTEAD NOTHING;
CREATE RULE no_delete_timeline AS ON DELETE TO institutional.timeline_entries DO INSTEAD NOTHING;
```

### Hash chain (rolling)

Cada entry carrega hash SHA-256 do payload canonicalizado + hash da entry anterior do mesmo condomínio. Rompimento do chain = forense tampering alert.

### M2+ WORM

- Dual-write: S3 Object Lock ou QLDB equivalente para audit trail permanente.
- Alinha com BeyondCorp §3.4.

## Consequências

### Positivas

- Prova documental inquestionável em disputa.
- Diferencial competitivo (ERPs não fazem).
- Debug e audit trivial (basta ler timeline).

### Negativas

- Correção precisa workflow explícito ("publicar correção" = nova entry linkada).
- Armazenamento cresce monotonicamente — necessário particionamento por `occurred_at` em M3+ quando tabela passa de ~50M rows.
- LGPD direito ao esquecimento vs imutabilidade — resolução: dados pessoais em payload JSONB são pseudonimizados quando usuário requisita exclusão (mantém hash + estrutura sem PII).

## LGPD

- Request exclusão LGPD → novo evento `user.data_anonymized` na timeline + substituição de PII no payload por `[REDACTED-<req_id>]` + preservação de hash original em `audit_lgpd_logs`.
- SLA 15 dias.
- Mais detalhes: `08-security/lgpd.md` (a escrever).

## Alternativas consideradas

1. **Soft-delete com `deleted_at`** — permite "apagar depois", destrói prova documental.
2. **Log append-only separado fora do DB** — Kafka tópico compactado; overhead; não queryável por BC.
3. **CRDTs** — overkill; dados condominais não são multi-master.

## Referências

- [[../../STEERING]] §41
- [[../../13-research/instagram/_destilado]] §2 (rejeitar ephemeral)
- [[../../13-research/beyond-corp/_destilado]] §3.4 (audit imutável)
- [[../../13-research/google-arch/_destilado]] §1 (SRE — dados)
