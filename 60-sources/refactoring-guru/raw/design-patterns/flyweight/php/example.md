:::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../index.html){.home} / [Design
Patterns](../../../design-patterns.html){.type} /
[Flyweight](../../flyweight.html){.type} / [PHP](../../php.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Flyweight](../../../images/patterns/cards/flyweight-minie3d8.png?id=422ca8d2f90614dce810a8812c626698){srcset="/images/patterns/cards/flyweight-mini-2x.png?id=857ca676fff84317bb0dab76abfce08e 2x"}
:::

# **Flyweight** in PHP {#flyweight-in-php .pattern-example-title-block-title}
::::

:::::::::::::: pattern-example-body
::: pattern-example-brief
**Flyweight** is a structural design pattern that allows programs to
support vast quantities of objects by keeping their memory consumption
low.

The pattern achieves it by sharing parts of object state between
multiple objects. In other words, the Flyweight saves RAM by caching the
same data used by different objects.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Learn more about Flyweight ](../../flyweight.html){.btn .btn-lg
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
 [Conceptual Example](#example-0)
:::

::: en-file
 [index](#example-0--index-php)
:::

::: en-file
 [Output](#example-0--Output-txt)
:::

::: en-intro
 [Real World Example](#example-1)
:::

::: en-file
 [index](#example-1--index-php)
:::

::: en-file
 [Output](#example-1--Output-txt)
:::
:::::::::::

**Complexity:** []{.fa-stars}

**Popularity:** []{.fa-stars}

**Usage examples:** The Flyweight pattern is especially rarely used in
PHP applications due to the very nature of the language. A PHP script
typically works with a part of the application's data and never loads
all of it into the memory at the same time.

**Identification:** Flyweight can be recognized by a creation method
that returns cached objects instead of creating new.
::::::::::::::

::: {#nav-tab .nav .nav-tabs .nav-justified role="tablist"}
[Conceptual Example](#example-0){#example-0-tab .nav-item .nav-link
.active toggle="tab" role="tab" aria-controls="example-0"
aria-selected="true"} [Real World Example](#example-1){#example-1-tab
.nav-item .nav-link toggle="tab" role="tab" aria-controls="example-1"
aria-selected="true"}
:::

::::: {#nav-tabContent .pattern-example-tab-content .tab-content}
::: {#example-0 .tab-pane .show .active role="tabpanel" aria-labelledby="example-0-tab" style="padding-top: 24px"}
## Conceptual Example {#example-0-title}

This example illustrates the structure of the **Flyweight** design
pattern and focuses on the following questions:

-   What classes does it consist of?
-   What roles do these classes play?
-   In what way the elements of the pattern are related?

After learning about the pattern's structure it'll be easier for you to
grasp the following example, based on a real-world PHP use case.

#### []{#example-0--index-php .anchor} **index.php:** Conceptual example

<figure class="code">
<pre class="code" lang="php"><code>&lt;?php

namespace RefactoringGuru\Flyweight\Conceptual;

/**
 * The Flyweight stores a common portion of the state (also called intrinsic
 * state) that belongs to multiple real business entities. The Flyweight accepts
 * the rest of the state (extrinsic state, unique for each entity) via its
 * method parameters.
 */
class Flyweight
{
    private $sharedState;

    public function __construct($sharedState)
    {
        $this-&gt;sharedState = $sharedState;
    }

    public function operation($uniqueState): void
    {
        $s = json_encode($this-&gt;sharedState);
        $u = json_encode($uniqueState);
        echo &quot;Flyweight: Displaying shared ($s) and unique ($u) state.\n&quot;;
    }
}

/**
 * The Flyweight Factory creates and manages the Flyweight objects. It ensures
 * that flyweights are shared correctly. When the client requests a flyweight,
 * the factory either returns an existing instance or creates a new one, if it
 * doesn&#39;t exist yet.
 */
class FlyweightFactory
{
    /**
     * @var Flyweight[]
     */
    private $flyweights = [];

    public function __construct(array $initialFlyweights)
    {
        foreach ($initialFlyweights as $state) {
            $this-&gt;flyweights[$this-&gt;getKey($state)] = new Flyweight($state);
        }
    }

    /**
     * Returns a Flyweight&#39;s string hash for a given state.
     */
    private function getKey(array $state): string
    {
        ksort($state);

        return implode(&quot;_&quot;, $state);
    }

    /**
     * Returns an existing Flyweight with a given state or creates a new one.
     */
    public function getFlyweight(array $sharedState): Flyweight
    {
        $key = $this-&gt;getKey($sharedState);

        if (!isset($this-&gt;flyweights[$key])) {
            echo &quot;FlyweightFactory: Can&#39;t find a flyweight, creating new one.\n&quot;;
            $this-&gt;flyweights[$key] = new Flyweight($sharedState);
        } else {
            echo &quot;FlyweightFactory: Reusing existing flyweight.\n&quot;;
        }

        return $this-&gt;flyweights[$key];
    }

    public function listFlyweights(): void
    {
        $count = count($this-&gt;flyweights);
        echo &quot;\nFlyweightFactory: I have $count flyweights:\n&quot;;
        foreach ($this-&gt;flyweights as $key =&gt; $flyweight) {
            echo $key . &quot;\n&quot;;
        }
    }
}

/**
 * The client code usually creates a bunch of pre-populated flyweights in the
 * initialization stage of the application.
 */
$factory = new FlyweightFactory([
    [&quot;Chevrolet&quot;, &quot;Camaro2018&quot;, &quot;pink&quot;],
    [&quot;Mercedes Benz&quot;, &quot;C300&quot;, &quot;black&quot;],
    [&quot;Mercedes Benz&quot;, &quot;C500&quot;, &quot;red&quot;],
    [&quot;BMW&quot;, &quot;M5&quot;, &quot;red&quot;],
    [&quot;BMW&quot;, &quot;X6&quot;, &quot;white&quot;],
    // ...
]);
$factory-&gt;listFlyweights();

// ...

function addCarToPoliceDatabase(
    FlyweightFactory $ff,
    $plates,
    $owner,
    $brand,
    $model,
    $color
) {
    echo &quot;\nClient: Adding a car to database.\n&quot;;
    $flyweight = $ff-&gt;getFlyweight([$brand, $model, $color]);

    // The client code either stores or calculates extrinsic state and passes it
    // to the flyweight&#39;s methods.
    $flyweight-&gt;operation([$plates, $owner]);
}

addCarToPoliceDatabase(
    $factory,
    &quot;CL234IR&quot;,
    &quot;James Doe&quot;,
    &quot;BMW&quot;,
    &quot;M5&quot;,
    &quot;red&quot;,
);

addCarToPoliceDatabase(
    $factory,
    &quot;CL234IR&quot;,
    &quot;James Doe&quot;,
    &quot;BMW&quot;,
    &quot;X1&quot;,
    &quot;red&quot;,
);

$factory-&gt;listFlyweights();</code></pre>
</figure>

#### []{#example-0--Output-txt .anchor} **Output.txt:** Execution result

<figure class="code">
<pre class="code" lang="output"><code>FlyweightFactory: I have 5 flyweights:
Chevrolet_Camaro2018_pink
Mercedes Benz_C300_black
Mercedes Benz_C500_red
BMW_M5_red
BMW_X6_white

Client: Adding a car to database.
FlyweightFactory: Reusing existing flyweight.
Flyweight: Displaying shared ([&quot;BMW&quot;,&quot;M5&quot;,&quot;red&quot;]) and unique ([&quot;CL234IR&quot;,&quot;James Doe&quot;]) state.

Client: Adding a car to database.
FlyweightFactory: Can&#39;t find a flyweight, creating new one.
Flyweight: Displaying shared ([&quot;BMW&quot;,&quot;X1&quot;,&quot;red&quot;]) and unique ([&quot;CL234IR&quot;,&quot;James Doe&quot;]) state.

FlyweightFactory: I have 6 flyweights:
Chevrolet_Camaro2018_pink
Mercedes Benz_C300_black
Mercedes Benz_C500_red
BMW_M5_red
BMW_X6_white
BMW_X1_red</code></pre>
</figure>
:::

::: {#example-1 .tab-pane role="tabpanel" aria-labelledby="example-1-tab" style="padding-top: 24px"}
## Real World Example {#example-1-title}

Before we begin, please note that real applications for the
**Flyweight** pattern in PHP are pretty scarce. This stems from the
single-thread nature of PHP, where you're not supposed to be storing ALL
of your application's objects in memory at the same time, in the same
thread. While the idea for this example is only half-serious, and the
whole RAM problem might be solved by structuring the app differently, it
still demonstrates the concept of the pattern as it works in the real
world. All right, I've given you the disclaimer. Now, let's begin.

In this example, the Flyweight pattern is used to minimize the RAM usage
of objects in an animal database of a cat-only veterinary clinic. Each
record in the database is represented by a `Cat` object. Its data
consists of two parts:

1.  Unique (extrinsic) data such as a pet's name, age, and owner info.
2.  Shared (intrinsic) data such as breed name, color, texture, etc.

The first part is stored directly inside the `Cat` class, which acts as
a context. The second part, however, is stored separately and can be
shared by multiple cats. This shareable data resides inside the
`CatVariation` class. All cats that have similar features are linked to
the same `CatVariation` class, instead of storing the duplicate data in
each of their objects.

#### []{#example-1--index-php .anchor} **index.php:** Real world example

<figure class="code">
<pre class="code" lang="php"><code>&lt;?php

namespace RefactoringGuru\Flyweight\RealWorld;

/**
 * Flyweight objects represent the data shared by multiple Cat objects. This is
 * the combination of breed, color, texture, etc.
 */
class CatVariation
{
    /**
     * The so-called &quot;intrinsic&quot; state.
     */
    public $breed;

    public $image;

    public $color;

    public $texture;

    public $fur;

    public $size;

    public function __construct(
        string $breed,
        string $image,
        string $color,
        string $texture,
        string $fur,
        string $size
    ) {
        $this-&gt;breed = $breed;
        $this-&gt;image = $image;
        $this-&gt;color = $color;
        $this-&gt;texture = $texture;
        $this-&gt;fur = $fur;
        $this-&gt;size = $size;
    }

    /**
     * This method displays the cat information. The method accepts the
     * extrinsic  state as arguments. The rest of the state is stored inside
     * Flyweight&#39;s fields.
     *
     * You might be wondering why we had put the primary cat&#39;s logic into the
     * CatVariation class instead of keeping it in the Cat class. I agree, it
     * does sound confusing.
     *
     * Keep in mind that in the real world, the Flyweight pattern can either be
     * implemented from the start or forced onto an existing application
     * whenever the developers realize they&#39;ve hit upon a RAM problem.
     *
     * In the latter case, you end up with such classes as we have here. We kind
     * of &quot;refactored&quot; an ideal app where all the data was initially inside the
     * Cat class. If we had implemented the Flyweight from the start, our class
     * names might be different and less confusing. For example, Cat and
     * CatContext.
     *
     * However, the actual reason why the primary behavior should live in the
     * Flyweight class is that you might not have the Context class declared at
     * all. The context data might be stored in an array or some other more
     * efficient data structure. You won&#39;t have another place to put your
     * methods in, except the Flyweight class.
     */
    public function renderProfile(string $name, string $age, string $owner): void
    {
        echo &quot;= $name =\n&quot;;
        echo &quot;Age: $age\n&quot;;
        echo &quot;Owner: $owner\n&quot;;
        echo &quot;Breed: $this-&gt;breed\n&quot;;
        echo &quot;Image: $this-&gt;image\n&quot;;
        echo &quot;Color: $this-&gt;color\n&quot;;
        echo &quot;Texture: $this-&gt;texture\n&quot;;
    }
}

/**
 * The context stores the data unique for each cat.
 *
 * A designated class for storing context is optional and not always viable. The
 * context may be stored inside a massive data structure within the Client code
 * and passed to the flyweight methods when needed.
 */
class Cat
{
    /**
     * The so-called &quot;extrinsic&quot; state.
     */
    public $name;

    public $age;

    public $owner;

    /**
     * @var CatVariation
     */
    private $variation;

    public function __construct(string $name, string $age, string $owner, CatVariation $variation)
    {
        $this-&gt;name = $name;
        $this-&gt;age = $age;
        $this-&gt;owner = $owner;
        $this-&gt;variation = $variation;
    }

    /**
     * Since the Context objects don&#39;t own all of their state, sometimes, for
     * the sake of convenience, you may need to implement some helper methods
     * (for example, for comparing several Context objects.)
     */
    public function matches(array $query): bool
    {
        foreach ($query as $key =&gt; $value) {
            if (property_exists($this, $key)) {
                if ($this-&gt;$key != $value) {
                    return false;
                }
            } elseif (property_exists($this-&gt;variation, $key)) {
                if ($this-&gt;variation-&gt;$key != $value) {
                    return false;
                }
            } else {
                return false;
            }
        }

        return true;
    }

    /**
     * The Context might also define several shortcut methods, that delegate
     * execution to the Flyweight object. These methods might be remnants of
     * real methods, extracted to the Flyweight class during a massive
     * refactoring to the Flyweight pattern.
     */
    public function render(): void
    {
        $this-&gt;variation-&gt;renderProfile($this-&gt;name, $this-&gt;age, $this-&gt;owner);
    }
}

/**
 * The Flyweight Factory stores both the Context and Flyweight objects,
 * effectively hiding any notion of the Flyweight pattern from the client.
 */
class CatDataBase
{
    /**
     * The list of cat objects (Contexts).
     */
    private $cats = [];

    /**
     * The list of cat variations (Flyweights).
     */
    private $variations = [];

    /**
     * When adding a cat to the database, we look for an existing cat variation
     * first.
     */
    public function addCat(
        string $name,
        string $age,
        string $owner,
        string $breed,
        string $image,
        string $color,
        string $texture,
        string $fur,
        string $size
    ) {
        $variation =
            $this-&gt;getVariation($breed, $image, $color, $texture, $fur, $size);
        $this-&gt;cats[] = new Cat($name, $age, $owner, $variation);
        echo &quot;CatDataBase: Added a cat ($name, $breed).\n&quot;;
    }

    /**
     * Return an existing variation (Flyweight) by given data or create a new
     * one if it doesn&#39;t exist yet.
     */
    public function getVariation(
        string $breed,
        string $image,
        string $color,
        string $texture,
        string $fur,
        string $size
    ): CatVariation {
        $key = $this-&gt;getKey(get_defined_vars());

        if (!isset($this-&gt;variations[$key])) {
            $this-&gt;variations[$key] =
                new CatVariation($breed, $image, $color, $texture, $fur, $size);
        }

        return $this-&gt;variations[$key];
    }

    /**
     * This function helps to generate unique array keys.
     */
    private function getKey(array $data): string
    {
        return md5(implode(&quot;_&quot;, $data));
    }

    /**
     * Look for a cat in the database using the given query parameters.
     */
    public function findCat(array $query)
    {
        foreach ($this-&gt;cats as $cat) {
            if ($cat-&gt;matches($query)) {
                return $cat;
            }
        }
        echo &quot;CatDataBase: Sorry, your query does not yield any results.&quot;;
    }
}

/**
 * The client code.
 */
$db = new CatDataBase();

echo &quot;Client: Let&#39;s see what we have in \&quot;cats.csv\&quot;.\n&quot;;

// To see the real effect of the pattern, you should have a large database with
// several millions of records. Feel free to experiment with code to see the
// real extent of the pattern.
$handle = fopen(__DIR__ . &quot;/cats.csv&quot;, &quot;r&quot;);
$row = 0;
$columns = [];
while (($data = fgetcsv($handle)) !== false) {
    if ($row == 0) {
        for ($c = 0; $c &lt; count($data); $c++) {
            $columnIndex = $c;
            $columnKey = strtolower($data[$c]);
            $columns[$columnKey] = $columnIndex;
        }
        $row++;
        continue;
    }

    $db-&gt;addCat(
        $data[$columns[&#39;name&#39;]],
        $data[$columns[&#39;age&#39;]],
        $data[$columns[&#39;owner&#39;]],
        $data[$columns[&#39;breed&#39;]],
        $data[$columns[&#39;image&#39;]],
        $data[$columns[&#39;color&#39;]],
        $data[$columns[&#39;texture&#39;]],
        $data[$columns[&#39;fur&#39;]],
        $data[$columns[&#39;size&#39;]],
    );
    $row++;
}
fclose($handle);

// ...

echo &quot;\nClient: Let&#39;s look for a cat named \&quot;Siri\&quot;.\n&quot;;
$cat = $db-&gt;findCat([&#39;name&#39; =&gt; &quot;Siri&quot;]);
if ($cat) {
    $cat-&gt;render();
}

echo &quot;\nClient: Let&#39;s look for a cat named \&quot;Bob\&quot;.\n&quot;;
$cat = $db-&gt;findCat([&#39;name&#39; =&gt; &quot;Bob&quot;]);
if ($cat) {
    $cat-&gt;render();
}</code></pre>
</figure>

#### []{#example-1--Output-txt .anchor} **Output.txt:** Execution result

<figure class="code">
<pre class="code" lang="output"><code>Client: Let&#39;s see what we have in &quot;cats.csv&quot;.
CatDataBase: Added a cat (Steve, Bengal).
CatDataBase: Added a cat (Siri, Domestic short-haired).
CatDataBase: Added a cat (Fluffy, Maine Coon).

Client: Let&#39;s look for a cat named &quot;Siri&quot;.
= Siri =
Age: 2
Owner: Alexander Shvets
Breed: Domestic short-haired
Image: /cats/domestic-sh.jpg
Color: Black
Texture: Solid

Client: Let&#39;s look for a cat named &quot;Bob&quot;.
CatDataBase: Sorry, your query does not yield any results.</code></pre>
</figure>
:::
:::::

::: {#nav-tab .nav .nav-tabs .nav-tabs-bottom .nav-justified role="tablist"}
[Conceptual Example](#example-0){#example-0-tab-bottom .nav-item
.nav-link .active toggle="tab" role="tab" aria-controls="example-0"
aria-selected="true"} [Real World
Example](#example-1){#example-1-tab-bottom .nav-item .nav-link
toggle="tab" role="tab" aria-controls="example-1" aria-selected="true"}
:::

::: next
#### Read next

[Proxy in PHP []{.fa
.fa-arrow-right}](../../proxy/php/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Return

[[]{.fa .fa-arrow-left} Facade in
PHP](../../facade/php/example.html){.btn .btn-default rel="prev"}
:::

## **Flyweight** in Other Languages

[![Flyweight in
C#](../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Flyweight in C#"){.prog-lang-link}
[![Flyweight in
C++](../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Flyweight in C++"){.prog-lang-link}
[![Flyweight in
Go](../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Flyweight in Go"){.prog-lang-link}
[![Flyweight in
Java](../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Flyweight in Java"){.prog-lang-link}
[![Flyweight in
Python](../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Flyweight in Python"){.prog-lang-link}
[![Flyweight in
Ruby](../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Flyweight in Ruby"){.prog-lang-link}
[![Flyweight in
Rust](../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Flyweight in Rust"){.prog-lang-link}
[![Flyweight in
Swift](../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Flyweight in Swift"){.prog-lang-link}
[![Flyweight in
TypeScript](../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Flyweight in TypeScript"){.prog-lang-link}
:::::::::::::::::::::::::

::::::: {.prom .banner-sidebar .banner .banner-striped .banner-examples .banner-removable .banner-removable-patterns data-id="DP: Examples-IDE" creative-id="examples-sidebar-en" position="sidebar"}
:::::: {.banner-inner style="font-size: 14px; line-height: 21px;"}
::: banner-examples-img
[![](../../../images/patterns/banners/examples-ide91ca.png?id=3115b4b548fb96b75974e2de8f4f49bc){width="215"
height="158" loading="lazy"
srcset="/images/patterns/banners/examples-ide-2x.png?id=93c007a6d157b97c28eb213f59d716ae 2x"}](../../book.html)
:::

::: banner-examples-fade
[ Archive with examples](../../book.html){.btn .btn-secondary
.btn-block}
:::

::: {.banner-examples-text style="margin-top: 1rem;"}
Buy the eBook **Dive Into Design Patterns** and get the access to
archive with dozens of detailed examples that can be opened right in
your IDE.

[ Learn more...](../../book.html){.btn .btn-secondary .btn-block}
:::
::::::
:::::::
:::::::::::::::::::::::::::::::
::::::::::::::::::::::::::::::::
