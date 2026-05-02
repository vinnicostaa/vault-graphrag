:::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [Patrones de
diseño](../../../design-patterns.html){.type} /
[Prototype](../../prototype.html){.type} / [PHP](../../php.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Prototype](../../../../images/patterns/cards/prototype-mini6540.png?id=bc3046bb39ff36574c08d49839fd1c8e){srcset="/images/patterns/cards/prototype-mini-2x.png?id=b871f789a736e7efbb1cd082d2de6398 2x"}
:::

# **Prototype** en PHP {#prototype-en-php .pattern-example-title-block-title}
::::

:::::::::::::: pattern-example-body
::: pattern-example-brief
**Prototype** es un patrón de diseño creacional que permite la clonación
de objetos, incluso los complejos, sin acoplarse a sus clases
específicas.

Todas las clases prototipo deben tener una interfaz común que haga
posible copiar objetos incluso si sus clases concretas son desconocidas.
Los objetos prototipo pueden producir copias completas, ya que los
objetos de la misma clase pueden acceder a los campos privados de los
demás.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Aprende más sobre el patrón Prototype ](../../prototype.html){.btn
.btn-lg .btn-primary}
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

**Ejemplos de uso:** El patrón Prototype está disponible en PHP [listo
para usarse](http://php.net/manual/es/language.oop5.cloning.php). Puedes
utilizar la palabra clave `clone` para crear un copia exacta de un
objeto. Para añadir soporte de clonación a una clase, debes implementar
un método `__clone`.

**Identificación:** El prototipo puede reconocerse fácilmente por un
método `clone` o `copy`, etc.
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

Este ejemplo ilustra la estructura del patrón de diseño **Prototype** y
se centra en las siguientes preguntas:

-   ¿De qué clases se compone?
-   ¿Qué papeles juegan esas clases?
-   ¿De qué forma se relacionan los elementos del patrón?

Después de conocer la estructura del patrón, será más fácil comprender
el siguiente ejemplo basado en un caso de uso real de PHP.

#### []{#example-0--index-php .anchor} **index.php:** Ejemplo conceptual

<figure class="code">
<pre class="code" lang="php"><code>&lt;?php

namespace RefactoringGuru\Prototype\Conceptual;

/**
 * The example class that has cloning ability. We&#39;ll see how the values of field
 * with different types will be cloned.
 */
class Prototype
{
    public $primitive;
    public $component;
    public $circularReference;

    /**
     * PHP has built-in cloning support. You can `clone` an object without
     * defining any special methods as long as it has fields of primitive types.
     * Fields containing objects retain their references in a cloned object.
     * Therefore, in some cases, you might want to clone those referenced
     * objects as well. You can do this in a special `__clone()` method.
     */
    public function __clone()
    {
        $this-&gt;component = clone $this-&gt;component;

        // Cloning an object that has a nested object with backreference
        // requires special treatment. After the cloning is completed, the
        // nested object should point to the cloned object, instead of the
        // original object.
        $this-&gt;circularReference = clone $this-&gt;circularReference;
        $this-&gt;circularReference-&gt;prototype = $this;
    }
}

class ComponentWithBackReference
{
    public $prototype;

    /**
     * Note that the constructor won&#39;t be executed during cloning. If you have
     * complex logic inside the constructor, you may need to execute it in the
     * `__clone` method as well.
     */
    public function __construct(Prototype $prototype)
    {
        $this-&gt;prototype = $prototype;
    }
}

/**
 * The client code.
 */
function clientCode()
{
    $p1 = new Prototype();
    $p1-&gt;primitive = 245;
    $p1-&gt;component = new \DateTime();
    $p1-&gt;circularReference = new ComponentWithBackReference($p1);

    $p2 = clone $p1;
    if ($p1-&gt;primitive === $p2-&gt;primitive) {
        echo &quot;Primitive field values have been carried over to a clone. Yay!\n&quot;;
    } else {
        echo &quot;Primitive field values have not been copied. Booo!\n&quot;;
    }
    if ($p1-&gt;component === $p2-&gt;component) {
        echo &quot;Simple component has not been cloned. Booo!\n&quot;;
    } else {
        echo &quot;Simple component has been cloned. Yay!\n&quot;;
    }

    if ($p1-&gt;circularReference === $p2-&gt;circularReference) {
        echo &quot;Component with back reference has not been cloned. Booo!\n&quot;;
    } else {
        echo &quot;Component with back reference has been cloned. Yay!\n&quot;;
    }

    if ($p1-&gt;circularReference-&gt;prototype === $p2-&gt;circularReference-&gt;prototype) {
        echo &quot;Component with back reference is linked to original object. Booo!\n&quot;;
    } else {
        echo &quot;Component with back reference is linked to the clone. Yay!\n&quot;;
    }
}

clientCode();</code></pre>
</figure>

#### []{#example-0--Output-txt .anchor} **Output.txt:** Resultado de la ejecución

<figure class="code">
<pre class="code" lang="output"><code>Primitive field values have been carried over to a clone. Yay!
Simple component has been cloned. Yay!
Component with back reference has been cloned. Yay!
Component with back reference is linked to the clone. Yay!</code></pre>
</figure>
:::

::: {#example-1 .tab-pane role="tabpanel" aria-labelledby="example-1-tab" style="padding-top: 24px"}
## Ejemplo del mundo real {#example-1-title}

El patrón **Prototype** proporciona una forma cómoda de replicar objetos
existentes en lugar de intentar reconstruirlos copiando directamente
todos sus campos. La vía directa no solo te acopla a las clases de los
objetos clonados, sino que además no te permite copiar los contenidos de
los campos privados. El patrón Prototype te permite realizar la
clonación dentro del contexto de la clase clonada, donde el acceso a los
campos privados de la clase no está restringido.

Este ejemplo te muestra cómo clonar un objeto Page (Página) complejo
utilizando el patrón Prototype. La clase Page tiene muchos campos
privados, que se trasladarán al objeto clonado gracias al patrón
Prototype.

#### []{#example-1--index-php .anchor} **index.php:** Ejemplo del mundo real

<figure class="code">
<pre class="code" lang="php"><code>&lt;?php

namespace RefactoringGuru\Prototype\RealWorld;

/**
 * Prototype.
 */
class Page
{
    private $title;

    private $body;

    /**
     * @var Author
     */
    private $author;

    private $comments = [];

    /**
     * @var \DateTime
     */
    private $date;

    // +100 private fields.

    public function __construct(string $title, string $body, Author $author)
    {
        $this-&gt;title = $title;
        $this-&gt;body = $body;
        $this-&gt;author = $author;
        $this-&gt;author-&gt;addToPage($this);
        $this-&gt;date = new \DateTime();
    }

    public function addComment(string $comment): void
    {
        $this-&gt;comments[] = $comment;
    }

    /**
     * You can control what data you want to carry over to the cloned object.
     *
     * For instance, when a page is cloned:
     * - It gets a new &quot;Copy of ...&quot; title.
     * - The author of the page remains the same. Therefore we leave the
     * reference to the existing object while adding the cloned page to the list
     * of the author&#39;s pages.
     * - We don&#39;t carry over the comments from the old page.
     * - We also attach a new date object to the page.
     */
    public function __clone()
    {
        $this-&gt;title = &quot;Copy of &quot; . $this-&gt;title;
        $this-&gt;author-&gt;addToPage($this);
        $this-&gt;comments = [];
        $this-&gt;date = new \DateTime();
    }
}

class Author
{
    private $name;

    /**
     * @var Page[]
     */
    private $pages = [];

    public function __construct(string $name)
    {
        $this-&gt;name = $name;
    }

    public function addToPage(Page $page): void
    {
        $this-&gt;pages[] = $page;
    }
}

/**
 * The client code.
 */
function clientCode()
{
    $author = new Author(&quot;John Smith&quot;);
    $page = new Page(&quot;Tip of the day&quot;, &quot;Keep calm and carry on.&quot;, $author);

    // ...

    $page-&gt;addComment(&quot;Nice tip, thanks!&quot;);

    // ...

    $draft = clone $page;
    echo &quot;Dump of the clone. Note that the author is now referencing two objects.\n\n&quot;;
    print_r($draft);
}

clientCode();</code></pre>
</figure>

#### []{#example-1--Output-txt .anchor} **Output.txt:** Resultado de la ejecución

<figure class="code">
<pre class="code" lang="output"><code>Dump of the clone. Note that the author is now referencing two objects.

RefactoringGuru\Prototype\RealWorld\Page Object
(
    [title:RefactoringGuru\Prototype\RealWorld\Page:private] =&gt; Copy of Tip of the day
    [body:RefactoringGuru\Prototype\RealWorld\Page:private] =&gt; Keep calm and carry on.
    [author:RefactoringGuru\Prototype\RealWorld\Page:private] =&gt; RefactoringGuru\Prototype\RealWorld\Author Object
        (
            [name:RefactoringGuru\Prototype\RealWorld\Author:private] =&gt; John Smith
            [pages:RefactoringGuru\Prototype\RealWorld\Author:private] =&gt; Array
                (
                    [0] =&gt; RefactoringGuru\Prototype\RealWorld\Page Object
                        (
                            [title:RefactoringGuru\Prototype\RealWorld\Page:private] =&gt; Tip of the day
                            [body:RefactoringGuru\Prototype\RealWorld\Page:private] =&gt; Keep calm and carry on.
                            [author:RefactoringGuru\Prototype\RealWorld\Page:private] =&gt; RefactoringGuru\Prototype\RealWorld\Author Object
 *RECURSION*
                            [comments:RefactoringGuru\Prototype\RealWorld\Page:private] =&gt; Array
                                (
                                    [0] =&gt; Nice tip, thanks!
                                )

                            [date:RefactoringGuru\Prototype\RealWorld\Page:private] =&gt; DateTime Object
                                (
                                    [date] =&gt; 2018-06-04 14:50:39.306237
                                    [timezone_type] =&gt; 3
                                    [timezone] =&gt; UTC
                                )

                        )

                    [1] =&gt; RefactoringGuru\Prototype\RealWorld\Page Object
 *RECURSION*
                )

        )

    [comments:RefactoringGuru\Prototype\RealWorld\Page:private] =&gt; Array
        (
        )

    [date:RefactoringGuru\Prototype\RealWorld\Page:private] =&gt; DateTime Object
        (
            [date] =&gt; 2018-06-04 14:50:39.306272
            [timezone_type] =&gt; 3
            [timezone] =&gt; UTC
        )

)</code></pre>
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

[Singleton en PHP []{.fa
.fa-arrow-right}](../../singleton/php/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Volver

[[]{.fa .fa-arrow-left} Factory Method en
PHP](../../factory-method/php/example.html){.btn .btn-default
rel="prev"}
:::

## **Prototype** en otros lenguajes

[![Prototype en
C#](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Prototype en C#"){.prog-lang-link}
[![Prototype en
C++](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Prototype en C++"){.prog-lang-link}
[![Prototype en
Go](../../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Prototype en Go"){.prog-lang-link}
[![Prototype en
Java](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Prototype en Java"){.prog-lang-link}
[![Prototype en
Python](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Prototype en Python"){.prog-lang-link}
[![Prototype en
Ruby](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Prototype en Ruby"){.prog-lang-link}
[![Prototype en
Rust](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Prototype en Rust"){.prog-lang-link}
[![Prototype en
Swift](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Prototype en Swift"){.prog-lang-link}
[![Prototype en
TypeScript](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Prototype en TypeScript"){.prog-lang-link}
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
