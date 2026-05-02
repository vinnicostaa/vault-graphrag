---
title: Null Object
type: pattern
tags:
  - pattern
  - behavioral
  - refactoring-catalog
aliases:
  - Null Object Pattern
  - Active Nothing
created: '2026-04-27'
updated: '2026-04-27'
---

# Null Object

Pattern comportamental do *Refactoring catalog* (Fowler/Woolf). Substitui `null` (ou `nil`/`None`/`undefined`) por **um objeto neutro com a mesma interface** — chamadas em "vazio" não levantam exceção, retornam o resultado neutro do tipo.

## Quando usar

- Caller não consegue agir diferente quando alvo é nulo: `logger.Info("...")` num `NullLogger` é no-op; `cache.Get(k)` num `NullCache` retorna `(zero, false)`; `metrics.Inc()` num `NoopMetrics` ignora.
- Eliminar guard `if x == nil` repetido por toda a chamada — caller chama direto.
- Stub para testes: `NoopEmailProvider`, `NullPaymentGateway` com comportamento neutro previsível.

## Quando evitar

- Quando o caller **precisa** distinguir "não tem" de "tem com valor zero" — usar `Option`/`Maybe` é mais explícito.
- Quando a operação no nulo não é trivialmente "no-op" (deletar registro inexistente pode precisar erro).

## Sinais de reinvenção

- Provider com `if provider == nil { return }` em todo método público.
- Funções helpers `safeCall(fn, default)` espalhadas.
- Mocks de teste que só implementam silêncio — codifica como `Noop*` real, reutilizável.

## Trade-offs

**Vantagens**: caller fica linear (sem guard); facilita injeção de dependência opcional; dispatch polimórfico mantém shape de tipo único; perfeito para [[../principles/solid]] DIP — interface canônica + impl real ou no-op.

**Desvantagens**: pode mascarar bug ("salvei mas não chegou em lugar nenhum"); exige disciplina de logging para detectar `Noop*` ativo em prod por engano.

## Relações

- Antagônico de [[../runtime-antipatterns/op-016-force-unwrap]]: em vez de unwrap forçado de `nil`, sub-objeto neutro.
- Útil ao lado de [[strategy]] (NullStrategy) e [[state]] (NullState).
- Complementar a `Option<T>` quando linguagem suporta (Rust, Kotlin, TS strict).

## Fontes

- Bobby Woolf, "Null Object" — *Pattern Languages of Program Design 3* (1998).
- [Refactoring catalog — Introduce Null Object](https://refactoring.com/catalog/introduceNullObject.html)
- Fowler, *Refactoring* 2nd ed. (2018).

## Links

- [[_moc]]
- [[../runtime-antipatterns/op-016-force-unwrap]]
- [[strategy]]
- [[state]]
