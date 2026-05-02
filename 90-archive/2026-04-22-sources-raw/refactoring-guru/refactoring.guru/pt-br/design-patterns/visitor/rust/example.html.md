:::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [Padrões de
Projeto](../../../design-patterns.html){.type} /
[Visitor](../../visitor.html){.type} / [Rust](../../rust.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Visitor](../../../../images/patterns/cards/visitor-mini0a1c.png?id=854a35a62963bec1d75eab996918989b){srcset="/images/patterns/cards/visitor-mini-2x.png?id=9b87f3f3b772f434b28a25876829b504 2x"}
:::

# **Visitor** em Rust {#visitor-em-rust .pattern-example-title-block-title}
::::

::::::::::: pattern-example-body
::: pattern-example-brief
O **Visitor** é um padrão de projeto comportamental que permite
adicionar novos comportamentos à hierarquia de classes existente sem
alterar nenhum código existente.

> Leia por que os Visitors não podem ser simplesmente substituídos pela
> sobrecarga de método em nosso artigo [Visitor e Double
> Dispatch](../../visitor-double-dispatch.html).
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Saiba mais sobre o Visitor ](../../visitor.html){.btn .btn-lg
.btn-primary}
:::

:::::::: {.sidebar-navigation .with-scroll}
::: en-title
Navegação
:::

::: en-intro
 [Introdução](#)
:::

::: en-intro
 [Deserialization](#example-0)
:::

::: en-file
 [visitor](#example-0--visitor-rs)
:::

::: en-file
 [main](#example-0--main-rs)
:::
::::::::
:::::::::::

[]{#example-0}

## Deserialization {#example-0-title}

A real-world example of the Visitor pattern is [serde serialization
framework](https://serde.rs/) and its deserialization model (see [Serde
data model](https://serde.rs/data-model.html)).

1.  `Visitor` should be implemented for a deserializable type.
2.  `Visitor` is passed to a `Deserializer` (an "Element" in terms of
    the Visitor Pattern), which accepts and drives the `Visitor` in
    order to construct a desired type.

Let's reproduce this deserializing model in our example.

#### []{#example-0--visitor-rs .anchor} **visitor.rs**

<figure class="code">
<pre class="code" lang="rust"><code>use crate::{TwoValuesArray, TwoValuesStruct};

/// Visitor can visit one type, do conversions, and output another type.
///
/// It&#39;s not like all visitors must return a new type, it&#39;s just an example
/// that demonstrates the technique.
pub trait Visitor {
    type Value;

    /// Visits a vector of integers and outputs a desired type.
    fn visit_vec(&amp;self, v: Vec&lt;i32&gt;) -&gt; Self::Value;
}

/// Visitor implementation for a struct of two values.
impl Visitor for TwoValuesStruct {
    type Value = TwoValuesStruct;

    fn visit_vec(&amp;self, v: Vec&lt;i32&gt;) -&gt; Self::Value {
        TwoValuesStruct { a: v[0], b: v[1] }
    }
}

/// Visitor implementation for a struct of values array.
impl Visitor for TwoValuesArray {
    type Value = TwoValuesArray;

    fn visit_vec(&amp;self, v: Vec&lt;i32&gt;) -&gt; Self::Value {
        let mut ab = [0i32; 2];

        ab[0] = v[0];
        ab[1] = v[1];

        TwoValuesArray { ab }
    }
}</code></pre>
</figure>

#### []{#example-0--main-rs .anchor} **main.rs**

<figure class="code">
<pre class="code" lang="rust"><code>#![allow(unused)]

mod visitor;

use visitor::Visitor;

/// A struct of two integer values.
///
/// It&#39;s going to be an output of `Visitor` trait which is defined for the type
/// in `visitor.rs`.
#[derive(Default, Debug)]
pub struct TwoValuesStruct {
    a: i32,
    b: i32,
}

/// A struct of values array.
///
/// It&#39;s going to be an output of `Visitor` trait which is defined for the type
/// in `visitor.rs`.
#[derive(Default, Debug)]
pub struct TwoValuesArray {
    ab: [i32; 2],
}

/// `Deserializer` trait defines methods that can parse either a string or
/// a vector, it accepts a visitor which knows how to construct a new object
/// of a desired type (in our case, `TwoValuesArray` and `TwoValuesStruct`).
trait Deserializer&lt;V: Visitor&gt; {
    fn create(visitor: V) -&gt; Self;
    fn parse_str(&amp;self, input: &amp;str) -&gt; Result&lt;V::Value, &amp;&#39;static str&gt; {
        Err(&quot;parse_str is unimplemented&quot;)
    }
    fn parse_vec(&amp;self, input: Vec&lt;i32&gt;) -&gt; Result&lt;V::Value, &amp;&#39;static str&gt; {
        Err(&quot;parse_vec is unimplemented&quot;)
    }
}

struct StringDeserializer&lt;V: Visitor&gt; {
    visitor: V,
}

impl&lt;V: Visitor&gt; Deserializer&lt;V&gt; for StringDeserializer&lt;V&gt; {
    fn create(visitor: V) -&gt; Self {
        Self { visitor }
    }

    fn parse_str(&amp;self, input: &amp;str) -&gt; Result&lt;V::Value, &amp;&#39;static str&gt; {
        // In this case, in order to apply a visitor, a deserializer should do
        // some preparation. The visitor does its stuff, but it doesn&#39;t do everything.
        let input_vec = input
            .split_ascii_whitespace()
            .map(|x| x.parse().unwrap())
            .collect();

        Ok(self.visitor.visit_vec(input_vec))
    }
}

struct VecDeserializer&lt;V: Visitor&gt; {
    visitor: V,
}

impl&lt;V: Visitor&gt; Deserializer&lt;V&gt; for VecDeserializer&lt;V&gt; {
    fn create(visitor: V) -&gt; Self {
        Self { visitor }
    }

    fn parse_vec(&amp;self, input: Vec&lt;i32&gt;) -&gt; Result&lt;V::Value, &amp;&#39;static str&gt; {
        Ok(self.visitor.visit_vec(input))
    }
}

fn main() {
    let deserializer = StringDeserializer::create(TwoValuesStruct::default());
    let result = deserializer.parse_str(&quot;123 456&quot;);
    println!(&quot;{:?}&quot;, result);

    let deserializer = VecDeserializer::create(TwoValuesStruct::default());
    let result = deserializer.parse_vec(vec![123, 456]);
    println!(&quot;{:?}&quot;, result);

    let deserializer = VecDeserializer::create(TwoValuesArray::default());
    let result = deserializer.parse_vec(vec![123, 456]);
    println!(&quot;{:?}&quot;, result);

    println!(
        &quot;Error: {}&quot;,
        deserializer.parse_str(&quot;123 456&quot;).err().unwrap()
    )
}</code></pre>
</figure>

### Output

<figure class="code">
<pre class="code"><code>Ok(TwoValuesStruct { a: 123, b: 456 })
Ok(TwoValuesStruct { a: 123, b: 456 })
Ok(TwoValuesArray { ab: [123, 456] })
Error: parse_str unimplemented</code></pre>
</figure>

::: next
#### Leia a seguir

[Padrões de Projeto em Swift []{.fa
.fa-arrow-right}](../../swift.html){.btn .btn-primary rel="next"}
:::

::: prev
#### Voltar

[[]{.fa .fa-arrow-left} Template Method em
Rust](../../template-method/rust/example.html){.btn .btn-default
rel="prev"}
:::

## **Visitor** em outras linguagens

[![Visitor em
C#](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Visitor em C#"){.prog-lang-link}
[![Visitor em
C++](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Visitor em C++"){.prog-lang-link}
[![Visitor em
Go](../../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Visitor em Go"){.prog-lang-link}
[![Visitor em
Java](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Visitor em Java"){.prog-lang-link}
[![Visitor em
PHP](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Visitor em PHP"){.prog-lang-link}
[![Visitor em
Python](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Visitor em Python"){.prog-lang-link}
[![Visitor em
Ruby](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Visitor em Ruby"){.prog-lang-link}
[![Visitor em
Swift](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Visitor em Swift"){.prog-lang-link}
[![Visitor em
TypeScript](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Visitor em TypeScript"){.prog-lang-link}
:::::::::::::::::

::::::: {.prom .banner-sidebar .banner .banner-striped .banner-examples .banner-removable .banner-removable-patterns data-id="DP: Examples-IDE" creative-id="examples-sidebar-pt-br" position="sidebar"}
:::::: {.banner-inner style="font-size: 14px; line-height: 21px;"}
::: banner-examples-img
[![](../../../../images/patterns/banners/examples-ide91ca.png?id=3115b4b548fb96b75974e2de8f4f49bc){width="215"
height="158" loading="lazy"
srcset="/images/patterns/banners/examples-ide-2x.png?id=93c007a6d157b97c28eb213f59d716ae 2x"}](../../book.html)
:::

::: banner-examples-fade
[ Arquivo com exemplos de código](../../book.html){.btn .btn-secondary
.btn-block}
:::

::: {.banner-examples-text style="margin-top: 1rem;"}
Compre o eBook **Mergulho nos Padrões de Projeto** e tenha acesso ao
arquivo com dúzias de exemplos detalhados que podem ser abertos
diretamente em seu IDE.

[ Saiba mais...](../../book.html){.btn .btn-secondary .btn-block}
:::
::::::
:::::::
:::::::::::::::::::::::
::::::::::::::::::::::::
