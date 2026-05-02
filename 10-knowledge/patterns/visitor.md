---
title: Visitor
type: pattern
tags: [pattern, behavioral, gof, double-dispatch, traversal]
source: https://refactoring.guru/design-patterns/visitor
created: 2026-04-24
updated: 2026-04-24
aliases: [Visitor Pattern]
---

# Visitor

Padrão **comportamental** que permite **separar algoritmos dos objetos** sobre os quais eles operam, movendo novos comportamentos para classes visitor externas.

## Problema que resolve

Dada uma hierarquia estável de classes (nós de um grafo, elementos de uma AST, formas geométricas), surgem continuamente novas operações sobre elas (export XML, export JSON, cálculo de métricas, validação). Adicionar cada operação como método na hierarquia:

1. **Polui** as classes com responsabilidades alheias (um `Node` não deveria saber exportar XML).
2. **Obriga** alterar código testado e em produção toda vez que uma operação nova aparece.
3. **Propaga** a mesma mudança por dezenas de subclasses.

Tentar resolver com `if (instanceof X)` quebra polimorfismo, espalha condicionais e esquece classes quando a hierarquia muda.

## Solução

Extrair o novo comportamento para um **Visitor** com um método por tipo concreto da hierarquia (`visitCircle`, `visitRectangle`, ...). Como o compilador não consegue escolher o método certo pelo tipo dinâmico (method overloading resolve estaticamente), usa-se **double dispatch**: cada elemento implementa `accept(visitor)` que chama `visitor.visitX(this)`, revelando seu tipo real.

Novas operações viram novos visitors — zero mudança nas classes de elementos. O preço é adicionar `accept()` uma única vez na hierarquia.

## Estrutura

- **Visitor** (interface) — declara `visitX(X)`, `visitY(Y)`, ... um por tipo concreto.
- **Concrete Visitor** — implementa uma versão do algoritmo para cada tipo.
- **Element** (interface) — declara `accept(visitor)`.
- **Concrete Element** — implementa `accept` chamando `visitor.visitX(this)`.
- **Client** — percorre estrutura e chama `accept` em cada elemento.

## Pseudocódigo

```
interface Shape is
    method accept(v: Visitor)

class Circle implements Shape is
    method accept(v) is v.visitCircle(this)

class Rectangle implements Shape is
    method accept(v) is v.visitRectangle(this)

interface Visitor is
    method visitCircle(c: Circle)
    method visitRectangle(r: Rectangle)

class XMLExportVisitor implements Visitor is
    method visitCircle(c) is emit("<circle r=" + c.radius + "/>")
    method visitRectangle(r) is emit("<rect w=" + r.w + "/>")

class AreaVisitor implements Visitor is
    method visitCircle(c) is total += PI * c.radius^2
    method visitRectangle(r) is total += r.w * r.h

// Client:
foreach (shape in shapes) shape.accept(new XMLExportVisitor())
```

## Quando usar

- Precisa aplicar uma operação a todos os elementos de uma estrutura heterogênea (árvore Composite, AST, grafo).
- Hierarquia de elementos é **estável**, mas operações sobre ela mudam/crescem.
- Quer manter a lógica de negócio dos elementos limpa, extraindo comportamentos auxiliares.
- Operação só faz sentido para alguns tipos da hierarquia — visitors implementam apenas os métodos necessários.

## Quando evitar

- Hierarquia de elementos muda frequentemente — cada mudança força atualizar **todos** os visitors existentes.
- Só há uma operação natural sobre a hierarquia — polimorfismo direto é suficiente.
- Operação precisa acessar estado privado dos elementos — getters forçados violam encapsulamento.
- Linguagem não tem overloading ou é puramente funcional — padrões alternativos (pattern matching, typeclasses) são mais naturais.

## Sinais de que você está reinventando este pattern

- Cadeia de `if (x instanceof TypeA) ... else if (x instanceof TypeB) ...`.
- Método de domínio sendo adicionado repetidas vezes só para suportar uma operação técnica (export, logging).
- Função externa que faz downcast manual pra cada tipo.

## Trade-offs

### Vantagens
- **OCP** — nova operação = novo visitor, sem tocar nos elementos.
- **SRP** — operações auxiliares ficam em visitors, elementos focam no essencial.
- Visitor acumula estado ao percorrer — bom para agregações (soma, contagem, relatório).

### Desvantagens
- Adicionar novo **tipo de elemento** exige atualizar todos os visitors existentes.
- Visitor pode precisar acessar estado privado — pode forçar vazamento de encapsulamento.
- Double dispatch deixa fluxo menos óbvio para quem lê.
- Alto boilerplate em hierarquias grandes.

## Relações com outros patterns

- [[command]] — Visitor pode ser visto como Command poderoso, capaz de operar sobre objetos de classes diferentes.
- [[composite]] — Visitor aplica operação sobre árvore Composite inteira.
- [[iterator]] — combina-se com Iterator para percorrer estrutura complexa antes de aplicar o visitor a cada elemento.

## Fontes

- [Refactoring Guru — Visitor](https://refactoring.guru/design-patterns/visitor)
- Gamma, Helm, Johnson, Vlissides (1994). *Design Patterns: Elements of Reusable Object-Oriented Software*. Addison-Wesley.

## Links

- [[_moc]]
- [[behavioral-patterns]]
- [[../principles/solid]] — OCP e SRP aplicados por este pattern
- [[../_moc]] — knowledge raiz
