::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [Wzorce
projektowe](../../../design-patterns.html){.type} /
[Pamiątka](../../memento.html){.type} / [Go](../../go.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Pamiątka](../../../../images/patterns/cards/memento-mini723d.png?id=8b2ea4dc2c5d15775a654808cc9de099){srcset="/images/patterns/cards/memento-mini-2x.png?id=1d7cba189261dd84b11369a6838b1055 2x"}
:::

# **Pamiątka** w języku Go {#pamiątka-w-języku-go .pattern-example-title-block-title}
::::

:::::::::::::: pattern-example-body
::: pattern-example-brief
**Pamiątka** to behawioralny wzorzec projektowy umożliwiający
zapisywanie "migawek" stanu obiektu i późniejsze jego przywracanie.

Wzorzec Pamiątka nie wpływa na wewnętrzną strukturę obiektu z którym
współpracuje, ani na dane przechowywane w migawkach.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Dowiedz się więcej o Pamiątka ](../../memento.html){.btn .btn-lg
.btn-primary}
:::

::::::::::: {.sidebar-navigation .with-scroll}
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
 [originator](#example-0--originator-go)
:::

::: en-file
 [memento](#example-0--memento-go)
:::

::: en-file
 [caretaker](#example-0--caretaker-go)
:::

::: en-file
 [main](#example-0--main-go)
:::

::: en-file
 [output](#example-0--output-txt)
:::
:::::::::::
::::::::::::::

[]{#example-0}

## Przykład koncepcyjny {#example-0-title}

Wzorzec Pamiątka pozwala zapisywać migawki stanu obiektu. Można dzięki
temu przywrócić obiekt do któregoś z pierwotnych stanów, co przydaje się
szczególnie w implementacji funkcjonalności cofnij-ponów.

#### []{#example-0--originator-go .anchor} **originator.go:** Źródło

<figure class="code">
<pre class="code" lang="go"><code>package main

type Originator struct {
    state string
}

func (e *Originator) createMemento() *Memento {
    return &amp;Memento{state: e.state}
}

func (e *Originator) restoreMemento(m *Memento) {
    e.state = m.getSavedState()
}

func (e *Originator) setState(state string) {
    e.state = state
}

func (e *Originator) getState() string {
    return e.state
}</code></pre>
</figure>

#### []{#example-0--memento-go .anchor} **memento.go:** Pamiątka

<figure class="code">
<pre class="code" lang="go"><code>package main

type Memento struct {
    state string
}

func (m *Memento) getSavedState() string {
    return m.state
}</code></pre>
</figure>

#### []{#example-0--caretaker-go .anchor} **caretaker.go:** Zarządca

<figure class="code">
<pre class="code" lang="go"><code>package main

type Caretaker struct {
    mementoArray []*Memento
}

func (c *Caretaker) addMemento(m *Memento) {
    c.mementoArray = append(c.mementoArray, m)
}

func (c *Caretaker) getMemento(index int) *Memento {
    return c.mementoArray[index]
}</code></pre>
</figure>

#### []{#example-0--main-go .anchor} **main.go:** Kod klienta

<figure class="code">
<pre class="code" lang="go"><code>package main

import &quot;fmt&quot;

func main() {

    caretaker := &amp;Caretaker{
        mementoArray: make([]*Memento, 0),
    }

    originator := &amp;Originator{
        state: &quot;A&quot;,
    }

    fmt.Printf(&quot;Originator Current State: %s\n&quot;, originator.getState())
    caretaker.addMemento(originator.createMemento())

    originator.setState(&quot;B&quot;)
    fmt.Printf(&quot;Originator Current State: %s\n&quot;, originator.getState())
    caretaker.addMemento(originator.createMemento())

    originator.setState(&quot;C&quot;)
    fmt.Printf(&quot;Originator Current State: %s\n&quot;, originator.getState())
    caretaker.addMemento(originator.createMemento())

    originator.restoreMemento(caretaker.getMemento(1))
    fmt.Printf(&quot;Restored to State: %s\n&quot;, originator.getState())

    originator.restoreMemento(caretaker.getMemento(0))
    fmt.Printf(&quot;Restored to State: %s\n&quot;, originator.getState())

}</code></pre>
</figure>

#### []{#example-0--output-txt .anchor} **output.txt:** Wynik działania

<figure class="code">
<pre class="code" lang="output"><code>originator Current State: A
originator Current State: B
originator Current State: C
Restored to State: B
Restored to State: A</code></pre>
</figure>

::: next
#### Czytaj dalej

[Obserwator w języku Go []{.fa
.fa-arrow-right}](../../observer/go/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Wróć

[[]{.fa .fa-arrow-left} Mediator w języku
Go](../../mediator/go/example.html){.btn .btn-default rel="prev"}
:::

## **Pamiątka** w innych językach

[![Pamiątka w języku
C#](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Pamiątka w języku C#"){.prog-lang-link}
[![Pamiątka w języku
C++](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Pamiątka w języku C++"){.prog-lang-link}
[![Pamiątka w języku
Java](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Pamiątka w języku Java"){.prog-lang-link}
[![Pamiątka w języku
PHP](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Pamiątka w języku PHP"){.prog-lang-link}
[![Pamiątka w języku
Python](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Pamiątka w języku Python"){.prog-lang-link}
[![Pamiątka w języku
Ruby](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Pamiątka w języku Ruby"){.prog-lang-link}
[![Pamiątka w języku
Rust](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Pamiątka w języku Rust"){.prog-lang-link}
[![Pamiątka w języku
Swift](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Pamiątka w języku Swift"){.prog-lang-link}
[![Pamiątka w języku
TypeScript](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Pamiątka w języku TypeScript"){.prog-lang-link}
::::::::::::::::::::

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
::::::::::::::::::::::::::
:::::::::::::::::::::::::::
