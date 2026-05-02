::::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../index.html){.home} / [Design
Patterns](../../../design-patterns.html){.type} / [Abstract
Factory](../../abstract-factory.html){.type} /
[Rust](../../rust.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Abstract
Factory](../../../images/patterns/cards/abstract-factory-minid1c5.png?id=4c3927c446313a38ce77dfee38111e27){srcset="/images/patterns/cards/abstract-factory-mini-2x.png?id=22236aaa65ff52cbde1c713216d52c1f 2x"}
:::

# **Abstract Factory** in Rust {#abstract-factory-in-rust .pattern-example-title-block-title}
::::

:::::::::::::::::::::::::::: pattern-example-body
::: pattern-example-brief
**Abstract Factory** is a creational design pattern, which solves the
problem of creating entire product families without specifying their
concrete classes.

Abstract Factory defines an interface for creating all distinct products
but leaves the actual product creation to concrete factory classes. Each
factory type corresponds to a certain product variety.

The client code calls the creation methods of a factory object instead
of creating products directly with a constructor call (`new` operator).
Since a factory corresponds to a single product variant, all its
products will be compatible.

Client code works with factories and products only through their
abstract interfaces. This lets the client code work with any product
variants, created by the factory object. You just create a new concrete
factory class and pass it to the client code.

> If you can't figure out the difference between various factory
> patterns and concepts, then read our [Factory
> Comparison](../../factory-comparison.html).
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Learn more about Abstract Factory ](../../abstract-factory.html){.btn
.btn-lg .btn-primary}
:::

::::::::::::::::::::::::: {.sidebar-navigation .with-scroll}
::: en-title
Navigation
:::

::: en-intro
 [Intro](#)
:::

::: en-intro
 [GUI Elements Factory](#example-0)
:::

::: en-file
 gui
:::

:::: en-inside
::: en-file
  [lib](#example-0--gui-lib-rs)
:::
::::

::: en-file
 macos-gui
:::

:::: en-inside
::: en-file
  [lib](#example-0--macos-gui-lib-rs)
:::
::::

::: en-file
 windows-gui
:::

:::: en-inside
::: en-file
  [lib](#example-0--windows-gui-lib-rs)
:::
::::

::: en-file
 app
:::

:::: en-inside
::: en-file
  [main](#example-0--app-main-rs)
:::
::::

:::: en-inside
::: en-file
  [render](#example-0--app-render-rs)
:::
::::

::: en-file
 app-dyn
:::

:::: en-inside
::: en-file
  [main](#example-0--app-dyn-main-rs)
:::
::::

:::: en-inside
::: en-file
  [render](#example-0--app-dyn-render-rs)
:::
::::
:::::::::::::::::::::::::
::::::::::::::::::::::::::::

[]{#example-0}

## GUI Elements Factory {#example-0-title}

This example illustrates how a GUI framework can organize its classes
into independent libraries:

1.  The `gui` library defines interfaces for all the components.\
    It has no external dependencies.
2.  The `windows-gui` library provides Windows implementation of the
    base GUI.\
    Depends on `gui`.
3.  The `macos-gui` library provides Mac OS implementation of the base
    GUI.\
    Depends on `gui`.

The `app` is a client application that can use several implementations
of the GUI framework, depending on the current environment or
configuration. However, most of the `app` code *doesn't depend on
specific types of GUI elements*. All the client code works with GUI
elements through abstract interfaces (traits) defined by the `gui` lib.

There are two approaches to implementing abstract factories in Rust:

-   using generics (*static dispatch*)
-   using dynamic allocation (*dynamic dispatch*)

When you're given a choice between static and dynamic dispatch, there is
rarely a clear-cut correct answer. You'll want to use static dispatch in
your libraries and dynamic dispatch in your binaries. In a library, you
want to allow your users to decide what kind of dispatch is best for
them since you don't know what their needs are. If you use dynamic
dispatch, they're forced to do the same, whereas if you use static
dispatch, they can choose whether to use dynamic dispatch or not.

### []{#example-0--gui .anchor} **gui:** Abstract Factory and Abstract Products {#gui-abstract-factory-and-abstract-products .example-title}

#### []{#example-0--gui-lib-rs .anchor} **gui/lib.rs**

<figure class="code">
<pre class="code" lang="rust"><code>pub trait Button {
    fn press(&amp;self);
}

pub trait Checkbox {
    fn switch(&amp;self);
}

/// Abstract Factory defined using generics.
pub trait GuiFactory {
    type B: Button;
    type C: Checkbox;

    fn create_button(&amp;self) -&gt; Self::B;
    fn create_checkbox(&amp;self) -&gt; Self::C;
}

/// Abstract Factory defined using Box pointer.
pub trait GuiFactoryDynamic {
    fn create_button(&amp;self) -&gt; Box&lt;dyn Button&gt;;
    fn create_checkbox(&amp;self) -&gt; Box&lt;dyn Checkbox&gt;;
}</code></pre>
</figure>

### []{#example-0--macos-gui .anchor} **macos-gui:** One family of products {#macos-gui-one-family-of-products .example-title}

#### []{#example-0--macos-gui-lib-rs .anchor} **macos-gui/lib.rs**

<figure class="code">
<pre class="code" lang="rust"><code>pub mod button;
pub mod checkbox;
pub mod factory;</code></pre>
</figure>

### []{#example-0--windows-gui .anchor} **windows-gui:** Another family of products {#windows-gui-another-family-of-products .example-title}

#### []{#example-0--windows-gui-lib-rs .anchor} **windows-gui/lib.rs**

<figure class="code">
<pre class="code" lang="rust"><code>pub mod button;
pub mod checkbox;
pub mod factory;</code></pre>
</figure>

#### Static dispatch

Here, the abstract factory is implemented via **generics** which lets
the compiler create a code that does NOT require dynamic dispatch in
runtime.

### []{#example-0--app .anchor} **app:** Client code with static dispatch {#app-client-code-with-static-dispatch .example-title}

#### []{#example-0--app-main-rs .anchor} **app/main.rs**

<figure class="code">
<pre class="code" lang="rust"><code>mod render;

use render::render;

use macos_gui::factory::MacFactory;
use windows_gui::factory::WindowsFactory;

fn main() {
    let windows = true;

    if windows {
        render(WindowsFactory);
    } else {
        render(MacFactory);
    }
}</code></pre>
</figure>

#### []{#example-0--app-render-rs .anchor} **app/render.rs**

<figure class="code">
<pre class="code" lang="rust"><code>//! The code demonstrates that it doesn&#39;t depend on a concrete
//! factory implementation.

use gui::GuiFactory;

// Renders GUI. Factory object must be passed as a parameter to such the
// generic function with factory invocation to utilize static dispatch.
pub fn render(factory: impl GuiFactory) {
    let button1 = factory.create_button();
    let button2 = factory.create_button();
    let checkbox1 = factory.create_checkbox();
    let checkbox2 = factory.create_checkbox();

    use gui::{Button, Checkbox};

    button1.press();
    button2.press();
    checkbox1.switch();
    checkbox2.switch();
}</code></pre>
</figure>

#### Dynamic dispatch

If a concrete type of abstract factory is not known at the compilation
time, then is should be implemented using `Box` pointers.

### []{#example-0--app-dyn .anchor} **app-dyn:** Client code with dynamic dispatch {#app-dyn-client-code-with-dynamic-dispatch .example-title}

#### []{#example-0--app-dyn-main-rs .anchor} **app-dyn/main.rs**

<figure class="code">
<pre class="code" lang="rust"><code>mod render;

use render::render;

use gui::GuiFactoryDynamic;
use macos_gui::factory::MacFactory;
use windows_gui::factory::WindowsFactory;

fn main() {
    let windows = false;

    // Allocate a factory object in runtime depending on unpredictable input.
    let factory: &amp;dyn GuiFactoryDynamic = if windows {
        &amp;WindowsFactory
    } else {
        &amp;MacFactory
    };

    // Factory invocation can be inlined right here.
    let button = factory.create_button();
    button.press();

    // Factory object can be passed to a function as a parameter.
    render(factory);
}</code></pre>
</figure>

#### []{#example-0--app-dyn-render-rs .anchor} **app-dyn/render.rs**

<figure class="code">
<pre class="code" lang="rust"><code>//! The code demonstrates that it doesn&#39;t depend on a concrete
//! factory implementation.

use gui::GuiFactoryDynamic;

/// Renders GUI.
pub fn render(factory: &amp;dyn GuiFactoryDynamic) {
    let button1 = factory.create_button();
    let button2 = factory.create_button();
    let checkbox1 = factory.create_checkbox();
    let checkbox2 = factory.create_checkbox();

    button1.press();
    button2.press();
    checkbox1.switch();
    checkbox2.switch();
}</code></pre>
</figure>

### Output

<figure class="code">
<pre class="code"><code>Windows button has pressed
Windows button has pressed
Windows checkbox has switched
Windows checkbox has switched</code></pre>
</figure>

::: next
#### Read next

[Builder in Rust []{.fa
.fa-arrow-right}](../../builder/rust/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Return

[[]{.fa .fa-arrow-left} Design Patterns in Rust](../../rust.html){.btn
.btn-default rel="prev"}
:::

## **Abstract Factory** in Other Languages

[![Abstract Factory in
C#](../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Abstract Factory in C#"){.prog-lang-link}
[![Abstract Factory in
C++](../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Abstract Factory in C++"){.prog-lang-link}
[![Abstract Factory in
Go](../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Abstract Factory in Go"){.prog-lang-link}
[![Abstract Factory in
Java](../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Abstract Factory in Java"){.prog-lang-link}
[![Abstract Factory in
PHP](../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Abstract Factory in PHP"){.prog-lang-link}
[![Abstract Factory in
Python](../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Abstract Factory in Python"){.prog-lang-link}
[![Abstract Factory in
Ruby](../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Abstract Factory in Ruby"){.prog-lang-link}
[![Abstract Factory in
Swift](../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Abstract Factory in Swift"){.prog-lang-link}
[![Abstract Factory in
TypeScript](../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Abstract Factory in TypeScript"){.prog-lang-link}
::::::::::::::::::::::::::::::::::

::::::: {.prom .banner-sidebar .banner .banner-striped .banner-examples .banner-removable .banner-removable-patterns data-id="DP: Examples-IDE" creative-id="examples-sidebar-en" position="sidebar"}
:::::: {.banner-inner style="font-size: 14px; line-height: 21px;"}
::: banner-examples-img
[![](../../../images/patterns/banners/examples-ide91ca.png?id=3115b4b548fb96b75974e2de8f4f49bc){width="215"
height="158" loading="lazy"
srcset="/images/patterns/banners/examples-ide-2x.png?id=93c007a6d157b97c28eb213f59d716ae 2x"}](../../book.html)
:::

::: banner-examples-fade
[ Archive with examples](../../book.html){.btn .btn-secondary
.btn-block}
:::

::: {.banner-examples-text style="margin-top: 1rem;"}
Buy the eBook **Dive Into Design Patterns** and get the access to
archive with dozens of detailed examples that can be opened right in
your IDE.

[ Learn more...](../../book.html){.btn .btn-secondary .btn-block}
:::
::::::
:::::::
::::::::::::::::::::::::::::::::::::::::
:::::::::::::::::::::::::::::::::::::::::
