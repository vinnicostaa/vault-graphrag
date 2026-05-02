:::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [Patrons de
conception](../../../design-patterns.html){.type} /
[Pont](../../bridge.html){.type} / [PHP](../../php.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Pont](../../../../images/patterns/cards/bridge-mini9e60.png?id=b389101d8ee8e23ffa1b534c704d0774){srcset="/images/patterns/cards/bridge-mini-2x.png?id=2622384cf623ed150ee9c21a0812dd87 2x"}
:::

# **Pont** en PHP {#pont-en-php .pattern-example-title-block-title}
::::

:::::::::::::: pattern-example-body
::: pattern-example-brief
Le **Pont** est un patron de conception structurel qui scinde la logique
métier ou divise de grandes classes dans des hiérarchies de classes
séparées qui vont ensuite évoluer indépendamment.

Une de ces hiérarchies (souvent appelée l'abstraction) gardera une
référence vers un objet de la seconde hiérarchie (l'implémentation).
L'abstraction pourra déléguer certains (parfois la majorité) de ses
appels aux objets de l'implémentation. Puisque toutes les
implémentations ont une interface commune, elles sont interchangeables à
l'intérieur de l'abstraction.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ En savoir plus sur la patron Pont ](../../bridge.html){.btn .btn-lg
.btn-primary}
:::

::::::::::: {.sidebar-navigation .with-scroll}
::: en-title
Navigation
:::

::: en-intro
 [Intro](#)
:::

::: en-intro
 [Exemple conceptuel](#example-0)
:::

::: en-file
 [index](#example-0--index-php)
:::

::: en-file
 [Output](#example-0--Output-txt)
:::

::: en-intro
 [Analogie du monde réel](#example-1)
:::

::: en-file
 [index](#example-1--index-php)
:::

::: en-file
 [Output](#example-1--Output-txt)
:::
:::::::::::

**Complexité :** []{.fa-stars}

**Popularité :** []{.fa-stars}

**Exemples d'utilisation :** Le pont est très utile pour prendre en
charge différents types de serveurs de bases de données ou pour
travailler avec plusieurs fournisseurs d'API d'un certain genre (par
exemple les plateformes de cloud, réseaux sociaux, etc.).

**Identification :** Le pont peut être identifié grâce à une distinction
très nette entre une entité de contrôle et plusieurs plateformes
différentes dont elle dépend.
::::::::::::::

::: {#nav-tab .nav .nav-tabs .nav-justified role="tablist"}
[Exemple conceptuel](#example-0){#example-0-tab .nav-item .nav-link
.active toggle="tab" role="tab" aria-controls="example-0"
aria-selected="true"} [Analogie du monde
réel](#example-1){#example-1-tab .nav-item .nav-link toggle="tab"
role="tab" aria-controls="example-1" aria-selected="true"}
:::

::::: {#nav-tabContent .pattern-example-tab-content .tab-content}
::: {#example-0 .tab-pane .show .active role="tabpanel" aria-labelledby="example-0-tab" style="padding-top: 24px"}
## Exemple conceptuel {#example-0-title}

Dans cet exemple, nous allons voir la structure du **Pont** et répondre
aux questions suivantes :

-   Que contiennent les classes ?
-   Quels rôles jouent-elles ?
-   Comment les éléments du patron sont-ils reliés ?

Après avoir étudié la structure du patron, vous pourrez plus facilement
comprendre l'exemple suivant qui est basé sur un cas d'utilisation réel
en PHP.

#### []{#example-0--index-php .anchor} **index.php:** Exemple conceptuel

<figure class="code">
<pre class="code" lang="php"><code>&lt;?php

namespace RefactoringGuru\Bridge\Conceptual;

/**
 * The Abstraction defines the interface for the &quot;control&quot; part of the two class
 * hierarchies. It maintains a reference to an object of the Implementation
 * hierarchy and delegates all of the real work to this object.
 */
class Abstraction
{
    /**
     * @var Implementation
     */
    protected $implementation;

    public function __construct(Implementation $implementation)
    {
        $this-&gt;implementation = $implementation;
    }

    public function operation(): string
    {
        return &quot;Abstraction: Base operation with:\n&quot; .
            $this-&gt;implementation-&gt;operationImplementation();
    }
}

/**
 * You can extend the Abstraction without changing the Implementation classes.
 */
class ExtendedAbstraction extends Abstraction
{
    public function operation(): string
    {
        return &quot;ExtendedAbstraction: Extended operation with:\n&quot; .
            $this-&gt;implementation-&gt;operationImplementation();
    }
}

/**
 * The Implementation defines the interface for all implementation classes. It
 * doesn&#39;t have to match the Abstraction&#39;s interface. In fact, the two
 * interfaces can be entirely different. Typically the Implementation interface
 * provides only primitive operations, while the Abstraction defines higher-
 * level operations based on those primitives.
 */
interface Implementation
{
    public function operationImplementation(): string;
}

/**
 * Each Concrete Implementation corresponds to a specific platform and
 * implements the Implementation interface using that platform&#39;s API.
 */
class ConcreteImplementationA implements Implementation
{
    public function operationImplementation(): string
    {
        return &quot;ConcreteImplementationA: Here&#39;s the result on the platform A.\n&quot;;
    }
}

class ConcreteImplementationB implements Implementation
{
    public function operationImplementation(): string
    {
        return &quot;ConcreteImplementationB: Here&#39;s the result on the platform B.\n&quot;;
    }
}

/**
 * Except for the initialization phase, where an Abstraction object gets linked
 * with a specific Implementation object, the client code should only depend on
 * the Abstraction class. This way the client code can support any abstraction-
 * implementation combination.
 */
function clientCode(Abstraction $abstraction)
{
    // ...

    echo $abstraction-&gt;operation();

    // ...
}

/**
 * The client code should be able to work with any pre-configured abstraction-
 * implementation combination.
 */
$implementation = new ConcreteImplementationA();
$abstraction = new Abstraction($implementation);
clientCode($abstraction);

echo &quot;\n&quot;;

$implementation = new ConcreteImplementationB();
$abstraction = new ExtendedAbstraction($implementation);
clientCode($abstraction);</code></pre>
</figure>

#### []{#example-0--Output-txt .anchor} **Output.txt:** Résultat de l'exécution

<figure class="code">
<pre class="code" lang="output"><code>Abstraction: Base operation with:
ConcreteImplementationA: Here&#39;s the result on the platform A.

ExtendedAbstraction: Extended operation with:
ConcreteImplementationB: Here&#39;s the result on the platform B.</code></pre>
</figure>
:::

::: {#example-1 .tab-pane role="tabpanel" aria-labelledby="example-1-tab" style="padding-top: 24px"}
## Analogie du monde réel {#example-1-title}

Dans cet exemple, la hiérarchie de la `Page` joue le rôle de
l'**Abstraction** et la hiérarchie du `Convertisseur` (renderer), celui
de l'**Implémentation**. Les objets de la classe `Page` peuvent
assembler des pages Web d'un type particulier en utilisant des éléments
basiques que l'objet `Convertisseur` affecté à la page leur a fournis.
Puisque les deux hiérarchies de classes sont séparées, vous pouvez
ajouter une nouvelle classe `Convertisseur` sans toucher aux classes
`Page` et inversement.

#### []{#example-1--index-php .anchor} **index.php:** Exemple du monde réel

<figure class="code">
<pre class="code" lang="php"><code>&lt;?php

namespace RefactoringGuru\Bridge\RealWorld;

/**
 * The Abstraction.
 */
abstract class Page
{
    /**
     * @var Renderer
     */
    protected $renderer;

    /**
     * The Abstraction is usually initialized with one of the Implementation
     * objects.
     */
    public function __construct(Renderer $renderer)
    {
        $this-&gt;renderer = $renderer;
    }

    /**
     * The Bridge pattern allows replacing the attached Implementation object
     * dynamically.
     */
    public function changeRenderer(Renderer $renderer): void
    {
        $this-&gt;renderer = $renderer;
    }

    /**
     * The &quot;view&quot; behavior stays abstract since it can only be provided by
     * Concrete Abstraction classes.
     */
    abstract public function view(): string;
}

/**
 * This Concrete Abstraction represents a simple page.
 */
class SimplePage extends Page
{
    protected $title;
    protected $content;

    public function __construct(Renderer $renderer, string $title, string $content)
    {
        parent::__construct($renderer);
        $this-&gt;title = $title;
        $this-&gt;content = $content;
    }

    public function view(): string
    {
        return $this-&gt;renderer-&gt;renderParts([
            $this-&gt;renderer-&gt;renderHeader(),
            $this-&gt;renderer-&gt;renderTitle($this-&gt;title),
            $this-&gt;renderer-&gt;renderTextBlock($this-&gt;content),
            $this-&gt;renderer-&gt;renderFooter()
        ]);
    }
}

/**
 * This Concrete Abstraction represents a more complex page.
 */
class ProductPage extends Page
{
    protected $product;

    public function __construct(Renderer $renderer, Product $product)
    {
        parent::__construct($renderer);
        $this-&gt;product = $product;
    }

    public function view(): string
    {
        return $this-&gt;renderer-&gt;renderParts([
            $this-&gt;renderer-&gt;renderHeader(),
            $this-&gt;renderer-&gt;renderTitle($this-&gt;product-&gt;getTitle()),
            $this-&gt;renderer-&gt;renderTextBlock($this-&gt;product-&gt;getDescription()),
            $this-&gt;renderer-&gt;renderImage($this-&gt;product-&gt;getImage()),
            $this-&gt;renderer-&gt;renderTextBlock(&#39;$&#39; . number_format($this-&gt;product-&gt;getPrice(), 2)),
            $this-&gt;renderer-&gt;renderLink(&quot;/cart/add/&quot; . $this-&gt;product-&gt;getId(), &quot;Add to cart&quot;),
            $this-&gt;renderer-&gt;renderFooter()
        ]);
    }
}

/**
 * A helper class for the ProductPage class.
 */
class Product
{
    private $id, $title, $description, $image, $price;

    public function __construct(
        string $id,
        string $title,
        string $description,
        string $image,
        float $price
    ) {
        $this-&gt;id = $id;
        $this-&gt;title = $title;
        $this-&gt;description = $description;
        $this-&gt;image = $image;
        $this-&gt;price = $price;
    }

    public function getId(): string
    {
        return $this-&gt;id;
    }

    public function getTitle(): string
    {
        return $this-&gt;title;
    }

    public function getDescription(): string
    {
        return $this-&gt;description;
    }

    public function getImage(): string
    {
        return $this-&gt;image;
    }

    public function getPrice(): float
    {
        return $this-&gt;price;
    }
}


/**
 * The Implementation declares a set of &quot;real&quot;, &quot;under-the-hood&quot;, &quot;platform&quot;
 * methods.
 *
 * In this case, the Implementation lists rendering methods that can be used to
 * compose any web page. Different Abstractions may use different methods of the
 * Implementation.
 */
interface Renderer
{
    public function renderTitle(string $title): string;

    public function renderTextBlock(string $text): string;

    public function renderImage(string $url): string;

    public function renderLink(string $url, string $title): string;

    public function renderHeader(): string;

    public function renderFooter(): string;

    public function renderParts(array $parts): string;
}

/**
 * This Concrete Implementation renders a web page as HTML.
 */
class HTMLRenderer implements Renderer
{
    public function renderTitle(string $title): string
    {
        return &quot;&lt;h1&gt;$title&lt;/h1&gt;&quot;;
    }

    public function renderTextBlock(string $text): string
    {
        return &quot;&lt;div class=&#39;text&#39;&gt;$text&lt;/div&gt;&quot;;
    }

    public function renderImage(string $url): string
    {
        return &quot;&lt;img src=&#39;$url&#39;&gt;&quot;;
    }

    public function renderLink(string $url, string $title): string
    {
        return &quot;&lt;a href=&#39;$url&#39;&gt;$title&lt;/a&gt;&quot;;
    }

    public function renderHeader(): string
    {
        return &quot;&lt;html&gt;&lt;body&gt;&quot;;
    }

    public function renderFooter(): string
    {
        return &quot;&lt;/body&gt;&lt;/html&gt;&quot;;
    }

    public function renderParts(array $parts): string
    {
        return implode(&quot;\n&quot;, $parts);
    }
}

/**
 * This Concrete Implementation renders a web page as JSON strings.
 */
class JsonRenderer implements Renderer
{
    public function renderTitle(string $title): string
    {
        return &#39;&quot;title&quot;: &quot;&#39; . $title . &#39;&quot;&#39;;
    }

    public function renderTextBlock(string $text): string
    {
        return &#39;&quot;text&quot;: &quot;&#39; . $text . &#39;&quot;&#39;;
    }

    public function renderImage(string $url): string
    {
        return &#39;&quot;img&quot;: &quot;&#39; . $url . &#39;&quot;&#39;;
    }

    public function renderLink(string $url, string $title): string
    {
        return &#39;&quot;link&quot;: {&quot;href&quot;: &quot;&#39; . $url . &#39;&quot;, &quot;title&quot;: &quot;&#39; . $title . &#39;&quot;}&#39;;
    }

    public function renderHeader(): string
    {
        return &#39;&#39;;
    }

    public function renderFooter(): string
    {
        return &#39;&#39;;
    }

    public function renderParts(array $parts): string
    {
        return &quot;{\n&quot; . implode(&quot;,\n&quot;, array_filter($parts)) . &quot;\n}&quot;;
    }
}

/**
 * The client code usually deals only with the Abstraction objects.
 */
function clientCode(Page $page)
{
    // ...

    echo $page-&gt;view();

    // ...
}

/**
 * The client code can be executed with any pre-configured combination of the
 * Abstraction+Implementation.
 */
$HTMLRenderer = new HTMLRenderer();
$JSONRenderer = new JsonRenderer();

$page = new SimplePage($HTMLRenderer, &quot;Home&quot;, &quot;Welcome to our website!&quot;);
echo &quot;HTML view of a simple content page:\n&quot;;
clientCode($page);
echo &quot;\n\n&quot;;

/**
 * The Abstraction can change the linked Implementation at runtime if needed.
 */
$page-&gt;changeRenderer($JSONRenderer);
echo &quot;JSON view of a simple content page, rendered with the same client code:\n&quot;;
clientCode($page);
echo &quot;\n\n&quot;;


$product = new Product(
    &quot;123&quot;,
    &quot;Star Wars, episode1&quot;,
    &quot;A long time ago in a galaxy far, far away...&quot;,
    &quot;/images/star-wars.jpeg&quot;,
    39.95
);

$page = new ProductPage($HTMLRenderer, $product);
echo &quot;HTML view of a product page, same client code:\n&quot;;
clientCode($page);
echo &quot;\n\n&quot;;

$page-&gt;changeRenderer($JSONRenderer);
echo &quot;JSON view of a simple content page, with the same client code:\n&quot;;
clientCode($page);</code></pre>
</figure>

#### []{#example-1--Output-txt .anchor} **Output.txt:** Résultat de l'exécution

<figure class="code">
<pre class="code" lang="output"><code>HTML view of a simple content page:
&lt;html&gt;&lt;body&gt;
&lt;h1&gt;Home&lt;/h1&gt;
&lt;div class=&#39;text&#39;&gt;Welcome to our website!&lt;/div&gt;
&lt;/body&gt;&lt;/html&gt;

JSON view of a simple content page, rendered with the same client code:
{
&quot;title&quot;: &quot;Home&quot;,
&quot;text&quot;: &quot;Welcome to our website!&quot;
}

HTML view of a product page, same client code:
&lt;html&gt;&lt;body&gt;
&lt;h1&gt;Star Wars, episode1&lt;/h1&gt;
&lt;div class=&#39;text&#39;&gt;A long time ago in a galaxy far, far away...&lt;/div&gt;
&lt;img src=&#39;/images/star-wars.jpeg&#39;&gt;
&lt;a href=&#39;/cart/add/123&#39;&gt;Add to cart&lt;/a&gt;
&lt;/body&gt;&lt;/html&gt;

JSON view of a simple content page, with the same client code:
{
&quot;title&quot;: &quot;Star Wars, episode1&quot;,
&quot;text&quot;: &quot;A long time ago in a galaxy far, far away...&quot;,
&quot;img&quot;: &quot;/images/star-wars.jpeg&quot;,
&quot;link&quot;: {&quot;href&quot;: &quot;/cart/add/123&quot;, &quot;title&quot;: &quot;Add to cart&quot;}
}</code></pre>
</figure>
:::
:::::

::: {#nav-tab .nav .nav-tabs .nav-tabs-bottom .nav-justified role="tablist"}
[Exemple conceptuel](#example-0){#example-0-tab-bottom .nav-item
.nav-link .active toggle="tab" role="tab" aria-controls="example-0"
aria-selected="true"} [Analogie du monde
réel](#example-1){#example-1-tab-bottom .nav-item .nav-link toggle="tab"
role="tab" aria-controls="example-1" aria-selected="true"}
:::

::: next
#### Suivant

[Composite en PHP []{.fa
.fa-arrow-right}](../../composite/php/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Retour

[[]{.fa .fa-arrow-left} Adaptateur en
PHP](../../adapter/php/example.html){.btn .btn-default rel="prev"}
:::

## **Pont** dans les autres langues

[![Pont en
C#](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Pont en C#"){.prog-lang-link}
[![Pont en
C++](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Pont en C++"){.prog-lang-link}
[![Pont en
Go](../../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Pont en Go"){.prog-lang-link}
[![Pont en
Java](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Pont en Java"){.prog-lang-link}
[![Pont en
Python](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Pont en Python"){.prog-lang-link}
[![Pont en
Ruby](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Pont en Ruby"){.prog-lang-link}
[![Pont en
Rust](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Pont en Rust"){.prog-lang-link}
[![Pont en
Swift](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Pont en Swift"){.prog-lang-link}
[![Pont en
TypeScript](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Pont en TypeScript"){.prog-lang-link}
:::::::::::::::::::::::::

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
:::::::::::::::::::::::::::::::
::::::::::::::::::::::::::::::::
