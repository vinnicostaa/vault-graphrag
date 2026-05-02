:::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../index.html){.home} / [Design
Patterns](../../../design-patterns.html){.type} /
[Mediator](../../mediator.html){.type} / [C#](../../csharp.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Mediator](../../../images/patterns/cards/mediator-mini1561.png?id=a7e43ee8e17e4474737b1fcb3201d7ba){srcset="/images/patterns/cards/mediator-mini-2x.png?id=d288d7c73f5581ae09701d61200ae09e 2x"}
:::

# **Mediator** in C# {#mediator-in-c .pattern-example-title-block-title}
::::

::::::::::: pattern-example-body
::: pattern-example-brief
**Mediator** is a behavioral design pattern that reduces coupling
between components of a program by making them communicate indirectly,
through a special mediator object.

The Mediator makes it easy to modify, extend and reuse individual
components because they're no longer dependent on the dozens of other
classes.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Learn more about Mediator ](../../mediator.html){.btn .btn-lg
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
 [Program](#example-0--Program-cs)
:::

::: en-file
 [Output](#example-0--Output-txt)
:::
::::::::

**Complexity:** []{.fa-stars}

**Popularity:** []{.fa-stars}

**Usage examples:** The most popular usage of the Mediator pattern in C#
code is facilitating communications between GUI components of an app.
The synonym of the Mediator is the Controller part of MVC pattern.
:::::::::::

[]{#example-0}

## Conceptual Example {#example-0-title}

This example illustrates the structure of the **Mediator** design
pattern. It focuses on answering these questions:

-   What classes does it consist of?
-   What roles do these classes play?
-   In what way the elements of the pattern are related?

#### []{#example-0--Program-cs .anchor} **Program.cs:** Conceptual example

<figure class="code">
<pre class="code" lang="csharp"><code>using System;

namespace RefactoringGuru.DesignPatterns.Mediator.Conceptual
{
    // The Mediator interface declares a method used by components to notify the
    // mediator about various events. The Mediator may react to these events and
    // pass the execution to other components.
    public interface IMediator
    {
        void Notify(object sender, string ev);
    }

    // Concrete Mediators implement cooperative behavior by coordinating several
    // components.
    class ConcreteMediator : IMediator
    {
        private Component1 _component1;

        private Component2 _component2;

        public ConcreteMediator(Component1 component1, Component2 component2)
        {
            this._component1 = component1;
            this._component1.SetMediator(this);
            this._component2 = component2;
            this._component2.SetMediator(this);
        } 

        public void Notify(object sender, string ev)
        {
            if (ev == &quot;A&quot;)
            {
                Console.WriteLine(&quot;Mediator reacts on A and triggers following operations:&quot;);
                this._component2.DoC();
            }
            if (ev == &quot;D&quot;)
            {
                Console.WriteLine(&quot;Mediator reacts on D and triggers following operations:&quot;);
                this._component1.DoB();
                this._component2.DoC();
            }
        }
    }

    // The Base Component provides the basic functionality of storing a
    // mediator&#39;s instance inside component objects.
    class BaseComponent
    {
        protected IMediator _mediator;

        public BaseComponent(IMediator mediator = null)
        {
            this._mediator = mediator;
        }

        public void SetMediator(IMediator mediator)
        {
            this._mediator = mediator;
        }
    }

    // Concrete Components implement various functionality. They don&#39;t depend on
    // other components. They also don&#39;t depend on any concrete mediator
    // classes.
    class Component1 : BaseComponent
    {
        public void DoA()
        {
            Console.WriteLine(&quot;Component 1 does A.&quot;);

            this._mediator.Notify(this, &quot;A&quot;);
        }

        public void DoB()
        {
            Console.WriteLine(&quot;Component 1 does B.&quot;);

            this._mediator.Notify(this, &quot;B&quot;);
        }
    }

    class Component2 : BaseComponent
    {
        public void DoC()
        {
            Console.WriteLine(&quot;Component 2 does C.&quot;);

            this._mediator.Notify(this, &quot;C&quot;);
        }

        public void DoD()
        {
            Console.WriteLine(&quot;Component 2 does D.&quot;);

            this._mediator.Notify(this, &quot;D&quot;);
        }
    }
    
    class Program
    {
        static void Main(string[] args)
        {
            // The client code.
            Component1 component1 = new Component1();
            Component2 component2 = new Component2();
            new ConcreteMediator(component1, component2);

            Console.WriteLine(&quot;Client triggers operation A.&quot;);
            component1.DoA();

            Console.WriteLine();

            Console.WriteLine(&quot;Client triggers operation D.&quot;);
            component2.DoD();
        }
    }
}</code></pre>
</figure>

#### []{#example-0--Output-txt .anchor} **Output.txt:** Execution result

<figure class="code">
<pre class="code" lang="output"><code>Client triggers operation A.
Component 1 does A.
Mediator reacts on A and triggers following operations:
Component 2 does C.

Client triggers operation D.
Component 2 does D.
Mediator reacts on D and triggers following operations:
Component 1 does B.
Component 2 does C.</code></pre>
</figure>

::: next
#### Read next

[Memento in C# []{.fa
.fa-arrow-right}](../../memento/csharp/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Return

[[]{.fa .fa-arrow-left} Iterator in
C#](../../iterator/csharp/example.html){.btn .btn-default rel="prev"}
:::

## **Mediator** in Other Languages

[![Mediator in
C++](../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Mediator in C++"){.prog-lang-link}
[![Mediator in
Go](../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Mediator in Go"){.prog-lang-link}
[![Mediator in
Java](../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Mediator in Java"){.prog-lang-link}
[![Mediator in
PHP](../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Mediator in PHP"){.prog-lang-link}
[![Mediator in
Python](../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Mediator in Python"){.prog-lang-link}
[![Mediator in
Ruby](../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Mediator in Ruby"){.prog-lang-link}
[![Mediator in
Rust](../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Mediator in Rust"){.prog-lang-link}
[![Mediator in
Swift](../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Mediator in Swift"){.prog-lang-link}
[![Mediator in
TypeScript](../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Mediator in TypeScript"){.prog-lang-link}
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
