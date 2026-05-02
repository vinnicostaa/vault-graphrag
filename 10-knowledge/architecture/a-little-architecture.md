---
title: A Little Architecture
type: concept
tags: [architecture, dependency-inversion, interface-segregation, uncle-bob]
source: https://blog.cleancoder.com/uncle-bob/2016/01/04/ALittleArchitecture.html
source_date: 2016-01-04
created: 2026-04-24
updated: 2026-04-24
aliases: [A Little Architecture, Uncle Bob Dialogue 2016]
---

# A Little Architecture

Destilado do diálogo socrático de Robert C. Martin (Uncle Bob), 2016. Enquadra o que **realmente** é uma decisão arquitetural — e por que as decisões que muitos atribuem ao arquiteto (DB, framework, webserver) são **irrelevantes**.

## A tese provocativa

> — *"Quero virar arquiteto de software para tomar as decisões importantes sobre DB, framework, webserver."*
>
> — *"Então você não quer virar arquiteto. Essas decisões são irrelevantes."*

**As decisões importantes que um arquiteto toma são as que permitem NÃO decidir sobre DB, framework, webserver — e deferir essas escolhas até quando se tem mais informação.**

## DB é IO device

- Database é "merely an IO device". Fornece ferramentas úteis (sorting, querying, reporting), mas elas são **ancillary**, não intrínsecas às business rules.
- Business rules **podem** usar essas ferramentas; **não devem** depender delas.
- Se a business rule depende da ferramenta, trocar o DB quebra a business rule. **Isto é o problema.**

## A mudança de mentalidade

Em vez de: *"business rules chamam o DB"* (callers dependem de callees).

Pense: *"DB implementa uma interface que business rules definem"* — **Dependency Inversion**.

- High-level policies **não** devem depender de low-level policies.
- Em runtime: `businessRule.execute() → gateway.getSomething()` — chamada sai do interno.
- Em compile-time: source code do gateway depende de source code da business rule — dependência entra.

## Como isso funciona na prática

Exemplo de Uncle Bob (adaptado):

```java
// package businessRules
interface BusinessRuleGateway {
    Something getSomething(String id);
    void startTransaction();
    void saveSomething(Something thing);
    void endTransaction();
}

class BusinessRule {
    private BusinessRuleGateway gateway;
    // ... usa apenas gateway, não conhece DB
}

// package database — IMPORTA businessRules, não o contrário
class MySqlBusinessRuleGateway implements BusinessRuleGateway {
    // ... aqui mora SQL
}
```

Em OO, o sender pode **causar** uma função do receiver sem **mencionar** o tipo do receiver. O source code do receiver **depende** do source code do sender — não o contrário.

## Interface Segregation Principle aplicado

A interface do gateway **não contém todas as ferramentas do DB**. Cada business rule define **sua própria interface** com apenas o que precisa.

- Business rule A precisa de `getX() / saveX()` → interface `AGateway`.
- Business rule B precisa de `getY() / startTx() / endTx()` → interface `BGateway`.
- Uma classe concreta `MySqlGateway` pode implementar ambas.

Resultado: muitas interfaces pequenas, muitas implementações finas. Mais código? Sim. **Esse é o ponto** — é o que permite deferir decisões irrelevantes.

## As bênçãos do arquiteto

- **Woe** o arquiteto que decide DB cedo e descobre que flat files bastariam.
- **Woe** o arquiteto que decide webserver cedo e descobre que socket simples bastaria.
- **Woe** o time cujo arquiteto impõe framework cedo e depois descobre que ele traz constraints inviáveis.

- **Blessed** o time cujo arquiteto provê os meios para **deferir** todas essas decisões até que haja informação suficiente para tomá-las.
- **Blessed** o time que pode criar ambientes de teste rápidos e leves, isolados de IO devices e frameworks.
- **Blessed** o time cujo arquiteto **cuida do que importa** e **difere o que não importa**.

## Resumo prático

| O arquiteto **não** decide | O arquiteto **decide** |
|---|---|
| Qual DB | Qual a interface que business rules expõem pro DB |
| Qual framework | Como frameworks ficam fora do core |
| Qual webserver | Como o core é agnóstico de delivery mechanism |
| Rails vs Spring vs Django | Use cases + entities no centro |

## Fontes

- [Robert C. Martin — A Little Architecture (blog.cleancoder.com, 2016-01-04)](https://blog.cleancoder.com/uncle-bob/2016/01/04/ALittleArchitecture.html)
- Martin, Robert C. (2017). *Clean Architecture*.

## Links

- [[_moc]]
- [[the-clean-architecture]] — DIP é o motor aqui
- [[no-db]] — DB como detalhe diferível
- [[framework-bound]] — manter frameworks a braços de distância
- [[../principles/solid]] — DIP + ISP aplicados
- [[../patterns/adapter]] — implementação do Gateway
