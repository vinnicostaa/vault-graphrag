---
title: ADR-0028 — Pseudonimização LGPD via HMAC keyed com secret rotating
type: adr
tags: [adr, lgpd, pseudonymization, hmac, privacy, master-sindico, v2]
status: accepted
date: 2026-04-23
created: 2026-04-23
updated: 2026-04-23
aliases: [lgpd-keyed-hmac, pseudonymization-keyed-hmac]
---

# ADR-0028 — Pseudonimização LGPD via HMAC keyed com secret rotating

## Status

accepted — 2026-04-23

## Contexto

Finding [[../../11-audit/audit-log/2026-04-23-pentest-specs#A-DC-PEN-038]] (🔴 M1 bloqueante) identificou que a spec atual de [[../../04-requirements/functional/identity#IDN-022]] + [[../../08-security/lgpd#4.5 Workflow de exclusão]] descreve pseudonimização pós-hard-delete como `anon_<hash>` — implícita ou explicitamente **hash determinístico** do tipo `SHA256(user_id)`.

**Problema**: hash determinístico é **reversível** por força bruta + cross-reference:

- UUID v7 tem espaço grande mas previsível (timestamp prefix), tabelas pequenas (e.g. condomínio com 100 moradores) → **rainbow table trivial**.
- Com dados correlatos publicamente acessíveis no timeline (unit_id, role, horários de voto) + `anon_<hash>` consistente entre entries, atacante **re-identifica** comparando padrões.
- **LGPD Art. 12 §1º**: pseudonimização só vale se **razoavelmente** impede re-identificação. Hash determinístico **não impede**.
- ANPD Guia de Anonimização (2023) explicitamente recomenda **keyed** + **rotation** para casos em que pseudonimização precisa resistir a cross-reference attack.

Ausência do controle = 🔴 **violação direta LGPD Art. 18 V + Art. 12** — e o workflow SLA 15d de IDN-022 entra em produção M1 como exigência regulatória.

## Decisão

**Pseudonimização LGPD passa a usar HMAC keyed com secret rotating** (não mais SHA256 determinístico).

### Algoritmo

```go
// pkg/lgpd/pseudonymize.go

// Pseudonymize gera um identificador pseudônimo não-determinístico para user_id
// dentro do contexto `purpose` (ex: "timeline_anon", "vote_anon", "audit_anon").
// A secret_key rotaciona anualmente — uma vez rotacionada, pseudônimos antigos
// NÃO podem ser re-calculados (irreversibilidade forward).
func Pseudonymize(ctx context.Context, userID uuid.UUID, purpose string, when time.Time) string {
    key := secretStore.GetKey(when.Year())  // key_2026, key_2027, ...
    mac := hmac.New(sha256.New, key)
    mac.Write([]byte(purpose))
    mac.Write([]byte{0})                    // separator
    mac.Write(userID[:])
    sum := mac.Sum(nil)
    return "anon_" + base32.StdEncoding.WithPadding(base32.NoPadding).EncodeToString(sum[:15])
}
```

### Key lifecycle

```sql
CREATE TABLE lgpd_pseudonym_keys (
    year            INT PRIMARY KEY,              -- 2026, 2027, ...
    key_hash        BYTEA NOT NULL,               -- key encrypted via envelope (KMS)
    created_at      TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    retired_at      TIMESTAMPTZ,                  -- set when year expires
    retention_ends  TIMESTAMPTZ NOT NULL          -- key destroyed after retention (5y)
);

-- Nenhuma key é deletada antes de 5y após aposentada (para reconciliação LGPD audit).
-- Após retention_ends: hard-delete da row (dados pseudonimizados ficam irreversíveis para sempre).
```

### Regras de uso

1. **`user_id` original NUNCA é escrito** em `timeline_entries.actor_id`, `votes.voter_id`, etc. após hard-delete — só o pseudônimo.
2. **Purpose scope**: cada contexto (`timeline_anon`, `vote_anon`, `audit_anon`, `contract_anon`) tem purpose distinto → mesmo user_id gera pseudônimos diferentes por contexto. Impede correlação cross-context.
3. **Year scope**: pseudônimos gerados em 2026 usam `key_2026`. Em 2027, user excluído nesse ano usa `key_2027` — pseudônimos de anos diferentes são incorrelatáveis mesmo para o mesmo user.
4. **Key storage**: `secret_key_N` em AWS Secrets Manager (M2+, ver Gap-SEC-015) ou Railway env var criptografada (M1). Acesso read-only via IAM role específico; audit log obrigatório.
5. **Rotação automática**: cron 1º janeiro gera `key_Y+1`; `key_Y` é marcada `retired_at` mas mantida 5 anos para reconciliação audit.
6. **Destruição**: após `retention_ends`, key é hard-deleted da KMS/Secrets Manager. Daí em diante, pseudônimos daquele ano são **matematicamente irreversíveis**.

### Workflow atualizado de hard-delete (substitui [[../../08-security/lgpd#4.5]])

```
Day 0: User requests DELETE /me
  → marca users.deletion_scheduled_at = NOW() + 15d
  → marca users._pending_hard_delete = true (soft-flag — ver ADR-0031 ou A-DC-SEC-004)
  → lgpd_logs: 'deletion_requested'
  → pre-cancela Stripe subscription (API); aguarda `customer.subscription.deleted` webhook

Day 0-14: Grace period
  → webhooks Stripe validam `_pending_hard_delete` antes de UPDATE; se true + customer deleted → skip + log

Day 15: Execution async worker
  → verify soft-flag _pending_hard_delete AINDA setado
  → verify nenhum webhook Stripe em-trânsito (`webhook_events` status=pending com user match) → else espera 48h
  → PSEUDONIMIZE ANTES DE HARD-DELETE:
      anon_id := Pseudonymize(user_id, "timeline_anon", NOW())
      UPDATE timeline_entries SET actor_id = NULL, actor_pseudonym = anon_id WHERE actor_id = user_id
      anon_vote := Pseudonymize(user_id, "vote_anon", NOW())
      UPDATE votes SET voter_id = NULL, voter_pseudonym = anon_vote WHERE voter_id = user_id
      -- repete por purpose
  → HARD DELETE users + sessions + device_fingerprints + user_consents
  → lgpd_logs: 'deletion_executed' com user_id_hash = Pseudonymize(user_id, "audit_anon", NOW())
```

## Consequências

### Positivas

- Fecha A-DC-PEN-038 (pseudonimização re-identificável).
- Compliance LGPD Art. 12 + Art. 18 V verificável por auditor externo.
- Purpose separation previne correlação cross-context.
- Year rotation + retention_ends garantem irreversibilidade forward após 5 anos (pré-requisito para passar em DPIA).
- Preserva utilidade legal (Lei 4.591/64 exige ata + contagem de votos mantidos).

### Negativas

- Gestão de keys é infra-crítica: perda da key = dados pseudonimizados do ano ficam irreversíveis **antes** do prazo de 5 anos (bom para privacy, ruim se precisar reconciliar em disputa judicial).
- Cada novo aggregate que precisa pseudonimização precisa declarar `purpose` (dev discipline).
- Key rotation job precisa runbook + alerta (ver [[../../08-security/lgpd#14.1 M1]]).

## Regras

1. **Nenhum uso de `SHA256(user_id)` ou similar determinístico** para pseudonimização. CI grep pattern `sha256.*user_id` em código de anonimização → PR bloqueado.
2. **Toda chamada a `Pseudonymize()`** deve vir com `purpose` explícito + timestamp (`now()` ou `entry.created_at`).
3. **Keys nunca em código, config, logs** — só KMS / Secrets Manager + acesso via IAM.
4. **Audit trail** ([[../../01-domain/invariants#INV-090]]) registra `lgpd.pseudonym.key.accessed` em toda chamada ao secret store.
5. **Retention enforcement**: cron job diário verifica `retention_ends < now()` → dispara hard-delete da key.

## Alternativas consideradas

1. **SHA256 determinístico** (spec atual) — rejeitado: LGPD non-compliance.
2. **bcrypt/argon2 do user_id** — lento demais para batch (worker processa 10k+ users em hard-delete); bcrypt não é pra este propósito.
3. **Encryption AES-GCM com key rotation** — reversível por design (AES-GCM decrypta); pseudonimização **precisa ser one-way**.
4. **Differential privacy (noise injection)** — overkill; destrói utilidade legal da ata (contagem de votos ±noise não serve em disputa judicial).
5. **HMAC com key fixa (sem rotation)** — insuficiente: se key vaza uma vez, todos os pseudônimos históricos viram reversíveis. Rotation anual + retention+destroy é o controle.

## Referências

- Finding: [[../../11-audit/audit-log/2026-04-23-pentest-specs#A-DC-PEN-038]] (🔴 M1 bloqueante).
- [[../../04-requirements/functional/identity#IDN-022]] — requirement atualizado.
- [[../../08-security/lgpd#4.5 Workflow de exclusão]] — workflow atualizado.
- [[../../08-security/lgpd#4.6 Justificativa da pseudonimização vs hard delete (Art. 16)]] — base legal.
- [[../../01-domain/invariants#INV-109]] — novo invariante.
- [[../../11-audit/dual-check-log/2026-04-23-security-gaps#A-DC-SEC-004]] — race Stripe relacionado (resolvido via `_pending_hard_delete` — coordenar ambos).
- [[0026-admin-mfa-m1]] — `DELETE /me` exige step-up que dispara este workflow.
- LGPD Art. 12 §1º: https://www.planalto.gov.br/ccivil_03/_ato2015-2018/2018/lei/l13709.htm#art12
- ANPD Guia Anonimização (2023): https://www.gov.br/anpd/pt-br/documentos-e-publicacoes/guia-anonimizacao
- NIST SP 800-188 (De-Identification) §3.3: https://nvlpubs.nist.gov/nistpubs/SpecialPublications/NIST.SP.800-188.pdf
- RFC 2104 (HMAC): https://www.rfc-editor.org/rfc/rfc2104
