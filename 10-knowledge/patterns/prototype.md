---
title: Prototype
type: pattern
tags: [pattern, creational, gof, cloning, configuration-presets]
source: https://refactoring.guru/design-patterns/prototype
created: 2026-04-24
updated: 2026-04-24
aliases: [Clone, Prototype Pattern]
---

# Prototype

Padrão **criacional** que permite **copiar objetos existentes** sem que o código dependa das suas classes concretas, delegando a operação de cópia aos próprios objetos.

## Problema que resolve

Copiar um objeto "por fora" — instanciando a classe e iterando pelos campos — falha por duas razões:

1. **Encapsulamento** — parte dos campos é privada e inacessível de fora, então a cópia fica incompleta.
2. **Acoplamento à classe concreta** — quem copia precisa conhecer o tipo exato; se o código só vê uma interface comum (ex.: parâmetro polimórfico), não dá para chamar o construtor certo.

Um segundo problema correlato: quando um objeto tem muitas configurações possíveis, é comum criar subclasses só para fixar cada configuração — o resultado é uma hierarquia inflada de subclasses "vazias".

## Solução

Declarar uma interface comum (normalmente com um único método `clone()`) que **cada classe que suporta cópia implementa**. A implementação tipicamente invoca um construtor de cópia da própria classe, que acessa os campos privados diretamente (possível porque é o mesmo tipo) e carrega tudo para a nova instância.

Como o cliente só chama `clone()` pela interface, deixa de depender do tipo concreto. Para configurações pré-prontas, um **prototype registry** (map `name → prototype`) guarda instâncias modelo que são clonadas sob demanda — eliminando subclasses só-para-configurar.

## Estrutura

- **Prototype** (interface) — declara o método `clone()`.
- **Concrete Prototype** — implementa `clone()`, tipicamente delegando a um construtor de cópia que carrega todos os campos (inclusive privados herdados, via chamada ao construtor de cópia da superclasse).
- **Client** — obtém cópias chamando `clone()` pela interface; não conhece a classe concreta.
- **Prototype Registry** (opcional) — catálogo de protótipos pré-configurados acessíveis por chave.

## Pseudocódigo

```
abstract class Shape is
    field X, Y: int
    field color: string

    constructor Shape()       is ...
    constructor Shape(src: Shape) is      // construtor de cópia
        this.X = src.X
        this.Y = src.Y
        this.color = src.color

    abstract method clone(): Shape

class Rectangle extends Shape is
    field width, height: int
    constructor Rectangle(src: Rectangle) is
        super(src)                         // copia campos privados do pai
        this.width  = src.width
        this.height = src.height
    method clone(): Shape is return new Rectangle(this)

class Circle extends Shape is
    field radius: int
    constructor Circle(src: Circle) is
        super(src)
        this.radius = src.radius
    method clone(): Shape is return new Circle(this)

// Cliente: trabalha com Shape sem saber o tipo concreto.
foreach (s in shapes) do
    copies.add(s.clone())
```

## Quando usar

- O código não deve depender das classes concretas dos objetos que precisa copiar — típico quando os objetos vêm de código third-party via interface.
- Para reduzir subclasses cujo único propósito é fixar uma configuração: um conjunto de protótipos pré-configurados pode substituir a hierarquia.
- Para guardar snapshots no histórico de [[command]]s, ou clonar rapidamente estruturas complexas de [[composite]] / [[decorator]] em vez de reconstruí-las do zero.

## Quando evitar

- Objetos com **referências circulares** complicam a cópia profunda — shallow copy pode surpreender, deep copy exige cuidado explícito.
- Objeto guarda recursos externos (sockets, file handles, conexões) que não faz sentido duplicar — clone sem política clara vira bug.
- Linguagens com clone "mágico" profundo (ex.: serialização) podem levar a cópias silenciosamente errôneas — o pattern exige `clone()` **explícito** com semântica definida.

## Sinais de que você está reinventando este pattern

- Função "copy" isolada que conhece cada tipo e faz `switch` sobre ele para instanciar a classe certa.
- Subclasses que existem só para sobrescrever valores default no construtor, sem adicionar comportamento.
- Serialização + desserialização usada apenas para duplicar objeto em memória.

## Trade-offs

### Vantagens
- Cópia sem acoplamento às classes concretas.
- Elimina código repetido de inicialização favorecendo clonagem de protótipos pré-prontos.
- Alternativa à herança para presets de configuração de objetos complexos.
- Permite construir objetos complexos mais convenientemente.

### Desvantagens
- Clonar objetos com referências circulares é delicado e pode exigir lógica customizada.
- Semântica shallow vs. deep copy precisa ser decidida e documentada por classe — erro silencioso é comum.

## Relações com outros patterns

- [[factory-method]] — alternativa baseada em herança; não exige passo de inicialização do clone, mas acopla a uma hierarquia.
- [[abstract-factory]] — seus métodos podem ser implementados clonando protótipos em vez de instanciar classes concretas.
- [[builder]] — Builder constrói passo a passo; Prototype copia um objeto já construído e configurado.
- [[composite]], [[decorator]] — designs pesados nesses patterns se beneficiam de Prototype para clonar estruturas inteiras.
- [[command]] — Prototype ajuda a salvar cópias de Commands num histórico.
- [[memento]] — Prototype pode ser alternativa mais simples quando o estado a preservar é direto e sem links externos delicados.
- [[singleton]] — um registry de protótipos é tipicamente implementado como Singleton.

## Fontes

- [Refactoring Guru — Prototype](https://refactoring.guru/design-patterns/prototype)
- Gamma, Helm, Johnson, Vlissides (1994). *Design Patterns: Elements of Reusable Object-Oriented Software*. Addison-Wesley.

## Links

- [[_moc]]
- [[creational-patterns]]
- [[../principles/solid]] — alternativa a herança excessiva (OCP sem proliferação de subclasses)
- [[../_moc]] — knowledge raiz
