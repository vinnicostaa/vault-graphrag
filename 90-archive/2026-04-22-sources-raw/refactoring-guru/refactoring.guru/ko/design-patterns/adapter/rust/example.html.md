:::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [디자인
패턴들](../../../design-patterns.html){.type} /
[어댑터](../../adapter.html){.type} / [러스트](../../rust.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![어댑터](../../../../images/patterns/cards/adapter-minid866.png?id=b2ee4f681fb589be5a0685b94692aebb){srcset="/images/patterns/cards/adapter-mini-2x.png?id=8274d99afbbe9c63bfbfd0d68ceeffc7 2x"}
:::

# 러스트로 작성된 **어댑터** {#러스트로-작성된-어댑터 .pattern-example-title-block-title}
::::

::::::::::::: pattern-example-body
::: pattern-example-brief
**어댑터**는 구조 디자인 패턴이며, 호환되지 않는 객체들이 협업할 수
있도록 합니다.

어댑터는 두 객체 사이의 래퍼 역할을 합니다. 하나의 객체에 대한 호출을
캐치하고 두 번째 객체가 인식할 수 있는 형식과 인터페이스로 변환합니다.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ 어댑터에 대하여 더 자세히 알아보세요 ](../../adapter.html){.btn
.btn-lg .btn-primary}
:::

:::::::::: {.sidebar-navigation .with-scroll}
::: en-title
내비게이션
:::

::: en-intro
 [소개](#)
:::

::: en-intro
 [Adapter in Rust](#example-0)
:::

::: en-file
 [adapter](#example-0--adapter-rs)
:::

::: en-file
 [adaptee](#example-0--adaptee-rs)
:::

::: en-file
 [target](#example-0--target-rs)
:::

::: en-file
 [main](#example-0--main-rs)
:::
::::::::::
:::::::::::::

[]{#example-0}

## Adapter in Rust {#example-0-title}

In this example, the `trait SpecificTarget` is incompatible with a
`call` function which accepts `trait Target` only.

<figure class="code">
<pre class="code" lang="rust"><code>fn call(target: impl Target);</code></pre>
</figure>

The adapter helps to pass the incompatible interface to the `call`
function.

<figure class="code">
<pre class="code" lang="rust"><code>let target = TargetAdapter::new(specific_target);
call(target);</code></pre>
</figure>

#### []{#example-0--adapter-rs .anchor} **adapter.rs**

<figure class="code">
<pre class="code" lang="rust"><code>use crate::{adaptee::SpecificTarget, Target};

/// Converts adaptee&#39;s specific interface to a compatible `Target` output.
pub struct TargetAdapter {
    adaptee: SpecificTarget,
}

impl TargetAdapter {
    pub fn new(adaptee: SpecificTarget) -&gt; Self {
        Self { adaptee }
    }
}

impl Target for TargetAdapter {
    fn request(&amp;self) -&gt; String {
        // Here&#39;s the &quot;adaptation&quot; of a specific output to a compatible output.
        self.adaptee.specific_request().chars().rev().collect()
    }
}</code></pre>
</figure>

#### []{#example-0--adaptee-rs .anchor} **adaptee.rs**

<figure class="code">
<pre class="code" lang="rust"><code>pub struct SpecificTarget;

impl SpecificTarget {
    pub fn specific_request(&amp;self) -&gt; String {
        &quot;.tseuqer cificepS&quot;.into()
    }
}</code></pre>
</figure>

#### []{#example-0--target-rs .anchor} **target.rs**

<figure class="code">
<pre class="code" lang="rust"><code>pub trait Target {
    fn request(&amp;self) -&gt; String;
}

pub struct OrdinaryTarget;

impl Target for OrdinaryTarget {
    fn request(&amp;self) -&gt; String {
        &quot;Ordinary request.&quot;.into()
    }
}</code></pre>
</figure>

#### []{#example-0--main-rs .anchor} **main.rs**

<figure class="code">
<pre class="code" lang="rust"><code>mod adaptee;
mod adapter;
mod target;

use adaptee::SpecificTarget;
use adapter::TargetAdapter;
use target::{OrdinaryTarget, Target};

/// Calls any object of a `Target` trait.
///
/// To understand the Adapter pattern better, imagine that this is
/// a client code, which can operate over a specific interface only
/// (`Target` trait only). It means that an incompatible interface cannot be
/// passed here without an adapter.
fn call(target: impl Target) {
    println!(&quot;&#39;{}&#39;&quot;, target.request());
}

fn main() {
    let target = OrdinaryTarget;

    print!(&quot;A compatible target can be directly called: &quot;);
    call(target);

    let adaptee = SpecificTarget;

    println!(
        &quot;Adaptee is incompatible with client: &#39;{}&#39;&quot;,
        adaptee.specific_request()
    );

    let adapter = TargetAdapter::new(adaptee);

    print!(&quot;But with adapter client can call its method: &quot;);
    call(adapter);
}</code></pre>
</figure>

### Output

<figure class="code">
<pre class="code"><code>A compatible target can be directly called: &#39;Ordinary request.&#39;
Adaptee is incompatible with client: &#39;.tseuqer cificepS&#39;
But with adapter client can call its method: &#39;Specific request.&#39;</code></pre>
</figure>

::: next
#### 다음을 보세요

[러스트로 작성된 브리지 []{.fa
.fa-arrow-right}](../../bridge/rust/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### 돌아가기

[[]{.fa .fa-arrow-left} 러스트로 작성된
싱글턴](../../singleton/rust/example.html){.btn .btn-default rel="prev"}
:::

## 다른 언어로 작성된 **어댑터**

[![C#으로 작성된
어댑터](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "C#으로 작성된 어댑터"){.prog-lang-link}
[![C++로 작성된
어댑터](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "C++로 작성된 어댑터"){.prog-lang-link}
[![Go로 작성된
어댑터](../../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Go로 작성된 어댑터"){.prog-lang-link}
[![자바로 작성된
어댑터](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "자바로 작성된 어댑터"){.prog-lang-link}
[![PHP로 작성된
어댑터](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "PHP로 작성된 어댑터"){.prog-lang-link}
[![파이썬으로 작성된
어댑터](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "파이썬으로 작성된 어댑터"){.prog-lang-link}
[![루비로 작성된
어댑터](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "루비로 작성된 어댑터"){.prog-lang-link}
[![스위프트로 작성된
어댑터](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "스위프트로 작성된 어댑터"){.prog-lang-link}
[![타입스크립트로 작성된
어댑터](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "타입스크립트로 작성된 어댑터"){.prog-lang-link}
:::::::::::::::::::

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
:::::::::::::::::::::::::
::::::::::::::::::::::::::
