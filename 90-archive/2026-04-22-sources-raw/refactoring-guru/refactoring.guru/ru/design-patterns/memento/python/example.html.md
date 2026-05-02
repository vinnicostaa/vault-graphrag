:::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [Паттерны
проектирования](../../../design-patterns.html){.type} /
[Снимок](../../memento.html){.type} / [Python](../../python.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Снимок](../../../../images/patterns/cards/memento-mini723d.png?id=8b2ea4dc2c5d15775a654808cc9de099){srcset="/images/patterns/cards/memento-mini-2x.png?id=1d7cba189261dd84b11369a6838b1055 2x"}
:::

# **Снимок** на Python {#снимок-на-python .pattern-example-title-block-title}
::::

::::::::::: pattern-example-body
::: pattern-example-brief
**Снимок** --- это поведенческий паттерн, позволяющий делать снимки
внутреннего состояния объектов, а затем восстанавливать их.

При этом Снимок не раскрывает подробностей реализации объектов, и клиент
не имеет доступа к защищённой информации объекта.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Подробней о паттерне Снимок ](../../memento.html){.btn .btn-lg
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

**Применимость:** Снимок на Python чаще всего реализуют с помощью
сериализации. Но это не единственный, да и не самый эффективный метод
сохранения состояния объектов во время выполнения программы.
:::::::::::

[]{#example-0}

## Концептуальный пример {#example-0-title}

Этот пример показывает структуру паттерна **Снимок**, а именно --- из
каких классов он состоит, какие роли эти классы выполняют и как они
взаимодействуют друг с другом.

#### []{#example-0--main-py .anchor} **main.py:** Пример структуры паттерна

<figure class="code">
<pre class="code" lang="python"><code>from __future__ import annotations
from abc import ABC, abstractmethod
from datetime import datetime
from random import sample
from string import ascii_letters


class Originator:
    &quot;&quot;&quot;
    Создатель содержит некоторое важное состояние, которое может со временем
    меняться. Он также объявляет метод сохранения состояния внутри снимка и
    метод восстановления состояния из него.
    &quot;&quot;&quot;

    _state = None
    &quot;&quot;&quot;
    Для удобства состояние создателя хранится внутри одной переменной.
    &quot;&quot;&quot;

    def __init__(self, state: str) -&gt; None:
        self._state = state
        print(f&quot;Originator: My initial state is: {self._state}&quot;)

    def do_something(self) -&gt; None:
        &quot;&quot;&quot;
        Бизнес-логика Создателя может повлиять на его внутреннее состояние.
        Поэтому клиент должен выполнить резервное копирование состояния с
        помощью метода save перед запуском методов бизнес-логики.
        &quot;&quot;&quot;

        print(&quot;Originator: I&#39;m doing something important.&quot;)
        self._state = self._generate_random_string(30)
        print(f&quot;Originator: and my state has changed to: {self._state}&quot;)

    @staticmethod
    def _generate_random_string(length: int = 10) -&gt; str:
        return &quot;&quot;.join(sample(ascii_letters, length))

    def save(self) -&gt; Memento:
        &quot;&quot;&quot;
        Сохраняет текущее состояние внутри снимка.
        &quot;&quot;&quot;

        return ConcreteMemento(self._state)

    def restore(self, memento: Memento) -&gt; None:
        &quot;&quot;&quot;
        Восстанавливает состояние Создателя из объекта снимка.
        &quot;&quot;&quot;

        self._state = memento.get_state()
        print(f&quot;Originator: My state has changed to: {self._state}&quot;)


class Memento(ABC):
    &quot;&quot;&quot;
    Интерфейс Снимка предоставляет способ извлечения метаданных снимка, таких
    как дата создания или название. Однако он не раскрывает состояние Создателя.
    &quot;&quot;&quot;

    @abstractmethod
    def get_name(self) -&gt; str:
        pass

    @abstractmethod
    def get_date(self) -&gt; str:
        pass


class ConcreteMemento(Memento):
    def __init__(self, state: str) -&gt; None:
        self._state = state
        self._date = str(datetime.now())[:19]

    def get_state(self) -&gt; str:
        &quot;&quot;&quot;
        Создатель использует этот метод, когда восстанавливает своё состояние.
        &quot;&quot;&quot;
        return self._state

    def get_name(self) -&gt; str:
        &quot;&quot;&quot;
        Остальные методы используются Опекуном для отображения метаданных.
        &quot;&quot;&quot;

        return f&quot;{self._date} / ({self._state[0:9]}...)&quot;

    def get_date(self) -&gt; str:
        return self._date


class Caretaker:
    &quot;&quot;&quot;
    Опекун не зависит от класса Конкретного Снимка. Таким образом, он не имеет
    доступа к состоянию создателя, хранящемуся внутри снимка. Он работает со
    всеми снимками через базовый интерфейс Снимка.
    &quot;&quot;&quot;

    def __init__(self, originator: Originator) -&gt; None:
        self._mementos = []
        self._originator = originator

    def backup(self) -&gt; None:
        print(&quot;\nCaretaker: Saving Originator&#39;s state...&quot;)
        self._mementos.append(self._originator.save())

    def undo(self) -&gt; None:
        if not len(self._mementos):
            return

        memento = self._mementos.pop()
        print(f&quot;Caretaker: Restoring state to: {memento.get_name()}&quot;)
        try:
            self._originator.restore(memento)
        except Exception:
            self.undo()

    def show_history(self) -&gt; None:
        print(&quot;Caretaker: Here&#39;s the list of mementos:&quot;)
        for memento in self._mementos:
            print(memento.get_name())


if __name__ == &quot;__main__&quot;:
    originator = Originator(&quot;Super-duper-super-puper-super.&quot;)
    caretaker = Caretaker(originator)

    caretaker.backup()
    originator.do_something()

    caretaker.backup()
    originator.do_something()

    caretaker.backup()
    originator.do_something()

    print()
    caretaker.show_history()

    print(&quot;\nClient: Now, let&#39;s rollback!\n&quot;)
    caretaker.undo()

    print(&quot;\nClient: Once more!\n&quot;)
    caretaker.undo()</code></pre>
</figure>

#### []{#example-0--Output-txt .anchor} **Output.txt:** Результат выполнения

<figure class="code">
<pre class="code" lang="output"><code>Originator: My initial state is: Super-duper-super-puper-super.

Caretaker: Saving Originator&#39;s state...
Originator: I&#39;m doing something important.
Originator: and my state has changed to: wQAehHYOqVSlpEXjyIcgobrxsZUnat

Caretaker: Saving Originator&#39;s state...
Originator: I&#39;m doing something important.
Originator: and my state has changed to: lHxNORKcsgMWYnJqoXjVCbQLEIeiSp

Caretaker: Saving Originator&#39;s state...
Originator: I&#39;m doing something important.
Originator: and my state has changed to: cvIYsRilNOtwynaKdEZpDCQkFAXVMf

Caretaker: Here&#39;s the list of mementos:
2019-01-26 21:11:24 / (Super-dup...)
2019-01-26 21:11:24 / (wQAehHYOq...)
2019-01-26 21:11:24 / (lHxNORKcs...)

Client: Now, let&#39;s rollback!

Caretaker: Restoring state to: 2019-01-26 21:11:24 / (lHxNORKcs...)
Originator: My state has changed to: lHxNORKcsgMWYnJqoXjVCbQLEIeiSp

Client: Once more!

Caretaker: Restoring state to: 2019-01-26 21:11:24 / (wQAehHYOq...)
Originator: My state has changed to: wQAehHYOqVSlpEXjyIcgobrxsZUnat</code></pre>
</figure>

::: next
#### Читаем дальше

[Наблюдатель на Python []{.fa
.fa-arrow-right}](../../observer/python/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Вернуться назад

[[]{.fa .fa-arrow-left} Посредник на
Python](../../mediator/python/example.html){.btn .btn-default
rel="prev"}
:::

## **Снимок** на других языках программирования

[![Снимок на
C#](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Снимок на C#"){.prog-lang-link}
[![Снимок на
C++](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Снимок на C++"){.prog-lang-link}
[![Снимок на
Go](../../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Снимок на Go"){.prog-lang-link}
[![Снимок на
Java](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Снимок на Java"){.prog-lang-link}
[![Снимок на
PHP](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Снимок на PHP"){.prog-lang-link}
[![Снимок на
Ruby](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Снимок на Ruby"){.prog-lang-link}
[![Снимок на
Rust](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Снимок на Rust"){.prog-lang-link}
[![Снимок на
Swift](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Снимок на Swift"){.prog-lang-link}
[![Снимок на
TypeScript](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Снимок на TypeScript"){.prog-lang-link}
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
