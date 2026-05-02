:::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} /
[デザインパターン](../../../design-patterns.html){.type} /
[Iterator](../../iterator.html){.type} / [Rust](../../rust.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Iterator](../../../../images/patterns/cards/iterator-minie2c2.png?id=76c28bb48f997b36965983dd2b41f02e){srcset="/images/patterns/cards/iterator-mini-2x.png?id=7d28f66a487066774a743cfceaa40ea4 2x"}
:::

# **Iterator** を Rust で {#iterator-を-rust-で .pattern-example-title-block-title}
::::

:::::::::::: pattern-example-body
::: pattern-example-brief
**Iterator** は[、]{.chpule2}[
]{.chpuri2}振る舞いに関するデザインパターンの一つで[、]{.chpule2}[
]{.chpuri2}複雑なデータ構造の内部の詳細を公開することなく[、]{.chpule2}[
]{.chpuri2}順次横断的に探索することを可能とします[。]{.chpule2}[
]{.chpuri2}

Iterator のおかげで[、]{.chpule2}[
]{.chpuri2}クライアントは[、]{.chpule2}[
]{.chpuri2}異なるコレクション上の要素の探索を[、]{.chpule2}[
]{.chpuri2}単一のイテレーター・インターフェースを使用して同様の方法で行えます[。]{.chpule2}[
]{.chpuri2}
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Iterator の詳細 ](../../iterator.html){.btn .btn-lg .btn-primary}
:::

::::::::: {.sidebar-navigation .with-scroll}
::: en-title
ナビゲーション
:::

::: en-intro
 [はじめに](#)
:::

::: en-intro
 [Standard Iterator](#example-0)
:::

::: en-intro
 [Custom Iterator](#example-1)
:::

::: en-file
 [users](#example-1--users-rs)
:::

::: en-file
 [main](#example-1--main-rs)
:::
:::::::::
::::::::::::

::: {#nav-tab .nav .nav-tabs .nav-justified role="tablist"}
[Standard Iterator](#example-0){#example-0-tab .nav-item .nav-link
.active toggle="tab" role="tab" aria-controls="example-0"
aria-selected="true"} [Custom Iterator](#example-1){#example-1-tab
.nav-item .nav-link toggle="tab" role="tab" aria-controls="example-1"
aria-selected="true"}
:::

::::: {#nav-tabContent .pattern-example-tab-content .tab-content}
::: {#example-0 .tab-pane .show .active role="tabpanel" aria-labelledby="example-0-tab" style="padding-top: 24px"}
## Standard Iterator {#example-0-title}

Iterators are heavily used in idiomatic Rust code. This is how to use
iterators over a standard array collection.

<figure class="code">
<pre class="code" lang="rust"><code>let array = &amp;[1, 2, 3];
let iterator = array.iter();

// Traversal over each element of the vector.
iterator.for_each(|e| print!(&quot;{}, &quot;, e));</code></pre>
</figure>
:::

::: {#example-1 .tab-pane role="tabpanel" aria-labelledby="example-1-tab" style="padding-top: 24px"}
## Custom Iterator {#example-1-title}

In Rust, the recommended way to define your *custom* iterator is to use
a standard `Iterator` trait. The example doesn\'t contain a synthetic
iterator interface, because it is really recommended to use the
idiomatic Rust way.

<figure class="code">
<pre class="code" lang="rust"><code>let users = UserCollection::new();
let mut iterator = users.iter();

iterator.next();</code></pre>
</figure>

A `next` method is the only `Iterator` trait method which is mandatory
to be implemented. It makes accessible a huge range of standard methods,
e.g. `fold`, `map`, `for_each`.

<figure class="code">
<pre class="code" lang="rust"><code>impl Iterator for UserIterator&lt;&#39;_&gt; {
    fn next(&amp;mut self) -&gt; Option&lt;Self::Item&gt;;
}</code></pre>
</figure>

#### []{#example-1--users-rs .anchor} **users.rs:** Collection & Iterator

<figure class="code">
<pre class="code" lang="rust"><code>pub struct UserCollection {
    users: [&amp;&#39;static str; 3],
}

/// A custom collection contains an arbitrary user array under the hood.
impl UserCollection {
    /// Returns a custom user collection.
    pub fn new() -&gt; Self {
        Self {
            users: [&quot;Alice&quot;, &quot;Bob&quot;, &quot;Carl&quot;],
        }
    }

    /// Returns an iterator over a user collection.
    ///
    /// The method name may be different, however, `iter` is used as a de facto
    /// standard in a Rust naming convention.
    pub fn iter(&amp;self) -&gt; UserIterator {
        UserIterator {
            index: 0,
            user_collection: self,
        }
    }
}

/// UserIterator allows sequential traversal through a complex user collection
/// without exposing its internal details.
pub struct UserIterator&lt;&#39;a&gt; {
    index: usize,
    user_collection: &amp;&#39;a UserCollection,
}

/// `Iterator` is a standard interface for dealing with iterators
/// from the Rust standard library.
impl Iterator for UserIterator&lt;&#39;_&gt; {
    type Item = &amp;&#39;static str;

    /// A `next` method is the only `Iterator` trait method which is mandatory to be
    /// implemented. It makes accessible a huge range of standard methods,
    /// e.g. `fold`, `map`, `for_each`.
    fn next(&amp;mut self) -&gt; Option&lt;Self::Item&gt; {
        if self.index &lt; self.user_collection.users.len() {
            let user = Some(self.user_collection.users[self.index]);
            self.index += 1;
            return user;
        }

        None
    }
}</code></pre>
</figure>

#### []{#example-1--main-rs .anchor} **main.rs:** Client code

<figure class="code">
<pre class="code" lang="rust"><code>use crate::users::UserCollection;

mod users;

fn main() {
    print!(&quot;Iterators are widely used in the standard library: &quot;);

    let array = &amp;[1, 2, 3];
    let iterator = array.iter();

    // Traversal over each element of the array.
    iterator.for_each(|e| print!(&quot;{} &quot;, e));

    println!(&quot;\n\nLet&#39;s test our own iterator.\n&quot;);

    let users = UserCollection::new();
    let mut iterator = users.iter();

    println!(&quot;1nd element: {:?}&quot;, iterator.next());
    println!(&quot;2nd element: {:?}&quot;, iterator.next());
    println!(&quot;3rd element: {:?}&quot;, iterator.next());
    println!(&quot;4th element: {:?}&quot;, iterator.next());

    print!(&quot;\nAll elements in user collection: &quot;);
    users.iter().for_each(|e| print!(&quot;{} &quot;, e));

    println!();
}</code></pre>
</figure>

### Output

<figure class="code">
<pre class="code"><code>Iterators are widely used in the standard library: 1 2 3

Let&#39;s test our own iterator.

1nd element: Some(&quot;Alice&quot;)
2nd element: Some(&quot;Bob&quot;)
3rd element: Some(&quot;Carl&quot;)
4th element: None


All elements in user collection: Alice Bob Carl</code></pre>
</figure>
:::
:::::

::: {#nav-tab .nav .nav-tabs .nav-tabs-bottom .nav-justified role="tablist"}
[Standard Iterator](#example-0){#example-0-tab-bottom .nav-item
.nav-link .active toggle="tab" role="tab" aria-controls="example-0"
aria-selected="true"} [Custom
Iterator](#example-1){#example-1-tab-bottom .nav-item .nav-link
toggle="tab" role="tab" aria-controls="example-1" aria-selected="true"}
:::

::: next
#### 次を読む

[Mediator を Rust で []{.fa
.fa-arrow-right}](../../mediator/rust/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### 戻る

[[]{.fa .fa-arrow-left} Command を Rust
で](../../command/rust/example.html){.btn .btn-default rel="prev"}
:::

## 他言語での **Iterator**

[![Iterator を C#
で](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Iterator を C# で"){.prog-lang-link}
[![Iterator を C++
で](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Iterator を C++ で"){.prog-lang-link}
[![Iterator を Go
で](../../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Iterator を Go で"){.prog-lang-link}
[![Iterator を Java
で](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Iterator を Java で"){.prog-lang-link}
[![Iterator を PHP
で](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Iterator を PHP で"){.prog-lang-link}
[![Iterator を Python
で](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Iterator を Python で"){.prog-lang-link}
[![Iterator を Ruby
で](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Iterator を Ruby で"){.prog-lang-link}
[![Iterator を Swift
で](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Iterator を Swift で"){.prog-lang-link}
[![Iterator を TypeScript
で](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Iterator を TypeScript で"){.prog-lang-link}
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
