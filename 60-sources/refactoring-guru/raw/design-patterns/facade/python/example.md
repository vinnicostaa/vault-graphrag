:::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../index.html){.home} / [Design
Patterns](../../../design-patterns.html){.type} /
[Facade](../../facade.html){.type} / [Python](../../python.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Facade](../../../images/patterns/cards/facade-mini14fd.png?id=71ad6fa98b168c11cb3a1a9517dedf78){srcset="/images/patterns/cards/facade-mini-2x.png?id=d4cc6a5d81a31143cc665f7ac1481ac8 2x"}
:::

# **Facade** in Python {#facade-in-python .pattern-example-title-block-title}
::::

::::::::::: pattern-example-body
::: pattern-example-brief
**Facade** is a structural design pattern that provides a simplified
(but limited) interface to a complex system of classes, library or
framework.

While Facade decreases the overall complexity of the application, it
also helps to move unwanted dependencies to one place.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Learn more about Facade ](../../facade.html){.btn .btn-lg
.btn-primary}
:::

:::::::: {.sidebar-navigation .with-scroll}
::: en-title
Navigation
:::

::: en-intro
 [Intro](#)
:::

::: en-intro
 [Conceptual Example](#example-0)
:::

::: en-file
 [main](#example-0--main-py)
:::

::: en-file
 [Output](#example-0--Output-txt)
:::
::::::::

**Complexity:** []{.fa-stars}

**Popularity:** []{.fa-stars}

**Usage examples:** The Facade pattern is commonly used in apps written
in Python. It's especially handy when working with complex libraries and
APIs.

**Identification:** Facade can be recognized in a class that has a
simple interface, but delegates most of the work to other classes.
Usually, facades manage the full life cycle of objects they use.
:::::::::::

[]{#example-0}

## Conceptual Example {#example-0-title}

This example illustrates the structure of the **Facade** design pattern.
It focuses on answering these questions:

-   What classes does it consist of?
-   What roles do these classes play?
-   In what way the elements of the pattern are related?

#### []{#example-0--main-py .anchor} **main.py:** Conceptual example

<figure class="code">
<pre class="code" lang="python"><code>from __future__ import annotations


class Facade:
    &quot;&quot;&quot;
    The Facade class provides a simple interface to the complex logic of one or
    several subsystems. The Facade delegates the client requests to the
    appropriate objects within the subsystem. The Facade is also responsible for
    managing their lifecycle. All of this shields the client from the undesired
    complexity of the subsystem.
    &quot;&quot;&quot;

    def __init__(self, subsystem1: Subsystem1, subsystem2: Subsystem2) -&gt; None:
        &quot;&quot;&quot;
        Depending on your application&#39;s needs, you can provide the Facade with
        existing subsystem objects or force the Facade to create them on its
        own.
        &quot;&quot;&quot;

        self._subsystem1 = subsystem1 or Subsystem1()
        self._subsystem2 = subsystem2 or Subsystem2()

    def operation(self) -&gt; str:
        &quot;&quot;&quot;
        The Facade&#39;s methods are convenient shortcuts to the sophisticated
        functionality of the subsystems. However, clients get only to a fraction
        of a subsystem&#39;s capabilities.
        &quot;&quot;&quot;

        results = []
        results.append(&quot;Facade initializes subsystems:&quot;)
        results.append(self._subsystem1.operation1())
        results.append(self._subsystem2.operation1())
        results.append(&quot;Facade orders subsystems to perform the action:&quot;)
        results.append(self._subsystem1.operation_n())
        results.append(self._subsystem2.operation_z())
        return &quot;\n&quot;.join(results)


class Subsystem1:
    &quot;&quot;&quot;
    The Subsystem can accept requests either from the facade or client directly.
    In any case, to the Subsystem, the Facade is yet another client, and it&#39;s
    not a part of the Subsystem.
    &quot;&quot;&quot;

    def operation1(self) -&gt; str:
        return &quot;Subsystem1: Ready!&quot;

    # ...

    def operation_n(self) -&gt; str:
        return &quot;Subsystem1: Go!&quot;


class Subsystem2:
    &quot;&quot;&quot;
    Some facades can work with multiple subsystems at the same time.
    &quot;&quot;&quot;

    def operation1(self) -&gt; str:
        return &quot;Subsystem2: Get ready!&quot;

    # ...

    def operation_z(self) -&gt; str:
        return &quot;Subsystem2: Fire!&quot;


def client_code(facade: Facade) -&gt; None:
    &quot;&quot;&quot;
    The client code works with complex subsystems through a simple interface
    provided by the Facade. When a facade manages the lifecycle of the
    subsystem, the client might not even know about the existence of the
    subsystem. This approach lets you keep the complexity under control.
    &quot;&quot;&quot;

    print(facade.operation(), end=&quot;&quot;)


if __name__ == &quot;__main__&quot;:
    # The client code may have some of the subsystem&#39;s objects already created.
    # In this case, it might be worthwhile to initialize the Facade with these
    # objects instead of letting the Facade create new instances.
    subsystem1 = Subsystem1()
    subsystem2 = Subsystem2()
    facade = Facade(subsystem1, subsystem2)
    client_code(facade)</code></pre>
</figure>

#### []{#example-0--Output-txt .anchor} **Output.txt:** Execution result

<figure class="code">
<pre class="code" lang="output"><code>Facade initializes subsystems:
Subsystem1: Ready!
Subsystem2: Get ready!
Facade orders subsystems to perform the action:
Subsystem1: Go!
Subsystem2: Fire!</code></pre>
</figure>

::: next
#### Read next

[Flyweight in Python []{.fa
.fa-arrow-right}](../../flyweight/python/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Return

[[]{.fa .fa-arrow-left} Decorator in
Python](../../decorator/python/example.html){.btn .btn-default
rel="prev"}
:::

## **Facade** in Other Languages

[![Facade in
C#](../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Facade in C#"){.prog-lang-link}
[![Facade in
C++](../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Facade in C++"){.prog-lang-link}
[![Facade in
Go](../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Facade in Go"){.prog-lang-link}
[![Facade in
Java](../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Facade in Java"){.prog-lang-link}
[![Facade in
PHP](../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Facade in PHP"){.prog-lang-link}
[![Facade in
Ruby](../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Facade in Ruby"){.prog-lang-link}
[![Facade in
Rust](../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Facade in Rust"){.prog-lang-link}
[![Facade in
Swift](../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Facade in Swift"){.prog-lang-link}
[![Facade in
TypeScript](../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Facade in TypeScript"){.prog-lang-link}
:::::::::::::::::

::::::: {.prom .banner-sidebar .banner .banner-striped .banner-examples .banner-removable .banner-removable-patterns data-id="DP: Examples-IDE" creative-id="examples-sidebar-en" position="sidebar"}
:::::: {.banner-inner style="font-size: 14px; line-height: 21px;"}
::: banner-examples-img
[![](../../../images/patterns/banners/examples-ide91ca.png?id=3115b4b548fb96b75974e2de8f4f49bc){width="215"
height="158" loading="lazy"
srcset="/images/patterns/banners/examples-ide-2x.png?id=93c007a6d157b97c28eb213f59d716ae 2x"}](../../book.html)
:::

::: banner-examples-fade
[ Archive with examples](../../book.html){.btn .btn-secondary
.btn-block}
:::

::: {.banner-examples-text style="margin-top: 1rem;"}
Buy the eBook **Dive Into Design Patterns** and get the access to
archive with dozens of detailed examples that can be opened right in
your IDE.

[ Learn more...](../../book.html){.btn .btn-secondary .btn-block}
:::
::::::
:::::::
:::::::::::::::::::::::
::::::::::::::::::::::::
