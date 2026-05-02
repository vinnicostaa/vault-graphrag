:::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [Patrones de
diseño](../../../design-patterns.html){.type} / [Chain of
Responsibility](../../chain-of-responsibility.html){.type} /
[Python](../../python.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Chain of
Responsibility](../../../../images/patterns/cards/chain-of-responsibility-mini6d42.png?id=36d85eba8d14986f053123de17aac7a7){srcset="/images/patterns/cards/chain-of-responsibility-mini-2x.png?id=8c81f7069e51259b2443801b91135f7f 2x"}
:::

# **Chain of Responsibility** en Python {#chain-of-responsibility-en-python .pattern-example-title-block-title}
::::

::::::::::: pattern-example-body
::: pattern-example-brief
**Chain of Responsibility** es un patrón de diseño de comportamiento que
permite pasar solicitudes a lo largo de la cadena de manejadores
potenciales hasta que uno de ellos gestiona la solicitud.

El patrón permite que varios objetos gestionen la solicitud sin acoplar
la clase emisora a las clases concretas de los receptores. La cadena
puede componerse dinámicamente durante el tiempo de ejecución con
cualquier manejador que siga una interfaz manejadora estándar.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Aprende más sobre el patrón Chain of Responsibility
](../../chain-of-responsibility.html){.btn .btn-lg .btn-primary}
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

**Ejemplos de uso:** El patrón Chain of Responsibility no es un invitado
habitual en el programa Python, ya que tan solo es relevante cuando el
código opera con cadenas de objetos.

**Identificación:** El patrón es reconocible porque los métodos de
comportamiento de un grupo de objetos invocan indirectamente los mismos
métodos en otros objetos, mientras que todos los objetos siguen la
interfaz común.
:::::::::::

[]{#example-0}

## Ejemplo conceptual {#example-0-title}

Este ejemplo ilustra la estructura del patrón de diseño **Chain of
Responsibility**. Se centra en responder las siguientes preguntas:

-   ¿De qué clases se compone?
-   ¿Qué papeles juegan esas clases?
-   ¿De qué forma se relacionan los elementos del patrón?

#### []{#example-0--main-py .anchor} **main.py:** Ejemplo conceptual

<figure class="code">
<pre class="code" lang="python"><code>from __future__ import annotations
from abc import ABC, abstractmethod
from typing import Any, Optional


class Handler(ABC):
    &quot;&quot;&quot;
    The Handler interface declares a method for building the chain of handlers.
    It also declares a method for executing a request.
    &quot;&quot;&quot;

    @abstractmethod
    def set_next(self, handler: Handler) -&gt; Handler:
        pass

    @abstractmethod
    def handle(self, request) -&gt; Optional[str]:
        pass


class AbstractHandler(Handler):
    &quot;&quot;&quot;
    The default chaining behavior can be implemented inside a base handler
    class.
    &quot;&quot;&quot;

    _next_handler: Handler = None

    def set_next(self, handler: Handler) -&gt; Handler:
        self._next_handler = handler
        # Returning a handler from here will let us link handlers in a
        # convenient way like this:
        # monkey.set_next(squirrel).set_next(dog)
        return handler

    @abstractmethod
    def handle(self, request: Any) -&gt; str:
        if self._next_handler:
            return self._next_handler.handle(request)

        return None


&quot;&quot;&quot;
All Concrete Handlers either handle a request or pass it to the next handler in
the chain.
&quot;&quot;&quot;


class MonkeyHandler(AbstractHandler):
    def handle(self, request: Any) -&gt; str:
        if request == &quot;Banana&quot;:
            return f&quot;Monkey: I&#39;ll eat the {request}&quot;
        else:
            return super().handle(request)


class SquirrelHandler(AbstractHandler):
    def handle(self, request: Any) -&gt; str:
        if request == &quot;Nut&quot;:
            return f&quot;Squirrel: I&#39;ll eat the {request}&quot;
        else:
            return super().handle(request)


class DogHandler(AbstractHandler):
    def handle(self, request: Any) -&gt; str:
        if request == &quot;MeatBall&quot;:
            return f&quot;Dog: I&#39;ll eat the {request}&quot;
        else:
            return super().handle(request)


def client_code(handler: Handler) -&gt; None:
    &quot;&quot;&quot;
    The client code is usually suited to work with a single handler. In most
    cases, it is not even aware that the handler is part of a chain.
    &quot;&quot;&quot;

    for food in [&quot;Nut&quot;, &quot;Banana&quot;, &quot;Cup of coffee&quot;]:
        print(f&quot;\nClient: Who wants a {food}?&quot;)
        result = handler.handle(food)
        if result:
            print(f&quot;  {result}&quot;, end=&quot;&quot;)
        else:
            print(f&quot;  {food} was left untouched.&quot;, end=&quot;&quot;)


if __name__ == &quot;__main__&quot;:
    monkey = MonkeyHandler()
    squirrel = SquirrelHandler()
    dog = DogHandler()

    monkey.set_next(squirrel).set_next(dog)

    # The client should be able to send a request to any handler, not just the
    # first one in the chain.
    print(&quot;Chain: Monkey &gt; Squirrel &gt; Dog&quot;)
    client_code(monkey)
    print(&quot;\n&quot;)

    print(&quot;Subchain: Squirrel &gt; Dog&quot;)
    client_code(squirrel)</code></pre>
</figure>

#### []{#example-0--Output-txt .anchor} **Output.txt:** Resultado de la ejecución

<figure class="code">
<pre class="code" lang="output"><code>Chain: Monkey &gt; Squirrel &gt; Dog

Client: Who wants a Nut?
  Squirrel: I&#39;ll eat the Nut
Client: Who wants a Banana?
  Monkey: I&#39;ll eat the Banana
Client: Who wants a Cup of coffee?
  Cup of coffee was left untouched.

Subchain: Squirrel &gt; Dog

Client: Who wants a Nut?
  Squirrel: I&#39;ll eat the Nut
Client: Who wants a Banana?
  Banana was left untouched.
Client: Who wants a Cup of coffee?
  Cup of coffee was left untouched.</code></pre>
</figure>

::: next
#### Leer siguiente

[Command en Python []{.fa
.fa-arrow-right}](../../command/python/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Volver

[[]{.fa .fa-arrow-left} Proxy en
Python](../../proxy/python/example.html){.btn .btn-default rel="prev"}
:::

## **Chain of Responsibility** en otros lenguajes

[![Chain of Responsibility en
C#](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Chain of Responsibility en C#"){.prog-lang-link}
[![Chain of Responsibility en
C++](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Chain of Responsibility en C++"){.prog-lang-link}
[![Chain of Responsibility en
Go](../../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Chain of Responsibility en Go"){.prog-lang-link}
[![Chain of Responsibility en
Java](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Chain of Responsibility en Java"){.prog-lang-link}
[![Chain of Responsibility en
PHP](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Chain of Responsibility en PHP"){.prog-lang-link}
[![Chain of Responsibility en
Ruby](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Chain of Responsibility en Ruby"){.prog-lang-link}
[![Chain of Responsibility en
Rust](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Chain of Responsibility en Rust"){.prog-lang-link}
[![Chain of Responsibility en
Swift](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Chain of Responsibility en Swift"){.prog-lang-link}
[![Chain of Responsibility en
TypeScript](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Chain of Responsibility en TypeScript"){.prog-lang-link}
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
