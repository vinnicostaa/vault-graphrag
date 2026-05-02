:::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [Паттерны
проектирования](../../../design-patterns.html){.type} /
[Мост](../../bridge.html){.type} / [Python](../../python.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Мост](../../../../images/patterns/cards/bridge-mini9e60.png?id=b389101d8ee8e23ffa1b534c704d0774){srcset="/images/patterns/cards/bridge-mini-2x.png?id=2622384cf623ed150ee9c21a0812dd87 2x"}
:::

# **Мост** на Python {#мост-на-python .pattern-example-title-block-title}
::::

::::::::::: pattern-example-body
::: pattern-example-brief
**Мост** --- это структурный паттерн, который разделяет бизнес-логику
или большой класс на несколько отдельных иерархий, которые потом можно
развивать отдельно друг от друга.

Одна из этих иерархий (абстракция) получит ссылку на объекты другой
иерархии (реализация) и будет делегировать им основную работу. Благодаря
тому, что все реализации будут следовать общему интерфейсу, их можно
будет взаимозаменять внутри абстракции.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Подробней о паттерне Мост ](../../bridge.html){.btn .btn-lg
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

**Применимость:** Паттерн Мост особенно полезен когда вам приходится
делать кросс-платформенные приложения, поддерживать несколько типов баз
данных или работать с разными поставщиками похожего API (например,
cloud-сервисы, социальные сети и т. д.)

**Признаки применения паттерна:** Если в программе чётко выделены классы
«управления» и несколько видов классов «платформ», причём управляющие
объекты делегируют выполнение платформам, то можно сказать, что у вас
используется Мост.
:::::::::::

[]{#example-0}

## Концептуальный пример {#example-0-title}

Этот пример показывает структуру паттерна **Мост**, а именно --- из
каких классов он состоит, какие роли эти классы выполняют и как они
взаимодействуют друг с другом.

#### []{#example-0--main-py .anchor} **main.py:** Пример структуры паттерна

<figure class="code">
<pre class="code" lang="python"><code>from __future__ import annotations
from abc import ABC, abstractmethod


class Abstraction:
    &quot;&quot;&quot;
    Абстракция устанавливает интерфейс для «управляющей» части двух иерархий
    классов. Она содержит ссылку на объект из иерархии Реализации и делегирует
    ему всю настоящую работу.
    &quot;&quot;&quot;

    def __init__(self, implementation: Implementation) -&gt; None:
        self.implementation = implementation

    def operation(self) -&gt; str:
        return (f&quot;Abstraction: Base operation with:\n&quot;
                f&quot;{self.implementation.operation_implementation()}&quot;)


class ExtendedAbstraction(Abstraction):
    &quot;&quot;&quot;
    Можно расширить Абстракцию без изменения классов Реализации.
    &quot;&quot;&quot;

    def operation(self) -&gt; str:
        return (f&quot;ExtendedAbstraction: Extended operation with:\n&quot;
                f&quot;{self.implementation.operation_implementation()}&quot;)


class Implementation(ABC):
    &quot;&quot;&quot;
    Реализация устанавливает интерфейс для всех классов реализации. Он не должен
    соответствовать интерфейсу Абстракции. На практике оба интерфейса могут быть
    совершенно разными. Как правило, интерфейс Реализации предоставляет только
    примитивные операции, в то время как Абстракция определяет операции более
    высокого уровня, основанные на этих примитивах.
    &quot;&quot;&quot;

    @abstractmethod
    def operation_implementation(self) -&gt; str:
        pass


&quot;&quot;&quot;
Каждая Конкретная Реализация соответствует определённой платформе и реализует
интерфейс Реализации с использованием API этой платформы.
&quot;&quot;&quot;


class ConcreteImplementationA(Implementation):
    def operation_implementation(self) -&gt; str:
        return &quot;ConcreteImplementationA: Here&#39;s the result on the platform A.&quot;


class ConcreteImplementationB(Implementation):
    def operation_implementation(self) -&gt; str:
        return &quot;ConcreteImplementationB: Here&#39;s the result on the platform B.&quot;


def client_code(abstraction: Abstraction) -&gt; None:
    &quot;&quot;&quot;
    За исключением этапа инициализации, когда объект Абстракции связывается с
    определённым объектом Реализации, клиентский код должен зависеть только от
    класса Абстракции. Таким образом, клиентский код может поддерживать любую
    комбинацию абстракции и реализации.
    &quot;&quot;&quot;

    # ...

    print(abstraction.operation(), end=&quot;&quot;)

    # ...


if __name__ == &quot;__main__&quot;:
    &quot;&quot;&quot;
    Клиентский код должен работать с любой предварительно сконфигурированной
    комбинацией абстракции и реализации.
    &quot;&quot;&quot;

    implementation = ConcreteImplementationA()
    abstraction = Abstraction(implementation)
    client_code(abstraction)

    print(&quot;\n&quot;)

    implementation = ConcreteImplementationB()
    abstraction = ExtendedAbstraction(implementation)
    client_code(abstraction)</code></pre>
</figure>

#### []{#example-0--Output-txt .anchor} **Output.txt:** Результат выполнения

<figure class="code">
<pre class="code" lang="output"><code>Abstraction: Base operation with:
ConcreteImplementationA: Here&#39;s the result on the platform A.

ExtendedAbstraction: Extended operation with:
ConcreteImplementationB: Here&#39;s the result on the platform B.</code></pre>
</figure>

::: next
#### Читаем дальше

[Компоновщик на Python []{.fa
.fa-arrow-right}](../../composite/python/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Вернуться назад

[[]{.fa .fa-arrow-left} Адаптер на
Python](../../adapter/python/example.html){.btn .btn-default rel="prev"}
:::

## **Мост** на других языках программирования

[![Мост на
C#](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Мост на C#"){.prog-lang-link}
[![Мост на
C++](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Мост на C++"){.prog-lang-link}
[![Мост на
Go](../../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Мост на Go"){.prog-lang-link}
[![Мост на
Java](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Мост на Java"){.prog-lang-link}
[![Мост на
PHP](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Мост на PHP"){.prog-lang-link}
[![Мост на
Ruby](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Мост на Ruby"){.prog-lang-link}
[![Мост на
Rust](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Мост на Rust"){.prog-lang-link}
[![Мост на
Swift](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Мост на Swift"){.prog-lang-link}
[![Мост на
TypeScript](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Мост на TypeScript"){.prog-lang-link}
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
