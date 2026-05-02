:::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [Patrones de
diseño](../../../design-patterns.html){.type} /
[Iterator](../../iterator.html){.type} /
[Python](../../python.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Iterator](../../../../images/patterns/cards/iterator-minie2c2.png?id=76c28bb48f997b36965983dd2b41f02e){srcset="/images/patterns/cards/iterator-mini-2x.png?id=7d28f66a487066774a743cfceaa40ea4 2x"}
:::

# **Iterator** en Python {#iterator-en-python .pattern-example-title-block-title}
::::

::::::::::: pattern-example-body
::: pattern-example-brief
**Iterator** es un patrón de diseño de comportamiento que permite el
recorrido secuencial por una estructura de datos compleja sin exponer
sus detalles internos.

Gracias al patrón Iterator, los clientes pueden recorrer elementos de
colecciones diferentes de un modo similar, utilizando una única interfaz
iteradora.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Aprende más sobre el patrón Iterator ](../../iterator.html){.btn
.btn-lg .btn-primary}
:::

:::::::: {.sidebar-navigation .with-scroll}
::: en-title
Navegación
:::

::: en-intro
 [Intro](#)
:::

::: en-intro
 [Ejemplo conceptual](#example-0)
:::

::: en-file
 [main](#example-0--main-py)
:::

::: en-file
 [Output](#example-0--Output-txt)
:::
::::::::

**Complejidad:** []{.fa-stars}

**Popularidad:** []{.fa-stars}

**Ejemplos de uso:** El patrón es muy común en el código Python. Muchos
*frameworks* y bibliotecas lo utilizan para proporcionar una forma
estandarizada de recorrer sus colecciones.

**Identificación:** El patrón Iterator es fácil de reconocer por sus
métodos de navegación (como `next`, `previous` y otros). El código
cliente que utiliza iteradores puede no tener acceso directo a la
colección recorrida.
:::::::::::

[]{#example-0}

## Ejemplo conceptual {#example-0-title}

Este ejemplo ilustra la estructura del patrón de diseño **Iterator**. Se
centra en responder las siguientes preguntas:

-   ¿De qué clases se compone?
-   ¿Qué papeles juegan esas clases?
-   ¿De qué forma se relacionan los elementos del patrón?

#### []{#example-0--main-py .anchor} **main.py:** Ejemplo conceptual

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

#### []{#example-0--Output-txt .anchor} **Output.txt:** Resultado de la ejecución

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
#### Leer siguiente

[Mediator en Python []{.fa
.fa-arrow-right}](../../mediator/python/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Volver

[[]{.fa .fa-arrow-left} Command en
Python](../../command/python/example.html){.btn .btn-default rel="prev"}
:::

## **Iterator** en otros lenguajes

[![Iterator en
C#](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Iterator en C#"){.prog-lang-link}
[![Iterator en
C++](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Iterator en C++"){.prog-lang-link}
[![Iterator en
Go](../../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Iterator en Go"){.prog-lang-link}
[![Iterator en
Java](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Iterator en Java"){.prog-lang-link}
[![Iterator en
PHP](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Iterator en PHP"){.prog-lang-link}
[![Iterator en
Ruby](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Iterator en Ruby"){.prog-lang-link}
[![Iterator en
Rust](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Iterator en Rust"){.prog-lang-link}
[![Iterator en
Swift](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Iterator en Swift"){.prog-lang-link}
[![Iterator en
TypeScript](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Iterator en TypeScript"){.prog-lang-link}
:::::::::::::::::

::::::: {.prom .banner-sidebar .banner .banner-striped .banner-examples .banner-removable .banner-removable-patterns data-id="DP: Examples-IDE" creative-id="examples-sidebar-es" position="sidebar"}
:::::: {.banner-inner style="font-size: 14px; line-height: 21px;"}
::: banner-examples-img
[![](../../../../images/patterns/banners/examples-ide91ca.png?id=3115b4b548fb96b75974e2de8f4f49bc){width="215"
height="158" loading="lazy"
srcset="/images/patterns/banners/examples-ide-2x.png?id=93c007a6d157b97c28eb213f59d716ae 2x"}](../../book.html)
:::

::: banner-examples-fade
[ Archivo con ejemplos de código](../../book.html){.btn .btn-secondary
.btn-block}
:::

::: {.banner-examples-text style="margin-top: 1rem;"}
Compra el libro electrónico **Sumérgete en los patrones de diseño** para
tener acceso al archivo con decenas de ejemplos detallados que puedes
abrir en tu IDE.

[ Saber más...](../../book.html){.btn .btn-secondary .btn-block}
:::
::::::
:::::::
:::::::::::::::::::::::
::::::::::::::::::::::::
