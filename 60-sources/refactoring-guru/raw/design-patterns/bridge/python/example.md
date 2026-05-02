:::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../index.html){.home} / [Design
Patterns](../../../design-patterns.html){.type} /
[Bridge](../../bridge.html){.type} / [Python](../../python.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Bridge](../../../images/patterns/cards/bridge-mini9e60.png?id=b389101d8ee8e23ffa1b534c704d0774){srcset="/images/patterns/cards/bridge-mini-2x.png?id=2622384cf623ed150ee9c21a0812dd87 2x"}
:::

# **Bridge** in Python {#bridge-in-python .pattern-example-title-block-title}
::::

::::::::::: pattern-example-body
::: pattern-example-brief
**Bridge** is a structural design pattern that divides business logic or
huge class into separate class hierarchies that can be developed
independently.

One of these hierarchies (often called the Abstraction) will get a
reference to an object of the second hierarchy (Implementation). The
abstraction will be able to delegate some (sometimes, most) of its calls
to the implementations object. Since all implementations will have a
common interface, they'd be interchangeable inside the abstraction.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Learn more about Bridge ](../../bridge.html){.btn .btn-lg
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

**Usage examples:** The Bridge pattern is especially useful when dealing
with cross-platform apps, supporting multiple types of database servers
or working with several API providers of a certain kind (for example,
cloud platforms, social networks, etc.)

**Identification:** Bridge can be recognized by a clear distinction
between some controlling entity and several different platforms that it
relies on.
:::::::::::

[]{#example-0}

## Conceptual Example {#example-0-title}

This example illustrates the structure of the **Bridge** design pattern.
It focuses on answering these questions:

-   What classes does it consist of?
-   What roles do these classes play?
-   In what way the elements of the pattern are related?

#### []{#example-0--main-py .anchor} **main.py:** Conceptual example

<figure class="code">
<pre class="code" lang="python"><code>from __future__ import annotations
from abc import ABC, abstractmethod


class Abstraction:
    &quot;&quot;&quot;
    The Abstraction defines the interface for the &quot;control&quot; part of the two
    class hierarchies. It maintains a reference to an object of the
    Implementation hierarchy and delegates all of the real work to this object.
    &quot;&quot;&quot;

    def __init__(self, implementation: Implementation) -&gt; None:
        self.implementation = implementation

    def operation(self) -&gt; str:
        return (f&quot;Abstraction: Base operation with:\n&quot;
                f&quot;{self.implementation.operation_implementation()}&quot;)


class ExtendedAbstraction(Abstraction):
    &quot;&quot;&quot;
    You can extend the Abstraction without changing the Implementation classes.
    &quot;&quot;&quot;

    def operation(self) -&gt; str:
        return (f&quot;ExtendedAbstraction: Extended operation with:\n&quot;
                f&quot;{self.implementation.operation_implementation()}&quot;)


class Implementation(ABC):
    &quot;&quot;&quot;
    The Implementation defines the interface for all implementation classes. It
    doesn&#39;t have to match the Abstraction&#39;s interface. In fact, the two
    interfaces can be entirely different. Typically the Implementation interface
    provides only primitive operations, while the Abstraction defines higher-
    level operations based on those primitives.
    &quot;&quot;&quot;

    @abstractmethod
    def operation_implementation(self) -&gt; str:
        pass


&quot;&quot;&quot;
Each Concrete Implementation corresponds to a specific platform and implements
the Implementation interface using that platform&#39;s API.
&quot;&quot;&quot;


class ConcreteImplementationA(Implementation):
    def operation_implementation(self) -&gt; str:
        return &quot;ConcreteImplementationA: Here&#39;s the result on the platform A.&quot;


class ConcreteImplementationB(Implementation):
    def operation_implementation(self) -&gt; str:
        return &quot;ConcreteImplementationB: Here&#39;s the result on the platform B.&quot;


def client_code(abstraction: Abstraction) -&gt; None:
    &quot;&quot;&quot;
    Except for the initialization phase, where an Abstraction object gets linked
    with a specific Implementation object, the client code should only depend on
    the Abstraction class. This way the client code can support any abstraction-
    implementation combination.
    &quot;&quot;&quot;

    # ...

    print(abstraction.operation(), end=&quot;&quot;)

    # ...


if __name__ == &quot;__main__&quot;:
    &quot;&quot;&quot;
    The client code should be able to work with any pre-configured abstraction-
    implementation combination.
    &quot;&quot;&quot;

    implementation = ConcreteImplementationA()
    abstraction = Abstraction(implementation)
    client_code(abstraction)

    print(&quot;\n&quot;)

    implementation = ConcreteImplementationB()
    abstraction = ExtendedAbstraction(implementation)
    client_code(abstraction)</code></pre>
</figure>

#### []{#example-0--Output-txt .anchor} **Output.txt:** Execution result

<figure class="code">
<pre class="code" lang="output"><code>Abstraction: Base operation with:
ConcreteImplementationA: Here&#39;s the result on the platform A.

ExtendedAbstraction: Extended operation with:
ConcreteImplementationB: Here&#39;s the result on the platform B.</code></pre>
</figure>

::: next
#### Read next

[Composite in Python []{.fa
.fa-arrow-right}](../../composite/python/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Return

[[]{.fa .fa-arrow-left} Adapter in
Python](../../adapter/python/example.html){.btn .btn-default rel="prev"}
:::

## **Bridge** in Other Languages

[![Bridge in
C#](../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Bridge in C#"){.prog-lang-link}
[![Bridge in
C++](../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Bridge in C++"){.prog-lang-link}
[![Bridge in
Go](../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Bridge in Go"){.prog-lang-link}
[![Bridge in
Java](../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Bridge in Java"){.prog-lang-link}
[![Bridge in
PHP](../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Bridge in PHP"){.prog-lang-link}
[![Bridge in
Ruby](../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Bridge in Ruby"){.prog-lang-link}
[![Bridge in
Rust](../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Bridge in Rust"){.prog-lang-link}
[![Bridge in
Swift](../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Bridge in Swift"){.prog-lang-link}
[![Bridge in
TypeScript](../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Bridge in TypeScript"){.prog-lang-link}
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
