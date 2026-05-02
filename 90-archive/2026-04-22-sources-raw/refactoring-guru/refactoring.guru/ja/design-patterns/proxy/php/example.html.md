:::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} /
[デザインパターン](../../../design-patterns.html){.type} /
[Proxy](../../proxy.html){.type} / [PHP](../../php.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Proxy](../../../../images/patterns/cards/proxy-minid8d7.png?id=25890b11e7dc5af29625ccd0678b63a8){srcset="/images/patterns/cards/proxy-mini-2x.png?id=8638fac9dc08c992852492f9cb29d9c6 2x"}
:::

# **Proxy** を PHP で {#proxy-を-php-で .pattern-example-title-block-title}
::::

:::::::::::::: pattern-example-body
::: pattern-example-brief
**Proxy** は[、]{.chpule2}[
]{.chpuri2}構造に関するデザインパターンの一つで[、]{.chpule2}[
]{.chpuri2}クライアントが使う本物のサービス・オブジェクトの代理として機能するオブジェクト[
]{.chpule1}[（]{.chpuri1}プロキシー[）]{.chpule2}[
]{.chpuri2}を提供します[。]{.chpule2}[
]{.chpuri2}プロキシーは[、]{.chpule2}[
]{.chpuri2}アクセス制御[、]{.chpule2}[
]{.chpuri2}キャッシングなど[、]{.chpule2}[
]{.chpuri2}何らかの作業を行なった後[、]{.chpule2}[
]{.chpuri2}リクエストをサービス・オブジェクトに渡します[。]{.chpule2}[
]{.chpuri2}

プロキシー・オブジェクトはサービスと同じインターフェースを持ち[、]{.chpule2}[
]{.chpuri2}クライアントにとっては[、]{.chpule2}[
]{.chpuri2}本物のオブジェクトと交換可能です[。]{.chpule2}[ ]{.chpuri2}
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Proxy の詳細 ](../../proxy.html){.btn .btn-lg .btn-primary}
:::

::::::::::: {.sidebar-navigation .with-scroll}
::: en-title
ナビゲーション
:::

::: en-intro
 [はじめに](#)
:::

::: en-intro
 [概念的な例](#example-0)
:::

::: en-file
 [index](#example-0--index-php)
:::

::: en-file
 [Output](#example-0--Output-txt)
:::

::: en-intro
 [現実的な例](#example-1)
:::

::: en-file
 [index](#example-1--index-php)
:::

::: en-file
 [Output](#example-1--Output-txt)
:::
:::::::::::

**複雑度[：]{.chpule2}[ ]{.chpuri2}**[]{.fa-stars}

**人気度[：]{.chpule2}[ ]{.chpuri2}**[]{.fa-stars}

**使用例[：]{.chpule2}[ ]{.chpuri2}** Proxy パターンは[、]{.chpule2}[
]{.chpuri2}ほとんどの PHP アプリケーションにおいては[、]{.chpule2}[
]{.chpuri2}あまり見かけませんが[、]{.chpule2}[
]{.chpuri2}いくつかの特殊なケースでは便利です[。]{.chpule2}[
]{.chpuri2}何らかの既存クラスのオブジェクトに何らかの振る舞いを追加したいがクライアント・コードには手を加えたくない時[、]{.chpule2}[
]{.chpuri2}かけがえのないものです[。]{.chpule2}[ ]{.chpuri2}

**見つけ方[：]{.chpule2}[ ]{.chpuri2}**プロキシーは[、]{.chpule2}[
]{.chpuri2}実際の作業はすべて他のオブジェクトに委任します[。]{.chpule2}[
]{.chpuri2}プロキシーがサービスのサブクラスである場合を除き[、]{.chpule2}[
]{.chpuri2}プロキシーのメソッドのそれぞれは[、]{.chpule2}[
]{.chpuri2}最終的にはサービス・オブジェクトを参照するはずです[。]{.chpule2}[
]{.chpuri2}
::::::::::::::

::: {#nav-tab .nav .nav-tabs .nav-justified role="tablist"}
[概念的な例](#example-0){#example-0-tab .nav-item .nav-link .active
toggle="tab" role="tab" aria-controls="example-0" aria-selected="true"}
[現実的な例](#example-1){#example-1-tab .nav-item .nav-link toggle="tab"
role="tab" aria-controls="example-1" aria-selected="true"}
:::

::::: {#nav-tabContent .pattern-example-tab-content .tab-content}
::: {#example-0 .tab-pane .show .active role="tabpanel" aria-labelledby="example-0-tab" style="padding-top: 24px"}
## 概念的な例 {#example-0-title}

この例は[、]{.chpule2}[ ]{.chpuri2}**Proxy**
デザインパターンの構造を説明するためのものです[。]{.chpule2}[
]{.chpuri2}以下の質問に答えることを目的としています[：]{.chpule2}[
]{.chpuri2}

-   どういうクラスからできているか[？]{.chpule2}[ ]{.chpuri2}
-   それぞれのクラスの役割は[？]{.chpule2}[ ]{.chpuri2}
-   パターンの要素同士はどう関係しているのか[？]{.chpule2}[ ]{.chpuri2}

ここでパターンの構造を学んだ後だと[、]{.chpule2}[
]{.chpuri2}これに続く[、]{.chpule2}[ ]{.chpuri2}現実世界の PHP
でのユースケースが理解しやすくなります[。]{.chpule2}[ ]{.chpuri2}

#### []{#example-0--index-php .anchor} **index.php:** 概念的な例

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

#### []{#example-0--Output-txt .anchor} **Output.txt:** 実行結果

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
## 現実的な例 {#example-1-title}

キャッシング[、]{.chpule2}[ ]{.chpuri2}ロギング[、]{.chpule2}[
]{.chpuri2}アクセス制御[、]{.chpule2}[
]{.chpuri2}遅延初期化など[、]{.chpule2}[ ]{.chpuri2}**Proxy**
の用途は数限りなくあります[。]{.chpule2}[ ]{.chpuri2}この例では Proxy
パターンを使い[、]{.chpule2}[
]{.chpuri2}結果をキャッシングすることによって[、]{.chpule2}[
]{.chpuri2}ダウンローダー・オブジェクトの効率向上を達成する方法を紹介します[。]{.chpule2}[
]{.chpuri2}

#### []{#example-1--index-php .anchor} **index.php:** 現実的な例

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

#### []{#example-1--Output-txt .anchor} **Output.txt:** 実行結果

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
[概念的な例](#example-0){#example-0-tab-bottom .nav-item .nav-link
.active toggle="tab" role="tab" aria-controls="example-0"
aria-selected="true"} [現実的な例](#example-1){#example-1-tab-bottom
.nav-item .nav-link toggle="tab" role="tab" aria-controls="example-1"
aria-selected="true"}
:::

::: next
#### 次を読む

[Chain of Responsibility を PHP で []{.fa
.fa-arrow-right}](../../chain-of-responsibility/php/example.html){.btn
.btn-primary rel="next"}
:::

::: prev
#### 戻る

[[]{.fa .fa-arrow-left} Flyweight を PHP
で](../../flyweight/php/example.html){.btn .btn-default rel="prev"}
:::

## 他言語での **Proxy**

[![Proxy を C#
で](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Proxy を C# で"){.prog-lang-link}
[![Proxy を C++
で](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Proxy を C++ で"){.prog-lang-link}
[![Proxy を Go
で](../../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Proxy を Go で"){.prog-lang-link}
[![Proxy を Java
で](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Proxy を Java で"){.prog-lang-link}
[![Proxy を Python
で](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Proxy を Python で"){.prog-lang-link}
[![Proxy を Ruby
で](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Proxy を Ruby で"){.prog-lang-link}
[![Proxy を Rust
で](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Proxy を Rust で"){.prog-lang-link}
[![Proxy を Swift
で](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Proxy を Swift で"){.prog-lang-link}
[![Proxy を TypeScript
で](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Proxy を TypeScript で"){.prog-lang-link}
:::::::::::::::::::::::::

::::::: {.prom .banner-sidebar .banner .banner-striped .banner-examples .banner-removable .banner-removable-patterns data-id="DP: Examples-IDE" creative-id="examples-sidebar-ja" position="sidebar"}
:::::: {.banner-inner style="font-size: 14px; line-height: 21px;"}
::: banner-examples-img
[![](../../../../images/patterns/banners/examples-ide91ca.png?id=3115b4b548fb96b75974e2de8f4f49bc){width="215"
height="158" loading="lazy"
srcset="/images/patterns/banners/examples-ide-2x.png?id=93c007a6d157b97c28eb213f59d716ae 2x"}](../../book.html)
:::

::: banner-examples-fade
[ コード例の書庫](../../book.html){.btn .btn-secondary .btn-block}
:::

::: {.banner-examples-text style="margin-top: 1rem;"}
**直撃！デザインパターン**を電子書籍で購入し、IDE
で開くことができる数多くの詳細な例を含む書庫にアクセス。

[ 詳細](../../book.html){.btn .btn-secondary .btn-block}
:::
::::::
:::::::
:::::::::::::::::::::::::::::::
::::::::::::::::::::::::::::::::
