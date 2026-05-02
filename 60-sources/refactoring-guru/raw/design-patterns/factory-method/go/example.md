::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../index.html){.home} / [Design
Patterns](../../../design-patterns.html){.type} / [Factory
Method](../../factory-method.html){.type} / [Go](../../go.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Factory
Method](../../../images/patterns/cards/factory-method-minid7f6.png?id=72619e9527893374b98a5913779ac167){srcset="/images/patterns/cards/factory-method-mini-2x.png?id=fa9d4a8d61a67cc3822e52b9daf69dad 2x"}
:::

# **Factory Method** in Go {#factory-method-in-go .pattern-example-title-block-title}
::::

:::::::::::::::: pattern-example-body
::: pattern-example-brief
**Factory method** is a creational design pattern which solves the
problem of creating product objects without specifying their concrete
classes.

The Factory Method defines a method, which should be used for creating
objects instead of using a direct constructor call (`new` operator).
Subclasses can override this method to change the class of objects that
will be created.

> If you can't figure out the difference between various factory
> patterns and concepts, then read our [Factory
> Comparison](../../factory-comparison.html).
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Learn more about Factory Method ](../../factory-method.html){.btn
.btn-lg .btn-primary}
:::

::::::::::::: {.sidebar-navigation .with-scroll}
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
 [i­Gun](#example-0--iGun-go)
:::

::: en-file
 [gun](#example-0--gun-go)
:::

::: en-file
 [ak47](#example-0--ak47-go)
:::

::: en-file
 [musket](#example-0--musket-go)
:::

::: en-file
 [gun­Factory](#example-0--gunFactory-go)
:::

::: en-file
 [main](#example-0--main-go)
:::

::: en-file
 [output](#example-0--output-txt)
:::
:::::::::::::
::::::::::::::::

[]{#example-0}

## Conceptual Example {#example-0-title}

It's impossible to implement the classic Factory Method pattern in Go
due to lack of OOP features such as classes and inheritance. However, we
can still implement the basic version of the pattern, the Simple
Factory.

In this example, we're going to build various types of weapons using a
factory struct.

First, we create the `iGun` interface, which defines all methods a gun
should have. There is a `gun` struct type that implements the iGun
interface. Two concrete guns---`ak47` and `musket`---both embed gun
struct and indirectly implement all `iGun` methods.

The `gunFactory` struct serves as a factory, which creates guns of the
desired type based on an incoming argument. The *main.go* acts as a
client. Instead of directly interacting with `ak47` or `musket`, it
relies on `gunFactory` to create instances of various guns, only using
string parameters to control the production.

#### []{#example-0--iGun-go .anchor} **iGun.go:** Product interface

<figure class="code">
<pre class="code" lang="go"><code>package main

type IGun interface {
    setName(name string)
    setPower(power int)
    getName() string
    getPower() int
}</code></pre>
</figure>

#### []{#example-0--gun-go .anchor} **gun.go:** Concrete product

<figure class="code">
<pre class="code" lang="go"><code>package main

type Gun struct {
    name  string
    power int
}

func (g *Gun) setName(name string) {
    g.name = name
}

func (g *Gun) getName() string {
    return g.name
}

func (g *Gun) setPower(power int) {
    g.power = power
}

func (g *Gun) getPower() int {
    return g.power
}</code></pre>
</figure>

#### []{#example-0--ak47-go .anchor} **ak47.go:** Concrete product

<figure class="code">
<pre class="code" lang="go"><code>package main

type Ak47 struct {
    Gun
}

func newAk47() IGun {
    return &amp;Ak47{
        Gun: Gun{
            name:  &quot;AK47 gun&quot;,
            power: 4,
        },
    }
}</code></pre>
</figure>

#### []{#example-0--musket-go .anchor} **musket.go:** Concrete product

<figure class="code">
<pre class="code" lang="go"><code>package main

type musket struct {
    Gun
}

func newMusket() IGun {
    return &amp;musket{
        Gun: Gun{
            name:  &quot;Musket gun&quot;,
            power: 1,
        },
    }
}</code></pre>
</figure>

#### []{#example-0--gunFactory-go .anchor} **gunFactory.go:** Factory

<figure class="code">
<pre class="code" lang="go"><code>package main

import &quot;fmt&quot;

func getGun(gunType string) (IGun, error) {
    if gunType == &quot;ak47&quot; {
        return newAk47(), nil
    }
    if gunType == &quot;musket&quot; {
        return newMusket(), nil
    }
    return nil, fmt.Errorf(&quot;Wrong gun type passed&quot;)
}</code></pre>
</figure>

#### []{#example-0--main-go .anchor} **main.go:** Client code

<figure class="code">
<pre class="code" lang="go"><code>package main

import &quot;fmt&quot;

func main() {
    ak47, _ := getGun(&quot;ak47&quot;)
    musket, _ := getGun(&quot;musket&quot;)

    printDetails(ak47)
    printDetails(musket)
}

func printDetails(g IGun) {
    fmt.Printf(&quot;Gun: %s&quot;, g.getName())
    fmt.Println()
    fmt.Printf(&quot;Power: %d&quot;, g.getPower())
    fmt.Println()
}</code></pre>
</figure>

#### []{#example-0--output-txt .anchor} **output.txt:** Execution result

<figure class="code">
<pre class="code" lang="output"><code>Gun: AK47 gun
Power: 4
Gun: Musket gun
Power: 1</code></pre>
</figure>

::: next
#### Read next

[Prototype in Go []{.fa
.fa-arrow-right}](../../prototype/go/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Return

[[]{.fa .fa-arrow-left} Builder in
Go](../../builder/go/example.html){.btn .btn-default rel="prev"}
:::

## **Factory Method** in Other Languages

[![Factory Method in
C#](../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Factory Method in C#"){.prog-lang-link}
[![Factory Method in
C++](../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Factory Method in C++"){.prog-lang-link}
[![Factory Method in
Java](../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Factory Method in Java"){.prog-lang-link}
[![Factory Method in
PHP](../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Factory Method in PHP"){.prog-lang-link}
[![Factory Method in
Python](../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Factory Method in Python"){.prog-lang-link}
[![Factory Method in
Ruby](../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Factory Method in Ruby"){.prog-lang-link}
[![Factory Method in
Rust](../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Factory Method in Rust"){.prog-lang-link}
[![Factory Method in
Swift](../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Factory Method in Swift"){.prog-lang-link}
[![Factory Method in
TypeScript](../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Factory Method in TypeScript"){.prog-lang-link}
::::::::::::::::::::::

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
::::::::::::::::::::::::::::
:::::::::::::::::::::::::::::
