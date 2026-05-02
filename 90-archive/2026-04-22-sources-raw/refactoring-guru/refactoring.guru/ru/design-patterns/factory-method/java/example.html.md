::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [Паттерны
проектирования](../../../design-patterns.html){.type} / [Фабричный
метод](../../factory-method.html){.type} /
[Java](../../java.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Фабричный
метод](../../../../images/patterns/cards/factory-method-minid7f6.png?id=72619e9527893374b98a5913779ac167){srcset="/images/patterns/cards/factory-method-mini-2x.png?id=fa9d4a8d61a67cc3822e52b9daf69dad 2x"}
:::

# **Фабричный метод** на Java {#фабричный-метод-на-java .pattern-example-title-block-title}
::::

:::::::::::::::::::::::::: pattern-example-body
::: pattern-example-brief
**Фабричный метод** --- это порождающий паттерн проектирования, который
решает проблему создания различных продуктов, без указания конкретных
классов продуктов.

Фабричный метод задаёт метод, который следует использовать вместо вызова
оператора `new` для создания объектов-продуктов. Подклассы могут
переопределить этот метод, чтобы изменять тип создаваемых продуктов.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Подробней о паттерне Фабричный метод ](../../factory-method.html){.btn
.btn-lg .btn-primary}
:::

::::::::::::::::::::::: {.sidebar-navigation .with-scroll}
::: en-title
Навигация
:::

::: en-intro
 [Интро](#)
:::

::: en-intro
 [Производство кросс-платформенных элементов GUI](#example-0)
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
  [Html­Button](#example-0--buttons-HtmlButton-java)
:::
::::

:::: en-inside
::: en-file
  [Windows­Button](#example-0--buttons-WindowsButton-java)
:::
::::

::: en-file
 factory
:::

:::: en-inside
::: en-file
  [Dialog](#example-0--factory-Dialog-java)
:::
::::

:::: en-inside
::: en-file
  [Html­Dialog](#example-0--factory-HtmlDialog-java)
:::
::::

:::: en-inside
::: en-file
  [Windows­Dialog](#example-0--factory-WindowsDialog-java)
:::
::::

::: en-file
 [Demo](#example-0--Demo-java)
:::

::: en-file
 [Output­Demo](#example-0--OutputDemo-txt)
:::

::: en-file
 [Output­Demo](#example-0--OutputDemo-png)
:::
:::::::::::::::::::::::

**Сложность:** []{.fa-stars}

**Популярность:** []{.fa-stars}

**Применимость:** Паттерн можно часто встретить в любом Java-коде, где
требуется гибкость при создании продуктов.

Паттерн широко используется в стандартных библиотеках Java:

-   [`java.util.Calendar#getInstance()`](http://docs.oracle.com/javase/8/docs/api/java/util/Calendar.html#getInstance--)
-   [`java.util.ResourceBundle#getBundle()`](http://docs.oracle.com/javase/8/docs/api/java/util/ResourceBundle.html#getBundle-java.lang.String-)
-   [`java.text.NumberFormat#getInstance()`](http://docs.oracle.com/javase/8/docs/api/java/text/NumberFormat.html#getInstance--)
-   [`java.nio.charset.Charset#forName()`](http://docs.oracle.com/javase/8/docs/api/java/nio/charset/Charset.html#forName-java.lang.String-)
-   [`java.net.URLStreamHandlerFactory#createURLStreamHandler(String)`](http://docs.oracle.com/javase/8/docs/api/java/net/URLStreamHandlerFactory.html)
    (Возвращает разные объекты-одиночки, в зависимости от протокола)
-   [`java.util.EnumSet#of()`](https://docs.oracle.com/javase/8/docs/api/java/util/EnumSet.html#of(E))
-   [`javax.xml.bind.JAXBContext#createMarshaller()`](https://docs.oracle.com/javase/8/docs/api/javax/xml/bind/JAXBContext.html#createMarshaller--)
    и другие похожие методы.

**Признаки применения паттерна:** Фабричный метод можно определить по
создающим методам, которые возвращают объекты продуктов через
абстрактные типы или интерфейсы. Это позволяет переопределять типы
создаваемых продуктов в подклассах.
::::::::::::::::::::::::::

[]{#example-0}

## Производство кросс-платформенных элементов GUI {#example-0-title}

В этом примере в роли продуктов выступают кнопки, а в роли создателя ---
диалог.

Разным типам диалогов соответствуют свои собственные типы элементов.
Поэтому для каждого типа диалога мы создаём свой подкласс и
переопределяем в нём фабричный метод.

Каждый конкретный диалог будет порождать те кнопки, которые к нему
подходят. При этом базовый код диалогов не сломается, так как он
работает с продуктами только через их общий интерфейс.

### []{#example-0--buttons .anchor} **buttons** {#buttons .example-title}

#### []{#example-0--buttons-Button-java .anchor} **buttons/Button.java:** Общий интерфейс кнопок

<figure class="code">
<pre class="code" lang="java"><code>package refactoring_guru.factory_method.example.buttons;

/**
 * Общий интерфейс для всех продуктов.
 */
public interface Button {
    void render();
    void onClick();
}</code></pre>
</figure>

#### []{#example-0--buttons-HtmlButton-java .anchor} **buttons/HtmlButton.java:** Конкретный класс кнопок

<figure class="code">
<pre class="code" lang="java"><code>package refactoring_guru.factory_method.example.buttons;

/**
 * Реализация HTML кнопок.
 */
public class HtmlButton implements Button {

    public void render() {
        System.out.println(&quot;&lt;button&gt;Test Button&lt;/button&gt;&quot;);
        onClick();
    }

    public void onClick() {
        System.out.println(&quot;Click! Button says - &#39;Hello World!&#39;&quot;);
    }
}</code></pre>
</figure>

#### []{#example-0--buttons-WindowsButton-java .anchor} **buttons/WindowsButton.java:** Ещё один класс кнопок

<figure class="code">
<pre class="code" lang="java"><code>package refactoring_guru.factory_method.example.buttons;

import javax.swing.*;
import java.awt.*;
import java.awt.event.ActionEvent;
import java.awt.event.ActionListener;

/**
 * Реализация нативных кнопок операционной системы.
 */
public class WindowsButton implements Button {
    JPanel panel = new JPanel();
    JFrame frame = new JFrame();
    JButton button;

    public void render() {
        frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
        JLabel label = new JLabel(&quot;Hello World!&quot;);
        label.setOpaque(true);
        label.setBackground(new Color(235, 233, 126));
        label.setFont(new Font(&quot;Dialog&quot;, Font.BOLD, 44));
        label.setHorizontalAlignment(SwingConstants.CENTER);
        panel.setLayout(new FlowLayout(FlowLayout.CENTER));
        frame.getContentPane().add(panel);
        panel.add(label);
        onClick();
        panel.add(button);

        frame.setSize(320, 200);
        frame.setVisible(true);
        onClick();
    }

    public void onClick() {
        button = new JButton(&quot;Exit&quot;);
        button.addActionListener(new ActionListener() {
            public void actionPerformed(ActionEvent e) {
                frame.setVisible(false);
                System.exit(0);
            }
        });
    }
}</code></pre>
</figure>

### []{#example-0--factory .anchor} **factory** {#factory .example-title}

#### []{#example-0--factory-Dialog-java .anchor} **factory/Dialog.java:** Базовый диалог

<figure class="code">
<pre class="code" lang="java"><code>package refactoring_guru.factory_method.example.factory;

import refactoring_guru.factory_method.example.buttons.Button;

/**
 * Базовый класс фабрики. Заметьте, что &quot;фабрика&quot; – это всего лишь
 * дополнительная роль для класса. Он уже имеет какую-то бизнес-логику, в
 * которой требуется создание разнообразных продуктов.
 */
public abstract class Dialog {

    public void renderWindow() {
        // ... остальной код диалога ...

        Button okButton = createButton();
        okButton.render();
    }

    /**
     * Подклассы будут переопределять этот метод, чтобы создавать конкретные
     * объекты продуктов, разные для каждой фабрики.
     */
    public abstract Button createButton();
}</code></pre>
</figure>

#### []{#example-0--factory-HtmlDialog-java .anchor} **factory/HtmlDialog.java:** Конкретный класс диалогов

<figure class="code">
<pre class="code" lang="java"><code>package refactoring_guru.factory_method.example.factory;

import refactoring_guru.factory_method.example.buttons.Button;
import refactoring_guru.factory_method.example.buttons.HtmlButton;

/**
 * HTML-диалог.
 */
public class HtmlDialog extends Dialog {

    @Override
    public Button createButton() {
        return new HtmlButton();
    }
}</code></pre>
</figure>

#### []{#example-0--factory-WindowsDialog-java .anchor} **factory/WindowsDialog.java:** Ещё один класс диалогов

<figure class="code">
<pre class="code" lang="java"><code>package refactoring_guru.factory_method.example.factory;

import refactoring_guru.factory_method.example.buttons.Button;
import refactoring_guru.factory_method.example.buttons.WindowsButton;

/**
 * Диалог на элементах операционной системы.
 */
public class WindowsDialog extends Dialog {

    @Override
    public Button createButton() {
        return new WindowsButton();
    }
}</code></pre>
</figure>

#### []{#example-0--Demo-java .anchor} **Demo.java:** Клиентский код

<figure class="code">
<pre class="code" lang="java"><code>package refactoring_guru.factory_method.example;

import refactoring_guru.factory_method.example.factory.Dialog;
import refactoring_guru.factory_method.example.factory.HtmlDialog;
import refactoring_guru.factory_method.example.factory.WindowsDialog;

/**
 * Демо-класс. Здесь всё сводится воедино.
 */
public class Demo {
    private static Dialog dialog;

    public static void main(String[] args) {
        configure();
        runBusinessLogic();
    }

    /**
     * Приложение создаёт определённую фабрику в зависимости от конфигурации или
     * окружения.
     */
    static void configure() {
        if (System.getProperty(&quot;os.name&quot;).equals(&quot;Windows 10&quot;)) {
            dialog = new WindowsDialog();
        } else {
            dialog = new HtmlDialog();
        }
    }

    /**
     * Весь остальной клиентский код работает с фабрикой и продуктами только
     * через общий интерфейс, поэтому для него неважно какая фабрика была
     * создана.
     */
    static void runBusinessLogic() {
        dialog.renderWindow();
    }
}</code></pre>
</figure>

#### []{#example-0--OutputDemo-txt .anchor} **OutputDemo.txt:** Результат с фабрикой HtmlDialog

<figure class="code">
<pre class="code" lang="output"><code>&lt;button&gt;Test Button&lt;/button&gt;
Click! Button says - &#39;Hello World!&#39;</code></pre>
</figure>

#### []{#example-0--OutputDemo-png .anchor} **OutputDemo.png:** Результат с фабрикой WindowsDialog

![](../../../../images/patterns/examples/java/factory-method/OutputDemof025.png?id=36afce413161f6650321896d3023fb65)

::: next
#### Читаем дальше

[Прототип на Java []{.fa
.fa-arrow-right}](../../prototype/java/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Вернуться назад

[[]{.fa .fa-arrow-left} Строитель на
Java](../../builder/java/example.html){.btn .btn-default rel="prev"}
:::

## **Фабричный метод** на других языках программирования

[![Фабричный метод на
C#](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Фабричный метод на C#"){.prog-lang-link}
[![Фабричный метод на
C++](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Фабричный метод на C++"){.prog-lang-link}
[![Фабричный метод на
Go](../../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Фабричный метод на Go"){.prog-lang-link}
[![Фабричный метод на
PHP](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Фабричный метод на PHP"){.prog-lang-link}
[![Фабричный метод на
Python](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Фабричный метод на Python"){.prog-lang-link}
[![Фабричный метод на
Ruby](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Фабричный метод на Ruby"){.prog-lang-link}
[![Фабричный метод на
Rust](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Фабричный метод на Rust"){.prog-lang-link}
[![Фабричный метод на
Swift](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Фабричный метод на Swift"){.prog-lang-link}
[![Фабричный метод на
TypeScript](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Фабричный метод на TypeScript"){.prog-lang-link}
::::::::::::::::::::::::::::::::

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
::::::::::::::::::::::::::::::::::::::
:::::::::::::::::::::::::::::::::::::::
