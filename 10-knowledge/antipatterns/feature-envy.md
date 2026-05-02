---
title: Feature Envy (AP-007)
type: antipattern
tags: [antipattern, code-smell, coupler, cohesion]
source: "https://refactoring.guru/smells/feature-envy"
source_date: 2026-04-24
created: 2026-04-24
updated: 2026-04-24
aliases: [envy, wrong-home]
---

# AP-007 — Feature Envy

Método que está "apaixonado" por **outra** classe — acessa mais campos/métodos dela do que da sua própria. Sinal de que o método foi colocado no lugar errado.

## Sintomas

- Método `A.compute()` chama `b.x()`, `b.y()`, `b.z()` e quase nada de `this`
- Sequências `obj.a.b.c()` (combina com *Message Chains* — smell irmão)
- Mudanças em classe `B` exigem sempre mexer em métodos de classe `A`
- Getters de `B` expostos só para `A` funcionar
- Conversa: "por que `A.calcularFrete(b)` e não `b.calcularFrete()`?" — e ninguém tem resposta

## Por que é problema

**Coesão errada**: lógica que pertence a `B` mora em `A`, criando acoplamento invertido.

**Quebra encapsulamento**: para `A` funcionar, `B` precisa expor internals via getters, transformando entity em *data class*.

**Regra de negócio fragmentada**: o que deveria ser `invoice.total()` espalha em `Calculator.computeTotal(invoice)`, `Report.sumInvoice(invoice)`, `Tax.invoiceBase(invoice)`.

**Viola Tell-Don't-Ask**: você pergunta tudo ao objeto em vez de pedir que ele faça.

## Exemplo

```java
// ❌ Feature envy
class Invoice {
    List<Line> lines; // getter público
    Discount discount; // getter público
    Money currency;
}

class InvoicePrinter {
    Money total(Invoice inv) {
        Money sum = new Money(0, inv.getCurrency());
        for (Line l : inv.getLines()) sum = sum.add(l.getSubtotal());
        return sum.subtract(inv.getDiscount().apply(sum));
    }
}

// ✅ lógica migrada para onde os dados moram
class Invoice {
    public Money total() {
        Money sum = lines.stream()
            .map(Line::subtotal)
            .reduce(Money.zero(currency), Money::add);
        return discount.apply(sum);
    }
}
```

## Como refatorar

- **Move Method**: migrar o método invejoso para a classe alvo
- **Extract Method** primeiro se só parte do método é envy — mover só a parte
- **Extract Class** quando o método envy + campos formam um conceito novo
- **Tell-Don't-Ask**: converter "lê getters e decide" em "pede para o objeto decidir"

## Quando tolerar

- **Visitor pattern** — por definição opera sobre outra estrutura; envy é intencional e controlado
- **Relatórios e agregadores cross-entity** (read-model) que precisam juntar campos de várias fontes
- **Strategy pattern** onde o algoritmo é legitimamente externo e configurável
- **DTO mapping layer** (domain ↔ JSON) — acesso extensivo aos campos é esperado

## Patterns relacionados (que resolvem)

- [[../patterns/visitor]] — legitima o envy como design
- [[anemic-domain-model]] — envy crônico é sintoma de modelo anêmico
- [[../patterns/strategy]] — extrair algoritmo quando varia
- [[../principles/solid]] — Law of Demeter / SRP

## Fontes

- Refactoring Guru — [Feature Envy](https://refactoring.guru/smells/feature-envy)
- Fowler, M. (2018). *Refactoring: Improving the Design of Existing Code*. 2nd ed. Addison-Wesley.
- Beck, K. (1997). *Smalltalk Best Practice Patterns* — "tell, don't ask".

## Links

- [[_moc]]
- [[../_moc]]
- [[../patterns/_moc]]
- [[../patterns/visitor]]
- [[anemic-domain-model]]
