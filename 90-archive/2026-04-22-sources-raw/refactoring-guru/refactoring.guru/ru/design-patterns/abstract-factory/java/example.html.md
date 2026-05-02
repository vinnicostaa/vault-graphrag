:::::::::::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::::::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [Паттерны
проектирования](../../../design-patterns.html){.type} / [Абстрактная
фабрика](../../abstract-factory.html){.type} /
[Java](../../java.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Абстрактная
фабрика](../../../../images/patterns/cards/abstract-factory-minid1c5.png?id=4c3927c446313a38ce77dfee38111e27){srcset="/images/patterns/cards/abstract-factory-mini-2x.png?id=22236aaa65ff52cbde1c713216d52c1f 2x"}
:::

# **Абстрактная фабрика** на Java {#абстрактная-фабрика-на-java .pattern-example-title-block-title}
::::

::::::::::::::::::::::::::::::::::: pattern-example-body
::: pattern-example-brief
**Абстрактная фабрика** --- это порождающий паттерн проектирования,
который решает проблему создания целых семейств связанных продуктов, без
указания конкретных классов продуктов.

Абстрактная фабрика задаёт интерфейс создания всех доступных типов
продуктов, а каждая конкретная реализация фабрики порождает продукты
одной из вариаций. Клиентский код вызывает методы фабрики для получения
продуктов, вместо самостоятельного создания с помощью оператора `new`.
При этом фабрика сама следит за тем, чтобы создать продукт нужной
вариации.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Подробней о паттерне Абстрактная фабрика
](../../abstract-factory.html){.btn .btn-lg .btn-primary}
:::

:::::::::::::::::::::::::::::::: {.sidebar-navigation .with-scroll}
::: en-title
Навигация
:::

::: en-intro
 [Интро](#)
:::

::: en-intro
 [Производство семейств кросс-платформенных элементов GUI](#example-0)
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

**Сложность:** []{.fa-stars}

**Популярность:** []{.fa-stars}

**Применимость:** Паттерн можно часто встретить в Java-коде, особенно
там, где требуется создание семейств продуктов (например, внутри
фреймворков).

Примеры Абстрактной фабрики в стандартных библиотеках Java:

-   [`javax.xml.parsers.DocumentBuilderFactory#newInstance()`](http://docs.oracle.com/javase/8/docs/api/javax/xml/parsers/DocumentBuilderFactory.html#newInstance--)

-   [`javax.xml.transform.TransformerFactory#newInstance()`](http://docs.oracle.com/javase/8/docs/api/javax/xml/transform/TransformerFactory.html#newInstance--)

-   [`javax.xml.xpath.XPathFactory#newInstance()`](http://docs.oracle.com/javase/8/docs/api/javax/xml/xpath/XPathFactory.html#newInstance--)

**Признаки применения паттерна:** Паттерн можно определить по методам,
возвращающим фабрику, которая, в свою очередь, используется для создания
конкретных продуктов, возвращая их через абстрактные типы или
интерфейсы.
:::::::::::::::::::::::::::::::::::

[]{#example-0}

## Производство семейств кросс-платформенных элементов GUI {#example-0-title}

В этом примере в роли двух семейств продуктов выступают кнопки и
чекбоксы. Оба семейства продуктов имеют одинаковые вариации: для работы
под MacOS и Windows.

Абстрактная фабрика задаёт интерфейс создания продуктов всех семейств.
Конкретные фабрики создают различные продукты одной вариации (MacOS или
Windows).

Клиенты фабрики работают как с фабрикой, так и с продуктами только через
абстрактные интерфейсы. Благодаря этому, один и тот же клиентский код
может работать с различными фабриками и продуктами.

### []{#example-0--buttons .anchor} **buttons:** Первая иерархия продуктов (кнопки) {#buttons-первая-иерархия-продуктов-кнопки .example-title}

#### []{#example-0--buttons-Button-java .anchor} **buttons/Button.java**

<figure class="code">
<pre class="code" lang="java"><code>package refactoring_guru.abstract_factory.example.buttons;

/**
 * Паттерн предполагает, что у вас есть несколько семейств продуктов,
 * находящихся в отдельных иерархиях классов (Button/Checkbox). Продукты одного
 * семейства должны иметь общий интерфейс.
 *
 * Это — общий интерфейс для семейства продуктов кнопок.
 */
public interface Button {
    void paint();
}</code></pre>
</figure>

#### []{#example-0--buttons-MacOSButton-java .anchor} **buttons/MacOSButton.java**

<figure class="code">
<pre class="code" lang="java"><code>package refactoring_guru.abstract_factory.example.buttons;

/**
 * Все семейства продуктов имеют одни и те же вариации (MacOS/Windows).
 *
 * Это вариант кнопки под MacOS.
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
 * Все семейства продуктов имеют одни и те же вариации (MacOS/Windows).
 *
 * Это вариант кнопки под Windows.
 */
public class WindowsButton implements Button {

    @Override
    public void paint() {
        System.out.println(&quot;You have created WindowsButton.&quot;);
    }
}</code></pre>
</figure>

### []{#example-0--checkboxes .anchor} **checkboxes:** Вторая иерархия продуктов (чекбоксы) {#checkboxes-вторая-иерархия-продуктов-чекбоксы .example-title}

#### []{#example-0--checkboxes-Checkbox-java .anchor} **checkboxes/Checkbox.java**

<figure class="code">
<pre class="code" lang="java"><code>package refactoring_guru.abstract_factory.example.checkboxes;

/**
 * Чекбоксы — это второе семейство продуктов. Оно имеет те же вариации, что и
 * кнопки.
 */
public interface Checkbox {
    void paint();
}</code></pre>
</figure>

#### []{#example-0--checkboxes-MacOSCheckbox-java .anchor} **checkboxes/MacOSCheckbox.java**

<figure class="code">
<pre class="code" lang="java"><code>package refactoring_guru.abstract_factory.example.checkboxes;

/**
 * Все семейства продуктов имеют одинаковые вариации (MacOS/Windows).
 *
 * Вариация чекбокса под MacOS.
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
 * Все семейства продуктов имеют одинаковые вариации (MacOS/Windows).
 *
 * Вариация чекбокса под Windows.
 */
public class WindowsCheckbox implements Checkbox {

    @Override
    public void paint() {
        System.out.println(&quot;You have created WindowsCheckbox.&quot;);
    }
}</code></pre>
</figure>

### []{#example-0--factories .anchor} **factories** {#factories .example-title}

#### []{#example-0--factories-GUIFactory-java .anchor} **factories/GUIFactory.java:** Абстрактная фабрика

<figure class="code">
<pre class="code" lang="java"><code>package refactoring_guru.abstract_factory.example.factories;

import refactoring_guru.abstract_factory.example.buttons.Button;
import refactoring_guru.abstract_factory.example.checkboxes.Checkbox;

/**
 * Абстрактная фабрика знает обо всех (абстрактных) типах продуктов.
 */
public interface GUIFactory {
    Button createButton();
    Checkbox createCheckbox();
}</code></pre>
</figure>

#### []{#example-0--factories-MacOSFactory-java .anchor} **factories/MacOSFactory.java:** Конкретная фабрика (MacOS)

<figure class="code">
<pre class="code" lang="java"><code>package refactoring_guru.abstract_factory.example.factories;

import refactoring_guru.abstract_factory.example.buttons.Button;
import refactoring_guru.abstract_factory.example.buttons.MacOSButton;
import refactoring_guru.abstract_factory.example.checkboxes.Checkbox;
import refactoring_guru.abstract_factory.example.checkboxes.MacOSCheckbox;

/**
 * Каждая конкретная фабрика знает и создаёт только продукты своей вариации.
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

#### []{#example-0--factories-WindowsFactory-java .anchor} **factories/WindowsFactory.java:** Конкретная фабрика (Windows)

<figure class="code">
<pre class="code" lang="java"><code>package refactoring_guru.abstract_factory.example.factories;

import refactoring_guru.abstract_factory.example.buttons.Button;
import refactoring_guru.abstract_factory.example.buttons.WindowsButton;
import refactoring_guru.abstract_factory.example.checkboxes.Checkbox;
import refactoring_guru.abstract_factory.example.checkboxes.WindowsCheckbox;

/**
 * Каждая конкретная фабрика знает и создаёт только продукты своей вариации.
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

#### []{#example-0--app-Application-java .anchor} **app/Application.java:** Клиентский код

<figure class="code">
<pre class="code" lang="java"><code>package refactoring_guru.abstract_factory.example.app;

import refactoring_guru.abstract_factory.example.buttons.Button;
import refactoring_guru.abstract_factory.example.checkboxes.Checkbox;
import refactoring_guru.abstract_factory.example.factories.GUIFactory;

/**
 * Код, использующий фабрику, не волнует с какой конкретно фабрикой он работает.
 * Все получатели продуктов работают с продуктами через абстрактный интерфейс.
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

#### []{#example-0--Demo-java .anchor} **Demo.java:** Конфигуратор приложения

<figure class="code">
<pre class="code" lang="java"><code>package refactoring_guru.abstract_factory.example;

import refactoring_guru.abstract_factory.example.app.Application;
import refactoring_guru.abstract_factory.example.factories.GUIFactory;
import refactoring_guru.abstract_factory.example.factories.MacOSFactory;
import refactoring_guru.abstract_factory.example.factories.WindowsFactory;

/**
 * Демо-класс. Здесь всё сводится воедино.
 */
public class Demo {

    /**
     * Приложение выбирает тип и создаёт конкретные фабрики динамически исходя
     * из конфигурации или окружения.
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

#### []{#example-0--OutputDemo-txt .anchor} **OutputDemo.txt:** Результат выполнения

<figure class="code">
<pre class="code" lang="output"><code>You create WindowsButton.
You created WindowsCheckbox.</code></pre>
</figure>

::: next
#### Читаем дальше

[Строитель на Java []{.fa
.fa-arrow-right}](../../builder/java/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Вернуться назад

[[]{.fa .fa-arrow-left} Паттерны проектирования на
Java](../../java.html){.btn .btn-default rel="prev"}
:::

## **Абстрактная фабрика** на других языках программирования

[![Абстрактная фабрика на
C#](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Абстрактная фабрика на C#"){.prog-lang-link}
[![Абстрактная фабрика на
C++](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Абстрактная фабрика на C++"){.prog-lang-link}
[![Абстрактная фабрика на
Go](../../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Абстрактная фабрика на Go"){.prog-lang-link}
[![Абстрактная фабрика на
PHP](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Абстрактная фабрика на PHP"){.prog-lang-link}
[![Абстрактная фабрика на
Python](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Абстрактная фабрика на Python"){.prog-lang-link}
[![Абстрактная фабрика на
Ruby](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Абстрактная фабрика на Ruby"){.prog-lang-link}
[![Абстрактная фабрика на
Rust](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Абстрактная фабрика на Rust"){.prog-lang-link}
[![Абстрактная фабрика на
Swift](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Абстрактная фабрика на Swift"){.prog-lang-link}
[![Абстрактная фабрика на
TypeScript](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Абстрактная фабрика на TypeScript"){.prog-lang-link}
:::::::::::::::::::::::::::::::::::::::::

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
:::::::::::::::::::::::::::::::::::::::::::::::
::::::::::::::::::::::::::::::::::::::::::::::::
