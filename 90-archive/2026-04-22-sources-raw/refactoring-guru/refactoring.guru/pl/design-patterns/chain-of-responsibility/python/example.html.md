:::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [Wzorce
projektowe](../../../design-patterns.html){.type} / [Łańcuch
zobowiązań](../../chain-of-responsibility.html){.type} /
[Python](../../python.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Łańcuch
zobowiązań](../../../../images/patterns/cards/chain-of-responsibility-mini6d42.png?id=36d85eba8d14986f053123de17aac7a7){srcset="/images/patterns/cards/chain-of-responsibility-mini-2x.png?id=8c81f7069e51259b2443801b91135f7f 2x"}
:::

# **Łańcuch zobowiązań** w języku Python {#łańcuch-zobowiązań-w-języku-python .pattern-example-title-block-title}
::::

::::::::::: pattern-example-body
::: pattern-example-brief
**Łańcuch zobowiązań** to behawioralny wzorzec projektowy pozwalający
przekazywać żądanie wzdłuż łańcucha potencjalnych obiektów
obsługujących aż zostanie obsłużone.

W łańcuchu zobowiązań wiele obiektów może obsłużyć żądanie bez
konieczności sprzęgania klas wysyłających je z konkretnymi klasami
odbierającymi. Łańcuch można układać dynamicznie w trakcie działania
programu z dowolnych obiektów obsługujących, wyposażonych w standardowy
interfejs obsługi żądań.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Dowiedz się więcej o Łańcuch zobowiązań
](../../chain-of-responsibility.html){.btn .btn-lg .btn-primary}
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

**Przykłady użycia:** Łańcuch zobowiązań jest rzadkim rozwiązaniem w
programach napisanych w Pythonie, gdyż ma sens tylko w tym kodzie, w
którym występują łańcuchy obiektów.

**Identyfikacja:** Wzorzec można rozpoznać na podstawie obecności
behawioralnych metod jednej grupy obiektów pośrednio wywołujących
analogiczne metody w innych obiektach za pośrednictwem wspólnego
interfejsu.
:::::::::::

[]{#example-0}

## Przykład koncepcyjny {#example-0-title}

Poniższy przykład ilustruje strukturę wzorca **Łańcuch zobowiązań** ze
szczególnym naciskiem na następujące kwestie:

-   Z jakich składa się klas?
-   Jakie role pełnią te klasy?
-   W jaki sposób elementy wzorca są ze sobą powiązane?

#### []{#example-0--main-py .anchor} **main.py:** Przykład koncepcyjny

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

#### []{#example-0--Output-txt .anchor} **Output.txt:** Wynik działania

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
#### Czytaj dalej

[Polecenie w języku Python []{.fa
.fa-arrow-right}](../../command/python/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Wróć

[[]{.fa .fa-arrow-left} Pełnomocnik w języku
Python](../../proxy/python/example.html){.btn .btn-default rel="prev"}
:::

## **Łańcuch zobowiązań** w innych językach

[![Łańcuch zobowiązań w języku
C#](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Łańcuch zobowiązań w języku C#"){.prog-lang-link}
[![Łańcuch zobowiązań w języku
C++](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Łańcuch zobowiązań w języku C++"){.prog-lang-link}
[![Łańcuch zobowiązań w języku
Go](../../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Łańcuch zobowiązań w języku Go"){.prog-lang-link}
[![Łańcuch zobowiązań w języku
Java](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Łańcuch zobowiązań w języku Java"){.prog-lang-link}
[![Łańcuch zobowiązań w języku
PHP](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Łańcuch zobowiązań w języku PHP"){.prog-lang-link}
[![Łańcuch zobowiązań w języku
Ruby](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Łańcuch zobowiązań w języku Ruby"){.prog-lang-link}
[![Łańcuch zobowiązań w języku
Rust](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Łańcuch zobowiązań w języku Rust"){.prog-lang-link}
[![Łańcuch zobowiązań w języku
Swift](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Łańcuch zobowiązań w języku Swift"){.prog-lang-link}
[![Łańcuch zobowiązań w języku
TypeScript](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Łańcuch zobowiązań w języku TypeScript"){.prog-lang-link}
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
