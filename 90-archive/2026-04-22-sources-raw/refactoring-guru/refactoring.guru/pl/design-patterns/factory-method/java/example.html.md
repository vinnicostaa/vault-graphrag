::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [Wzorce
projektowe](../../../design-patterns.html){.type} / [Metoda
wytwórcza](../../factory-method.html){.type} /
[Java](../../java.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Metoda
wytwórcza](../../../../images/patterns/cards/factory-method-minid7f6.png?id=72619e9527893374b98a5913779ac167){srcset="/images/patterns/cards/factory-method-mini-2x.png?id=fa9d4a8d61a67cc3822e52b9daf69dad 2x"}
:::

# **Metoda wytwórcza** w języku Java {#metoda-wytwórcza-w-języku-java .pattern-example-title-block-title}
::::

:::::::::::::::::::::::::: pattern-example-body
::: pattern-example-brief
**Metoda wytwórcza** jest kreacyjnym wzorcem projektowym rozwiązującym
problem tworzenia obiektów-produktów bez określania ich konkretnych
klas.

Metoda wytwórcza definiuje metodę która ma służyć tworzeniu obiektów bez
bezpośredniego wywoływania konstruktora (poprzez operator `new`).
Podklasy mogą nadpisać tę metodę w celu zmiany klasy tworzonych
obiektów.

> Jeśli masz problem ze zrozumieniem różnicy pomiędzy poszczególnymi
> koncepcjami i wzorcami wytwórczymi, przeczytaj nasze [Porównanie
> fabryk](../../factory-comparison.html).
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Dowiedz się więcej o Metoda wytwórcza
](../../factory-method.html){.btn .btn-lg .btn-primary}
:::

::::::::::::::::::::::: {.sidebar-navigation .with-scroll}
::: en-title
Nawigacja
:::

::: en-intro
 [Intro](#)
:::

::: en-intro
 [Tworzenie międzyplatformowych elementów graficznego interfejsu
użytkownika (GUI)](#example-0)
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

**Złożoność:** []{.fa-stars}

**Popularność:** []{.fa-stars}

**Przykłady użycia:** Wzorzec Metoda wytwórcza jest szeroko
rozpowszechniony w kodzie Java. Przydaje się gdy trzeba wyposażyć kod w
wysoki poziom elastyczności.

Wzorzec jest obecny w głównych bibliotekach Java:

-   [`java.util.Calendar#getInstance()`](http://docs.oracle.com/javase/8/docs/api/java/util/Calendar.html#getInstance--)
-   [`java.util.ResourceBundle#getBundle()`](http://docs.oracle.com/javase/8/docs/api/java/util/ResourceBundle.html#getBundle-java.lang.String-)
-   [`java.text.NumberFormat#getInstance()`](http://docs.oracle.com/javase/8/docs/api/java/text/NumberFormat.html#getInstance--)
-   [`java.nio.charset.Charset#forName()`](http://docs.oracle.com/javase/8/docs/api/java/nio/charset/Charset.html#forName-java.lang.String-)
-   [`java.net.URLStreamHandlerFactory#createURLStreamHandler(String)`](http://docs.oracle.com/javase/8/docs/api/java/net/URLStreamHandlerFactory.html)
    (Zwraca różne obiekty singleton, zależnie od protokołu)
-   [`java.util.EnumSet#of()`](https://docs.oracle.com/javase/8/docs/api/java/util/EnumSet.html#of(E))
-   [`javax.xml.bind.JAXBContext#createMarshaller()`](https://docs.oracle.com/javase/8/docs/api/javax/xml/bind/JAXBContext.html#createMarshaller--) i
    inne podobne metody.

**Identyfikacja:** Metody wytwórcze można rozpoznać po metodach
kreacyjnych tworzących obiekty na podstawie konkretnych klas, ale
zwracających typ abstrakcyjny lub interfejs.
::::::::::::::::::::::::::

[]{#example-0}

## Tworzenie międzyplatformowych elementów graficznego interfejsu użytkownika (GUI) {#example-0-title}

W poniższym przykładzie przyciski odgrywają rolę produktów, a okna
dialogowe --- twórców.

Różne rodzaje okien dialogowych wymagają własnych typów elementów.
Dlatego na każde okno dialogowe tworzymy podklasę i nadpisujemy jej
metody wytwórcze.

W rezultacie każdy rodzaj okna dialogowego stworzy instancje
odpowiednich klas przycisków. Klasa bazowa okna dialogowego
współdziała z produktami za pośrednictwem ich wspólnego interfejsu, więc
jej kod będzie działał nawet po wszystkich zmianach.

### []{#example-0--buttons .anchor} **buttons** {#buttons .example-title}

#### []{#example-0--buttons-Button-java .anchor} **buttons/Button.java:** Wspólny interfejs produktu

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

#### []{#example-0--buttons-HtmlButton-java .anchor} **buttons/HtmlButton.java:** Konkretny produkt

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

#### []{#example-0--buttons-WindowsButton-java .anchor} **buttons/WindowsButton.java:** Kolejny konkretny produkt

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

#### []{#example-0--factory-Dialog-java .anchor} **factory/Dialog.java:** Bazowy twórca

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

#### []{#example-0--factory-HtmlDialog-java .anchor} **factory/HtmlDialog.java:** Konkretny twórca

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

#### []{#example-0--factory-WindowsDialog-java .anchor} **factory/WindowsDialog.java:** Kolejny konkretny twórca

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

#### []{#example-0--Demo-java .anchor} **Demo.java:** Kod klienta

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

#### []{#example-0--OutputDemo-txt .anchor} **OutputDemo.txt:** Wyniki działania (HtmlDialog)

<figure class="code">
<pre class="code" lang="output"><code>&lt;button&gt;Test Button&lt;/button&gt;
Click! Button says - &#39;Hello World!&#39;</code></pre>
</figure>

#### []{#example-0--OutputDemo-png .anchor} **OutputDemo.png:** Wyniki działania (WindowsDialog)

![](../../../../images/patterns/examples/java/factory-method/OutputDemof025.png?id=36afce413161f6650321896d3023fb65)

::: next
#### Czytaj dalej

[Prototyp w języku Java []{.fa
.fa-arrow-right}](../../prototype/java/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Wróć

[[]{.fa .fa-arrow-left} Budowniczy w języku
Java](../../builder/java/example.html){.btn .btn-default rel="prev"}
:::

## **Metoda wytwórcza** w innych językach

[![Metoda wytwórcza w języku
C#](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Metoda wytwórcza w języku C#"){.prog-lang-link}
[![Metoda wytwórcza w języku
C++](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Metoda wytwórcza w języku C++"){.prog-lang-link}
[![Metoda wytwórcza w języku
Go](../../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Metoda wytwórcza w języku Go"){.prog-lang-link}
[![Metoda wytwórcza w języku
PHP](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Metoda wytwórcza w języku PHP"){.prog-lang-link}
[![Metoda wytwórcza w języku
Python](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Metoda wytwórcza w języku Python"){.prog-lang-link}
[![Metoda wytwórcza w języku
Ruby](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Metoda wytwórcza w języku Ruby"){.prog-lang-link}
[![Metoda wytwórcza w języku
Rust](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Metoda wytwórcza w języku Rust"){.prog-lang-link}
[![Metoda wytwórcza w języku
Swift](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Metoda wytwórcza w języku Swift"){.prog-lang-link}
[![Metoda wytwórcza w języku
TypeScript](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Metoda wytwórcza w języku TypeScript"){.prog-lang-link}
::::::::::::::::::::::::::::::::

::::::: {.prom .banner-sidebar .banner .banner-striped .banner-examples .banner-removable .banner-removable-patterns data-id="DP: Examples-IDE" creative-id="examples-sidebar-pl" position="sidebar"}
:::::: {.banner-inner style="font-size: 14px; line-height: 21px;"}
::: banner-examples-img
[![](../../../../images/patterns/banners/examples-ide91ca.png?id=3115b4b548fb96b75974e2de8f4f49bc){width="215"
height="158" loading="lazy"
srcset="/images/patterns/banners/examples-ide-2x.png?id=93c007a6d157b97c28eb213f59d716ae 2x"}](../../book.html)
:::

::: banner-examples-fade
[ Archiwum\
przykładów kodu](../../book.html){.btn .btn-secondary .btn-block}
:::

::: {.banner-examples-text style="margin-top: 1rem;"}
Kupując eBook **Wzorce projektowe. Nowoczesny podręcznik** zyskujesz
dostęp do archiwum zawierającego mnóstwo szczegółowych przykładów, które
można od razu wkleić do swojego preferowanego zintegrowanego środowiska
programistycznego.

[ Dowiedz się więcej...](../../book.html){.btn .btn-secondary
.btn-block}
:::
::::::
:::::::
::::::::::::::::::::::::::::::::::::::
:::::::::::::::::::::::::::::::::::::::
