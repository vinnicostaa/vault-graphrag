:::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [Patrones de
diseño](../../../design-patterns.html){.type} /
[Proxy](../../proxy.html){.type} / [PHP](../../php.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Proxy](../../../../images/patterns/cards/proxy-minid8d7.png?id=25890b11e7dc5af29625ccd0678b63a8){srcset="/images/patterns/cards/proxy-mini-2x.png?id=8638fac9dc08c992852492f9cb29d9c6 2x"}
:::

# **Proxy** en PHP {#proxy-en-php .pattern-example-title-block-title}
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
 [index](#example-0--index-php)
:::

::: en-file
 [Output](#example-0--Output-txt)
:::

::: en-intro
 [Ejemplo del mundo real](#example-1)
:::

::: en-file
 [index](#example-1--index-php)
:::

::: en-file
 [Output](#example-1--Output-txt)
:::
:::::::::::

**Complejidad:** []{.fa-stars}

**Popularidad:** []{.fa-stars}

**Ejemplos de uso:** Aunque el patrón Proxy no es un invitado habitual
en la mayoría de aplicaciones PHP, resulta de mucha utilidad en algunos
casos especiales. Es insustituible cuando queremos añadir algunos
comportamientos adicionales a un objeto de una clase existente sin
cambiar el código cliente.

**Identificación:** Los proxies delegan todo el trabajo real a otro
objeto. Cada método proxy debe, al final, referirse a un objeto de
servicio, a no ser que el proxy sea una subclase de un servicio.
::::::::::::::

::: {#nav-tab .nav .nav-tabs .nav-justified role="tablist"}
[Ejemplo conceptual](#example-0){#example-0-tab .nav-item .nav-link
.active toggle="tab" role="tab" aria-controls="example-0"
aria-selected="true"} [Ejemplo del mundo
real](#example-1){#example-1-tab .nav-item .nav-link toggle="tab"
role="tab" aria-controls="example-1" aria-selected="true"}
:::

::::: {#nav-tabContent .pattern-example-tab-content .tab-content}
::: {#example-0 .tab-pane .show .active role="tabpanel" aria-labelledby="example-0-tab" style="padding-top: 24px"}
## Ejemplo conceptual {#example-0-title}

Este ejemplo ilustra la estructura del patrón de diseño **Proxy** y se
centra en las siguientes preguntas:

-   ¿De qué clases se compone?
-   ¿Qué papeles juegan esas clases?
-   ¿De qué forma se relacionan los elementos del patrón?

Después de conocer la estructura del patrón, será más fácil comprender
el siguiente ejemplo basado en un caso de uso real de PHP.

#### []{#example-0--index-php .anchor} **index.php:** Ejemplo conceptual

<figure class="code">
<pre class="code" lang="php"><code>&lt;?php

namespace RefactoringGuru\Proxy\Conceptual;

/**
 * The Subject interface declares common operations for both RealSubject and the
 * Proxy. As long as the client works with RealSubject using this interface,
 * you&#39;ll be able to pass it a proxy instead of a real subject.
 */
interface Subject
{
    public function request(): void;
}

/**
 * The RealSubject contains some core business logic. Usually, RealSubjects are
 * capable of doing some useful work which may also be very slow or sensitive -
 * e.g. correcting input data. A Proxy can solve these issues without any
 * changes to the RealSubject&#39;s code.
 */
class RealSubject implements Subject
{
    public function request(): void
    {
        echo &quot;RealSubject: Handling request.\n&quot;;
    }
}

/**
 * The Proxy has an interface identical to the RealSubject.
 */
class Proxy implements Subject
{
    /**
     * @var RealSubject
     */
    private $realSubject;

    /**
     * The Proxy maintains a reference to an object of the RealSubject class. It
     * can be either lazy-loaded or passed to the Proxy by the client.
     */
    public function __construct(RealSubject $realSubject)
    {
        $this-&gt;realSubject = $realSubject;
    }

    /**
     * The most common applications of the Proxy pattern are lazy loading,
     * caching, controlling the access, logging, etc. A Proxy can perform one of
     * these things and then, depending on the result, pass the execution to the
     * same method in a linked RealSubject object.
     */
    public function request(): void
    {
        if ($this-&gt;checkAccess()) {
            $this-&gt;realSubject-&gt;request();
            $this-&gt;logAccess();
        }
    }

    private function checkAccess(): bool
    {
        // Some real checks should go here.
        echo &quot;Proxy: Checking access prior to firing a real request.\n&quot;;

        return true;
    }

    private function logAccess(): void
    {
        echo &quot;Proxy: Logging the time of request.\n&quot;;
    }
}

/**
 * The client code is supposed to work with all objects (both subjects and
 * proxies) via the Subject interface in order to support both real subjects and
 * proxies. In real life, however, clients mostly work with their real subjects
 * directly. In this case, to implement the pattern more easily, you can extend
 * your proxy from the real subject&#39;s class.
 */
function clientCode(Subject $subject)
{
    // ...

    $subject-&gt;request();

    // ...
}

echo &quot;Client: Executing the client code with a real subject:\n&quot;;
$realSubject = new RealSubject();
clientCode($realSubject);

echo &quot;\n&quot;;

echo &quot;Client: Executing the same client code with a proxy:\n&quot;;
$proxy = new Proxy($realSubject);
clientCode($proxy);</code></pre>
</figure>

#### []{#example-0--Output-txt .anchor} **Output.txt:** Resultado de la ejecución

<figure class="code">
<pre class="code" lang="output"><code>Client: Executing the client code with a real subject:
RealSubject: Handling request.

Client: Executing the same client code with a proxy:
Proxy: Checking access prior to firing a real request.
RealSubject: Handling request.
Proxy: Logging the time of request.</code></pre>
</figure>
:::

::: {#example-1 .tab-pane role="tabpanel" aria-labelledby="example-1-tab" style="padding-top: 24px"}
## Ejemplo del mundo real {#example-1-title}

Hay incontables maneras de utilizar **proxies**: almacenamiento en
caché, registro, control de acceso, inicialización retardada, etc. Este
ejemplo demuestra cómo puede el patrón Proxy mejorar el rendimiento de
un objeto de descarga guardando en caché sus resultados.

#### []{#example-1--index-php .anchor} **index.php:** Ejemplo del mundo real

<figure class="code">
<pre class="code" lang="php"><code>&lt;?php

namespace RefactoringGuru\Proxy\RealWorld;

/**
 * The Subject interface describes the interface of a real object.
 *
 * The truth is that many real apps may not have this interface clearly defined.
 * If you&#39;re in that boat, your best bet would be to extend the Proxy from one
 * of your existing application classes. If that&#39;s awkward, then extracting a
 * proper interface should be your first step.
 */
interface Downloader
{
    public function download(string $url): string;
}

/**
 * The Real Subject does the real job, albeit not in the most efficient way.
 * When a client tries to download the same file for the second time, our
 * downloader does just that, instead of fetching the result from cache.
 */
class SimpleDownloader implements Downloader
{
    public function download(string $url): string
    {
        echo &quot;Downloading a file from the Internet.\n&quot;;
        $result = file_get_contents($url);
        echo &quot;Downloaded bytes: &quot; . strlen($result) . &quot;\n&quot;;

        return $result;
    }
}

/**
 * The Proxy class is our attempt to make the download more efficient. It wraps
 * the real downloader object and delegates it the first download calls. The
 * result is then cached, making subsequent calls return an existing file
 * instead of downloading it again.
 *
 * Note that the Proxy MUST implement the same interface as the Real Subject.
 */
class CachingDownloader implements Downloader
{
    /**
     * @var SimpleDownloader
     */
    private $downloader;

    /**
     * @var string[]
     */
    private $cache = [];

    public function __construct(SimpleDownloader $downloader)
    {
        $this-&gt;downloader = $downloader;
    }

    public function download(string $url): string
    {
        if (!isset($this-&gt;cache[$url])) {
            echo &quot;CacheProxy MISS. &quot;;
            $result = $this-&gt;downloader-&gt;download($url);
            $this-&gt;cache[$url] = $result;
        } else {
            echo &quot;CacheProxy HIT. Retrieving result from cache.\n&quot;;
        }
        return $this-&gt;cache[$url];
    }
}

/**
 * The client code may issue several similar download requests. In this case,
 * the caching proxy saves time and traffic by serving results from cache.
 *
 * The client is unaware that it works with a proxy because it works with
 * downloaders via the abstract interface.
 */
function clientCode(Downloader $subject)
{
    // ...

    $result = $subject-&gt;download(&quot;http://example.com/&quot;);

    // Duplicate download requests could be cached for a speed gain.

    $result = $subject-&gt;download(&quot;http://example.com/&quot;);

    // ...
}

echo &quot;Executing client code with real subject:\n&quot;;
$realSubject = new SimpleDownloader();
clientCode($realSubject);

echo &quot;\n&quot;;

echo &quot;Executing the same client code with a proxy:\n&quot;;
$proxy = new CachingDownloader($realSubject);
clientCode($proxy);</code></pre>
</figure>

#### []{#example-1--Output-txt .anchor} **Output.txt:** Resultado de la ejecución

<figure class="code">
<pre class="code" lang="output"><code>Executing client code with real subject:
Downloading a file from the Internet.
Downloaded bytes: 1270
Downloading a file from the Internet.
Downloaded bytes: 1270

Executing the same client code with a proxy:
CacheProxy MISS. Downloading a file from the Internet.
Downloaded bytes: 1270
CacheProxy HIT. Retrieving result from cache.</code></pre>
</figure>
:::
:::::

::: {#nav-tab .nav .nav-tabs .nav-tabs-bottom .nav-justified role="tablist"}
[Ejemplo conceptual](#example-0){#example-0-tab-bottom .nav-item
.nav-link .active toggle="tab" role="tab" aria-controls="example-0"
aria-selected="true"} [Ejemplo del mundo
real](#example-1){#example-1-tab-bottom .nav-item .nav-link toggle="tab"
role="tab" aria-controls="example-1" aria-selected="true"}
:::

::: next
#### Leer siguiente

[Chain of Responsibility en PHP []{.fa
.fa-arrow-right}](../../chain-of-responsibility/php/example.html){.btn
.btn-primary rel="next"}
:::

::: prev
#### Volver

[[]{.fa .fa-arrow-left} Flyweight en
PHP](../../flyweight/php/example.html){.btn .btn-default rel="prev"}
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
Go](../../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Proxy en Go"){.prog-lang-link}
[![Proxy en
Java](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Proxy en Java"){.prog-lang-link}
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
:::::::::::::::::::::::::

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
:::::::::::::::::::::::::::::::
::::::::::::::::::::::::::::::::
