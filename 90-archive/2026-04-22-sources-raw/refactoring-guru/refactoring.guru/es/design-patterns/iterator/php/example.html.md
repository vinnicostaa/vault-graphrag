:::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [Patrones de
diseño](../../../design-patterns.html){.type} /
[Iterator](../../iterator.html){.type} / [PHP](../../php.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Iterator](../../../../images/patterns/cards/iterator-minie2c2.png?id=76c28bb48f997b36965983dd2b41f02e){srcset="/images/patterns/cards/iterator-mini-2x.png?id=7d28f66a487066774a743cfceaa40ea4 2x"}
:::

# **Iterator** en PHP {#iterator-en-php .pattern-example-title-block-title}
::::

:::::::::::::: pattern-example-body
::: pattern-example-brief
**Iterator** es un patrón de diseño de comportamiento que permite el
recorrido secuencial por una estructura de datos compleja sin exponer
sus detalles internos.

Gracias al patrón Iterator, los clientes pueden recorrer elementos de
colecciones diferentes de un modo similar, utilizando una única interfaz
iteradora.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Aprende más sobre el patrón Iterator ](../../iterator.html){.btn
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

**Ejemplos de uso:** El patrón es muy común en el código PHP. Muchos
*frameworks* y bibliotecas lo utilizan para proporcionar una forma
estandarizada de recorrer sus colecciones.

El PHP tiene una interfaz
[Iteradora](http://php.net/manual/es/language.oop5.iterations.php)
integrada que se puede utilizar para crear iteradores a medida
compatibles con el resto del código PHP.

**Identificación:** El patrón Iterator es fácil de reconocer por sus
métodos de navegación (como `next`, `previous` y otros). El código
cliente que utiliza iteradores puede no tener acceso directo a la
colección recorrida.
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

Este ejemplo ilustra la estructura del patrón de diseño **Iterator** y
se centra en las siguientes preguntas:

-   ¿De qué clases se compone?
-   ¿Qué papeles juegan esas clases?
-   ¿De qué forma se relacionan los elementos del patrón?

Después de conocer la estructura del patrón, será más fácil comprender
el siguiente ejemplo basado en un caso de uso real de PHP.

#### []{#example-0--index-php .anchor} **index.php:** Ejemplo conceptual

<figure class="code">
<pre class="code" lang="php"><code>&lt;?php

namespace RefactoringGuru\Iterator\Conceptual;

/**
 * Concrete Iterators implement various traversal algorithms. These classes
 * store the current traversal position at all times.
 */
class AlphabeticalOrderIterator implements \Iterator
{
    /**
     * @var WordsCollection
     */
    private $collection;

    /**
     * @var int Stores the current traversal position. An iterator may have a
     * lot of other fields for storing iteration state, especially when it is
     * supposed to work with a particular kind of collection.
     */
    private $position = 0;

    /**
     * @var bool This variable indicates the traversal direction.
     */
    private $reverse = false;

    public function __construct($collection, $reverse = false)
    {
        $this-&gt;collection = $collection;
        $this-&gt;reverse = $reverse;
    }

    public function rewind()
    {
        $this-&gt;position = $this-&gt;reverse ?
            count($this-&gt;collection-&gt;getItems()) - 1 : 0;
    }

    public function current()
    {
        return $this-&gt;collection-&gt;getItems()[$this-&gt;position];
    }

    public function key()
    {
        return $this-&gt;position;
    }

    public function next()
    {
        $this-&gt;position = $this-&gt;position + ($this-&gt;reverse ? -1 : 1);
    }

    public function valid()
    {
        return isset($this-&gt;collection-&gt;getItems()[$this-&gt;position]);
    }
}

/**
 * Concrete Collections provide one or several methods for retrieving fresh
 * iterator instances, compatible with the collection class.
 */
class WordsCollection implements \IteratorAggregate
{
    private $items = [];

    public function getItems()
    {
        return $this-&gt;items;
    }

    public function addItem($item)
    {
        $this-&gt;items[] = $item;
    }

    public function getIterator(): Iterator
    {
        return new AlphabeticalOrderIterator($this);
    }

    public function getReverseIterator(): Iterator
    {
        return new AlphabeticalOrderIterator($this, true);
    }
}

/**
 * The client code may or may not know about the Concrete Iterator or Collection
 * classes, depending on the level of indirection you want to keep in your
 * program.
 */
$collection = new WordsCollection();
$collection-&gt;addItem(&quot;First&quot;);
$collection-&gt;addItem(&quot;Second&quot;);
$collection-&gt;addItem(&quot;Third&quot;);

echo &quot;Straight traversal:\n&quot;;
foreach ($collection-&gt;getIterator() as $item) {
    echo $item . &quot;\n&quot;;
}

echo &quot;\n&quot;;
echo &quot;Reverse traversal:\n&quot;;
foreach ($collection-&gt;getReverseIterator() as $item) {
    echo $item . &quot;\n&quot;;
}</code></pre>
</figure>

#### []{#example-0--Output-txt .anchor} **Output.txt:** Resultado de la ejecución

<figure class="code">
<pre class="code" lang="output"><code>Straight traversal:
First
Second
Third

Reverse traversal:
Third
Second
First</code></pre>
</figure>
:::

::: {#example-1 .tab-pane role="tabpanel" aria-labelledby="example-1-tab" style="padding-top: 24px"}
## Ejemplo del mundo real {#example-1-title}

Como PHP ya cuenta con una interfaz **Iteradora** integrada que
proporciona una práctica integración con bucles *foreach*, es muy fácil
crear tus propios iteradores para recorrer casi cualquier estructura de
datos imaginable.

Este ejemplo del patrón Iterator proporciona un acceso fácil a archivos
CSV.

#### []{#example-1--index-php .anchor} **index.php:** Ejemplo del mundo real

<figure class="code">
<pre class="code" lang="php"><code>&lt;?php

namespace RefactoringGuru\Iterator\RealWorld;

/**
 * CSV File Iterator.
 *
 * @author Josh Lockhart
 */
class CsvIterator implements \Iterator
{
    const ROW_SIZE = 4096;

    /**
     * The pointer to the CSV file.
     *
     * @var resource
     */
    protected $filePointer = null;

    /**
     * The current element, which is returned on each iteration.
     *
     * @var array
     */
    protected $currentElement = null;

    /**
     * The row counter.
     *
     * @var int
     */
    protected $rowCounter = null;

    /**
     * The delimiter for the CSV file.
     *
     * @var string
     */
    protected $delimiter = null;

    /**
     * The constructor tries to open the CSV file. It throws an exception on
     * failure.
     *
     * @param string $file The CSV file.
     * @param string $delimiter The delimiter.
     *
     * @throws \Exception
     */
    public function __construct($file, $delimiter = &#39;,&#39;)
    {
        try {
            $this-&gt;filePointer = fopen($file, &#39;rb&#39;);
            $this-&gt;delimiter = $delimiter;
        } catch (\Exception $e) {
            throw new \Exception(&#39;The file &quot;&#39; . $file . &#39;&quot; cannot be read.&#39;);
        }
    }

    /**
     * This method resets the file pointer.
     */
    public function rewind(): void
    {
        $this-&gt;rowCounter = 0;
        rewind($this-&gt;filePointer);
        // Read the first row to initialize
        $this-&gt;currentElement = fgetcsv($this-&gt;filePointer, self::ROW_SIZE, $this-&gt;delimiter);
    }

    /**
     * This method returns the current CSV row as a 2-dimensional array.
     *
     * @return array The current CSV row as a 2-dimensional array.
     */
    public function current(): array
    {
        return $this-&gt;currentElement ?: [];
    }

    /**
     * This method returns the current row number.
     *
     * @return int The current row number.
     */
    public function key(): int
    {
        return $this-&gt;rowCounter;
    }

    /**
     * This method moves to the next element.
     */
    public function next(): void
    {
        if (is_resource($this-&gt;filePointer)) {
            $this-&gt;currentElement = fgetcsv($this-&gt;filePointer, self::ROW_SIZE, $this-&gt;delimiter);
            $this-&gt;rowCounter++;
        }
    }

    /**
     * This method checks if the current position is valid.
     *
     * @return bool If the current position is valid.
     */
    public function valid(): bool
    {
        if ($this-&gt;currentElement === false) {
            if (is_resource($this-&gt;filePointer)) {
                fclose($this-&gt;filePointer);
            }

            return false;
        }

        return is_resource($this-&gt;filePointer);
    }
}

/**
 * The client code.
 */
$csv = new CsvIterator(__DIR__ . &#39;/cats.csv&#39;);

foreach ($csv as $key =&gt; $row) {
    print_r($row);
}</code></pre>
</figure>

#### []{#example-1--Output-txt .anchor} **Output.txt:** Resultado de la ejecución

<figure class="code">
<pre class="code" lang="output"><code>Array
(
    [0] =&gt; Name
    [1] =&gt; Age
    [2] =&gt; Owner
    [3] =&gt; Breed
    [4] =&gt; Image
    [5] =&gt; Color
    [6] =&gt; Texture
    [7] =&gt; Fur
    [8] =&gt; Size
)
Array
(
    [0] =&gt; Steve
    [1] =&gt; 3
    [2] =&gt; Alexander Shvets
    [3] =&gt; Bengal
    [4] =&gt; /cats/bengal.jpg
    [5] =&gt; Brown
    [6] =&gt; Stripes
    [7] =&gt; Short
    [8] =&gt; Medium
)
Array
(
    [0] =&gt; Siri
    [1] =&gt; 2
    [2] =&gt; Alexander Shvets
    [3] =&gt; Domestic short-haired
    [4] =&gt; /cats/domestic-sh.jpg
    [5] =&gt; Black
    [6] =&gt; Solid
    [7] =&gt; Medium
    [8] =&gt; Medium
)
Array
(
    [0] =&gt; Fluffy
    [1] =&gt; 5
    [2] =&gt; John Smith
    [3] =&gt; Maine Coon
    [4] =&gt; /cats/Maine-Coon.jpg
    [5] =&gt; Gray
    [6] =&gt; Stripes
    [7] =&gt; Long
    [8] =&gt; Large
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

[Mediator en PHP []{.fa
.fa-arrow-right}](../../mediator/php/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Volver

[[]{.fa .fa-arrow-left} Command en
PHP](../../command/php/example.html){.btn .btn-default rel="prev"}
:::

## **Iterator** en otros lenguajes

[![Iterator en
C#](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Iterator en C#"){.prog-lang-link}
[![Iterator en
C++](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Iterator en C++"){.prog-lang-link}
[![Iterator en
Go](../../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Iterator en Go"){.prog-lang-link}
[![Iterator en
Java](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Iterator en Java"){.prog-lang-link}
[![Iterator en
Python](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Iterator en Python"){.prog-lang-link}
[![Iterator en
Ruby](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Iterator en Ruby"){.prog-lang-link}
[![Iterator en
Rust](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Iterator en Rust"){.prog-lang-link}
[![Iterator en
Swift](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Iterator en Swift"){.prog-lang-link}
[![Iterator en
TypeScript](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Iterator en TypeScript"){.prog-lang-link}
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
