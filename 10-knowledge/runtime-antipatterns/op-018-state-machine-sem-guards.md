---
title: OP-018 — Transição de state machine sem guards
type: antipattern
tags:
  - antipattern
  - runtime
  - operational
  - state
  - invariants
id: OP-018
severity: High
created: 2026-04-24
updated: 2026-04-24
doc-consulted: 2026-04-24
aliases:
  - Unguarded state transition
---

# OP-018 — Transição de state machine sem guards

**Severidade**: High
**Categoria**: Estado / Invariantes

## Sintoma

Tabela de transições `VALID_TRANSITIONS[from].includes(to)` valida **apenas topologia** (posso sair de `paused` para `active`?), mas não **pré-condições de negócio** (tem payment method? não está em dispute? não excedeu tentativas?). Sistema aceita transição que quebra invariantes.

## Por que é problema

- **Estado inválido persistido**: `Subscription` com status `active` sem payment method.
- **Fix N vezes**: cada caminho que faz a transição precisa re-validar manualmente.
- **Debug caro**: sintoma aparece em cobrança falhar — distante da transição que deveria ter bloqueado.

## Exemplo (anti-pattern)

```ts
// ❌ ERRADO — só valida topologia
const VALID_TRANSITIONS = {
  trial:    ['active', 'canceled'],
  active:   ['paused', 'canceled'],
  paused:   ['active', 'canceled'],
}

function transition(sub: Subscription, to: SubscriptionStatus) {
  if (!VALID_TRANSITIONS[sub.status].includes(to)) {
    throw new InvalidTransitionError()
  }
  sub.status = to   // aceita paused → active sem payment method
}
```

## Fix preferido — guards por transição

```ts
type Guard = (sub: Subscription) => void   // throw se falha

const GUARDS: Record<`${Status}->${Status}`, Guard> = {
  'paused->active': (sub) => {
    if (!sub.paymentMethod) throw new BusinessError('paymentMethodRequired')
    if (sub.inDispute)      throw new BusinessError('cannotResumeInDispute')
  },
  'trial->active': (sub) => {
    if (!sub.paymentMethod) throw new BusinessError('paymentMethodRequired')
  },
  // ... demais combinações
}

function transition(sub: Subscription, to: SubscriptionStatus) {
  const key = `${sub.status}->${to}` as const
  const guard = GUARDS[key]
  if (!guard) throw new InvalidTransitionError()   // topologia
  guard(sub)                                        // pré-condições
  sub.status = to
}
```

## Alternativa

- **Bibliotecas de state machine tipadas**: [XState](https://xstate.js.org/) em TS, [Statelyai](https://stately.ai) para design visual + export, máquinas-específicas-de-linguagem.
- **Métodos de domínio na entidade**: `subscription.resume()` valida internamente, caller não conhece transição.
- **Sealed/tagged union de estados** (Rust enum, Kotlin sealed, TS discriminated union) — transições viram pattern matching compile-time-checked.

## Quando tolerar

State machine com 2-3 estados, 1 transição, sem pré-condições de negócio (ex.: toggle simples). Adicionar guard só cria cerimônia sem valor.

## Relações

- **Patterns**: [[../patterns/state]] (objetos-estado com lógica polimórfica), Specification Pattern (guards compostos)
- **OPs relacionados**: [[op-019-mutacao-sem-validacao]] (mesmo family), [[op-002-check-then-act]] (invariante entre check e transição)

## Fontes

- [[../../40-templates/AGENTS_SPEC]] §4 OP-018
- XState — [State Machines and Statecharts](https://stately.ai/docs/state-machines-and-statecharts) (consultada 2026-04-24)
- David Harel — *Statecharts: a visual formalism for complex systems* (1987)

## Links

- [[_moc]]
- [[../patterns/state]]
- [[op-019-mutacao-sem-validacao]]
- [[../principles/solid]]
