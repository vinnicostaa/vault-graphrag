::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [Патерни
проектування](../../../design-patterns.html){.type} /
[Адаптер](../../adapter.html){.type} / [Go](../../go.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Адаптер](../../../../images/patterns/cards/adapter-minid866.png?id=b2ee4f681fb589be5a0685b94692aebb){srcset="/images/patterns/cards/adapter-mini-2x.png?id=8274d99afbbe9c63bfbfd0d68ceeffc7 2x"}
:::

# **Адаптер** на Go {#адаптер-на-go .pattern-example-title-block-title}
::::

:::::::::::::::: pattern-example-body
::: pattern-example-brief
**Адаптер** --- це структурний патерн, який дозволяє подружити несумісні
об'єкти.

Адаптер виступає прошарком між двома об'єктами, перетворюючи виклики
одного у виклики, що зрозумілі іншому.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Детальніше про Адаптер ](../../adapter.html){.btn .btn-lg
.btn-primary}
:::

::::::::::::: {.sidebar-navigation .with-scroll}
::: en-title
Навігація
:::

::: en-intro
 [Інтро](#)
:::

::: en-intro
 [Концептуальний приклад](#example-0)
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

## Концептуальний приклад {#example-0-title}

Ми маємо код клієнта, котрий очікує від об'єкта певних якостей (порт
Lightning), але також ми маємо інший об'єкт під назвою *adaptee*
(ноутбук на Windows), який надає той же функціонал, але через інший
інтерфейс (USB порт).

У такій ситуації нам підійде патерн Адаптер. Ми створимо структуру
*adapter*, яка буде:

-   Реалізувати той же інтерфейс, який очікує клієнт (порт Lightning).

-   Перекладати запит від клієнта до об'єкту, що адаптується, у формі,
    яку він очікує. Адаптер приймає коннектор Lightning, після чого
    переводить його сигнали у формат USB у ноутбуці на Windows.

#### []{#example-0--client-go .anchor} **client.go:** Клієнтський код

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

#### []{#example-0--computer-go .anchor} **computer.go:** Інтерфейс клієнта

<figure class="code">
<pre class="code" lang="go"><code>package main

type Computer interface {
    InsertIntoLightningPort()
}</code></pre>
</figure>

#### []{#example-0--mac-go .anchor} **mac.go:** Сервіс

<figure class="code">
<pre class="code" lang="go"><code>package main

import &quot;fmt&quot;

type Mac struct {
}

func (m *Mac) InsertIntoLightningPort() {
    fmt.Println(&quot;Lightning connector is plugged into mac machine.&quot;)
}</code></pre>
</figure>

#### []{#example-0--windows-go .anchor} **windows.go:** Невідомий сервіс

<figure class="code">
<pre class="code" lang="go"><code>package main

import &quot;fmt&quot;

type Windows struct{}

func (w *Windows) insertIntoUSBPort() {
    fmt.Println(&quot;USB connector is plugged into windows machine.&quot;)
}</code></pre>
</figure>

#### []{#example-0--windowsAdapter-go .anchor} **windowsAdapter.go:** Адаптер

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

#### []{#example-0--output-txt .anchor} **output.txt:** Результат виконання

<figure class="code">
<pre class="code" lang="output"><code>Client inserts Lightning connector into computer.
Lightning connector is plugged into mac machine.
Client inserts Lightning connector into computer.
Adapter converts Lightning signal to USB.
USB connector is plugged into windows machine.</code></pre>
</figure>

::: next
#### Читаємо далі

[Міст на Go []{.fa .fa-arrow-right}](../../bridge/go/example.html){.btn
.btn-primary rel="next"}
:::

::: prev
#### Повернутись назад

[[]{.fa .fa-arrow-left} Одинак на
Go](../../singleton/go/example.html){.btn .btn-default rel="prev"}
:::

## **Адаптер** іншими мовами програмування

[![Адаптер на
C#](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Адаптер на C#"){.prog-lang-link}
[![Адаптер на
C++](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Адаптер на C++"){.prog-lang-link}
[![Адаптер на
Java](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Адаптер на Java"){.prog-lang-link}
[![Адаптер на
PHP](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Адаптер на PHP"){.prog-lang-link}
[![Адаптер на
Python](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Адаптер на Python"){.prog-lang-link}
[![Адаптер на
Ruby](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Адаптер на Ruby"){.prog-lang-link}
[![Адаптер на
Rust](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Адаптер на Rust"){.prog-lang-link}
[![Адаптер на
Swift](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Адаптер на Swift"){.prog-lang-link}
[![Адаптер на
TypeScript](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Адаптер на TypeScript"){.prog-lang-link}
::::::::::::::::::::::

::::::: {.prom .banner-sidebar .banner .banner-striped .banner-examples .banner-removable .banner-removable-patterns data-id="DP: Examples-IDE" creative-id="examples-sidebar-uk" position="sidebar"}
:::::: {.banner-inner style="font-size: 14px; line-height: 21px;"}
::: banner-examples-img
[![](../../../../images/patterns/banners/examples-ide91ca.png?id=3115b4b548fb96b75974e2de8f4f49bc){width="215"
height="158" loading="lazy"
srcset="/images/patterns/banners/examples-ide-2x.png?id=93c007a6d157b97c28eb213f59d716ae 2x"}](../../book.html)
:::

::: banner-examples-fade
[ Архів з прикладами](../../book.html){.btn .btn-secondary .btn-block}
:::

::: {.banner-examples-text style="margin-top: 1rem;"}
Придбай книгу **Занурення в Патерни** та отримай архів з десятками
детальних прикладів, які можна відкривати прямо в IDE.

[ Дізнатися більше...](../../book.html){.btn .btn-secondary .btn-block}
:::
::::::
:::::::
::::::::::::::::::::::::::::
:::::::::::::::::::::::::::::
