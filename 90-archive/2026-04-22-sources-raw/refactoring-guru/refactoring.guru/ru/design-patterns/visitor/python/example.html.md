:::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [Паттерны
проектирования](../../../design-patterns.html){.type} /
[Посетитель](../../visitor.html){.type} /
[Python](../../python.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Посетитель](../../../../images/patterns/cards/visitor-mini0a1c.png?id=854a35a62963bec1d75eab996918989b){srcset="/images/patterns/cards/visitor-mini-2x.png?id=9b87f3f3b772f434b28a25876829b504 2x"}
:::

# **Посетитель** на Python {#посетитель-на-python .pattern-example-title-block-title}
::::

::::::::::: pattern-example-body
::: pattern-example-brief
**Посетитель** --- это поведенческий паттерн, который позволяет добавить
новую операцию для целой иерархии классов, не изменяя код этих классов.

> Подробней о том, почему Посетитель нельзя заменить простой перегрузкой
> методов читайте в статье [Посетитель и Double
> Dispatch](../../visitor-double-dispatch.html).
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Подробней о паттерне Посетитель ](../../visitor.html){.btn .btn-lg
.btn-primary}
:::

:::::::: {.sidebar-navigation .with-scroll}
::: en-title
Навигация
:::

::: en-intro
 [Интро](#)
:::

::: en-intro
 [Концептуальный пример](#example-0)
:::

::: en-file
 [main](#example-0--main-py)
:::

::: en-file
 [Output](#example-0--Output-txt)
:::
::::::::

**Сложность:** []{.fa-stars}

**Популярность:** []{.fa-stars}

**Применимость:** Посетитель нечасто встречается в Python-коде из-за
своей сложности и нюансов реализазации.
:::::::::::

[]{#example-0}

## Концептуальный пример {#example-0-title}

Этот пример показывает структуру паттерна **Посетитель**, а именно ---
из каких классов он состоит, какие роли эти классы выполняют и как они
взаимодействуют друг с другом.

#### []{#example-0--main-py .anchor} **main.py:** Пример структуры паттерна

<figure class="code">
<pre class="code" lang="python"><code>from __future__ import annotations
from abc import ABC, abstractmethod
from typing import List


class Component(ABC):
    &quot;&quot;&quot;
    Интерфейс Компонента объявляет метод accept, который в качестве аргумента
    может получать любой объект, реализующий интерфейс посетителя.
    &quot;&quot;&quot;

    @abstractmethod
    def accept(self, visitor: Visitor) -&gt; None:
        pass


class ConcreteComponentA(Component):
    &quot;&quot;&quot;
    Каждый Конкретный Компонент должен реализовать метод accept таким образом,
    чтобы он вызывал метод посетителя, соответствующий классу компонента.
    &quot;&quot;&quot;

    def accept(self, visitor: Visitor) -&gt; None:
        &quot;&quot;&quot;
        Обратите внимание, мы вызываем visitConcreteComponentA, что
        соответствует названию текущего класса. Таким образом мы позволяем
        посетителю узнать, с каким классом компонента он работает.
        &quot;&quot;&quot;

        visitor.visit_concrete_component_a(self)

    def exclusive_method_of_concrete_component_a(self) -&gt; str:
        &quot;&quot;&quot;
        Конкретные Компоненты могут иметь особые методы, не объявленные в их
        базовом классе или интерфейсе. Посетитель всё же может использовать эти
        методы, поскольку он знает о конкретном классе компонента.
        &quot;&quot;&quot;

        return &quot;A&quot;


class ConcreteComponentB(Component):
    &quot;&quot;&quot;
    То же самое здесь: visitConcreteComponentB =&gt; ConcreteComponentB
    &quot;&quot;&quot;

    def accept(self, visitor: Visitor):
        visitor.visit_concrete_component_b(self)

    def special_method_of_concrete_component_b(self) -&gt; str:
        return &quot;B&quot;


class Visitor(ABC):
    &quot;&quot;&quot;
    Интерфейс Посетителя объявляет набор методов посещения, соответствующих
    классам компонентов. Сигнатура метода посещения позволяет посетителю
    определить конкретный класс компонента, с которым он имеет дело.
    &quot;&quot;&quot;

    @abstractmethod
    def visit_concrete_component_a(self, element: ConcreteComponentA) -&gt; None:
        pass

    @abstractmethod
    def visit_concrete_component_b(self, element: ConcreteComponentB) -&gt; None:
        pass


&quot;&quot;&quot;
Конкретные Посетители реализуют несколько версий одного и того же алгоритма,
которые могут работать со всеми классами конкретных компонентов.

Максимальную выгоду от паттерна Посетитель вы почувствуете, используя его со
сложной структурой объектов, такой как дерево Компоновщика. В этом случае было
бы полезно хранить некоторое промежуточное состояние алгоритма при выполнении
методов посетителя над различными объектами структуры.
&quot;&quot;&quot;


class ConcreteVisitor1(Visitor):
    def visit_concrete_component_a(self, element) -&gt; None:
        print(f&quot;{element.exclusive_method_of_concrete_component_a()} + ConcreteVisitor1&quot;)

    def visit_concrete_component_b(self, element) -&gt; None:
        print(f&quot;{element.special_method_of_concrete_component_b()} + ConcreteVisitor1&quot;)


class ConcreteVisitor2(Visitor):
    def visit_concrete_component_a(self, element) -&gt; None:
        print(f&quot;{element.exclusive_method_of_concrete_component_a()} + ConcreteVisitor2&quot;)

    def visit_concrete_component_b(self, element) -&gt; None:
        print(f&quot;{element.special_method_of_concrete_component_b()} + ConcreteVisitor2&quot;)


def client_code(components: List[Component], visitor: Visitor) -&gt; None:
    &quot;&quot;&quot;
    Клиентский код может выполнять операции посетителя над любым набором
    элементов, не выясняя их конкретных классов. Операция принятия направляет
    вызов к соответствующей операции в объекте посетителя.
    &quot;&quot;&quot;

    # ...
    for component in components:
        component.accept(visitor)
    # ...


if __name__ == &quot;__main__&quot;:
    components = [ConcreteComponentA(), ConcreteComponentB()]

    print(&quot;The client code works with all visitors via the base Visitor interface:&quot;)
    visitor1 = ConcreteVisitor1()
    client_code(components, visitor1)

    print(&quot;It allows the same client code to work with different types of visitors:&quot;)
    visitor2 = ConcreteVisitor2()
    client_code(components, visitor2)</code></pre>
</figure>

#### []{#example-0--Output-txt .anchor} **Output.txt:** Результат выполнения

<figure class="code">
<pre class="code" lang="output"><code>The client code works with all visitors via the base Visitor interface:
A + ConcreteVisitor1
B + ConcreteVisitor1
It allows the same client code to work with different types of visitors:
A + ConcreteVisitor2
B + ConcreteVisitor2</code></pre>
</figure>

::: next
#### Читаем дальше

[Паттерны проектирования на Ruby []{.fa
.fa-arrow-right}](../../ruby.html){.btn .btn-primary rel="next"}
:::

::: prev
#### Вернуться назад

[[]{.fa .fa-arrow-left} Шаблонный метод на
Python](../../template-method/python/example.html){.btn .btn-default
rel="prev"}
:::

## **Посетитель** на других языках программирования

[![Посетитель на
C#](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Посетитель на C#"){.prog-lang-link}
[![Посетитель на
C++](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Посетитель на C++"){.prog-lang-link}
[![Посетитель на
Go](../../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Посетитель на Go"){.prog-lang-link}
[![Посетитель на
Java](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Посетитель на Java"){.prog-lang-link}
[![Посетитель на
PHP](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Посетитель на PHP"){.prog-lang-link}
[![Посетитель на
Ruby](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Посетитель на Ruby"){.prog-lang-link}
[![Посетитель на
Rust](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Посетитель на Rust"){.prog-lang-link}
[![Посетитель на
Swift](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Посетитель на Swift"){.prog-lang-link}
[![Посетитель на
TypeScript](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Посетитель на TypeScript"){.prog-lang-link}
:::::::::::::::::

::::::: {.prom .banner-sidebar .banner .banner-striped .banner-examples .banner-removable .banner-removable-patterns data-id="DP: Examples-IDE" creative-id="examples-sidebar-ru" position="sidebar"}
:::::: {.banner-inner style="font-size: 14px; line-height: 21px;"}
::: banner-examples-img
[![](../../../../images/patterns/banners/examples-ide91ca.png?id=3115b4b548fb96b75974e2de8f4f49bc){width="215"
height="158" loading="lazy"
srcset="/images/patterns/banners/examples-ide-2x.png?id=93c007a6d157b97c28eb213f59d716ae 2x"}](../../book.html)
:::

::: banner-examples-fade
[ Архив с примерами](../../book.html){.btn .btn-secondary .btn-block}
:::

::: {.banner-examples-text style="margin-top: 1rem;"}
Купи книгу **Погружение в Паттерны** и получи архив с десятками
детальных примеров, которые можно открывать прямо в IDE.

[ Узнать больше...](../../book.html){.btn .btn-secondary .btn-block}
:::
::::::
:::::::
:::::::::::::::::::::::
::::::::::::::::::::::::
