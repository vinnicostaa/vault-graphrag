---
title: OP-014 — Construtor com 10+ parâmetros posicionais
type: antipattern
tags:
  - antipattern
  - runtime
  - operational
  - duplication
  - design
id: OP-014
severity: Medium
created: 2026-04-24
updated: 2026-04-24
doc-consulted: 2026-04-24
aliases:
  - God constructor
  - Positional args hell
---

# OP-014 — Construtor com 10+ parâmetros posicionais

**Severidade**: Medium
**Categoria**: Duplicação / Design

## Sintoma

Entidade / use case com 10+ parâmetros posicionais: `new Subscription(id, userId, planId, startDate, endDate, status, trialEnd, canceledAt, paymentMethodId, providerRef, metadata, tenantID, createdAt, updatedAt)`. Trocar ordem por engano compila (se tipos forem similares) mas explode em runtime. Relaciona-se com [[../antipatterns/long-parameter-list|AP-004]].

## Por que é problema

- **Fragilidade**: refactoring automatizado pode perder ordem sutil.
- **Ilegibilidade**: na call-site `new X(..., "", null, 0, false, "active", ...)` — impossível saber qual é qual.
- **Testes frágeis**: mocks passam valores "quaisquer" posicionalmente; mudança quebra todos.

## Exemplo (anti-pattern)

```ts
// ❌ ERRADO — 14 parâmetros posicionais
const s = new Subscription(
  id, userId, planId,
  startDate, endDate, status,
  trialEnd, canceledAt,
  paymentMethodId, providerRef,
  metadata, tenantID,
  createdAt, updatedAt
)
```

## Fix preferido — objeto de configuração ou Builder

### Opção 1: Config object (mais simples)

```ts
const s = new Subscription({
  id, userId, planId,
  startDate, endDate, status,
  trialEnd, canceledAt,
  paymentMethodId, providerRef,
  metadata, tenantID,
  createdAt, updatedAt,
})
```

Vantagens: nomeado, ordem irrelevante, IDE autocompleta, defaults simples.

### Opção 2: Builder Pattern (pra quando há etapas de validação)

```go
s := NewSubscriptionBuilder().
    WithID(id).
    WithUser(userId).
    WithPlan(planId).
    WithPeriod(startDate, endDate).
    WithTenant(tenantID).
    Build()   // valida invariantes (endDate > startDate, etc.)
```

Vantagens: validação centralizada no `Build()`, partial-builds impossíveis.

## Alternativa

- **Factory methods semânticos**: `Subscription.newTrial(user, plan, 14)`, `Subscription.reactivate(canceled)`. Cada factory encapsula o subset de params relevante.
- **Value Objects agrupando campos relacionados**: `new Subscription(id, user, Period.of(start, end), payment)`.

## Quando tolerar

Construtor de 3-5 params bem tipados (Email, Money, UUIDv7) é aceitável. Regra canônica (Code Calisthenics #8): máx 3-5 params antes de consolidar.

## Relações

- **Patterns**: [[../patterns/builder]] (fix canônico), Factory Method, Value Object
- **OPs relacionados**: [[op-015-validacao-repetida]], [[op-019-mutacao-sem-validacao]]
- **Code smells clássicos**: [[../antipatterns/long-parameter-list|AP-004]], [[../antipatterns/data-clumps|AP-006]]

## Fontes

- [[../../40-templates/AGENTS_SPEC]] §4 OP-014, §3.23 Code Calisthenics #8
- Fowler — *Refactoring 2nd ed.*, Replace Constructor with Factory Function
- Effective Java — Item 2: Consider a builder when faced with many constructor parameters

## Links

- [[_moc]]
- [[../patterns/builder]]
- [[../antipatterns/long-parameter-list]]
- [[../principles/code-calisthenics]]
