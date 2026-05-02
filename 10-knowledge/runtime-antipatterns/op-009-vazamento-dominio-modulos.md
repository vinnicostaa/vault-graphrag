---
title: OP-009 — Vazamento de domínio entre módulos
type: antipattern
tags:
  - antipattern
  - runtime
  - operational
  - coupling
  - ddd
id: OP-009
severity: High
created: 2026-04-24T00:00:00.000Z
updated: 2026-04-24T00:00:00.000Z
aliases:
  - Cross-module enum leak
doc-consulted: '2026-04-24'
---

# OP-009 — Vazamento de domínio entre módulos

**Severidade**: High
**Categoria**: Acoplamento

## Sintoma

Módulo/Bounded Context B tem `switch (someEnumFromModuleA)` em sua camada de aplicação/domínio. Adicionar um valor novo no enum do A quebra a lógica de B silenciosamente (ou explicitamente se a linguagem exige exhaustive match). B conhece detalhes internos de A — DDD quebrado.

## Por que é problema

- **Change preventer**: simples adição em A exige tocar B.
- **Domain leakage**: ubiquitous language de A vaza para B, gera drift conceitual.
- **Independent deploy quebrado**: em sistema distribuído, A não pode evoluir sem B saber.

## Exemplo (anti-pattern)

```go
// ❌ ERRADO — BC "Billing" importa enum de "Identity"
func (s *BillingService) OnUserEvent(ctx context.Context, e identity.UserEvent) error {
    switch e.Kind {
    case identity.UserKindAdmin:    // ← conhecimento do domínio alheio
    case identity.UserKindCustomer:
    // ...
    }
}
```

## Fix preferido — mapper + enum local

```go
// BC Billing define seu próprio vocabulário
type BillingSubject int
const (
    BillingSubjectStandard BillingSubject = iota
    BillingSubjectTaxExempt
)

// Mapper na camada de aplicação (adapter layer)
func mapFromIdentity(u identity.UserKind) BillingSubject {
    switch u {
    case identity.UserKindAdmin:    return BillingSubjectTaxExempt
    case identity.UserKindCustomer: return BillingSubjectStandard
    default:                        return BillingSubjectStandard
    }
}

// BC Billing opera só com BillingSubject
func (s *BillingService) OnUserEvent(ctx context.Context, e identity.UserEvent) error {
    subject := mapFromIdentity(e.Kind)
    // ... lógica em termos de BillingSubject
}
```

Alternativa de comunicação: **Domain Events** com payload contratual estável (ver AGENTS_SPEC §3.10) — B consome evento sem conhecer enum interno de A.

## Alternativa

- Published Language / Open Host Service (padrões DDD) para contrato versionado entre BCs.
- Shared Kernel **apenas** se os dois BCs são propriedade do mesmo time e evoluem juntos.

## Quando tolerar

Nos primeiros dias de um MVP monolítico, antes das fronteiras estarem claras, cross-module enum pode ser "dívida conhecida". Marcar com `// TODO DDD: mapear pra enum local quando contexto X emergir`.

## Relações

- **Patterns**: [[../patterns/adapter]], [[../patterns/facade]], Domain Events (AGENTS_SPEC §3.10)
- **OPs relacionados**: [[op-010-feature-importa-internals]] (mesma família), [[op-013-switch-replicado]] (sintoma derivado)

## Fontes

- [[../../40-templates/AGENTS_SPEC]] §4 OP-009, §3.18 (DDD + Vertical Slices)
- Evans — *Domain-Driven Design*, Ch. 14 (Bounded Context)
- Vernon — *Implementing DDD*, Ch. 13 (Integrating BCs)

## Links

- [[_moc]]
- [[../architecture/ddd-strategic-tactical]]
- [[../patterns/adapter]]
- [[op-010-feature-importa-internals]]
