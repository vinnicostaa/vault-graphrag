:::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [디자인
패턴들](../../../design-patterns.html){.type} /
[반복자](../../iterator.html){.type} / [PHP](../../php.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![반복자](../../../../images/patterns/cards/iterator-minie2c2.png?id=76c28bb48f997b36965983dd2b41f02e){srcset="/images/patterns/cards/iterator-mini-2x.png?id=7d28f66a487066774a743cfceaa40ea4 2x"}
:::

# PHP로 작성된 **반복자** {#php로-작성된-반복자 .pattern-example-title-block-title}
::::

:::::::::::::: pattern-example-body
::: pattern-example-brief
**반복자**는 복잡한 데이터 구조의 내부 세부 정보를 노출하지 않고 해당
구조를 차례대로 순회할 수 있도록 하는 행동 디자인 패턴입니다.

반복자 덕분에 클라이언트들은 단일 반복기 인터페이스를 사용하여 유사한
방식으로 다른 컬렉션들의 요소들을 탐색할 수 있습니다.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ 반복자에 대하여 더 자세히 알아보세요 ](../../iterator.html){.btn
.btn-lg .btn-primary}
:::

::::::::::: {.sidebar-navigation .with-scroll}
::: en-title
내비게이션
:::

::: en-intro
 [소개](#)
:::

::: en-intro
 [개념적인 예시](#example-0)
:::

::: en-file
 [index](#example-0--index-php)
:::

::: en-file
 [Output](#example-0--Output-txt)
:::

::: en-intro
 [구상적인 실제 예시](#example-1)
:::

::: en-file
 [index](#example-1--index-php)
:::

::: en-file
 [Output](#example-1--Output-txt)
:::
:::::::::::

**복잡도:** []{.fa-stars}

**인기도:** []{.fa-stars}

**적용 예시들:** 이 패턴은 PHP에 자주 사용됩니다. 많은 프레임워크들과
라이브러리들은 이 패턴을 컬렉션을 탐색하는 표준 방법을 제공하기 위해
사용합니다.

PHP 언어에는 내장된 [반복자](../../iterator.html) 인터페이스가 있습니다.
이 인터페이스는 나머지 PHP 코드와 호환되는 사용자 정의된 반복자를
구축하는 데 사용할 수 있습니다.

**식별법:** 반복자는 그의 탐색 메서드들​(예: `next`, `previous` 등)​로
쉽게 인식할 수 있습니다. 또 반복자를 사용하는 클라이언트 코드는 반복자가
순회하는 컬렉션을 직접 접근하지 못할 수도 있습니다.
::::::::::::::

::: {#nav-tab .nav .nav-tabs .nav-justified role="tablist"}
[개념적인 예시](#example-0){#example-0-tab .nav-item .nav-link .active
toggle="tab" role="tab" aria-controls="example-0" aria-selected="true"}
[구상적인 실제 예시](#example-1){#example-1-tab .nav-item .nav-link
toggle="tab" role="tab" aria-controls="example-1" aria-selected="true"}
:::

::::: {#nav-tabContent .pattern-example-tab-content .tab-content}
::: {#example-0 .tab-pane .show .active role="tabpanel" aria-labelledby="example-0-tab" style="padding-top: 24px"}
## 개념적인 예시 {#example-0-title}

이 예시는 **반복자** 디자인 패턴의 구조를 보여주고 다음 질문에 중점을
둡니다:

-   패턴은 어떤 클래스들로 구성되어 있나요?
-   이 클래스들은 어떤 역할을 하나요?
-   패턴의 요소들은 어떻게 서로 연관되어 있나요?

이 패턴의 구조를 배우면 실제 PHP 사용 사례를 기반으로 하는 다음 예시를
더욱 쉽게 이해할 수 있을 것입니다.

#### []{#example-0--index-php .anchor} **index.php:** 개념적인 예시

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

#### []{#example-0--Output-txt .anchor} **Output.txt:** 실행 결과

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
## 구상적인 실제 예시 {#example-1-title}

PHP에는 이미 foreach 루프와의 편리한 통합을 제공하는 내장된 **반복자**
인터페이스가 있기 때문에 상상할 수 있는 거의 모든 종류의 데이터 구조를
순회하기 위한 자체 반복자를 쉽게 만들 수 있습니다.

이 반복자 패턴의 예시에서는 사용자가 CSV 파일에 쉽게 접근할 수 있도록
합니다.

#### []{#example-1--index-php .anchor} **index.php:** 개념적인 예시

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

#### []{#example-1--Output-txt .anchor} **Output.txt:** 실행 결과

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
[개념적인 예시](#example-0){#example-0-tab-bottom .nav-item .nav-link
.active toggle="tab" role="tab" aria-controls="example-0"
aria-selected="true"} [구상적인 실제
예시](#example-1){#example-1-tab-bottom .nav-item .nav-link toggle="tab"
role="tab" aria-controls="example-1" aria-selected="true"}
:::

::: next
#### 다음을 보세요

[PHP로 작성된 중재자 []{.fa
.fa-arrow-right}](../../mediator/php/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### 돌아가기

[[]{.fa .fa-arrow-left} PHP로 작성된
커맨드](../../command/php/example.html){.btn .btn-default rel="prev"}
:::

## 다른 언어로 작성된 **반복자**

[![C#으로 작성된
반복자](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "C#으로 작성된 반복자"){.prog-lang-link}
[![C++로 작성된
반복자](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "C++로 작성된 반복자"){.prog-lang-link}
[![Go로 작성된
반복자](../../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Go로 작성된 반복자"){.prog-lang-link}
[![자바로 작성된
반복자](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "자바로 작성된 반복자"){.prog-lang-link}
[![파이썬으로 작성된
반복자](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "파이썬으로 작성된 반복자"){.prog-lang-link}
[![루비로 작성된
반복자](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "루비로 작성된 반복자"){.prog-lang-link}
[![러스트로 작성된
반복자](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "러스트로 작성된 반복자"){.prog-lang-link}
[![스위프트로 작성된
반복자](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "스위프트로 작성된 반복자"){.prog-lang-link}
[![타입스크립트로 작성된
반복자](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "타입스크립트로 작성된 반복자"){.prog-lang-link}
:::::::::::::::::::::::::

::::::: {.prom .banner-sidebar .banner .banner-striped .banner-examples .banner-removable .banner-removable-patterns data-id="DP: Examples-IDE" creative-id="examples-sidebar-ko" position="sidebar"}
:::::: {.banner-inner style="font-size: 14px; line-height: 21px;"}
::: banner-examples-img
[![](../../../../images/patterns/banners/examples-ide91ca.png?id=3115b4b548fb96b75974e2de8f4f49bc){width="215"
height="158" loading="lazy"
srcset="/images/patterns/banners/examples-ide-2x.png?id=93c007a6d157b97c28eb213f59d716ae 2x"}](../../book.html)
:::

::: banner-examples-fade
[ 예시들을 포함한 저장소](../../book.html){.btn .btn-secondary
.btn-block}
:::

::: {.banner-examples-text style="margin-top: 1rem;"}
전자책 **디자인 패턴에 뛰어들기**를 구매하면 IDE에서 바로 열 수 있는
수십 가지의 상세한 코드 예시들이 포함된 저장소를 사용할 수 있습니다.

[ 더 알아보기...](../../book.html){.btn .btn-secondary .btn-block}
:::
::::::
:::::::
:::::::::::::::::::::::::::::::
::::::::::::::::::::::::::::::::
