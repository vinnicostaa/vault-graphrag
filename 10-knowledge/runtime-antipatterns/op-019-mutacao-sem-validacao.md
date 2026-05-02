---
title: OP-019 — Mutação de campo derivado sem validação
type: antipattern
tags:
  - antipattern
  - runtime
  - operational
  - state
  - invariants
id: OP-019
severity: Medium
created: 2026-04-24
updated: 2026-04-24
doc-consulted: 2026-04-24
aliases:
  - Unvalidated mutation
  - Broken invariant on set
---

# OP-019 — Mutação de campo derivado sem validação

**Severidade**: Medium
**Categoria**: Estado / Invariantes

## Sintoma

Setters/mutators aceitam valores que violam invariantes da entidade. Ex.: `renewPeriod(start, end)` aceita `start >= end`, cria período inválido. `addMember(member, role)` aceita membro duplicado. Invariante só valida "na borda de entrada" do sistema, não no ponto de mutação.

## Por que é problema

- **Invariante não é proteção real** se qualquer caminho pode quebrá-la.
- **Código defensivo espalha**: cada consumidor do estado precisa re-validar.
- **Invalid state persisted**: persistência salva entidade corrompida; load posterior traz bug novo.

## Exemplo (anti-pattern)

```go
// ❌ ERRADO — setter aceita período inválido
func (p *Period) Renew(start, end time.Time) {
    p.start = start
    p.end   = end   // se end < start, Period agora é inválido
}
```

## Fix preferido — validar no setter; melhor ainda, Value Object imutável

### Opção 1: guard no setter

```go
func (p *Period) Renew(start, end time.Time) error {
    if !end.After(start) {
        return ValidationError("end must be after start")
    }
    p.start = start
    p.end   = end
    return nil
}
```

### Opção 2: Value Object imutável (preferida)

```go
// period.go
type Period struct {
    start, end time.Time
}

func NewPeriod(start, end time.Time) (Period, error) {
    if !end.After(start) {
        return Period{}, ValidationError("end must be after start")
    }
    return Period{start: start, end: end}, nil
}

func (p Period) Start() time.Time { return p.start }
func (p Period) End()   time.Time { return p.end }

// Nunca muta — cria novo
func (p Period) ExtendBy(d time.Duration) (Period, error) {
    return NewPeriod(p.start, p.end.Add(d))
}
```

Entidade que usa Period fica naturalmente livre de estado inválido — a classe inteira de bug desaparece.

## Alternativa

- **Invariantes como métodos de classe** (`subscription.extend(duration)` valida internamente).
- **Smart constructors** em linguagens funcionais — valida na construção, não exporta setter.
- **Property-based testing** das invariantes — gera casos adversariais automaticamente.

## Quando tolerar

Mutação em DTO de transporte (sem regra de negócio). Dentro do domínio, quase nunca — o custo de uma VO é menor que o custo do bug.

## Relações

- **Patterns**: Value Object, [[../patterns/builder]] (build valida), [[../patterns/state]]
- **OPs relacionados**: [[op-018-state-machine-sem-guards]] (guards em transições), [[op-016-force-unwrap]]
- **Code smells clássicos**: [[../antipatterns/primitive-obsession|AP-005]], [[../antipatterns/data-class|AP-014]]

## Fontes

- [[../../40-templates/AGENTS_SPEC]] §4 OP-019, §3.7 Value Object
- Evans — *Domain-Driven Design*, Ch. 5 (Value Objects)
- Fowler — [ValueObject](https://martinfowler.com/bliki/ValueObject.html) (consultada 2026-04-24)

## Links

- [[_moc]]
- [[../patterns/builder]]
- [[../antipatterns/primitive-obsession]]
- [[op-018-state-machine-sem-guards]]
