---
title: ADR-0037 — soft_delete universal + pseudonimização HMAC-keyed + retenção 5y
type: adr
tags:
  - adr
  - lgpd
  - soft-delete
  - pseudonymization
  - retention
  - privacy
  - master-sindico
  - v2
  - fase-11
status: accepted
date: '2026-04-24'
created: '2026-04-24T00:00:00.000Z'
updated: '2026-04-24T00:00:00.000Z'
aliases:
  - soft-delete-universal-pseudonymize
  - lgpd-soft-delete
---

# ADR-0037 — soft_delete universal + pseudonimização HMAC-keyed + retenção 5y

## Status

accepted — 2026-04-24 (Fase 11, derivada de [[../../STATE#d-101-—-lgpd-soft-delete-universal-pseudonimização-hmac-keyed-fecha-a-dc-sec-004-lgpd-m1-007|D-101]])

## Contexto

Finding [[../../AUDIT#bucket-a-—-segurança-dual-check-sec|A-DC-SEC-004]] (🔴 M1 bloqueante) descreveu race condition entre hard-delete LGPD e webhooks Stripe: user emite `DELETE /me` → sistema executa `DELETE FROM users WHERE id=X` → 200ms depois chega webhook `invoice.payment_succeeded` tentando `UPDATE users SET stripe_customer_id=... WHERE id=X` em row inexistente. Resultado: dados órfãos em tabelas transacionais + inconsistência Stripe + audit trail incompleto (violação LGPD Art. 46 de audit de dados pessoais).

A mitigação proposta inicial em AUDIT foi "soft-flag `_pending_hard_delete` com janela 48h + webhook valida flag + hard-delete job async" (cobertura parcial implementada em [[0028-lgpd-keyed-hmac#Workflow atualizado de hard-delete]]). Essa abordagem resolve a race mas deixa **3 problemas abertos**:

1. **Janela de 48h é insuficiente para webhook tardio Stripe**. Em casos de disputa/refund, Stripe pode mandar webhooks semanas depois.
2. **Retenção LGPD 5 anos** (obrigação Art. 15 + Art. 16 + normativa fiscal federal) não é atendida se hard-delete acontece após 48h.
3. **Audit trail fica órfão**: `audit_log_entries` referenciam `user_id` — se user é hard-deleted em 48h, a FK quebra ou vira NULL, destruindo a cadeia de evidências.

LGPD-M1-007 (bloqueador M1 em [[../../AUDIT#🔴 Bloqueadores LGPD M1 — MANTIDOS ABERTOS]]) dependia de resolução arquitetural dessa questão — sem ela, M1 não pode entrar em produção.

## Decisão

**soft_delete universal em todas as tabelas com dados pessoais ou transacionais do usuário** + **pseudonimização via HMAC-keyed (reusa mecanismo de [[0028-lgpd-keyed-hmac|ADR-0028]])** + **retenção 5 anos + hard-delete noturno depois**.

### 1. Coluna `deleted_at` em todas as tabelas

```sql
ALTER TABLE <tabela>
  ADD COLUMN deleted_at TIMESTAMPTZ NULL;

CREATE INDEX CONCURRENTLY idx_<tabela>_deleted_at
  ON <tabela> (deleted_at)
  WHERE deleted_at IS NOT NULL;
```

Tabelas afetadas (~34 no backend, escopo M1+M2):

- **Identity BC**: `users`, `memberships`, `user_consents`, `user_devices`, `sessions`.
- **Billing BC**: `subscriptions`, `invoices`, `feature_usages`, `coupons_usage`.
- **Institutional BC**: `condominiums`, `units`, `timeline_entries`, `administrators`, `compliance_declarations`.
- **Commercial BC**: `connect_me_requests`, `company_evaluations`, `supplier_vote_sessions`, `deals`, `seals`.
- **Content BC**: `videos`, `courses`, `enrollments`, `certificates`, `library_items`, `forum_posts`.
- **Assembly BC**: `assemblies`, `votes`, `proxies`, `minutes`.
- **Apenas `audit_log_entries` NÃO tem soft_delete** — é append-only; a remoção final só acontece via hard-delete pós-retenção 5y.

### 2. Semântica de operações

- **`DELETE /me`** (LGPD Art. 18 VI):
  1. `UPDATE users SET deleted_at = NOW()` (não `DELETE`).
  2. Pseudonimizar campos pessoais (ver §3 abaixo).
  3. Revogar sessões ativas + devices.
  4. Cancel Stripe subscription via API (best-effort, idempotente).
  5. Emit `user.deleted` event → cascateia soft-delete dependências (memberships, user_devices).
  6. `lgpd_logs`: entry `'deletion_requested'` com timestamp.
- **Qualquer query de produto** (busca, feed, autenticação):
  - Default scope `WHERE deleted_at IS NULL` via query builder ou scope global. Enforcement via linter AST: qualquer `SELECT * FROM users` sem `deleted_at IS NULL` falha PR.
  - ABAC engine: `user.deleted_at IS NOT NULL` = equivalente a `user.role = banned` — 403 em todo endpoint de negócio; só endpoints de suporte LGPD (export, confirmação) respondem.
- **Webhook Stripe pós-soft-delete**:
  - Webhook chega (pode ser horas, dias, anos depois) → `UPDATE subscriptions SET status=... WHERE stripe_customer_id=X` funciona normalmente (row existe; `deleted_at` não bloqueia UPDATE transacional).
  - **Race resolvida por construção** — não precisa soft-flag `_pending_hard_delete` nem janela 48h.
- **Reativação**: usuário que muda de ideia dentro de 30d pode reativar (`UPDATE users SET deleted_at = NULL` + restore sessões). Depois de 30d, pseudonimização já rodou → reativação não é mais possível (dados pessoais irreversíveis).

### 3. Pseudonimização (reusa [[0028-lgpd-keyed-hmac|ADR-0028]])

No momento do soft_delete (Day 0), campos pessoais são substituídos por hash HMAC-keyed:

```go
// pkg/lgpd/soft_delete.go

func SoftDeleteUser(ctx context.Context, userID uuid.UUID) error {
    return db.Transaction(ctx, func(tx *sql.Tx) error {
        anonName  := lgpd.Pseudonymize(ctx, userID, "user_name",  time.Now())
        anonEmail := lgpd.Pseudonymize(ctx, userID, "user_email", time.Now())
        anonCPF   := lgpd.Pseudonymize(ctx, userID, "user_cpf",   time.Now())
        anonPhone := lgpd.Pseudonymize(ctx, userID, "user_phone", time.Now())

        _, err := tx.ExecContext(ctx, `
            UPDATE users SET
                deleted_at = NOW(),
                name       = $2,
                email      = $3,
                cpf_hash   = $4,
                phone      = $5,
                avatar_url = NULL,
                bio        = NULL
            WHERE id = $1 AND deleted_at IS NULL
        `, userID, anonName, anonEmail, anonCPF, anonPhone)
        return err
    })
}
```

- **Campos transacionais permanecem intactos** (`stripe_customer_id`, `zitadel_sub`, `role`, `created_at`) — necessários para reconciliação Stripe + audit trail.
- **Campos pessoais** (`name`, `email`, `cpf`, `phone`, `avatar_url`, `bio`) viram pseudônimos irreversíveis após year rotation (5y retention — ver ADR-0028).
- **Reativação 30d**: os valores originais ficam em uma tabela de backup criptografada (`users_pii_backup`) com `expires_at = deleted_at + 30d`; depois disso, backup é hard-deleted e pseudonimização é permanente.

### 4. Retenção 5 anos + hard-delete noturno

```sql
-- cron diário 03:00 UTC: hard-delete rows com deleted_at > 5 anos
WITH expired AS (
    SELECT id FROM users
    WHERE deleted_at IS NOT NULL
      AND deleted_at < NOW() - INTERVAL '5 years'
    LIMIT 1000
)
DELETE FROM users WHERE id IN (SELECT id FROM expired);

-- Repete por tabela, cascateando via FK ON DELETE CASCADE.
```

- Cron job orquestrado via Railway cron ou NATS scheduler (ver [[../../AUDIT#🟡-a-fase10-dev-007-—-crons-noturnos-referenciados-sem-scheduler|A-FASE10-DEV-007]]).
- Limite 1000 rows/exec para não travar DB; job roda diariamente, convergindo natural.
- Métrica `lgpd_hard_deletes_total{table}` exposta em Prometheus; alerta se 0 por 7+ dias (indicativo de cron quebrado) OU > 10k por dia (indicativo de bug).

### 5. Invariante + linting

- **Linter AST Go** (`pkg/lint/soft_delete.go`): qualquer query que faça `SELECT ... FROM <tabela com deleted_at>` **sem** `WHERE deleted_at IS NULL` → PR bloqueado. Bypass via comment `// +lint:allow-soft-deleted` (raríssimo — justificar no PR).
- **Invariante** [[../../01-domain/invariants|INV-soft-delete-universal]] (a criar): "Nenhuma tabela com dados pessoais sofre `DELETE` físico direto em runtime; apenas `UPDATE ... SET deleted_at = NOW()`. Hard-delete é exclusivamente executado pelo cron noturno `lgpd_retention_cleanup` após retenção 5 anos."
- **Teste de regressão**: PBT (property-based) em `09-testing/`: para qualquer sequência de ações do usuário, se user emite `DELETE /me`, em nenhum ponto a row é hard-deleted antes de 5 anos.

## Consequências

### Positivas

- Fecha A-DC-SEC-004 + LGPD-M1-007 🟢 (destrava 2 bloqueadores M1).
- Race condition Stripe resolvida **por construção** (não por timing frágil).
- LGPD Art. 15/16/18 VI + obrigação fiscal de retenção 5y atendidos.
- Audit trail preservado: `audit_log_entries.user_id` permanece apontando para row real; FK inteligível mesmo após soft_delete.
- Reativação 30d oferece UX humana (erro de clique em conta crítica).
- Uniform schema-wide é fácil de auditar/lintar/testar.

### Negativas

- Overhead storage: rows soft-deleted ocupam disco por 5 anos. Estimativa para M1 (10k users): ~50 MB extras/ano — desprezível vs disk costs.
- Todos queries precisam `deleted_at IS NULL` — adiciona cost de manutenção. Mitigado via query builder scope + linter.
- Índices parciais `WHERE deleted_at IS NOT NULL` adicionam overhead em UPDATE (mas insignificante: cardinalidade é <1% dos rows).
- Reativação 30d exige tabela separada `users_pii_backup` — complexidade extra.

## Alternativas consideradas

1. **Hard-delete imediato com 2-phase-commit entre sistema interno + Stripe** — rejeitado. 2PC não existe entre sistemas independentes; best-effort cascade é frágil e Stripe API não suporta semântica transacional.

2. **Hard-delete com job cascade** (delete all FK children em ordem) — rejeitado. Violaria retenção LGPD 5y. Audit trail incompleto (Art. 46 exige 5y).

3. **Soft-flag `_pending_hard_delete` 48h intermediário + hard-delete depois** (mitigação original em AUDIT) — rejeitado. Resolve a race imediata mas:
   - Não atende retenção 5y (fere Art. 15 + normativa fiscal).
   - Webhooks tardios Stripe (disputas, refunds semanas depois) ainda podem furar a janela.
   - Overhead de 2 mecanismos paralelos (soft-flag + job) para resolver problema que soft_delete universal resolve limpo.

4. **Tombstone separado** (row-delete + INSERT em `tombstone_users`) — rejeitado. Duplica FK semantics (qual tabela é source of truth?). Audit trail perde linking natural. Complexidade de migrations cascade.

5. **Encryption-at-rest + key-destroy** (chave de user é destruída após 30d = "cryptographic erasure") — rejeitado como **única** técnica. Ainda precisaria de soft_delete para webhooks Stripe funcionarem. Reservado como **enhancement futuro** (M3+) para reduzir retention storage quando datasets crescerem.

6. **GDPR-style "right to be forgotten" com hard-delete imediato** — rejeitado. LGPD brasileira tem retenção 5y obrigatória que supera "esquecimento imediato"; GDPR tem exceções análogas.

## Regras

1. **Todo novo aggregate que persiste dados pessoais** declara `deleted_at TIMESTAMPTZ NULL` desde a migration inicial.
2. **Zero chamadas `DELETE FROM ...`** em código de runtime (fora cron `lgpd_retention_cleanup`). CI linter enforça.
3. **`SoftDeleteX()` functions em domain/use_case layer** são o único caminho para remoção — não expõem `DELETE` direto.
4. **Pseudonimização é síncrona** no mesmo UoW do `UPDATE deleted_at` — nunca async (race anterior fica ativa).
5. **Reativação 30d** documentada em runbook [[../../12-docs/runbooks/_moc|runbooks (lgpd-user-deletion-recovery a criar)]] (a criar).
6. **Retenção 5y hard-codada**: `RETENTION_YEARS = 5` constante; mudança exige ADR novo.
7. **Métrica Prometheus obrigatória**: `lgpd_soft_deletes_total{table}`, `lgpd_hard_deletes_total{table}`, `lgpd_reactivations_total`.

## Referências

- Finding: [[../../AUDIT#bucket-a-—-segurança-dual-check-sec|A-DC-SEC-004]] (🔴 M1 bloqueante — fechada via este ADR).
- Finding: [[../../AUDIT#🔴 Bloqueadores LGPD M1 — MANTIDOS ABERTOS|LGPD-M1-007]] (🔴 M1 bloqueante — fechada via este ADR).
- Decisão pai: [[../../STATE#d-101-—-lgpd-soft-delete-universal-pseudonimização-hmac-keyed-fecha-a-dc-sec-004-lgpd-m1-007|STATE D-101]].
- ADR relacionada: [[0028-lgpd-keyed-hmac]] — mecanismo de pseudonimização (reusado aqui).
- ADR relacionada: [[0026-admin-mfa-m1]] — `DELETE /me` exige step-up MFA que dispara soft_delete workflow.
- [[../../08-security/lgpd]] — atualiza política de deleção.
- [[../../04-requirements/compliance-lgpd]] — atualiza requisitos LGPD.
- [[../../12-docs/runbooks/secret-rotation]] — rotation da key HMAC usada na pseudonimização.
- LGPD Lei 13.709/18 Art. 12, Art. 15, Art. 16, Art. 18 VI, Art. 46: https://www.planalto.gov.br/ccivil_03/_ato2015-2018/2018/lei/l13709.htm
- ANPD Guia de Anonimização (2023): https://www.gov.br/anpd/pt-br/documentos-e-publicacoes/guia-anonimizacao
- NIST SP 800-188 (De-Identification) §3.3: https://nvlpubs.nist.gov/nistpubs/SpecialPublications/NIST.SP.800-188.pdf
