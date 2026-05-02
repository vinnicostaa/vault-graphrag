:::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [디자인
패턴들](../../../design-patterns.html){.type} /
[퍼사드](../../facade.html){.type} / [PHP](../../php.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![퍼사드](../../../../images/patterns/cards/facade-mini14fd.png?id=71ad6fa98b168c11cb3a1a9517dedf78){srcset="/images/patterns/cards/facade-mini-2x.png?id=d4cc6a5d81a31143cc665f7ac1481ac8 2x"}
:::

# PHP로 작성된 **퍼사드** {#php로-작성된-퍼사드 .pattern-example-title-block-title}
::::

:::::::::::::: pattern-example-body
::: pattern-example-brief
**퍼사드**는 구조 디자인 패턴이며 간단한 (그러나 제한된 인터페이스를
클래스, 라이브러리 또는 프레임워크로 구성된 복잡한 시스템에 제공합니다.

퍼사드는 앱의 전반적인 복잡성을 줄이는 동시에 원치 않는 의존성들을 한
곳으로 옮기는 것을 돕습니다.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ 퍼사드에 대하여 더 자세히 알아보세요 ](../../facade.html){.btn .btn-lg
.btn-primary}
:::

::::::::::: {.sidebar-navigation .with-scroll}
::: en-title
내비게이션
:::

::: en-intro
 [소개](#)
:::

::: en-intro
 [개념적인 예시](#example-0)
:::

::: en-file
 [index](#example-0--index-php)
:::

::: en-file
 [Output](#example-0--Output-txt)
:::

::: en-intro
 [실제 사례 예시](#example-1)
:::

::: en-file
 [index](#example-1--index-php)
:::

::: en-file
 [Output](#example-1--Output-txt)
:::
:::::::::::

**복잡도:** []{.fa-stars}

**인기도:** []{.fa-stars}

**사용 사례들:** 퍼사드 패턴은 복잡한 라이브러리 및 API와 작업할 때 PHP
앱에 자주 사용됩니다.

**식별:** 퍼사드는 인터페이스가 간단하나 대부분의 작업을 다른 클래스들에
위임하는 클래스의 존재여부로 식별할 수 있으며 일반적으로 퍼사드들은
사용하는 객체들의 전체 수명 주기를 관리합니다.
::::::::::::::

::: {#nav-tab .nav .nav-tabs .nav-justified role="tablist"}
[개념적인 예시](#example-0){#example-0-tab .nav-item .nav-link .active
toggle="tab" role="tab" aria-controls="example-0" aria-selected="true"}
[실제 사례 예시](#example-1){#example-1-tab .nav-item .nav-link
toggle="tab" role="tab" aria-controls="example-1" aria-selected="true"}
:::

::::: {#nav-tabContent .pattern-example-tab-content .tab-content}
::: {#example-0 .tab-pane .show .active role="tabpanel" aria-labelledby="example-0-tab" style="padding-top: 24px"}
## 개념적인 예시 {#example-0-title}

이 예시는 **퍼사드** 패턴의 구조를 보여주고 다음 질문에 중점을 둡니다:

-   패턴은 어떤 클래스들로 구성되어 있나요?
-   이 클래스들은 어떤 역할을 하나요?
-   패턴의 요소들은 어떻게 서로 연관되어 있나요?

이 패턴의 구조를 배우면 실제 PHP 사용 사례를 기반으로 하는 다음 예시를
더욱 쉽게 이해할 수 있을 것입니다.

#### []{#example-0--index-php .anchor} **index.php:** 개념적인 예시

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

#### []{#example-0--Output-txt .anchor} **Output.txt:** 실행 결과

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
## 실제 사례 예시 {#example-1-title}

**퍼사드** 패턴을 복잡한 하위 시스템을 위한 단순성 어댑터로 생각하세요.
퍼사드는 단일 클래스 내에 복잡성을 고립시키며 다른 어플리케이션 코드가
간단명료한 인터페이스를 사용할 수 있도록 합니다.

이 예시에서는 퍼사드는 유튜브 API와 FFmpeg 라이브러리의 복잡성을
클라이언트 코드로부터 숨깁니다. 클라이언트는 수십 개의 클래스와 작업하는
대신 퍼사드와 간단한 메서드를 사용합니다.

#### []{#example-1--index-php .anchor} **index.php:** 실제 사례 예시

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

#### []{#example-1--Output-txt .anchor} **Output.txt:** 실행 결과

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
[개념적인 예시](#example-0){#example-0-tab-bottom .nav-item .nav-link
.active toggle="tab" role="tab" aria-controls="example-0"
aria-selected="true"} [실제 사례 예시](#example-1){#example-1-tab-bottom
.nav-item .nav-link toggle="tab" role="tab" aria-controls="example-1"
aria-selected="true"}
:::

::: next
#### 다음을 보세요

[PHP로 작성된 플라이웨이트 []{.fa
.fa-arrow-right}](../../flyweight/php/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### 돌아가기

[[]{.fa .fa-arrow-left} PHP로 작성된
데코레이터](../../decorator/php/example.html){.btn .btn-default
rel="prev"}
:::

## 다른 언어로 작성된 **퍼사드**

[![C#으로 작성된
퍼사드](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "C#으로 작성된 퍼사드"){.prog-lang-link}
[![C++로 작성된
퍼사드](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "C++로 작성된 퍼사드"){.prog-lang-link}
[![Go로 작성된
퍼사드](../../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Go로 작성된 퍼사드"){.prog-lang-link}
[![자바로 작성된
퍼사드](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "자바로 작성된 퍼사드"){.prog-lang-link}
[![파이썬으로 작성된
퍼사드](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "파이썬으로 작성된 퍼사드"){.prog-lang-link}
[![루비로 작성된
퍼사드](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "루비로 작성된 퍼사드"){.prog-lang-link}
[![러스트로 작성된
퍼사드](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "러스트로 작성된 퍼사드"){.prog-lang-link}
[![스위프트로 작성된
퍼사드](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "스위프트로 작성된 퍼사드"){.prog-lang-link}
[![타입스크립트로 작성된
퍼사드](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "타입스크립트로 작성된 퍼사드"){.prog-lang-link}
:::::::::::::::::::::::::

::::::: {.prom .banner-sidebar .banner .banner-striped .banner-examples .banner-removable .banner-removable-patterns data-id="DP: Examples-IDE" creative-id="examples-sidebar-ko" position="sidebar"}
:::::: {.banner-inner style="font-size: 14px; line-height: 21px;"}
::: banner-examples-img
[![](../../../../images/patterns/banners/examples-ide91ca.png?id=3115b4b548fb96b75974e2de8f4f49bc){width="215"
height="158" loading="lazy"
srcset="/images/patterns/banners/examples-ide-2x.png?id=93c007a6d157b97c28eb213f59d716ae 2x"}](../../book.html)
:::

::: banner-examples-fade
[ 예시들을 포함한 저장소](../../book.html){.btn .btn-secondary
.btn-block}
:::

::: {.banner-examples-text style="margin-top: 1rem;"}
전자책 **디자인 패턴에 뛰어들기**를 구매하면 IDE에서 바로 열 수 있는
수십 가지의 상세한 코드 예시들이 포함된 저장소를 사용할 수 있습니다.

[ 더 알아보기...](../../book.html){.btn .btn-secondary .btn-block}
:::
::::::
:::::::
:::::::::::::::::::::::::::::::
::::::::::::::::::::::::::::::::
