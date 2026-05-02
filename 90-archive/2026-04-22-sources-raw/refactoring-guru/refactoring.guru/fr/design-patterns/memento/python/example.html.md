:::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [Patrons de
conception](../../../design-patterns.html){.type} /
[Mémento](../../memento.html){.type} /
[Python](../../python.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Mémento](../../../../images/patterns/cards/memento-mini723d.png?id=8b2ea4dc2c5d15775a654808cc9de099){srcset="/images/patterns/cards/memento-mini-2x.png?id=1d7cba189261dd84b11369a6838b1055 2x"}
:::

# **Mémento** en Python {#mémento-en-python .pattern-example-title-block-title}
::::

::::::::::: pattern-example-body
::: pattern-example-brief
Le **Mémento** est un patron de conception comportemental qui permet de
prendre des instantanés de l'état d'un objet et de les restaurer plus
tard.

Le mémento ne compromet pas l'intégrité de la structure interne de
l'objet avec lequel il interagit, ni celle des données contenues dans
l'instantané.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ En savoir plus sur la patron Mémento ](../../memento.html){.btn
.btn-lg .btn-primary}
:::

:::::::: {.sidebar-navigation .with-scroll}
::: en-title
Navigation
:::

::: en-intro
 [Intro](#)
:::

::: en-intro
 [Exemple conceptuel](#example-0)
:::

::: en-file
 [main](#example-0--main-py)
:::

::: en-file
 [Output](#example-0--Output-txt)
:::
::::::::

**Complexité :** []{.fa-stars}

**Popularité :** []{.fa-stars}

**Exemples d'utilisation :** Le principe du mémento peut être imité en
utilisant la sérialisation, ce qui est assez commun en Python. Ce n'est
pas forcément la manière la plus efficace de prendre des instantanés de
l'état d'un objet, mais il permet de stocker des sauvegardes de l'état
tout en protégeant la structure du créateur des autres objets.
:::::::::::

[]{#example-0}

## Exemple conceptuel {#example-0-title}

Dans cet exemple, nous allons voir la structure du **Mémento**. Nous
allons répondre aux questions suivantes :

-   Que contiennent les classes ?
-   Quels rôles jouent-elles ?
-   Comment les éléments du patron sont-ils reliés ?

#### []{#example-0--main-py .anchor} **main.py:** Exemple conceptuel

<figure class="code">
<pre class="code" lang="python"><code>from __future__ import annotations
from abc import ABC, abstractmethod
from datetime import datetime
from random import sample
from string import ascii_letters


class Originator:
    &quot;&quot;&quot;
    The Originator holds some important state that may change over time. It also
    defines a method for saving the state inside a memento and another method
    for restoring the state from it.
    &quot;&quot;&quot;

    _state = None
    &quot;&quot;&quot;
    For the sake of simplicity, the originator&#39;s state is stored inside a single
    variable.
    &quot;&quot;&quot;

    def __init__(self, state: str) -&gt; None:
        self._state = state
        print(f&quot;Originator: My initial state is: {self._state}&quot;)

    def do_something(self) -&gt; None:
        &quot;&quot;&quot;
        The Originator&#39;s business logic may affect its internal state.
        Therefore, the client should backup the state before launching methods
        of the business logic via the save() method.
        &quot;&quot;&quot;

        print(&quot;Originator: I&#39;m doing something important.&quot;)
        self._state = self._generate_random_string(30)
        print(f&quot;Originator: and my state has changed to: {self._state}&quot;)

    @staticmethod
    def _generate_random_string(length: int = 10) -&gt; str:
        return &quot;&quot;.join(sample(ascii_letters, length))

    def save(self) -&gt; Memento:
        &quot;&quot;&quot;
        Saves the current state inside a memento.
        &quot;&quot;&quot;

        return ConcreteMemento(self._state)

    def restore(self, memento: Memento) -&gt; None:
        &quot;&quot;&quot;
        Restores the Originator&#39;s state from a memento object.
        &quot;&quot;&quot;

        self._state = memento.get_state()
        print(f&quot;Originator: My state has changed to: {self._state}&quot;)


class Memento(ABC):
    &quot;&quot;&quot;
    The Memento interface provides a way to retrieve the memento&#39;s metadata,
    such as creation date or name. However, it doesn&#39;t expose the Originator&#39;s
    state.
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
        The Originator uses this method when restoring its state.
        &quot;&quot;&quot;
        return self._state

    def get_name(self) -&gt; str:
        &quot;&quot;&quot;
        The rest of the methods are used by the Caretaker to display metadata.
        &quot;&quot;&quot;

        return f&quot;{self._date} / ({self._state[0:9]}...)&quot;

    def get_date(self) -&gt; str:
        return self._date


class Caretaker:
    &quot;&quot;&quot;
    The Caretaker doesn&#39;t depend on the Concrete Memento class. Therefore, it
    doesn&#39;t have access to the originator&#39;s state, stored inside the memento. It
    works with all mementos via the base Memento interface.
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

#### []{#example-0--Output-txt .anchor} **Output.txt:** Résultat de l'exécution

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
#### Suivant

[Observateur en Python []{.fa
.fa-arrow-right}](../../observer/python/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Retour

[[]{.fa .fa-arrow-left} Médiateur en
Python](../../mediator/python/example.html){.btn .btn-default
rel="prev"}
:::

## **Mémento** dans les autres langues

[![Mémento en
C#](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Mémento en C#"){.prog-lang-link}
[![Mémento en
C++](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Mémento en C++"){.prog-lang-link}
[![Mémento en
Go](../../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Mémento en Go"){.prog-lang-link}
[![Mémento en
Java](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Mémento en Java"){.prog-lang-link}
[![Mémento en
PHP](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Mémento en PHP"){.prog-lang-link}
[![Mémento en
Ruby](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Mémento en Ruby"){.prog-lang-link}
[![Mémento en
Rust](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Mémento en Rust"){.prog-lang-link}
[![Mémento en
Swift](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Mémento en Swift"){.prog-lang-link}
[![Mémento en
TypeScript](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Mémento en TypeScript"){.prog-lang-link}
:::::::::::::::::

::::::: {.prom .banner-sidebar .banner .banner-striped .banner-examples .banner-removable .banner-removable-patterns data-id="DP: Examples-IDE" creative-id="examples-sidebar-fr" position="sidebar"}
:::::: {.banner-inner style="font-size: 14px; line-height: 21px;"}
::: banner-examples-img
[![](../../../../images/patterns/banners/examples-ide91ca.png?id=3115b4b548fb96b75974e2de8f4f49bc){width="215"
height="158" loading="lazy"
srcset="/images/patterns/banners/examples-ide-2x.png?id=93c007a6d157b97c28eb213f59d716ae 2x"}](../../book.html)
:::

::: banner-examples-fade
[ Archive avec exemples](../../book.html){.btn .btn-secondary
.btn-block}
:::

::: {.banner-examples-text style="margin-top: 1rem;"}
Achetez l'ebook **Plongée au cœur des patrons de conception** et obtenez
l'accès à une archive comprenant des dizaines d'exemples détaillés qui
peuvent être ouverts dans votre environnement de développement.

[ En savoir plus...](../../book.html){.btn .btn-secondary .btn-block}
:::
::::::
:::::::
:::::::::::::::::::::::
::::::::::::::::::::::::
