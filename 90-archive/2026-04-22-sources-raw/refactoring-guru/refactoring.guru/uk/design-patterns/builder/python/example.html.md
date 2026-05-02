:::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [Патерни
проектування](../../../design-patterns.html){.type} /
[Будівельник](../../builder.html){.type} /
[Python](../../python.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Будівельник](../../../../images/patterns/cards/builder-mini6d24.png?id=19b95fd05e6469679752c0554b116815){srcset="/images/patterns/cards/builder-mini-2x.png?id=de6d0938678b86903a1426dddfdd13bf 2x"}
:::

# **Будівельник** на Python {#будівельник-на-python .pattern-example-title-block-title}
::::

::::::::::: pattern-example-body
::: pattern-example-brief
**Будівельник** --- це породжуючий патерн проектування, який дозволяє
створювати об'єкти покроково.

На відміну від інших породжуючих патернів, Будівельник дозволяє
виготовляти різні продукти, використовуючи один і той же процес
будівництва.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Детальніше про Будівельник ](../../builder.html){.btn .btn-lg
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

**Застосування:** Патерн можна часто зустріти в Python-коді, особливо
там, де необхідним є покрокове створення продуктів або конфігурація
складних об'єктів.

**Ознаки застосування патерна:** Будівельника можна визначити у класі,
який має один створюючий метод та декілька методів налаштування
створюваного продукту. Зазвичай, для зручності, методи налаштувань
викликають ланцюжком (наприклад,
`someBuilder.setValueA(1).setValueB(2).create()`).
:::::::::::

[]{#example-0}

## Концептуальний приклад {#example-0-title}

Цей приклад показує структуру патерна **Будівельник**, а саме --- з яких
класів він складається, які ролі ці класи виконують і як вони
взаємодіють один з одним.

#### []{#example-0--main-py .anchor} **main.py:** Приклад структури патерна

<figure class="code">
<pre class="code" lang="python"><code>from __future__ import annotations
from abc import ABC, abstractmethod
from typing import Any


class Builder(ABC):
    &quot;&quot;&quot;
    The Builder interface specifies methods for creating the different parts of
    the Product objects.
    &quot;&quot;&quot;

    @property
    @abstractmethod
    def product(self) -&gt; None:
        pass

    @abstractmethod
    def produce_part_a(self) -&gt; None:
        pass

    @abstractmethod
    def produce_part_b(self) -&gt; None:
        pass

    @abstractmethod
    def produce_part_c(self) -&gt; None:
        pass


class ConcreteBuilder1(Builder):
    &quot;&quot;&quot;
    The Concrete Builder classes follow the Builder interface and provide
    specific implementations of the building steps. Your program may have
    several variations of Builders, implemented differently.
    &quot;&quot;&quot;

    def __init__(self) -&gt; None:
        &quot;&quot;&quot;
        A fresh builder instance should contain a blank product object, which is
        used in further assembly.
        &quot;&quot;&quot;
        self.reset()

    def reset(self) -&gt; None:
        self._product = Product1()

    @property
    def product(self) -&gt; Product1:
        &quot;&quot;&quot;
        Concrete Builders are supposed to provide their own methods for
        retrieving results. That&#39;s because various types of builders may create
        entirely different products that don&#39;t follow the same interface.
        Therefore, such methods cannot be declared in the base Builder interface
        (at least in a statically typed programming language).

        Usually, after returning the end result to the client, a builder
        instance is expected to be ready to start producing another product.
        That&#39;s why it&#39;s a usual practice to call the reset method at the end of
        the `getProduct` method body. However, this behavior is not mandatory,
        and you can make your builders wait for an explicit reset call from the
        client code before disposing of the previous result.
        &quot;&quot;&quot;
        product = self._product
        self.reset()
        return product

    def produce_part_a(self) -&gt; None:
        self._product.add(&quot;PartA1&quot;)

    def produce_part_b(self) -&gt; None:
        self._product.add(&quot;PartB1&quot;)

    def produce_part_c(self) -&gt; None:
        self._product.add(&quot;PartC1&quot;)


class Product1():
    &quot;&quot;&quot;
    It makes sense to use the Builder pattern only when your products are quite
    complex and require extensive configuration.

    Unlike in other creational patterns, different concrete builders can produce
    unrelated products. In other words, results of various builders may not
    always follow the same interface.
    &quot;&quot;&quot;

    def __init__(self) -&gt; None:
        self.parts = []

    def add(self, part: Any) -&gt; None:
        self.parts.append(part)

    def list_parts(self) -&gt; None:
        print(f&quot;Product parts: {&#39;, &#39;.join(self.parts)}&quot;, end=&quot;&quot;)


class Director:
    &quot;&quot;&quot;
    The Director is only responsible for executing the building steps in a
    particular sequence. It is helpful when producing products according to a
    specific order or configuration. Strictly speaking, the Director class is
    optional, since the client can control builders directly.
    &quot;&quot;&quot;

    def __init__(self) -&gt; None:
        self._builder = None

    @property
    def builder(self) -&gt; Builder:
        return self._builder

    @builder.setter
    def builder(self, builder: Builder) -&gt; None:
        &quot;&quot;&quot;
        The Director works with any builder instance that the client code passes
        to it. This way, the client code may alter the final type of the newly
        assembled product.
        &quot;&quot;&quot;
        self._builder = builder

    &quot;&quot;&quot;
    The Director can construct several product variations using the same
    building steps.
    &quot;&quot;&quot;

    def build_minimal_viable_product(self) -&gt; None:
        self.builder.produce_part_a()

    def build_full_featured_product(self) -&gt; None:
        self.builder.produce_part_a()
        self.builder.produce_part_b()
        self.builder.produce_part_c()


if __name__ == &quot;__main__&quot;:
    &quot;&quot;&quot;
    The client code creates a builder object, passes it to the director and then
    initiates the construction process. The end result is retrieved from the
    builder object.
    &quot;&quot;&quot;

    director = Director()
    builder = ConcreteBuilder1()
    director.builder = builder

    print(&quot;Standard basic product: &quot;)
    director.build_minimal_viable_product()
    builder.product.list_parts()

    print(&quot;\n&quot;)

    print(&quot;Standard full featured product: &quot;)
    director.build_full_featured_product()
    builder.product.list_parts()

    print(&quot;\n&quot;)

    # Remember, the Builder pattern can be used without a Director class.
    print(&quot;Custom product: &quot;)
    builder.produce_part_a()
    builder.produce_part_b()
    builder.product.list_parts()</code></pre>
</figure>

#### []{#example-0--Output-txt .anchor} **Output.txt:** Результат виконання

<figure class="code">
<pre class="code" lang="output"><code>Standard basic product: 
Product parts: PartA1

Standard full featured product: 
Product parts: PartA1, PartB1, PartC1

Custom product: 
Product parts: PartA1, PartB1</code></pre>
</figure>

::: next
#### Читаємо далі

[Фабричний метод на Python []{.fa
.fa-arrow-right}](../../factory-method/python/example.html){.btn
.btn-primary rel="next"}
:::

::: prev
#### Повернутись назад

[[]{.fa .fa-arrow-left} Абстрактна фабрика на
Python](../../abstract-factory/python/example.html){.btn .btn-default
rel="prev"}
:::

## **Будівельник** іншими мовами програмування

[![Будівельник на
C#](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Будівельник на C#"){.prog-lang-link}
[![Будівельник на
C++](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Будівельник на C++"){.prog-lang-link}
[![Будівельник на
Go](../../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Будівельник на Go"){.prog-lang-link}
[![Будівельник на
Java](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Будівельник на Java"){.prog-lang-link}
[![Будівельник на
PHP](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Будівельник на PHP"){.prog-lang-link}
[![Будівельник на
Ruby](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Будівельник на Ruby"){.prog-lang-link}
[![Будівельник на
Rust](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Будівельник на Rust"){.prog-lang-link}
[![Будівельник на
Swift](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Будівельник на Swift"){.prog-lang-link}
[![Будівельник на
TypeScript](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Будівельник на TypeScript"){.prog-lang-link}
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
