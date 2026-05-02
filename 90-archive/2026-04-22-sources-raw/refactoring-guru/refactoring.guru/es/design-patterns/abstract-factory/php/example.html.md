:::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [Patrones de
diseño](../../../design-patterns.html){.type} / [Abstract
Factory](../../abstract-factory.html){.type} /
[PHP](../../php.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Abstract
Factory](../../../../images/patterns/cards/abstract-factory-minid1c5.png?id=4c3927c446313a38ce77dfee38111e27){srcset="/images/patterns/cards/abstract-factory-mini-2x.png?id=22236aaa65ff52cbde1c713216d52c1f 2x"}
:::

# **Abstract Factory** en PHP {#abstract-factory-en-php .pattern-example-title-block-title}
::::

:::::::::::::: pattern-example-body
::: pattern-example-brief
**Abstract Factory** es un patrón de diseño creacional que resuelve el
problema de crear familias enteras de productos sin especificar sus
clases concretas.

El patrón Abstract Factory define una interfaz para crear todos los
productos, pero deja la propia creación de productos para las clases de
fábrica concretas. Cada tipo de fábrica se corresponde con cierta
variedad de producto.

El código cliente invoca los métodos de creación de un objeto de fábrica
en lugar de crear los productos directamente con una llamada al
constructor (operador `new`). Como una fábrica se corresponde con una
única variante de producto, todos sus productos serán compatibles.

El código cliente trabaja con fábricas y productos únicamente a través
de sus interfaces abstractas. Esto permite al mismo código cliente
trabajar con productos diferentes. Simplemente, creas una nueva clase
fábrica concreta y la pasas al código cliente.

> Si no sabes la diferencia entre los distintos patrones de fábrica y
> sus conceptos, lee nuestra [Comparación de
> fábricas](../../factory-comparison.html).
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Aprende más sobre el patrón Abstract Factory
](../../abstract-factory.html){.btn .btn-lg .btn-primary}
:::

::::::::::: {.sidebar-navigation .with-scroll}
::: en-title
Navegación
:::

::: en-intro
 [Intro](#)
:::

::: en-intro
 [Ejemplo conceptual](#example-0)
:::

::: en-file
 [index](#example-0--index-php)
:::

::: en-file
 [Output](#example-0--Output-txt)
:::

::: en-intro
 [Ejemplo del mundo real](#example-1)
:::

::: en-file
 [index](#example-1--index-php)
:::

::: en-file
 [Output](#example-1--Output-txt)
:::
:::::::::::

**Complejidad:** []{.fa-stars}

**Popularidad:** []{.fa-stars}

**Ejemplos de uso:** El patrón Abstract Factory es muy común en el
código PHP. Muchos *frameworks* y bibliotecas lo utilizan para
proporcionar una forma de extender y personalizar sus componentes
estándar.

**Identificación:** El patrón es fácil de reconocer por los métodos, que
devuelven un objeto de fábrica. Después, la fábrica se utiliza para
crear subcomponentes específicos.
::::::::::::::

::: {#nav-tab .nav .nav-tabs .nav-justified role="tablist"}
[Ejemplo conceptual](#example-0){#example-0-tab .nav-item .nav-link
.active toggle="tab" role="tab" aria-controls="example-0"
aria-selected="true"} [Ejemplo del mundo
real](#example-1){#example-1-tab .nav-item .nav-link toggle="tab"
role="tab" aria-controls="example-1" aria-selected="true"}
:::

::::: {#nav-tabContent .pattern-example-tab-content .tab-content}
::: {#example-0 .tab-pane .show .active role="tabpanel" aria-labelledby="example-0-tab" style="padding-top: 24px"}
## Ejemplo conceptual {#example-0-title}

Este ejemplo ilustra la estructura del patrón de diseño **Abstract
Factory**, centrándose en responder las siguientes preguntas:

-   ¿De qué clases se compone?
-   ¿Qué papeles juegan esas clases?
-   ¿De qué forma se relacionan los elementos del patrón?

Después de conocer la estructura del patrón, será más fácil comprender
el siguiente ejemplo basado en un caso de uso real de PHP.

#### []{#example-0--index-php .anchor} **index.php:** Ejemplo conceptual

<figure class="code">
<pre class="code" lang="php"><code>&lt;?php

namespace RefactoringGuru\AbstractFactory\Conceptual;

/**
 * The Abstract Factory interface declares a set of methods that return
 * different abstract products. These products are called a family and are
 * related by a high-level theme or concept. Products of one family are usually
 * able to collaborate among themselves. A family of products may have several
 * variants, but the products of one variant are incompatible with products of
 * another.
 */
interface AbstractFactory
{
    public function createProductA(): AbstractProductA;

    public function createProductB(): AbstractProductB;
}

/**
 * Concrete Factories produce a family of products that belong to a single
 * variant. The factory guarantees that resulting products are compatible. Note
 * that signatures of the Concrete Factory&#39;s methods return an abstract product,
 * while inside the method a concrete product is instantiated.
 */
class ConcreteFactory1 implements AbstractFactory
{
    public function createProductA(): AbstractProductA
    {
        return new ConcreteProductA1();
    }

    public function createProductB(): AbstractProductB
    {
        return new ConcreteProductB1();
    }
}

/**
 * Each Concrete Factory has a corresponding product variant.
 */
class ConcreteFactory2 implements AbstractFactory
{
    public function createProductA(): AbstractProductA
    {
        return new ConcreteProductA2();
    }

    public function createProductB(): AbstractProductB
    {
        return new ConcreteProductB2();
    }
}

/**
 * Each distinct product of a product family should have a base interface. All
 * variants of the product must implement this interface.
 */
interface AbstractProductA
{
    public function usefulFunctionA(): string;
}

/**
 * Concrete Products are created by corresponding Concrete Factories.
 */
class ConcreteProductA1 implements AbstractProductA
{
    public function usefulFunctionA(): string
    {
        return &quot;The result of the product A1.&quot;;
    }
}

class ConcreteProductA2 implements AbstractProductA
{
    public function usefulFunctionA(): string
    {
        return &quot;The result of the product A2.&quot;;
    }
}

/**
 * Here&#39;s the the base interface of another product. All products can interact
 * with each other, but proper interaction is possible only between products of
 * the same concrete variant.
 */
interface AbstractProductB
{
    /**
     * Product B is able to do its own thing...
     */
    public function usefulFunctionB(): string;

    /**
     * ...but it also can collaborate with the ProductA.
     *
     * The Abstract Factory makes sure that all products it creates are of the
     * same variant and thus, compatible.
     */
    public function anotherUsefulFunctionB(AbstractProductA $collaborator): string;
}

/**
 * Concrete Products are created by corresponding Concrete Factories.
 */
class ConcreteProductB1 implements AbstractProductB
{
    public function usefulFunctionB(): string
    {
        return &quot;The result of the product B1.&quot;;
    }

    /**
     * The variant, Product B1, is only able to work correctly with the variant,
     * Product A1. Nevertheless, it accepts any instance of AbstractProductA as
     * an argument.
     */
    public function anotherUsefulFunctionB(AbstractProductA $collaborator): string
    {
        $result = $collaborator-&gt;usefulFunctionA();

        return &quot;The result of the B1 collaborating with the ({$result})&quot;;
    }
}

class ConcreteProductB2 implements AbstractProductB
{
    public function usefulFunctionB(): string
    {
        return &quot;The result of the product B2.&quot;;
    }

    /**
     * The variant, Product B2, is only able to work correctly with the variant,
     * Product A2. Nevertheless, it accepts any instance of AbstractProductA as
     * an argument.
     */
    public function anotherUsefulFunctionB(AbstractProductA $collaborator): string
    {
        $result = $collaborator-&gt;usefulFunctionA();

        return &quot;The result of the B2 collaborating with the ({$result})&quot;;
    }
}

/**
 * The client code works with factories and products only through abstract
 * types: AbstractFactory and AbstractProduct. This lets you pass any factory or
 * product subclass to the client code without breaking it.
 */
function clientCode(AbstractFactory $factory)
{
    $productA = $factory-&gt;createProductA();
    $productB = $factory-&gt;createProductB();

    echo $productB-&gt;usefulFunctionB() . &quot;\n&quot;;
    echo $productB-&gt;anotherUsefulFunctionB($productA) . &quot;\n&quot;;
}

/**
 * The client code can work with any concrete factory class.
 */
echo &quot;Client: Testing client code with the first factory type:\n&quot;;
clientCode(new ConcreteFactory1());

echo &quot;\n&quot;;

echo &quot;Client: Testing the same client code with the second factory type:\n&quot;;
clientCode(new ConcreteFactory2());</code></pre>
</figure>

#### []{#example-0--Output-txt .anchor} **Output.txt:** Resultado de la ejecución

<figure class="code">
<pre class="code" lang="output"><code>Client: Testing client code with the first factory type:
The result of the product B1.
The result of the B1 collaborating with the (The result of the product A1.)

Client: Testing the same client code with the second factory type:
The result of the product B2.
The result of the B2 collaborating with the (The result of the product A2.)</code></pre>
</figure>
:::

::: {#example-1 .tab-pane role="tabpanel" aria-labelledby="example-1-tab" style="padding-top: 24px"}
## Ejemplo del mundo real {#example-1-title}

En este ejemplo, el patrón **Abstract Factory** proporciona una
infraestructura para crear varios tipos de plantillas para distintos
elementos de una página web.

Una aplicación web puede soportar diferentes motores de renderización al
mismo tiempo, pero sólo si sus clases son independientes de las clases
concretas de esos motores. Por ello, los objetos de la aplicación sólo
deben comunicarse con los objetos de plantilla a través de sus
interfaces abstractas. Tu código no debe crear los objetos de plantilla
directamente, sino delegar su creación a objetos de fábrica especiales.
Por último, tu código tampoco debe depender de los objetos de fábrica
sino, en su lugar, debe trabajar con ellos a través de la interfaz de
fábrica abstracta.

Como resultado, podrás proporcionar a la aplicación el objeto de fábrica
que se corresponde con uno de los motores de renderización. Todas las
plantillas creadas en la aplicación serán creadas por esa fábrica y su
tipo coincidirá con el tipo de la fábrica. Si decides cambiar el motor
de representación, podrás pasar una nueva fábrica al código cliente sin
descomponer el código existente.

#### []{#example-1--index-php .anchor} **index.php:** Ejemplo del mundo real

<figure class="code">
<pre class="code" lang="php"><code>&lt;?php

namespace RefactoringGuru\AbstractFactory\RealWorld;

/**
 * The Abstract Factory interface declares creation methods for each distinct
 * product type.
 */
interface TemplateFactory
{
    public function createTitleTemplate(): TitleTemplate;

    public function createPageTemplate(): PageTemplate;

    public function getRenderer(): TemplateRenderer;
}

/**
 * Each Concrete Factory corresponds to a specific variant (or family) of
 * products.
 *
 * This Concrete Factory creates Twig templates.
 */
class TwigTemplateFactory implements TemplateFactory
{
    public function createTitleTemplate(): TitleTemplate
    {
        return new TwigTitleTemplate();
    }

    public function createPageTemplate(): PageTemplate
    {
        return new TwigPageTemplate($this-&gt;createTitleTemplate());
    }

    public function getRenderer(): TemplateRenderer
    {
        return new TwigRenderer();
    }
}

/**
 * And this Concrete Factory creates PHPTemplate templates.
 */
class PHPTemplateFactory implements TemplateFactory
{
    public function createTitleTemplate(): TitleTemplate
    {
        return new PHPTemplateTitleTemplate();
    }

    public function createPageTemplate(): PageTemplate
    {
        return new PHPTemplatePageTemplate($this-&gt;createTitleTemplate());
    }

    public function getRenderer(): TemplateRenderer
    {
        return new PHPTemplateRenderer();
    }
}

/**
 * Each distinct product type should have a separate interface. All variants of
 * the product must follow the same interface.
 *
 * For instance, this Abstract Product interface describes the behavior of page
 * title templates.
 */
interface TitleTemplate
{
    public function getTemplateString(): string;
}

/**
 * This Concrete Product provides Twig page title templates.
 */
class TwigTitleTemplate implements TitleTemplate
{
    public function getTemplateString(): string
    {
        return &quot;&lt;h1&gt;{{ title }}&lt;/h1&gt;&quot;;
    }
}

/**
 * And this Concrete Product provides PHPTemplate page title templates.
 */
class PHPTemplateTitleTemplate implements TitleTemplate
{
    public function getTemplateString(): string
    {
        return &quot;&lt;h1&gt;&lt;?= \$title; ?&gt;&lt;/h1&gt;&quot;;
    }
}

/**
 * This is another Abstract Product type, which describes whole page templates.
 */
interface PageTemplate
{
    public function getTemplateString(): string;
}

/**
 * The page template uses the title sub-template, so we have to provide the way
 * to set it in the sub-template object. The abstract factory will link the page
 * template with a title template of the same variant.
 */
abstract class BasePageTemplate implements PageTemplate
{
    protected $titleTemplate;

    public function __construct(TitleTemplate $titleTemplate)
    {
        $this-&gt;titleTemplate = $titleTemplate;
    }
}

/**
 * The Twig variant of the whole page templates.
 */
class TwigPageTemplate extends BasePageTemplate
{
    public function getTemplateString(): string
    {
        $renderedTitle = $this-&gt;titleTemplate-&gt;getTemplateString();

        return &lt;&lt;&lt;HTML
        &lt;div class=&quot;page&quot;&gt;
            $renderedTitle
            &lt;article class=&quot;content&quot;&gt;{{ content }}&lt;/article&gt;
        &lt;/div&gt;
        HTML;
    }
}

/**
 * The PHPTemplate variant of the whole page templates.
 */
class PHPTemplatePageTemplate extends BasePageTemplate
{
    public function getTemplateString(): string
    {
        $renderedTitle = $this-&gt;titleTemplate-&gt;getTemplateString();

        return &lt;&lt;&lt;HTML
        &lt;div class=&quot;page&quot;&gt;
            $renderedTitle
            &lt;article class=&quot;content&quot;&gt;&lt;?= \$content; ?&gt;&lt;/article&gt;
        &lt;/div&gt;
        HTML;
    }
}

/**
 * The renderer is responsible for converting a template string into the actual
 * HTML code. Each renderer behaves differently and expects its own type of
 * template strings passed to it. Baking templates with the factory let you pass
 * proper types of templates to proper renders.
 */
interface TemplateRenderer
{
    public function render(string $templateString, array $arguments = []): string;
}

/**
 * The renderer for Twig templates.
 */
class TwigRenderer implements TemplateRenderer
{
    public function render(string $templateString, array $arguments = []): string
    {
        return \Twig::render($templateString, $arguments);
    }
}

/**
 * The renderer for PHPTemplate templates. Note that this implementation is very
 * basic, if not crude. Using the `eval` function has many security
 * implications, so use it with caution in real projects.
 */
class PHPTemplateRenderer implements TemplateRenderer
{
    public function render(string $templateString, array $arguments = []): string
    {
        extract($arguments);

        ob_start();
        eval(&#39; ?&gt;&#39; . $templateString . &#39;&lt;?php &#39;);
        $result = ob_get_contents();
        ob_end_clean();

        return $result;
    }
}

/**
 * The client code. Note that it accepts the Abstract Factory class as the
 * parameter, which allows the client to work with any concrete factory type.
 */
class Page
{
    public $title;

    public $content;

    public function __construct($title, $content)
    {
        $this-&gt;title = $title;
        $this-&gt;content = $content;
    }

    // Here&#39;s how would you use the template further in real life. Note that the
    // page class does not depend on any concrete template classes.
    public function render(TemplateFactory $factory): string
    {
        $pageTemplate = $factory-&gt;createPageTemplate();

        $renderer = $factory-&gt;getRenderer();

        return $renderer-&gt;render($pageTemplate-&gt;getTemplateString(), [
            &#39;title&#39; =&gt; $this-&gt;title,
            &#39;content&#39; =&gt; $this-&gt;content
        ]);
    }
}

/**
 * Now, in other parts of the app, the client code can accept factory objects of
 * any type.
 */
$page = new Page(&#39;Sample page&#39;, &#39;This is the body.&#39;);

echo &quot;Testing actual rendering with the PHPTemplate factory:\n&quot;;
echo $page-&gt;render(new PHPTemplateFactory());


// Uncomment the following if you have Twig installed.

// echo &quot;Testing rendering with the Twig factory:\n&quot;; echo $page-&gt;render(new
// TwigTemplateFactory());</code></pre>
</figure>

#### []{#example-1--Output-txt .anchor} **Output.txt:** Resultado de la ejecución

<figure class="code">
<pre class="code" lang="output"><code>Testing actual rendering with the PHPTemplate factory:
&lt;div class=&quot;page&quot;&gt;
    &lt;h1&gt;Sample page&lt;/h1&gt;
    &lt;article class=&quot;content&quot;&gt;This it the body.&lt;/article&gt;
&lt;/div&gt;</code></pre>
</figure>
:::
:::::

::: {#nav-tab .nav .nav-tabs .nav-tabs-bottom .nav-justified role="tablist"}
[Ejemplo conceptual](#example-0){#example-0-tab-bottom .nav-item
.nav-link .active toggle="tab" role="tab" aria-controls="example-0"
aria-selected="true"} [Ejemplo del mundo
real](#example-1){#example-1-tab-bottom .nav-item .nav-link toggle="tab"
role="tab" aria-controls="example-1" aria-selected="true"}
:::

::: next
#### Leer siguiente

[Builder en PHP []{.fa
.fa-arrow-right}](../../builder/php/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Volver

[[]{.fa .fa-arrow-left} Patrones de diseño en PHP](../../php.html){.btn
.btn-default rel="prev"}
:::

## **Abstract Factory** en otros lenguajes

[![Abstract Factory en
C#](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Abstract Factory en C#"){.prog-lang-link}
[![Abstract Factory en
C++](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Abstract Factory en C++"){.prog-lang-link}
[![Abstract Factory en
Go](../../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Abstract Factory en Go"){.prog-lang-link}
[![Abstract Factory en
Java](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Abstract Factory en Java"){.prog-lang-link}
[![Abstract Factory en
Python](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Abstract Factory en Python"){.prog-lang-link}
[![Abstract Factory en
Ruby](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Abstract Factory en Ruby"){.prog-lang-link}
[![Abstract Factory en
Rust](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Abstract Factory en Rust"){.prog-lang-link}
[![Abstract Factory en
Swift](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Abstract Factory en Swift"){.prog-lang-link}
[![Abstract Factory en
TypeScript](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Abstract Factory en TypeScript"){.prog-lang-link}
:::::::::::::::::::::::::

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
:::::::::::::::::::::::::::::::
::::::::::::::::::::::::::::::::
