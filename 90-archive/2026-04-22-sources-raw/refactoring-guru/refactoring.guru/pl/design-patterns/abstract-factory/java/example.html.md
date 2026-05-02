:::::::::::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::::::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [Wzorce
projektowe](../../../design-patterns.html){.type} / [Fabryka
abstrakcyjna](../../abstract-factory.html){.type} /
[Java](../../java.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Fabryka
abstrakcyjna](../../../../images/patterns/cards/abstract-factory-minid1c5.png?id=4c3927c446313a38ce77dfee38111e27){srcset="/images/patterns/cards/abstract-factory-mini-2x.png?id=22236aaa65ff52cbde1c713216d52c1f 2x"}
:::

# **Fabryka abstrakcyjna** w języku Java {#fabryka-abstrakcyjna-w-języku-java .pattern-example-title-block-title}
::::

::::::::::::::::::::::::::::::::::: pattern-example-body
::: pattern-example-brief
**Fabryka abstrakcyjna** jest kreacyjnym wzorcem projektowym, który
pozwala tworzyć rodziny spokrewnionych ze sobą obiektów bez określania
ich konkretnych klas.

Fabryka abstrakcyjna definiuje interfejs służący tworzeniu
poszczególnych produktów, ale pozostawia faktyczne tworzenie produktów
konkretnym klasom fabrycznym. Każdy typ fabryki odpowiada jednemu z
wariantów produktu.

Kod klienta wywołuje metody kreacyjne obiektu fabrycznego zamiast
tworzyć produkty bezpośrednio --- wywołując konstruktor (za pomocą
operatora `new`). Skoro dana fabryka odpowiada jednemu z wariantów
produktu, to wszystkie jej produkty będą ze sobą kompatybilne.

Kod klienta współpracuje z fabrykami i produktami wyłącznie poprzez ich
abstrakcyjne interfejsy. Dzięki temu jeden klient jest kompatybilny z
wieloma różnymi produktami. Wystarczy stworzyć nową konkretną klasę
fabryczną i przekazać ją kodowi klienta.

> Jeśli masz problem ze zrozumieniem różnicy pomiędzy poszczególnymi
> koncepcjami i wzorcami wytwórczymi, przeczytaj nasze [Porównanie
> fabryk](../../factory-comparison.html).
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Dowiedz się więcej o Fabryka abstrakcyjna
](../../abstract-factory.html){.btn .btn-lg .btn-primary}
:::

:::::::::::::::::::::::::::::::: {.sidebar-navigation .with-scroll}
::: en-title
Nawigacja
:::

::: en-intro
 [Intro](#)
:::

::: en-intro
 [Rodziny międzyplatformowych komponentów graficznego interfejsu
użytkownika (GUI) i ich wytwarzanie](#example-0)
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

**Złożoność:** []{.fa-stars}

**Popularność:** []{.fa-stars}

**Przykłady użycia:** Wzorzec Fabryka abstrakcyjna jest dość powszechnie
stosowany w kodzie Java. Wiele frameworków i bibliotek dzięki niemu
pozwala rozszerzać i dostosowywać swoje standardowe komponenty.

Oto przykłady użycia wzorca w głównych bibliotekach Java:

-   [`javax.xml.parsers.DocumentBuilderFactory#newInstance()`](http://docs.oracle.com/javase/8/docs/api/javax/xml/parsers/DocumentBuilderFactory.html#newInstance--)

-   [`javax.xml.transform.TransformerFactory#newInstance()`](http://docs.oracle.com/javase/8/docs/api/javax/xml/transform/TransformerFactory.html#newInstance--)

-   [`javax.xml.xpath.XPathFactory#newInstance()`](http://docs.oracle.com/javase/8/docs/api/javax/xml/xpath/XPathFactory.html#newInstance--)

**Identyfikacja:** Wzorzec łatwo można rozpoznać na podstawie metod
które zwracają obiekty fabryczne, służące następnie tworzeniu
konkretnych pod-komponentów.
:::::::::::::::::::::::::::::::::::

[]{#example-0}

## Rodziny międzyplatformowych komponentów graficznego interfejsu użytkownika (GUI) i ich wytwarzanie {#example-0-title}

W tym przykładzie przyciski i pola wyboru posłużą za produkty. Mamy dwa
warianty: kontrolki w stylu macOS oraz Windows.

Fabryka abstrakcyjna definiuje interfejs mający służyć tworzeniu
przycisków oraz pól wyboru. Są dwie konkretne fabryki, z których każda
zwraca produkty w jednym z wariantów.

Kod klienta współpracuje z fabrykami i produktami za pośrednictwem
abstrakcyjnych interfejsów. Pozwala to jednemu klientowi współdziałać z
wieloma wariantami produktu --- zależnie od typu obiektu fabrycznego.

### []{#example-0--buttons .anchor} **buttons:** Pierwsza hierarchia produktów {#buttons-pierwsza-hierarchia-produktów .example-title}

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

### []{#example-0--checkboxes .anchor} **checkboxes:** Druga hierarchia produktów {#checkboxes-druga-hierarchia-produktów .example-title}

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

#### []{#example-0--factories-GUIFactory-java .anchor} **factories/GUIFactory.java:** Fabryka abstrakcyjna

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

#### []{#example-0--factories-MacOSFactory-java .anchor} **factories/MacOSFactory.java:** Konkretna fabryka (macOS)

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

#### []{#example-0--factories-WindowsFactory-java .anchor} **factories/WindowsFactory.java:** Konkretna fabryka (Windows)

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

#### []{#example-0--app-Application-java .anchor} **app/Application.java:** Kod klienta

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

#### []{#example-0--Demo-java .anchor} **Demo.java:** Konfiguracja aplikacji

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

#### []{#example-0--OutputDemo-txt .anchor} **OutputDemo.txt:** Wyniki działania

<figure class="code">
<pre class="code" lang="output"><code>You create WindowsButton.
You created WindowsCheckbox.</code></pre>
</figure>

::: next
#### Czytaj dalej

[Budowniczy w języku Java []{.fa
.fa-arrow-right}](../../builder/java/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Wróć

[[]{.fa .fa-arrow-left} Wzorce projektowe w Java](../../java.html){.btn
.btn-default rel="prev"}
:::

## **Fabryka abstrakcyjna** w innych językach

[![Fabryka abstrakcyjna w języku
C#](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Fabryka abstrakcyjna w języku C#"){.prog-lang-link}
[![Fabryka abstrakcyjna w języku
C++](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Fabryka abstrakcyjna w języku C++"){.prog-lang-link}
[![Fabryka abstrakcyjna w języku
Go](../../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Fabryka abstrakcyjna w języku Go"){.prog-lang-link}
[![Fabryka abstrakcyjna w języku
PHP](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Fabryka abstrakcyjna w języku PHP"){.prog-lang-link}
[![Fabryka abstrakcyjna w języku
Python](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Fabryka abstrakcyjna w języku Python"){.prog-lang-link}
[![Fabryka abstrakcyjna w języku
Ruby](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Fabryka abstrakcyjna w języku Ruby"){.prog-lang-link}
[![Fabryka abstrakcyjna w języku
Rust](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Fabryka abstrakcyjna w języku Rust"){.prog-lang-link}
[![Fabryka abstrakcyjna w języku
Swift](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Fabryka abstrakcyjna w języku Swift"){.prog-lang-link}
[![Fabryka abstrakcyjna w języku
TypeScript](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Fabryka abstrakcyjna w języku TypeScript"){.prog-lang-link}
:::::::::::::::::::::::::::::::::::::::::

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
:::::::::::::::::::::::::::::::::::::::::::::::
::::::::::::::::::::::::::::::::::::::::::::::::
