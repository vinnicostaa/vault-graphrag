:::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [Паттерны
проектирования](../../../design-patterns.html){.type} /
[Снимок](../../memento.html){.type} / [Rust](../../rust.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Снимок](../../../../images/patterns/cards/memento-mini723d.png?id=8b2ea4dc2c5d15775a654808cc9de099){srcset="/images/patterns/cards/memento-mini-2x.png?id=1d7cba189261dd84b11369a6838b1055 2x"}
:::

# **Снимок** на Rust {#снимок-на-rust .pattern-example-title-block-title}
::::

:::::::::::: pattern-example-body
::: pattern-example-brief
**Снимок** --- это поведенческий паттерн, позволяющий делать снимки
внутреннего состояния объектов, а затем восстанавливать их.

При этом Снимок не раскрывает подробностей реализации объектов, и клиент
не имеет доступа к защищённой информации объекта.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Подробней о паттерне Снимок ](../../memento.html){.btn .btn-lg
.btn-primary}
:::

::::::::: {.sidebar-navigation .with-scroll}
::: en-title
Навигация
:::

::: en-intro
 [Интро](#)
:::

::: en-intro
 [Conceptual example](#example-0)
:::

::: en-file
 [conceptual](#example-0--conceptual-rs)
:::

::: en-intro
 [Serde serialization framework](#example-1)
:::

::: en-file
 [serde](#example-1--serde-rs)
:::
:::::::::
::::::::::::

::: {#nav-tab .nav .nav-tabs .nav-justified role="tablist"}
[Conceptual example](#example-0){#example-0-tab .nav-item .nav-link
.active toggle="tab" role="tab" aria-controls="example-0"
aria-selected="true"} [Serde serialization
framework](#example-1){#example-1-tab .nav-item .nav-link toggle="tab"
role="tab" aria-controls="example-1" aria-selected="true"}
:::

::::: {#nav-tabContent .pattern-example-tab-content .tab-content}
::: {#example-0 .tab-pane .show .active role="tabpanel" aria-labelledby="example-0-tab" style="padding-top: 24px"}
## Conceptual example {#example-0-title}

This is a conceptual example of Memento pattern.

#### []{#example-0--conceptual-rs .anchor} **conceptual.rs**

<figure class="code">
<pre class="code" lang="rust"><code>trait Memento&lt;T&gt; {
    fn restore(self) -&gt; T;
    fn print(&amp;self);
}

struct Originator {
    state: u32,
}

impl Originator {
    pub fn save(&amp;self) -&gt; OriginatorBackup {
        OriginatorBackup {
            state: self.state.to_string(),
        }
    }
}

struct OriginatorBackup {
    state: String,
}

impl Memento&lt;Originator&gt; for OriginatorBackup {
    fn restore(self) -&gt; Originator {
        Originator {
            state: self.state.parse().unwrap(),
        }
    }

    fn print(&amp;self) {
        println!(&quot;Originator backup: &#39;{}&#39;&quot;, self.state);
    }
}

fn main() {
    let mut history = Vec::&lt;OriginatorBackup&gt;::new();

    let mut originator = Originator { state: 0 };

    originator.state = 1;
    history.push(originator.save());

    originator.state = 2;
    history.push(originator.save());

    for moment in history.iter() {
        moment.print();
    }

    let originator = history.pop().unwrap().restore();
    println!(&quot;Restored to state: {}&quot;, originator.state);

    let originator = history.pop().unwrap().restore();
    println!(&quot;Restored to state: {}&quot;, originator.state);
}</code></pre>
</figure>

### Output

<figure class="code">
<pre class="code"><code>Originator backup: &#39;1&#39;
Originator backup: &#39;2&#39;
Restored to state: 2
Restored to state: 1</code></pre>
</figure>
:::

::: {#example-1 .tab-pane role="tabpanel" aria-labelledby="example-1-tab" style="padding-top: 24px"}
## Serde serialization framework {#example-1-title}

A common way to make a structure serializable is to derive `Serialize`
and `Deserialize` traits from the [serde serialization
framework](https://crates.io/crates/serde). Then an object of
serializable type can be converted to many different formats, e.g. JSON
with [serde_json](https://crates.io/crates/serde_json) crate.

<figure class="code">
<pre class="code" lang="rust"><code>use serde::{Deserialize, Serialize};

#[derive(Serialize, Deserialize)]
struct Originator {
    state: u32,
}</code></pre>
</figure>

#### []{#example-1--serde-rs .anchor} **serde.rs**

<figure class="code">
<pre class="code" lang="rust"><code>use serde::{Deserialize, Serialize};

/// An object to be stored. It derives a default
/// `Serialize` and `Deserialize` trait implementation, which
/// allows to convert it into many different formats (e.g. JSON).
#[derive(Serialize, Deserialize)]
struct Originator {
    state: u32,
}

impl Originator {
    /// Serializes an originator into a string of JSON format.
    pub fn save(&amp;self) -&gt; String {
        serde_json::to_string(self).unwrap()
    }

    /// Deserializes an originator into a string of JSON format.
    pub fn restore(json: &amp;str) -&gt; Self {
        serde_json::from_str(json).unwrap()
    }
}

fn main() {
    // A stack of mementos.
    let mut history = Vec::&lt;String&gt;::new();

    let mut originator = Originator { state: 0 };

    originator.state = 1;
    history.push(originator.save());

    originator.state = 2;
    history.push(originator.save());

    for moment in history.iter() {
        println!(&quot;{}&quot;, moment);
    }

    let originator = Originator::restore(&amp;history.pop().unwrap());
    println!(&quot;Restored to state: {}&quot;, originator.state);

    let originator = Originator::restore(&amp;history.pop().unwrap());
    println!(&quot;Restored to state: {}&quot;, originator.state);
}</code></pre>
</figure>

### Output

<figure class="code">
<pre class="code"><code>{&quot;state&quot;:1}
{&quot;state&quot;:2}
Restored to state: 2
Restored to state: 1</code></pre>
</figure>
:::
:::::

::: {#nav-tab .nav .nav-tabs .nav-tabs-bottom .nav-justified role="tablist"}
[Conceptual example](#example-0){#example-0-tab-bottom .nav-item
.nav-link .active toggle="tab" role="tab" aria-controls="example-0"
aria-selected="true"} [Serde serialization
framework](#example-1){#example-1-tab-bottom .nav-item .nav-link
toggle="tab" role="tab" aria-controls="example-1" aria-selected="true"}
:::

::: next
#### Читаем дальше

[Наблюдатель на Rust []{.fa
.fa-arrow-right}](../../observer/rust/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Вернуться назад

[[]{.fa .fa-arrow-left} Посредник на
Rust](../../mediator/rust/example.html){.btn .btn-default rel="prev"}
:::

## **Снимок** на других языках программирования

[![Снимок на
C#](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Снимок на C#"){.prog-lang-link}
[![Снимок на
C++](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Снимок на C++"){.prog-lang-link}
[![Снимок на
Go](../../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Снимок на Go"){.prog-lang-link}
[![Снимок на
Java](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Снимок на Java"){.prog-lang-link}
[![Снимок на
PHP](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Снимок на PHP"){.prog-lang-link}
[![Снимок на
Python](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Снимок на Python"){.prog-lang-link}
[![Снимок на
Ruby](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Снимок на Ruby"){.prog-lang-link}
[![Снимок на
Swift](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Снимок на Swift"){.prog-lang-link}
[![Снимок на
TypeScript](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Снимок на TypeScript"){.prog-lang-link}
:::::::::::::::::::::::

::::::: {.prom .banner-sidebar .banner .banner-striped .banner-examples .banner-removable .banner-removable-patterns data-id="DP: Examples-IDE" creative-id="examples-sidebar-ru" position="sidebar"}
:::::: {.banner-inner style="font-size: 14px; line-height: 21px;"}
::: banner-examples-img
[![](../../../../images/patterns/banners/examples-ide91ca.png?id=3115b4b548fb96b75974e2de8f4f49bc){width="215"
height="158" loading="lazy"
srcset="/images/patterns/banners/examples-ide-2x.png?id=93c007a6d157b97c28eb213f59d716ae 2x"}](../../book.html)
:::

::: banner-examples-fade
[ Архив с примерами](../../book.html){.btn .btn-secondary .btn-block}
:::

::: {.banner-examples-text style="margin-top: 1rem;"}
Купи книгу **Погружение в Паттерны** и получи архив с десятками
детальных примеров, которые можно открывать прямо в IDE.

[ Узнать больше...](../../book.html){.btn .btn-secondary .btn-block}
:::
::::::
:::::::
:::::::::::::::::::::::::::::
::::::::::::::::::::::::::::::
