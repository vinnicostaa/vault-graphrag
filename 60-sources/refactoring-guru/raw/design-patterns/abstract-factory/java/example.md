:::::::::::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::::::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../index.html){.home} / [Design
Patterns](../../../design-patterns.html){.type} / [Abstract
Factory](../../abstract-factory.html){.type} /
[Java](../../java.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Abstract
Factory](../../../images/patterns/cards/abstract-factory-minid1c5.png?id=4c3927c446313a38ce77dfee38111e27){srcset="/images/patterns/cards/abstract-factory-mini-2x.png?id=22236aaa65ff52cbde1c713216d52c1f 2x"}
:::

# **Abstract Factory** in Java {#abstract-factory-in-java .pattern-example-title-block-title}
::::

::::::::::::::::::::::::::::::::::: pattern-example-body
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

:::::::::::::::::::::::::::::::: {.sidebar-navigation .with-scroll}
::: en-title
Navigation
:::

::: en-intro
 [Intro](#)
:::

::: en-intro
 [Families of cross-platform GUI components and their
production](#example-0)
:::

::: en-file
 buttons
:::

:::: en-inside
::: en-file
  [Button](#example-0--buttons-Button-java)
:::
::::

:::: en-inside
::: en-file
  [Mac­OSButton](#example-0--buttons-MacOSButton-java)
:::
::::

:::: en-inside
::: en-file
  [Windows­Button](#example-0--buttons-WindowsButton-java)
:::
::::

::: en-file
 checkboxes
:::

:::: en-inside
::: en-file
  [Checkbox](#example-0--checkboxes-Checkbox-java)
:::
::::

:::: en-inside
::: en-file
  [Mac­OSCheckbox](#example-0--checkboxes-MacOSCheckbox-java)
:::
::::

:::: en-inside
::: en-file
  [Windows­Checkbox](#example-0--checkboxes-WindowsCheckbox-java)
:::
::::

::: en-file
 factories
:::

:::: en-inside
::: en-file
  [GUIFactory](#example-0--factories-GUIFactory-java)
:::
::::

:::: en-inside
::: en-file
  [Mac­OSFactory](#example-0--factories-MacOSFactory-java)
:::
::::

:::: en-inside
::: en-file
  [Windows­Factory](#example-0--factories-WindowsFactory-java)
:::
::::

::: en-file
 app
:::

:::: en-inside
::: en-file
  [Application](#example-0--app-Application-java)
:::
::::

::: en-file
 [Demo](#example-0--Demo-java)
:::

::: en-file
 [Output­Demo](#example-0--OutputDemo-txt)
:::
::::::::::::::::::::::::::::::::

**Complexity:** []{.fa-stars}

**Popularity:** []{.fa-stars}

**Usage examples:** The Abstract Factory pattern is pretty common in
Java code. Many frameworks and libraries use it to provide a way to
extend and customize their standard components.

Here are some examples from core Java libraries:

-   [`javax.xml.parsers.DocumentBuilderFactory#newInstance()`](http://docs.oracle.com/javase/8/docs/api/javax/xml/parsers/DocumentBuilderFactory.html#newInstance--)

-   [`javax.xml.transform.TransformerFactory#newInstance()`](http://docs.oracle.com/javase/8/docs/api/javax/xml/transform/TransformerFactory.html#newInstance--)

-   [`javax.xml.xpath.XPathFactory#newInstance()`](http://docs.oracle.com/javase/8/docs/api/javax/xml/xpath/XPathFactory.html#newInstance--)

**Identification:** The pattern is easy to recognize by methods, which
return a factory object. Then, the factory is used for creating specific
sub-components.
:::::::::::::::::::::::::::::::::::

[]{#example-0}

## Families of cross-platform GUI components and their production {#example-0-title}

In this example, buttons and checkboxes will act as products. They have
two variants: macOS and Windows.

The abstract factory defines an interface for creating buttons and
checkboxes. There are two concrete factories, which return both products
in a single variant.

Client code works with factories and products using abstract interfaces.
It makes the same client code working with many product variants,
depending on the type of factory object.

### []{#example-0--buttons .anchor} **buttons:** First product hierarchy {#buttons-first-product-hierarchy .example-title}

#### []{#example-0--buttons-Button-java .anchor} **buttons/Button.java**

<figure class="code">
<pre class="code" lang="java"><code>package refactoring_guru.abstract_factory.example.buttons;

/**
 * Abstract Factory assumes that you have several families of products,
 * structured into separate class hierarchies (Button/Checkbox). All products of
 * the same family have the common interface.
 *
 * This is the common interface for buttons family.
 */
public interface Button {
    void paint();
}</code></pre>
</figure>

#### []{#example-0--buttons-MacOSButton-java .anchor} **buttons/MacOSButton.java**

<figure class="code">
<pre class="code" lang="java"><code>package refactoring_guru.abstract_factory.example.buttons;

/**
 * All products families have the same varieties (MacOS/Windows).
 *
 * This is a MacOS variant of a button.
 */
public class MacOSButton implements Button {

    @Override
    public void paint() {
        System.out.println(&quot;You have created MacOSButton.&quot;);
    }
}</code></pre>
</figure>

#### []{#example-0--buttons-WindowsButton-java .anchor} **buttons/WindowsButton.java**

<figure class="code">
<pre class="code" lang="java"><code>package refactoring_guru.abstract_factory.example.buttons;

/**
 * All products families have the same varieties (MacOS/Windows).
 *
 * This is another variant of a button.
 */
public class WindowsButton implements Button {

    @Override
    public void paint() {
        System.out.println(&quot;You have created WindowsButton.&quot;);
    }
}</code></pre>
</figure>

### []{#example-0--checkboxes .anchor} **checkboxes:** Second product hierarchy {#checkboxes-second-product-hierarchy .example-title}

#### []{#example-0--checkboxes-Checkbox-java .anchor} **checkboxes/Checkbox.java**

<figure class="code">
<pre class="code" lang="java"><code>package refactoring_guru.abstract_factory.example.checkboxes;

/**
 * Checkboxes is the second product family. It has the same variants as buttons.
 */
public interface Checkbox {
    void paint();
}</code></pre>
</figure>

#### []{#example-0--checkboxes-MacOSCheckbox-java .anchor} **checkboxes/MacOSCheckbox.java**

<figure class="code">
<pre class="code" lang="java"><code>package refactoring_guru.abstract_factory.example.checkboxes;

/**
 * All products families have the same varieties (MacOS/Windows).
 *
 * This is a variant of a checkbox.
 */
public class MacOSCheckbox implements Checkbox {

    @Override
    public void paint() {
        System.out.println(&quot;You have created MacOSCheckbox.&quot;);
    }
}</code></pre>
</figure>

#### []{#example-0--checkboxes-WindowsCheckbox-java .anchor} **checkboxes/WindowsCheckbox.java**

<figure class="code">
<pre class="code" lang="java"><code>package refactoring_guru.abstract_factory.example.checkboxes;

/**
 * All products families have the same varieties (MacOS/Windows).
 *
 * This is another variant of a checkbox.
 */
public class WindowsCheckbox implements Checkbox {

    @Override
    public void paint() {
        System.out.println(&quot;You have created WindowsCheckbox.&quot;);
    }
}</code></pre>
</figure>

### []{#example-0--factories .anchor} **factories** {#factories .example-title}

#### []{#example-0--factories-GUIFactory-java .anchor} **factories/GUIFactory.java:** Abstract factory

<figure class="code">
<pre class="code" lang="java"><code>package refactoring_guru.abstract_factory.example.factories;

import refactoring_guru.abstract_factory.example.buttons.Button;
import refactoring_guru.abstract_factory.example.checkboxes.Checkbox;

/**
 * Abstract factory knows about all (abstract) product types.
 */
public interface GUIFactory {
    Button createButton();
    Checkbox createCheckbox();
}</code></pre>
</figure>

#### []{#example-0--factories-MacOSFactory-java .anchor} **factories/MacOSFactory.java:** Concrete factory (macOS)

<figure class="code">
<pre class="code" lang="java"><code>package refactoring_guru.abstract_factory.example.factories;

import refactoring_guru.abstract_factory.example.buttons.Button;
import refactoring_guru.abstract_factory.example.buttons.MacOSButton;
import refactoring_guru.abstract_factory.example.checkboxes.Checkbox;
import refactoring_guru.abstract_factory.example.checkboxes.MacOSCheckbox;

/**
 * Each concrete factory extends basic factory and responsible for creating
 * products of a single variety.
 */
public class MacOSFactory implements GUIFactory {

    @Override
    public Button createButton() {
        return new MacOSButton();
    }

    @Override
    public Checkbox createCheckbox() {
        return new MacOSCheckbox();
    }
}</code></pre>
</figure>

#### []{#example-0--factories-WindowsFactory-java .anchor} **factories/WindowsFactory.java:** Concrete factory (Windows)

<figure class="code">
<pre class="code" lang="java"><code>package refactoring_guru.abstract_factory.example.factories;

import refactoring_guru.abstract_factory.example.buttons.Button;
import refactoring_guru.abstract_factory.example.buttons.WindowsButton;
import refactoring_guru.abstract_factory.example.checkboxes.Checkbox;
import refactoring_guru.abstract_factory.example.checkboxes.WindowsCheckbox;

/**
 * Each concrete factory extends basic factory and responsible for creating
 * products of a single variety.
 */
public class WindowsFactory implements GUIFactory {

    @Override
    public Button createButton() {
        return new WindowsButton();
    }

    @Override
    public Checkbox createCheckbox() {
        return new WindowsCheckbox();
    }
}</code></pre>
</figure>

### []{#example-0--app .anchor} **app** {#app .example-title}

#### []{#example-0--app-Application-java .anchor} **app/Application.java:** Client code

<figure class="code">
<pre class="code" lang="java"><code>package refactoring_guru.abstract_factory.example.app;

import refactoring_guru.abstract_factory.example.buttons.Button;
import refactoring_guru.abstract_factory.example.checkboxes.Checkbox;
import refactoring_guru.abstract_factory.example.factories.GUIFactory;

/**
 * Factory users don&#39;t care which concrete factory they use since they work with
 * factories and products through abstract interfaces.
 */
public class Application {
    private Button button;
    private Checkbox checkbox;

    public Application(GUIFactory factory) {
        button = factory.createButton();
        checkbox = factory.createCheckbox();
    }

    public void paint() {
        button.paint();
        checkbox.paint();
    }
}</code></pre>
</figure>

#### []{#example-0--Demo-java .anchor} **Demo.java:** App configuration

<figure class="code">
<pre class="code" lang="java"><code>package refactoring_guru.abstract_factory.example;

import refactoring_guru.abstract_factory.example.app.Application;
import refactoring_guru.abstract_factory.example.factories.GUIFactory;
import refactoring_guru.abstract_factory.example.factories.MacOSFactory;
import refactoring_guru.abstract_factory.example.factories.WindowsFactory;

/**
 * Demo class. Everything comes together here.
 */
public class Demo {

    /**
     * Application picks the factory type and creates it in run time (usually at
     * initialization stage), depending on the configuration or environment
     * variables.
     */
    private static Application configureApplication() {
        Application app;
        GUIFactory factory;
        String osName = System.getProperty(&quot;os.name&quot;).toLowerCase();
        if (osName.contains(&quot;mac&quot;)) {
            factory = new MacOSFactory();
        } else {
            factory = new WindowsFactory();
        }
        app = new Application(factory);
        return app;
    }

    public static void main(String[] args) {
        Application app = configureApplication();
        app.paint();
    }
}</code></pre>
</figure>

#### []{#example-0--OutputDemo-txt .anchor} **OutputDemo.txt:** Execution result

<figure class="code">
<pre class="code" lang="output"><code>You create WindowsButton.
You created WindowsCheckbox.</code></pre>
</figure>

::: next
#### Read next

[Builder in Java []{.fa
.fa-arrow-right}](../../builder/java/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Return

[[]{.fa .fa-arrow-left} Design Patterns in Java](../../java.html){.btn
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
Rust](../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Abstract Factory in Rust"){.prog-lang-link}
[![Abstract Factory in
Swift](../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Abstract Factory in Swift"){.prog-lang-link}
[![Abstract Factory in
TypeScript](../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Abstract Factory in TypeScript"){.prog-lang-link}
:::::::::::::::::::::::::::::::::::::::::

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
:::::::::::::::::::::::::::::::::::::::::::::::
::::::::::::::::::::::::::::::::::::::::::::::::
