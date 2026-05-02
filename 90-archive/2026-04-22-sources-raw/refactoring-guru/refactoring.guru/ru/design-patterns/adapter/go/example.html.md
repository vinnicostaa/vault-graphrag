::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [Паттерны
проектирования](../../../design-patterns.html){.type} /
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
**Адаптер** --- это структурный паттерн, который позволяет подружить
несовместимые объекты.

Адаптер выступает прослойкой между двумя объектами, превращая вызовы
одного в вызовы понятные другому.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Подробней о паттерне Адаптер ](../../adapter.html){.btn .btn-lg
.btn-primary}
:::

::::::::::::: {.sidebar-navigation .with-scroll}
::: en-title
Навигация
:::

::: en-intro
 [Интро](#)
:::

::: en-intro
 [Концептуальный пример](#example-0)
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

## Концептуальный пример {#example-0-title}

Мы имеем код клиента, ожидающий определенных от объекта определенных
качеств (порт Lightning), но также мы имеем другой объект под названием
*adaptee* (ноутбук на Windows), который предоставляет тот же функционал,
но через другой интерфейс (USB порт).

В такой ситуации нам подойдет паттерн Адаптер. Мы создадим структуру
*adapter*, которая будет:

-   Реализовать тот же интерфейс, который ожидает клиент (порт
    Lightning).

-   Переводить запрос от клиента к адаптируемому объекту в форме,
    которую он ожидает. Адаптер принимает коннектор Lightning, после
    чего переводит его сигналы в формат USB в ноутбуке на Windows.

#### []{#example-0--client-go .anchor} **client.go:** Клиентский код

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

#### []{#example-0--computer-go .anchor} **computer.go:** Интерфейс клиента

<figure class="code">
<pre class="code" lang="go"><code>package main

type Computer interface {
    InsertIntoLightningPort()
}</code></pre>
</figure>

#### []{#example-0--mac-go .anchor} **mac.go:** Сервис

<figure class="code">
<pre class="code" lang="go"><code>package main

import &quot;fmt&quot;

type Mac struct {
}

func (m *Mac) InsertIntoLightningPort() {
    fmt.Println(&quot;Lightning connector is plugged into mac machine.&quot;)
}</code></pre>
</figure>

#### []{#example-0--windows-go .anchor} **windows.go:** Неизвестный сервис

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

#### []{#example-0--output-txt .anchor} **output.txt:** Результат выполнения

<figure class="code">
<pre class="code" lang="output"><code>Client inserts Lightning connector into computer.
Lightning connector is plugged into mac machine.
Client inserts Lightning connector into computer.
Adapter converts Lightning signal to USB.
USB connector is plugged into windows machine.</code></pre>
</figure>

::: next
#### Читаем дальше

[Мост на Go []{.fa .fa-arrow-right}](../../bridge/go/example.html){.btn
.btn-primary rel="next"}
:::

::: prev
#### Вернуться назад

[[]{.fa .fa-arrow-left} Одиночка на
Go](../../singleton/go/example.html){.btn .btn-default rel="prev"}
:::

## **Адаптер** на других языках программирования

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

::::::: {.prom .banner-sidebar .banner .banner-striped .banner-examples .banner-removable .banner-removable-patterns data-id="DP: Examples-IDE" creative-id="examples-sidebar-ru" position="sidebar"}
:::::: {.banner-inner style="font-size: 14px; line-height: 21px;"}
::: banner-examples-img
[![](../../../../images/patterns/banners/examples-ide91ca.png?id=3115b4b548fb96b75974e2de8f4f49bc){width="215"
height="158" loading="lazy"
srcset="/images/patterns/banners/examples-ide-2x.png?id=93c007a6d157b97c28eb213f59d716ae 2x"}](../../book.html)
:::

::: banner-examples-fade
[ Архив с примерами](../../book.html){.btn .btn-secondary .btn-block}
:::

::: {.banner-examples-text style="margin-top: 1rem;"}
Купи книгу **Погружение в Паттерны** и получи архив с десятками
детальных примеров, которые можно открывать прямо в IDE.

[ Узнать больше...](../../book.html){.btn .btn-secondary .btn-block}
:::
::::::
:::::::
::::::::::::::::::::::::::::
:::::::::::::::::::::::::::::
