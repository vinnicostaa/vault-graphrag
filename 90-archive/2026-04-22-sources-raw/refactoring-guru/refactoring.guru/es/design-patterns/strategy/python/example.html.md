:::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [Patrones de
diseño](../../../design-patterns.html){.type} /
[Strategy](../../strategy.html){.type} /
[Python](../../python.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Strategy](../../../../images/patterns/cards/strategy-mini28f6.png?id=d38abee4fb6f2aed909d262bdadca936){srcset="/images/patterns/cards/strategy-mini-2x.png?id=f4e6608561f8e5d18be6927d4620ad29 2x"}
:::

# **Strategy** en Python {#strategy-en-python .pattern-example-title-block-title}
::::

::::::::::: pattern-example-body
::: pattern-example-brief
**Strategy** es un patrón de diseño de comportamiento que convierte un
grupo de comportamientos en objetos y los hace intercambiables dentro
del objeto de contexto original.

El objeto original, llamado contexto, contiene una referencia a un
objeto de estrategia y le delega la ejecución del comportamiento. Para
cambiar la forma en que el contexto realiza su trabajo, otros objetos
pueden sustituir el objeto de estrategia actualmente vinculado, por
otro.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Aprende más sobre el patrón Strategy ](../../strategy.html){.btn
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

**Ejemplos de uso:** El patrón Strategy es muy común en el código
Python. Se utiliza a menudo en varios *frameworks* para proporcionar a
los usuarios una forma de cambiar el comportamiento de una clase sin
extenderla.

**Identificación:** El patrón Strategy se puede reconocer por un método
que permite a los objetos anidados hacer el verdadero trabajo, así como
el método *setter* (modificador) que permite sustituir ese objeto por
otro diferente.
:::::::::::

[]{#example-0}

## Ejemplo conceptual {#example-0-title}

Este ejemplo ilustra la estructura del patrón de diseño **Strategy**. Se
centra en responder las siguientes preguntas:

-   ¿De qué clases se compone?
-   ¿Qué papeles juegan esas clases?
-   ¿De qué forma se relacionan los elementos del patrón?

#### []{#example-0--main-py .anchor} **main.py:** Ejemplo conceptual

<figure class="code">
<pre class="code" lang="python"><code>from __future__ import annotations
from abc import ABC, abstractmethod
from typing import List


class Context():
    &quot;&quot;&quot;
    The Context defines the interface of interest to clients.
    &quot;&quot;&quot;

    def __init__(self, strategy: Strategy) -&gt; None:
        &quot;&quot;&quot;
        Usually, the Context accepts a strategy through the constructor, but
        also provides a setter to change it at runtime.
        &quot;&quot;&quot;

        self._strategy = strategy

    @property
    def strategy(self) -&gt; Strategy:
        &quot;&quot;&quot;
        The Context maintains a reference to one of the Strategy objects. The
        Context does not know the concrete class of a strategy. It should work
        with all strategies via the Strategy interface.
        &quot;&quot;&quot;

        return self._strategy

    @strategy.setter
    def strategy(self, strategy: Strategy) -&gt; None:
        &quot;&quot;&quot;
        Usually, the Context allows replacing a Strategy object at runtime.
        &quot;&quot;&quot;

        self._strategy = strategy

    def do_some_business_logic(self) -&gt; None:
        &quot;&quot;&quot;
        The Context delegates some work to the Strategy object instead of
        implementing multiple versions of the algorithm on its own.
        &quot;&quot;&quot;

        # ...

        print(&quot;Context: Sorting data using the strategy (not sure how it&#39;ll do it)&quot;)
        result = self._strategy.do_algorithm([&quot;a&quot;, &quot;b&quot;, &quot;c&quot;, &quot;d&quot;, &quot;e&quot;])
        print(&quot;,&quot;.join(result))

        # ...


class Strategy(ABC):
    &quot;&quot;&quot;
    The Strategy interface declares operations common to all supported versions
    of some algorithm.

    The Context uses this interface to call the algorithm defined by Concrete
    Strategies.
    &quot;&quot;&quot;

    @abstractmethod
    def do_algorithm(self, data: List):
        pass


&quot;&quot;&quot;
Concrete Strategies implement the algorithm while following the base Strategy
interface. The interface makes them interchangeable in the Context.
&quot;&quot;&quot;


class ConcreteStrategyA(Strategy):
    def do_algorithm(self, data: List) -&gt; List:
        return sorted(data)


class ConcreteStrategyB(Strategy):
    def do_algorithm(self, data: List) -&gt; List:
        return reversed(sorted(data))


if __name__ == &quot;__main__&quot;:
    # The client code picks a concrete strategy and passes it to the context.
    # The client should be aware of the differences between strategies in order
    # to make the right choice.

    context = Context(ConcreteStrategyA())
    print(&quot;Client: Strategy is set to normal sorting.&quot;)
    context.do_some_business_logic()
    print()

    print(&quot;Client: Strategy is set to reverse sorting.&quot;)
    context.strategy = ConcreteStrategyB()
    context.do_some_business_logic()</code></pre>
</figure>

#### []{#example-0--Output-txt .anchor} **Output.txt:** Resultado de la ejecución

<figure class="code">
<pre class="code" lang="output"><code>Client: Strategy is set to normal sorting.
Context: Sorting data using the strategy (not sure how it&#39;ll do it)
a,b,c,d,e

Client: Strategy is set to reverse sorting.
Context: Sorting data using the strategy (not sure how it&#39;ll do it)
e,d,c,b,a</code></pre>
</figure>

::: next
#### Leer siguiente

[Template Method en Python []{.fa
.fa-arrow-right}](../../template-method/python/example.html){.btn
.btn-primary rel="next"}
:::

::: prev
#### Volver

[[]{.fa .fa-arrow-left} State en
Python](../../state/python/example.html){.btn .btn-default rel="prev"}
:::

## **Strategy** en otros lenguajes

[![Strategy en
C#](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Strategy en C#"){.prog-lang-link}
[![Strategy en
C++](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Strategy en C++"){.prog-lang-link}
[![Strategy en
Go](../../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Strategy en Go"){.prog-lang-link}
[![Strategy en
Java](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Strategy en Java"){.prog-lang-link}
[![Strategy en
PHP](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Strategy en PHP"){.prog-lang-link}
[![Strategy en
Ruby](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Strategy en Ruby"){.prog-lang-link}
[![Strategy en
Rust](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Strategy en Rust"){.prog-lang-link}
[![Strategy en
Swift](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Strategy en Swift"){.prog-lang-link}
[![Strategy en
TypeScript](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Strategy en TypeScript"){.prog-lang-link}
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
