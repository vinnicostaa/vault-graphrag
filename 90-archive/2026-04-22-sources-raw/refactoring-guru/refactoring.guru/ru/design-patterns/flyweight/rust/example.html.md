:::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [Паттерны
проектирования](../../../design-patterns.html){.type} /
[Легковес](../../flyweight.html){.type} / [Rust](../../rust.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Легковес](../../../../images/patterns/cards/flyweight-minie3d8.png?id=422ca8d2f90614dce810a8812c626698){srcset="/images/patterns/cards/flyweight-mini-2x.png?id=857ca676fff84317bb0dab76abfce08e 2x"}
:::

# **Легковес** на Rust {#легковес-на-rust .pattern-example-title-block-title}
::::

::::::::::::: pattern-example-body
::: pattern-example-brief
**Легковес** --- это структурный паттерн, который экономит память,
благодаря разделению общего состояния, вынесенного в один объект, между
множеством объектов.

Легковес позволяет экономить память, кешируя одинаковые данные,
используемые в разных объектах.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Подробней о паттерне Легковес ](../../flyweight.html){.btn .btn-lg
.btn-primary}
:::

:::::::::: {.sidebar-navigation .with-scroll}
::: en-title
Навигация
:::

::: en-intro
 [Интро](#)
:::

::: en-intro
 [Rendering a Forest](#example-0)
:::

::: en-file
 [forest](#example-0--forest-rs)
:::

:::: en-inside
::: en-file
  [tree](#example-0--forest-tree-rs)
:::
::::

::: en-file
 [main](#example-0--main-rs)
:::
::::::::::
:::::::::::::

[]{#example-0}

## Rendering a Forest {#example-0-title}

::: next
#### Читаем дальше

[Заместитель на Rust []{.fa
.fa-arrow-right}](../../proxy/rust/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Вернуться назад

[[]{.fa .fa-arrow-left} Фасад на
Rust](../../facade/rust/example.html){.btn .btn-default rel="prev"}
:::

## **Легковес** на других языках программирования

[![Легковес на
C#](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Легковес на C#"){.prog-lang-link}
[![Легковес на
C++](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Легковес на C++"){.prog-lang-link}
[![Легковес на
Go](../../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Легковес на Go"){.prog-lang-link}
[![Легковес на
Java](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Легковес на Java"){.prog-lang-link}
[![Легковес на
PHP](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Легковес на PHP"){.prog-lang-link}
[![Легковес на
Python](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Легковес на Python"){.prog-lang-link}
[![Легковес на
Ruby](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Легковес на Ruby"){.prog-lang-link}
[![Легковес на
Swift](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Легковес на Swift"){.prog-lang-link}
[![Легковес на
TypeScript](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Легковес на TypeScript"){.prog-lang-link}
:::::::::::::::::::

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
:::::::::::::::::::::::::
::::::::::::::::::::::::::
