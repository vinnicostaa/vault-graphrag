:::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [Патерни
проектування](../../../design-patterns.html){.type} /
[Стратегія](../../strategy.html){.type} /
[Python](../../python.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Стратегія](../../../../images/patterns/cards/strategy-mini28f6.png?id=d38abee4fb6f2aed909d262bdadca936){srcset="/images/patterns/cards/strategy-mini-2x.png?id=f4e6608561f8e5d18be6927d4620ad29 2x"}
:::

# **Стратегія** на Python {#стратегія-на-python .pattern-example-title-block-title}
::::

::::::::::: pattern-example-body
::: pattern-example-brief
**Стратегія** --- це поведінковий патерн, який виносить набір алгоритмів
у власні класи і робить їх взаємозамінними.

Інші об'єкти містять посилання на об'єкт-стратегію та делегують їй
роботу. Програма може підмінити цей об'єкт іншим, якщо потрібен інший
спосіб вирішення завдання.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Детальніше про Стратегія ](../../strategy.html){.btn .btn-lg
.btn-primary}
:::

:::::::: {.sidebar-navigation .with-scroll}
::: en-title
Навігація
:::

::: en-intro
 [Інтро](#)
:::

::: en-intro
 [Концептуальний приклад](#example-0)
:::

::: en-file
 [main](#example-0--main-py)
:::

::: en-file
 [Output](#example-0--Output-txt)
:::
::::::::

**Складність:** []{.fa-stars}

**Популярність:** []{.fa-stars}

**Застосування:** Стратегія часто використовується в Python-коді,
особливо там, де потрібно підміняти алгоритм під час виконання програми.
Багато прикладів стратегії можна замінити простими lambda-виразами.

**Ознаки застосування патерна:** Клас делегує виконання вкладеному
об'єктові абстрактного типу чи інтерфейсу.
:::::::::::

[]{#example-0}

## Концептуальний приклад {#example-0-title}

Цей приклад показує структуру патерна **Стратегія**, а саме --- з яких
класів він складається, які ролі ці класи виконують і як вони
взаємодіють один з одним.

#### []{#example-0--main-py .anchor} **main.py:** Приклад структури патерна

<figure class="code">
<pre class="code" lang="python"><code>from __future__ import annotations
from abc import ABC, abstractmethod
from typing import List


class Context():
    &quot;&quot;&quot;
    The Context defines the interface of interest to clients.
    &quot;&quot;&quot;

    def __init__(self, strategy: Strategy) -&gt; None:
        &quot;&quot;&quot;
        Usually, the Context accepts a strategy through the constructor, but
        also provides a setter to change it at runtime.
        &quot;&quot;&quot;

        self._strategy = strategy

    @property
    def strategy(self) -&gt; Strategy:
        &quot;&quot;&quot;
        The Context maintains a reference to one of the Strategy objects. The
        Context does not know the concrete class of a strategy. It should work
        with all strategies via the Strategy interface.
        &quot;&quot;&quot;

        return self._strategy

    @strategy.setter
    def strategy(self, strategy: Strategy) -&gt; None:
        &quot;&quot;&quot;
        Usually, the Context allows replacing a Strategy object at runtime.
        &quot;&quot;&quot;

        self._strategy = strategy

    def do_some_business_logic(self) -&gt; None:
        &quot;&quot;&quot;
        The Context delegates some work to the Strategy object instead of
        implementing multiple versions of the algorithm on its own.
        &quot;&quot;&quot;

        # ...

        print(&quot;Context: Sorting data using the strategy (not sure how it&#39;ll do it)&quot;)
        result = self._strategy.do_algorithm([&quot;a&quot;, &quot;b&quot;, &quot;c&quot;, &quot;d&quot;, &quot;e&quot;])
        print(&quot;,&quot;.join(result))

        # ...


class Strategy(ABC):
    &quot;&quot;&quot;
    The Strategy interface declares operations common to all supported versions
    of some algorithm.

    The Context uses this interface to call the algorithm defined by Concrete
    Strategies.
    &quot;&quot;&quot;

    @abstractmethod
    def do_algorithm(self, data: List):
        pass


&quot;&quot;&quot;
Concrete Strategies implement the algorithm while following the base Strategy
interface. The interface makes them interchangeable in the Context.
&quot;&quot;&quot;


class ConcreteStrategyA(Strategy):
    def do_algorithm(self, data: List) -&gt; List:
        return sorted(data)


class ConcreteStrategyB(Strategy):
    def do_algorithm(self, data: List) -&gt; List:
        return reversed(sorted(data))


if __name__ == &quot;__main__&quot;:
    # The client code picks a concrete strategy and passes it to the context.
    # The client should be aware of the differences between strategies in order
    # to make the right choice.

    context = Context(ConcreteStrategyA())
    print(&quot;Client: Strategy is set to normal sorting.&quot;)
    context.do_some_business_logic()
    print()

    print(&quot;Client: Strategy is set to reverse sorting.&quot;)
    context.strategy = ConcreteStrategyB()
    context.do_some_business_logic()</code></pre>
</figure>

#### []{#example-0--Output-txt .anchor} **Output.txt:** Результат виконання

<figure class="code">
<pre class="code" lang="output"><code>Client: Strategy is set to normal sorting.
Context: Sorting data using the strategy (not sure how it&#39;ll do it)
a,b,c,d,e

Client: Strategy is set to reverse sorting.
Context: Sorting data using the strategy (not sure how it&#39;ll do it)
e,d,c,b,a</code></pre>
</figure>

::: next
#### Читаємо далі

[Шаблонний метод на Python []{.fa
.fa-arrow-right}](../../template-method/python/example.html){.btn
.btn-primary rel="next"}
:::

::: prev
#### Повернутись назад

[[]{.fa .fa-arrow-left} Стан на
Python](../../state/python/example.html){.btn .btn-default rel="prev"}
:::

## **Стратегія** іншими мовами програмування

[![Стратегія на
C#](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Стратегія на C#"){.prog-lang-link}
[![Стратегія на
C++](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Стратегія на C++"){.prog-lang-link}
[![Стратегія на
Go](../../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Стратегія на Go"){.prog-lang-link}
[![Стратегія на
Java](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Стратегія на Java"){.prog-lang-link}
[![Стратегія на
PHP](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Стратегія на PHP"){.prog-lang-link}
[![Стратегія на
Ruby](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Стратегія на Ruby"){.prog-lang-link}
[![Стратегія на
Rust](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Стратегія на Rust"){.prog-lang-link}
[![Стратегія на
Swift](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Стратегія на Swift"){.prog-lang-link}
[![Стратегія на
TypeScript](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Стратегія на TypeScript"){.prog-lang-link}
:::::::::::::::::

::::::: {.prom .banner-sidebar .banner .banner-striped .banner-examples .banner-removable .banner-removable-patterns data-id="DP: Examples-IDE" creative-id="examples-sidebar-uk" position="sidebar"}
:::::: {.banner-inner style="font-size: 14px; line-height: 21px;"}
::: banner-examples-img
[![](../../../../images/patterns/banners/examples-ide91ca.png?id=3115b4b548fb96b75974e2de8f4f49bc){width="215"
height="158" loading="lazy"
srcset="/images/patterns/banners/examples-ide-2x.png?id=93c007a6d157b97c28eb213f59d716ae 2x"}](../../book.html)
:::

::: banner-examples-fade
[ Архів з прикладами](../../book.html){.btn .btn-secondary .btn-block}
:::

::: {.banner-examples-text style="margin-top: 1rem;"}
Придбай книгу **Занурення в Патерни** та отримай архів з десятками
детальних прикладів, які можна відкривати прямо в IDE.

[ Дізнатися більше...](../../book.html){.btn .btn-secondary .btn-block}
:::
::::::
:::::::
:::::::::::::::::::::::
::::::::::::::::::::::::
