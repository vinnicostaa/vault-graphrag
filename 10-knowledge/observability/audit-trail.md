---
title: Audit Trail — log imutável para compliance e forense
type: concept
tags: [observability, audit, compliance, lgpd, gdpr, immutability]
source: https://owasp.org/www-project-proactive-controls/v3/en/c9-security-logging
source_date: 2026-04-24
created: 2026-04-24
updated: 2026-04-24
---

# Audit Trail — log imutável para compliance e forense

Audit trail é um registro **append-only, tamper-evident** de ações relevantes do sistema — quem fez o quê, quando, em qual recurso, com qual resultado. Não é sinônimo de log: tem requisitos legais, retenção longa e ciclo de vida separado.

## Audit trail ≠ log de aplicação

| Aspecto | Log de aplicação | Audit trail |
|---|---|---|
| **Propósito** | Debug, troubleshooting, performance | Compliance, forense, não-repúdio |
| **Público** | Engenharia | Auditoria, legal, regulador, investigador |
| **Retenção** | 7-180 dias | **Anos** (5+ LGPD, 6+ GDPR, 7+ SOX) |
| **Mutabilidade** | Pode ser truncado/rotacionado | **Imutável** (append-only, WORM) |
| **Schema** | Flexível, evolutivo | **Estável**, versionado |
| **Completude** | Sample ok | **Zero perda** |
| **Acesso** | Amplo (dev, SRE) | Restrito + próprio audit log |

Log de aplicação e audit trail **podem** coexistir no mesmo pipeline (ex: OTel Collector roteia "audit" para bucket WORM e "app" para Loki), mas **nunca** são a mesma coisa.

## Campos obrigatórios

Cada evento de audit trail carrega, no mínimo:

```json
{
  "event_id":     "uuid-v4-global-unique",
  "ts":           "2026-04-24T10:32:11.123Z",    // UTC ISO-8601 c/ ms
  "actor": {
    "type":       "user | service | system",
    "id":         "identificador estável",
    "display":    "nome humano (opcional, p/ UI)"
  },
  "action":       "resource.create | resource.update | ...",
  "resource": {
    "type":       "invoice | user | payment | ...",
    "id":         "identificador do recurso"
  },
  "outcome":      "success | failure | denied",
  "reason":       "motivo em caso de failure/denied",
  "context": {
    "ip":         "mascarado se PII regional",
    "user_agent": "...",
    "request_id": "correlação com logs/traces",
    "tenant_id":  "isolamento multi-tenant",
    "trace_id":   "W3C trace id"
  },
  "changes": {                                   // opcional, em updates
    "before":   { "campo": "v1" },
    "after":    { "campo": "v2" }
  },
  "schema_version": "1.2.0"
}
```

**Regras**:
- **`ts` sempre UTC** — fuso local corrompe análise cross-region
- **`actor` nunca anônimo** em ação autenticada — se é anônimo, registrar `type=guest` explícito
- **Diff em `changes`** deve mascarar PII (hashear antes, se necessário correlacionar)
- **`schema_version`** permite evoluir sem quebrar consumidores históricos

## Imutabilidade — o que torna o trail confiável

Um trail que pode ser editado **não tem valor legal**. Técnicas para garantir imutabilidade:

### 1. Append-only na tabela (DB tradicional)

```sql
CREATE TABLE audit_events (
    event_id    UUID PRIMARY KEY,
    ts          TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    actor_type  TEXT NOT NULL,
    actor_id    TEXT NOT NULL,
    action      TEXT NOT NULL,
    resource    TEXT NOT NULL,
    outcome     TEXT NOT NULL,
    payload     JSONB NOT NULL,
    prev_hash   TEXT NOT NULL,   -- hash do evento anterior (chaining)
    hash        TEXT NOT NULL    -- SHA-256(event_id || ts || ... || prev_hash)
);

-- proibir UPDATE e DELETE via trigger/permissão
REVOKE UPDATE, DELETE ON audit_events FROM PUBLIC;
CREATE RULE no_update AS ON UPDATE TO audit_events DO INSTEAD NOTHING;
CREATE RULE no_delete AS ON DELETE TO audit_events DO INSTEAD NOTHING;
```

`prev_hash` + `hash` formam uma **chain** (como blockchain simplificada): adulterar evento N invalida hash de N+1 até o fim — detecção trivial.

### 2. Object storage WORM (S3 Object Lock, GCS retention)

- Escrever cada evento como objeto `.jsonl` em bucket com **Object Lock** em modo *Compliance* (ninguém, nem root AWS, pode apagar antes do retention period)
- Ideal para audit logs com retenção multi-ano
- Custo baixo, compliance-grade

### 3. Appender em disco com WAL + fsync

- Log binário append-only (formato `Raft log`, `RocksDB WAL`)
- `fsync()` após cada append pra sobreviver a crash/power-off
- Hash chain para detectar tampering post-hoc

### 4. Ledger dedicado (Merkle tree, blockchain)

- Soluções como AWS QLDB, Google Cloud Spanner change streams, Hyperledger Fabric
- Overkill pra maioria dos casos — reservado pra regulações pesadas (financeiro, votação, cadeia de custódia)

## Retenção legal — tabela resumo

| Regulação | Escopo | Retenção mínima típica |
|---|---|---|
| **LGPD (Brasil)** | Dados pessoais em geral | 5 anos após término da relação (Art. 15/16) |
| **GDPR (EU)** | Logs de processamento de dados pessoais | 6 anos (varia por país; DPIA define) |
| **SOX (EUA, companhias abertas)** | Controles financeiros | 7 anos |
| **HIPAA (EUA, saúde)** | Logs de acesso a PHI | 6 anos |
| **PCI-DSS (cartões)** | Logs de transação | 1 ano online + total ≥ arquivo específico |
| **BACEN / COAF (BR, financeiro)** | KYC/AML | 10 anos (Lei 9.613) |

Arquivar logs antigos em **cold storage** (S3 Glacier, Archive Storage) reduz custo 10×. Lifecycle policy automatizado.

**Atenção**: retenção por si só não protege; é obrigatório que o trail permaneça **íntegro e recuperável** durante todo o período.

## Onde armazenar — tabela decisional

| Storage | Quando |
|---|---|
| **Tabela dedicada no mesmo DB** | Setup inicial, baixo volume, `prev_hash` chain |
| **DB separado (read-replica + append-only write user)** | Médio volume, querying pesado, isolação |
| **S3/GCS Object Lock WORM** | Alto volume, retenção longa, custo ótimo |
| **SIEM (Splunk, Elastic Security, Sumo)** | Integração com detecção de ameaça, investigação |
| **Ledger (QLDB, Hyperledger)** | Regulação estrita exigindo tamper-evident criptográfico |

Comum: **duplo destino** — hot (DB ou SIEM pra query operacional) + cold (S3 WORM pra retenção legal). OTel Collector ou pipeline próprio roteia.

## Patterns — como escrever sem ficar frágil

### Outbox pattern (recomendado)

Escrever o audit junto com a mutação **na mesma transação local**, em tabela `outbox`. Um worker separado publica para o destino final (S3, Kafka, SIEM).

```
BEGIN;
  UPDATE account SET balance = balance - 100 WHERE id = 42;
  INSERT INTO audit_outbox (event) VALUES ('{"action":"withdrawal", ...}');
COMMIT;
-- worker async lê outbox e publica, depois marca como enviado
```

Vantagem: **atomicidade** entre mutação e audit. Se uma falha, as duas falham juntas.

### CQRS + Event Sourcing

Se o projeto já é event-sourced, audit trail **emerge naturalmente** do event store — cada evento do domínio **é** o audit event (com projeção separada pra consulta auditorial).

### Trigger de DB (cuidado)

Triggers `AFTER INSERT/UPDATE/DELETE` podem popular tabela de audit automaticamente — mas dificultam enriquecer com `actor_id`, `request_id` etc., que vivem na aplicação. Usar apenas para **audit técnica de integridade** (quem mexeu em tabela sensível fora do app).

## Antipatterns

- **Audit no mesmo fluxo síncrono da transação primária** sem outbox — pode falhar parcialmente (tx ok, audit não gravou), quebrando a garantia
- **Audit em `INFO` log comum** — vai ser truncado, perdido em rotação, sem schema estável
- **Sem hash chain nem Object Lock** — qualquer admin pode editar e negar ação depois
- **Campos variáveis / sem `schema_version`** — quebrar parsers históricos obriga migrations massivas
- **PII em claro no audit** — audit retido 5 anos viola LGPD se tiver CPF/email em claro; **mascarar ou tokenizar** antes de persistir
- **Actor `null`** em operação autenticada — se foi login de usuário, tem que ter user_id; "system" só quando é automação
- **Sem teste de recuperação** — trail de 5 anos atrás é inútil se nunca foi testado restore
- **Misturar audit com log de debug** — dilui sinal, amplia superficie de PII

## Links

- [[_moc]]
- [[red-use-sli-slo]]
- [[structured-logging]]
- [[../security/_moc]]
- [[../_moc]]

## Fontes

- OWASP — [Security Logging & Monitoring](https://owasp.org/www-project-proactive-controls/v3/en/c9-security-logging)
- LGPD (Lei 13.709/2018) — Art. 15, 16, 37, 46.
- GDPR — Art. 30 (records of processing).
- NIST SP 800-92 — *Guide to Computer Security Log Management*.
- AWS S3 Object Lock — https://docs.aws.amazon.com/AmazonS3/latest/userguide/object-lock.html
- Kleppmann, M. (2017). *Designing Data-Intensive Applications*. O'Reilly — event sourcing, outbox pattern.
