::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [Патерни
проектування](../../../design-patterns.html){.type} /
[Прототип](../../prototype.html){.type} / [Rust](../../rust.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Прототип](../../../../images/patterns/cards/prototype-mini6540.png?id=bc3046bb39ff36574c08d49839fd1c8e){srcset="/images/patterns/cards/prototype-mini-2x.png?id=b871f789a736e7efbb1cd082d2de6398 2x"}
:::

# **Прототип** на Rust {#прототип-на-rust .pattern-example-title-block-title}
::::

:::::::::: pattern-example-body
::: pattern-example-brief
**Прототип** --- це породжуючий патерн, який дозволяє копіювати об'єкти
будь-якої складності без прив'язки до їхніх конкретних класів.

Усі класи-Прототипи мають спільний інтерфейс. Тому ви можете копіювати
об'єкти, не звертаючи уваги на їхні конкретні типи та бути завжди
впевненими в тому, що отримаєте точну копію. Клонування здійснюється
самим об'єктом-прототипу, що дозволяє йому скопіювати значення всіх
полів, навіть приватних.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Детальніше про Прототип ](../../prototype.html){.btn .btn-lg
.btn-primary}
:::

::::::: {.sidebar-navigation .with-scroll}
::: en-title
Навігація
:::

::: en-intro
 [Інтро](#)
:::

::: en-intro
 [Built-in *Clone* trait](#example-0)
:::

::: en-file
 [main](#example-0--main-rs)
:::
:::::::
::::::::::

[]{#example-0}

## Built-in *Clone* trait {#example-0-title}

Rust has a built-in `std::clone::Clone` trait with many implementations
for various types (via `#[derive(Clone)]`). Thus, the Prototype pattern
is ready to use out of the box.

#### []{#example-0--main-rs .anchor} **main.rs**

<figure class="code">
<pre class="code" lang="rust"><code>#[derive(Clone)]
struct Circle {
    pub x: u32,
    pub y: u32,
    pub radius: u32,
}

fn main() {
    let circle1 = Circle {
        x: 10,
        y: 15,
        radius: 10,
    };

    // Prototype in action.
    let mut circle2 = circle1.clone();
    circle2.radius = 77;

    println!(&quot;Circle 1: {}, {}, {}&quot;, circle1.x, circle1.y, circle1.radius);
    println!(&quot;Circle 2: {}, {}, {}&quot;, circle2.x, circle2.y, circle2.radius);
}</code></pre>
</figure>

### Output

<figure class="code">
<pre class="code"><code>Circle 1: 10, 15, 10
Circle 2: 10, 15, 77</code></pre>
</figure>

::: next
#### Читаємо далі

[Одинак на Rust []{.fa
.fa-arrow-right}](../../singleton/rust/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Повернутись назад

[[]{.fa .fa-arrow-left} Фабричний метод на
Rust](../../factory-method/rust/example.html){.btn .btn-default
rel="prev"}
:::

## **Прототип** іншими мовами програмування

[![Прототип на
C#](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Прототип на C#"){.prog-lang-link}
[![Прототип на
C++](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Прототип на C++"){.prog-lang-link}
[![Прототип на
Go](../../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Прототип на Go"){.prog-lang-link}
[![Прототип на
Java](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Прототип на Java"){.prog-lang-link}
[![Прототип на
PHP](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Прототип на PHP"){.prog-lang-link}
[![Прототип на
Python](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Прототип на Python"){.prog-lang-link}
[![Прототип на
Ruby](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Прототип на Ruby"){.prog-lang-link}
[![Прототип на
Swift](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Прототип на Swift"){.prog-lang-link}
[![Прототип на
TypeScript](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Прототип на TypeScript"){.prog-lang-link}
::::::::::::::::

::::::: {.prom .banner-sidebar .banner .banner-striped .banner-examples .banner-removable .banner-removable-patterns data-id="DP: Examples-IDE" creative-id="examples-sidebar-uk" position="sidebar"}
:::::: {.banner-inner style="font-size: 14px; line-height: 21px;"}
::: banner-examples-img
[![](../../../../images/patterns/banners/examples-ide91ca.png?id=3115b4b548fb96b75974e2de8f4f49bc){width="215"
height="158" loading="lazy"
srcset="/images/patterns/banners/examples-ide-2x.png?id=93c007a6d157b97c28eb213f59d716ae 2x"}](../../book.html)
:::

::: banner-examples-fade
[ Архів з прикладами](../../book.html){.btn .btn-secondary .btn-block}
:::

::: {.banner-examples-text style="margin-top: 1rem;"}
Придбай книгу **Занурення в Патерни** та отримай архів з десятками
детальних прикладів, які можна відкривати прямо в IDE.

[ Дізнатися більше...](../../book.html){.btn .btn-secondary .btn-block}
:::
::::::
:::::::
::::::::::::::::::::::
:::::::::::::::::::::::
