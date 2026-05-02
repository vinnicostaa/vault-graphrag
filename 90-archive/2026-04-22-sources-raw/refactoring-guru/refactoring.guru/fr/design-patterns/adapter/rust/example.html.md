:::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [Patrons de
conception](../../../design-patterns.html){.type} /
[Adaptateur](../../adapter.html){.type} / [Rust](../../rust.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Adaptateur](../../../../images/patterns/cards/adapter-minid866.png?id=b2ee4f681fb589be5a0685b94692aebb){srcset="/images/patterns/cards/adapter-mini-2x.png?id=8274d99afbbe9c63bfbfd0d68ceeffc7 2x"}
:::

# **Adaptateur** en Rust {#adaptateur-en-rust .pattern-example-title-block-title}
::::

::::::::::::: pattern-example-body
::: pattern-example-brief
L'**Adaptateur** est un patron de conception structurel qui permet à des
objets incompatibles de collaborer.

L'adaptateur fait office d'emballeur entre les deux objets. Il récupère
les appels à un objet et les met dans un format et une interface
reconnaissables par le second objet.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ En savoir plus sur la patron Adaptateur ](../../adapter.html){.btn
.btn-lg .btn-primary}
:::

:::::::::: {.sidebar-navigation .with-scroll}
::: en-title
Navigation
:::

::: en-intro
 [Intro](#)
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
#### Suivant

[Pont en Rust []{.fa
.fa-arrow-right}](../../bridge/rust/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Retour

[[]{.fa .fa-arrow-left} Singleton en
Rust](../../singleton/rust/example.html){.btn .btn-default rel="prev"}
:::

## **Adaptateur** dans les autres langues

[![Adaptateur en
C#](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Adaptateur en C#"){.prog-lang-link}
[![Adaptateur en
C++](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Adaptateur en C++"){.prog-lang-link}
[![Adaptateur en
Go](../../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Adaptateur en Go"){.prog-lang-link}
[![Adaptateur en
Java](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Adaptateur en Java"){.prog-lang-link}
[![Adaptateur en
PHP](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Adaptateur en PHP"){.prog-lang-link}
[![Adaptateur en
Python](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Adaptateur en Python"){.prog-lang-link}
[![Adaptateur en
Ruby](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Adaptateur en Ruby"){.prog-lang-link}
[![Adaptateur en
Swift](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Adaptateur en Swift"){.prog-lang-link}
[![Adaptateur en
TypeScript](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Adaptateur en TypeScript"){.prog-lang-link}
:::::::::::::::::::

::::::: {.prom .banner-sidebar .banner .banner-striped .banner-examples .banner-removable .banner-removable-patterns data-id="DP: Examples-IDE" creative-id="examples-sidebar-fr" position="sidebar"}
:::::: {.banner-inner style="font-size: 14px; line-height: 21px;"}
::: banner-examples-img
[![](../../../../images/patterns/banners/examples-ide91ca.png?id=3115b4b548fb96b75974e2de8f4f49bc){width="215"
height="158" loading="lazy"
srcset="/images/patterns/banners/examples-ide-2x.png?id=93c007a6d157b97c28eb213f59d716ae 2x"}](../../book.html)
:::

::: banner-examples-fade
[ Archive avec exemples](../../book.html){.btn .btn-secondary
.btn-block}
:::

::: {.banner-examples-text style="margin-top: 1rem;"}
Achetez l'ebook **Plongée au cœur des patrons de conception** et obtenez
l'accès à une archive comprenant des dizaines d'exemples détaillés qui
peuvent être ouverts dans votre environnement de développement.

[ En savoir plus...](../../book.html){.btn .btn-secondary .btn-block}
:::
::::::
:::::::
:::::::::::::::::::::::::
::::::::::::::::::::::::::
