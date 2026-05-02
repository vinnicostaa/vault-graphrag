::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [Patrones de
diseño](../../../design-patterns.html){.type} / [Factory
Method](../../factory-method.html){.type} /
[Java](../../java.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Factory
Method](../../../../images/patterns/cards/factory-method-minid7f6.png?id=72619e9527893374b98a5913779ac167){srcset="/images/patterns/cards/factory-method-mini-2x.png?id=fa9d4a8d61a67cc3822e52b9daf69dad 2x"}
:::

# **Factory Method** en Java {#factory-method-en-java .pattern-example-title-block-title}
::::

:::::::::::::::::::::::::: pattern-example-body
::: pattern-example-brief
**Factory method** es un patrón de diseño creacional que resuelve el
problema de crear objetos de producto sin especificar sus clases
concretas.

El patrón Factory Method define un método que debe utilizarse para crear
objetos, en lugar de una llamada directa al constructor (operador
`new`). Las subclases pueden sobrescribir este método para cambiar las
clases de los objetos que se crearán.

> Si no sabes la diferencia entre varios patrones y conceptos de la
> fábrica, lee nuestra [Comparación de
> fábricas](../../factory-comparison.html).
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Aprende más sobre el patrón Factory Method
](../../factory-method.html){.btn .btn-lg .btn-primary}
:::

::::::::::::::::::::::: {.sidebar-navigation .with-scroll}
::: en-title
Navegación
:::

::: en-intro
 [Intro](#)
:::

::: en-intro
 [Producción de elementos GUI multiplataforma](#example-0)
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

**Complejidad:** []{.fa-stars}

**Popularidad:** []{.fa-stars}

**Ejemplos de uso:** El patrón Factory Method se utiliza mucho en el
código Java. Resulta muy útil cuando necesitas proporcionar un alto
nivel de flexibilidad a tu código.

El patrón está presente en las principales bibliotecas de Java:

-   [`java.util.Calendar#getInstance()`](http://docs.oracle.com/javase/8/docs/api/java/util/Calendar.html#getInstance--)
-   [`java.util.ResourceBundle#getBundle()`](http://docs.oracle.com/javase/8/docs/api/java/util/ResourceBundle.html#getBundle-java.lang.String-)
-   [`java.text.NumberFormat#getInstance()`](http://docs.oracle.com/javase/8/docs/api/java/text/NumberFormat.html#getInstance--)
-   [`java.nio.charset.Charset#forName()`](http://docs.oracle.com/javase/8/docs/api/java/nio/charset/Charset.html#forName-java.lang.String-)
-   [`java.net.URLStreamHandlerFactory#createURLStreamHandler(String)`](http://docs.oracle.com/javase/8/docs/api/java/net/URLStreamHandlerFactory.html)
    (Devuelve distintos objetos singleton, dependiendo de un protocolo).
-   [`java.util.EnumSet#of()`](https://docs.oracle.com/javase/8/docs/api/java/util/EnumSet.html#of(E))
-   [`javax.xml.bind.JAXBContext#createMarshaller()`](https://docs.oracle.com/javase/8/docs/api/javax/xml/bind/JAXBContext.html#createMarshaller--)
    y otros métodos similares.

**Identificación:** Los métodos fábrica pueden ser reconocidos por
métodos de creación, que crean objetos a partir de clases concretas,
pero los devuelven como objetos del tipo abstracto o interfaz.
::::::::::::::::::::::::::

[]{#example-0}

## Producción de elementos GUI multiplataforma {#example-0-title}

En este ejemplo, Buttons (Botones) juega el papel de producto y los
diálogos actúan como creadores.

Los distintos tipos de diálogos requieren sus propios tipos de
elementos. Por eso creamos una subclase para cada tipo de diálogo y
sobrescribimos sus métodos fábrica.

Ahora, cada tipo de diálogo instanciará clases de botón. El diálogo base
trabaja con productos utilizando su interfaz común, por eso su código
sigue siendo funcional tras todos los cambios.

### []{#example-0--buttons .anchor} **buttons** {#buttons .example-title}

#### []{#example-0--buttons-Button-java .anchor} **buttons/Button.java:** Interfaz común de producto

<figure class="code">
<pre class="code" lang="java"><code>package refactoring_guru.factory_method.example.buttons;

/**
 * Common interface for all buttons.
 */
public interface Button {
    void render();
    void onClick();
}</code></pre>
</figure>

#### []{#example-0--buttons-HtmlButton-java .anchor} **buttons/HtmlButton.java:** Producto concreto

<figure class="code">
<pre class="code" lang="java"><code>package refactoring_guru.factory_method.example.buttons;

/**
 * HTML button implementation.
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

#### []{#example-0--buttons-WindowsButton-java .anchor} **buttons/WindowsButton.java:** Otro producto concreto

<figure class="code">
<pre class="code" lang="java"><code>package refactoring_guru.factory_method.example.buttons;

import javax.swing.*;
import java.awt.*;
import java.awt.event.ActionEvent;
import java.awt.event.ActionListener;

/**
 * Windows button implementation.
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

#### []{#example-0--factory-Dialog-java .anchor} **factory/Dialog.java:** Creador base

<figure class="code">
<pre class="code" lang="java"><code>package refactoring_guru.factory_method.example.factory;

import refactoring_guru.factory_method.example.buttons.Button;

/**
 * Base factory class. Note that &quot;factory&quot; is merely a role for the class. It
 * should have some core business logic which needs different products to be
 * created.
 */
public abstract class Dialog {

    public void renderWindow() {
        // ... other code ...

        Button okButton = createButton();
        okButton.render();
    }

    /**
     * Subclasses will override this method in order to create specific button
     * objects.
     */
    public abstract Button createButton();
}</code></pre>
</figure>

#### []{#example-0--factory-HtmlDialog-java .anchor} **factory/HtmlDialog.java:** Creador concreto

<figure class="code">
<pre class="code" lang="java"><code>package refactoring_guru.factory_method.example.factory;

import refactoring_guru.factory_method.example.buttons.Button;
import refactoring_guru.factory_method.example.buttons.HtmlButton;

/**
 * HTML Dialog will produce HTML buttons.
 */
public class HtmlDialog extends Dialog {

    @Override
    public Button createButton() {
        return new HtmlButton();
    }
}</code></pre>
</figure>

#### []{#example-0--factory-WindowsDialog-java .anchor} **factory/WindowsDialog.java:** Otro creador concreto

<figure class="code">
<pre class="code" lang="java"><code>package refactoring_guru.factory_method.example.factory;

import refactoring_guru.factory_method.example.buttons.Button;
import refactoring_guru.factory_method.example.buttons.WindowsButton;

/**
 * Windows Dialog will produce Windows buttons.
 */
public class WindowsDialog extends Dialog {

    @Override
    public Button createButton() {
        return new WindowsButton();
    }
}</code></pre>
</figure>

#### []{#example-0--Demo-java .anchor} **Demo.java:** Código cliente

<figure class="code">
<pre class="code" lang="java"><code>package refactoring_guru.factory_method.example;

import refactoring_guru.factory_method.example.factory.Dialog;
import refactoring_guru.factory_method.example.factory.HtmlDialog;
import refactoring_guru.factory_method.example.factory.WindowsDialog;

/**
 * Demo class. Everything comes together here.
 */
public class Demo {
    private static Dialog dialog;

    public static void main(String[] args) {
        configure();
        runBusinessLogic();
    }

    /**
     * The concrete factory is usually chosen depending on configuration or
     * environment options.
     */
    static void configure() {
        if (System.getProperty(&quot;os.name&quot;).equals(&quot;Windows 10&quot;)) {
            dialog = new WindowsDialog();
        } else {
            dialog = new HtmlDialog();
        }
    }

    /**
     * All of the client code should work with factories and products through
     * abstract interfaces. This way it does not care which factory it works
     * with and what kind of product it returns.
     */
    static void runBusinessLogic() {
        dialog.renderWindow();
    }
}</code></pre>
</figure>

#### []{#example-0--OutputDemo-txt .anchor} **OutputDemo.txt:** Resultados de la ejecución (HtmlDialog)

<figure class="code">
<pre class="code" lang="output"><code>&lt;button&gt;Test Button&lt;/button&gt;
Click! Button says - &#39;Hello World!&#39;</code></pre>
</figure>

#### []{#example-0--OutputDemo-png .anchor} **OutputDemo.png:** Resultados de la ejecución (WindowsDialog)

![](../../../../images/patterns/examples/java/factory-method/OutputDemof025.png?id=36afce413161f6650321896d3023fb65)

::: next
#### Leer siguiente

[Prototype en Java []{.fa
.fa-arrow-right}](../../prototype/java/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Volver

[[]{.fa .fa-arrow-left} Builder en
Java](../../builder/java/example.html){.btn .btn-default rel="prev"}
:::

## **Factory Method** en otros lenguajes

[![Factory Method en
C#](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Factory Method en C#"){.prog-lang-link}
[![Factory Method en
C++](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Factory Method en C++"){.prog-lang-link}
[![Factory Method en
Go](../../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Factory Method en Go"){.prog-lang-link}
[![Factory Method en
PHP](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Factory Method en PHP"){.prog-lang-link}
[![Factory Method en
Python](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Factory Method en Python"){.prog-lang-link}
[![Factory Method en
Ruby](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Factory Method en Ruby"){.prog-lang-link}
[![Factory Method en
Rust](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Factory Method en Rust"){.prog-lang-link}
[![Factory Method en
Swift](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Factory Method en Swift"){.prog-lang-link}
[![Factory Method en
TypeScript](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Factory Method en TypeScript"){.prog-lang-link}
::::::::::::::::::::::::::::::::

::::::: {.prom .banner-sidebar .banner .banner-striped .banner-examples .banner-removable .banner-removable-patterns data-id="DP: Examples-IDE" creative-id="examples-sidebar-es" position="sidebar"}
:::::: {.banner-inner style="font-size: 14px; line-height: 21px;"}
::: banner-examples-img
[![](../../../../images/patterns/banners/examples-ide91ca.png?id=3115b4b548fb96b75974e2de8f4f49bc){width="215"
height="158" loading="lazy"
srcset="/images/patterns/banners/examples-ide-2x.png?id=93c007a6d157b97c28eb213f59d716ae 2x"}](../../book.html)
:::

::: banner-examples-fade
[ Archivo con ejemplos de código](../../book.html){.btn .btn-secondary
.btn-block}
:::

::: {.banner-examples-text style="margin-top: 1rem;"}
Compra el libro electrónico **Sumérgete en los patrones de diseño** para
tener acceso al archivo con decenas de ejemplos detallados que puedes
abrir en tu IDE.

[ Saber más...](../../book.html){.btn .btn-secondary .btn-block}
:::
::::::
:::::::
::::::::::::::::::::::::::::::::::::::
:::::::::::::::::::::::::::::::::::::::
