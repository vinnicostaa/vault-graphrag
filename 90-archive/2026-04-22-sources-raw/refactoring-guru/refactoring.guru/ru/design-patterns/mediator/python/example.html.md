:::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [Паттерны
проектирования](../../../design-patterns.html){.type} /
[Посредник](../../mediator.html){.type} /
[Python](../../python.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Посредник](../../../../images/patterns/cards/mediator-mini1561.png?id=a7e43ee8e17e4474737b1fcb3201d7ba){srcset="/images/patterns/cards/mediator-mini-2x.png?id=d288d7c73f5581ae09701d61200ae09e 2x"}
:::

# **Посредник** на Python {#посредник-на-python .pattern-example-title-block-title}
::::

::::::::::: pattern-example-body
::: pattern-example-brief
**Посредник** --- это поведенческий паттерн, который упрощает
коммуникацию между компонентами системы.

Посредник убирает прямые связи между отдельными компонентами, заставляя
их общаться друг с другом через себя.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Подробней о паттерне Посредник ](../../mediator.html){.btn .btn-lg
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

**Применимость:** Пожалуй, самое популярное применение Посредника в
Python-коде --- это связь нескольких компонентов GUI одной программы.
:::::::::::

[]{#example-0}

## Концептуальный пример {#example-0-title}

Этот пример показывает структуру паттерна **Посредник**, а именно --- из
каких классов он состоит, какие роли эти классы выполняют и как они
взаимодействуют друг с другом.

#### []{#example-0--main-py .anchor} **main.py:** Пример структуры паттерна

<figure class="code">
<pre class="code" lang="python"><code>from __future__ import annotations
from abc import ABC


class Mediator(ABC):
    &quot;&quot;&quot;
    Интерфейс Посредника предоставляет метод, используемый компонентами для
    уведомления посредника о различных событиях. Посредник может реагировать на
    эти события и передавать исполнение другим компонентам.
    &quot;&quot;&quot;

    def notify(self, sender: object, event: str) -&gt; None:
        pass


class ConcreteMediator(Mediator):
    def __init__(self, component1: Component1, component2: Component2) -&gt; None:
        self._component1 = component1
        self._component1.mediator = self
        self._component2 = component2
        self._component2.mediator = self

    def notify(self, sender: object, event: str) -&gt; None:
        if event == &quot;A&quot;:
            print(&quot;Mediator reacts on A and triggers following operations:&quot;)
            self._component2.do_c()
        elif event == &quot;D&quot;:
            print(&quot;Mediator reacts on D and triggers following operations:&quot;)
            self._component1.do_b()
            self._component2.do_c()


class BaseComponent:
    &quot;&quot;&quot;
    Базовый Компонент обеспечивает базовую функциональность хранения экземпляра
    посредника внутри объектов компонентов.
    &quot;&quot;&quot;

    def __init__(self, mediator: Mediator = None) -&gt; None:
        self._mediator = mediator

    @property
    def mediator(self) -&gt; Mediator:
        return self._mediator

    @mediator.setter
    def mediator(self, mediator: Mediator) -&gt; None:
        self._mediator = mediator


&quot;&quot;&quot;
Конкретные Компоненты реализуют различную функциональность. Они не зависят от
других компонентов. Они также не зависят от каких-либо конкретных классов
посредников.
&quot;&quot;&quot;


class Component1(BaseComponent):
    def do_a(self) -&gt; None:
        print(&quot;Component 1 does A.&quot;)
        self.mediator.notify(self, &quot;A&quot;)

    def do_b(self) -&gt; None:
        print(&quot;Component 1 does B.&quot;)
        self.mediator.notify(self, &quot;B&quot;)


class Component2(BaseComponent):
    def do_c(self) -&gt; None:
        print(&quot;Component 2 does C.&quot;)
        self.mediator.notify(self, &quot;C&quot;)

    def do_d(self) -&gt; None:
        print(&quot;Component 2 does D.&quot;)
        self.mediator.notify(self, &quot;D&quot;)


if __name__ == &quot;__main__&quot;:
    # Клиентский код.
    c1 = Component1()
    c2 = Component2()
    mediator = ConcreteMediator(c1, c2)

    print(&quot;Client triggers operation A.&quot;)
    c1.do_a()

    print(&quot;\n&quot;, end=&quot;&quot;)

    print(&quot;Client triggers operation D.&quot;)
    c2.do_d()</code></pre>
</figure>

#### []{#example-0--Output-txt .anchor} **Output.txt:** Результат выполнения

<figure class="code">
<pre class="code" lang="output"><code>Client triggers operation A.
Component 1 does A.
Mediator reacts on A and triggers following operations:
Component 2 does C.


Client triggers operation D.
Component 2 does D.
Mediator reacts on D and triggers following operations:
Component 1 does B.
Component 2 does C.</code></pre>
</figure>

::: next
#### Читаем дальше

[Снимок на Python []{.fa
.fa-arrow-right}](../../memento/python/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Вернуться назад

[[]{.fa .fa-arrow-left} Итератор на
Python](../../iterator/python/example.html){.btn .btn-default
rel="prev"}
:::

## **Посредник** на других языках программирования

[![Посредник на
C#](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Посредник на C#"){.prog-lang-link}
[![Посредник на
C++](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Посредник на C++"){.prog-lang-link}
[![Посредник на
Go](../../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Посредник на Go"){.prog-lang-link}
[![Посредник на
Java](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Посредник на Java"){.prog-lang-link}
[![Посредник на
PHP](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Посредник на PHP"){.prog-lang-link}
[![Посредник на
Ruby](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Посредник на Ruby"){.prog-lang-link}
[![Посредник на
Rust](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Посредник на Rust"){.prog-lang-link}
[![Посредник на
Swift](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Посредник на Swift"){.prog-lang-link}
[![Посредник на
TypeScript](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Посредник на TypeScript"){.prog-lang-link}
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
