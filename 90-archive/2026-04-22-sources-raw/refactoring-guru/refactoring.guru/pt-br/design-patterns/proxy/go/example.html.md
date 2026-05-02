::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [Padrões de
Projeto](../../../design-patterns.html){.type} /
[Proxy](../../proxy.html){.type} / [Go](../../go.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Proxy](../../../../images/patterns/cards/proxy-minid8d7.png?id=25890b11e7dc5af29625ccd0678b63a8){srcset="/images/patterns/cards/proxy-mini-2x.png?id=8638fac9dc08c992852492f9cb29d9c6 2x"}
:::

# **Proxy** em Go {#proxy-em-go .pattern-example-title-block-title}
::::

:::::::::::::: pattern-example-body
::: pattern-example-brief
O **Proxy** é um padrão de projeto estrutural que fornece um objeto que
atua como um substituto para um objeto de serviço real usado por um
cliente. Um proxy recebe solicitações do cliente, realiza alguma tarefa
(controle de acesso, armazenamento em cache etc.) e passa a solicitação
para um objeto de serviço.

O objeto proxy tem a mesma interface que um serviço, o que o torna
intercambiável com um objeto real quando passado para um cliente.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Saiba mais sobre o Proxy ](../../proxy.html){.btn .btn-lg
.btn-primary}
:::

::::::::::: {.sidebar-navigation .with-scroll}
::: en-title
Navegação
:::

::: en-intro
 [Introdução](#)
:::

::: en-intro
 [Exemplo conceitual](#example-0)
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

## Exemplo conceitual {#example-0-title}

Um servidor da web como o Nginx pode atuar como proxy para seu servidor
de aplicativos:

-   Ele fornece acesso controlado ao seu servidor de aplicativos.
-   Pode limitar a taxa.
-   Pode fazer solicitação de cache.

#### []{#example-0--server-go .anchor} **server.go:** Alvo

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

#### []{#example-0--application-go .anchor} **application.go:** Alvo real

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

#### []{#example-0--output-txt .anchor} **output.txt:** Resultados da execução

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
#### Leia a seguir

[Chain of Responsibility em Go []{.fa
.fa-arrow-right}](../../chain-of-responsibility/go/example.html){.btn
.btn-primary rel="next"}
:::

::: prev
#### Voltar

[[]{.fa .fa-arrow-left} Flyweight em
Go](../../flyweight/go/example.html){.btn .btn-default rel="prev"}
:::

## **Proxy** em outras linguagens

[![Proxy em
C#](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Proxy em C#"){.prog-lang-link}
[![Proxy em
C++](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Proxy em C++"){.prog-lang-link}
[![Proxy em
Java](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Proxy em Java"){.prog-lang-link}
[![Proxy em
PHP](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Proxy em PHP"){.prog-lang-link}
[![Proxy em
Python](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Proxy em Python"){.prog-lang-link}
[![Proxy em
Ruby](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Proxy em Ruby"){.prog-lang-link}
[![Proxy em
Rust](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Proxy em Rust"){.prog-lang-link}
[![Proxy em
Swift](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Proxy em Swift"){.prog-lang-link}
[![Proxy em
TypeScript](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Proxy em TypeScript"){.prog-lang-link}
::::::::::::::::::::

::::::: {.prom .banner-sidebar .banner .banner-striped .banner-examples .banner-removable .banner-removable-patterns data-id="DP: Examples-IDE" creative-id="examples-sidebar-pt-br" position="sidebar"}
:::::: {.banner-inner style="font-size: 14px; line-height: 21px;"}
::: banner-examples-img
[![](../../../../images/patterns/banners/examples-ide91ca.png?id=3115b4b548fb96b75974e2de8f4f49bc){width="215"
height="158" loading="lazy"
srcset="/images/patterns/banners/examples-ide-2x.png?id=93c007a6d157b97c28eb213f59d716ae 2x"}](../../book.html)
:::

::: banner-examples-fade
[ Arquivo com exemplos de código](../../book.html){.btn .btn-secondary
.btn-block}
:::

::: {.banner-examples-text style="margin-top: 1rem;"}
Compre o eBook **Mergulho nos Padrões de Projeto** e tenha acesso ao
arquivo com dúzias de exemplos detalhados que podem ser abertos
diretamente em seu IDE.

[ Saiba mais...](../../book.html){.btn .btn-secondary .btn-block}
:::
::::::
:::::::
::::::::::::::::::::::::::
:::::::::::::::::::::::::::
