::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [Patrons de
conception](../../../design-patterns.html){.type} /
[Fabrique](../../factory-method.html){.type} /
[Java](../../java.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Fabrique](../../../../images/patterns/cards/factory-method-minid7f6.png?id=72619e9527893374b98a5913779ac167){srcset="/images/patterns/cards/factory-method-mini-2x.png?id=fa9d4a8d61a67cc3822e52b9daf69dad 2x"}
:::

# **Fabrique** en Java {#fabrique-en-java .pattern-example-title-block-title}
::::

:::::::::::::::::::::::::: pattern-example-body
::: pattern-example-brief
La **Fabrique** est un patron de conception de création qui permet de
créer des produits sans avoir à préciser leurs classes concrètes.

La fabrique définit une méthode qui doit être utilisée pour créer des
objets à la place de l'appel au constructeur (opérateur `new`). Les
sous-classes peuvent redéfinir cette méthode pour modifier la classe des
objets qui seront créés.

> Lisez notre [Comparaison des fabriques](../../factory-comparison.html)
> si vous avez du mal à saisir la différence entre les divers concepts
> et patrons.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ En savoir plus sur la patron Fabrique
](../../factory-method.html){.btn .btn-lg .btn-primary}
:::

::::::::::::::::::::::: {.sidebar-navigation .with-scroll}
::: en-title
Navigation
:::

::: en-intro
 [Intro](#)
:::

::: en-intro
 [Fabrication d'éléments d'une GUI multiplateforme](#example-0)
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

**Complexité :** []{.fa-stars}

**Popularité :** []{.fa-stars}

**Exemples d'utilisation :** La fabrique est très largement utilisée en
Java. Elle va permettre de rendre votre code très flexible.

Ce patron est présent dans les bibliothèques principales de Java :

-   [`java.util.Calendar#getInstance()`](http://docs.oracle.com/javase/8/docs/api/java/util/Calendar.html#getInstance--)
-   [`java.util.ResourceBundle#getBundle()`](http://docs.oracle.com/javase/8/docs/api/java/util/ResourceBundle.html#getBundle-java.lang.String-)
-   [`java.text.NumberFormat#getInstance()`](http://docs.oracle.com/javase/8/docs/api/java/text/NumberFormat.html#getInstance--)
-   [`java.nio.charset.Charset#forName()`](http://docs.oracle.com/javase/8/docs/api/java/nio/charset/Charset.html#forName-java.lang.String-)
-   [`java.net.URLStreamHandlerFactory#createURLStreamHandler(String)`](http://docs.oracle.com/javase/8/docs/api/java/net/URLStreamHandlerFactory.html)
    (retourne différents objets Singleton en fonction du protocole)
-   [`java.util.EnumSet#of()`](https://docs.oracle.com/javase/8/docs/api/java/util/EnumSet.html#of(E))
-   [`javax.xml.bind.JAXBContext#createMarshaller()`](https://docs.oracle.com/javase/8/docs/api/javax/xml/bind/JAXBContext.html#createMarshaller--)
    et d'autres méthodes similaires.

**Identification :** La fabrique peut être reconnue grâce à ses méthodes
de création qui produisent des objets depuis les classes concrètes, mais
les retournent en tant qu'objets de type abstrait ou d'interface.
::::::::::::::::::::::::::

[]{#example-0}

## Fabrication d'éléments d'une GUI multiplateforme {#example-0-title}

Dans cet exemple, les boutons jouent le rôle du produit et les boîtes de
dialogue sont les créateurs.

Les différentes boîtes de dialogues ont besoin de leurs propres types
d'éléments. C'est pour cela que nous créons une sous-classe pour chaque
boîte de dialogue et redéfinissons leur fabrique.

À présent, chaque boîte de dialogue instanciera ses propres classes de
boutons. La boîte de dialogue de base utilise une interface commune pour
manipuler les produits, ainsi son code fonctionne toujours même après
toutes ces modifications.

### []{#example-0--buttons .anchor} **buttons** {#buttons .example-title}

#### []{#example-0--buttons-Button-java .anchor} **buttons/Button.java:** Interface commune des produits

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

#### []{#example-0--buttons-HtmlButton-java .anchor} **buttons/HtmlButton.java:** Produit concret

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

#### []{#example-0--buttons-WindowsButton-java .anchor} **buttons/WindowsButton.java:** Produit concret supplémentaire

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

#### []{#example-0--factory-Dialog-java .anchor} **factory/Dialog.java:** Créateur de base

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

#### []{#example-0--factory-HtmlDialog-java .anchor} **factory/HtmlDialog.java:** Créateur concret

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

#### []{#example-0--factory-WindowsDialog-java .anchor} **factory/WindowsDialog.java:** Créateur concret supplémentaire

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

#### []{#example-0--Demo-java .anchor} **Demo.java:** Code client

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

#### []{#example-0--OutputDemo-txt .anchor} **OutputDemo.txt:** Résultat de l'exécution (HtmlDialog)

<figure class="code">
<pre class="code" lang="output"><code>&lt;button&gt;Test Button&lt;/button&gt;
Click! Button says - &#39;Hello World!&#39;</code></pre>
</figure>

#### []{#example-0--OutputDemo-png .anchor} **OutputDemo.png:** Résultat de l'exécution (WindowsDialog)

![](../../../../images/patterns/examples/java/%20factory-/OutputDemo.html)

::: next
#### Suivant

[Prototype en Java []{.fa
.fa-arrow-right}](../../prototype/java/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Retour

[[]{.fa .fa-arrow-left} Monteur en
Java](../../builder/java/example.html){.btn .btn-default rel="prev"}
:::

## **Fabrique** dans les autres langues

[![Fabrique en
C#](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Fabrique en C#"){.prog-lang-link}
[![Fabrique en
C++](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Fabrique en C++"){.prog-lang-link}
[![Fabrique en
Go](../../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Fabrique en Go"){.prog-lang-link}
[![Fabrique en
PHP](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Fabrique en PHP"){.prog-lang-link}
[![Fabrique en
Python](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Fabrique en Python"){.prog-lang-link}
[![Fabrique en
Ruby](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Fabrique en Ruby"){.prog-lang-link}
[![Fabrique en
Rust](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Fabrique en Rust"){.prog-lang-link}
[![Fabrique en
Swift](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Fabrique en Swift"){.prog-lang-link}
[![Fabrique en
TypeScript](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Fabrique en TypeScript"){.prog-lang-link}
::::::::::::::::::::::::::::::::

::::::: {.prom .banner-sidebar .banner .banner-striped .banner-examples .banner-removable .banner-removable-patterns data-id="DP: Examples-IDE" creative-id="examples-sidebar-fr" position="sidebar"}
:::::: {.banner-inner style="font-size: 14px; line-height: 21px;"}
::: banner-examples-img
[![](../../../../images/patterns/banners/examples-ide91ca.png?id=3115b4b548fb96b75974e2de8f4f49bc){width="215"
height="158" loading="lazy"
srcset="/images/patterns/banners/examples-ide-2x.png?id=93c007a6d157b97c28eb213f59d716ae 2x"}](../../book.html)
:::

::: banner-examples-fade
[ Archive avec exemples](../../book.html){.btn .btn-secondary
.btn-block}
:::

::: {.banner-examples-text style="margin-top: 1rem;"}
Achetez l'ebook **Plongée au cœur des patrons de conception** et obtenez
l'accès à une archive comprenant des dizaines d'exemples détaillés qui
peuvent être ouverts dans votre environnement de développement.

[ En savoir plus...](../../book.html){.btn .btn-secondary .btn-block}
:::
::::::
:::::::
::::::::::::::::::::::::::::::::::::::
:::::::::::::::::::::::::::::::::::::::
