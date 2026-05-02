---
title: ADR-0014 — 1-device policy via device_fingerprint
type: adr
tags: [adr, security, device, beyondcorp, master-sindico, v2]
status: accepted
date: 2026-04-23
created: 2026-04-23
updated: 2026-04-23
---

# ADR-0014 — 1-device policy via device_fingerprint

## Status

accepted — 2026-04-23 (herdado de legado A-023)

## Contexto

Master Síndico é plataforma para governança — fraude de assembleia (votar de múltiplos devices, conta compartilhada entre síndicos) destrói legitimidade. BeyondCorp Device Inventory ([[../../13-research/beyond-corp/_destilado]] §1.3) define identidade de dispositivo como cidadão de primeira classe.

## Decisão

**1 dispositivo ativo por conta**: login novo invalida sessão anterior. Implementação via `device_fingerprint` e tabela `devices`.

### Device fingerprint (hash)

Composto client-side:
- **IP público** (via header `X-Forwarded-For` honrando proxy chain + BeyondCorp).
- **User-Agent** normalizado (browser/OS stripado de versões patch).
- **Canvas hash** (web) / **Play Integrity / DeviceCheck attestation** (mobile M2+).
- **Timezone + accept-language** como complemento.

Final fingerprint = SHA-256 dos campos concatenados em ordem canônica. TTL = 30 dias antes de recomputar.

### Tabela `devices`

```sql
CREATE TABLE identity.devices (
    tenant_id          BYTEA,
    id                 BYTEA NOT NULL,
    user_id            BYTEA NOT NULL,
    fingerprint        TEXT NOT NULL,
    trust_tier         TEXT NOT NULL DEFAULT 'new',   -- new / confirmed / suspicious / banned
    platform           TEXT NOT NULL,                 -- web / ios / android
    first_seen_at      TIMESTAMPTZ NOT NULL DEFAULT now(),
    last_seen_at       TIMESTAMPTZ NOT NULL DEFAULT now(),
    last_ip            INET,
    active             BOOLEAN NOT NULL DEFAULT true,
    revoked_at         TIMESTAMPTZ,
    revoked_reason     TEXT,
    PRIMARY KEY (user_id, id)
);
CREATE INDEX ON identity.devices (user_id, active) WHERE active;
```

### Fluxo

1. Login (OIDC callback) → API calcula fingerprint → busca em `devices`:
   - **Match existente active** → renova `last_seen_at`.
   - **Novo fingerprint** → cria device tier `new` + **invalida todos os outros devices ativos** do mesmo `user_id` + publica `DeviceTrustChanged` audit.
2. Subsequent requests → middleware `DeviceFingerprint` valida fingerprint bate com device ativo.
3. Fingerprint mismatch → 401 + audit A-### candidato.

### Audit log

- Troca de device: `timeline_entries` do user (tipo `device_switched`) + `lgpd_logs` (WORM).
- Alertar user por email/push "novo dispositivo detectado em seu login".

## Consequências

### Positivas

- Bloqueia compartilhamento de conta trivial.
- Reduz fraude de voto em assembleia (1 user, 1 device, 1 voto por unidade).
- Audit trail forense em contestações.

### Negativas

- UX: user troca laptop → sessão mobile caiu (trade-off aceito).
- Fingerprint tem falsos positivos (VPN muda IP, trocar browser); tier `confirmed` mitiga após MFA step-up.
- LGPD: fingerprint é dado pessoal — consent explícito + finalidade legítima (prevenir fraude) documentados em DPA.

## Mitigações UX

- UI mostra "dispositivos ativos" em `/me/devices`; user pode revogar manualmente.
- Step-up MFA quando fingerprint muda (mesmo que user reconheça).
- Tier `confirmed` permite "lembrar dispositivo" por 30d.

## Alternativas consideradas

1. **Multi-device livre (nenhuma restrição)** — abre festa de fraude de voto.
2. **2 devices (mobile + web)** — comum em banking; considerado mas não simples em fingerprint (é mesma conta em 2 browsers).
3. **Device binding via cert** — overkill para master-sindico; considerar enterprise tier M3+.

## Referências

- [[../../13-research/beyond-corp/_destilado]] §1.3 + §3.1 + §3.7
- [[0016-beyondcorp-security-posture]]
- Histórico legado: A-023 (audit log troca de device).
