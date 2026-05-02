---
title: Strategy
type: pattern
tags: [pattern, behavioral, gof, algorithms, runtime-swap]
source: https://refactoring.guru/design-patterns/strategy
created: 2026-04-24
updated: 2026-04-24
aliases: [Strategy Pattern]
---

# Strategy

Padrão **comportamental** que permite **definir uma família de algoritmos**, encapsular cada um em uma classe separada e **trocá-los em tempo de execução** sem alterar o código que os usa.

## Problema que resolve

Classe acumula múltiplas variantes do mesmo algoritmo via condicional gigante (`if/else`/`switch`) ou via subclasses que divergem só em uma operação. Consequências:

- Cada nova variante infla a classe.
- Mudança em uma variante afeta o arquivo inteiro, aumentando risco de regressão.
- *Merge conflicts* crescem quando múltiplos devs editam o mesmo arquivo para adicionar variantes distintas.

## Solução

- Extrair cada variante do algoritmo para sua própria classe — a **strategy**.
- A classe original (**context**) mantém referência para uma strategy via **interface comum** e delega o trabalho a ela em vez de decidir como executar.
- Context desconhece os detalhes das strategies — comunica só via interface. Cliente escolhe a strategy concreta e injeta no context.

## Estrutura

- **Context** — mantém referência a uma strategy e comunica-se com ela pela interface; expõe setter para trocar strategy em runtime.
- **Strategy** (interface) — declara o(s) método(s) que o context chama.
- **Concrete Strategies** — implementam variantes diferentes do algoritmo seguindo a interface.
- **Client** — instancia uma strategy concreta e passa ao context.

## Pseudocódigo

```
interface Strategy is
    method execute(a, b)

class StrategyAdd implements Strategy is
    method execute(a, b) is return a + b

class StrategySubtract implements Strategy is
    method execute(a, b) is return a - b

class Context is
    private strategy: Strategy

    method setStrategy(s: Strategy) is
        this.strategy = s

    method executeStrategy(a, b) is
        return strategy.execute(a, b)

// Client:
ctx = new Context()
if (action == "add") then ctx.setStrategy(new StrategyAdd())
if (action == "sub") then ctx.setStrategy(new StrategySubtract())
result = ctx.executeStrategy(x, y)
```

## Quando usar

- Precisa alternar variantes de um algoritmo em runtime.
- Tem várias classes semelhantes que só diferem em uma parte do comportamento.
- Quer isolar detalhes de um algoritmo (dados internos, dependências) do código que o consome.
- Tem uma condicional gigante selecionando variante do mesmo algoritmo.

## Quando evitar

- Se existem só 2 variantes estáveis e elas raramente mudam, a abstração adiciona código sem ganho.
- Clientes precisam conhecer as diferenças entre strategies para escolher a certa — trade-off explícito, não bug.
- Linguagens com funções de 1ª classe frequentemente permitem **funções anônimas/lambdas** no lugar de classes Strategy — avaliar antes de criar hierarquia.

## Sinais de que você está reinventando este pattern

- Classe com `switch` gigante escolhendo entre variantes do mesmo algoritmo.
- Hierarquia de subclasses onde cada uma só sobrescreve **um** método.
- Parâmetro `mode: string` que dispara ramificação interna grande dentro do método.

## Trade-offs

### Vantagens
- Algoritmos trocáveis em runtime.
- Isola detalhes de implementação do código que consome.
- Composição substitui herança.
- **Open/Closed Principle** — novas strategies entram sem alterar o context.

### Desvantagens
- Complexidade extra se há poucas variantes estáveis.
- Cliente precisa conhecer diferenças entre strategies para escolher.
- Em linguagens com lambdas/closures, pode-se alcançar o mesmo efeito sem classes adicionais.

## Relações com outros patterns

- [[command]] — ambos parametrizam objeto com ação, mas Command converte **uma operação** em objeto (com contexto + parâmetros) para fila/histórico/remoto; Strategy descreve **formas diferentes de fazer a mesma coisa**.
- [[state]] — extensão de Strategy. States podem conhecer e transicionar entre si; Strategies são independentes.
- [[bridge]], [[adapter]] — estruturalmente semelhantes (todos baseados em composição); distinção é de intenção.
- [[decorator]] — muda "a pele" do objeto; Strategy muda "as tripas".
- [[template-method]] — altera partes de um algoritmo via herança (estático, *class-level*); Strategy altera via composição (dinâmico, *object-level*).

## Fontes

- [Refactoring Guru — Strategy](https://refactoring.guru/design-patterns/strategy)
- Gamma, Helm, Johnson, Vlissides (1994). *Design Patterns: Elements of Reusable Object-Oriented Software*. Addison-Wesley.

## Links

- [[_moc]]
- [[behavioral-patterns]]
- [[../principles/solid]] — OCP aplicado por este pattern
- [[../_moc]] — knowledge raiz
