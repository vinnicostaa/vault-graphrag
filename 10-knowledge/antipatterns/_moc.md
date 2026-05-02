---
title: MOC — Antipatterns
type: moc
tags: [moc, antipatterns, code-smells, fowler, refactoring-guru]
created: 2026-04-22
updated: 2026-04-24
---

# Antipatterns

Catálogo canônico cross-linguagem de **code smells** e **design smells** recorrentes, com o fix preferido linkado a [[../patterns/_moc]]. Destilação própria (pt-BR) de fontes abertas: Fowler *Refactoring 2nd ed.*, Martin *Clean Code*, Evans *DDD*, [refactoring.guru/smells](https://refactoring.guru/smells).

Cada AP possui nota atômica com sintomas, por que é problema, exemplo curto, como refatorar, quando tolerar e patterns relacionados.

## Catálogo canônico (AP-001 a AP-023)

> **Distinção** (reconciliação 2026-04-24, [[../../00-meta/STATE|D-vault-012]]): este catálogo cobre **code smells clássicos** (Fowler/GoF — design/structure). Para **antipatterns operacionais** (concorrência/idempotência/cache/segurança runtime/performance) ver [[../runtime-antipatterns/_moc|OP-001..OP-026]]. Os dois catálogos são **distintos e não-colidentes** desde esta reconciliação.

| ID | Nome | 1-liner | Categoria |
|---|---|---|---|
| [[global-mutable-state\|AP-001]] | Global Mutable State | Estado compartilhado mutável acopla tudo e quebra testes/concorrência | design-smell |
| [[god-object\|AP-002]] | God Object / Large Class | Classe concentra múltiplas responsabilidades, viola SRP | bloater |
| [[anemic-domain-model\|AP-003]] | Anemic Domain Model | Entidades só com getters/setters; regras fora, no service | design-smell |
| [[long-parameter-list\|AP-004]] | Long Parameter List | Funções com 4+ parâmetros soltos em vez de objeto agregador | bloater |
| [[primitive-obsession\|AP-005]] | Primitive Obsession | `string/int` onde cabe Value Object com invariantes | bloater |
| [[data-clumps\|AP-006]] | Data Clumps | Campos que andam juntos sem classe que os represente | bloater |
| [[feature-envy\|AP-007]] | Feature Envy | Método mais interessado noutra classe que na sua | coupler |
| [[shotgun-surgery\|AP-008]] | Shotgun Surgery | Mudança simples obriga editar muitos arquivos | change-preventer |
| [[magic-numbers\|AP-009]] | Magic Numbers | Literais numéricos/string sem nome explicativo | readability |
| [[long-method\|AP-010]] | Long Method | Função grande que faz muitas coisas | bloater |
| [[duplicate-code\|AP-011]] | Duplicate Code | Mesma lógica replicada — fix e bug viajam em N cópias | dispensable |
| [[speculative-generality\|AP-012]] | Speculative Generality | Abstração pronta "para o futuro" — YAGNI violation | dispensable |
| [[temporary-field\|AP-013]] | Temporary Field | Campo usado só em certos cenários, `null` o resto | oo-abuser |
| [[data-class\|AP-014]] | Data Class | Classe só com campos + getters/setters, sem comportamento | bloater |
| [[refused-bequest\|AP-015]] | Refused Bequest | Subclasse rejeita herança — quebra LSP | oo-abuser |
| [[comments\|AP-016]] | Comments (excessive) | Comentários como deodorante de código obscuro | dispensable |
| [[lazy-class\|AP-017]] | Lazy Class | Classe não faz o suficiente pra justificar existir | dispensable |
| [[middle-man\|AP-018]] | Middle Man | Classe só delega — cliente poderia falar direto | coupler |
| [[message-chains\|AP-019]] | Message Chains | `a.b().c().d()` — viola Law of Demeter | coupler |
| [[inappropriate-intimacy\|AP-020]] | Inappropriate Intimacy | Classes conhecem internals uma da outra | coupler |
| [[alternative-classes-different-interfaces\|AP-021]] | Alternative Classes w/ Different Interfaces | Mesma semântica, APIs divergentes; perde polimorfismo | oo-abuser |
| [[divergent-change\|AP-022]] | Divergent Change | Uma classe muda por muitas razões; SRP violado | change-preventer |
| [[parallel-inheritance-hierarchies\|AP-023]] | Parallel Inheritance Hierarchies | Subclasse nova aqui obriga subclasse nova lá | change-preventer |

## Categorias (taxonomia Fowler + Refactoring Guru)

### Bloaters — cresceram demais
- [[god-object]] (AP-002)
- [[long-method]] (AP-010)
- [[long-parameter-list]] (AP-004)
- [[primitive-obsession]] (AP-005)
- [[data-clumps]] (AP-006)
- [[data-class]] (AP-014)

### Design smells — arquitetura comprometida
- [[global-mutable-state]] (AP-001)
- [[anemic-domain-model]] (AP-003)

### OO abusers
- [[temporary-field]] (AP-013)
- [[refused-bequest]] (AP-015)
- [[alternative-classes-different-interfaces]] (AP-021)

### Change preventers
- [[shotgun-surgery]] (AP-008)
- [[divergent-change]] (AP-022)
- [[parallel-inheritance-hierarchies]] (AP-023)

### Couplers
- [[feature-envy]] (AP-007)
- [[middle-man]] (AP-018)
- [[message-chains]] (AP-019)
- [[inappropriate-intimacy]] (AP-020)

### Dispensables
- [[duplicate-code]] (AP-011)
- [[speculative-generality]] (AP-012)
- [[comments]] (AP-016)
- [[lazy-class]] (AP-017)

### Readability / clean code
- [[magic-numbers]] (AP-009)

## Mapa antipattern → pattern

| Antipattern | Fix primário | Patterns relacionados |
|---|---|---|
| AP-001 Global Mutable | DI + passar contexto | [[../patterns/singleton]] disciplinado, [[../patterns/facade]] |
| AP-002 God Object | Extract Class, SRP | [[../patterns/facade]], [[../patterns/mediator]], [[../patterns/strategy]] |
| AP-003 Anemic Domain | Move Method para entidade | [[../architecture/ddd-strategic-tactical]], [[../patterns/state]] |
| AP-004 Long Parameter List | Introduce Parameter Object | [[../patterns/builder]] |
| AP-005 Primitive Obsession | Replace Data Value with Object | Value Object |
| AP-006 Data Clumps | Extract Class / VO | [[../patterns/builder]] |
| AP-007 Feature Envy | Move Method | [[../patterns/visitor]] |
| AP-008 Shotgun Surgery | Consolidate + Mediator | [[../patterns/mediator]], [[../patterns/strategy]] |
| AP-009 Magic Numbers | Replace with Symbolic Constant | [[../principles/clean-code]] |
| AP-010 Long Method | Extract Method + guard clauses | [[../patterns/template-method]], [[../principles/code-calisthenics]] |
| AP-011 Duplicate Code | Extract Method/Class | [[../patterns/template-method]], [[../principles/do-dont-checklist]] |
| AP-012 Speculative Generality | Inline + YAGNI | [[../methodology/gsd-get-shit-done]] |
| AP-013 Temporary Field | Extract Class / Null Object | [[../patterns/state]], [[../patterns/builder]] |
| AP-014 Data Class | Move Method / Encapsulate + VO | Value Object, [[../architecture/ddd-strategic-tactical]] |
| AP-015 Refused Bequest | Replace Inheritance with Delegation | [[../patterns/strategy]], [[../patterns/decorator]] |
| AP-016 Comments (excessive) | Extract Method / Rename | [[../principles/clean-code]] |
| AP-017 Lazy Class | Inline Class / Collapse Hierarchy | [[../methodology/gsd-get-shit-done]] |
| AP-018 Middle Man | Remove Middle Man | [[../patterns/facade]], [[../patterns/proxy]] |
| AP-019 Message Chains | Hide Delegate / Extract Method | [[../patterns/facade]], [[../patterns/mediator]] |
| AP-020 Inappropriate Intimacy | Move Method / Extract Class | [[../patterns/facade]], [[../patterns/mediator]] |
| AP-021 Alternative Classes | Extract Interface / Rename | [[../patterns/strategy]], [[../patterns/adapter]] |
| AP-022 Divergent Change | Extract Class por razão-de-mudança | [[../patterns/strategy]], SRP |
| AP-023 Parallel Inheritance | Move Method / Visitor / Sum Types | [[../patterns/visitor]], [[../patterns/strategy]] |

## Como contribuir

1. Antipattern novo num projeto: verificar se cabe num AP-001..AP-023 existente
2. Cabe → enriquecer a nota com caso concreto na seção apropriada
3. Não cabe → propor nota atômica nova em kebab-case aqui, linkar de volta no MOC
4. Reviewer cita o ID (`AP-XXX`) em review de PR

## Princípios relacionados

- [[../principles/solid]] — vários APs violam SRP, OCP, LSP, DIP diretamente
- [[../principles/clean-code]] — naming, funções pequenas, comentários
- [[../principles/code-calisthenics]] — regras que previnem bloaters
- [[../principles/do-dont-checklist]] — DRY, YAGNI, KISS em forma de checklist
- [[../methodology/gsd-get-shit-done]] — GSD/YAGNI versus over-engineering

## Fontes

- Fowler, M. (2018). *Refactoring: Improving the Design of Existing Code*. 2nd ed. Addison-Wesley.
- Martin, R. C. (2008). *Clean Code*. Prentice Hall.
- Evans, E. (2003). *Domain-Driven Design*. Addison-Wesley.
- Refactoring Guru — https://refactoring.guru/smells (destilado em [[../../60-sources/refactoring-guru/_toc]])

## Vizinhos

- [[../_moc]] — knowledge
- [[../patterns/_moc]] — patterns que resolvem os APs
- [[../principles/_moc|principles]]
- [[../../60-sources/refactoring-guru/_toc]] — fonte raw
