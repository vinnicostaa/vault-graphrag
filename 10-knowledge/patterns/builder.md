---
title: Builder
type: pattern
tags: [pattern, creational, gof, step-by-step, telescoping-constructor]
source: https://refactoring.guru/design-patterns/builder
created: 2026-04-24
updated: 2026-04-24
aliases: [Builder Pattern]
---

# Builder

Padrão **criacional** que permite construir objetos complexos **passo a passo**, reaproveitando o mesmo código de construção para produzir representações diferentes do objeto.

## Problema que resolve

Objetos com muitos campos opcionais e configurações possíveis geram dois sintomas ruins:

1. **Explosão de subclasses** — uma subclasse por combinação de parâmetros rapidamente vira dezenas de classes quase idênticas.
2. **Telescoping constructor** — construtor único com longa lista de parâmetros opcionais, chamado quase sempre com a maioria deles sem uso, em diferentes ordens e combinações. Cria ruído e propensão a erro.

Nenhum dos dois escala bem quando novas opções aparecem.

## Solução

Extrair a lógica de construção para um objeto separado — o **builder** — que expõe **passos nomeados** (`buildWalls`, `setEngine`, `setGPS`, …). O cliente chama só os passos que interessam, na ordem que fizer sentido, e por fim pede o produto pronto.

Quando é preciso produzir **representações diferentes** do produto com a mesma sequência de passos, cria-se mais de um *concrete builder* implementando a mesma interface de passos. Um **director** opcional encapsula sequências de passos reutilizáveis, escondendo a ordem de construção do cliente.

O builder não expõe o produto pela metade — só depois de `getProduct()`. Isso evita cliente pegar objeto inconsistente.

## Estrutura

- **Builder** (interface) — declara os passos de construção comuns.
- **Concrete Builders** — implementam os passos; podem produzir produtos de tipos diferentes, inclusive sem interface comum.
- **Product** — objeto resultante; não precisa pertencer a uma hierarquia única.
- **Director** (opcional) — define ordens de execução dos passos para produzir configurações padronizadas e reutilizáveis.
- **Client** — associa um builder a um director (ou controla o builder diretamente) e obtém o produto final do builder.

## Pseudocódigo

```
interface Builder is
    method reset()
    method setSeats(n)
    method setEngine(e)
    method setGPS(enabled)

class CarBuilder implements Builder is
    private field car: Car
    constructor() is this.reset()
    method reset() is this.car = new Car()
    method setSeats(n) is ...
    method setEngine(e) is ...
    method setGPS(enabled) is ...
    method getProduct(): Car is
        product = this.car
        this.reset()
        return product

class CarManualBuilder implements Builder is
    // mesmos passos, porém produzindo um Manual em vez de Car

class Director is
    method constructSportsCar(b: Builder) is
        b.reset()
        b.setSeats(2)
        b.setEngine(new SportEngine())
        b.setGPS(true)

// Cliente:
dir = new Director()
cb  = new CarBuilder()
dir.constructSportsCar(cb)
car = cb.getProduct()
```

## Quando usar

- Para eliminar um *telescoping constructor* com muitos parâmetros opcionais.
- Quando o mesmo processo de construção precisa produzir **representações diferentes** do produto (ex.: objeto de domínio **e** seu manual / DTO / schema).
- Para construir árvores do tipo [[composite]] ou outras estruturas complexas — passos podem ser adiados ou recursivos, e o produto incompleto não vaza para o cliente.

## Quando evitar

- Objeto é simples e tem poucos campos — construtor direto é mais curto e claro.
- Linguagens com **named arguments** ou **struct literals** cobrem o caso de muitos opcionais sem nova hierarquia (avaliar antes de introduzir builder).
- Um único concrete builder sem director, usado uma vez — frequentemente vira cerimônia sem ganho.

## Sinais de que você está reinventando este pattern

- Construtor com 8+ parâmetros, a maioria opcional, e chamadas com longas sequências de `null` / valores default.
- Proliferação de subclasses cujo único objetivo é fixar um conjunto de valores de campos.
- Método `with*` encadeado (fluent interface) acumulando estado antes de um `build()` final — é builder "escondido".

## Trade-offs

### Vantagens
- Construção passo a passo, com passos adiáveis ou recursivos.
- Reaproveita código de construção para variações do produto.
- **Single Responsibility Principle** — separa lógica de construção da lógica de negócio do produto.
- Cliente não vê produto incompleto.

### Desvantagens
- Mais classes e indireção.
- Vale a pena só quando o objeto realmente é complexo ou quando há múltiplas representações — caso contrário é cerimônia.

## Relações com outros patterns

- [[factory-method]] — muitos designs começam com Factory Method e evoluem para Builder quando a construção fica complexa demais para uma chamada única.
- [[abstract-factory]] — Abstract Factory retorna o produto imediatamente; Builder permite passos adicionais antes de entregar. Abstract Factory garante **família coerente**; Builder foca em **um objeto complexo**.
- [[composite]] — Builder é útil para montar árvores Composite com passos recursivos, sem expor a árvore pela metade.
- [[bridge]] — Builder pode combinar com Bridge: director no papel de abstração, concrete builders como implementações.
- [[singleton]] — builders são frequentemente implementados como Singleton na prática.
- [[prototype]] — alternativa quando a construção passo a passo não agrega e basta clonar uma configuração pré-pronta.

## Fontes

- [Refactoring Guru — Builder](https://refactoring.guru/design-patterns/builder)
- Gamma, Helm, Johnson, Vlissides (1994). *Design Patterns: Elements of Reusable Object-Oriented Software*. Addison-Wesley.

## Links

- [[_moc]]
- [[creational-patterns]]
- [[../principles/solid]] — SRP aplicado por este pattern
- [[../_moc]] — knowledge raiz
