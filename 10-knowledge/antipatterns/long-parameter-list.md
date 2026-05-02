---
title: Long Parameter List (AP-004)
type: antipattern
tags: [antipattern, code-smell, bloater, api-design]
source: "https://refactoring.guru/smells/long-parameter-list"
source_date: 2026-04-24
created: 2026-04-24
updated: 2026-04-24
aliases: [too-many-params, parameter-explosion]
---

# AP-004 — Long Parameter List

Função ou construtor que recebe muitos parâmetros (4+ já é suspeito; 6+ é claro sinal). Indica acoplamento excessivo, ausência de agrupamento conceitual ou função que faz mais de uma coisa.

## Sintomas

- Assinatura quebra em múltiplas linhas
- Parâmetros relacionados (city, street, zip) soltos em vez de agrupados
- Booleans de configuração: `create(name, active, persistent, cached, lazy)` — cada bool esconde um modo distinto
- Callers montam lista com `nil`, `""`, `0` como placeholder
- Adicionar funcionalidade = mais um parâmetro, nunca menos
- Testes repetem o mesmo bloco de argumentos com 1-2 variações

## Por que é problema

**Legibilidade no call site**: `charge(u, 100, "USD", true, false, nil, ctx)` — quem lê não sabe o que cada literal significa sem olhar a assinatura.

**Ordem importa demais**: trocar dois params do mesmo tipo (dois `string`) compila e quebra em runtime.

**Assinaturas instáveis**: qualquer campo novo é breaking change.

**Falta abstração**: parâmetros que andam juntos (`lat, lng`, `startDate, endDate`) revelam um conceito ausente (`Coordinate`, `DateRange`).

## Exemplo

```typescript
// ❌ 9 parâmetros
function createUser(
  firstName: string,
  lastName: string,
  email: string,
  birthDate: Date,
  addressStreet: string,
  addressCity: string,
  addressZip: string,
  isAdmin: boolean,
  isActive: boolean,
) { ... }

// ✅ agrupado + builder
type UserInput = {
  name: FullName;
  email: Email;
  birthDate: Date;
  address: Address;
  flags: UserFlags;
};
function createUser(input: UserInput) { ... }
```

## Como refatorar

- **Introduce Parameter Object**: agrupar params correlatos em struct/record
- **Preserve Whole Object**: se chamador extrai 5 campos de `user` e passa separados, passar `user` inteiro
- **Builder** (ver [[../patterns/builder]]): construção passo a passo nomeada para objetos complexos
- **Named arguments / destructuring** quando a linguagem permite (Python, Kotlin, JS/TS)
- **Replace Parameter With Explicit Methods**: bool `isAdmin` → dois métodos `createAdmin()` e `createUser()`

## Quando tolerar

- **Funções matemáticas puras** com 4-5 parâmetros de mesmo papel (`clamp(value, min, max)`)
- **Entry points de framework** (controllers HTTP que recebem ctx, req, res)
- **Construtores de Value Object** imutáveis com 3-4 campos coesos

## Patterns relacionados (que resolvem)

- [[../patterns/builder]] — construção fluente
- [[../patterns/factory-method]] — factory recebendo um input object
- [[primitive-obsession]] — muitos parâmetros primitivos viram um VO
- [[data-clumps]] — params que andam juntos são data clump disfarçado

## Fontes

- Refactoring Guru — [Long Parameter List](https://refactoring.guru/smells/long-parameter-list)
- Fowler, M. (2018). *Refactoring: Improving the Design of Existing Code*. 2nd ed. Addison-Wesley.
- Martin, R. C. (2008). *Clean Code*. Cap. 3 — "ideal number of arguments is zero; three is max".

## Links

- [[_moc]]
- [[../_moc]]
- [[../patterns/_moc]]
- [[../patterns/builder]]
- [[data-clumps]]
- [[primitive-obsession]]
