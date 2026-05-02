:::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../index.html){.home} / [Design
Patterns](../../../design-patterns.html){.type} /
[Visitor](../../visitor.html){.type} /
[Python](../../python.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Visitor](../../../images/patterns/cards/visitor-mini0a1c.png?id=854a35a62963bec1d75eab996918989b){srcset="/images/patterns/cards/visitor-mini-2x.png?id=9b87f3f3b772f434b28a25876829b504 2x"}
:::

# **Visitor** in Python {#visitor-in-python .pattern-example-title-block-title}
::::

::::::::::: pattern-example-body
::: pattern-example-brief
**Visitor** is a behavioral design pattern that allows adding new
behaviors to existing class hierarchy without altering any existing
code.

> Read why Visitors can't be simply replaced with method overloading in
> our article [Visitor and Double
> Dispatch](../../visitor-double-dispatch.html).
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Learn more about Visitor ](../../visitor.html){.btn .btn-lg
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

**Usage examples:** Visitor isn't a very common pattern because of its
complexity and narrow applicability.
:::::::::::

[]{#example-0}

## Conceptual Example {#example-0-title}

This example illustrates the structure of the **Visitor** design
pattern. It focuses on answering these questions:

-   What classes does it consist of?
-   What roles do these classes play?
-   In what way the elements of the pattern are related?

#### []{#example-0--main-py .anchor} **main.py:** Conceptual example

<figure class="code">
<pre class="code" lang="python"><code>from __future__ import annotations
from abc import ABC, abstractmethod
from typing import List


class Component(ABC):
    &quot;&quot;&quot;
    The Component interface declares an `accept` method that should take the
    base visitor interface as an argument.
    &quot;&quot;&quot;

    @abstractmethod
    def accept(self, visitor: Visitor) -&gt; None:
        pass


class ConcreteComponentA(Component):
    &quot;&quot;&quot;
    Each Concrete Component must implement the `accept` method in such a way
    that it calls the visitor&#39;s method corresponding to the component&#39;s class.
    &quot;&quot;&quot;

    def accept(self, visitor: Visitor) -&gt; None:
        &quot;&quot;&quot;
        Note that we&#39;re calling `visitConcreteComponentA`, which matches the
        current class name. This way we let the visitor know the class of the
        component it works with.
        &quot;&quot;&quot;

        visitor.visit_concrete_component_a(self)

    def exclusive_method_of_concrete_component_a(self) -&gt; str:
        &quot;&quot;&quot;
        Concrete Components may have special methods that don&#39;t exist in their
        base class or interface. The Visitor is still able to use these methods
        since it&#39;s aware of the component&#39;s concrete class.
        &quot;&quot;&quot;

        return &quot;A&quot;


class ConcreteComponentB(Component):
    &quot;&quot;&quot;
    Same here: visitConcreteComponentB =&gt; ConcreteComponentB
    &quot;&quot;&quot;

    def accept(self, visitor: Visitor):
        visitor.visit_concrete_component_b(self)

    def special_method_of_concrete_component_b(self) -&gt; str:
        return &quot;B&quot;


class Visitor(ABC):
    &quot;&quot;&quot;
    The Visitor Interface declares a set of visiting methods that correspond to
    component classes. The signature of a visiting method allows the visitor to
    identify the exact class of the component that it&#39;s dealing with.
    &quot;&quot;&quot;

    @abstractmethod
    def visit_concrete_component_a(self, element: ConcreteComponentA) -&gt; None:
        pass

    @abstractmethod
    def visit_concrete_component_b(self, element: ConcreteComponentB) -&gt; None:
        pass


&quot;&quot;&quot;
Concrete Visitors implement several versions of the same algorithm, which can
work with all concrete component classes.

You can experience the biggest benefit of the Visitor pattern when using it with
a complex object structure, such as a Composite tree. In this case, it might be
helpful to store some intermediate state of the algorithm while executing
visitor&#39;s methods over various objects of the structure.
&quot;&quot;&quot;


class ConcreteVisitor1(Visitor):
    def visit_concrete_component_a(self, element) -&gt; None:
        print(f&quot;{element.exclusive_method_of_concrete_component_a()} + ConcreteVisitor1&quot;)

    def visit_concrete_component_b(self, element) -&gt; None:
        print(f&quot;{element.special_method_of_concrete_component_b()} + ConcreteVisitor1&quot;)


class ConcreteVisitor2(Visitor):
    def visit_concrete_component_a(self, element) -&gt; None:
        print(f&quot;{element.exclusive_method_of_concrete_component_a()} + ConcreteVisitor2&quot;)

    def visit_concrete_component_b(self, element) -&gt; None:
        print(f&quot;{element.special_method_of_concrete_component_b()} + ConcreteVisitor2&quot;)


def client_code(components: List[Component], visitor: Visitor) -&gt; None:
    &quot;&quot;&quot;
    The client code can run visitor operations over any set of elements without
    figuring out their concrete classes. The accept operation directs a call to
    the appropriate operation in the visitor object.
    &quot;&quot;&quot;

    # ...
    for component in components:
        component.accept(visitor)
    # ...


if __name__ == &quot;__main__&quot;:
    components = [ConcreteComponentA(), ConcreteComponentB()]

    print(&quot;The client code works with all visitors via the base Visitor interface:&quot;)
    visitor1 = ConcreteVisitor1()
    client_code(components, visitor1)

    print(&quot;It allows the same client code to work with different types of visitors:&quot;)
    visitor2 = ConcreteVisitor2()
    client_code(components, visitor2)</code></pre>
</figure>

#### []{#example-0--Output-txt .anchor} **Output.txt:** Execution result

<figure class="code">
<pre class="code" lang="output"><code>The client code works with all visitors via the base Visitor interface:
A + ConcreteVisitor1
B + ConcreteVisitor1
It allows the same client code to work with different types of visitors:
A + ConcreteVisitor2
B + ConcreteVisitor2</code></pre>
</figure>

::: next
#### Read next

[Design Patterns in Ruby []{.fa .fa-arrow-right}](../../ruby.html){.btn
.btn-primary rel="next"}
:::

::: prev
#### Return

[[]{.fa .fa-arrow-left} Template Method in
Python](../../template-method/python/example.html){.btn .btn-default
rel="prev"}
:::

## **Visitor** in Other Languages

[![Visitor in
C#](../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Visitor in C#"){.prog-lang-link}
[![Visitor in
C++](../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Visitor in C++"){.prog-lang-link}
[![Visitor in
Go](../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Visitor in Go"){.prog-lang-link}
[![Visitor in
Java](../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Visitor in Java"){.prog-lang-link}
[![Visitor in
PHP](../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Visitor in PHP"){.prog-lang-link}
[![Visitor in
Ruby](../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Visitor in Ruby"){.prog-lang-link}
[![Visitor in
Rust](../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Visitor in Rust"){.prog-lang-link}
[![Visitor in
Swift](../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Visitor in Swift"){.prog-lang-link}
[![Visitor in
TypeScript](../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Visitor in TypeScript"){.prog-lang-link}
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
