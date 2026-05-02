:::::::::::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::::::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [Патерни
проектування](../../../design-patterns.html){.type} / [Абстрактна
фабрика](../../abstract-factory.html){.type} /
[Java](../../java.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Абстрактна
фабрика](../../../../images/patterns/cards/abstract-factory-minid1c5.png?id=4c3927c446313a38ce77dfee38111e27){srcset="/images/patterns/cards/abstract-factory-mini-2x.png?id=22236aaa65ff52cbde1c713216d52c1f 2x"}
:::

# **Абстрактна фабрика** на Java {#абстрактна-фабрика-на-java .pattern-example-title-block-title}
::::

::::::::::::::::::::::::::::::::::: pattern-example-body
::: pattern-example-brief
**Абстрактна фабрика** --- це породжуючий патерн проектування, який
вирішує проблему створення цілих сімейств пов'язаних продуктів, без
прив'язки коду до конкретних класів продуктів.

Абстрактна фабрика задає інтерфейс створення всіх доступних типів
продуктів, а кожна конкретна реалізація фабрики породжує продукти однієї
з варіацій. Клієнтський код викликає методи фабрики для отримання
продуктів, замість самостійного створювання їх за допомогою оператора
`new`. При цьому, фабрика сама стежить за тим, щоб створюваний продукт
був потрібної варіації.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Детальніше про Абстрактна фабрика ](../../abstract-factory.html){.btn
.btn-lg .btn-primary}
:::

:::::::::::::::::::::::::::::::: {.sidebar-navigation .with-scroll}
::: en-title
Навігація
:::

::: en-intro
 [Інтро](#)
:::

::: en-intro
 [Виробництво сімейств крос-платформних елементів GUI](#example-0)
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

**Складність:** []{.fa-stars}

**Популярність:** []{.fa-stars}

**Застосування:** Патерн можна часто зустріти в Java-коді, особливо там,
де потрібно породжувати сімейства класів, не прив'язуючи до них свій код
(наприклад, у фреймворках).

Приклади Абстрактної фабрики в стандартних бібліотеках Java:

-   [`javax.xml.parsers.DocumentBuilderFactory#newInstance()`](http://docs.oracle.com/javase/8/docs/api/javax/xml/parsers/DocumentBuilderFactory.html#newInstance--)

-   [`javax.xml.transform.TransformerFactory#newInstance()`](http://docs.oracle.com/javase/8/docs/api/javax/xml/transform/TransformerFactory.html#newInstance--)

-   [`javax.xml.xpath.XPathFactory#newInstance()`](http://docs.oracle.com/javase/8/docs/api/javax/xml/xpath/XPathFactory.html#newInstance--)

**Ознаки застосування патерна:** Патерн можна визначити за методами, що
повертають фабрику, яка, в свою чергу, використовується для створення
конкретних продуктів, повертаючи їх через абстрактні типи або
інтерфейси.
:::::::::::::::::::::::::::::::::::

[]{#example-0}

## Виробництво сімейств крос-платформних елементів GUI {#example-0-title}

У цьому прикладі в ролі двох сімейств продуктів виступають кнопки та
чекбокси. Обидва сімейства продуктів мають однакові варіації: для роботи
під MacOS та Windows.

Абстрактна фабрика задає інтерфейс створення продуктів усіх сімейств.
Конкретні фабрики створюють різні продукти однієї варіації (MacOS або
Windows).

Клієнти фабрики працюють як з фабрикою, так і з продуктами тільки через
абстрактні інтерфейси. Завдяки цьому, один і той же клієнтський код може
працювати з різними фабриками та продуктами.

### []{#example-0--buttons .anchor} **buttons:** Перша ієрархія продуктів (кнопки) {#buttons-перша-ієрархія-продуктів-кнопки .example-title}

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

### []{#example-0--checkboxes .anchor} **checkboxes:** Друга ієрархія продуктів (чекбокси) {#checkboxes-друга-ієрархія-продуктів-чекбокси .example-title}

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

#### []{#example-0--factories-GUIFactory-java .anchor} **factories/GUIFactory.java:** Абстрактна фабрика

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

#### []{#example-0--factories-MacOSFactory-java .anchor} **factories/MacOSFactory.java:** Конкретна фабрика (MacOS)

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

#### []{#example-0--factories-WindowsFactory-java .anchor} **factories/WindowsFactory.java:** Конкретна фабрика (Windows)

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

#### []{#example-0--app-Application-java .anchor} **app/Application.java:** Клієнтський код

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

#### []{#example-0--Demo-java .anchor} **Demo.java:** Конфігуратор програми

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

#### []{#example-0--OutputDemo-txt .anchor} **OutputDemo.txt:** Результат виконання

<figure class="code">
<pre class="code" lang="output"><code>You create WindowsButton.
You created WindowsCheckbox.</code></pre>
</figure>

::: next
#### Читаємо далі

[Будівельник на Java []{.fa
.fa-arrow-right}](../../builder/java/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Повернутись назад

[[]{.fa .fa-arrow-left} Патерни проектування на
Java](../../java.html){.btn .btn-default rel="prev"}
:::

## **Абстрактна фабрика** іншими мовами програмування

[![Абстрактна фабрика на
C#](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Абстрактна фабрика на C#"){.prog-lang-link}
[![Абстрактна фабрика на
C++](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Абстрактна фабрика на C++"){.prog-lang-link}
[![Абстрактна фабрика на
Go](../../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Абстрактна фабрика на Go"){.prog-lang-link}
[![Абстрактна фабрика на
PHP](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Абстрактна фабрика на PHP"){.prog-lang-link}
[![Абстрактна фабрика на
Python](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Абстрактна фабрика на Python"){.prog-lang-link}
[![Абстрактна фабрика на
Ruby](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Абстрактна фабрика на Ruby"){.prog-lang-link}
[![Абстрактна фабрика на
Rust](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Абстрактна фабрика на Rust"){.prog-lang-link}
[![Абстрактна фабрика на
Swift](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Абстрактна фабрика на Swift"){.prog-lang-link}
[![Абстрактна фабрика на
TypeScript](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Абстрактна фабрика на TypeScript"){.prog-lang-link}
:::::::::::::::::::::::::::::::::::::::::

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
:::::::::::::::::::::::::::::::::::::::::::::::
::::::::::::::::::::::::::::::::::::::::::::::::
