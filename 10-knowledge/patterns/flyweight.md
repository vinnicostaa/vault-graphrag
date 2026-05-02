---
title: Flyweight
type: pattern
tags: [pattern, structural, gof, memory, sharing]
source: https://refactoring.guru/design-patterns/flyweight
created: 2026-04-24
updated: 2026-04-24
aliases: [Flyweight Pattern, Cache]
---

# Flyweight

Padrão **estrutural** que reduz consumo de memória **compartilhando partes comuns** de estado entre muitos objetos similares, em vez de duplicar esses dados em cada instância.

## Problema que resolve

Quando um app precisa manter **vastíssima quantidade de objetos similares** em memória (partículas de um jogo, células de uma planilha, marcadores num mapa, tokens num lexer), e cada objeto carrega campos pesados que são, na prática, **idênticos** em muitos deles — textura de sprite, fonte, cor, bitmap — a RAM estoura antes do app conseguir representar a cena.

Alocar o mesmo bitmap de 2 MB em um milhão de partículas é desperdício: só coordenadas e velocidade de fato variam.

## Solução

Separar o estado em duas partes:

- **Intrinsic state** — imutável, compartilhável, igual entre muitos objetos (sprite, cor, textura).
- **Extrinsic state** — único por objeto, mutável, dependente de contexto (coordenadas, velocidade, dono).

Manter só o *intrinsic* dentro da classe **Flyweight** (imutável, inicializado no construtor, sem setters). O *extrinsic* é passado como parâmetro nos métodos ou guardado em um objeto **Context** que referencia o Flyweight.

Uma **Flyweight Factory** mantém um pool: dado um estado intrínseco, retorna o Flyweight existente ou cria um novo. Clientes nunca instanciam Flyweight diretamente.

## Estrutura

- **Flyweight** — contém apenas o estado intrínseco; imutável; é compartilhado.
- **Context** — guarda o estado extrínseco e uma referência ao Flyweight; representa o objeto original quando pareado com o Flyweight.
- **Flyweight Factory** — gerencia o pool; dado o estado intrínseco desejado, devolve (ou cria) o Flyweight correspondente.
- **Client** — calcula/guarda o estado extrínseco; pede Flyweights à factory; chama métodos passando o extrínseco como parâmetro.

## Pseudocódigo

```
// Flyweight: intrinsic state, imutável.
class TreeType is
    field name, color, texture
    constructor(name, color, texture) { ... }
    method draw(canvas, x, y) is
        /* bitmap com (name, color, texture) desenhado em (x, y) */

// Factory: garante compartilhamento.
class TreeFactory is
    static field treeTypes: map
    static method getTreeType(name, color, texture) is
        key = (name, color, texture)
        if treeTypes[key] == null then
            treeTypes[key] = new TreeType(name, color, texture)
        return treeTypes[key]

// Context: extrinsic state + referência ao Flyweight.
class Tree is
    field x, y
    field type: TreeType
    constructor(x, y, type) { ... }
    method draw(canvas) is type.draw(canvas, this.x, this.y)

// Cliente.
class Forest is
    field trees: array of Tree
    method plant(x, y, name, color, texture) is
        type = TreeFactory.getTreeType(name, color, texture)
        trees.add(new Tree(x, y, type))
    method draw(canvas) is
        foreach t in trees do t.draw(canvas)
```

## Quando usar

- O app precisa suportar um **número enorme** de objetos similares.
- Esses objetos estouram (ou ameaçam estourar) a RAM disponível.
- Há **dados duplicados** claramente identificáveis que podem ser extraídos e compartilhados.
- Objetos de baixa granularidade em editores gráficos, engines de jogo, árvores de UI, caches de tokens.

## Quando evitar

- Não há problema real de memória — é **apenas otimização**; aplicar sem medir é complicar de graça.
- O estado intrínseco tem variações demais — pool vira quase 1:1 com instâncias.
- Clientes precisam mutar o estado supostamente intrínseco — quebra a invariante de imutabilidade.

## Sinais de que você está reinventando este pattern

- Cache manual de configurações/fontes/sprites por chave composta.
- Classe imutável com factory que deduplica instâncias (ex.: interning de strings).
- Separação explícita entre "template" e "instância leve" do mesmo conceito.

## Trade-offs

### Vantagens
- Economia expressiva de RAM quando há muitos objetos similares.

### Desvantagens
- Troca RAM por **CPU** — parte do estado pode precisar ser recalculado a cada chamada.
- Código fica mais complexo; novo desenvolvedor estranha a separação intrínseco/extrínseco.
- Flyweight **imutável** — se o domínio evoluir e exigir mutação, o pattern precisa ser revertido.
- Garantir thread-safety do pool exige cuidado.

## Relações com outros patterns

- [[composite]] — folhas compartilhadas de uma árvore Composite podem ser implementadas como Flyweights.
- [[facade]] — Facade representa **um** subsistema inteiro; Flyweight representa **muitos** objetos pequenos compartilháveis.
- [[singleton]] — lembra Singleton se o pool reduz tudo a uma única instância, mas: (1) Singleton tem uma instância, Flyweight tem várias com estados intrínsecos distintos; (2) Singleton pode ser mutável, Flyweight é imutável.
- [[factory-method]], [[abstract-factory]] — a Flyweight Factory é uma aplicação de factory dedicada a deduplicação.

## Fontes

- [Refactoring Guru — Flyweight](https://refactoring.guru/design-patterns/flyweight)
- Gamma, Helm, Johnson, Vlissides (1994). *Design Patterns: Elements of Reusable Object-Oriented Software*. Addison-Wesley.

## Links

- [[_moc]]
- [[structural-patterns]]
- [[composite]] — folhas compartilhadas como Flyweights
- [[../_moc]] — knowledge raiz
