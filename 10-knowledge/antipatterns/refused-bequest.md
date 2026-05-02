---
title: Refused Bequest (AP-015)
type: antipattern
tags: [antipattern, code-smell, oo-abuser, inheritance]
source: https://refactoring.guru/smells/refused-bequest
source_date: 2026-04-24
created: 2026-04-24
updated: 2026-04-24
aliases: [rejected-inheritance, subclass-doesnt-fit]
---

# AP-015 — Refused Bequest

Subclasse que **herda** de uma superclasse mas **rejeita parte da herança** — sobrescreve métodos pra jogar exception, ignora atributos, ou só usa um pedaço do contrato. Quebra Liskov Substitution (LSP) e sinaliza que herança foi escolhida por reuso e não por substituibilidade.

## Sintomas

- Métodos sobrescritos com `throw new UnsupportedOperationException()` ou `return;` vazio
- `Stack extends Vector` (clássico JDK) — `Stack` herda `get(int)` que não faz sentido
- Subclasse ignora campos protegidos da pai
- `if (obj instanceof SubType) { ... }` aparece porque o polimorfismo não funciona mais
- Testes do pai falham quando rodados contra filho
- Comentário tipo `// not applicable here` sobre método herdado

## Por que é problema

**Violação de LSP**: código escrito pra `Parent` deveria funcionar com qualquer filho — se filho "recusa" parte, `Parent p = new Child(); p.method()` explode em runtime.

**Hierarquia mentirosa**: herança comunica "é um" — se `Square extends Rectangle` não se comporta como `Rectangle`, a hierarquia está errada (caso clássico Square/Rectangle).

**Contrato quebrado**: consumidores de `Parent` precisam checar tipo concreto antes de chamar métodos — derrota polimorfismo, reintroduz `switch` implícito.

**Reuso mal-motivado**: quase sempre a pessoa queria **compartilhar implementação**, não substituibilidade. Composição/delegação resolveria sem mentir.

## Exemplo

```kotlin
// ❌ Stack "é uma" Vector? tem .get(i) que quebra semântica de pilha
open class Vector { open fun get(i: Int): Any { /* ... */ } }
class Stack : Vector() {
    override fun get(i: Int): Any = throw UnsupportedOperationException()
    fun push(x: Any) { /* ... */ }
    fun pop(): Any   { /* ... */ }
}

// ✅ Composição — Stack USA Vector, não É Vector
class Stack {
    private val store = Vector()
    fun push(x: Any) { store.addLast(x) }
    fun pop(): Any   = store.removeLast()
    // nenhum get(i) leaked
}
```

## Como refatorar

- **Replace Inheritance with Delegation**: subclasse guarda instância do ex-pai e delega o que lhe interessa
- **Extract Superclass**: mover o que **é comum de verdade** pra superclasse menor; deixar o resto fora
- **Pull Up / Push Down Method**: realinhar métodos onde fazem sentido semanticamente
- **Compor via interface**: criar interface com só as capacidades que o filho honra
- Em linguagens com traits/mixins: usar mixin em vez de herança clássica

## Quando tolerar

- **Template Method** onde métodos abstratos são "hooks" que o filho **deliberadamente** escolhe não estender (mas mantém contrato)
- **Framework base classes** que expõem N capacidades opcionais (ex: `HttpServlet` com doGet/doPost) — é documentado e esperado
- **Hierarquias de erro/exception** onde filho não agrega comportamento, só tipo — trivial e aceitável

## Patterns relacionados (que resolvem)

- [[../patterns/strategy]] — composição de comportamento sem herança
- [[../patterns/decorator]] — adicionar responsabilidade sem herdar contrato inteiro
- [[../patterns/adapter]] — adaptar interface antiga a nova sem "herdar pela metade"
- [[../principles/solid]] — LSP (L de SOLID); "favor composition over inheritance"
- [[speculative-generality]] — herança especulativa frequentemente gera Refused Bequest

## Fontes

- Refactoring Guru — [Refused Bequest](https://refactoring.guru/smells/refused-bequest)
- Fowler, M. (2018). *Refactoring*. 2nd ed. Addison-Wesley.
- Liskov, B. (1987). "Data Abstraction and Hierarchy" — origem do LSP.
- Gamma et al. (1994). *Design Patterns: Elements of Reusable Object-Oriented Software* — "favor composition over inheritance".

## Links

- [[_moc]]
- [[../_moc]]
- [[../patterns/_moc]]
- [[../principles/solid]]
- [[speculative-generality]]
