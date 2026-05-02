::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [Patrons de
conception](../../../design-patterns.html){.type} /
[Décorateur](../../decorator.html){.type} /
[Rust](../../rust.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Décorateur](../../../../images/patterns/cards/decorator-mini6adb.png?id=d30458908e315af195cb183bc52dbef9){srcset="/images/patterns/cards/decorator-mini-2x.png?id=3b58e540d7d28523080cad341ed9b2e9 2x"}
:::

# **Décorateur** en Rust {#décorateur-en-rust .pattern-example-title-block-title}
::::

:::::::::: pattern-example-body
::: pattern-example-brief
Le **Décorateur** est un patron de conception structurel qui permet
d'ajouter dynamiquement de nouveaux comportements à des objets en les
plaçant à l'intérieur d'objets spéciaux appelés emballeurs (wrappers).

À l'aide de ces décorateurs, vous pouvez emballer des objets de
nombreuses fois, puisque les objets ciblés et les décorateurs
implémentent la même interface. L'objet final recevra tous les
comportements de tous les emballeurs.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ En savoir plus sur la patron Décorateur ](../../decorator.html){.btn
.btn-lg .btn-primary}
:::

::::::: {.sidebar-navigation .with-scroll}
::: en-title
Navigation
:::

::: en-intro
 [Intro](#)
:::

::: en-intro
 [Input streams decoration](#example-0)
:::

::: en-file
 [main](#example-0--main-rs)
:::
:::::::
::::::::::

[]{#example-0}

## Input streams decoration {#example-0-title}

There is a ***practical example*** in Rust's standard library for
input/output operations.

A buffered reader decorates a vector reader adding buffered behavior.

<figure class="code">
<pre class="code" lang="rust"><code>let mut input = BufReader::new(Cursor::new(&quot;Input data&quot;));
input.read(&amp;mut buf).ok();</code></pre>
</figure>

#### []{#example-0--main-rs .anchor} **main.rs**

<figure class="code">
<pre class="code" lang="rust"><code>use std::io::{BufReader, Cursor, Read};

fn main() {
    let mut buf = [0u8; 10];

    // A buffered reader decorates a vector reader which wraps input data.
    let mut input = BufReader::new(Cursor::new(&quot;Input data&quot;));

    input.read(&amp;mut buf).ok();

    print!(&quot;Read from a buffered reader: &quot;);

    for byte in buf {
        print!(&quot;{}&quot;, char::from(byte));
    }

    println!();
}</code></pre>
</figure>

### Output

<figure class="code">
<pre class="code"><code>Read from a buffered reader: Input data</code></pre>
</figure>

::: next
#### Suivant

[Façade en Rust []{.fa
.fa-arrow-right}](../../facade/rust/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Retour

[[]{.fa .fa-arrow-left} Composite en
Rust](../../composite/rust/example.html){.btn .btn-default rel="prev"}
:::

## **Décorateur** dans les autres langues

[![Décorateur en
C#](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Décorateur en C#"){.prog-lang-link}
[![Décorateur en
C++](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Décorateur en C++"){.prog-lang-link}
[![Décorateur en
Go](../../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Décorateur en Go"){.prog-lang-link}
[![Décorateur en
Java](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Décorateur en Java"){.prog-lang-link}
[![Décorateur en
PHP](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Décorateur en PHP"){.prog-lang-link}
[![Décorateur en
Python](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Décorateur en Python"){.prog-lang-link}
[![Décorateur en
Ruby](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Décorateur en Ruby"){.prog-lang-link}
[![Décorateur en
Swift](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Décorateur en Swift"){.prog-lang-link}
[![Décorateur en
TypeScript](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Décorateur en TypeScript"){.prog-lang-link}
::::::::::::::::

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
::::::::::::::::::::::
:::::::::::::::::::::::
