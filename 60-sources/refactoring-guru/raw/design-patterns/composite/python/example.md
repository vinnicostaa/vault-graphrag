:::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../index.html){.home} / [Design
Patterns](../../../design-patterns.html){.type} /
[Composite](../../composite.html){.type} /
[Python](../../python.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Composite](../../../images/patterns/cards/composite-minib543.png?id=a369d98d18b417f255d04568fd0131b8){srcset="/images/patterns/cards/composite-mini-2x.png?id=3f7f811fefeb0b64f6774746eb42af09 2x"}
:::

# **Composite** in Python {#composite-in-python .pattern-example-title-block-title}
::::

::::::::::: pattern-example-body
::: pattern-example-brief
**Composite** is a structural design pattern that lets you compose
objects into tree structures and then work with these structures as if
they were individual objects.

Composite became a pretty popular solution for the most problems that
require building a tree structure. Composite's great feature is the
ability to run methods recursively over the whole tree structure and sum
up the results.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Learn more about Composite ](../../composite.html){.btn .btn-lg
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

**Usage examples:** The Composite pattern is pretty common in Python
code. It's often used to represent hierarchies of user interface
components or the code that works with graphs.

**Identification:** If you have an object tree, and each object of a
tree is a part of the same class hierarchy, this is most likely a
composite. If methods of these classes delegate the work to child
objects of the tree and do it via the base class/interface of the
hierarchy, this is definitely a composite.
:::::::::::

[]{#example-0}

## Conceptual Example {#example-0-title}

This example illustrates the structure of the **Composite** design
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
    The base Component class declares common operations for both simple and
    complex objects of a composition.
    &quot;&quot;&quot;

    @property
    def parent(self) -&gt; Component:
        return self._parent

    @parent.setter
    def parent(self, parent: Component):
        &quot;&quot;&quot;
        Optionally, the base Component can declare an interface for setting and
        accessing a parent of the component in a tree structure. It can also
        provide some default implementation for these methods.
        &quot;&quot;&quot;

        self._parent = parent

    &quot;&quot;&quot;
    In some cases, it would be beneficial to define the child-management
    operations right in the base Component class. This way, you won&#39;t need to
    expose any concrete component classes to the client code, even during the
    object tree assembly. The downside is that these methods will be empty for
    the leaf-level components.
    &quot;&quot;&quot;

    def add(self, component: Component) -&gt; None:
        pass

    def remove(self, component: Component) -&gt; None:
        pass

    def is_composite(self) -&gt; bool:
        &quot;&quot;&quot;
        You can provide a method that lets the client code figure out whether a
        component can bear children.
        &quot;&quot;&quot;

        return False

    @abstractmethod
    def operation(self) -&gt; str:
        &quot;&quot;&quot;
        The base Component may implement some default behavior or leave it to
        concrete classes (by declaring the method containing the behavior as
        &quot;abstract&quot;).
        &quot;&quot;&quot;

        pass


class Leaf(Component):
    &quot;&quot;&quot;
    The Leaf class represents the end objects of a composition. A leaf can&#39;t
    have any children.

    Usually, it&#39;s the Leaf objects that do the actual work, whereas Composite
    objects only delegate to their sub-components.
    &quot;&quot;&quot;

    def operation(self) -&gt; str:
        return &quot;Leaf&quot;


class Composite(Component):
    &quot;&quot;&quot;
    The Composite class represents the complex components that may have
    children. Usually, the Composite objects delegate the actual work to their
    children and then &quot;sum-up&quot; the result.
    &quot;&quot;&quot;

    def __init__(self) -&gt; None:
        self._children: List[Component] = []

    &quot;&quot;&quot;
    A composite object can add or remove other components (both simple or
    complex) to or from its child list.
    &quot;&quot;&quot;

    def add(self, component: Component) -&gt; None:
        self._children.append(component)
        component.parent = self

    def remove(self, component: Component) -&gt; None:
        self._children.remove(component)
        component.parent = None

    def is_composite(self) -&gt; bool:
        return True

    def operation(self) -&gt; str:
        &quot;&quot;&quot;
        The Composite executes its primary logic in a particular way. It
        traverses recursively through all its children, collecting and summing
        their results. Since the composite&#39;s children pass these calls to their
        children and so forth, the whole object tree is traversed as a result.
        &quot;&quot;&quot;

        results = []
        for child in self._children:
            results.append(child.operation())
        return f&quot;Branch({&#39;+&#39;.join(results)})&quot;


def client_code(component: Component) -&gt; None:
    &quot;&quot;&quot;
    The client code works with all of the components via the base interface.
    &quot;&quot;&quot;

    print(f&quot;RESULT: {component.operation()}&quot;, end=&quot;&quot;)


def client_code2(component1: Component, component2: Component) -&gt; None:
    &quot;&quot;&quot;
    Thanks to the fact that the child-management operations are declared in the
    base Component class, the client code can work with any component, simple or
    complex, without depending on their concrete classes.
    &quot;&quot;&quot;

    if component1.is_composite():
        component1.add(component2)

    print(f&quot;RESULT: {component1.operation()}&quot;, end=&quot;&quot;)


if __name__ == &quot;__main__&quot;:
    # This way the client code can support the simple leaf components...
    simple = Leaf()
    print(&quot;Client: I&#39;ve got a simple component:&quot;)
    client_code(simple)
    print(&quot;\n&quot;)

    # ...as well as the complex composites.
    tree = Composite()

    branch1 = Composite()
    branch1.add(Leaf())
    branch1.add(Leaf())

    branch2 = Composite()
    branch2.add(Leaf())

    tree.add(branch1)
    tree.add(branch2)

    print(&quot;Client: Now I&#39;ve got a composite tree:&quot;)
    client_code(tree)
    print(&quot;\n&quot;)

    print(&quot;Client: I don&#39;t need to check the components classes even when managing the tree:&quot;)
    client_code2(tree, simple)</code></pre>
</figure>

#### []{#example-0--Output-txt .anchor} **Output.txt:** Execution result

<figure class="code">
<pre class="code" lang="output"><code>Client: I&#39;ve got a simple component:
RESULT: Leaf

Client: Now I&#39;ve got a composite tree:
RESULT: Branch(Branch(Leaf+Leaf)+Branch(Leaf))

Client: I don&#39;t need to check the components classes even when managing the tree:
RESULT: Branch(Branch(Leaf+Leaf)+Branch(Leaf)+Leaf)</code></pre>
</figure>

::: next
#### Read next

[Decorator in Python []{.fa
.fa-arrow-right}](../../decorator/python/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Return

[[]{.fa .fa-arrow-left} Bridge in
Python](../../bridge/python/example.html){.btn .btn-default rel="prev"}
:::

## **Composite** in Other Languages

[![Composite in
C#](../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Composite in C#"){.prog-lang-link}
[![Composite in
C++](../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Composite in C++"){.prog-lang-link}
[![Composite in
Go](../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Composite in Go"){.prog-lang-link}
[![Composite in
Java](../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Composite in Java"){.prog-lang-link}
[![Composite in
PHP](../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Composite in PHP"){.prog-lang-link}
[![Composite in
Ruby](../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Composite in Ruby"){.prog-lang-link}
[![Composite in
Rust](../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Composite in Rust"){.prog-lang-link}
[![Composite in
Swift](../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Composite in Swift"){.prog-lang-link}
[![Composite in
TypeScript](../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Composite in TypeScript"){.prog-lang-link}
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
