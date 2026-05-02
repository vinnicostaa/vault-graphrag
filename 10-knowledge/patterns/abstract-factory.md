---
title: Abstract Factory
type: pattern
tags: [pattern, creational, gof, product-families, variants]
source: https://refactoring.guru/design-patterns/abstract-factory
created: 2026-04-24
updated: 2026-04-24
aliases: [Abstract Factory Pattern, Kit]
---

# Abstract Factory

Padrão **criacional** que permite produzir **famílias de objetos relacionados** sem acoplar o código cliente às classes concretas desses objetos.

## Problema que resolve

Você tem **várias famílias** de produtos relacionados (ex.: `Chair` + `Sofa` + `CoffeeTable`) e **várias variantes** da mesma família (ex.: `Modern`, `Victorian`, `ArtDeco`). Objetos da mesma variante precisam **combinar entre si** — um sofá Modern com cadeiras Victorianas é inconsistente.

Duas dificuldades aparecem juntas:

1. Garantir que o cliente monte sempre um conjunto consistente da mesma variante.
2. Permitir adicionar novas variantes (ou novos produtos da família) sem editar o código central que monta os conjuntos.

## Solução

- Declarar **interfaces separadas** para cada produto distinto da família (`Chair`, `Sofa`, `CoffeeTable`) e fazer cada variante concreta implementá-las.
- Declarar uma **Abstract Factory** — interface com um método de criação por produto (`createChair`, `createSofa`, …). Cada método retorna o **tipo abstrato** do produto.
- Para cada variante da família criar uma **Concrete Factory** que implementa todos esses métodos e retorna produtos de uma única variante.
- O cliente recebe uma fábrica já selecionada (normalmente na inicialização) e conversa com ela e com os produtos só pelas interfaces abstratas — nunca pelas classes concretas.

## Estrutura

- **Abstract Products** — interfaces para cada produto distinto da família.
- **Concrete Products** — implementações agrupadas por variante; cada produto abstrato existe em todas as variantes.
- **Abstract Factory** — interface com um método de criação para cada produto abstrato.
- **Concrete Factories** — implementam a Abstract Factory para uma variante específica; instanciam produtos concretos mas assinam retornando produtos abstratos.
- **Client** — trabalha com fábrica e produtos só via interfaces abstratas.

## Pseudocódigo

```
interface Button   is method paint()
interface Checkbox is method paint()

class WinButton   implements Button   is ...
class MacButton   implements Button   is ...
class WinCheckbox implements Checkbox is ...
class MacCheckbox implements Checkbox is ...

interface GUIFactory is
    method createButton(): Button
    method createCheckbox(): Checkbox

class WinFactory implements GUIFactory is
    method createButton(): Button     is return new WinButton()
    method createCheckbox(): Checkbox is return new WinCheckbox()

class MacFactory implements GUIFactory is
    method createButton(): Button     is return new MacButton()
    method createCheckbox(): Checkbox is return new MacCheckbox()

class Application is
    field factory: GUIFactory
    constructor(factory) is this.factory = factory
    method createUI() is this.button = factory.createButton()

// Bootstrap escolhe a fábrica uma única vez:
factory = (config.OS == "Windows") ? new WinFactory() : new MacFactory()
app = new Application(factory)
```

## Quando usar

- Código precisa trabalhar com várias famílias de produtos relacionados e não quer depender das classes concretas — elas podem ser desconhecidas no momento do design ou podem evoluir.
- Os produtos de uma variante precisam ser **consistentes entre si** (ex.: todos no mesmo tema visual, no mesmo protocolo, no mesmo dialeto SQL).
- Uma classe tem um conjunto de [[factory-method]]s que "borra" sua responsabilidade primária — extrair para uma Abstract Factory dedicada.

## Quando evitar

- Há apenas **um** produto (não uma família) — [[factory-method]] basta.
- As variantes não precisam combinar entre si — Abstract Factory introduz acoplamento que não se paga.
- Pouca probabilidade de surgirem novas variantes — o número extra de interfaces e classes pesa mais do que o benefício.

## Sinais de que você está reinventando este pattern

- Objeto "config" / "theme" / "provider" com vários métodos `create*` que retornam tipos abstratos.
- Conjuntos de classes nomeadas com o mesmo prefixo de variante (`Win*`, `Mac*`, `Oracle*`, `Postgres*`) instanciadas em bloco.
- Switch enorme no bootstrap escolhendo qual combinação de implementações instanciar conforme ambiente.

## Trade-offs

### Vantagens
- Garante que os produtos obtidos da mesma fábrica são compatíveis entre si.
- Evita acoplamento forte entre cliente e produtos concretos.
- **Single Responsibility Principle** — criação concentrada nas fábricas.
- **Open/Closed Principle** — nova variante entra com nova Concrete Factory, sem tocar nos clientes.

### Desvantagens
- Muitas interfaces e classes novas — complexidade sobe rápido, especialmente se a família tem muitos produtos.
- Adicionar um **novo produto** à família obriga a mexer na interface Abstract Factory e em **todas** as fábricas concretas.

## Relações com outros patterns

- [[factory-method]] — Abstract Factory é frequentemente implementada como um conjunto de Factory Methods. Começar por Factory Method e evoluir para Abstract Factory é caminho comum.
- [[builder]] — Builder constrói objeto complexo **passo a passo**; Abstract Factory retorna o produto imediatamente. Builder pode produzir coisas que não compartilham interface comum; Abstract Factory garante família coerente.
- [[prototype]] — métodos de uma Abstract Factory podem ser implementados via clone de protótipos em vez de herança.
- [[facade]] — Abstract Factory é alternativa quando o objetivo é só esconder do cliente como o subsistema é criado.
- [[bridge]] — combinar Abstract Factory com Bridge quando certas abstrações do Bridge só funcionam com implementações específicas.
- [[singleton]] — fábricas concretas frequentemente viram Singletons na prática.

## Fontes

- [Refactoring Guru — Abstract Factory](https://refactoring.guru/design-patterns/abstract-factory)
- Gamma, Helm, Johnson, Vlissides (1994). *Design Patterns: Elements of Reusable Object-Oriented Software*. Addison-Wesley.

## Links

- [[_moc]]
- [[creational-patterns]]
- [[../principles/solid]] — OCP aplicado; risco de violar OCP ao adicionar novo produto à família
- [[../_moc]] — knowledge raiz
