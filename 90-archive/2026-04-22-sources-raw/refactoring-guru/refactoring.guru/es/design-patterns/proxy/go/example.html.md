::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [Patrones de
diseño](../../../design-patterns.html){.type} /
[Proxy](../../proxy.html){.type} / [Go](../../go.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Proxy](../../../../images/patterns/cards/proxy-minid8d7.png?id=25890b11e7dc5af29625ccd0678b63a8){srcset="/images/patterns/cards/proxy-mini-2x.png?id=8638fac9dc08c992852492f9cb29d9c6 2x"}
:::

# **Proxy** en Go {#proxy-en-go .pattern-example-title-block-title}
::::

:::::::::::::: pattern-example-body
::: pattern-example-brief
**Proxy** es un patrón de diseño estructural que proporciona un objeto
que actúa como sustituto de un objeto de servicio real utilizado por un
cliente. Un proxy recibe solicitudes del cliente, realiza parte del
trabajo (control de acceso, almacenamiento en caché, etc.) y después
pasa la solicitud a un objeto de servicio.

El objeto proxy tiene la misma interfaz que un servicio, lo que lo hace
intercambiable con un objeto real cuando se pasa a un cliente.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Aprende más sobre el patrón Proxy ](../../proxy.html){.btn .btn-lg
.btn-primary}
:::

::::::::::: {.sidebar-navigation .with-scroll}
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
 [server](#example-0--server-go)
:::

::: en-file
 [nginx](#example-0--nginx-go)
:::

::: en-file
 [application](#example-0--application-go)
:::

::: en-file
 [main](#example-0--main-go)
:::

::: en-file
 [output](#example-0--output-txt)
:::
:::::::::::
::::::::::::::

[]{#example-0}

## Ejemplo conceptual {#example-0-title}

Un servidor web como Nginx puede actuar como proxy del servidor de tu
aplicación:

-   Ofrece un acceso controlado al servidor de tu aplicación.
-   Puede realizar limitación de velocidad.
-   Puede guardar solicitudes en caché.

#### []{#example-0--server-go .anchor} **server.go:** Sujeto

<figure class="code">
<pre class="code" lang="go"><code>package main

type server interface {
    handleRequest(string, string) (int, string)
}</code></pre>
</figure>

#### []{#example-0--nginx-go .anchor} **nginx.go:** Proxy

<figure class="code">
<pre class="code" lang="go"><code>package main

type Nginx struct {
    application       *Application
    maxAllowedRequest int
    rateLimiter       map[string]int
}

func newNginxServer() *Nginx {
    return &amp;Nginx{
        application:       &amp;Application{},
        maxAllowedRequest: 2,
        rateLimiter:       make(map[string]int),
    }
}

func (n *Nginx) handleRequest(url, method string) (int, string) {
    allowed := n.checkRateLimiting(url)
    if !allowed {
        return 403, &quot;Not Allowed&quot;
    }
    return n.application.handleRequest(url, method)
}

func (n *Nginx) checkRateLimiting(url string) bool {
    if n.rateLimiter[url] == 0 {
        n.rateLimiter[url] = 1
    }
    if n.rateLimiter[url] &gt; n.maxAllowedRequest {
        return false
    }
    n.rateLimiter[url] = n.rateLimiter[url] + 1
    return true
}</code></pre>
</figure>

#### []{#example-0--application-go .anchor} **application.go:** Sujeto real

<figure class="code">
<pre class="code" lang="go"><code>package main

type Application struct {
}

func (a *Application) handleRequest(url, method string) (int, string) {
    if url == &quot;/app/status&quot; &amp;&amp; method == &quot;GET&quot; {
        return 200, &quot;Ok&quot;
    }

    if url == &quot;/create/user&quot; &amp;&amp; method == &quot;POST&quot; {
        return 201, &quot;User Created&quot;
    }
    return 404, &quot;Not Ok&quot;
}</code></pre>
</figure>

#### []{#example-0--main-go .anchor} **main.go:** Código cliente

<figure class="code">
<pre class="code" lang="go"><code>package main

import &quot;fmt&quot;

func main() {

    nginxServer := newNginxServer()
    appStatusURL := &quot;/app/status&quot;
    createuserURL := &quot;/create/user&quot;

    httpCode, body := nginxServer.handleRequest(appStatusURL, &quot;GET&quot;)
    fmt.Printf(&quot;\nUrl: %s\nHttpCode: %d\nBody: %s\n&quot;, appStatusURL, httpCode, body)

    httpCode, body = nginxServer.handleRequest(appStatusURL, &quot;GET&quot;)
    fmt.Printf(&quot;\nUrl: %s\nHttpCode: %d\nBody: %s\n&quot;, appStatusURL, httpCode, body)

    httpCode, body = nginxServer.handleRequest(appStatusURL, &quot;GET&quot;)
    fmt.Printf(&quot;\nUrl: %s\nHttpCode: %d\nBody: %s\n&quot;, appStatusURL, httpCode, body)

    httpCode, body = nginxServer.handleRequest(createuserURL, &quot;POST&quot;)
    fmt.Printf(&quot;\nUrl: %s\nHttpCode: %d\nBody: %s\n&quot;, appStatusURL, httpCode, body)

    httpCode, body = nginxServer.handleRequest(createuserURL, &quot;GET&quot;)
    fmt.Printf(&quot;\nUrl: %s\nHttpCode: %d\nBody: %s\n&quot;, appStatusURL, httpCode, body)
}</code></pre>
</figure>

#### []{#example-0--output-txt .anchor} **output.txt:** Resultado de la ejecución

<figure class="code">
<pre class="code" lang="output"><code>Url: /app/status
HttpCode: 200
Body: Ok

Url: /app/status
HttpCode: 200
Body: Ok

Url: /app/status
HttpCode: 403
Body: Not Allowed

Url: /app/status
HttpCode: 201
Body: User Created

Url: /app/status
HttpCode: 404
Body: Not Ok</code></pre>
</figure>

::: next
#### Leer siguiente

[Chain of Responsibility en Go []{.fa
.fa-arrow-right}](../../chain-of-responsibility/go/example.html){.btn
.btn-primary rel="next"}
:::

::: prev
#### Volver

[[]{.fa .fa-arrow-left} Flyweight en
Go](../../flyweight/go/example.html){.btn .btn-default rel="prev"}
:::

## **Proxy** en otros lenguajes

[![Proxy en
C#](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Proxy en C#"){.prog-lang-link}
[![Proxy en
C++](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Proxy en C++"){.prog-lang-link}
[![Proxy en
Java](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Proxy en Java"){.prog-lang-link}
[![Proxy en
PHP](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Proxy en PHP"){.prog-lang-link}
[![Proxy en
Python](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Proxy en Python"){.prog-lang-link}
[![Proxy en
Ruby](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Proxy en Ruby"){.prog-lang-link}
[![Proxy en
Rust](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Proxy en Rust"){.prog-lang-link}
[![Proxy en
Swift](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Proxy en Swift"){.prog-lang-link}
[![Proxy en
TypeScript](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Proxy en TypeScript"){.prog-lang-link}
::::::::::::::::::::

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
::::::::::::::::::::::::::
:::::::::::::::::::::::::::
