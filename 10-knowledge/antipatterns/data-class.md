---
title: Data Class (AP-014)
type: antipattern
tags: [antipattern, code-smell, bloater]
source: https://refactoring.guru/smells/data-class
source_date: 2026-04-24
created: 2026-04-24
updated: 2026-04-24
aliases: [dumb-data-holder, record-only-class]
---

# AP-014 — Data Class

Classe que é **só campos + getters/setters** (ou DTO anêmico usado em toda parte). Não tem comportamento próprio, não protege invariantes, serve de "saco de dados" manipulado de fora. Em domínio rico é o sintoma oposto de *Feature Envy* visto pelo outro lado.

## Sintomas

- Classe com 100% de linhas sendo `get/set` (ou campos públicos)
- Toda validação e cálculo acontece **fora** dela (no service, no controller)
- Construtor aceita qualquer combinação — sem regra de montagem válida
- Herança semântica ausente: `Address` que aceita CEP vazio, cidade nula, estado com 20 caracteres
- Múltiplos lugares chamam a mesma sequência `obj.setX(...); obj.setY(...); validate(obj);`
- Campos mutáveis que "ninguém devia mudar depois de criado" — mas nada impede

## Por que é problema

**Invariantes quebradas**: sem o próprio objeto guardando regra, qualquer caller pode montar estado inválido — "CPF vazio", "total negativo", "data fim antes de início".

**Lógica espalhada**: regras que pertenciam ao objeto acabam repetidas em service/handler/report — atrai [[duplicate-code]] e [[feature-envy]].

**Modelo anêmico**: é o tijolo principal do [[anemic-domain-model]] — entidades sem comportamento viram apenas estrutura de dados com rótulo de classe.

**Encapsulamento zero**: setters públicos permitem mutação descontrolada; refactor interno (mudar representação) é breaking change pra N chamadores.

## Exemplo

```java
// ❌ Data class — só dados, nenhum comportamento
class Money {
    public BigDecimal amount;
    public String currency;
    public BigDecimal getAmount() { return amount; }
    public void setAmount(BigDecimal a) { this.amount = a; }
    public String getCurrency() { return currency; }
    public void setCurrency(String c) { this.currency = c; }
}
// cálculo mora fora e duplica validação:
Money total = new Money();
total.setAmount(a.getAmount().add(b.getAmount()));
total.setCurrency(a.getCurrency()); // e se a≠b? ninguém garante

// ✅ Value Object com comportamento e invariante
final class Money {
    private final BigDecimal amount;
    private final Currency currency;
    public Money(BigDecimal amount, Currency c) {
        if (amount.signum() < 0) throw new IllegalArgumentException("negative");
        this.amount = amount; this.currency = c;
    }
    public Money add(Money other) {
        requireSameCurrency(other);
        return new Money(amount.add(other.amount), currency);
    }
}
```

## Como refatorar

- **Move Method**: migrar cálculos que operam só nos campos para dentro da classe
- **Encapsulate Field**: remover setters públicos; acesso por método de negócio (`withdraw` em vez de `setBalance`)
- **Replace Data Value with Object** / Value Object: imutável + invariantes no construtor
- **Remove Setting Method**: setters que só o ORM precisava viram `protected`/privados
- Se é legítimamente estrutura de transporte (DTO de borda): **marcar como `record`/`@Immutable`** e não misturar com entidade de domínio

## Quando tolerar

- **DTOs de borda** (request/response, Kafka payload) — são contratos de fronteira, ter só dados é o trabalho deles. Nome + pasta `dto/` deixam intenção explícita.
- **Records** para representar tuplas nomeadas em código interno curto (Java `record`, C# `record`, Kotlin `data class` como VO)
- **Read-models** em CQRS — projeções são estruturas de leitura sem comportamento
- **Event payloads** em event sourcing — imutáveis por definição

## Patterns relacionados (que resolvem)

- [[anemic-domain-model]] — Data Class é o componente que alimenta o modelo anêmico
- [[feature-envy]] — lógica que deveria estar na Data Class e migrou pra fora
- [[../patterns/_moc]] — Value Object (VO) como fix clássico
- [[../architecture/ddd-strategic-tactical]] — entidades com comportamento, VOs com invariantes
- [[../principles/solid]] — encapsulamento, SRP

## Fontes

- Refactoring Guru — [Data Class](https://refactoring.guru/smells/data-class)
- Fowler, M. (2018). *Refactoring: Improving the Design of Existing Code*. 2nd ed. Addison-Wesley.
- Evans, E. (2003). *Domain-Driven Design*. Addison-Wesley — Value Objects.

## Links

- [[_moc]]
- [[../_moc]]
- [[../patterns/_moc]]
- [[anemic-domain-model]]
- [[feature-envy]]
