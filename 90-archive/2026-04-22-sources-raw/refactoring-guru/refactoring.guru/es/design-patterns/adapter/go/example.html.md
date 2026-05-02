::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [Patrones de
diseño](../../../design-patterns.html){.type} /
[Adapter](../../adapter.html){.type} / [Go](../../go.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Adapter](../../../../images/patterns/cards/adapter-minid866.png?id=b2ee4f681fb589be5a0685b94692aebb){srcset="/images/patterns/cards/adapter-mini-2x.png?id=8274d99afbbe9c63bfbfd0d68ceeffc7 2x"}
:::

# **Adapter** en Go {#adapter-en-go .pattern-example-title-block-title}
::::

:::::::::::::::: pattern-example-body
::: pattern-example-brief
**Adapter** es un patrón de diseño estructural que permite colaborar a
objetos incompatibles.

El patrón Adapter actúa como envoltorio entre dos objetos. Atrapa las
llamadas a un objeto y las transforma a un formato y una interfaz
reconocible para el segundo objeto.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Aprende más sobre el patrón Adapter ](../../adapter.html){.btn .btn-lg
.btn-primary}
:::

::::::::::::: {.sidebar-navigation .with-scroll}
::: en-title
Navegación
:::

::: en-intro
 [Intro](#)
:::

::: en-intro
 [Ejemplo conceptual](#example-0)
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

## Ejemplo conceptual {#example-0-title}

Tenemos un código cliente que espera ciertas funciones de un objeto
(puerto Lightning), pero tenemos otro objeto llamado *adaptee* (portátil
Windows), que ofrece la misma funcionalidad pero a través de una
interfaz diferente (puerto USB).

Aquí es donde el patrón Adapter entra en escena. Creamos un tipo de
estructura conocido como *adapter* que:

-   Se adherirá a la misma interfaz que espera el cliente (puerto
    Lightning).

-   Traducirá la solicitud del cliente al objeto adaptado en la forma
    que éste espera. El adaptador acepta un conector Lightning y después
    traduce sus señales a un formato USB y las pasa al puerto USB en el
    portátil Windows.

#### []{#example-0--client-go .anchor} **client.go:** Código cliente

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

#### []{#example-0--computer-go .anchor} **computer.go:** Interfaz de cliente

<figure class="code">
<pre class="code" lang="go"><code>package main

type Computer interface {
    InsertIntoLightningPort()
}</code></pre>
</figure>

#### []{#example-0--mac-go .anchor} **mac.go:** Servicio

<figure class="code">
<pre class="code" lang="go"><code>package main

import &quot;fmt&quot;

type Mac struct {
}

func (m *Mac) InsertIntoLightningPort() {
    fmt.Println(&quot;Lightning connector is plugged into mac machine.&quot;)
}</code></pre>
</figure>

#### []{#example-0--windows-go .anchor} **windows.go:** Servicio desconocido

<figure class="code">
<pre class="code" lang="go"><code>package main

import &quot;fmt&quot;

type Windows struct{}

func (w *Windows) insertIntoUSBPort() {
    fmt.Println(&quot;USB connector is plugged into windows machine.&quot;)
}</code></pre>
</figure>

#### []{#example-0--windowsAdapter-go .anchor} **windowsAdapter.go:** Adaptador

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

#### []{#example-0--output-txt .anchor} **output.txt:** Resultado de la ejecución

<figure class="code">
<pre class="code" lang="output"><code>Client inserts Lightning connector into computer.
Lightning connector is plugged into mac machine.
Client inserts Lightning connector into computer.
Adapter converts Lightning signal to USB.
USB connector is plugged into windows machine.</code></pre>
</figure>

::: next
#### Leer siguiente

[Bridge en Go []{.fa
.fa-arrow-right}](../../bridge/go/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Volver

[[]{.fa .fa-arrow-left} Singleton en
Go](../../singleton/go/example.html){.btn .btn-default rel="prev"}
:::

## **Adapter** en otros lenguajes

[![Adapter en
C#](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Adapter en C#"){.prog-lang-link}
[![Adapter en
C++](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Adapter en C++"){.prog-lang-link}
[![Adapter en
Java](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Adapter en Java"){.prog-lang-link}
[![Adapter en
PHP](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Adapter en PHP"){.prog-lang-link}
[![Adapter en
Python](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Adapter en Python"){.prog-lang-link}
[![Adapter en
Ruby](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Adapter en Ruby"){.prog-lang-link}
[![Adapter en
Rust](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Adapter en Rust"){.prog-lang-link}
[![Adapter en
Swift](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Adapter en Swift"){.prog-lang-link}
[![Adapter en
TypeScript](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Adapter en TypeScript"){.prog-lang-link}
::::::::::::::::::::::

::::::: {.prom .banner-sidebar .banner .banner-striped .banner-examples .banner-removable .banner-removable-patterns data-id="DP: Examples-IDE" creative-id="examples-sidebar-es" position="sidebar"}
:::::: {.banner-inner style="font-size: 14px; line-height: 21px;"}
::: banner-examples-img
[![](../../../../images/patterns/banners/examples-ide91ca.png?id=3115b4b548fb96b75974e2de8f4f49bc){width="215"
height="158" loading="lazy"
srcset="/images/patterns/banners/examples-ide-2x.png?id=93c007a6d157b97c28eb213f59d716ae 2x"}](../../book.html)
:::

::: banner-examples-fade
[ Archivo con ejemplos de código](../../book.html){.btn .btn-secondary
.btn-block}
:::

::: {.banner-examples-text style="margin-top: 1rem;"}
Compra el libro electrónico **Sumérgete en los patrones de diseño** para
tener acceso al archivo con decenas de ejemplos detallados que puedes
abrir en tu IDE.

[ Saber más...](../../book.html){.btn .btn-secondary .btn-block}
:::
::::::
:::::::
::::::::::::::::::::::::::::
:::::::::::::::::::::::::::::
