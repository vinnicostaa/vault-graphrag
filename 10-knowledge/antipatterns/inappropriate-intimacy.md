---
title: Inappropriate Intimacy (AP-020)
type: antipattern
tags: [antipattern, code-smell, coupler, encapsulation]
source: https://refactoring.guru/smells/inappropriate-intimacy
source_date: 2026-04-24
created: 2026-04-24
updated: 2026-04-24
aliases: [too-close-classes, encapsulation-leak]
---

# AP-020 — Inappropriate Intimacy

Duas classes que **conhecem demais** uma da outra — acessam campos privados, chamam métodos internos, conhecem detalhes de implementação que deveriam ser invisíveis. Elas "namoram escondido" e refactor de uma obriga refactor da outra.

## Sintomas

- Classe `A` chama `B.internalField` ou método package-private de `B` em alto volume
- Uso frequente de `friend` (C++), `internal` (C#), `package-private` (Java) pra expor coisas só pra classe parceira
- Reflection pra pular `private`
- Testes de `A` precisam montar estado interno de `B`
- Mudar representação de `B` quebra `A` imediatamente
- Herança usada **pra obter acesso protegido** (não pra substituição real) — ver [[refused-bequest]]

## Por que é problema

**Encapsulamento rompido**: `B` não pode evoluir sem quebrar `A`, que sabe coisas que não deveria. Resultado: custo O(N) de mudança onde deveria ser O(1).

**Acoplamento bidirecional**: mudança numa força mudança na outra — em prática são **uma só classe** mal dividida.

**Semântica confusa**: se `A` precisa tanto dos internos de `B`, talvez o comportamento pertença a `B` (ver [[feature-envy]]) ou as duas sejam partes de um conceito maior.

**Testabilidade ruim**: não dá pra testar `A` sem configurar internals de `B` — mocks frágeis espalhados.

## Exemplo

```java
// ❌ Intimidade inadequada: Order mexe nos internos de Customer
class Customer {
    List<Order> orders = new ArrayList<>();   // package-private "por conveniência"
    BigDecimal creditLimit;
    BigDecimal outstandingBalance;
}

class OrderService {
    void place(Customer c, Order o) {
        // Sabe DEMAIS da estrutura de Customer
        if (c.outstandingBalance.add(o.total()).compareTo(c.creditLimit) > 0)
            throw new CreditExceeded();
        c.orders.add(o);
        c.outstandingBalance = c.outstandingBalance.add(o.total());
    }
}

// ✅ Customer protege invariante
class Customer {
    private final List<Order> orders = new ArrayList<>();
    private BigDecimal creditLimit;
    private BigDecimal outstandingBalance;

    public void place(Order o) {
        if (outstandingBalance.add(o.total()).compareTo(creditLimit) > 0)
            throw new CreditExceeded();
        orders.add(o);
        outstandingBalance = outstandingBalance.add(o.total());
    }
}
```

## Como refatorar

- **Move Method / Move Field**: migrar lógica e campos para a classe com legitimidade sobre eles
- **Extract Class**: se as duas compartilham conceito, extrair o conceito como 3ª classe e dar a ele responsabilidade clara
- **Change Bidirectional Association to Unidirectional**: se `A` conhece `B` e `B` conhece `A` sem necessidade, cortar um lado
- **Hide Delegate**: cliente fala com uma, que internamente fala com a outra — resolve [[message-chains]] também
- **Replace Inheritance with Delegation**: quando a intimidade veio de herança "por acesso protegido"

## Quando tolerar

- **Aggregate root + entidade interna** (DDD) — agregado pode ver seus filhos porque é **uma unidade de consistência**; root controla
- **Inner class / nested class** — por design compartilha escopo com a outer
- **Módulo coeso** (package/namespace) — classes coirmãs que formam conceito único podem expor internos entre si
- **Friend pattern em C++** usado de forma limitada e documentada (operator overload, iterator interno)
- **Testes da mesma classe** (test friend) — diferente de produção

## Patterns relacionados (que resolvem)

- [[feature-envy]] — smell relacionado; `A` usa demais `B` e talvez o método pertença a `B`
- [[message-chains]] — cadeias longas são sintoma de intimidade excessiva
- [[data-class]] — classes-dado alimentam intimidade porque expõem tudo
- [[../patterns/facade]] — expor subconjunto estável
- [[../patterns/mediator]] — coordenar sem intimidade mútua entre componentes
- [[../architecture/ddd-strategic-tactical]] — aggregate boundaries como contorno explícito

## Fontes

- Refactoring Guru — [Inappropriate Intimacy](https://refactoring.guru/smells/inappropriate-intimacy)
- Fowler, M. (2018). *Refactoring*. 2nd ed. Addison-Wesley.
- Evans, E. (2003). *Domain-Driven Design* — Aggregates.

## Links

- [[_moc]]
- [[../_moc]]
- [[../patterns/_moc]]
- [[feature-envy]]
- [[message-chains]]
- [[data-class]]
