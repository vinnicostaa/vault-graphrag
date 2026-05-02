:::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} /
[デザインパターン](../../../design-patterns.html){.type} /
[Bridge](../../bridge.html){.type} / [PHP](../../php.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Bridge](../../../../images/patterns/cards/bridge-mini9e60.png?id=b389101d8ee8e23ffa1b534c704d0774){srcset="/images/patterns/cards/bridge-mini-2x.png?id=2622384cf623ed150ee9c21a0812dd87 2x"}
:::

# **Bridge** を PHP で {#bridge-を-php-で .pattern-example-title-block-title}
::::

:::::::::::::: pattern-example-body
::: pattern-example-brief
**Bridge** は[、]{.chpule2}[
]{.chpuri2}構造に関するデザインパターンの一つで[、]{.chpule2}[
]{.chpuri2}ビジネス・ロジックや巨大なクラスを独立して開発可能なクラス階層に分割します[。]{.chpule2}[
]{.chpuri2}

階層の一つ[ ]{.chpule1}[（]{.chpuri1}抽象化と呼ばれる[）]{.chpule2}[
]{.chpuri2}は[、]{.chpule2}[ ]{.chpuri2}二つ目の階層[
]{.chpule1}[（]{.chpuri1}実装[）]{.chpule2}[
]{.chpuri2}のオブジェクトへの参照を持ちます[。]{.chpule2}[
]{.chpuri2}抽象化階層は[、]{.chpule2}[
]{.chpuri2}その呼び出しのいくつか[
]{.chpule1}[（]{.chpuri1}場合によっては大多数[）]{.chpule2}[
]{.chpuri2}を実装階層のオブジェクトに委任します[。]{.chpule2}[
]{.chpuri2}すべての実装は[、]{.chpule2}[
]{.chpuri2}共通のインターフェースを持っているので[、]{.chpule2}[
]{.chpuri2}抽象化の中で入れ替え可能です[。]{.chpule2}[ ]{.chpuri2}
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Bridge の詳細 ](../../bridge.html){.btn .btn-lg .btn-primary}
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

**使用例[：]{.chpule2}[ ]{.chpuri2}** Bridge パターンは[、]{.chpule2}[
]{.chpuri2}複数の種類のデータベース・サーバーをサポートする時[、]{.chpule2}[
]{.chpuri2}あるいはある種の API プロバイダー[
]{.chpule1}[（]{.chpuri1}クラウド・プラットフォーム[、]{.chpule2}[
]{.chpuri2}ソーシャル・ネットワークなど[）]{.chpule2}[
]{.chpuri2}を複数利用したい場合に特に便利です[。]{.chpule2}[ ]{.chpuri2}

**見つけ方[：]{.chpule2}[ ]{.chpuri2}** Bridge は[、]{.chpule2}[
]{.chpuri2}制御するものと[、]{.chpule2}[
]{.chpuri2}それが依存するいくつかの異なるプラットフォームとが明らかに分かれていることから識別できます[。]{.chpule2}[
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

この例は[、]{.chpule2}[ ]{.chpuri2}**Bridge**
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

namespace RefactoringGuru\Bridge\Conceptual;

/**
 * The Abstraction defines the interface for the &quot;control&quot; part of the two class
 * hierarchies. It maintains a reference to an object of the Implementation
 * hierarchy and delegates all of the real work to this object.
 */
class Abstraction
{
    /**
     * @var Implementation
     */
    protected $implementation;

    public function __construct(Implementation $implementation)
    {
        $this-&gt;implementation = $implementation;
    }

    public function operation(): string
    {
        return &quot;Abstraction: Base operation with:\n&quot; .
            $this-&gt;implementation-&gt;operationImplementation();
    }
}

/**
 * You can extend the Abstraction without changing the Implementation classes.
 */
class ExtendedAbstraction extends Abstraction
{
    public function operation(): string
    {
        return &quot;ExtendedAbstraction: Extended operation with:\n&quot; .
            $this-&gt;implementation-&gt;operationImplementation();
    }
}

/**
 * The Implementation defines the interface for all implementation classes. It
 * doesn&#39;t have to match the Abstraction&#39;s interface. In fact, the two
 * interfaces can be entirely different. Typically the Implementation interface
 * provides only primitive operations, while the Abstraction defines higher-
 * level operations based on those primitives.
 */
interface Implementation
{
    public function operationImplementation(): string;
}

/**
 * Each Concrete Implementation corresponds to a specific platform and
 * implements the Implementation interface using that platform&#39;s API.
 */
class ConcreteImplementationA implements Implementation
{
    public function operationImplementation(): string
    {
        return &quot;ConcreteImplementationA: Here&#39;s the result on the platform A.\n&quot;;
    }
}

class ConcreteImplementationB implements Implementation
{
    public function operationImplementation(): string
    {
        return &quot;ConcreteImplementationB: Here&#39;s the result on the platform B.\n&quot;;
    }
}

/**
 * Except for the initialization phase, where an Abstraction object gets linked
 * with a specific Implementation object, the client code should only depend on
 * the Abstraction class. This way the client code can support any abstraction-
 * implementation combination.
 */
function clientCode(Abstraction $abstraction)
{
    // ...

    echo $abstraction-&gt;operation();

    // ...
}

/**
 * The client code should be able to work with any pre-configured abstraction-
 * implementation combination.
 */
$implementation = new ConcreteImplementationA();
$abstraction = new Abstraction($implementation);
clientCode($abstraction);

echo &quot;\n&quot;;

$implementation = new ConcreteImplementationB();
$abstraction = new ExtendedAbstraction($implementation);
clientCode($abstraction);</code></pre>
</figure>

#### []{#example-0--Output-txt .anchor} **Output.txt:** 実行結果

<figure class="code">
<pre class="code" lang="output"><code>Abstraction: Base operation with:
ConcreteImplementationA: Here&#39;s the result on the platform A.

ExtendedAbstraction: Extended operation with:
ConcreteImplementationB: Here&#39;s the result on the platform B.</code></pre>
</figure>
:::

::: {#example-1 .tab-pane role="tabpanel" aria-labelledby="example-1-tab" style="padding-top: 24px"}
## 現実的な例 {#example-1-title}

この例では[、]{.chpule2}[ ]{.chpuri2}`Page` 階層は[、]{.chpule2}[
]{.chpuri2}**抽象化**[、]{.chpule2}[ ]{.chpuri2}`Renderer`
階層は[、]{.chpule2}[
]{.chpuri2}**実装**としての役割を果たします[。]{.chpule2}[
]{.chpuri2}`Page` のオブジェクトは[、]{.chpule2}[
]{.chpuri2}そのページに付随した `Renderer`
オブジェクトが用意する基本要素からある種のウェブ・ページを組み立てることができます[。]{.chpule2}[
]{.chpuri2}この二つのクラス階層は独立しているため[、]{.chpule2}[
]{.chpuri2}新しい `Renderer` のサブクラスを追加するために[、]{.chpule2}[
]{.chpuri2}どの `Page` サブクラスも変更する必要ありません[。]{.chpule2}[
]{.chpuri2}逆も同様です[。]{.chpule2}[ ]{.chpuri2}

#### []{#example-1--index-php .anchor} **index.php:** 現実的な例

<figure class="code">
<pre class="code" lang="php"><code>&lt;?php

namespace RefactoringGuru\Bridge\RealWorld;

/**
 * The Abstraction.
 */
abstract class Page
{
    /**
     * @var Renderer
     */
    protected $renderer;

    /**
     * The Abstraction is usually initialized with one of the Implementation
     * objects.
     */
    public function __construct(Renderer $renderer)
    {
        $this-&gt;renderer = $renderer;
    }

    /**
     * The Bridge pattern allows replacing the attached Implementation object
     * dynamically.
     */
    public function changeRenderer(Renderer $renderer): void
    {
        $this-&gt;renderer = $renderer;
    }

    /**
     * The &quot;view&quot; behavior stays abstract since it can only be provided by
     * Concrete Abstraction classes.
     */
    abstract public function view(): string;
}

/**
 * This Concrete Abstraction represents a simple page.
 */
class SimplePage extends Page
{
    protected $title;
    protected $content;

    public function __construct(Renderer $renderer, string $title, string $content)
    {
        parent::__construct($renderer);
        $this-&gt;title = $title;
        $this-&gt;content = $content;
    }

    public function view(): string
    {
        return $this-&gt;renderer-&gt;renderParts([
            $this-&gt;renderer-&gt;renderHeader(),
            $this-&gt;renderer-&gt;renderTitle($this-&gt;title),
            $this-&gt;renderer-&gt;renderTextBlock($this-&gt;content),
            $this-&gt;renderer-&gt;renderFooter()
        ]);
    }
}

/**
 * This Concrete Abstraction represents a more complex page.
 */
class ProductPage extends Page
{
    protected $product;

    public function __construct(Renderer $renderer, Product $product)
    {
        parent::__construct($renderer);
        $this-&gt;product = $product;
    }

    public function view(): string
    {
        return $this-&gt;renderer-&gt;renderParts([
            $this-&gt;renderer-&gt;renderHeader(),
            $this-&gt;renderer-&gt;renderTitle($this-&gt;product-&gt;getTitle()),
            $this-&gt;renderer-&gt;renderTextBlock($this-&gt;product-&gt;getDescription()),
            $this-&gt;renderer-&gt;renderImage($this-&gt;product-&gt;getImage()),
            $this-&gt;renderer-&gt;renderTextBlock(&#39;$&#39; . number_format($this-&gt;product-&gt;getPrice(), 2)),
            $this-&gt;renderer-&gt;renderLink(&quot;/cart/add/&quot; . $this-&gt;product-&gt;getId(), &quot;Add to cart&quot;),
            $this-&gt;renderer-&gt;renderFooter()
        ]);
    }
}

/**
 * A helper class for the ProductPage class.
 */
class Product
{
    private $id, $title, $description, $image, $price;

    public function __construct(
        string $id,
        string $title,
        string $description,
        string $image,
        float $price
    ) {
        $this-&gt;id = $id;
        $this-&gt;title = $title;
        $this-&gt;description = $description;
        $this-&gt;image = $image;
        $this-&gt;price = $price;
    }

    public function getId(): string
    {
        return $this-&gt;id;
    }

    public function getTitle(): string
    {
        return $this-&gt;title;
    }

    public function getDescription(): string
    {
        return $this-&gt;description;
    }

    public function getImage(): string
    {
        return $this-&gt;image;
    }

    public function getPrice(): float
    {
        return $this-&gt;price;
    }
}


/**
 * The Implementation declares a set of &quot;real&quot;, &quot;under-the-hood&quot;, &quot;platform&quot;
 * methods.
 *
 * In this case, the Implementation lists rendering methods that can be used to
 * compose any web page. Different Abstractions may use different methods of the
 * Implementation.
 */
interface Renderer
{
    public function renderTitle(string $title): string;

    public function renderTextBlock(string $text): string;

    public function renderImage(string $url): string;

    public function renderLink(string $url, string $title): string;

    public function renderHeader(): string;

    public function renderFooter(): string;

    public function renderParts(array $parts): string;
}

/**
 * This Concrete Implementation renders a web page as HTML.
 */
class HTMLRenderer implements Renderer
{
    public function renderTitle(string $title): string
    {
        return &quot;&lt;h1&gt;$title&lt;/h1&gt;&quot;;
    }

    public function renderTextBlock(string $text): string
    {
        return &quot;&lt;div class=&#39;text&#39;&gt;$text&lt;/div&gt;&quot;;
    }

    public function renderImage(string $url): string
    {
        return &quot;&lt;img src=&#39;$url&#39;&gt;&quot;;
    }

    public function renderLink(string $url, string $title): string
    {
        return &quot;&lt;a href=&#39;$url&#39;&gt;$title&lt;/a&gt;&quot;;
    }

    public function renderHeader(): string
    {
        return &quot;&lt;html&gt;&lt;body&gt;&quot;;
    }

    public function renderFooter(): string
    {
        return &quot;&lt;/body&gt;&lt;/html&gt;&quot;;
    }

    public function renderParts(array $parts): string
    {
        return implode(&quot;\n&quot;, $parts);
    }
}

/**
 * This Concrete Implementation renders a web page as JSON strings.
 */
class JsonRenderer implements Renderer
{
    public function renderTitle(string $title): string
    {
        return &#39;&quot;title&quot;: &quot;&#39; . $title . &#39;&quot;&#39;;
    }

    public function renderTextBlock(string $text): string
    {
        return &#39;&quot;text&quot;: &quot;&#39; . $text . &#39;&quot;&#39;;
    }

    public function renderImage(string $url): string
    {
        return &#39;&quot;img&quot;: &quot;&#39; . $url . &#39;&quot;&#39;;
    }

    public function renderLink(string $url, string $title): string
    {
        return &#39;&quot;link&quot;: {&quot;href&quot;: &quot;&#39; . $url . &#39;&quot;, &quot;title&quot;: &quot;&#39; . $title . &#39;&quot;}&#39;;
    }

    public function renderHeader(): string
    {
        return &#39;&#39;;
    }

    public function renderFooter(): string
    {
        return &#39;&#39;;
    }

    public function renderParts(array $parts): string
    {
        return &quot;{\n&quot; . implode(&quot;,\n&quot;, array_filter($parts)) . &quot;\n}&quot;;
    }
}

/**
 * The client code usually deals only with the Abstraction objects.
 */
function clientCode(Page $page)
{
    // ...

    echo $page-&gt;view();

    // ...
}

/**
 * The client code can be executed with any pre-configured combination of the
 * Abstraction+Implementation.
 */
$HTMLRenderer = new HTMLRenderer();
$JSONRenderer = new JsonRenderer();

$page = new SimplePage($HTMLRenderer, &quot;Home&quot;, &quot;Welcome to our website!&quot;);
echo &quot;HTML view of a simple content page:\n&quot;;
clientCode($page);
echo &quot;\n\n&quot;;

/**
 * The Abstraction can change the linked Implementation at runtime if needed.
 */
$page-&gt;changeRenderer($JSONRenderer);
echo &quot;JSON view of a simple content page, rendered with the same client code:\n&quot;;
clientCode($page);
echo &quot;\n\n&quot;;


$product = new Product(
    &quot;123&quot;,
    &quot;Star Wars, episode1&quot;,
    &quot;A long time ago in a galaxy far, far away...&quot;,
    &quot;/images/star-wars.jpeg&quot;,
    39.95
);

$page = new ProductPage($HTMLRenderer, $product);
echo &quot;HTML view of a product page, same client code:\n&quot;;
clientCode($page);
echo &quot;\n\n&quot;;

$page-&gt;changeRenderer($JSONRenderer);
echo &quot;JSON view of a simple content page, with the same client code:\n&quot;;
clientCode($page);</code></pre>
</figure>

#### []{#example-1--Output-txt .anchor} **Output.txt:** 実行結果

<figure class="code">
<pre class="code" lang="output"><code>HTML view of a simple content page:
&lt;html&gt;&lt;body&gt;
&lt;h1&gt;Home&lt;/h1&gt;
&lt;div class=&#39;text&#39;&gt;Welcome to our website!&lt;/div&gt;
&lt;/body&gt;&lt;/html&gt;

JSON view of a simple content page, rendered with the same client code:
{
&quot;title&quot;: &quot;Home&quot;,
&quot;text&quot;: &quot;Welcome to our website!&quot;
}

HTML view of a product page, same client code:
&lt;html&gt;&lt;body&gt;
&lt;h1&gt;Star Wars, episode1&lt;/h1&gt;
&lt;div class=&#39;text&#39;&gt;A long time ago in a galaxy far, far away...&lt;/div&gt;
&lt;img src=&#39;/images/star-wars.jpeg&#39;&gt;
&lt;a href=&#39;/cart/add/123&#39;&gt;Add to cart&lt;/a&gt;
&lt;/body&gt;&lt;/html&gt;

JSON view of a simple content page, with the same client code:
{
&quot;title&quot;: &quot;Star Wars, episode1&quot;,
&quot;text&quot;: &quot;A long time ago in a galaxy far, far away...&quot;,
&quot;img&quot;: &quot;/images/star-wars.jpeg&quot;,
&quot;link&quot;: {&quot;href&quot;: &quot;/cart/add/123&quot;, &quot;title&quot;: &quot;Add to cart&quot;}
}</code></pre>
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

[Composite を PHP で []{.fa
.fa-arrow-right}](../../composite/php/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### 戻る

[[]{.fa .fa-arrow-left} Adapter を PHP
で](../../adapter/php/example.html){.btn .btn-default rel="prev"}
:::

## 他言語での **Bridge**

[![Bridge を C#
で](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Bridge を C# で"){.prog-lang-link}
[![Bridge を C++
で](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Bridge を C++ で"){.prog-lang-link}
[![Bridge を Go
で](../../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Bridge を Go で"){.prog-lang-link}
[![Bridge を Java
で](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Bridge を Java で"){.prog-lang-link}
[![Bridge を Python
で](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Bridge を Python で"){.prog-lang-link}
[![Bridge を Ruby
で](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Bridge を Ruby で"){.prog-lang-link}
[![Bridge を Rust
で](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Bridge を Rust で"){.prog-lang-link}
[![Bridge を Swift
で](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Bridge を Swift で"){.prog-lang-link}
[![Bridge を TypeScript
で](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Bridge を TypeScript で"){.prog-lang-link}
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
