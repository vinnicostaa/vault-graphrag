---
title: Message Chains (AP-019)
type: antipattern
tags: [antipattern, code-smell, coupler, demeter]
source: https://refactoring.guru/smells/message-chains
source_date: 2026-04-24
created: 2026-04-24
updated: 2026-04-24
aliases: [train-wreck, dot-chain, demeter-violation]
---

# AP-019 — Message Chains

Código da forma `a.getB().getC().getD().doE()` — cliente navega uma cadeia de objetos pra chegar no que realmente quer. Também chamado *train wreck*. Viola a **Lei de Deméter** ("fale só com amigos diretos").

## Sintomas

- Cadeias `obj.x().y().z().w()` com 3+ passos
- Testes precisam mockar 4 camadas pra testar 1 comportamento
- Mudança em tipo intermediário quebra N chamadores distantes
- Null-check defensivo em cada passo: `if (a != null && a.b != null && a.b.c != null) ...`
- `obj?.a?.b?.c` espalhado (optional chaining escondendo smell)
- Leitor precisa conhecer estrutura interna de 4 classes

## Por que é problema

**Acoplamento transitivo**: cliente acopla a **todos** os tipos da cadeia — se qualquer intermediário mudar, cliente quebra. Viola Law of Demeter.

**Quebra encapsulamento**: cada objeto da cadeia é forçado a expor seu interno via getter. [[data-class]] rola solto.

**Teste frágil**: stub de 4 níveis é sinal claro de abstração vazando. Teste acopla à estrutura em vez do comportamento.

**Refactor bloqueado**: mudar representação interna de `B` quebra não só `A` mas todo código que fez `a.b.x.y`.

## Exemplo

```ruby
# ❌ Train wreck
tax = order.customer.address.country.tax_rate
discount = order.promo.campaign.rules.first.percentage

# ✅ Hide Delegate — objeto topo expõe intenção, esconde estrutura
tax      = order.tax_rate            # order delega internamente
discount = order.current_discount    # encapsulado

class Order
  def tax_rate;         customer.tax_rate;         end
end
class Customer
  def tax_rate;         address.tax_rate;          end
end
class Address
  def tax_rate;         country.tax_rate;          end
end
# alternativa melhor: computar no domínio que "sabe"
class Order
  def tax_rate
    TaxPolicy.for(self).rate
  end
end
```

## Como refatorar

- **Hide Delegate**: objeto topo expõe método que internamente navega. Cliente fala com um amigo direto.
- **Extract Method**: cadeia repetida em vários lugares vira método único com nome de intenção
- **Move Method** (ver [[feature-envy]]): se muito método cliente está "apaixonado" por objeto do fim da cadeia, mover o método pra lá
- **Introduce Service / Policy**: quando a regra é cross-entity (tax, pricing, discount), extrair objeto de domínio que sabe computar
- **Tell, Don't Ask**: converter "leio dados dos objetos e decido" em "peço pro objeto fazer"

## Quando tolerar

- **Builder fluente**: `Http.post(url).header("x", "y").body(...).send()` — cadeia **é a API** desenhada. Cada passo retorna `self`/`this` por design.
- **LINQ / streams / iterators**: `list.filter(...).map(...).sum()` — cadeia é idioma funcional, não train wreck
- **Navegação em estrutura imutável de config**: `config.getDatabase().getPrimary().getUrl()` em config de inicialização, raro e contido
- **Generated code** (protobuf, GraphQL client): raramente se pode esconder, aceite

Critério: cada link da cadeia **retorna algo novo que conheço em API estável** (builder, stream, DSL) = ok. Cada link **expõe interno de objeto de domínio** = smell.

## Patterns relacionados (que resolvem)

- [[feature-envy]] — smell irmão; muitas vezes o método inteiro deveria mudar de casa
- [[inappropriate-intimacy]] — cadeias longas denunciam intimidade excessiva entre classes
- [[data-class]] — getters públicos alimentam a cadeia
- [[../principles/solid]] — Law of Demeter, DIP
- [[../patterns/facade]] — esconder estrutura interna atrás de API simples
- [[../patterns/mediator]] — centralizar coordenação em vez de navegar grafos

## Fontes

- Refactoring Guru — [Message Chains](https://refactoring.guru/smells/message-chains)
- Fowler, M. (2018). *Refactoring*. 2nd ed. Addison-Wesley.
- Lieberherr, K. & Holland, I. (1989). "Assuring Good Style for Object-Oriented Programs" — Law of Demeter.

## Links

- [[_moc]]
- [[../_moc]]
- [[../patterns/_moc]]
- [[feature-envy]]
- [[inappropriate-intimacy]]
- [[data-class]]
