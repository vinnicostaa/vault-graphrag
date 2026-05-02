:::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} /
[デザインパターン](../../../design-patterns.html){.type} /
[Strategy](../../strategy.html){.type} / [Rust](../../rust.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Strategy](../../../../images/patterns/cards/strategy-mini28f6.png?id=d38abee4fb6f2aed909d262bdadca936){srcset="/images/patterns/cards/strategy-mini-2x.png?id=f4e6608561f8e5d18be6927d4620ad29 2x"}
:::

# **Strategy** を Rust で {#strategy-を-rust-で .pattern-example-title-block-title}
::::

:::::::::::: pattern-example-body
::: pattern-example-brief
**Strategy** は[、]{.chpule2}[
]{.chpuri2}振る舞いに関するデザインパターンの一つで[、]{.chpule2}[
]{.chpuri2}一連の振る舞いをオブジェクトに転換し[、]{.chpule2}[
]{.chpuri2}元のコンテキスト・オブジェクト内で交換可能とします[。]{.chpule2}[
]{.chpuri2}

元のオブジェクトは[、]{.chpule2}[
]{.chpuri2}コンテキストと呼ばれ[、]{.chpule2}[
]{.chpuri2}一つのストラテジー・オブジェクトへの参照を保持し[、]{.chpule2}[
]{.chpuri2}それに振る舞いの実行を委任します[。]{.chpule2}[
]{.chpuri2}コンテキストがその作業を実行する方法を変えるために[、]{.chpule2}[
]{.chpuri2}他のオブジェクトが[、]{.chpule2}[
]{.chpuri2}現在リンクされているオブジェクトを違うものと置き換えるかもしれません[。]{.chpule2}[
]{.chpuri2}
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Strategy の詳細 ](../../strategy.html){.btn .btn-lg .btn-primary}
:::

::::::::: {.sidebar-navigation .with-scroll}
::: en-title
ナビゲーション
:::

::: en-intro
 [はじめに](#)
:::

::: en-intro
 [Conceptual Example](#example-0)
:::

::: en-file
 [conceptual](#example-0--conceptual-rs)
:::

::: en-intro
 [Functional approach](#example-1)
:::

::: en-file
 [functional](#example-1--functional-rs)
:::
:::::::::
::::::::::::

::: {#nav-tab .nav .nav-tabs .nav-justified role="tablist"}
[Conceptual Example](#example-0){#example-0-tab .nav-item .nav-link
.active toggle="tab" role="tab" aria-controls="example-0"
aria-selected="true"} [Functional approach](#example-1){#example-1-tab
.nav-item .nav-link toggle="tab" role="tab" aria-controls="example-1"
aria-selected="true"}
:::

::::: {#nav-tabContent .pattern-example-tab-content .tab-content}
::: {#example-0 .tab-pane .show .active role="tabpanel" aria-labelledby="example-0-tab" style="padding-top: 24px"}
## Conceptual Example {#example-0-title}

A conceptual Strategy example via traits.

#### []{#example-0--conceptual-rs .anchor} **conceptual.rs**

<figure class="code">
<pre class="code" lang="rust"><code>/// Defines an injectable strategy for building routes.
trait RouteStrategy {
    fn build_route(&amp;self, from: &amp;str, to: &amp;str);
}

struct WalkingStrategy;

impl RouteStrategy for WalkingStrategy {
    fn build_route(&amp;self, from: &amp;str, to: &amp;str) {
        println!(&quot;Walking route from {} to {}: 4 km, 30 min&quot;, from, to);
    }
}

struct PublicTransportStrategy;

impl RouteStrategy for PublicTransportStrategy {
    fn build_route(&amp;self, from: &amp;str, to: &amp;str) {
        println!(
            &quot;Public transport route from {} to {}: 3 km, 5 min&quot;,
            from, to
        );
    }
}

struct Navigator&lt;T: RouteStrategy&gt; {
    route_strategy: T,
}

impl&lt;T: RouteStrategy&gt; Navigator&lt;T&gt; {
    pub fn new(route_strategy: T) -&gt; Self {
        Self { route_strategy }
    }

    pub fn route(&amp;self, from: &amp;str, to: &amp;str) {
        self.route_strategy.build_route(from, to);
    }
}

fn main() {
    let navigator = Navigator::new(WalkingStrategy);
    navigator.route(&quot;Home&quot;, &quot;Club&quot;);
    navigator.route(&quot;Club&quot;, &quot;Work&quot;);

    let navigator = Navigator::new(PublicTransportStrategy);
    navigator.route(&quot;Home&quot;, &quot;Club&quot;);
    navigator.route(&quot;Club&quot;, &quot;Work&quot;);
}</code></pre>
</figure>

### Output

<figure class="code">
<pre class="code"><code>Walking route from Home to Club: 4 km, 30 min
Walking route from Club to Work: 4 km, 30 min
Public transport route from Home to Club: 3 km, 5 min
Public transport route from Club to Work: 3 km, 5 min</code></pre>
</figure>
:::

::: {#example-1 .tab-pane role="tabpanel" aria-labelledby="example-1-tab" style="padding-top: 24px"}
## Functional approach {#example-1-title}

Functions and closures simplify Strategy implementation as you can
inject behavior right into the object without complex interface
definition.

It seems that Strategy is often implicitly and widely used in the modern
development with Rust, e.g. it\'s just like iterators work:

<figure class="code">
<pre class="code" lang="rust"><code>let a = [0i32, 1, 2];

let mut iter = a.iter().filter(|x| x.is_positive());</code></pre>
</figure>

#### []{#example-1--functional-rs .anchor} **functional.rs**

<figure class="code">
<pre class="code" lang="rust"><code>type RouteStrategy = fn(from: &amp;str, to: &amp;str);

fn walking_strategy(from: &amp;str, to: &amp;str) {
    println!(&quot;Walking route from {} to {}: 4 km, 30 min&quot;, from, to);
}

fn public_transport_strategy(from: &amp;str, to: &amp;str) {
    println!(
        &quot;Public transport route from {} to {}: 3 km, 5 min&quot;,
        from, to
    );
}

struct Navigator {
    route_strategy: RouteStrategy,
}

impl Navigator {
    pub fn new(route_strategy: RouteStrategy) -&gt; Self {
        Self { route_strategy }
    }

    pub fn route(&amp;self, from: &amp;str, to: &amp;str) {
        (self.route_strategy)(from, to);
    }
}

fn main() {
    let navigator = Navigator::new(walking_strategy);
    navigator.route(&quot;Home&quot;, &quot;Club&quot;);
    navigator.route(&quot;Club&quot;, &quot;Work&quot;);

    let navigator = Navigator::new(public_transport_strategy);
    navigator.route(&quot;Home&quot;, &quot;Club&quot;);
    navigator.route(&quot;Club&quot;, &quot;Work&quot;);

    let navigator = Navigator::new(|from, to| println!(&quot;Specific route from {} to {}&quot;, from, to));
    navigator.route(&quot;Home&quot;, &quot;Club&quot;);
    navigator.route(&quot;Club&quot;, &quot;Work&quot;);
}</code></pre>
</figure>

### Output

<figure class="code">
<pre class="code"><code>Walking route from Home to Club: 4 km, 30 min
Walking route from Club to Work: 4 km, 30 min
Public transport route from Home to Club: 3 km, 5 min
Public transport route from Club to Work: 3 km, 5 min
Specific route from Home to Club
Specific route from Club to Work</code></pre>
</figure>
:::
:::::

::: {#nav-tab .nav .nav-tabs .nav-tabs-bottom .nav-justified role="tablist"}
[Conceptual Example](#example-0){#example-0-tab-bottom .nav-item
.nav-link .active toggle="tab" role="tab" aria-controls="example-0"
aria-selected="true"} [Functional
approach](#example-1){#example-1-tab-bottom .nav-item .nav-link
toggle="tab" role="tab" aria-controls="example-1" aria-selected="true"}
:::

::: next
#### 次を読む

[Template Method を Rust で []{.fa
.fa-arrow-right}](../../template-method/rust/example.html){.btn
.btn-primary rel="next"}
:::

::: prev
#### 戻る

[[]{.fa .fa-arrow-left} State を Rust
で](../../state/rust/example.html){.btn .btn-default rel="prev"}
:::

## 他言語での **Strategy**

[![Strategy を C#
で](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Strategy を C# で"){.prog-lang-link}
[![Strategy を C++
で](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Strategy を C++ で"){.prog-lang-link}
[![Strategy を Go
で](../../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Strategy を Go で"){.prog-lang-link}
[![Strategy を Java
で](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Strategy を Java で"){.prog-lang-link}
[![Strategy を PHP
で](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Strategy を PHP で"){.prog-lang-link}
[![Strategy を Python
で](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Strategy を Python で"){.prog-lang-link}
[![Strategy を Ruby
で](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Strategy を Ruby で"){.prog-lang-link}
[![Strategy を Swift
で](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Strategy を Swift で"){.prog-lang-link}
[![Strategy を TypeScript
で](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Strategy を TypeScript で"){.prog-lang-link}
:::::::::::::::::::::::

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
:::::::::::::::::::::::::::::
::::::::::::::::::::::::::::::
