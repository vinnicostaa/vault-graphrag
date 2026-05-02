::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../index.html){.home} / [Design
Patterns](../../../design-patterns.html){.type} /
[Adapter](../../adapter.html){.type} / [Go](../../go.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Adapter](../../../images/patterns/cards/adapter-minid866.png?id=b2ee4f681fb589be5a0685b94692aebb){srcset="/images/patterns/cards/adapter-mini-2x.png?id=8274d99afbbe9c63bfbfd0d68ceeffc7 2x"}
:::

# **Adapter** in Go {#adapter-in-go .pattern-example-title-block-title}
::::

:::::::::::::::: pattern-example-body
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
 [client](#example-0--client-go)
:::

::: en-file
 [computer](#example-0--computer-go)
:::

::: en-file
 [mac](#example-0--mac-go)
:::

::: en-file
 [windows](#example-0--windows-go)
:::

::: en-file
 [windows­Adapter](#example-0--windowsAdapter-go)
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

We have a client code that expects some features of an object (Lightning
port), but we have another object called *adaptee* (Windows laptop)
which offers the same functionality but through a different interface
(USB port)

This is where the Adapter pattern comes into the picture. We create a
struct type known as *adapter* that will:

-   Adhere to the same interface which the client expects (Lightning
    port).

-   Translate the request from the client to the adaptee in the form
    that the adaptee expects. The adapter accepts a Lightning connector
    and then translates its signals into a USB format and passes them to
    the USB port in windows laptop.

#### []{#example-0--client-go .anchor} **client.go:** Client code

<figure class="code">
<pre class="code" lang="go"><code>package main

import &quot;fmt&quot;

type Client struct {
}

func (c *Client) InsertLightningConnectorIntoComputer(com Computer) {
    fmt.Println(&quot;Client inserts Lightning connector into computer.&quot;)
    com.InsertIntoLightningPort()
}</code></pre>
</figure>

#### []{#example-0--computer-go .anchor} **computer.go:** Client interface

<figure class="code">
<pre class="code" lang="go"><code>package main

type Computer interface {
    InsertIntoLightningPort()
}</code></pre>
</figure>

#### []{#example-0--mac-go .anchor} **mac.go:** Service

<figure class="code">
<pre class="code" lang="go"><code>package main

import &quot;fmt&quot;

type Mac struct {
}

func (m *Mac) InsertIntoLightningPort() {
    fmt.Println(&quot;Lightning connector is plugged into mac machine.&quot;)
}</code></pre>
</figure>

#### []{#example-0--windows-go .anchor} **windows.go:** Unknown service

<figure class="code">
<pre class="code" lang="go"><code>package main

import &quot;fmt&quot;

type Windows struct{}

func (w *Windows) insertIntoUSBPort() {
    fmt.Println(&quot;USB connector is plugged into windows machine.&quot;)
}</code></pre>
</figure>

#### []{#example-0--windowsAdapter-go .anchor} **windowsAdapter.go:** Adapter

<figure class="code">
<pre class="code" lang="go"><code>package main

import &quot;fmt&quot;

type WindowsAdapter struct {
    windowMachine *Windows
}

func (w *WindowsAdapter) InsertIntoLightningPort() {
    fmt.Println(&quot;Adapter converts Lightning signal to USB.&quot;)
    w.windowMachine.insertIntoUSBPort()
}</code></pre>
</figure>

#### []{#example-0--main-go .anchor} **main.go**

<figure class="code">
<pre class="code" lang="go"><code>package main

func main() {

    client := &amp;Client{}
    mac := &amp;Mac{}

    client.InsertLightningConnectorIntoComputer(mac)

    windowsMachine := &amp;Windows{}
    windowsMachineAdapter := &amp;WindowsAdapter{
        windowMachine: windowsMachine,
    }

    client.InsertLightningConnectorIntoComputer(windowsMachineAdapter)
}</code></pre>
</figure>

#### []{#example-0--output-txt .anchor} **output.txt:** Execution result

<figure class="code">
<pre class="code" lang="output"><code>Client inserts Lightning connector into computer.
Lightning connector is plugged into mac machine.
Client inserts Lightning connector into computer.
Adapter converts Lightning signal to USB.
USB connector is plugged into windows machine.</code></pre>
</figure>

::: next
#### Read next

[Bridge in Go []{.fa
.fa-arrow-right}](../../bridge/go/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Return

[[]{.fa .fa-arrow-left} Singleton in
Go](../../singleton/go/example.html){.btn .btn-default rel="prev"}
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
[![Adapter in
TypeScript](../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Adapter in TypeScript"){.prog-lang-link}
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
