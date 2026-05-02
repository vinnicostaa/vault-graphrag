::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} /
[デザインパターン](../../../design-patterns.html){.type} /
[Composite](../../composite.html){.type} /
[Rust](../../rust.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Composite](../../../../images/patterns/cards/composite-minib543.png?id=a369d98d18b417f255d04568fd0131b8){srcset="/images/patterns/cards/composite-mini-2x.png?id=3f7f811fefeb0b64f6774746eb42af09 2x"}
:::

# **Composite** を Rust で {#composite-を-rust-で .pattern-example-title-block-title}
::::

:::::::::::::::: pattern-example-body
::: pattern-example-brief
**Composite** は[、]{.chpule2}[
]{.chpuri2}構造に関するデザインパターンの一つで[、]{.chpule2}[
]{.chpuri2}オブジェクトを木のような構造に構成し[、]{.chpule2}[
]{.chpuri2}あたかも単一のオブジェクトであるかのように扱えるようにします[。]{.chpule2}[
]{.chpuri2}

Composite は[、]{.chpule2}[
]{.chpuri2}ツリー構造の構築を必要とする問題の大部分の解決策として[、]{.chpule2}[
]{.chpuri2}かなりの人気を得るようになりました[。]{.chpule2}[
]{.chpuri2}Composite の大きな特徴は[、]{.chpule2}[
]{.chpuri2}ツリー構造全体でメソッドを再帰的に実行し[、]{.chpule2}[
]{.chpuri2}結果をまとめあげることです[。]{.chpule2}[ ]{.chpuri2}
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Composite の詳細 ](../../composite.html){.btn .btn-lg .btn-primary}
:::

::::::::::::: {.sidebar-navigation .with-scroll}
::: en-title
ナビゲーション
:::

::: en-intro
 [はじめに](#)
:::

::: en-intro
 [Files and Folders](#example-0)
:::

:::: en-inside
::: en-file
  [mod](#example-0--fs-mod-rs)
:::
::::

:::: en-inside
::: en-file
  [file](#example-0--fs-file-rs)
:::
::::

:::: en-inside
::: en-file
  [folder](#example-0--fs-folder-rs)
:::
::::

::: en-file
 [main](#example-0--main-rs)
:::
:::::::::::::
::::::::::::::::

[]{#example-0}

## Files and Folders {#example-0-title}

Let's try to understand the Composite pattern with an example of an
operating system's file system. In the file system, there are two types
of objects: files and folders. There are cases when files and folders
should be treated to be the same way. This is where the Composite
pattern comes in handy.

`File` and `Directory` are both of the `trait Component` with a single
`search` method. For a file, it will just look into the contents of the
file; for a folder, it will go through all files of that folder to find
that keyword.

#### []{#example-0--fs-mod-rs .anchor} **fs/mod.rs**

<figure class="code">
<pre class="code" lang="rust"><code>mod file;
mod folder;

pub use file::File;
pub use folder::Folder;

pub trait Component {
    fn search(&amp;self, keyword: &amp;str);
}</code></pre>
</figure>

#### []{#example-0--fs-file-rs .anchor} **fs/file.rs**

<figure class="code">
<pre class="code" lang="rust"><code>use super::Component;

pub struct File {
    name: &amp;&#39;static str,
}

impl File {
    pub fn new(name: &amp;&#39;static str) -&gt; Self {
        Self { name }
    }
}

impl Component for File {
    fn search(&amp;self, keyword: &amp;str) {
        println!(&quot;Searching for keyword {} in file {}&quot;, keyword, self.name);
    }
}</code></pre>
</figure>

#### []{#example-0--fs-folder-rs .anchor} **fs/folder.rs**

<figure class="code">
<pre class="code" lang="rust"><code>use super::Component;

pub struct Folder {
    name: &amp;&#39;static str,
    components: Vec&lt;Box&lt;dyn Component&gt;&gt;,
}

impl Folder {
    pub fn new(name: &amp;&#39;static str) -&gt; Self {
        Self {
            name,
            components: vec![],
        }
    }

    pub fn add(&amp;mut self, component: impl Component + &#39;static) {
        self.components.push(Box::new(component));
    }
}

impl Component for Folder {
    fn search(&amp;self, keyword: &amp;str) {
        println!(
            &quot;Searching recursively for keyword {} in folder {}&quot;,
            keyword, self.name
        );

        for component in self.components.iter() {
            component.search(keyword);
        }
    }
}</code></pre>
</figure>

#### []{#example-0--main-rs .anchor} **main.rs**

<figure class="code">
<pre class="code" lang="rust"><code>mod fs;

use fs::{Component, File, Folder};

fn main() {
    let file1 = File::new(&quot;File 1&quot;);
    let file2 = File::new(&quot;File 2&quot;);
    let file3 = File::new(&quot;File 3&quot;);

    let mut folder1 = Folder::new(&quot;Folder 1&quot;);
    folder1.add(file1);

    let mut folder2 = Folder::new(&quot;Folder 2&quot;);
    folder2.add(file2);
    folder2.add(file3);
    folder2.add(folder1);

    folder2.search(&quot;rose&quot;);
}</code></pre>
</figure>

### Output

<figure class="code">
<pre class="code"><code>Searching recursively for keyword rose in folder Folder 2
Searching for keyword rose in file File 2
Searching for keyword rose in file File 3
Searching recursively for keyword rose in folder Folder 1
Searching for keyword rose in file File 1
------------------------------------</code></pre>
</figure>

::: next
#### 次を読む

[Decorator を Rust で []{.fa
.fa-arrow-right}](../../decorator/rust/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### 戻る

[[]{.fa .fa-arrow-left} Bridge を Rust
で](../../bridge/rust/example.html){.btn .btn-default rel="prev"}
:::

## 他言語での **Composite**

[![Composite を C#
で](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Composite を C# で"){.prog-lang-link}
[![Composite を C++
で](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Composite を C++ で"){.prog-lang-link}
[![Composite を Go
で](../../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Composite を Go で"){.prog-lang-link}
[![Composite を Java
で](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Composite を Java で"){.prog-lang-link}
[![Composite を PHP
で](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Composite を PHP で"){.prog-lang-link}
[![Composite を Python
で](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Composite を Python で"){.prog-lang-link}
[![Composite を Ruby
で](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Composite を Ruby で"){.prog-lang-link}
[![Composite を Swift
で](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Composite を Swift で"){.prog-lang-link}
[![Composite を TypeScript
で](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Composite を TypeScript で"){.prog-lang-link}
::::::::::::::::::::::

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
::::::::::::::::::::::::::::
:::::::::::::::::::::::::::::
