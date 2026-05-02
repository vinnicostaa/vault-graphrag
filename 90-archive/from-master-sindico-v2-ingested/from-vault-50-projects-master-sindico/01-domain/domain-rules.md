---
title: Domain Rules — Master Síndico
type: project
tags: [master-sindico, domain, invariants, domain-rules]
project: master-sindico
created: 2026-04-22
---

# Domain Rules

**Invariantes e pré-/pós-condições** das entidades. Violação = bug. Testado via PBT quando crítico.

Distinto de [[business-rules]] (o **que** o negócio quer) — aqui é **como** o domínio assegura.

---

## DR-001 — Condominium: soma de FracaoIdeal ≤ 100%

```
PARA TODO c ∈ Condominium, u ∈ c.units:
  Σ u.fracao_ideal ≤ 100.00
```

- PBT: `agenda_item_pbt_test.go` (100 casos)
- Enforce: método `Condominium.AddUnit(Unit)` rejeita se somar > 100

## DR-002 — Unit: FracaoIdeal ∈ (0, 100]

```
PARA TODO u ∈ Unit: 0 < u.fracao_ideal ≤ 100.00
```

- Enforce: VO `FracaoIdeal.Validate()` rejeita 0 ou > 100

## DR-003 — Session: no máximo 1 ativa por user (default)

```
PARA TODO u ∈ User: |active_sessions(u)| ≤ MAX_ACTIVE_SESSIONS
```

- Enforce: `InvalidatePreviousByUserID` antes de criar sessão nova

## DR-004 — Vote: único por (agenda_item, voter)

```
PARA TODO (agenda_item_id, voter_id): |votes(agenda_item_id, voter_id)| ≤ 1
```

- Enforce: UNIQUE constraint `votes(agenda_item_id, voter_id)` (migration 00504)
- Double-vote retorna `ErrConflict` (A-025)

## DR-005 — SupplierVoteCast: único por (session, voter)

```
PARA TODO (session_id, voter_id): |casts(session_id, voter_id)| ≤ 1
```

- Enforce: UNIQUE em `supplier_vote_casts`
- Contador em `SupplierVoteSession` incrementado dentro de UoW (A-026 fix)

## DR-006 — CompanyEvaluation: única por (company, evaluator, condominium)

```
UNIQUE (company_id, evaluator_user_id, condominium_id)
```

- Enforce: UNIQUE constraint + `ErrConflict` em Create (A-029 fix)

## DR-007 — TimelineEntry: imutável

```
PARA TODO t ∈ TimelineEntry: t NUNCA é UPDATE ou DELETE
```

- Enforce: tabela sem `deleted_at` nem `updated_at`; sem queries de UPDATE
- Correção = entry novo com `corrects_entry_id` link

## DR-008 — Announcement: imutável após publicação

```
PARA TODO a ∈ Announcement, a.state = published:
  a.NUNCA edita content, title, attachments
```

- Enforce: Publish seta flag; handlers de update rejeitam se `state=published`

## DR-009 — Minutes (Ata): imutável após publicação

Igual a DR-008.

- Assinatura digital adicional (M3+)

## DR-010 — Quota: saldo ∈ [0, ∞]

```
PARA TODO q ∈ FeatureUsage:
  q.used ≤ q.limit (se !q.unlimited)
  q.used ≥ 0
```

- PBT: `feature_usage_pbt_test.go`
- `Consume(n)` retorna erro se `used + n > limit`

## DR-011 — Money BRL: int64 centavos

```
PARA TODO m ∈ Money:
  TYPE(m.value) == int64
  UNIT(m) == BRL centavos
```

- Enforce: pkg `money` expõe só int64; operações aritméticas preservam tipo

## DR-012 — ABAC: ordem estrita

```
decision(user, resource, action, ctx):
  1. IF user.is_admin → PERMIT (admin_bypass)
  2. IF action NOT IN catalog[resource.type] → DENY (action_not_configured)
  3. IF user.role NOT IN allowed_roles[...] → DENY (role_not_allowed)
  4. IF resource.tenant_id NOT IN user.tenant_ids → DENY (tenant_mismatch)
  5. IF plan_required[...] > user.plan_tier → DENY (scope_restricted)
  6. PERMIT
```

- PBT: `abac_engine_pbt_test.go` (400 casos)

## DR-013 — Tenant isolation: queries respeitam

```
PARA TODA query em recurso escopado: WHERE condominium_id = $user.tenant_id (ou IN)
```

- Enforce: convenção sqlc + revisão em PR; 100+ testes cross-tenant

## DR-014 — Assembly: não inicia sem edital publicado

```
Start(Assembly) REQUER Assembly.edital_published_at != NULL
```

- Enforce: caso de uso `StartAssemblyUseCase` valida (A-013 fix)

## DR-015 — Vídeo locked: sem remove por 90d

```
PARA TODO v ∈ Video, v.state = published:
  v.locked_until = v.published_at + 90 days
  REMOVE(v) requer NOW() ≥ v.locked_until OR user.is_admin
```

- Enforce: validação em `RemoveVideoUseCase`

## DR-016 — Reputação: determinística

```
tier(company) = f(company.closed_deals, company.avg_rating, company.condominiums_count, company.harassment_reports)
```

- Função `ReputationCalculator` sem randomness
- Recalcula em cada `CompanyEvaluation` nova

## DR-017 — Events ordenáveis (UUIDv7)

```
PARA TODO e ∈ DomainEvent: e.id é UUIDv7
```

- Permite ordenação global sem coordenação
- `pkg/utils/uuid7.go`

## DR-018 — Password: hash bcrypt cost ≥ 12

```
PARA TODO p ∈ Password armazenado:
  p é BCRYPT(plain) com cost ≥ 12
```

- Zitadel manage — confirmar config

## DR-019 — PII em logs: proibido

```
PARA TODA chamada a logger:
  fields NÃO contém email, cpf, cnpj, token, password, PAN, endereço completo
```

- Enforce: CI grep (lista negra) + code review

## DR-020 — Webhook body: HMAC antes de parse

```
POST /webhooks/stripe:
  verify(body, signature, secret) → se falhou: 400 SEM parse
  parse(body) → processa evento
```

- Enforce: middleware `VerifyStripeSignature` antes do handler

---

## Rastreamento

- PBT: DR-001, DR-010, DR-011, DR-012, DR-013
- Integration tests: DR-003, DR-004, DR-005, DR-006
- Unit tests: todos
- CI static check: DR-013 (grep), DR-019 (grep lista), DR-020 (code pattern)

## Links

- [[_moc]]
- [[business-rules]]
- [[implementation-rules]]
- [[invariants]]
- [[../09-testing/property-based-tests]]
- [[../07-quality/PATTERNS]]
