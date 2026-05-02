---
title: State
type: pattern
tags: [pattern, behavioral, gof, fsm, state-machine]
source: https://refactoring.guru/design-patterns/state
created: 2026-04-24
updated: 2026-04-24
aliases: [State Pattern]
---

# State

Padrão **comportamental** que permite que um objeto **altere seu comportamento quando seu estado interno muda**, como se trocasse de classe.

## Problema que resolve

Relaciona-se diretamente com **Finite-State Machines**: objetos que têm um conjunto finito de estados e cujo comportamento de cada método depende do estado atual. A implementação direta usa condicionais gigantes (`switch state`) dentro de cada método.

Conforme estados e transições crescem, cada método vira uma árvore de `if/else` inflada. Mudar uma regra de transição exige tocar em vários métodos. Adicionar um estado novo propaga-se para toda a classe. O código fica frágil, difícil de testar isoladamente e impossível de raciocinar.

Vários métodos terão o mesmo `switch`, duplicando o mapeamento estado → comportamento.

## Solução

Criar uma classe para cada estado e mover toda a lógica dependente daquele estado para ela. A classe original (**context**) guarda uma referência ao objeto de estado atual e **delega** para ele todas as chamadas que dependem do estado.

Para transicionar, basta trocar o objeto de estado no context. Todas as classes de estado implementam a mesma interface, então o context não as conhece diretamente. Estados podem também conhecer uns aos outros e disparar transições (diferença crucial para [[strategy]]).

## Estrutura

- **Context** — guarda referência ao state atual; delega operações; expõe `changeState(s)`.
- **State** (interface) — declara os métodos dependentes de estado.
- **Concrete States** — implementam comportamento daquele estado; podem iniciar transição para outros states.

## Pseudocódigo

```
interface State is
    method clickPlay()
    method clickLock()

class Player is
    private state: State
    constructor() is this.state = new ReadyState(this)
    method changeState(s) is this.state = s
    method clickPlay() is state.clickPlay()
    method clickLock() is state.clickLock()
    method startPlayback() is ...
    method stopPlayback() is ...

class ReadyState implements State is
    private player: Player
    method clickPlay() is
        player.startPlayback()
        player.changeState(new PlayingState(player))
    method clickLock() is
        player.changeState(new LockedState(player))

class PlayingState implements State is
    method clickPlay() is
        player.stopPlayback()
        player.changeState(new ReadyState(player))
    method clickLock() is
        player.changeState(new LockedState(player))
```

## Quando usar

- Objeto comporta-se diferente em estados diferentes, e o número de estados é grande ou cresce.
- Métodos da classe estão poluídos com condicionais grandes baseadas em estado.
- Duplicação de código entre estados similares — base abstrata ajuda.
- Transições têm regras complexas.

## Quando evitar

- Máquina de estado com 2-3 estados estáveis — `enum` + `switch` é mais simples.
- Estados raramente mudam; o ganho em manutenibilidade não compensa o overhead de classes.
- Domínio não tem realmente "estados" — só flags independentes.

## Sinais de que você está reinventando este pattern

- Campo `status: string` e `switch(status)` repetido em vários métodos.
- Condicional no início de quase todo método verificando "se posso fazer isso no meu estado".
- Tabela de transição (de, evento) → para implementada à mão.

## Trade-offs

### Vantagens
- **SRP** — cada estado vira uma classe coesa.
- **OCP** — novos estados sem alterar os existentes nem o context.
- Elimina condicionais de estado.
- Transições ficam explícitas como código, não como texto.

### Desvantagens
- Overkill se a máquina tem poucos estados.
- Mais classes para navegar.
- Estados que conhecem uns aos outros criam acoplamento entre estados.

## Relações com outros patterns

- [[strategy]] — estruturalmente idênticos (composição via interface). Diferença de intenção: Strategy são algoritmos independentes entre si, escolhidos de fora; State representa estados que conhecem uns aos outros e transicionam ativamente.
- [[bridge]] — composição semelhante, mas resolve problema de variação ortogonal de abstração/implementação.
- [[singleton]] — estados concretos sem estado próprio podem ser singletons reutilizáveis.

## Fontes

- [Refactoring Guru — State](https://refactoring.guru/design-patterns/state)
- Gamma, Helm, Johnson, Vlissides (1994). *Design Patterns: Elements of Reusable Object-Oriented Software*. Addison-Wesley.

## Links

- [[_moc]]
- [[behavioral-patterns]]
- [[../principles/solid]] — SRP e OCP aplicados por este pattern
- [[../_moc]] — knowledge raiz
