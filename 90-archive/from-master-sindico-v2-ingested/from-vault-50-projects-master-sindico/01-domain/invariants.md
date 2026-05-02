---
title: Invariants — Master Síndico (lista consolidada)
type: project
tags: [master-sindico, domain, invariants, pbt]
project: master-sindico
created: 2026-04-22
---

# Invariants — consolidado testável

Lista única dos invariantes que **nunca** violam. Cada um mapeia pra [[domain-rules]] + tipo de teste que garante.

---

## Core

| # | Invariante | Domain Rule | Teste |
|---|---|---|---|
| I-001 | Soma de FracaoIdeal em Condominium ≤ 100 | DR-001 | PBT (100 casos) |
| I-002 | FracaoIdeal ∈ (0, 100] | DR-002 | Unit + PBT |
| I-003 | 1 session ativa por user | DR-003 | Integration |
| I-004 | Vote único por (agenda_item, voter) | DR-004 | Integration + PBT TOCTOU |
| I-005 | SupplierVoteCast único por (session, voter) | DR-005 | Integration |
| I-006 | CompanyEvaluation única | DR-006 | Integration |
| I-007 | TimelineEntry imutável (sem UPDATE, DELETE) | DR-007 | Manual grep + unit |
| I-008 | Announcement imutável pós publish | DR-008 | Unit |
| I-009 | Minutes (Ata) imutável pós publish | DR-009 | Unit |
| I-010 | Quota saldo ∈ [0, ∞] e limit respeitado | DR-010 | PBT (300 casos) |
| I-011 | Money BRL é int64 centavos | DR-011 | Unit (tipo) |
| I-012 | ABAC ordem: admin → role → action → tenant → scope | DR-012 | PBT (400 casos) |
| I-013 | Tenant isolation: queries sempre filtram tenant | DR-013 | 100+ integration tests cross-tenant |
| I-014 | Assembly não inicia sem edital publicado | DR-014 | Unit + integration |
| I-015 | Vídeo locked por 90d pós publish | DR-015 | Unit |
| I-016 | Reputação Bronze→Diamante é determinística | DR-016 | Unit (mesmo input → mesmo output) |
| I-017 | UUIDv7 em domain events | DR-017 | Unit |
| I-018 | Password hash bcrypt cost ≥ 12 | DR-018 | Config check + integration |
| I-019 | PII nunca em logs | DR-019 | CI grep |
| I-020 | HMAC verify antes de parse em webhook | DR-020 | Unit + integration + pattern enforce |

---

## Como testar cada invariante

### PBT
Invariante precisa PBT quando:
- Há muitas combinações possíveis de input
- Mutações no estado devem preservar a propriedade
- Regras matemáticas envolvidas (soma, média, conservação)

Usamos `pgregory.net/rapid`. Exemplos em uso:
- `agenda_item_pbt_test.go` — fração ideal
- `feature_usage_pbt_test.go` — quotas
- `abac_engine_pbt_test.go` — ABAC

### Integration
Invariante precisa integration quando:
- Depende do DB (UNIQUE, FK, CHECK)
- TOCTOU — só testable com DB real + concorrência
- Cross-module

Usamos `testcontainers-go` + `testutil.TruncateAll()`.

### Unit
Invariante testável por construção em unit:
- VO validation
- Entity method precondition
- Simple state transitions

### CI static check
Invariante testável por análise estática:
- Grep por padrões proibidos (PII em log, `_ = fn()`, `SELECT *`, bloco else, etc.)

---

## Relatório: invariantes violados historicamente

| # | Invariante | Violação | Fix | AUDIT |
|---|---|---|---|---|
| I-004 | Vote único | TOCTOU (race HasVoted + Create) | UNIQUE DB + ErrConflict em Create | A-025 |
| I-005 | SupplierVote único | Race em contador | UoW + SELECT FOR UPDATE | A-026, A-030 |
| I-006 | CompanyEvaluation única | TOCTOU | ErrConflict em Create | A-029 |
| I-013 | Tenant isolation | 12 queries com SELECT *  | Colunas explícitas | A-021 |
| I-014 | Assembly start sem edital | Rota PublishEdital inexistente | Adicionada | A-013 |
| I-019 | PII em log | CNPJ, invite_token plain | Masked() | A-003, A-004 |
| I-020 | Mux webhook em /api/v1 | Authenticate bloqueava | Rota pública | A-022 |
| I-016 | Saga orfã | UploadVideo/StartLive sem compensar | CancelUpload/EndRoom | A-027, A-028 |

---

## Monitoramento pós-prod

Além dos testes, invariants devem ser verificáveis em prod:

- **I-001**: metric custom `condominium_fracao_sum` — alerta se > 100 em qualquer condomínio
- **I-003**: metric `active_sessions_per_user` — alerta se > MAX
- **I-007**: audit log de UPDATE/DELETE em timeline_entries → alerta se > 0
- **I-013**: log de "cross-tenant access attempt" (ABAC deny com tenant_mismatch) → dashboard

---

## Links

- [[_moc]]
- [[domain-rules]]
- [[business-rules]]
- [[implementation-rules]]
- [[../09-testing/test-strategy]]
- [[../09-testing/property-based-tests]]
- [[../11-audit/AUDIT]]
