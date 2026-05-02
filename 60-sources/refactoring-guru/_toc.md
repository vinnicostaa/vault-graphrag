---
title: refactoring-guru — TOC curado
type: moc
tags: [sources, refactoring-guru, patterns, refactoring, gof, fowler, toc]
created: 2026-04-22
---

# refactoring.guru — Índice curado

Navegação **por domínio** (não por slug de URL). Após reorg: 376 MDs em EN (vs 3513 originais — i18n arquivado).

## Entrada rápida

- [[raw/design-patterns-index|design-patterns-index]] — ★ landing GoF
- [[raw/refactoring-index|refactoring-index]] — ★ landing catálogo Fowler

## 1. Design Patterns (GoF, 22 padrões + meta)

Pasta: `raw/design-patterns/`. Cada padrão tem `<pattern>.md` (overview) + `<pattern>/<lang>/` com exemplos nas 10 linguagens (cpp, csharp, go, java, php, python, ruby, rust, swift, typescript).

### Creational

- [[raw/design-patterns/factory-method|factory-method]]
- [[raw/design-patterns/abstract-factory|abstract-factory]]
- [[raw/design-patterns/builder|builder]]
- [[raw/design-patterns/prototype|prototype]]
- [[raw/design-patterns/singleton|singleton]]

### Structural

- [[raw/design-patterns/adapter|adapter]]
- [[raw/design-patterns/bridge|bridge]]
- [[raw/design-patterns/composite|composite]]
- [[raw/design-patterns/decorator|decorator]]
- [[raw/design-patterns/facade|facade]]
- [[raw/design-patterns/flyweight|flyweight]]
- [[raw/design-patterns/proxy|proxy]]

### Behavioral

- [[raw/design-patterns/chain-of-responsibility|chain-of-responsibility]]
- [[raw/design-patterns/command|command]]
- [[raw/design-patterns/iterator|iterator]]
- [[raw/design-patterns/mediator|mediator]]
- [[raw/design-patterns/memento|memento]]
- [[raw/design-patterns/observer|observer]]
- [[raw/design-patterns/state|state]]
- [[raw/design-patterns/strategy|strategy]] ★
- [[raw/design-patterns/template-method|template-method]]
- [[raw/design-patterns/visitor|visitor]] + [[raw/design-patterns/visitor-double-dispatch|visitor-double-dispatch]]

### Meta / grupos

- [[raw/design-patterns/catalog|catalog]] — visão geral
- [[raw/design-patterns/creational-patterns|creational-patterns]] — grupo
- [[raw/design-patterns/structural-patterns|structural-patterns]] — grupo
- [[raw/design-patterns/behavioral-patterns|behavioral-patterns]] — grupo
- [[raw/design-patterns/classification|classification]]
- [[raw/design-patterns/what-is-pattern|what-is-pattern]]
- [[raw/design-patterns/why-learn-patterns|why-learn-patterns]]
- [[raw/design-patterns/criticism|criticism]]
- [[raw/design-patterns/examples|examples]]
- [[raw/design-patterns/factory-comparison|factory-comparison]]
- [[raw/design-patterns/history|history]]
- [[raw/design-patterns/book|book]]

### Por linguagem (índices)

- [[raw/design-patterns/go|Go]], [[raw/design-patterns/typescript|TypeScript]], [[raw/design-patterns/java|Java]], [[raw/design-patterns/csharp|C#]], [[raw/design-patterns/python|Python]], [[raw/design-patterns/rust|Rust]], [[raw/design-patterns/ruby|Ruby]], [[raw/design-patterns/swift|Swift]], [[raw/design-patterns/php|PHP]], [[raw/design-patterns/cpp|C++]]

## 2. Refactoring catalog (Fowler, 66 técnicas)

Pasta: `raw/refactorings/` — um arquivo `.md` por técnica. Overview meta em `raw/refactoring/`.

### Meta

- [[raw/refactoring/what-is-refactoring|what-is-refactoring]]
- [[raw/refactoring/when|when to refactor]]
- [[raw/refactoring/how-to|how-to]]
- [[raw/refactoring/course|course]]
- [[raw/refactoring/catalog|catalog]] — overview
- [[raw/refactoring/technical-debt|technical-debt]]
- [[raw/refactoring/techniques|techniques overview]]
- [[raw/refactoring/smells|smells overview]]

### Grupos de técnicas (raw/refactoring/techniques/)

- [[raw/refactoring/techniques/composing-methods|composing-methods]]
- [[raw/refactoring/techniques/moving-features-between-objects|moving-features-between-objects]]
- [[raw/refactoring/techniques/organizing-data|organizing-data]]
- [[raw/refactoring/techniques/simplifying-conditional-expressions|simplifying-conditional-expressions]]
- [[raw/refactoring/techniques/simplifying-method-calls|simplifying-method-calls]]
- [[raw/refactoring/techniques/dealing-with-generalization|dealing-with-generalization]]

### Técnicas individuais (66 em `raw/refactorings/`)

**Composing methods:**
- [[raw/refactorings/extract-method|extract-method]] ★
- [[raw/refactorings/inline-method|inline-method]]
- [[raw/refactorings/extract-variable|extract-variable]]
- [[raw/refactorings/inline-temp|inline-temp]]
- [[raw/refactorings/replace-temp-with-query|replace-temp-with-query]]
- [[raw/refactorings/split-temporary-variable|split-temporary-variable]]
- [[raw/refactorings/remove-assignments-to-parameters|remove-assignments-to-parameters]]
- [[raw/refactorings/replace-method-with-method-object|replace-method-with-method-object]]
- [[raw/refactorings/substitute-algorithm|substitute-algorithm]]

**Moving features:**
- [[raw/refactorings/move-method|move-method]]
- [[raw/refactorings/move-field|move-field]]
- [[raw/refactorings/extract-class|extract-class]]
- [[raw/refactorings/inline-class|inline-class]]
- [[raw/refactorings/hide-delegate|hide-delegate]]
- [[raw/refactorings/remove-middle-man|remove-middle-man]]
- [[raw/refactorings/introduce-foreign-method|introduce-foreign-method]]
- [[raw/refactorings/introduce-local-extension|introduce-local-extension]]

**Organizing data:**
- [[raw/refactorings/self-encapsulate-field|self-encapsulate-field]]
- [[raw/refactorings/replace-data-value-with-object|replace-data-value-with-object]]
- [[raw/refactorings/change-value-to-reference|change-value-to-reference]]
- [[raw/refactorings/change-reference-to-value|change-reference-to-value]]
- [[raw/refactorings/replace-array-with-object|replace-array-with-object]]
- [[raw/refactorings/duplicate-observed-data|duplicate-observed-data]]
- [[raw/refactorings/change-unidirectional-association-to-bidirectional|association-uni→bi]]
- [[raw/refactorings/change-bidirectional-association-to-unidirectional|association-bi→uni]]
- [[raw/refactorings/replace-magic-number-with-symbolic-constant|replace-magic-number-with-symbolic-constant]]
- [[raw/refactorings/encapsulate-field|encapsulate-field]]
- [[raw/refactorings/encapsulate-collection|encapsulate-collection]]
- [[raw/refactorings/replace-type-code-with-class|replace-type-code-with-class]]
- [[raw/refactorings/replace-type-code-with-subclasses|replace-type-code-with-subclasses]]
- [[raw/refactorings/replace-type-code-with-state-strategy|replace-type-code-with-state/strategy]]
- [[raw/refactorings/replace-subclass-with-fields|replace-subclass-with-fields]]

**Simplifying conditionals:**
- [[raw/refactorings/decompose-conditional|decompose-conditional]]
- [[raw/refactorings/consolidate-conditional-expression|consolidate-conditional-expression]]
- [[raw/refactorings/consolidate-duplicate-conditional-fragments|consolidate-duplicate-conditional-fragments]]
- [[raw/refactorings/remove-control-flag|remove-control-flag]]
- [[raw/refactorings/replace-nested-conditional-with-guard-clauses|replace-nested-conditional-with-guard-clauses]] ★
- [[raw/refactorings/replace-conditional-with-polymorphism|replace-conditional-with-polymorphism]] ★
- [[raw/refactorings/introduce-null-object|introduce-null-object]]
- [[raw/refactorings/introduce-assertion|introduce-assertion]]

**Simplifying method calls:**
- [[raw/refactorings/rename-method|rename-method]]
- [[raw/refactorings/add-parameter|add-parameter]]
- [[raw/refactorings/remove-parameter|remove-parameter]]
- [[raw/refactorings/separate-query-from-modifier|separate-query-from-modifier]]
- [[raw/refactorings/parameterize-method|parameterize-method]]
- [[raw/refactorings/replace-parameter-with-explicit-methods|replace-parameter-with-explicit-methods]]
- [[raw/refactorings/preserve-whole-object|preserve-whole-object]]
- [[raw/refactorings/replace-parameter-with-method-call|replace-parameter-with-method-call]]
- [[raw/refactorings/introduce-parameter-object|introduce-parameter-object]]
- [[raw/refactorings/remove-setting-method|remove-setting-method]]
- [[raw/refactorings/hide-method|hide-method]]
- [[raw/refactorings/replace-constructor-with-factory-method|replace-constructor-with-factory-method]]
- [[raw/refactorings/replace-error-code-with-exception|replace-error-code-with-exception]]
- [[raw/refactorings/replace-exception-with-test|replace-exception-with-test]]

**Dealing with generalization:**
- [[raw/refactorings/pull-up-field|pull-up-field]]
- [[raw/refactorings/pull-up-method|pull-up-method]]
- [[raw/refactorings/pull-up-constructor-body|pull-up-constructor-body]]
- [[raw/refactorings/push-down-field|push-down-field]]
- [[raw/refactorings/push-down-method|push-down-method]]
- [[raw/refactorings/extract-subclass|extract-subclass]]
- [[raw/refactorings/extract-superclass|extract-superclass]]
- [[raw/refactorings/extract-interface|extract-interface]]
- [[raw/refactorings/collapse-hierarchy|collapse-hierarchy]]
- [[raw/refactorings/form-template-method|form-template-method]]
- [[raw/refactorings/replace-inheritance-with-delegation|replace-inheritance-with-delegation]]
- [[raw/refactorings/replace-delegation-with-inheritance|replace-delegation-with-inheritance]]

## 3. Code Smells (23 em `raw/smells/`)

### Bloaters
- [[raw/smells/long-method|long-method]]
- [[raw/smells/large-class|large-class]]
- [[raw/smells/primitive-obsession|primitive-obsession]]
- [[raw/smells/long-parameter-list|long-parameter-list]]
- [[raw/smells/data-clumps|data-clumps]]

### OO Abusers
- [[raw/smells/switch-statements|switch-statements]] ★
- [[raw/smells/temporary-field|temporary-field]]
- [[raw/smells/refused-bequest|refused-bequest]]
- [[raw/smells/alternative-classes-with-different-interfaces|alternative-classes-with-different-interfaces]]

### Change Preventers
- [[raw/smells/divergent-change|divergent-change]]
- [[raw/smells/shotgun-surgery|shotgun-surgery]]
- [[raw/smells/parallel-inheritance-hierarchies|parallel-inheritance-hierarchies]]

### Dispensables
- [[raw/smells/comments|comments]]
- [[raw/smells/duplicate-code|duplicate-code]]
- [[raw/smells/lazy-class|lazy-class]]
- [[raw/smells/data-class|data-class]]
- [[raw/smells/dead-code|dead-code]]
- [[raw/smells/speculative-generality|speculative-generality]]

### Couplers
- [[raw/smells/feature-envy|feature-envy]]
- [[raw/smells/inappropriate-intimacy|inappropriate-intimacy]]
- [[raw/smells/message-chains|message-chains]]
- [[raw/smells/middle-man|middle-man]]

### Outros
- [[raw/smells/incomplete-library-class|incomplete-library-class]]

**Grupos** (em `raw/refactoring/smells/`): [[raw/refactoring/smells/bloaters|bloaters]], [[raw/refactoring/smells/change-preventers|change-preventers]], [[raw/refactoring/smells/couplers|couplers]], [[raw/refactoring/smells/dispensables|dispensables]], [[raw/refactoring/smells/oo-abusers|oo-abusers]], [[raw/refactoring/smells/other|other]].

## Prioridade de destilação

**P0 — destilar em `_notes/` e promover:**
1. 23 GoF patterns → notas atômicas em `[[../../10-knowledge/patterns/]]` (1 nota por pattern, destilado em pt-BR citando `raw/design-patterns/<pattern>` como fonte)
2. 23 Code smells → `[[../../10-knowledge/antipatterns/]]` como AP-### (mapeamento smell → refactoring sugerido)

**P1:**
3. 66 refactorings → referência em `[[../../10-knowledge/methodology/refactoring-techniques]]` (catálogo linkando raw/refactorings/)
4. Grupos de smells (`bloaters`, `change-preventers`, etc) → categorização em `10-knowledge/antipatterns/_moc`

## Vizinhos no grafo

- [[_index]]
- [[_notes/_moc]]
- [[../../10-knowledge/patterns/_moc]] — destino GoF
- [[../../10-knowledge/antipatterns/_moc]] — destino smells
- [[../../10-knowledge/methodology/_moc]] — destino refactoring catalog
- Site original: https://refactoring.guru
