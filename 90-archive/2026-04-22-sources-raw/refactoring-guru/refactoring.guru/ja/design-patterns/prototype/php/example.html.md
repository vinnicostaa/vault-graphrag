:::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} /
[デザインパターン](../../../design-patterns.html){.type} /
[Prototype](../../prototype.html){.type} / [PHP](../../php.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Prototype](../../../../images/patterns/cards/prototype-mini6540.png?id=bc3046bb39ff36574c08d49839fd1c8e){srcset="/images/patterns/cards/prototype-mini-2x.png?id=b871f789a736e7efbb1cd082d2de6398 2x"}
:::

# **Prototype** を PHP で {#prototype-を-php-で .pattern-example-title-block-title}
::::

:::::::::::::: pattern-example-body
::: pattern-example-brief
**Prototype** は[、]{.chpule2}[
]{.chpuri2}生成に関するデザインパターンの一つで[、]{.chpule2}[
]{.chpuri2}特定のクラスに結合することなく[、]{.chpule2}[
]{.chpuri2}オブジェクト[
]{.chpule1}[（]{.chpuri1}たとえ複雑なオブジェクトでも[）]{.chpule2}[
]{.chpuri2}のクローン作成を可能とします[。]{.chpule2}[ ]{.chpuri2}

プロトタイプのクラス全部には[、]{.chpule2}[
]{.chpuri2}共通するインターフェースが必要です[。]{.chpule2}[
]{.chpuri2}これにより[、]{.chpule2}[
]{.chpuri2}具象クラスが不明であってもオブジェクトを複製することが可能となります[。]{.chpule2}[
]{.chpuri2}プロトタイプ・オブジェクトが[、]{.chpule2}[
]{.chpuri2}完全なコピーを生成できるのは[、]{.chpule2}[
]{.chpuri2}同じクラスのオブジェクト同士が非公開フィールドを互いにアクセスできるからです[。]{.chpule2}[
]{.chpuri2}
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Prototype の詳細 ](../../prototype.html){.btn .btn-lg .btn-primary}
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

**使用例[：]{.chpule2}[ ]{.chpuri2}** Prototype
パターンは[、]{.chpule2}[ ]{.chpuri2}PHP では[、]{.chpule2}[
]{.chpuri2}[初めから利用可能](http://php.net/manual/en/language.oop5.cloning.php)です[。]{.chpule2}[
]{.chpuri2}オブジェクトの完全なコピーは[、]{.chpule2}[
]{.chpuri2}`clone` キーワードを使って取得できます[。]{.chpule2}[
]{.chpuri2}クラスにクローンのサポートを追加するには[、]{.chpule2}[
]{.chpuri2}`__clone` メソッドを実装する必要があります[。]{.chpule2}[
]{.chpuri2}

**見つけ方[：]{.chpule2}[ ]{.chpuri2}**このパターンは[、]{.chpule2}[
]{.chpuri2}`clone` や `copy`
といったメソッドで容易に識別可能です[。]{.chpule2}[ ]{.chpuri2}
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

この例は[、]{.chpule2}[ ]{.chpuri2}**Prototype**
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

namespace RefactoringGuru\Prototype\Conceptual;

/**
 * The example class that has cloning ability. We&#39;ll see how the values of field
 * with different types will be cloned.
 */
class Prototype
{
    public $primitive;
    public $component;
    public $circularReference;

    /**
     * PHP has built-in cloning support. You can `clone` an object without
     * defining any special methods as long as it has fields of primitive types.
     * Fields containing objects retain their references in a cloned object.
     * Therefore, in some cases, you might want to clone those referenced
     * objects as well. You can do this in a special `__clone()` method.
     */
    public function __clone()
    {
        $this-&gt;component = clone $this-&gt;component;

        // Cloning an object that has a nested object with backreference
        // requires special treatment. After the cloning is completed, the
        // nested object should point to the cloned object, instead of the
        // original object.
        $this-&gt;circularReference = clone $this-&gt;circularReference;
        $this-&gt;circularReference-&gt;prototype = $this;
    }
}

class ComponentWithBackReference
{
    public $prototype;

    /**
     * Note that the constructor won&#39;t be executed during cloning. If you have
     * complex logic inside the constructor, you may need to execute it in the
     * `__clone` method as well.
     */
    public function __construct(Prototype $prototype)
    {
        $this-&gt;prototype = $prototype;
    }
}

/**
 * The client code.
 */
function clientCode()
{
    $p1 = new Prototype();
    $p1-&gt;primitive = 245;
    $p1-&gt;component = new \DateTime();
    $p1-&gt;circularReference = new ComponentWithBackReference($p1);

    $p2 = clone $p1;
    if ($p1-&gt;primitive === $p2-&gt;primitive) {
        echo &quot;Primitive field values have been carried over to a clone. Yay!\n&quot;;
    } else {
        echo &quot;Primitive field values have not been copied. Booo!\n&quot;;
    }
    if ($p1-&gt;component === $p2-&gt;component) {
        echo &quot;Simple component has not been cloned. Booo!\n&quot;;
    } else {
        echo &quot;Simple component has been cloned. Yay!\n&quot;;
    }

    if ($p1-&gt;circularReference === $p2-&gt;circularReference) {
        echo &quot;Component with back reference has not been cloned. Booo!\n&quot;;
    } else {
        echo &quot;Component with back reference has been cloned. Yay!\n&quot;;
    }

    if ($p1-&gt;circularReference-&gt;prototype === $p2-&gt;circularReference-&gt;prototype) {
        echo &quot;Component with back reference is linked to original object. Booo!\n&quot;;
    } else {
        echo &quot;Component with back reference is linked to the clone. Yay!\n&quot;;
    }
}

clientCode();</code></pre>
</figure>

#### []{#example-0--Output-txt .anchor} **Output.txt:** 実行結果

<figure class="code">
<pre class="code" lang="output"><code>Primitive field values have been carried over to a clone. Yay!
Simple component has been cloned. Yay!
Component with back reference has been cloned. Yay!
Component with back reference is linked to the clone. Yay!</code></pre>
</figure>
:::

::: {#example-1 .tab-pane role="tabpanel" aria-labelledby="example-1-tab" style="padding-top: 24px"}
## 現実的な例 {#example-1-title}

**Prototype** パターンは[、]{.chpule2}[
]{.chpuri2}すべてのフィールドを直接コピーしてオブジェクトを再構築する代わりに[、]{.chpule2}[
]{.chpuri2}既存のオブジェクトを複製する便利な方法を提供します[。]{.chpule2}[
]{.chpuri2}直接的なやり方は[、]{.chpule2}[
]{.chpuri2}クローンされるオブジェクトのクラスに密に結合してしまうばかりか[、]{.chpule2}[
]{.chpuri2}非公開フィールドの内容はコピーできないという問題があります[。]{.chpule2}[
]{.chpuri2} Prototype では[、]{.chpule2}[
]{.chpuri2}実際のクローンの作業は[、]{.chpule2}[
]{.chpuri2}クローンされるクラス中で行われるため[、]{.chpule2}[
]{.chpuri2}非公開フィールドへのアクセスは無制限です[。]{.chpule2}[
]{.chpuri2}

この例では[、]{.chpule2}[ ]{.chpuri2}複雑な Page
オブジェクトのクローン作成に Prototype
パターンを使用しています[。]{.chpule2}[ ]{.chpuri2}Page
クラスには多数の非公開フィールドがありますが[、]{.chpule2}[
]{.chpuri2}Prototype パターンのおかげで[、]{.chpule2}[
]{.chpuri2}クローンされたオブジェクトにうまく引き継がれます[。]{.chpule2}[
]{.chpuri2}

#### []{#example-1--index-php .anchor} **index.php:** 現実的な例

<figure class="code">
<pre class="code" lang="php"><code>&lt;?php

namespace RefactoringGuru\Prototype\RealWorld;

/**
 * Prototype.
 */
class Page
{
    private $title;

    private $body;

    /**
     * @var Author
     */
    private $author;

    private $comments = [];

    /**
     * @var \DateTime
     */
    private $date;

    // +100 private fields.

    public function __construct(string $title, string $body, Author $author)
    {
        $this-&gt;title = $title;
        $this-&gt;body = $body;
        $this-&gt;author = $author;
        $this-&gt;author-&gt;addToPage($this);
        $this-&gt;date = new \DateTime();
    }

    public function addComment(string $comment): void
    {
        $this-&gt;comments[] = $comment;
    }

    /**
     * You can control what data you want to carry over to the cloned object.
     *
     * For instance, when a page is cloned:
     * - It gets a new &quot;Copy of ...&quot; title.
     * - The author of the page remains the same. Therefore we leave the
     * reference to the existing object while adding the cloned page to the list
     * of the author&#39;s pages.
     * - We don&#39;t carry over the comments from the old page.
     * - We also attach a new date object to the page.
     */
    public function __clone()
    {
        $this-&gt;title = &quot;Copy of &quot; . $this-&gt;title;
        $this-&gt;author-&gt;addToPage($this);
        $this-&gt;comments = [];
        $this-&gt;date = new \DateTime();
    }
}

class Author
{
    private $name;

    /**
     * @var Page[]
     */
    private $pages = [];

    public function __construct(string $name)
    {
        $this-&gt;name = $name;
    }

    public function addToPage(Page $page): void
    {
        $this-&gt;pages[] = $page;
    }
}

/**
 * The client code.
 */
function clientCode()
{
    $author = new Author(&quot;John Smith&quot;);
    $page = new Page(&quot;Tip of the day&quot;, &quot;Keep calm and carry on.&quot;, $author);

    // ...

    $page-&gt;addComment(&quot;Nice tip, thanks!&quot;);

    // ...

    $draft = clone $page;
    echo &quot;Dump of the clone. Note that the author is now referencing two objects.\n\n&quot;;
    print_r($draft);
}

clientCode();</code></pre>
</figure>

#### []{#example-1--Output-txt .anchor} **Output.txt:** 実行結果

<figure class="code">
<pre class="code" lang="output"><code>Dump of the clone. Note that the author is now referencing two objects.

RefactoringGuru\Prototype\RealWorld\Page Object
(
    [title:RefactoringGuru\Prototype\RealWorld\Page:private] =&gt; Copy of Tip of the day
    [body:RefactoringGuru\Prototype\RealWorld\Page:private] =&gt; Keep calm and carry on.
    [author:RefactoringGuru\Prototype\RealWorld\Page:private] =&gt; RefactoringGuru\Prototype\RealWorld\Author Object
        (
            [name:RefactoringGuru\Prototype\RealWorld\Author:private] =&gt; John Smith
            [pages:RefactoringGuru\Prototype\RealWorld\Author:private] =&gt; Array
                (
                    [0] =&gt; RefactoringGuru\Prototype\RealWorld\Page Object
                        (
                            [title:RefactoringGuru\Prototype\RealWorld\Page:private] =&gt; Tip of the day
                            [body:RefactoringGuru\Prototype\RealWorld\Page:private] =&gt; Keep calm and carry on.
                            [author:RefactoringGuru\Prototype\RealWorld\Page:private] =&gt; RefactoringGuru\Prototype\RealWorld\Author Object
 *RECURSION*
                            [comments:RefactoringGuru\Prototype\RealWorld\Page:private] =&gt; Array
                                (
                                    [0] =&gt; Nice tip, thanks!
                                )

                            [date:RefactoringGuru\Prototype\RealWorld\Page:private] =&gt; DateTime Object
                                (
                                    [date] =&gt; 2018-06-04 14:50:39.306237
                                    [timezone_type] =&gt; 3
                                    [timezone] =&gt; UTC
                                )

                        )

                    [1] =&gt; RefactoringGuru\Prototype\RealWorld\Page Object
 *RECURSION*
                )

        )

    [comments:RefactoringGuru\Prototype\RealWorld\Page:private] =&gt; Array
        (
        )

    [date:RefactoringGuru\Prototype\RealWorld\Page:private] =&gt; DateTime Object
        (
            [date] =&gt; 2018-06-04 14:50:39.306272
            [timezone_type] =&gt; 3
            [timezone] =&gt; UTC
        )

)</code></pre>
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

[Singleton を PHP で []{.fa
.fa-arrow-right}](../../singleton/php/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### 戻る

[[]{.fa .fa-arrow-left} Factory Method を PHP
で](../../factory-method/php/example.html){.btn .btn-default rel="prev"}
:::

## 他言語での **Prototype**

[![Prototype を C#
で](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Prototype を C# で"){.prog-lang-link}
[![Prototype を C++
で](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Prototype を C++ で"){.prog-lang-link}
[![Prototype を Go
で](../../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Prototype を Go で"){.prog-lang-link}
[![Prototype を Java
で](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Prototype を Java で"){.prog-lang-link}
[![Prototype を Python
で](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Prototype を Python で"){.prog-lang-link}
[![Prototype を Ruby
で](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Prototype を Ruby で"){.prog-lang-link}
[![Prototype を Rust
で](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Prototype を Rust で"){.prog-lang-link}
[![Prototype を Swift
で](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Prototype を Swift で"){.prog-lang-link}
[![Prototype を TypeScript
で](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Prototype を TypeScript で"){.prog-lang-link}
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
