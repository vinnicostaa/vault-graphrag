:::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [Wzorce
projektowe](../../../design-patterns.html){.type} /
[Iterator](../../iterator.html){.type} / [PHP](../../php.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Iterator](../../../../images/patterns/cards/iterator-minie2c2.png?id=76c28bb48f997b36965983dd2b41f02e){srcset="/images/patterns/cards/iterator-mini-2x.png?id=7d28f66a487066774a743cfceaa40ea4 2x"}
:::

# **Iterator** w języku PHP {#iterator-w-języku-php .pattern-example-title-block-title}
::::

:::::::::::::: pattern-example-body
::: pattern-example-brief
**Iterator** to behawioralny wzorzec projektowy pozwalający sekwencyjnie
przechodzić od elementu do elementu jakiegoś zbioru bez konieczności
eksponowania jego formy.

Dzięki Iteratorowi klienci mogą przeglądać kolejne elementy różnych
kolekcji w podobny sposób, za pośrednictwem jednego interfejsu.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Dowiedz się więcej o Iterator ](../../iterator.html){.btn .btn-lg
.btn-primary}
:::

::::::::::: {.sidebar-navigation .with-scroll}
::: en-title
Nawigacja
:::

::: en-intro
 [Intro](#)
:::

::: en-intro
 [Przykład koncepcyjny](#example-0)
:::

::: en-file
 [index](#example-0--index-php)
:::

::: en-file
 [Output](#example-0--Output-txt)
:::

::: en-intro
 [Przykład z prawdziwego życia](#example-1)
:::

::: en-file
 [index](#example-1--index-php)
:::

::: en-file
 [Output](#example-1--Output-txt)
:::
:::::::::::

**Złożoność:** []{.fa-stars}

**Popularność:** []{.fa-stars}

**Przypadki użycia:** Wzorzec Iterator jest bardzo rozpowszechniony w
kodzie PHP. Wiele frameworków i bibliotek pozwala za jego pomocą
poruszać się po elementach ich kolekcji.

PHP posiada wbudowany interfejs [Iterator](../../iterator.html) na
podstawie którego można budować własne iteratory kompatybilne z resztą
kodu PHP.

**Identyfikacja:** Iterator łatwo rozpoznać po obecności metod
nawigacyjnych (takich jak `następny`, `poprzedni` i innych). Kod klienta
stosujący iteratory może nie mieć bezpośredniego dostępu do badanej
kolekcji.
::::::::::::::

::: {#nav-tab .nav .nav-tabs .nav-justified role="tablist"}
[Przykład koncepcyjny](#example-0){#example-0-tab .nav-item .nav-link
.active toggle="tab" role="tab" aria-controls="example-0"
aria-selected="true"} [Przykład z prawdziwego
życia](#example-1){#example-1-tab .nav-item .nav-link toggle="tab"
role="tab" aria-controls="example-1" aria-selected="true"}
:::

::::: {#nav-tabContent .pattern-example-tab-content .tab-content}
::: {#example-0 .tab-pane .show .active role="tabpanel" aria-labelledby="example-0-tab" style="padding-top: 24px"}
## Przykład koncepcyjny {#example-0-title}

Poniższy przykład ilustruje strukturę wzorca **Iterator** ze szczególnym
naciskiem na następujące kwestie:

-   Z jakich składa się klas?
-   Jakie role pełnią te klasy?
-   W jaki sposób elementy wzorca są ze sobą powiązane?

Poznawszy strukturę wzorca będzie ci łatwiej zrozumieć następujący
przykład, oparty na prawdziwym przypadku użycia PHP.

#### []{#example-0--index-php .anchor} **index.php:** Przykład koncepcyjny

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

#### []{#example-0--Output-txt .anchor} **Output.txt:** Wynik działania

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
## Przykład z prawdziwego życia {#example-1-title}

Ponieważ PHP posiada wbudowany interfejs **Iterator**, pozwalający na
wygodną integrację z pętlami `foreach`, łatwo stworzyć własne iteratory
mogące przechodzić sekwencyjnie po elementach niemal każdej struktury
danych, jaką można sobie wyobrazić.

Poniższy przykład użycia wzorca Iterator daje łatwy dostęp do plików
CSV.

#### []{#example-1--index-php .anchor} **index.php:** Przykład z prawdziwego życia

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

#### []{#example-1--Output-txt .anchor} **Output.txt:** Wynik działania

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
[Przykład koncepcyjny](#example-0){#example-0-tab-bottom .nav-item
.nav-link .active toggle="tab" role="tab" aria-controls="example-0"
aria-selected="true"} [Przykład z prawdziwego
życia](#example-1){#example-1-tab-bottom .nav-item .nav-link
toggle="tab" role="tab" aria-controls="example-1" aria-selected="true"}
:::

::: next
#### Czytaj dalej

[Mediator w języku PHP []{.fa
.fa-arrow-right}](../../mediator/php/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Wróć

[[]{.fa .fa-arrow-left} Polecenie w języku
PHP](../../command/php/example.html){.btn .btn-default rel="prev"}
:::

## **Iterator** w innych językach

[![Iterator w języku
C#](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Iterator w języku C#"){.prog-lang-link}
[![Iterator w języku
C++](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Iterator w języku C++"){.prog-lang-link}
[![Iterator w języku
Go](../../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Iterator w języku Go"){.prog-lang-link}
[![Iterator w języku
Java](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Iterator w języku Java"){.prog-lang-link}
[![Iterator w języku
Python](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Iterator w języku Python"){.prog-lang-link}
[![Iterator w języku
Ruby](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Iterator w języku Ruby"){.prog-lang-link}
[![Iterator w języku
Rust](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Iterator w języku Rust"){.prog-lang-link}
[![Iterator w języku
Swift](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Iterator w języku Swift"){.prog-lang-link}
[![Iterator w języku
TypeScript](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Iterator w języku TypeScript"){.prog-lang-link}
:::::::::::::::::::::::::

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
:::::::::::::::::::::::::::::::
::::::::::::::::::::::::::::::::
