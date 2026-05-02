::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} /
[デザインパターン](../../../design-patterns.html){.type} /
[Singleton](../../singleton.html){.type} /
[Rust](../../rust.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Singleton](../../../../images/patterns/cards/singleton-minif559.png?id=914e1565dfdf15f240e766163bd303ec){srcset="/images/patterns/cards/singleton-mini-2x.png?id=4a7348083d7719d02709a5c9ff878b9f 2x"}
:::

# **Singleton** を Rust で {#singleton-を-rust-で .pattern-example-title-block-title}
::::

:::::::::::::: pattern-example-body
::: pattern-example-brief
**Singleton** は[、]{.chpule2}[
]{.chpuri2}生成に関するデザインパターンの一つで[、]{.chpule2}[
]{.chpuri2}この種類のオブジェクトがただ一つだけ存在することを保証し[、]{.chpule2}[
]{.chpuri2}他のコードに対して唯一のアクセス・ポイントを提供します[。]{.chpule2}[
]{.chpuri2}

Singleton には[、]{.chpule2}[
]{.chpuri2}大域変数とほぼ同じ長所と短所があります[。]{.chpule2}[
]{.chpuri2}両方とも随分と便利ですが[、]{.chpule2}[
]{.chpuri2}コードのモジュール性を犠牲にしています[。]{.chpule2}[
]{.chpuri2}

シングルトンのクラスに依存しているあるクラスを使う場合[、]{.chpule2}[
]{.chpuri2}シングルトンのクラスも一緒に使う必要があります[。]{.chpule2}[
]{.chpuri2}ほとんどの場合[、]{.chpule2}[
]{.chpuri2}この制限は[、]{.chpule2}[
]{.chpuri2}ユニット・テストの作成で問題となります[。]{.chpule2}[
]{.chpuri2}
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Singleton の詳細 ](../../singleton.html){.btn .btn-lg .btn-primary}
:::

::::::::::: {.sidebar-navigation .with-scroll}
::: en-title
ナビゲーション
:::

::: en-intro
 [はじめに](#)
:::

::: en-intro
 [Safe Singleton](#example-0)
:::

::: en-file
 [local](#example-0--local-rs)
:::

::: en-intro
 [Lazy Singleton](#example-1)
:::

::: en-file
 [lazy](#example-1--lazy-rs)
:::

::: en-intro
 [Singleton using *Mutex*](#example-2)
:::

::: en-file
 [mutex](#example-2--mutex-rs)
:::
:::::::::::

### Rust specifics

By definition, Singleton is a *global mutable object*. In Rust this is a
`static mut` item. Thus, to avoid all sorts of concurrency issues, the
function or block that is either reading or writing to a mutable static
variable should be marked as an [**`unsafe`**
block](https://doc.rust-lang.org/reference/items/static-items.html#mutable-statics).

For this reason, the Singleton pattern can be percieved as unsafe.
However, the pattern is still widely used in practice. A good read-world
example of Singleton is a `log` crate that introduces `log!`, `debug!`
and other logging macros, which you can use throughout your code after
setting up a concrete logger instance, such as
[env_logger](https://crates.io/crates/env_logger). As we can see,
`env_logger` uses
[log::set_boxed_logger](https://docs.rs/log/latest/log/fn.set_boxed_logger.html)
under the hood, which has an `unsafe` block to set up a global logger
object.

-   In order to provide safe and usable access to a singleton object,
    introduce an API hiding `unsafe` blocks under the hood.
-   See the thread about a mutable Singleton on
    [Stackoverflow](https://stackoverflow.com/questions/27791532/how-do-i-create-a-global-mutable-singleton)
    for more information.

Starting with Rust 1.63, `Mutex::new` is `const`, you can use global
static `Mutex` locks without needing lazy initialization. See the
*Singleton using Mutex* example below.
::::::::::::::

::: {#nav-tab .nav .nav-tabs .nav-justified role="tablist"}
[Safe Singleton](#example-0){#example-0-tab .nav-item .nav-link .active
toggle="tab" role="tab" aria-controls="example-0" aria-selected="true"}
[Lazy Singleton](#example-1){#example-1-tab .nav-item .nav-link
toggle="tab" role="tab" aria-controls="example-1" aria-selected="true"}
[Singleton using *Mutex*](#example-2){#example-2-tab .nav-item .nav-link
toggle="tab" role="tab" aria-controls="example-2" aria-selected="true"}
:::

:::::: {#nav-tabContent .pattern-example-tab-content .tab-content}
::: {#example-0 .tab-pane .show .active role="tabpanel" aria-labelledby="example-0-tab" style="padding-top: 24px"}
## Safe Singleton {#example-0-title}

A pure safe way to implement Singleton in Rust is using no global
variables at all and passing everything around through function
arguments. The oldest living variable is an object created at the start
of the `main()`.

#### []{#example-0--local-rs .anchor} **local.rs**

<figure class="code">
<pre class="code" lang="rust"><code>//! A pure safe way to implement Singleton in Rust is using no static variables
//! and passing everything around through function arguments.
//! The oldest living variable is an object created at the start of the `main()`.

fn change(global_state: &amp;mut u32) {
    *global_state += 1;
}

fn main() {
    let mut global_state = 0u32;

    change(&amp;mut global_state);

    println!(&quot;Final state: {}&quot;, global_state);
}</code></pre>
</figure>

### Output

<figure class="code">
<pre class="code"><code>Final state: 1</code></pre>
</figure>
:::

::: {#example-1 .tab-pane role="tabpanel" aria-labelledby="example-1-tab" style="padding-top: 24px"}
## Lazy Singleton {#example-1-title}

This is a singleton implementation via `lazy_static!`, which allows
declaring a static variable with lazy initialization at first access. It
is actually implemented via `unsafe` with `static mut` manipulation,
however, it keeps your code clear of `unsafe` blocks.

#### []{#example-1--lazy-rs .anchor} **lazy.rs**

<figure class="code">
<pre class="code" lang="rust"><code>//! Taken from: https://stackoverflow.com/questions/27791532/how-do-i-create-a-global-mutable-singleton
//!
//! Rust doesn&#39;t really allow a singleton pattern without `unsafe` because it
//! doesn&#39;t have a safe mutable global state.
//!
//! `lazy-static` allows declaring a static variable with lazy initialization
//! at first access. It is actually implemented via `unsafe` with `static mut`
//! manipulation, however, it keeps your code clear of `unsafe` blocks.
//!
//! `Mutex` provides safe access to a single object.

use lazy_static::lazy_static;
use std::sync::Mutex;

lazy_static! {
    static ref ARRAY: Mutex&lt;Vec&lt;u8&gt;&gt; = Mutex::new(vec![]);
}

fn do_a_call() {
    ARRAY.lock().unwrap().push(1);
}

fn main() {
    do_a_call();
    do_a_call();
    do_a_call();

    println!(&quot;Called {}&quot;, ARRAY.lock().unwrap().len());
}</code></pre>
</figure>

### Output

<figure class="code">
<pre class="code"><code>Called 3</code></pre>
</figure>
:::

::: {#example-2 .tab-pane role="tabpanel" aria-labelledby="example-2-tab" style="padding-top: 24px"}
## Singleton using *Mutex* {#example-2-title}

Starting with `Rust 1.63`, it can be easier to work with global mutable
singletons, although it\'s still preferable to avoid global variables in
mostcases.

Now that `Mutex::new` is `const`, you can use global static `Mutex`
locks without needing lazy initialization.

#### []{#example-2--mutex-rs .anchor} **mutex.rs**

<figure class="code">
<pre class="code" lang="rust"><code>//! ructc 1.63
//! https://stackoverflow.com/questions/27791532/how-do-i-create-a-global-mutable-singleton
//!
//! Starting with Rust 1.63, it can be easier to work with global mutable
//! singletons, although it&#39;s still preferable to avoid global variables in most
//! cases.
//!
//! Now that `Mutex::new` is `const`, you can use global static `Mutex` locks
//! without needing lazy initialization.

use std::sync::Mutex;

static ARRAY: Mutex&lt;Vec&lt;i32&gt;&gt; = Mutex::new(Vec::new());

fn do_a_call() {
    ARRAY.lock().unwrap().push(1);
}

fn main() {
    do_a_call();
    do_a_call();
    do_a_call();

    let array = ARRAY.lock().unwrap();
    println!(&quot;Called {} times: {:?}&quot;, array.len(), array);
    drop(array);

    *ARRAY.lock().unwrap() = vec![3, 4, 5];

    println!(&quot;New singleton object: {:?}&quot;, ARRAY.lock().unwrap());
}</code></pre>
</figure>

### Output

<figure class="code">
<pre class="code"><code>Called 3 times</code></pre>
</figure>
:::
::::::

::: {#nav-tab .nav .nav-tabs .nav-tabs-bottom .nav-justified role="tablist"}
[Safe Singleton](#example-0){#example-0-tab-bottom .nav-item .nav-link
.active toggle="tab" role="tab" aria-controls="example-0"
aria-selected="true"} [Lazy Singleton](#example-1){#example-1-tab-bottom
.nav-item .nav-link toggle="tab" role="tab" aria-controls="example-1"
aria-selected="true"} [Singleton using
*Mutex*](#example-2){#example-2-tab-bottom .nav-item .nav-link
toggle="tab" role="tab" aria-controls="example-2" aria-selected="true"}
:::

::: next
#### 次を読む

[Adapter を Rust で []{.fa
.fa-arrow-right}](../../adapter/rust/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### 戻る

[[]{.fa .fa-arrow-left} Prototype を Rust
で](../../prototype/rust/example.html){.btn .btn-default rel="prev"}
:::

## 他言語での **Singleton**

[![Singleton を C#
で](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Singleton を C# で"){.prog-lang-link}
[![Singleton を C++
で](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Singleton を C++ で"){.prog-lang-link}
[![Singleton を Go
で](../../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Singleton を Go で"){.prog-lang-link}
[![Singleton を Java
で](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Singleton を Java で"){.prog-lang-link}
[![Singleton を PHP
で](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Singleton を PHP で"){.prog-lang-link}
[![Singleton を Python
で](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Singleton を Python で"){.prog-lang-link}
[![Singleton を Ruby
で](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Singleton を Ruby で"){.prog-lang-link}
[![Singleton を Swift
で](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Singleton を Swift で"){.prog-lang-link}
[![Singleton を TypeScript
で](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Singleton を TypeScript で"){.prog-lang-link}
::::::::::::::::::::::::::

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
::::::::::::::::::::::::::::::::
:::::::::::::::::::::::::::::::::
