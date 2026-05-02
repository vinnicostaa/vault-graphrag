:::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../index.html){.home} / [Design
Patterns](../../../design-patterns.html){.type} /
[Adapter](../../adapter.html){.type} /
[TypeScript](../../typescript.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Adapter](../../../images/patterns/cards/adapter-minid866.png?id=b2ee4f681fb589be5a0685b94692aebb){srcset="/images/patterns/cards/adapter-mini-2x.png?id=8274d99afbbe9c63bfbfd0d68ceeffc7 2x"}
:::

# **Adapter** in TypeScript {#adapter-in-typescript .pattern-example-title-block-title}
::::

::::::::::: pattern-example-body
::: pattern-example-brief
**Adapter** is a structural design pattern, which allows incompatible
objects to collaborate.

The Adapter acts as a wrapper between two objects. It catches calls for
one object and transforms them to format and interface recognizable by
the second object.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Learn more about Adapter ](../../adapter.html){.btn .btn-lg
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
 [index](#example-0--index-ts)
:::

::: en-file
 [Output](#example-0--Output-txt)
:::
::::::::

**Complexity:** []{.fa-stars}

**Popularity:** []{.fa-stars}

**Usage examples:** The Adapter pattern is pretty common in TypeScript
code. It's very often used in systems based on some legacy code. In such
cases, Adapters make legacy code work with modern classes.

**Identification:** Adapter is recognizable by a constructor which takes
an instance of a different abstract/interface type. When the adapter
receives a call to any of its methods, it translates parameters to the
appropriate format and then directs the call to one or several methods
of the wrapped object.
:::::::::::

[]{#example-0}

## Conceptual Example {#example-0-title}

This example illustrates the structure of the **Adapter** design pattern
and focuses on the following questions:

-   What classes does it consist of?
-   What roles do these classes play?
-   In what way the elements of the pattern are related?

#### []{#example-0--index-ts .anchor} **index.ts:** Conceptual example

<figure class="code">
<pre class="code" lang="typescript"><code>/**
 * The Target defines the domain-specific interface used by the client code.
 */
class Target {
    public request(): string {
        return &#39;Target: The default target\&#39;s behavior.&#39;;
    }
}

/**
 * The Adaptee contains some useful behavior, but its interface is incompatible
 * with the existing client code. The Adaptee needs some adaptation before the
 * client code can use it.
 */
class Adaptee {
    public specificRequest(): string {
        return &#39;.eetpadA eht fo roivaheb laicepS&#39;;
    }
}

/**
 * The Adapter makes the Adaptee&#39;s interface compatible with the Target&#39;s
 * interface.
 */
class Adapter extends Target {
    private adaptee: Adaptee;

    constructor(adaptee: Adaptee) {
        super();
        this.adaptee = adaptee;
    }

    public request(): string {
        const result = this.adaptee.specificRequest().split(&#39;&#39;).reverse().join(&#39;&#39;);
        return `Adapter: (TRANSLATED) ${result}`;
    }
}

/**
 * The client code supports all classes that follow the Target interface.
 */
function clientCode(target: Target) {
    console.log(target.request());
}

console.log(&#39;Client: I can work just fine with the Target objects:&#39;);
const target = new Target();
clientCode(target);

console.log(&#39;&#39;);

const adaptee = new Adaptee();
console.log(&#39;Client: The Adaptee class has a weird interface. See, I don\&#39;t understand it:&#39;);
console.log(`Adaptee: ${adaptee.specificRequest()}`);

console.log(&#39;&#39;);

console.log(&#39;Client: But I can work with it via the Adapter:&#39;);
const adapter = new Adapter(adaptee);
clientCode(adapter);</code></pre>
</figure>

#### []{#example-0--Output-txt .anchor} **Output.txt:** Execution result

<figure class="code">
<pre class="code" lang="output"><code>Client: I can work just fine with the Target objects:
Target: The default target&#39;s behavior.

Client: The Adaptee class has a weird interface. See, I don&#39;t understand it:
Adaptee: .eetpadA eht fo roivaheb laicepS

Client: But I can work with it via the Adapter:
Adapter: (TRANSLATED) Special behavior of the Adaptee.</code></pre>
</figure>

::: next
#### Read next

[Bridge in TypeScript []{.fa
.fa-arrow-right}](../../bridge/typescript/example.html){.btn
.btn-primary rel="next"}
:::

::: prev
#### Return

[[]{.fa .fa-arrow-left} Singleton in
TypeScript](../../singleton/typescript/example.html){.btn .btn-default
rel="prev"}
:::

## **Adapter** in Other Languages

[![Adapter in
C#](../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Adapter in C#"){.prog-lang-link}
[![Adapter in
C++](../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Adapter in C++"){.prog-lang-link}
[![Adapter in
Go](../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Adapter in Go"){.prog-lang-link}
[![Adapter in
Java](../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Adapter in Java"){.prog-lang-link}
[![Adapter in
PHP](../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Adapter in PHP"){.prog-lang-link}
[![Adapter in
Python](../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Adapter in Python"){.prog-lang-link}
[![Adapter in
Ruby](../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Adapter in Ruby"){.prog-lang-link}
[![Adapter in
Rust](../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Adapter in Rust"){.prog-lang-link}
[![Adapter in
Swift](../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Adapter in Swift"){.prog-lang-link}
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
