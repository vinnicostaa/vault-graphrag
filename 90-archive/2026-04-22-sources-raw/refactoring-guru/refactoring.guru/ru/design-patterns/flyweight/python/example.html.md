:::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [Паттерны
проектирования](../../../design-patterns.html){.type} /
[Легковес](../../flyweight.html){.type} /
[Python](../../python.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Легковес](../../../../images/patterns/cards/flyweight-minie3d8.png?id=422ca8d2f90614dce810a8812c626698){srcset="/images/patterns/cards/flyweight-mini-2x.png?id=857ca676fff84317bb0dab76abfce08e 2x"}
:::

# **Легковес** на Python {#легковес-на-python .pattern-example-title-block-title}
::::

::::::::::: pattern-example-body
::: pattern-example-brief
**Легковес** --- это структурный паттерн, который экономит память,
благодаря разделению общего состояния, вынесенного в один объект, между
множеством объектов.

Легковес позволяет экономить память, кешируя одинаковые данные,
используемые в разных объектах.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Подробней о паттерне Легковес ](../../flyweight.html){.btn .btn-lg
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

**Применимость:** Весь смысл использования Легковеса --- в экономии
памяти. Поэтому, если в приложении нет такой проблемы, то вы вряд ли
найдёте там примеры Легковеса.

**Признаки применения паттерна:** Легковес можно определить по создающим
методам класса, которые возвращают закешированные объекты, вместо
создания новых.
:::::::::::

[]{#example-0}

## Концептуальный пример {#example-0-title}

Этот пример показывает структуру паттерна **Легковес**, а именно --- из
каких классов он состоит, какие роли эти классы выполняют и как они
взаимодействуют друг с другом.

#### []{#example-0--main-py .anchor} **main.py:** Пример структуры паттерна

<figure class="code">
<pre class="code" lang="python"><code>import json
from typing import Dict


class Flyweight():
    &quot;&quot;&quot;
    Легковес хранит общую часть состояния (также называемую внутренним
    состоянием), которая принадлежит нескольким реальным бизнес-объектам.
    Легковес принимает оставшуюся часть состояния (внешнее состояние, уникальное
    для каждого объекта) через его параметры метода.
    &quot;&quot;&quot;

    def __init__(self, shared_state: str) -&gt; None:
        self._shared_state = shared_state

    def operation(self, unique_state: str) -&gt; None:
        s = json.dumps(self._shared_state)
        u = json.dumps(unique_state)
        print(f&quot;Flyweight: Displaying shared ({s}) and unique ({u}) state.&quot;, end=&quot;&quot;)


class FlyweightFactory():
    &quot;&quot;&quot;
    Фабрика Легковесов создает объекты-Легковесы и управляет ими. Она
    обеспечивает правильное разделение легковесов. Когда клиент запрашивает
    легковес, фабрика либо возвращает существующий экземпляр, либо создает
    новый, если он ещё не существует.
    &quot;&quot;&quot;

    _flyweights: Dict[str, Flyweight] = {}

    def __init__(self, initial_flyweights: Dict) -&gt; None:
        for state in initial_flyweights:
            self._flyweights[self.get_key(state)] = Flyweight(state)

    def get_key(self, state: Dict) -&gt; str:
        &quot;&quot;&quot;
        Возвращает хеш строки Легковеса для данного состояния.
        &quot;&quot;&quot;

        return &quot;_&quot;.join(sorted(state))

    def get_flyweight(self, shared_state: Dict) -&gt; Flyweight:
        &quot;&quot;&quot;
        Возвращает существующий Легковес с заданным состоянием или создает
        новый.
        &quot;&quot;&quot;

        key = self.get_key(shared_state)

        if not self._flyweights.get(key):
            print(&quot;FlyweightFactory: Can&#39;t find a flyweight, creating new one.&quot;)
            self._flyweights[key] = Flyweight(shared_state)
        else:
            print(&quot;FlyweightFactory: Reusing existing flyweight.&quot;)

        return self._flyweights[key]

    def list_flyweights(self) -&gt; None:
        count = len(self._flyweights)
        print(f&quot;FlyweightFactory: I have {count} flyweights:&quot;)
        print(&quot;\n&quot;.join(map(str, self._flyweights.keys())), end=&quot;&quot;)


def add_car_to_police_database(
    factory: FlyweightFactory, plates: str, owner: str,
    brand: str, model: str, color: str
) -&gt; None:
    print(&quot;\n\nClient: Adding a car to database.&quot;)
    flyweight = factory.get_flyweight([brand, model, color])
    # Клиентский код либо сохраняет, либо вычисляет внешнее состояние и передает
    # его методам легковеса.
    flyweight.operation([plates, owner])


if __name__ == &quot;__main__&quot;:
    &quot;&quot;&quot;
    Клиентский код обычно создает кучу предварительно заполненных легковесов на
    этапе инициализации приложения.
    &quot;&quot;&quot;

    factory = FlyweightFactory([
        [&quot;Chevrolet&quot;, &quot;Camaro2018&quot;, &quot;pink&quot;],
        [&quot;Mercedes Benz&quot;, &quot;C300&quot;, &quot;black&quot;],
        [&quot;Mercedes Benz&quot;, &quot;C500&quot;, &quot;red&quot;],
        [&quot;BMW&quot;, &quot;M5&quot;, &quot;red&quot;],
        [&quot;BMW&quot;, &quot;X6&quot;, &quot;white&quot;],
    ])

    factory.list_flyweights()

    add_car_to_police_database(
        factory, &quot;CL234IR&quot;, &quot;James Doe&quot;, &quot;BMW&quot;, &quot;M5&quot;, &quot;red&quot;)

    add_car_to_police_database(
        factory, &quot;CL234IR&quot;, &quot;James Doe&quot;, &quot;BMW&quot;, &quot;X1&quot;, &quot;red&quot;)

    print(&quot;\n&quot;)

    factory.list_flyweights()</code></pre>
</figure>

#### []{#example-0--Output-txt .anchor} **Output.txt:** Результат выполнения

<figure class="code">
<pre class="code" lang="output"><code>FlyweightFactory: I have 5 flyweights:
Camaro2018_Chevrolet_pink
C300_Mercedes Benz_black
C500_Mercedes Benz_red
BMW_M5_red
BMW_X6_white

Client: Adding a car to database.
FlyweightFactory: Reusing existing flyweight.
Flyweight: Displaying shared ([&quot;BMW&quot;, &quot;M5&quot;, &quot;red&quot;]) and unique ([&quot;CL234IR&quot;, &quot;James Doe&quot;]) state.

Client: Adding a car to database.
FlyweightFactory: Can&#39;t find a flyweight, creating new one.
Flyweight: Displaying shared ([&quot;BMW&quot;, &quot;X1&quot;, &quot;red&quot;]) and unique ([&quot;CL234IR&quot;, &quot;James Doe&quot;]) state.

FlyweightFactory: I have 6 flyweights:
Camaro2018_Chevrolet_pink
C300_Mercedes Benz_black
C500_Mercedes Benz_red
BMW_M5_red
BMW_X6_white
BMW_X1_red</code></pre>
</figure>

::: next
#### Читаем дальше

[Заместитель на Python []{.fa
.fa-arrow-right}](../../proxy/python/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Вернуться назад

[[]{.fa .fa-arrow-left} Фасад на
Python](../../facade/python/example.html){.btn .btn-default rel="prev"}
:::

## **Легковес** на других языках программирования

[![Легковес на
C#](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Легковес на C#"){.prog-lang-link}
[![Легковес на
C++](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Легковес на C++"){.prog-lang-link}
[![Легковес на
Go](../../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Легковес на Go"){.prog-lang-link}
[![Легковес на
Java](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Легковес на Java"){.prog-lang-link}
[![Легковес на
PHP](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Легковес на PHP"){.prog-lang-link}
[![Легковес на
Ruby](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Легковес на Ruby"){.prog-lang-link}
[![Легковес на
Rust](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Легковес на Rust"){.prog-lang-link}
[![Легковес на
Swift](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Легковес на Swift"){.prog-lang-link}
[![Легковес на
TypeScript](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Легковес на TypeScript"){.prog-lang-link}
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
