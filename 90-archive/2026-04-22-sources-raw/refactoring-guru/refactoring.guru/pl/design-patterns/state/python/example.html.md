:::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [Wzorce
projektowe](../../../design-patterns.html){.type} /
[Stan](../../state.html){.type} / [Python](../../python.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Stan](../../../../images/patterns/cards/state-mini63fb.png?id=f4018837e0641d1dade756b6678fd4ee){srcset="/images/patterns/cards/state-mini-2x.png?id=7e24398b27a43c7bd286fc0ea54d2a35 2x"}
:::

# **Stan** w języku Python {#stan-w-języku-python .pattern-example-title-block-title}
::::

::::::::::: pattern-example-body
::: pattern-example-brief
**Stan** to behawioralny wzorzec projektowy pozwalający zmieniać
zachowanie obiektu w odpowiedzi na zmianę jego wewnętrznego stanu.

Wzorzec Stan zakłada ekstrakcję zachowań odnoszących się do stanu
obiektu do osobnych klas odpowiadających jego poszczególnym stanom.
Pierwotny obiekt wówczas deleguje pracę instancjom tych klas, zamiast
wykonywać ją samodzielnie.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Dowiedz się więcej o Stan ](../../state.html){.btn .btn-lg
.btn-primary}
:::

:::::::: {.sidebar-navigation .with-scroll}
::: en-title
Nawigacja
:::

::: en-intro
 [Intro](#)
:::

::: en-intro
 [Przykład koncepcyjny](#example-0)
:::

::: en-file
 [main](#example-0--main-py)
:::

::: en-file
 [Output](#example-0--Output-txt)
:::
::::::::

**Złożoność:** []{.fa-stars}

**Popularność:** []{.fa-stars}

**Przykłady użycia:** Wzorzec Stan jest powszechnie stosowany w kodzie
Python w celu konwersji obszernych maszyn stanów opartych na instrukcji
`switch` w obiekty.

**Identyfikacja:** Wzorzec Stan można poznać po obecności metod
zmieniających swoje zachowanie zależnie od stanu obiektu, sterowanego z
zewnątrz.
:::::::::::

[]{#example-0}

## Przykład koncepcyjny {#example-0-title}

Poniższy przykład ilustruje strukturę wzorca **Stan** ze szczególnym
naciskiem na następujące kwestie:

-   Z jakich składa się klas?
-   Jakie role pełnią te klasy?
-   W jaki sposób elementy wzorca są ze sobą powiązane?

#### []{#example-0--main-py .anchor} **main.py:** Przykład koncepcyjny

<figure class="code">
<pre class="code" lang="python"><code>from __future__ import annotations
from abc import ABC, abstractmethod


class Context:
    &quot;&quot;&quot;
    The Context defines the interface of interest to clients. It also maintains
    a reference to an instance of a State subclass, which represents the current
    state of the Context.
    &quot;&quot;&quot;

    _state = None
    &quot;&quot;&quot;
    A reference to the current state of the Context.
    &quot;&quot;&quot;

    def __init__(self, state: State) -&gt; None:
        self.transition_to(state)

    def transition_to(self, state: State):
        &quot;&quot;&quot;
        The Context allows changing the State object at runtime.
        &quot;&quot;&quot;

        print(f&quot;Context: Transition to {type(state).__name__}&quot;)
        self._state = state
        self._state.context = self

    &quot;&quot;&quot;
    The Context delegates part of its behavior to the current State object.
    &quot;&quot;&quot;

    def request1(self):
        self._state.handle1()

    def request2(self):
        self._state.handle2()


class State(ABC):
    &quot;&quot;&quot;
    The base State class declares methods that all Concrete State should
    implement and also provides a backreference to the Context object,
    associated with the State. This backreference can be used by States to
    transition the Context to another State.
    &quot;&quot;&quot;

    @property
    def context(self) -&gt; Context:
        return self._context

    @context.setter
    def context(self, context: Context) -&gt; None:
        self._context = context

    @abstractmethod
    def handle1(self) -&gt; None:
        pass

    @abstractmethod
    def handle2(self) -&gt; None:
        pass


&quot;&quot;&quot;
Concrete States implement various behaviors, associated with a state of the
Context.
&quot;&quot;&quot;


class ConcreteStateA(State):
    def handle1(self) -&gt; None:
        print(&quot;ConcreteStateA handles request1.&quot;)
        print(&quot;ConcreteStateA wants to change the state of the context.&quot;)
        self.context.transition_to(ConcreteStateB())

    def handle2(self) -&gt; None:
        print(&quot;ConcreteStateA handles request2.&quot;)


class ConcreteStateB(State):
    def handle1(self) -&gt; None:
        print(&quot;ConcreteStateB handles request1.&quot;)

    def handle2(self) -&gt; None:
        print(&quot;ConcreteStateB handles request2.&quot;)
        print(&quot;ConcreteStateB wants to change the state of the context.&quot;)
        self.context.transition_to(ConcreteStateA())


if __name__ == &quot;__main__&quot;:
    # The client code.

    context = Context(ConcreteStateA())
    context.request1()
    context.request2()</code></pre>
</figure>

#### []{#example-0--Output-txt .anchor} **Output.txt:** Wynik działania

<figure class="code">
<pre class="code" lang="output"><code>Context: Transition to ConcreteStateA
ConcreteStateA handles request1.
ConcreteStateA wants to change the state of the context.
Context: Transition to ConcreteStateB
ConcreteStateB handles request2.
ConcreteStateB wants to change the state of the context.
Context: Transition to ConcreteStateA</code></pre>
</figure>

::: next
#### Czytaj dalej

[Strategia w języku Python []{.fa
.fa-arrow-right}](../../strategy/python/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Wróć

[[]{.fa .fa-arrow-left} Obserwator w języku
Python](../../observer/python/example.html){.btn .btn-default
rel="prev"}
:::

## **Stan** w innych językach

[![Stan w języku
C#](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Stan w języku C#"){.prog-lang-link}
[![Stan w języku
C++](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Stan w języku C++"){.prog-lang-link}
[![Stan w języku
Go](../../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Stan w języku Go"){.prog-lang-link}
[![Stan w języku
Java](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Stan w języku Java"){.prog-lang-link}
[![Stan w języku
PHP](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Stan w języku PHP"){.prog-lang-link}
[![Stan w języku
Ruby](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Stan w języku Ruby"){.prog-lang-link}
[![Stan w języku
Rust](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Stan w języku Rust"){.prog-lang-link}
[![Stan w języku
Swift](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Stan w języku Swift"){.prog-lang-link}
[![Stan w języku
TypeScript](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Stan w języku TypeScript"){.prog-lang-link}
:::::::::::::::::

::::::: {.prom .banner-sidebar .banner .banner-striped .banner-examples .banner-removable .banner-removable-patterns data-id="DP: Examples-IDE" creative-id="examples-sidebar-pl" position="sidebar"}
:::::: {.banner-inner style="font-size: 14px; line-height: 21px;"}
::: banner-examples-img
[![](../../../../images/patterns/banners/examples-ide91ca.png?id=3115b4b548fb96b75974e2de8f4f49bc){width="215"
height="158" loading="lazy"
srcset="/images/patterns/banners/examples-ide-2x.png?id=93c007a6d157b97c28eb213f59d716ae 2x"}](../../book.html)
:::

::: banner-examples-fade
[ Archiwum\
przykładów kodu](../../book.html){.btn .btn-secondary .btn-block}
:::

::: {.banner-examples-text style="margin-top: 1rem;"}
Kupując eBook **Wzorce projektowe. Nowoczesny podręcznik** zyskujesz
dostęp do archiwum zawierającego mnóstwo szczegółowych przykładów, które
można od razu wkleić do swojego preferowanego zintegrowanego środowiska
programistycznego.

[ Dowiedz się więcej...](../../book.html){.btn .btn-secondary
.btn-block}
:::
::::::
:::::::
:::::::::::::::::::::::
::::::::::::::::::::::::
