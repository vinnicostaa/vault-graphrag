---
title: Composite
type: pattern
tags: [pattern, structural, gof, tree, recursion]
source: https://refactoring.guru/design-patterns/composite
created: 2026-04-24
updated: 2026-04-24
aliases: [Composite Pattern, Object Tree]
---

# Composite

Padrão **estrutural** que permite compor objetos em **estruturas de árvore** e tratar tanto elementos simples (folhas) quanto agregadores (containers) através da **mesma interface**.

## Problema que resolve

Quando o modelo de domínio é naturalmente hierárquico — produtos dentro de caixas dentro de caixas; diretórios dentro de diretórios; grupos dentro de grupos; componentes de UI — calcular algo sobre o todo (preço, tamanho, render) exige saber de antemão se cada nó é folha ou nó interno, e descer recursivamente manual em cada caso.

O cliente acaba cheio de `if (é caixa) então iterar senão somar`, acoplado às classes concretas da árvore e incapaz de aceitar novos tipos de nó sem mudanças.

## Solução

Declarar uma **interface comum** para folhas e containers. Cada folha implementa a operação diretamente. Cada container implementa a operação **delegando às filhas** pela mesma interface e "somando" os resultados — as filhas podem ser folhas ou outros containers, e o container não precisa saber.

O cliente fala apenas com a interface comum. Ele empurra uma requisição para a raiz; a estrutura passa a chamada árvore adentro recursivamente. Novos tipos de folha/container podem ser adicionados sem alterar o cliente.

## Estrutura

- **Component** — interface comum que declara operações válidas para folhas e containers.
- **Leaf** — elemento terminal da árvore; executa o trabalho real.
- **Container (Composite)** — nó interno que contém filhos (folhas ou outros containers) via interface `Component`; delega e agrega resultados.
- **Client** — interage com a árvore inteira através da interface `Component`, sem distinguir folhas de containers.

## Pseudocódigo

```
interface Graphic is
    method move(x, y)
    method draw()

class Dot implements Graphic is
    field x, y
    method move(dx, dy) is this.x += dx; this.y += dy
    method draw() is /* desenhar ponto */

class Circle extends Dot is
    field radius
    method draw() is /* desenhar círculo */

class CompoundGraphic implements Graphic is
    field children: array of Graphic
    method add(child) is children.append(child)
    method remove(child) is children.remove(child)
    method move(dx, dy) is
        foreach child in children do child.move(dx, dy)
    method draw() is
        foreach child in children do child.draw()

// Uso: cliente trata um grupo como se fosse um Graphic
group = new CompoundGraphic()
group.add(new Dot(1, 2))
group.add(new Circle(5, 3, 10))
group.move(10, 10)
group.draw()
```

## Quando usar

- O modelo do domínio é uma **árvore** (organizacional, geométrica, de arquivos, de componentes UI, de expressões).
- O cliente deveria tratar folhas e compostos **uniformemente**.
- Operações como "calcular total", "renderizar", "validar" fazem sentido em ambos os níveis e podem ser somadas recursivamente.

## Quando evitar

- Estrutura é plana (lista, conjunto) — Composite só adiciona complexidade.
- Folhas e containers têm comportamento **muito diferente**; forçar uma interface comum obriga a métodos "vazios" nas folhas e vira fonte de confusão.
- Operações variam drasticamente entre tipos de nó — talvez [[visitor]] seja melhor.

## Sinais de que você está reinventando este pattern

- Funções recursivas que recebem `union(Folha, Container)` e usam `switch` no tipo.
- Classes "grupo" que reimplementam manualmente `forEach` para cada operação agregada.
- Cálculos de "total" que descem a árvore com laços aninhados no cliente.

## Trade-offs

### Vantagens
- Trabalhar com árvores complexas fica mais conveniente — polimorfismo + recursão fazem o trabalho.
- *Open/Closed Principle* — novos tipos de folha/container não quebram o cliente.
- Cliente desacoplado da estrutura concreta.

### Desvantagens
- Interface comum pode ficar genérica demais quando folhas e containers divergem muito — prejudica legibilidade e viola *Interface Segregation*.
- Métodos de adição/remoção no `Component` fazem sentido pro container e ficam vazios na folha; declarar só no container preserva ISP mas obriga o cliente a distinguir.

## Relações com outros patterns

- [[builder]] — útil para construir árvores Composite recursivamente.
- [[chain-of-responsibility]] — pode propagar uma requisição da folha até a raiz da árvore.
- [[iterator]] — percorre árvores Composite.
- [[visitor]] — executa operações sobre a árvore inteira sem modificar as classes de nó.
- [[flyweight]] — folhas compartilhadas da árvore podem ser Flyweights para economizar memória.
- [[decorator]] — estruturalmente similar (composição recursiva); diferença: Decorator tem **um** filho e adiciona responsabilidade, Composite tem **vários** e apenas agrega.
- [[prototype]] — clonagem eficiente de árvores Composite/Decorator grandes.

## Fontes

- [Refactoring Guru — Composite](https://refactoring.guru/design-patterns/composite)
- Gamma, Helm, Johnson, Vlissides (1994). *Design Patterns: Elements of Reusable Object-Oriented Software*. Addison-Wesley.

## Links

- [[_moc]]
- [[structural-patterns]]
- [[decorator]] — estrutura recursiva similar
- [[../_moc]] — knowledge raiz
