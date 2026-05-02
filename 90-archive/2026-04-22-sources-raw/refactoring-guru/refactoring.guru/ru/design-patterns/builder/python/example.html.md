:::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [Паттерны
проектирования](../../../design-patterns.html){.type} /
[Строитель](../../builder.html){.type} /
[Python](../../python.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Строитель](../../../../images/patterns/cards/builder-mini6d24.png?id=19b95fd05e6469679752c0554b116815){srcset="/images/patterns/cards/builder-mini-2x.png?id=de6d0938678b86903a1426dddfdd13bf 2x"}
:::

# **Строитель** на Python {#строитель-на-python .pattern-example-title-block-title}
::::

::::::::::: pattern-example-body
::: pattern-example-brief
**Строитель** --- это порождающий паттерн проектирования, который
позволяет создавать объекты пошагово.

В отличие от других порождающих паттернов, Строитель позволяет
производить различные продукты, используя один и тот же процесс
строительства.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Подробней о паттерне Строитель ](../../builder.html){.btn .btn-lg
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

**Применимость:** Паттерн можно часто встретить в Python-коде, особенно
там, где требуется пошаговое создание продуктов или конфигурация сложных
объектов.

**Признаки применения паттерна:** Строителя можно узнать в классе,
который имеет один создающий метод и несколько методов настройки
создаваемого продукта. Обычно, методы настройки вызывают для удобства
цепочкой (например, `someBuilder.setValueA(1).setValueB(2).create()`).
:::::::::::

[]{#example-0}

## Концептуальный пример {#example-0-title}

Этот пример показывает структуру паттерна **Строитель**, а именно --- из
каких классов он состоит, какие роли эти классы выполняют и как они
взаимодействуют друг с другом.

#### []{#example-0--main-py .anchor} **main.py:** Пример структуры паттерна

<figure class="code">
<pre class="code" lang="python"><code>from __future__ import annotations
from abc import ABC, abstractmethod
from typing import Any


class Builder(ABC):
    &quot;&quot;&quot;
    Интерфейс Строителя объявляет создающие методы для различных частей объектов
    Продуктов.
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
    Классы Конкретного Строителя следуют интерфейсу Строителя и предоставляют
    конкретные реализации шагов построения. Ваша программа может иметь несколько
    вариантов Строителей, реализованных по-разному.
    &quot;&quot;&quot;

    def __init__(self) -&gt; None:
        &quot;&quot;&quot;
        Новый экземпляр строителя должен содержать пустой объект продукта,
        который используется в дальнейшей сборке.
        &quot;&quot;&quot;
        self.reset()

    def reset(self) -&gt; None:
        self._product = Product1()

    @property
    def product(self) -&gt; Product1:
        &quot;&quot;&quot;
        Конкретные Строители должны предоставить свои собственные методы
        получения результатов. Это связано с тем, что различные типы строителей
        могут создавать совершенно разные продукты с разными интерфейсами.
        Поэтому такие методы не могут быть объявлены в базовом интерфейсе
        Строителя (по крайней мере, в статически типизированном языке
        программирования).

        Как правило, после возвращения конечного результата клиенту, экземпляр
        строителя должен быть готов к началу производства следующего продукта.
        Поэтому обычной практикой является вызов метода сброса в конце тела
        метода getProduct. Однако такое поведение не является обязательным, вы
        можете заставить своих строителей ждать явного запроса на сброс из кода
        клиента, прежде чем избавиться от предыдущего результата.
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
    Имеет смысл использовать паттерн Строитель только тогда, когда ваши продукты
    достаточно сложны и требуют обширной конфигурации.

    В отличие от других порождающих паттернов, различные конкретные строители
    могут производить несвязанные продукты. Другими словами, результаты
    различных строителей могут не всегда следовать одному и тому же интерфейсу.
    &quot;&quot;&quot;

    def __init__(self) -&gt; None:
        self.parts = []

    def add(self, part: Any) -&gt; None:
        self.parts.append(part)

    def list_parts(self) -&gt; None:
        print(f&quot;Product parts: {&#39;, &#39;.join(self.parts)}&quot;, end=&quot;&quot;)


class Director:
    &quot;&quot;&quot;
    Директор отвечает только за выполнение шагов построения в определённой
    последовательности. Это полезно при производстве продуктов в определённом
    порядке или особой конфигурации. Строго говоря, класс Директор необязателен,
    так как клиент может напрямую управлять строителями.
    &quot;&quot;&quot;

    def __init__(self) -&gt; None:
        self._builder = None

    @property
    def builder(self) -&gt; Builder:
        return self._builder

    @builder.setter
    def builder(self, builder: Builder) -&gt; None:
        &quot;&quot;&quot;
        Директор работает с любым экземпляром строителя, который передаётся ему
        клиентским кодом. Таким образом, клиентский код может изменить конечный
        тип вновь собираемого продукта.
        &quot;&quot;&quot;
        self._builder = builder

    &quot;&quot;&quot;
    Директор может строить несколько вариаций продукта, используя одинаковые
    шаги построения.
    &quot;&quot;&quot;

    def build_minimal_viable_product(self) -&gt; None:
        self.builder.produce_part_a()

    def build_full_featured_product(self) -&gt; None:
        self.builder.produce_part_a()
        self.builder.produce_part_b()
        self.builder.produce_part_c()


if __name__ == &quot;__main__&quot;:
    &quot;&quot;&quot;
    Клиентский код создаёт объект-строитель, передаёт его директору, а затем
    инициирует процесс построения. Конечный результат извлекается из объекта-
    строителя.
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

    # Помните, что паттерн Строитель можно использовать без класса Директор.
    print(&quot;Custom product: &quot;)
    builder.produce_part_a()
    builder.produce_part_b()
    builder.product.list_parts()</code></pre>
</figure>

#### []{#example-0--Output-txt .anchor} **Output.txt:** Результат выполнения

<figure class="code">
<pre class="code" lang="output"><code>Standard basic product: 
Product parts: PartA1

Standard full featured product: 
Product parts: PartA1, PartB1, PartC1

Custom product: 
Product parts: PartA1, PartB1</code></pre>
</figure>

::: next
#### Читаем дальше

[Фабричный метод на Python []{.fa
.fa-arrow-right}](../../factory-method/python/example.html){.btn
.btn-primary rel="next"}
:::

::: prev
#### Вернуться назад

[[]{.fa .fa-arrow-left} Абстрактная фабрика на
Python](../../abstract-factory/python/example.html){.btn .btn-default
rel="prev"}
:::

## **Строитель** на других языках программирования

[![Строитель на
C#](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Строитель на C#"){.prog-lang-link}
[![Строитель на
C++](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Строитель на C++"){.prog-lang-link}
[![Строитель на
Go](../../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Строитель на Go"){.prog-lang-link}
[![Строитель на
Java](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Строитель на Java"){.prog-lang-link}
[![Строитель на
PHP](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Строитель на PHP"){.prog-lang-link}
[![Строитель на
Ruby](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Строитель на Ruby"){.prog-lang-link}
[![Строитель на
Rust](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Строитель на Rust"){.prog-lang-link}
[![Строитель на
Swift](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Строитель на Swift"){.prog-lang-link}
[![Строитель на
TypeScript](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Строитель на TypeScript"){.prog-lang-link}
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
