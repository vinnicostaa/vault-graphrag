:::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [Патерни
проектування](../../../design-patterns.html){.type} /
[Спостерігач](../../observer.html){.type} /
[Python](../../python.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Спостерігач](../../../../images/patterns/cards/observer-minifa2e.png?id=fd2081ab1cff29c60b499bcf6a62786a){srcset="/images/patterns/cards/observer-mini-2x.png?id=f205b0655572ac8e4636691c0e0debfd 2x"}
:::

# **Спостерігач** на Python {#спостерігач-на-python .pattern-example-title-block-title}
::::

::::::::::: pattern-example-body
::: pattern-example-brief
**Спостерігач** --- це поведінковий патерн, який дозволяє об'єктам
повідомляти інші об'єкти про зміни свого стану.

При цьому спостерігачі можуть вільно підписуватися і відписуватись від
цих повідомлень.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Детальніше про Спостерігач ](../../observer.html){.btn .btn-lg
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

**Застосування:** Спостерігач часто зустрічається в коді Python,
особливо там, де до відносин між компонентами застосовується модель
подій. Спостерігач дозволяє окремим компонентам реагувати на події, які
відбуваються в інших компонентах.

**Ознаки застосування патерна:** Спостерігач визначається за механізмом
підписки та методами повідомлення, які викликають компоненти програми.
:::::::::::

[]{#example-0}

## Концептуальний приклад {#example-0-title}

Цей приклад показує структуру патерна **Спостерігач**, а саме --- з яких
класів він складається, які ролі ці класи виконують і як вони
взаємодіють один з одним.

#### []{#example-0--main-py .anchor} **main.py:** Приклад структури патерна

<figure class="code">
<pre class="code" lang="python"><code>from __future__ import annotations
from abc import ABC, abstractmethod
from random import randrange
from typing import List


class Subject(ABC):
    &quot;&quot;&quot;
    The Subject interface declares a set of methods for managing subscribers.
    &quot;&quot;&quot;

    @abstractmethod
    def attach(self, observer: Observer) -&gt; None:
        &quot;&quot;&quot;
        Attach an observer to the subject.
        &quot;&quot;&quot;
        pass

    @abstractmethod
    def detach(self, observer: Observer) -&gt; None:
        &quot;&quot;&quot;
        Detach an observer from the subject.
        &quot;&quot;&quot;
        pass

    @abstractmethod
    def notify(self) -&gt; None:
        &quot;&quot;&quot;
        Notify all observers about an event.
        &quot;&quot;&quot;
        pass


class ConcreteSubject(Subject):
    &quot;&quot;&quot;
    The Subject owns some important state and notifies observers when the state
    changes.
    &quot;&quot;&quot;

    _state: int = None
    &quot;&quot;&quot;
    For the sake of simplicity, the Subject&#39;s state, essential to all
    subscribers, is stored in this variable.
    &quot;&quot;&quot;

    _observers: List[Observer] = []
    &quot;&quot;&quot;
    List of subscribers. In real life, the list of subscribers can be stored
    more comprehensively (categorized by event type, etc.).
    &quot;&quot;&quot;

    def attach(self, observer: Observer) -&gt; None:
        print(&quot;Subject: Attached an observer.&quot;)
        self._observers.append(observer)

    def detach(self, observer: Observer) -&gt; None:
        self._observers.remove(observer)

    &quot;&quot;&quot;
    The subscription management methods.
    &quot;&quot;&quot;

    def notify(self) -&gt; None:
        &quot;&quot;&quot;
        Trigger an update in each subscriber.
        &quot;&quot;&quot;

        print(&quot;Subject: Notifying observers...&quot;)
        for observer in self._observers:
            observer.update(self)

    def some_business_logic(self) -&gt; None:
        &quot;&quot;&quot;
        Usually, the subscription logic is only a fraction of what a Subject can
        really do. Subjects commonly hold some important business logic, that
        triggers a notification method whenever something important is about to
        happen (or after it).
        &quot;&quot;&quot;

        print(&quot;\nSubject: I&#39;m doing something important.&quot;)
        self._state = randrange(0, 10)

        print(f&quot;Subject: My state has just changed to: {self._state}&quot;)
        self.notify()


class Observer(ABC):
    &quot;&quot;&quot;
    The Observer interface declares the update method, used by subjects.
    &quot;&quot;&quot;

    @abstractmethod
    def update(self, subject: Subject) -&gt; None:
        &quot;&quot;&quot;
        Receive update from subject.
        &quot;&quot;&quot;
        pass


&quot;&quot;&quot;
Concrete Observers react to the updates issued by the Subject they had been
attached to.
&quot;&quot;&quot;


class ConcreteObserverA(Observer):
    def update(self, subject: Subject) -&gt; None:
        if subject._state &lt; 3:
            print(&quot;ConcreteObserverA: Reacted to the event&quot;)


class ConcreteObserverB(Observer):
    def update(self, subject: Subject) -&gt; None:
        if subject._state == 0 or subject._state &gt;= 2:
            print(&quot;ConcreteObserverB: Reacted to the event&quot;)


if __name__ == &quot;__main__&quot;:
    # The client code.

    subject = ConcreteSubject()

    observer_a = ConcreteObserverA()
    subject.attach(observer_a)

    observer_b = ConcreteObserverB()
    subject.attach(observer_b)

    subject.some_business_logic()
    subject.some_business_logic()

    subject.detach(observer_a)

    subject.some_business_logic()</code></pre>
</figure>

#### []{#example-0--Output-txt .anchor} **Output.txt:** Результат виконання

<figure class="code">
<pre class="code" lang="output"><code>Subject: Attached an observer.
Subject: Attached an observer.

Subject: I&#39;m doing something important.
Subject: My state has just changed to: 0
Subject: Notifying observers...
ConcreteObserverA: Reacted to the event
ConcreteObserverB: Reacted to the event

Subject: I&#39;m doing something important.
Subject: My state has just changed to: 5
Subject: Notifying observers...
ConcreteObserverB: Reacted to the event

Subject: I&#39;m doing something important.
Subject: My state has just changed to: 0
Subject: Notifying observers...
ConcreteObserverB: Reacted to the event</code></pre>
</figure>

::: next
#### Читаємо далі

[Стан на Python []{.fa
.fa-arrow-right}](../../state/python/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Повернутись назад

[[]{.fa .fa-arrow-left} Знімок на
Python](../../memento/python/example.html){.btn .btn-default rel="prev"}
:::

## **Спостерігач** іншими мовами програмування

[![Спостерігач на
C#](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Спостерігач на C#"){.prog-lang-link}
[![Спостерігач на
C++](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Спостерігач на C++"){.prog-lang-link}
[![Спостерігач на
Go](../../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Спостерігач на Go"){.prog-lang-link}
[![Спостерігач на
Java](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Спостерігач на Java"){.prog-lang-link}
[![Спостерігач на
PHP](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Спостерігач на PHP"){.prog-lang-link}
[![Спостерігач на
Ruby](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Спостерігач на Ruby"){.prog-lang-link}
[![Спостерігач на
Rust](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Спостерігач на Rust"){.prog-lang-link}
[![Спостерігач на
Swift](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Спостерігач на Swift"){.prog-lang-link}
[![Спостерігач на
TypeScript](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Спостерігач на TypeScript"){.prog-lang-link}
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
