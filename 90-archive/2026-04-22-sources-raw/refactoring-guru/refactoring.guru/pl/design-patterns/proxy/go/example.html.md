::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [Wzorce
projektowe](../../../design-patterns.html){.type} /
[Pełnomocnik](../../proxy.html){.type} / [Go](../../go.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Pełnomocnik](../../../../images/patterns/cards/proxy-minid8d7.png?id=25890b11e7dc5af29625ccd0678b63a8){srcset="/images/patterns/cards/proxy-mini-2x.png?id=8638fac9dc08c992852492f9cb29d9c6 2x"}
:::

# **Pełnomocnik** w języku Go {#pełnomocnik-w-języku-go .pattern-example-title-block-title}
::::

:::::::::::::: pattern-example-body
::: pattern-example-brief
**Pełnomocnik** to strukturalny wzorzec projektowy według którego
obiekt-usługodawca używany przez klienta jest zastępowany przez obiekt
zastępczy, zwany pełnomocnikiem. Pełnomocnik przechwytuje żądania od
klienta, wykonuje jakąś pracę (kontrola dostępu, zarządzanie pamięcią
podręczną, itp.) a następnie przekazuje żądanie usługodawcy.

Obiekt będący pełnomocnikiem ma ten sam interfejs co usługodawca, co
czyni go wymienialnym z obiektem usługodawcy dotychczas przekazywanym
klientowi.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Dowiedz się więcej o Pełnomocnik ](../../proxy.html){.btn .btn-lg
.btn-primary}
:::

::::::::::: {.sidebar-navigation .with-scroll}
::: en-title
Nawigacja
:::

::: en-intro
 [Intro](#)
:::

::: en-intro
 [Przykład koncepcyjny](#example-0)
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

## Przykład koncepcyjny {#example-0-title}

Serwer webowy, jak Nginx, może pełnić rolę pośrednika dla serwera
aplikacji:

-   Kontroluje dostęp do serwera aplikacyjnego.
-   Steruje szybkością transmisji.
-   Może przechowywać żądania w swej pamięci podręcznej.

#### []{#example-0--server-go .anchor} **server.go:** Podmiot

<figure class="code">
<pre class="code" lang="go"><code>package main

type server interface {
    handleRequest(string, string) (int, string)
}</code></pre>
</figure>

#### []{#example-0--nginx-go .anchor} **nginx.go:** Pośrednik

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

#### []{#example-0--application-go .anchor} **application.go:** Prawdziwy podmiot

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

#### []{#example-0--main-go .anchor} **main.go:** Kod klienta

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

#### []{#example-0--output-txt .anchor} **output.txt:** Wynik działania

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
#### Czytaj dalej

[Łańcuch zobowiązań w języku Go []{.fa
.fa-arrow-right}](../../chain-of-responsibility/go/example.html){.btn
.btn-primary rel="next"}
:::

::: prev
#### Wróć

[[]{.fa .fa-arrow-left} Pyłek w języku
Go](../../flyweight/go/example.html){.btn .btn-default rel="prev"}
:::

## **Pełnomocnik** w innych językach

[![Pełnomocnik w języku
C#](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Pełnomocnik w języku C#"){.prog-lang-link}
[![Pełnomocnik w języku
C++](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Pełnomocnik w języku C++"){.prog-lang-link}
[![Pełnomocnik w języku
Java](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Pełnomocnik w języku Java"){.prog-lang-link}
[![Pełnomocnik w języku
PHP](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Pełnomocnik w języku PHP"){.prog-lang-link}
[![Pełnomocnik w języku
Python](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Pełnomocnik w języku Python"){.prog-lang-link}
[![Pełnomocnik w języku
Ruby](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Pełnomocnik w języku Ruby"){.prog-lang-link}
[![Pełnomocnik w języku
Rust](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Pełnomocnik w języku Rust"){.prog-lang-link}
[![Pełnomocnik w języku
Swift](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Pełnomocnik w języku Swift"){.prog-lang-link}
[![Pełnomocnik w języku
TypeScript](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Pełnomocnik w języku TypeScript"){.prog-lang-link}
::::::::::::::::::::

::::::: {.prom .banner-sidebar .banner .banner-striped .banner-examples .banner-removable .banner-removable-patterns data-id="DP: Examples-IDE" creative-id="examples-sidebar-pl" position="sidebar"}
:::::: {.banner-inner style="font-size: 14px; line-height: 21px;"}
::: banner-examples-img
[![](../../../../images/patterns/banners/examples-ide91ca.png?id=3115b4b548fb96b75974e2de8f4f49bc){width="215"
height="158" loading="lazy"
srcset="/images/patterns/banners/examples-ide-2x.png?id=93c007a6d157b97c28eb213f59d716ae 2x"}](../../book.html)
:::

::: banner-examples-fade
[ Archiwum\
przykładów kodu](../../book.html){.btn .btn-secondary .btn-block}
:::

::: {.banner-examples-text style="margin-top: 1rem;"}
Kupując eBook **Wzorce projektowe. Nowoczesny podręcznik** zyskujesz
dostęp do archiwum zawierającego mnóstwo szczegółowych przykładów, które
można od razu wkleić do swojego preferowanego zintegrowanego środowiska
programistycznego.

[ Dowiedz się więcej...](../../book.html){.btn .btn-secondary
.btn-block}
:::
::::::
:::::::
::::::::::::::::::::::::::
:::::::::::::::::::::::::::
