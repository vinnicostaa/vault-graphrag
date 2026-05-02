---
title: Parallel Inheritance Hierarchies (AP-023)
type: antipattern
tags: [antipattern, code-smell, change-preventer, inheritance]
source: https://refactoring.guru/smells/parallel-inheritance-hierarchies
source_date: 2026-04-24
created: 2026-04-24
updated: 2026-04-24
aliases: [mirrored-hierarchies, twin-hierarchies]
---

# AP-023 — Parallel Inheritance Hierarchies

Toda vez que você **cria uma subclasse** numa hierarquia, precisa criar uma subclasse **espelho** em outra hierarquia. As duas evoluem em paralelo, sempre iguais no número de filhos, sempre custando dobrado. Forma específica de [[shotgun-surgery]] entre hierarquias.

## Sintomas

- Hierarquia `Shape → Circle, Square, Triangle` + hierarquia `ShapeRenderer → CircleRenderer, SquareRenderer, TriangleRenderer`
- Adicionar `Hexagon` exige criar `HexagonRenderer`
- Padrão "X + XFactory + XValidator + XMapper" onde cada um vira um arquivo novo por subtipo
- Prefixos/sufixos repetidos por toda a codebase (`Blog` → `BlogController`, `BlogService`, `BlogRepo`, `BlogView`, `BlogForm`, `BlogSerializer`)
- Time reclama "toda feature é tanto arquivo novo..."

## Por que é problema

**Custo N×M de mudança**: adicionar um tipo obriga tocar em M hierarquias espelho. O trabalho de cada feature vira cerimônia.

**Coesão errada entre hierarquias**: se as duas andam sempre juntas, talvez sejam **duas facetas do mesmo conceito** que foram mal separadas.

**Hierarquias frágeis**: esquecer de criar o "gêmeo" produz bug silencioso (faltou renderer, faltou validator).

**Sinal de herança em excesso**: muitas hierarquias paralelas = herança foi escolhida como mecanismo de dispatch onde composição/dispatch-table resolveria sem dobrar tudo.

## Exemplo

```java
// ❌ Duas hierarquias espelho
abstract class Shape { abstract double area(); }
class Circle   extends Shape { ... }
class Square   extends Shape { ... }
class Triangle extends Shape { ... }

abstract class Renderer { abstract void render(Shape s); }
class CircleRenderer   extends Renderer { void render(Shape s) { ... } }
class SquareRenderer   extends Renderer { void render(Shape s) { ... } }
class TriangleRenderer extends Renderer { void render(Shape s) { ... } }
// adicionar Hexagon → criar HexagonRenderer também

// ✅ Eliminar paralelismo: Shape conhece como se renderiza (visitor/strategy)
abstract class Shape {
    abstract double area();
    abstract void render(Canvas c);   // polimorfismo interno
}
class Circle extends Shape { void render(Canvas c) { ... } }
// ou: Visitor / Double-dispatch / dispatch-table sem hierarquia espelho
```

## Como refatorar

- **Move Method / Move Field**: trazer a responsabilidade da hierarquia paralela pra dentro da principal (ex: `render()` vira método de `Shape`)
- **Visitor pattern**: se render/export/validate precisa ficar **separado** do modelo (ex: domain core limpo), Visitor encapsula variação sem hierarquia espelho
- **Strategy + dispatch table**: substituir subclasses por estratégia passada como parâmetro — `Map<Type, Handler>` elimina duas hierarquias
- **Collapse Hierarchy**: se a hierarquia secundária não tem razão própria, absorver no main
- **Sum types / discriminated unions** (Rust enum, TS union, Kotlin sealed): dispensa herança espelho; pattern match no tipo

## Quando tolerar

- **Quando as duas hierarquias têm razões de mudança genuinamente diferentes** — ex: renderer evolui por razão gráfica, shape por razão matemática. Mas verifique: são **realmente** independentes? Costumam não ser.
- **Framework plugável** onde extensibilidade em 2 eixos é requisito do design (ex: IDE plugin + language server): o custo é aceito em troca de extensibilidade real
- **Fases iniciais** antes do padrão ficar claro — ok por enquanto, refatorar quando virar dor
- **Domain + Read-model** em CQRS: podem parecer paralelos mas atendem **leituras diferentes** com lógicas distintas

## Patterns relacionados (que resolvem)

- [[shotgun-surgery]] — Parallel Inheritance é forma específica de shotgun em hierarquias
- [[divergent-change]] — se a hierarquia secundária é sempre tocada junto, talvez seja a mesma classe
- [[refused-bequest]] — subclasses espelho frequentemente herdam só pra "casar estrutura"
- [[../patterns/visitor]] — encapsula variação de operação sobre estrutura
- [[../patterns/strategy]] — dispatch por composição
- [[../patterns/template-method]] — esqueleto + passo variável
- [[../principles/solid]] — OCP, DIP

## Fontes

- Refactoring Guru — [Parallel Inheritance Hierarchies](https://refactoring.guru/smells/parallel-inheritance-hierarchies)
- Fowler, M. (2018). *Refactoring*. 2nd ed. Addison-Wesley.
- Gamma et al. (1994). *Design Patterns* — Visitor, Strategy.

## Links

- [[_moc]]
- [[../_moc]]
- [[../patterns/_moc]]
- [[shotgun-surgery]]
- [[divergent-change]]
- [[../patterns/visitor]]
