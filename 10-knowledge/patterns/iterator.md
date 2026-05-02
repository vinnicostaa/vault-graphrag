---
title: Iterator
type: pattern
tags: [pattern, behavioral, gof, traversal, collection]
source: https://refactoring.guru/design-patterns/iterator
created: 2026-04-24
updated: 2026-04-24
aliases: [Iterator Pattern, Cursor]
---

# Iterator

Padrão **comportamental** que permite percorrer elementos de uma coleção **sem expor sua representação interna** (list, stack, tree, grafo, etc.).

## Problema que resolve

Coleções guardam elementos em estruturas variadas — lista, árvore, grafo, hash. Cada estrutura demanda algoritmos de traversal diferentes (depth-first, breadth-first, ordem de inserção, acesso aleatório).

Espalhar esses algoritmos dentro da classe de coleção infla sua responsabilidade (armazenar vs. percorrer) e polui a API. Pior: o cliente fica acoplado à estrutura concreta, pois o modo de iterar depende do tipo.

Também não dá para ter **várias travessias simultâneas independentes** na mesma coleção se a coleção guarda o estado da iteração em si mesma.

## Solução

Extrair a travessia para um objeto **Iterator** separado, que encapsula a lógica do algoritmo e o estado da iteração (posição, cache, cursor). A coleção apenas expõe uma factory (`createIterator()`) que devolve o iterator apropriado.

Todos os iterators implementam a mesma interface (tipicamente `hasNext()` / `next()`), então o cliente trabalha com qualquer coleção ou estratégia de travessia sem mudar seu código. Para novo algoritmo de travessia, basta nova classe Iterator.

## Estrutura

- **Iterator** (interface) — declara `next()`, `hasNext()` (e opcionalmente `reset()`, `current()`).
- **Concrete Iterator** — implementa o algoritmo de travessia; mantém posição/cursor.
- **Collection** (interface) — declara factory method(s) para criar iterators.
- **Concrete Collection** — retorna nova instância do iterator apropriado.
- **Client** — trabalha com ambos via interfaces.

## Pseudocódigo

```
interface Iterator<T> is
    method hasNext(): bool
    method next(): T

interface Collection<T> is
    method createIterator(): Iterator<T>

class Tree<T> implements Collection<T> is
    method createDepthFirstIterator() is return new DFSIterator(this)
    method createBreadthFirstIterator() is return new BFSIterator(this)

class DFSIterator<T> implements Iterator<T> is
    private stack
    constructor(tree) is stack.push(tree.root)
    method hasNext() is return !stack.empty()
    method next() is
        node = stack.pop()
        foreach (c in reverse(node.children)) stack.push(c)
        return node.value

// Client:
it = tree.createDepthFirstIterator()
while (it.hasNext()) process(it.next())
```

## Quando usar

- Quer esconder complexidade de estrutura interna (árvore, grafo, hash) do cliente.
- Precisa oferecer **múltiplas formas** de percorrer a mesma coleção.
- Várias iterações simultâneas e independentes sobre a mesma coleção.
- Reduzir duplicação de código de travessia espalhado pela aplicação.

## Quando evitar

- Coleções triviais baseadas em array/list — o loop direto é mais simples e eficiente.
- Linguagens com generators/yield já oferecem iteração sem classe dedicada.
- Quando acesso aleatório (índice) é o uso principal — Iterator é sequencial.

## Sinais de que você está reinventando este pattern

- Cliente navega diretamente em `node.children[i].next` misturando lógica de travessia com trabalho.
- Métodos `firstX/nextX` no objeto coleção que guardam cursor interno — quebra multi-iteração.
- Código de travessia duplicado em vários pontos da aplicação.

## Trade-offs

### Vantagens
- **SRP** — separa armazenamento de travessia.
- **OCP** — novo algoritmo = novo iterator, sem alterar coleção nem cliente.
- Múltiplas iterações paralelas independentes, cada uma com seu estado.
- Pausa/retomada de iteração é natural.

### Desvantagens
- Overkill em coleções simples.
- Iterator pode ser menos eficiente que acesso direto especializado (cache miss, wrapping).
- Coleção mutada durante iteração leva a estados inconsistentes (fail-fast vs. snapshot).

## Relações com outros patterns

- [[composite]] — Iterators percorrem árvores Composite.
- [[factory-method]] — frequentemente usado para criar o iterator certo para cada coleção.
- [[memento]] — captura estado da iteração para rollback posterior.
- [[visitor]] — Visitor percorre a estrutura frequentemente via Iterator, aplicando operações em cada elemento.

## Fontes

- [Refactoring Guru — Iterator](https://refactoring.guru/design-patterns/iterator)
- Gamma, Helm, Johnson, Vlissides (1994). *Design Patterns: Elements of Reusable Object-Oriented Software*. Addison-Wesley.

## Links

- [[_moc]]
- [[behavioral-patterns]]
- [[../principles/solid]] — SRP/OCP aplicados por este pattern
- [[../_moc]] — knowledge raiz
