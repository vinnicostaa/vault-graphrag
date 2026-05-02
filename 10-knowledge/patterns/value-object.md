---
title: Value Object
type: pattern
tags:
  - pattern
  - ddd
  - tactical-design
aliases:
  - VO
  - Value Object Pattern
created: '2026-04-27'
updated: '2026-04-27'
---

# Value Object

Pattern tático de DDD (Evans 2003). Objeto imutável definido por **valor** (não por identidade). Dois VOs com os mesmos campos são iguais; dois Entity com os mesmos campos podem ser diferentes (têm IDs distintos).

## Quando usar

- Conceito de domínio sem identidade própria: `Money(amount, currency)`, `Email(address)`, `CPF(digits)`, `DateRange(start, end)`, `Address(street, city, zip)`.
- Encapsular invariantes que pertencem ao tipo: `Email.parse()` valida formato; `Money.add()` rejeita moedas diferentes; `CPF.New()` rejeita dígitos inválidos.
- Substituir **primitive obsession** ([[../antipatterns/primitive-obsession]]) — em vez de `string` cru, tipo `Email`.

## Quando evitar

- Quando o conceito tem identidade que persiste mesmo se atributos mudam (ex.: usuário muda de email mas continua o mesmo usuário → `User` é Entity, não VO).
- Quando o objeto vai mudar com frequência — VOs são **imutáveis** por contrato; mutação significa criar novo VO.

## Sinais de reinvenção (já é VO sem nome)

- Struct/record com construtor que valida e nunca expõe setters.
- Métodos de comparação por valor (`Equals`/`==` baseado em campos).
- Tudo que se cria em factory `NewX(...)` que retorna erro em entrada inválida.

## Trade-offs

**Vantagens**: encapsula invariantes; comparação por valor reduz bugs de identidade; tipos significativos no lugar de primitives; trivialmente thread-safe (imutáveis).

**Desvantagens**: pode parecer overhead em domínios simples; serialização/desserialização exige boilerplate em algumas linguagens.

## Relações

- Encapsulamento natural quando aplicado com [[../principles/clean-code]] (nomes significativos) e [[../principles/solid]] (SRP — cada VO faz uma coisa).
- Resolve [[../antipatterns/primitive-obsession]] e [[../antipatterns/data-clumps]].
- Complemento de [[../architecture/ddd-strategic-tactical]] §táticos (Entity vs VO vs Aggregate).
- Em Go: structs sem mutators + factory `New*`. Em TS: classes ou objetos `readonly`. Em Rust: struct com `#[derive(PartialEq, Eq)]`.

## Fontes

- Evans, *Domain-Driven Design* (2003), cap. 5 ("A Model Expressed in Software").
- Vernon, *Implementing DDD* (2013), cap. 6.
- [Martin Fowler — ValueObject](https://martinfowler.com/bliki/ValueObject.html)

## Links

- [[_moc]]
- [[../architecture/ddd-strategic-tactical]]
- [[../antipatterns/primitive-obsession]]
- [[../antipatterns/data-clumps]]
