:::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [Wzorce
projektowe](../../../design-patterns.html){.type} /
[Iterator](../../iterator.html){.type} /
[Python](../../python.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Iterator](../../../../images/patterns/cards/iterator-minie2c2.png?id=76c28bb48f997b36965983dd2b41f02e){srcset="/images/patterns/cards/iterator-mini-2x.png?id=7d28f66a487066774a743cfceaa40ea4 2x"}
:::

# **Iterator** w języku Python {#iterator-w-języku-python .pattern-example-title-block-title}
::::

::::::::::: pattern-example-body
::: pattern-example-brief
**Iterator** to behawioralny wzorzec projektowy pozwalający sekwencyjnie
przechodzić od elementu do elementu jakiegoś zbioru bez konieczności
eksponowania jego formy.

Dzięki Iteratorowi klienci mogą przeglądać kolejne elementy różnych
kolekcji w podobny sposób, za pośrednictwem jednego interfejsu.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Dowiedz się więcej o Iterator ](../../iterator.html){.btn .btn-lg
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

**Przykłady użycia:** Wzorzec Iterator jest bardzo rozpowszechniony w
kodzie Python. Wiele frameworków i bibliotek pozwala za jego pomocą
poruszać się po elementach ich kolekcji.

**Identyfikacja:** Iterator łatwo rozpoznać po obecności metod
nawigacyjnych (takich jak `następny`, `poprzedni` i innych). Kod klienta
stosujący iteratory może nie mieć bezpośredniego dostępu do badanej
kolekcji.
:::::::::::

[]{#example-0}

## Przykład koncepcyjny {#example-0-title}

Poniższy przykład ilustruje strukturę wzorca **Iterator** ze szczególnym
naciskiem na następujące kwestie:

-   Z jakich składa się klas?
-   Jakie role pełnią te klasy?
-   W jaki sposób elementy wzorca są ze sobą powiązane?

#### []{#example-0--main-py .anchor} **main.py:** Przykład koncepcyjny

<figure class="code">
<pre class="code" lang="python"><code>from __future__ import annotations
from collections.abc import Iterable, Iterator
from typing import Any


&quot;&quot;&quot;
To create an iterator in Python, there are two abstract classes from the built-
in `collections` module - Iterable,Iterator. We need to implement the
`__iter__()` method in the iterated object (collection), and the `__next__ ()`
method in theiterator.
&quot;&quot;&quot;


class AlphabeticalOrderIterator(Iterator):
    &quot;&quot;&quot;
    Concrete Iterators implement various traversal algorithms. These classes
    store the current traversal position at all times.
    &quot;&quot;&quot;

    &quot;&quot;&quot;
    `_position` attribute stores the current traversal position. An iterator may
    have a lot of other fields for storing iteration state, especially when it
    is supposed to work with a particular kind of collection.
    &quot;&quot;&quot;
    _position: int = None

    &quot;&quot;&quot;
    This attribute indicates the traversal direction.
    &quot;&quot;&quot;
    _reverse: bool = False

    def __init__(self, collection: WordsCollection, reverse: bool = False) -&gt; None:
        self._collection = collection
        self._reverse = reverse
        self._sorted_items = None  # Will be set on first __next__ call
        self._position = 0

    def __next__(self) -&gt; Any:
        &quot;&quot;&quot;
        Optimization: sorting happens only when the first items is actually
        requested.
        &quot;&quot;&quot;
        if self._sorted_items is None:
            self._sorted_items = sorted(self._collection._collection)
            if self._reverse:
                self._sorted_items = list(reversed(self._sorted_items))

        &quot;&quot;&quot;
        The __next__() method must return the next item in the sequence. On
        reaching the end, and in subsequent calls, it must raise StopIteration.
        &quot;&quot;&quot;
        if self._position &gt;= len(self._sorted_items):
            raise StopIteration()
        value = self._sorted_items[self._position]
        self._position += 1
        return value


class WordsCollection(Iterable):
    &quot;&quot;&quot;
    Concrete Collections provide one or several methods for retrieving fresh
    iterator instances, compatible with the collection class.
    &quot;&quot;&quot;

    def __init__(self, collection: list[Any] | None = None) -&gt; None:
        self._collection = collection or []


    def __getitem__(self, index: int) -&gt; Any:
        return self._collection[index]

    def __iter__(self) -&gt; AlphabeticalOrderIterator:
        &quot;&quot;&quot;
        The __iter__() method returns the iterator object itself, by default we
        return the iterator in ascending order.
        &quot;&quot;&quot;
        return AlphabeticalOrderIterator(self)

    def get_reverse_iterator(self) -&gt; AlphabeticalOrderIterator:
        return AlphabeticalOrderIterator(self, True)

    def add_item(self, item: Any) -&gt; None:
        self._collection.append(item)


if __name__ == &quot;__main__&quot;:
    # The client code may or may not know about the Concrete Iterator or
    # Collection classes, depending on the level of indirection you want to keep
    # in your program.
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

#### []{#example-0--Output-txt .anchor} **Output.txt:** Wynik działania

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
#### Czytaj dalej

[Mediator w języku Python []{.fa
.fa-arrow-right}](../../mediator/python/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Wróć

[[]{.fa .fa-arrow-left} Polecenie w języku
Python](../../command/python/example.html){.btn .btn-default rel="prev"}
:::

## **Iterator** w innych językach

[![Iterator w języku
C#](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Iterator w języku C#"){.prog-lang-link}
[![Iterator w języku
C++](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Iterator w języku C++"){.prog-lang-link}
[![Iterator w języku
Go](../../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Iterator w języku Go"){.prog-lang-link}
[![Iterator w języku
Java](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Iterator w języku Java"){.prog-lang-link}
[![Iterator w języku
PHP](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Iterator w języku PHP"){.prog-lang-link}
[![Iterator w języku
Ruby](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Iterator w języku Ruby"){.prog-lang-link}
[![Iterator w języku
Rust](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Iterator w języku Rust"){.prog-lang-link}
[![Iterator w języku
Swift](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Iterator w języku Swift"){.prog-lang-link}
[![Iterator w języku
TypeScript](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Iterator w języku TypeScript"){.prog-lang-link}
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
