---
title: Lazy Class (AP-017)
type: antipattern
tags: [antipattern, code-smell, dispensable]
source: https://refactoring.guru/smells/lazy-class
source_date: 2026-04-24
created: 2026-04-24
updated: 2026-04-24
aliases: [freeloader-class, do-nothing-class]
---

# AP-017 — Lazy Class

Classe que **não faz o suficiente pra justificar existir**. Tipicamente 1-2 métodos finos, ou uma subclasse cuja especialização é trivial, ou abstração criada em antecipação (ver [[speculative-generality]]) que nunca ganhou massa crítica.

## Sintomas

- Classe com 1 método de 3 linhas
- Subclasse que adiciona 1 constante e nada mais
- "Wrapper" que só delega todo método pra outra classe (`foo.bar() → inner.bar()`)
- Arquivo com 80% imports e `class X {}` vazio/quase vazio
- Factory retornando sempre 1 tipo concreto
- Projeto com 40 classes onde 20 têm 1 teste trivial cada

## Por que é problema

**Custo cognitivo > benefício**: cada classe é um nível de indireção pro leitor navegar, um arquivo a abrir, um teste a manter. Se o valor é só "encapsular 3 linhas", está na conta errada.

**Dilui coesão**: lógica fragmentada em classes-bebê dificulta enxergar o algoritmo completo.

**Custo de refactor**: extrair depois é barato; ter que inlinar uma classe por feature pra enxergar o problema é caro.

**Aumenta surface**: mais classes públicas = mais API pra manter compatível.

## Exemplo

```typescript
// ❌ Lazy class — valor é zero, só polui namespace
class DiscountCalculator {
    calculate(price: number): number { return price * 0.9; }
}

class OrderService {
    constructor(private calc = new DiscountCalculator()) {}
    total(order: Order): number { return this.calc.calculate(order.sum()); }
}

// ✅ Inline — se aparecer 2ª variante, EXTRAIR
class OrderService {
    private static readonly DISCOUNT = 0.9;
    total(order: Order): number { return order.sum() * OrderService.DISCOUNT; }
}
```

## Como refatorar

- **Inline Class**: mover métodos pra classe que usa, deletar a lazy
- **Collapse Hierarchy**: subclasse trivial absorvida pelo pai (remove 1 nível de herança)
- **Remove Middle Man** (ver [[middle-man]]): se ela só delega, cliente fala direto com o alvo
- **YAGNI check**: essa classe nasceu pra atender uso real ou pra "um dia"?
- Verificar se há teste que depende *só* dela — se sim, o teste pode estar mal-posicionado também

## Quando tolerar

- **Value Object** pequeno com invariante real (mesmo que só 1 método — o construtor protege) — valor está na invariante, não no comportamento
- **Marker / tag classes** para DI, type discrimination, pattern matching
- **Exceptions** específicas — `class InvalidCpfError extends ValidationError {}` é legítimo e idiomático
- **Port/Adapter** (hexagonal) onde interface pequena é **intencional** — contrato explícito vale
- **Nova classe que vai crescer**: ok nascer pequena se há plano claro e horizonte curto (não "um dia")

## Patterns relacionados (que resolvem)

- [[speculative-generality]] — Lazy Class é frequentemente resultado
- [[middle-man]] — caso particular de Lazy Class que só delega
- [[../methodology/gsd-get-shit-done]] — regra YAGNI
- [[../principles/clean-code]] — "classes should be small, but have a reason"
- [[../principles/do-dont-checklist]] — YAGNI, KISS

## Fontes

- Refactoring Guru — [Lazy Class](https://refactoring.guru/smells/lazy-class)
- Fowler, M. (2018). *Refactoring*. 2nd ed. Addison-Wesley.
- Martin, R. C. (2000). *Extreme Programming: A Gentle Introduction* — YAGNI.

## Links

- [[_moc]]
- [[../_moc]]
- [[speculative-generality]]
- [[middle-man]]
- [[../methodology/gsd-get-shit-done]]
