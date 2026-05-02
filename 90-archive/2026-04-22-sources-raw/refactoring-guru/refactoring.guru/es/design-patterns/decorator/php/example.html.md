:::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [Patrones de
diseño](../../../design-patterns.html){.type} /
[Decorator](../../decorator.html){.type} / [PHP](../../php.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Decorator](../../../../images/patterns/cards/decorator-mini6adb.png?id=d30458908e315af195cb183bc52dbef9){srcset="/images/patterns/cards/decorator-mini-2x.png?id=3b58e540d7d28523080cad341ed9b2e9 2x"}
:::

# **Decorator** en PHP {#decorator-en-php .pattern-example-title-block-title}
::::

:::::::::::::: pattern-example-body
::: pattern-example-brief
**Decorator** es un patrón de diseño estructural que permite añadir
dinámicamente nuevos comportamientos a objetos colocándolos dentro de
objetos especiales que los envuelven (\_wrappers\_).

Utilizando decoradores puedes envolver objetos innumerables veces, ya
que los objetos objetivo y los decoradores siguen la misma interfaz. El
objeto resultante obtendrá un comportamiento de apilado de todos los
*wrappers*.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Aprende más sobre el patrón Decorator ](../../decorator.html){.btn
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

**Ejemplos de uso:** El patrón Decorator es bastante común en el código
PHP, especialmente en el código relacionado con los flujos (streams).

**Identificación:** El patrón Decorator puede ser reconocido por métodos
de creación o el constructor que acepta objetos de la misma clase o
interfaz que la clase actual.
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

Este ejemplo ilustra la estructura del patrón de diseño **Decorator** y
se centra en las siguientes preguntas:

-   ¿De qué clases se compone?
-   ¿Qué papeles juegan esas clases?
-   ¿De qué forma se relacionan los elementos del patrón?

Después de conocer la estructura del patrón, será más fácil comprender
el siguiente ejemplo basado en un caso de uso real de PHP.

#### []{#example-0--index-php .anchor} **index.php:** Ejemplo conceptual

<figure class="code">
<pre class="code" lang="php"><code>&lt;?php

namespace RefactoringGuru\Decorator\Conceptual;

/**
 * The base Component interface defines operations that can be altered by
 * decorators.
 */
interface Component
{
    public function operation(): string;
}

/**
 * Concrete Components provide default implementations of the operations. There
 * might be several variations of these classes.
 */
class ConcreteComponent implements Component
{
    public function operation(): string
    {
        return &quot;ConcreteComponent&quot;;
    }
}

/**
 * The base Decorator class follows the same interface as the other components.
 * The primary purpose of this class is to define the wrapping interface for all
 * concrete decorators. The default implementation of the wrapping code might
 * include a field for storing a wrapped component and the means to initialize
 * it.
 */
class Decorator implements Component
{
    /**
     * @var Component
     */
    protected $component;

    public function __construct(Component $component)
    {
        $this-&gt;component = $component;
    }

    /**
     * The Decorator delegates all work to the wrapped component.
     */
    public function operation(): string
    {
        return $this-&gt;component-&gt;operation();
    }
}

/**
 * Concrete Decorators call the wrapped object and alter its result in some way.
 */
class ConcreteDecoratorA extends Decorator
{
    /**
     * Decorators may call parent implementation of the operation, instead of
     * calling the wrapped object directly. This approach simplifies extension
     * of decorator classes.
     */
    public function operation(): string
    {
        return &quot;ConcreteDecoratorA(&quot; . parent::operation() . &quot;)&quot;;
    }
}

/**
 * Decorators can execute their behavior either before or after the call to a
 * wrapped object.
 */
class ConcreteDecoratorB extends Decorator
{
    public function operation(): string
    {
        return &quot;ConcreteDecoratorB(&quot; . parent::operation() . &quot;)&quot;;
    }
}

/**
 * The client code works with all objects using the Component interface. This
 * way it can stay independent of the concrete classes of components it works
 * with.
 */
function clientCode(Component $component)
{
    // ...

    echo &quot;RESULT: &quot; . $component-&gt;operation();

    // ...
}

/**
 * This way the client code can support both simple components...
 */
$simple = new ConcreteComponent();
echo &quot;Client: I&#39;ve got a simple component:\n&quot;;
clientCode($simple);
echo &quot;\n\n&quot;;

/**
 * ...as well as decorated ones.
 *
 * Note how decorators can wrap not only simple components but the other
 * decorators as well.
 */
$decorator1 = new ConcreteDecoratorA($simple);
$decorator2 = new ConcreteDecoratorB($decorator1);
echo &quot;Client: Now I&#39;ve got a decorated component:\n&quot;;
clientCode($decorator2);</code></pre>
</figure>

#### []{#example-0--Output-txt .anchor} **Output.txt:** Resultado de la ejecución

<figure class="code">
<pre class="code" lang="output"><code>Client: I&#39;ve got a simple component:
RESULT: ConcreteComponent

Client: Now I&#39;ve got a decorated component:
RESULT: ConcreteDecoratorB(ConcreteDecoratorA(ConcreteComponent))</code></pre>
</figure>
:::

::: {#example-1 .tab-pane role="tabpanel" aria-labelledby="example-1-tab" style="padding-top: 24px"}
## Ejemplo del mundo real {#example-1-title}

En este ejemplo, el patrón **Decorator** te ayuda a construir reglas
complejas de filtrado de texto para limpiar el contenido antes de
representarlo en una página web. Los distintos tipos de contenido, como
comentarios, publicaciones en foros, o mensajes privados, requieren
distintos grupos de filtros.

Por ejemplo, aunque quieras eliminar todo el HTML de los comentarios,
quizá desees mantener algunas etiquetas HTML básicas en publicaciones en
el foro. Además, puede que quieras permitir publicar en formato
Markdown, que debe procesarse antes de que se realice el filtrado HTML.
Todas estas reglas de filtrado pueden representarse como clases
decoradoras separadas, que se pueden apilar de otra forma, dependiendo
de la naturaleza del contenido que tengas.

#### []{#example-1--index-php .anchor} **index.php:** Ejemplo del mundo real

<figure class="code">
<pre class="code" lang="php"><code>&lt;?php

namespace RefactoringGuru\Decorator\RealWorld;

/**
 * The Component interface declares a filtering method that must be implemented
 * by all concrete components and decorators.
 */
interface InputFormat
{
    public function formatText(string $text): string;
}

/**
 * The Concrete Component is a core element of decoration. It contains the
 * original text, as is, without any filtering or formatting.
 */
class TextInput implements InputFormat
{
    public function formatText(string $text): string
    {
        return $text;
    }
}

/**
 * The base Decorator class doesn&#39;t contain any real filtering or formatting
 * logic. Its main purpose is to implement the basic decoration infrastructure:
 * a field for storing a wrapped component or another decorator and the basic
 * formatting method that delegates the work to the wrapped object. The real
 * formatting job is done by subclasses.
 */
class TextFormat implements InputFormat
{
    /**
     * @var InputFormat
     */
    protected $inputFormat;

    public function __construct(InputFormat $inputFormat)
    {
        $this-&gt;inputFormat = $inputFormat;
    }

    /**
     * Decorator delegates all work to a wrapped component.
     */
    public function formatText(string $text): string
    {
        return $this-&gt;inputFormat-&gt;formatText($text);
    }
}

/**
 * This Concrete Decorator strips out all HTML tags from the given text.
 */
class PlainTextFilter extends TextFormat
{
    public function formatText(string $text): string
    {
        $text = parent::formatText($text);
        return strip_tags($text);
    }
}

/**
 * This Concrete Decorator strips only dangerous HTML tags and attributes that
 * may lead to an XSS vulnerability.
 */
class DangerousHTMLTagsFilter extends TextFormat
{
    private $dangerousTagPatterns = [
        &quot;|&lt;script.*?&gt;([\s\S]*)?&lt;/script&gt;|i&quot;, // ...
    ];

    private $dangerousAttributes = [
        &quot;onclick&quot;, &quot;onkeypress&quot;, // ...
    ];


    public function formatText(string $text): string
    {
        $text = parent::formatText($text);

        foreach ($this-&gt;dangerousTagPatterns as $pattern) {
            $text = preg_replace($pattern, &#39;&#39;, $text);
        }

        foreach ($this-&gt;dangerousAttributes as $attribute) {
            $text = preg_replace_callback(&#39;|&lt;(.*?)&gt;|&#39;, function ($matches) use ($attribute) {
                $result = preg_replace(&quot;|$attribute=|i&quot;, &#39;&#39;, $matches[1]);
                return &quot;&lt;&quot; . $result . &quot;&gt;&quot;;
            }, $text);
        }

        return $text;
    }
}

/**
 * This Concrete Decorator provides a rudimentary Markdown → HTML conversion.
 */
class MarkdownFormat extends TextFormat
{
    public function formatText(string $text): string
    {
        $text = parent::formatText($text);

        // Format block elements.
        $chunks = preg_split(&#39;|\n\n|&#39;, $text);
        foreach ($chunks as &amp;$chunk) {
            // Format headers.
            if (preg_match(&#39;|^#+|&#39;, $chunk)) {
                $chunk = preg_replace_callback(&#39;|^(#+)(.*?)$|&#39;, function ($matches) {
                    $h = strlen($matches[1]);
                    return &quot;&lt;h$h&gt;&quot; . trim($matches[2]) . &quot;&lt;/h$h&gt;&quot;;
                }, $chunk);
            } // Format paragraphs.
            else {
                $chunk = &quot;&lt;p&gt;$chunk&lt;/p&gt;&quot;;
            }
        }
        $text = implode(&quot;\n\n&quot;, $chunks);

        // Format inline elements.
        $text = preg_replace(&quot;|__(.*?)__|&quot;, &#39;&lt;strong&gt;$1&lt;/strong&gt;&#39;, $text);
        $text = preg_replace(&quot;|\*\*(.*?)\*\*|&quot;, &#39;&lt;strong&gt;$1&lt;/strong&gt;&#39;, $text);
        $text = preg_replace(&quot;|_(.*?)_|&quot;, &#39;&lt;em&gt;$1&lt;/em&gt;&#39;, $text);
        $text = preg_replace(&quot;|\*(.*?)\*|&quot;, &#39;&lt;em&gt;$1&lt;/em&gt;&#39;, $text);

        return $text;
    }
}


/**
 * The client code might be a part of a real website, which renders user-
 * generated content. Since it works with formatters through the Component
 * interface, it doesn&#39;t care whether it gets a simple component object or a
 * decorated one.
 */
function displayCommentAsAWebsite(InputFormat $format, string $text)
{
    // ..

    echo $format-&gt;formatText($text);

    // ..
}

/**
 * Input formatters are very handy when dealing with user-generated content.
 * Displaying such content &quot;as is&quot; could be very dangerous, especially when
 * anonymous users can generate it (e.g. comments). Your website is not only
 * risking getting tons of spammy links but may also be exposed to XSS attacks.
 */
$dangerousComment = &lt;&lt;&lt;HERE
Hello! Nice blog post!
Please visit my &lt;a href=&#39;http://www.iwillhackyou.com&#39;&gt;homepage&lt;/a&gt;.
&lt;script src=&quot;http://www.iwillhackyou.com/script.js&quot;&gt;
  performXSSAttack();
&lt;/script&gt;
HERE;

/**
 * Naive comment rendering (unsafe).
 */
$naiveInput = new TextInput();
echo &quot;Website renders comments without filtering (unsafe):\n&quot;;
displayCommentAsAWebsite($naiveInput, $dangerousComment);
echo &quot;\n\n\n&quot;;

/**
 * Filtered comment rendering (safe).
 */
$filteredInput = new PlainTextFilter($naiveInput);
echo &quot;Website renders comments after stripping all tags (safe):\n&quot;;
displayCommentAsAWebsite($filteredInput, $dangerousComment);
echo &quot;\n\n\n&quot;;


/**
 * Decorator allows stacking multiple input formats to get fine-grained control
 * over the rendered content.
 */
$dangerousForumPost = &lt;&lt;&lt;HERE
# Welcome

This is my first post on this **gorgeous** forum.

&lt;script src=&quot;http://www.iwillhackyou.com/script.js&quot;&gt;
  performXSSAttack();
&lt;/script&gt;
HERE;

/**
 * Naive post rendering (unsafe, no formatting).
 */
$naiveInput = new TextInput();
echo &quot;Website renders a forum post without filtering and formatting (unsafe, ugly):\n&quot;;
displayCommentAsAWebsite($naiveInput, $dangerousForumPost);
echo &quot;\n\n\n&quot;;

/**
 * Markdown formatter + filtering dangerous tags (safe, pretty).
 */
$text = new TextInput();
$markdown = new MarkdownFormat($text);
$filteredInput = new DangerousHTMLTagsFilter($markdown);
echo &quot;Website renders a forum post after translating markdown markup&quot; .
    &quot; and filtering some dangerous HTML tags and attributes (safe, pretty):\n&quot;;
displayCommentAsAWebsite($filteredInput, $dangerousForumPost);
echo &quot;\n\n\n&quot;;</code></pre>
</figure>

#### []{#example-1--Output-txt .anchor} **Output.txt:** Resultado de la ejecución

<figure class="code">
<pre class="code" lang="output"><code>Website renders comments without filtering (unsafe):
Hello! Nice blog post!
Please visit my &lt;a href=&#39;http://www.iwillhackyou.com&#39;&gt;homepage&lt;/a&gt;.
&lt;script src=&quot;http://www.iwillhackyou.com/script.js&quot;&gt;
  performXSSAttack();
&lt;/script&gt;


Website renders comments after stripping all tags (safe):
Hello! Nice blog post!
Please visit my homepage.

  performXSSAttack();



Website renders a forum post without filtering and formatting (unsafe, ugly):
# Welcome

This is my first post on this **gorgeous** forum.

&lt;script src=&quot;http://www.iwillhackyou.com/script.js&quot;&gt;
  performXSSAttack();
&lt;/script&gt;


Website renders a forum post after translating markdown markupand filtering some dangerous HTML tags and attributes (safe, pretty):
&lt;h1&gt;Welcome&lt;/h1&gt;

&lt;p&gt;This is my first post on this &lt;strong&gt;gorgeous&lt;/strong&gt; forum.&lt;/p&gt;

&lt;p&gt;&lt;/p&gt;</code></pre>
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

[Facade en PHP []{.fa
.fa-arrow-right}](../../facade/php/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Volver

[[]{.fa .fa-arrow-left} Composite en
PHP](../../composite/php/example.html){.btn .btn-default rel="prev"}
:::

## **Decorator** en otros lenguajes

[![Decorator en
C#](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Decorator en C#"){.prog-lang-link}
[![Decorator en
C++](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Decorator en C++"){.prog-lang-link}
[![Decorator en
Go](../../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Decorator en Go"){.prog-lang-link}
[![Decorator en
Java](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Decorator en Java"){.prog-lang-link}
[![Decorator en
Python](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Decorator en Python"){.prog-lang-link}
[![Decorator en
Ruby](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Decorator en Ruby"){.prog-lang-link}
[![Decorator en
Rust](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Decorator en Rust"){.prog-lang-link}
[![Decorator en
Swift](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Decorator en Swift"){.prog-lang-link}
[![Decorator en
TypeScript](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Decorator en TypeScript"){.prog-lang-link}
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
