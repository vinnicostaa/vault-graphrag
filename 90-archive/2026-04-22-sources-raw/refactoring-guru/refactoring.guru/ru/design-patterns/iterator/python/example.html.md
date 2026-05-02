:::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [Паттерны
проектирования](../../../design-patterns.html){.type} /
[Итератор](../../iterator.html){.type} /
[Python](../../python.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Итератор](../../../../images/patterns/cards/iterator-minie2c2.png?id=76c28bb48f997b36965983dd2b41f02e){srcset="/images/patterns/cards/iterator-mini-2x.png?id=7d28f66a487066774a743cfceaa40ea4 2x"}
:::

# **Итератор** на Python {#итератор-на-python .pattern-example-title-block-title}
::::

::::::::::: pattern-example-body
::: pattern-example-brief
**Итератор** --- это поведенческий паттерн, позволяющий последовательно
обходить сложную коллекцию, без раскрытия деталей её реализации.

Благодаря Итератору, клиент может обходить разные коллекции одним и тем
же способом, используя единый интерфейс итераторов.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Подробней о паттерне Итератор ](../../iterator.html){.btn .btn-lg
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
в программах, работающих с разными типами коллекций, и где требуется
обход разных сущностей.

**Признаки применения паттерна:** Итератор легко определить по методам
навигации (например, получения следующего/предыдущего элемента и т. д.).
Код использующий итератор зачастую вообще не имеет ссылок на коллекцию,
с которой работает итератор. Итератор либо принимает коллекцию в
параметрах конструктора при создании, либо возвращается самой
коллекцией.
:::::::::::

[]{#example-0}

## Концептуальный пример {#example-0-title}

Этот пример показывает структуру паттерна **Итератор**, а именно --- из
каких классов он состоит, какие роли эти классы выполняют и как они
взаимодействуют друг с другом.

#### []{#example-0--main-py .anchor} **main.py:** Пример структуры паттерна

<figure class="code">
<pre class="code" lang="python"><code>from __future__ import annotations
from collections.abc import Iterable, Iterator
from typing import Any


&quot;&quot;&quot;
Для создания итератора в Python есть два абстрактных класса из встроенного
модуля collections - Iterable, Iterator. Нужно реализовать метод __iter__() в
итерируемом объекте (списке), а метод __next__() в итераторе.
&quot;&quot;&quot;


class AlphabeticalOrderIterator(Iterator):
    &quot;&quot;&quot;
    Конкретные Итераторы реализуют различные алгоритмы обхода. Эти классы
    постоянно хранят текущее положение обхода.
    &quot;&quot;&quot;

    &quot;&quot;&quot;
    Атрибут _position хранит текущее положение обхода. У итератора может быть
    множество других полей для хранения состояния итерации, особенно когда он
    должен работать с определённым типом коллекции.
    &quot;&quot;&quot;
    _position: int = None

    &quot;&quot;&quot;
    Этот атрибут указывает направление обхода.
    &quot;&quot;&quot;
    _reverse: bool = False

    def __init__(self, collection: WordsCollection, reverse: bool = False) -&gt; None:
        self._collection = collection
        self._reverse = reverse
        self._sorted_items = None  # Will be set on first __next__ call
        self._position = 0

    def __next__(self) -&gt; Any:
        &quot;&quot;&quot;
        Оптимизация: сортировка происходит только тогда, когда первый элемент
        впервые запрашивается.
        &quot;&quot;&quot;
        if self._sorted_items is None:
            self._sorted_items = sorted(self._collection._collection)
            if self._reverse:
                self._sorted_items = list(reversed(self._sorted_items))

        &quot;&quot;&quot;
        Метод __next __() должен вернуть следующий элемент в последовательности.
        При достижении конца коллекции и в последующих вызовах должно вызываться
        исключение StopIteration.
        &quot;&quot;&quot;
        if self._position &gt;= len(self._sorted_items):
            raise StopIteration()
        value = self._sorted_items[self._position]
        self._position += 1
        return value


class WordsCollection(Iterable):
    &quot;&quot;&quot;
    Конкретные Коллекции предоставляют один или несколько методов для получения
    новых экземпляров итератора, совместимых с классом коллекции.
    &quot;&quot;&quot;

    def __init__(self, collection: list[Any] | None = None) -&gt; None:
        self._collection = collection or []


    def __getitem__(self, index: int) -&gt; Any:
        return self._collection[index]

    def __iter__(self) -&gt; AlphabeticalOrderIterator:
        &quot;&quot;&quot;
        Метод __iter__() возвращает объект итератора, по умолчанию мы возвращаем
        итератор с сортировкой по возрастанию.
        &quot;&quot;&quot;
        return AlphabeticalOrderIterator(self)

    def get_reverse_iterator(self) -&gt; AlphabeticalOrderIterator:
        return AlphabeticalOrderIterator(self, True)

    def add_item(self, item: Any) -&gt; None:
        self._collection.append(item)


if __name__ == &quot;__main__&quot;:
    # Клиентский код может знать или не знать о Конкретном Итераторе или классах
    # Коллекций, в зависимости от уровня косвенности, который вы хотите
    # сохранить в своей программе.
    collection = WordsCollection()
    collection.add_item(&quot;B&quot;)
    collection.add_item(&quot;A&quot;)
    collection.add_item(&quot;C&quot;)

    print(&quot;Straight traversal:&quot;)
    print(&quot;\n&quot;.join(collection))
    print(&quot;&quot;)

    print(&quot;Reverse traversal:&quot;)
    print(&quot;\n&quot;.join(collection.get_reverse_iterator()), end=&quot;&quot;)</code></pre>
</figure>

#### []{#example-0--Output-txt .anchor} **Output.txt:** Результат выполнения

<figure class="code">
<pre class="code" lang="output"><code>Straight traversal:
A
B
C

Reverse traversal:
C
B
A</code></pre>
</figure>

::: next
#### Читаем дальше

[Посредник на Python []{.fa
.fa-arrow-right}](../../mediator/python/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Вернуться назад

[[]{.fa .fa-arrow-left} Команда на
Python](../../command/python/example.html){.btn .btn-default rel="prev"}
:::

## **Итератор** на других языках программирования

[![Итератор на
C#](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Итератор на C#"){.prog-lang-link}
[![Итератор на
C++](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Итератор на C++"){.prog-lang-link}
[![Итератор на
Go](../../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Итератор на Go"){.prog-lang-link}
[![Итератор на
Java](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Итератор на Java"){.prog-lang-link}
[![Итератор на
PHP](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Итератор на PHP"){.prog-lang-link}
[![Итератор на
Ruby](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Итератор на Ruby"){.prog-lang-link}
[![Итератор на
Rust](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Итератор на Rust"){.prog-lang-link}
[![Итератор на
Swift](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Итератор на Swift"){.prog-lang-link}
[![Итератор на
TypeScript](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Итератор на TypeScript"){.prog-lang-link}
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
