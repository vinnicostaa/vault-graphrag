::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} /
[デザインパターン](../../../design-patterns.html){.type} /
[Prototype](../../prototype.html){.type} /
[Rust](../../rust.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Prototype](../../../../images/patterns/cards/prototype-mini6540.png?id=bc3046bb39ff36574c08d49839fd1c8e){srcset="/images/patterns/cards/prototype-mini-2x.png?id=b871f789a736e7efbb1cd082d2de6398 2x"}
:::

# **Prototype** を Rust で {#prototype-を-rust-で .pattern-example-title-block-title}
::::

:::::::::: pattern-example-body
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

::::::: {.sidebar-navigation .with-scroll}
::: en-title
ナビゲーション
:::

::: en-intro
 [はじめに](#)
:::

::: en-intro
 [Built-in *Clone* trait](#example-0)
:::

::: en-file
 [main](#example-0--main-rs)
:::
:::::::
::::::::::

[]{#example-0}

## Built-in *Clone* trait {#example-0-title}

Rust has a built-in `std::clone::Clone` trait with many implementations
for various types (via `#[derive(Clone)]`). Thus, the Prototype pattern
is ready to use out of the box.

#### []{#example-0--main-rs .anchor} **main.rs**

<figure class="code">
<pre class="code" lang="rust"><code>#[derive(Clone)]
struct Circle {
    pub x: u32,
    pub y: u32,
    pub radius: u32,
}

fn main() {
    let circle1 = Circle {
        x: 10,
        y: 15,
        radius: 10,
    };

    // Prototype in action.
    let mut circle2 = circle1.clone();
    circle2.radius = 77;

    println!(&quot;Circle 1: {}, {}, {}&quot;, circle1.x, circle1.y, circle1.radius);
    println!(&quot;Circle 2: {}, {}, {}&quot;, circle2.x, circle2.y, circle2.radius);
}</code></pre>
</figure>

### Output

<figure class="code">
<pre class="code"><code>Circle 1: 10, 15, 10
Circle 2: 10, 15, 77</code></pre>
</figure>

::: next
#### 次を読む

[Singleton を Rust で []{.fa
.fa-arrow-right}](../../singleton/rust/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### 戻る

[[]{.fa .fa-arrow-left} Factory Method を Rust
で](../../factory-method/rust/example.html){.btn .btn-default
rel="prev"}
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
[![Prototype を PHP
で](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Prototype を PHP で"){.prog-lang-link}
[![Prototype を Python
で](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Prototype を Python で"){.prog-lang-link}
[![Prototype を Ruby
で](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Prototype を Ruby で"){.prog-lang-link}
[![Prototype を Swift
で](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Prototype を Swift で"){.prog-lang-link}
[![Prototype を TypeScript
で](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Prototype を TypeScript で"){.prog-lang-link}
::::::::::::::::

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
::::::::::::::::::::::
:::::::::::::::::::::::
