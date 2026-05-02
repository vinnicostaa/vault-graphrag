---
title: Primitive Obsession (AP-005)
type: antipattern
tags: [antipattern, code-smell, bloater, ddd, type-safety]
source: "https://refactoring.guru/smells/primitive-obsession"
source_date: 2026-04-24
created: 2026-04-24
updated: 2026-04-24
aliases: [stringly-typed, primitive-abuse]
---

# AP-005 — Primitive Obsession

Uso de tipos primitivos (`string`, `int`, `float`) para representar conceitos de domínio que têm invariantes próprios (email, dinheiro, CPF, UUID). Sinônimo informal: *stringly-typed*.

## Sintomas

- Parâmetros `email string`, `cpf string`, `amount int` espalhados — validação repetida em cada entrypoint
- Constantes primitivas representando categorias: `status int` com `0=pending, 1=paid, 2=cancelled`
- Valores monetários em `float` ou `int` sem moeda associada
- IDs como `string` passíveis de trocar entre tipos (`userID` passado onde se esperava `orderID`)
- Regex de validação duplicado em múltiplos lugares
- Comparação case-sensitive/insensitive inconsistente

## Por que é problema

**Invariantes não-encapsulados**: qualquer string pode ser atribuída a `email`, mesmo lixo. Validação vira responsabilidade do chamador — e é esquecida.

**Type safety perdido**: compilador não diferencia `UserID` de `OrderID`. Bug aparece em produção.

**Lógica espalhada**: formatação, parsing, normalização replicados. Mudar regra obriga `grep`.

**Linguagem ubíqua fraca**: domínio fala "Money BRL 2 casas decimais"; código fala `int`.

## Exemplo

```go
// ❌ primitivos
func Transfer(fromID string, toID string, amount int64, currency string) error {
    if amount <= 0 { return ErrInvalidAmount }
    if currency != "BRL" && currency != "USD" { return ErrInvalidCurrency }
    // ... validação repetida em cada função
}

// ✅ Value Objects
type AccountID string // newtype / branded type
type Money struct {
    amount   int64    // minor units
    currency Currency
}
func NewMoney(amount int64, c Currency) (Money, error) { ... }

func Transfer(from, to AccountID, amount Money) error { ... }
```

## Como refatorar

- **Replace Data Value with Object**: criar Value Object para cada conceito (Email, Money, PhoneNumber)
- **Replace Type Code with Class/Enum**: status numérico vira enum com comportamento
- **Introduce Parameter Object** para grupos de primitivos relacionados
- **Validação no construtor**: criar VO só com estado válido; erros falham cedo
- Em linguagens com tipos nominais (Go newtype, Rust newtype, TS branded types) — adotar consistentemente

## Quando tolerar

- **Fronteira de serialização** (JSON, protobuf): lá é primitivo mesmo; converter para VO no parse e de volta no response
- **Scripts e one-offs** sem regras de negócio
- **Chaves de cache** e logs: strings bastam
- **Campos sem invariante** (comentário livre, descrição) — `string` é honesto

## Patterns relacionados (que resolvem)

- Value Object pattern (ver roadmap em [[../patterns/_moc]])
- [[anemic-domain-model]] — VO é pilar de domínio rico
- [[../patterns/factory-method]] — factory valida e cria VO
- [[long-parameter-list]] — muitos primitivos viram um VO
- [[data-clumps]]

## Fontes

- Refactoring Guru — [Primitive Obsession](https://refactoring.guru/smells/primitive-obsession)
- Fowler, M. (2018). *Refactoring: Improving the Design of Existing Code*. 2nd ed. Addison-Wesley.
- Evans, E. (2003). *Domain-Driven Design*. Cap. 5 (Value Objects).

## Links

- [[_moc]]
- [[../_moc]]
- [[../patterns/_moc]]
- [[anemic-domain-model]]
- [[long-parameter-list]]
- [[data-clumps]]
