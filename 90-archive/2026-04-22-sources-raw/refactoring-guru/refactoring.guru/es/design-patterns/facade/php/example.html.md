:::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [Patrones de
diseño](../../../design-patterns.html){.type} /
[Facade](../../facade.html){.type} / [PHP](../../php.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Facade](../../../../images/patterns/cards/facade-mini14fd.png?id=71ad6fa98b168c11cb3a1a9517dedf78){srcset="/images/patterns/cards/facade-mini-2x.png?id=d4cc6a5d81a31143cc665f7ac1481ac8 2x"}
:::

# **Facade** en PHP {#facade-en-php .pattern-example-title-block-title}
::::

:::::::::::::: pattern-example-body
::: pattern-example-brief
**Facade** es un patrón de diseño estructural que proporciona una
interfaz simplificada (pero limitada) a un sistema complejo de clases,
bibliotecas o \_frameworks\_.

El patrón Facade disminuye la complejidad general de la aplicación, al
mismo tiempo que ayuda a mover dependencias no deseadas a un solo lugar.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Aprende más sobre el patrón Facade ](../../facade.html){.btn .btn-lg
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

**Ejemplos de uso:** El patrón Facade se utiliza habitualmente en
aplicaciones PHP, donde las clases fachada simplifican el trabajo con
bibliotecas o API complejas.

**Identificación:** El patrón Facade se puede reconocer en una clase con
una interfaz simple, pero que delega la mayor parte del trabajo a otras
clases. Normalmente, las fachadas gestionan todo el ciclo de vida de los
objetos que utilizan.
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

Este ejemplo ilustra la estructura del patrón de diseño **Facade** y se
centra en las siguientes preguntas:

-   ¿De qué clases se compone?
-   ¿Qué papeles juegan esas clases?
-   ¿De qué forma se relacionan los elementos del patrón?

Después de conocer la estructura del patrón, será más fácil comprender
el siguiente ejemplo basado en un caso de uso real de PHP.

#### []{#example-0--index-php .anchor} **index.php:** Ejemplo conceptual

<figure class="code">
<pre class="code" lang="php"><code>&lt;?php

namespace RefactoringGuru\Facade\Conceptual;

/**
 * The Facade class provides a simple interface to the complex logic of one or
 * several subsystems. The Facade delegates the client requests to the
 * appropriate objects within the subsystem. The Facade is also responsible for
 * managing their lifecycle. All of this shields the client from the undesired
 * complexity of the subsystem.
 */
class Facade
{
    protected $subsystem1;

    protected $subsystem2;

    /**
     * Depending on your application&#39;s needs, you can provide the Facade with
     * existing subsystem objects or force the Facade to create them on its own.
     */
    public function __construct(
        Subsystem1 $subsystem1 = null,
        Subsystem2 $subsystem2 = null
    ) {
        $this-&gt;subsystem1 = $subsystem1 ?: new Subsystem1();
        $this-&gt;subsystem2 = $subsystem2 ?: new Subsystem2();
    }

    /**
     * The Facade&#39;s methods are convenient shortcuts to the sophisticated
     * functionality of the subsystems. However, clients get only to a fraction
     * of a subsystem&#39;s capabilities.
     */
    public function operation(): string
    {
        $result = &quot;Facade initializes subsystems:\n&quot;;
        $result .= $this-&gt;subsystem1-&gt;operation1();
        $result .= $this-&gt;subsystem2-&gt;operation1();
        $result .= &quot;Facade orders subsystems to perform the action:\n&quot;;
        $result .= $this-&gt;subsystem1-&gt;operationN();
        $result .= $this-&gt;subsystem2-&gt;operationZ();

        return $result;
    }
}

/**
 * The Subsystem can accept requests either from the facade or client directly.
 * In any case, to the Subsystem, the Facade is yet another client, and it&#39;s not
 * a part of the Subsystem.
 */
class Subsystem1
{
    public function operation1(): string
    {
        return &quot;Subsystem1: Ready!\n&quot;;
    }

    // ...

    public function operationN(): string
    {
        return &quot;Subsystem1: Go!\n&quot;;
    }
}

/**
 * Some facades can work with multiple subsystems at the same time.
 */
class Subsystem2
{
    public function operation1(): string
    {
        return &quot;Subsystem2: Get ready!\n&quot;;
    }

    // ...

    public function operationZ(): string
    {
        return &quot;Subsystem2: Fire!\n&quot;;
    }
}

/**
 * The client code works with complex subsystems through a simple interface
 * provided by the Facade. When a facade manages the lifecycle of the subsystem,
 * the client might not even know about the existence of the subsystem. This
 * approach lets you keep the complexity under control.
 */
function clientCode(Facade $facade)
{
    // ...

    echo $facade-&gt;operation();

    // ...
}

/**
 * The client code may have some of the subsystem&#39;s objects already created. In
 * this case, it might be worthwhile to initialize the Facade with these objects
 * instead of letting the Facade create new instances.
 */
$subsystem1 = new Subsystem1();
$subsystem2 = new Subsystem2();
$facade = new Facade($subsystem1, $subsystem2);
clientCode($facade);</code></pre>
</figure>

#### []{#example-0--Output-txt .anchor} **Output.txt:** Resultado de la ejecución

<figure class="code">
<pre class="code" lang="output"><code>Facade initializes subsystems:
Subsystem1: Ready!
Subsystem2: Get ready!
Facade orders subsystems to perform the action:
Subsystem1: Go!
Subsystem2: Fire!</code></pre>
</figure>
:::

::: {#example-1 .tab-pane role="tabpanel" aria-labelledby="example-1-tab" style="padding-top: 24px"}
## Ejemplo del mundo real {#example-1-title}

Piensa en el patrón **Facade** como un adaptador de simplicidad para
algunos subsistemas complejos. El patrón Facade aísla la complejidad
dentro de una única clase y permite al código de otras aplicaciones
utilizar la interfaz directa.

En este ejemplo, el patrón Facade esconde al código cliente la
complejidad de la API de YouTube y la biblioteca de FFmpeg. En lugar de
trabajar con decenas de clases, el cliente utiliza un método simple en
la fachada.

#### []{#example-1--index-php .anchor} **index.php:** Ejemplo del mundo real

<figure class="code">
<pre class="code" lang="php"><code>&lt;?php

namespace RefactoringGuru\Facade\RealWorld;

/**
 * The Facade provides a single method for downloading videos from YouTube. This
 * method hides all the complexity of the PHP network layer, YouTube API and the
 * video conversion library (FFmpeg).
 */
class YouTubeDownloader
{
    protected $youtube;
    protected $ffmpeg;

    /**
     * It is handy when the Facade can manage the lifecycle of the subsystem it
     * uses.
     */
    public function __construct(string $youtubeApiKey)
    {
        $this-&gt;youtube = new YouTube($youtubeApiKey);
        $this-&gt;ffmpeg = new FFMpeg();
    }

    /**
     * The Facade provides a simple method for downloading video and encoding it
     * to a target format (for the sake of simplicity, the real-world code is
     * commented-out).
     */
    public function downloadVideo(string $url): void
    {
        echo &quot;Fetching video metadata from youtube...\n&quot;;
        // $title = $this-&gt;youtube-&gt;fetchVideo($url)-&gt;getTitle();
        echo &quot;Saving video file to a temporary file...\n&quot;;
        // $this-&gt;youtube-&gt;saveAs($url, &quot;video.mpg&quot;);

        echo &quot;Processing source video...\n&quot;;
        // $video = $this-&gt;ffmpeg-&gt;open(&#39;video.mpg&#39;);
        echo &quot;Normalizing and resizing the video to smaller dimensions...\n&quot;;
        // $video
        //     -&gt;filters()
        //     -&gt;resize(new FFMpeg\Coordinate\Dimension(320, 240))
        //     -&gt;synchronize();
        echo &quot;Capturing preview image...\n&quot;;
        // $video
        //     -&gt;frame(FFMpeg\Coordinate\TimeCode::fromSeconds(10))
        //     -&gt;save($title . &#39;frame.jpg&#39;);
        echo &quot;Saving video in target formats...\n&quot;;
        // $video
        //     -&gt;save(new FFMpeg\Format\Video\X264(), $title . &#39;.mp4&#39;)
        //     -&gt;save(new FFMpeg\Format\Video\WMV(), $title . &#39;.wmv&#39;)
        //     -&gt;save(new FFMpeg\Format\Video\WebM(), $title . &#39;.webm&#39;);
        echo &quot;Done!\n&quot;;
    }
}

/**
 * The YouTube API subsystem.
 */
class YouTube
{
    public function fetchVideo(): string
    {
      /* ... */
    }

    public function saveAs(string $path): void
    {
      /* ... */
    }

    // ...more methods and classes...
}

/**
 * The FFmpeg subsystem (a complex video/audio conversion library).
 */
class FFMpeg
{
    public static function create(): FFMpeg
    {
      /* ... */
    }

    public function open(string $video): void
    {
      /* ... */
    }

    // ...more methods and classes... RU: ...дополнительные методы и классы...
}

class FFMpegVideo
{
    public function filters(): self
    {
      /* ... */
    }

    public function resize(): self
    {
      /* ... */
    }

    public function synchronize(): self
    {
      /* ... */
    }

    public function frame(): self
    {
      /* ... */
    }

    public function save(string $path): self
    {
      /* ... */
    }

    // ...more methods and classes... RU: ...дополнительные методы и классы...
}


/**
 * The client code does not depend on any subsystem&#39;s classes. Any changes
 * inside the subsystem&#39;s code won&#39;t affect the client code. You will only need
 * to update the Facade.
 */
function clientCode(YouTubeDownloader $facade)
{
    // ...

    $facade-&gt;downloadVideo(&quot;https://www.youtube.com/watch?v=QH2-TGUlwu4&quot;);

    // ...
}

$facade = new YouTubeDownloader(&quot;APIKEY-XXXXXXXXX&quot;);
clientCode($facade);</code></pre>
</figure>

#### []{#example-1--Output-txt .anchor} **Output.txt:** Resultado de la ejecución

<figure class="code">
<pre class="code" lang="output"><code>Fetching video metadata from youtube...
Saving video file to a temporary file...
Processing source video...
Normalizing and resizing the video to smaller dimensions...
Capturing preview image...
Saving video in target formats...
Done!</code></pre>
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

[Flyweight en PHP []{.fa
.fa-arrow-right}](../../flyweight/php/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Volver

[[]{.fa .fa-arrow-left} Decorator en
PHP](../../decorator/php/example.html){.btn .btn-default rel="prev"}
:::

## **Facade** en otros lenguajes

[![Facade en
C#](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Facade en C#"){.prog-lang-link}
[![Facade en
C++](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Facade en C++"){.prog-lang-link}
[![Facade en
Go](../../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Facade en Go"){.prog-lang-link}
[![Facade en
Java](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Facade en Java"){.prog-lang-link}
[![Facade en
Python](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Facade en Python"){.prog-lang-link}
[![Facade en
Ruby](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Facade en Ruby"){.prog-lang-link}
[![Facade en
Rust](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Facade en Rust"){.prog-lang-link}
[![Facade en
Swift](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Facade en Swift"){.prog-lang-link}
[![Facade en
TypeScript](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Facade en TypeScript"){.prog-lang-link}
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
