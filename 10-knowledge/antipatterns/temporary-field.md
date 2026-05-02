---
title: Temporary Field (AP-013)
type: antipattern
tags: [antipattern, code-smell, oo-abuser, state]
source: "https://refactoring.guru/smells/temporary-field"
source_date: 2026-04-24
created: 2026-04-24
updated: 2026-04-24
aliases: [optional-field, sometimes-field]
---

# AP-013 — Temporary Field

Campo de uma classe que é usado **só em certas circunstâncias** — fica `null`/`default` a maior parte do tempo e só é preenchido em cenários específicos. Leitor precisa entender o ciclo de vida invisível para saber quando o campo é válido.

## Sintomas

- Campos `@Nullable` verificados com `if (x != null)` em toda leitura
- Código lê um campo e a regra "foi setado só se método Y rodou antes"
- Construtor inicializa metade dos campos como `null`/vazio
- Testes precisam chamar sequência específica de métodos para popular o estado
- Métodos começam com "set this, then compute, then read" — ordem frágil

## Por que é problema

**Invariantes implícitas**: a validade do campo depende de estado oculto. Usar o objeto "errado" (fora da ordem esperada) produz `NullPointerException` ou resultado incorreto.

**Baixa coesão**: o campo só pertence à classe em subset de operações — é um *subtipo* escondido.

**Teste difícil**: mockar o ciclo completo é necessário para testar a operação que usa o campo.

**Acoplamento temporal**: chamador precisa conhecer ordem de chamadas. Violação da *command-query separation* e do encapsulamento.

## Exemplo

```java
// ❌ campo temporário
class RouteCalculator {
    private Graph graph;
    private Node source;
    private Map<Node, Distance> distances;   // só setado após calculate()
    private List<Node> visited;              // só setado após calculate()

    public void calculate(Node from) {
        this.source = from;
        this.distances = new HashMap<>();
        this.visited  = new ArrayList<>();
        // ... dijkstra
    }
    public Distance distanceTo(Node n) { return distances.get(n); } // boom se calculate não rodou
}

// ✅ extrair estado transitório para objeto próprio
class RouteCalculator {
    private final Graph graph;
    public Route calculate(Node from) {
        return new DijkstraRun(graph, from).execute(); // Route imutável
    }
}
class Route { private Map<Node, Distance> distances; public Distance to(Node n) { ... } }
```

## Como refatorar

- **Extract Class**: campos temporários + métodos que os usam viram classe própria com ciclo de vida claro
- **Introduce Null Object**: quando campo é usado "se existir", criar objeto nulo com comportamento padrão
- **Replace Method with Method Object**: quando o campo só existe durante execução de um método complexo, transformar em objeto efêmero
- **Preserve Whole Object**: passar o contexto de cálculo completo como parâmetro em vez de campo

## Quando tolerar

- **Cache / memoization** preenchido lazily — temporário **por design**, semântica documentada
- **Builder pattern** — campos parciais durante construção são esperados
- **State machine pattern** explícito onde campos fazem sentido só em certos estados (preferir objeto State separado — ver [[../patterns/state]])

## Patterns relacionados (que resolvem)

- Null Object pattern (refactoring clássico de Fowler)
- [[../patterns/state]] — cada estado tem os campos que precisa
- [[../patterns/memento]] — estado transitório encapsulado
- [[../patterns/builder]] — construção passo-a-passo legítima
- [[../principles/solid]] — SRP / encapsulamento

## Fontes

- Refactoring Guru — [Temporary Field](https://refactoring.guru/smells/temporary-field)
- Fowler, M. (2018). *Refactoring: Improving the Design of Existing Code*. 2nd ed. Addison-Wesley.
- Beck, K. (1997). *Smalltalk Best Practice Patterns* — "don't create optional state".

## Links

- [[_moc]]
- [[../_moc]]
- [[../patterns/_moc]]
- [[../patterns/state]]
- [[../patterns/builder]]
