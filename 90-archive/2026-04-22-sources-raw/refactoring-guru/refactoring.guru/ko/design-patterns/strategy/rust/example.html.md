:::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [디자인
패턴들](../../../design-patterns.html){.type} /
[전략](../../strategy.html){.type} / [러스트](../../rust.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![전략](../../../../images/patterns/cards/strategy-mini28f6.png?id=d38abee4fb6f2aed909d262bdadca936){srcset="/images/patterns/cards/strategy-mini-2x.png?id=f4e6608561f8e5d18be6927d4620ad29 2x"}
:::

# 러스트로 작성된 **전략** {#러스트로-작성된-전략 .pattern-example-title-block-title}
::::

:::::::::::: pattern-example-body
::: pattern-example-brief
**전략**은 행동들의 객체들을 객체들로 변환하며 이들이 원래 콘텍스트 객체
내에서 상호 교환이 가능하게 만드는 행동 디자인 패턴입니다.

원래 객체는 콘텍스트라고 불리며 전략 객체에 대한 참조를 포함합니다.
콘텍스트는 행동의 실행을 연결된 전략 객체에 위임합니다. 콘텍스트가
작업을 수행하는 방식을 변경하기 위하여 다른 객체들은 현재 연결된 전략
객체를 다른 전략 객체와 대체할 수 있습니다.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ 전략에 대하여 더 자세히 알아보세요 ](../../strategy.html){.btn .btn-lg
.btn-primary}
:::

::::::::: {.sidebar-navigation .with-scroll}
::: en-title
내비게이션
:::

::: en-intro
 [소개](#)
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
#### 다음을 보세요

[러스트로 작성된 템플릿 메서드 []{.fa
.fa-arrow-right}](../../template-method/rust/example.html){.btn
.btn-primary rel="next"}
:::

::: prev
#### 돌아가기

[[]{.fa .fa-arrow-left} 러스트로 작성된
상태](../../state/rust/example.html){.btn .btn-default rel="prev"}
:::

## 다른 언어로 작성된 **전략**

[![C#으로 작성된
전략](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "C#으로 작성된 전략"){.prog-lang-link}
[![C++로 작성된
전략](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "C++로 작성된 전략"){.prog-lang-link}
[![Go로 작성된
전략](../../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Go로 작성된 전략"){.prog-lang-link}
[![자바로 작성된
전략](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "자바로 작성된 전략"){.prog-lang-link}
[![PHP로 작성된
전략](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "PHP로 작성된 전략"){.prog-lang-link}
[![파이썬으로 작성된
전략](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "파이썬으로 작성된 전략"){.prog-lang-link}
[![루비로 작성된
전략](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "루비로 작성된 전략"){.prog-lang-link}
[![스위프트로 작성된
전략](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "스위프트로 작성된 전략"){.prog-lang-link}
[![타입스크립트로 작성된
전략](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "타입스크립트로 작성된 전략"){.prog-lang-link}
:::::::::::::::::::::::

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
:::::::::::::::::::::::::::::
::::::::::::::::::::::::::::::
