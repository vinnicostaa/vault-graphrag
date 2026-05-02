:::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [Паттерны
проектирования](../../../design-patterns.html){.type} / [Фабричный
метод](../../factory-method.html){.type} /
[Python](../../python.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Фабричный
метод](../../../../images/patterns/cards/factory-method-minid7f6.png?id=72619e9527893374b98a5913779ac167){srcset="/images/patterns/cards/factory-method-mini-2x.png?id=fa9d4a8d61a67cc3822e52b9daf69dad 2x"}
:::

# **Фабричный метод** на Python {#фабричный-метод-на-python .pattern-example-title-block-title}
::::

::::::::::: pattern-example-body
::: pattern-example-brief
**Фабричный метод** --- это порождающий паттерн проектирования, который
решает проблему создания различных продуктов, без указания конкретных
классов продуктов.

Фабричный метод задаёт метод, который следует использовать вместо вызова
оператора `new` для создания объектов-продуктов. Подклассы могут
переопределить этот метод, чтобы изменять тип создаваемых продуктов.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Подробней о паттерне Фабричный метод ](../../factory-method.html){.btn
.btn-lg .btn-primary}
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

**Применимость:** Паттерн можно часто встретить в любом Python-коде, где
требуется гибкость при создании продуктов.

**Признаки применения паттерна:** Фабричный метод можно определить по
создающим методам, которые возвращают объекты продуктов через
абстрактные типы или интерфейсы. Это позволяет переопределять типы
создаваемых продуктов в подклассах.
:::::::::::

[]{#example-0}

## Концептуальный пример {#example-0-title}

Этот пример показывает структуру паттерна **Фабричный метод**, а
именно --- из каких классов он состоит, какие роли эти классы выполняют
и как они взаимодействуют друг с другом.

#### []{#example-0--main-py .anchor} **main.py:** Пример структуры паттерна

<figure class="code">
<pre class="code" lang="python"><code>from __future__ import annotations
from abc import ABC, abstractmethod


class Creator(ABC):
    &quot;&quot;&quot;
    Класс Создатель объявляет фабричный метод, который должен возвращать объект
    класса Продукт. Подклассы Создателя обычно предоставляют реализацию этого
    метода.
    &quot;&quot;&quot;

    @abstractmethod
    def factory_method(self):
        &quot;&quot;&quot;
        Обратите внимание, что Создатель может также обеспечить реализацию
        фабричного метода по умолчанию.
        &quot;&quot;&quot;
        pass

    def some_operation(self) -&gt; str:
        &quot;&quot;&quot;
        Также заметьте, что, несмотря на название, основная обязанность
        Создателя не заключается в создании продуктов. Обычно он содержит
        некоторую базовую бизнес-логику, которая основана на объектах Продуктов,
        возвращаемых фабричным методом. Подклассы могут косвенно изменять эту
        бизнес-логику, переопределяя фабричный метод и возвращая из него другой
        тип продукта.
        &quot;&quot;&quot;

        # Вызываем фабричный метод, чтобы получить объект-продукт.
        product = self.factory_method()

        # Далее, работаем с этим продуктом.
        result = f&quot;Creator: The same creator&#39;s code has just worked with {product.operation()}&quot;

        return result


&quot;&quot;&quot;
Конкретные Создатели переопределяют фабричный метод для того, чтобы изменить тип
результирующего продукта.
&quot;&quot;&quot;


class ConcreteCreator1(Creator):
    &quot;&quot;&quot;
    Обратите внимание, что сигнатура метода по-прежнему использует тип
    абстрактного продукта, хотя фактически из метода возвращается конкретный
    продукт. Таким образом, Создатель может оставаться независимым от конкретных
    классов продуктов.
    &quot;&quot;&quot;

    def factory_method(self) -&gt; Product:
        return ConcreteProduct1()


class ConcreteCreator2(Creator):
    def factory_method(self) -&gt; Product:
        return ConcreteProduct2()


class Product(ABC):
    &quot;&quot;&quot;
    Интерфейс Продукта объявляет операции, которые должны выполнять все
    конкретные продукты.
    &quot;&quot;&quot;

    @abstractmethod
    def operation(self) -&gt; str:
        pass


&quot;&quot;&quot;
Конкретные Продукты предоставляют различные реализации интерфейса Продукта.
&quot;&quot;&quot;


class ConcreteProduct1(Product):
    def operation(self) -&gt; str:
        return &quot;{Result of the ConcreteProduct1}&quot;


class ConcreteProduct2(Product):
    def operation(self) -&gt; str:
        return &quot;{Result of the ConcreteProduct2}&quot;


def client_code(creator: Creator) -&gt; None:
    &quot;&quot;&quot;
    Клиентский код работает с экземпляром конкретного создателя, хотя и через
    его базовый интерфейс. Пока клиент продолжает работать с создателем через
    базовый интерфейс, вы можете передать ему любой подкласс создателя.
    &quot;&quot;&quot;

    print(f&quot;Client: I&#39;m not aware of the creator&#39;s class, but it still works.\n&quot;
          f&quot;{creator.some_operation()}&quot;, end=&quot;&quot;)


if __name__ == &quot;__main__&quot;:
    print(&quot;App: Launched with the ConcreteCreator1.&quot;)
    client_code(ConcreteCreator1())
    print(&quot;\n&quot;)

    print(&quot;App: Launched with the ConcreteCreator2.&quot;)
    client_code(ConcreteCreator2())</code></pre>
</figure>

#### []{#example-0--Output-txt .anchor} **Output.txt:** Результат выполнения

<figure class="code">
<pre class="code" lang="output"><code>App: Launched with the ConcreteCreator1.
Client: I&#39;m not aware of the creator&#39;s class, but it still works.
Creator: The same creator&#39;s code has just worked with {Result of the ConcreteProduct1}

App: Launched with the ConcreteCreator2.
Client: I&#39;m not aware of the creator&#39;s class, but it still works.
Creator: The same creator&#39;s code has just worked with {Result of the ConcreteProduct2}</code></pre>
</figure>

::: next
#### Читаем дальше

[Прототип на Python []{.fa
.fa-arrow-right}](../../prototype/python/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Вернуться назад

[[]{.fa .fa-arrow-left} Строитель на
Python](../../builder/python/example.html){.btn .btn-default rel="prev"}
:::

## **Фабричный метод** на других языках программирования

[![Фабричный метод на
C#](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Фабричный метод на C#"){.prog-lang-link}
[![Фабричный метод на
C++](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Фабричный метод на C++"){.prog-lang-link}
[![Фабричный метод на
Go](../../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Фабричный метод на Go"){.prog-lang-link}
[![Фабричный метод на
Java](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Фабричный метод на Java"){.prog-lang-link}
[![Фабричный метод на
PHP](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Фабричный метод на PHP"){.prog-lang-link}
[![Фабричный метод на
Ruby](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Фабричный метод на Ruby"){.prog-lang-link}
[![Фабричный метод на
Rust](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Фабричный метод на Rust"){.prog-lang-link}
[![Фабричный метод на
Swift](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Фабричный метод на Swift"){.prog-lang-link}
[![Фабричный метод на
TypeScript](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Фабричный метод на TypeScript"){.prog-lang-link}
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
