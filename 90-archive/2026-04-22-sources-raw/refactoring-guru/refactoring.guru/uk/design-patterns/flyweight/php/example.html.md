:::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [Патерни
проектування](../../../design-patterns.html){.type} /
[Легковаговик](../../flyweight.html){.type} /
[PHP](../../php.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Легковаговик](../../../../images/patterns/cards/flyweight-minie3d8.png?id=422ca8d2f90614dce810a8812c626698){srcset="/images/patterns/cards/flyweight-mini-2x.png?id=857ca676fff84317bb0dab76abfce08e 2x"}
:::

# **Легковаговик** на PHP {#легковаговик-на-php .pattern-example-title-block-title}
::::

:::::::::::::: pattern-example-body
::: pattern-example-brief
**Легковаговик** --- це структурний патерн, який економить пам'ять
завдяки розподілу спільного стану, винесеного в один об'єкт, між
безліччю об'єктів.

Легковаговик дозволяє економити пам'ять, записуючи в кеш однакові дані,
що використовуються різними об'єктами.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Детальніше про Легковаговик ](../../flyweight.html){.btn .btn-lg
.btn-primary}
:::

::::::::::: {.sidebar-navigation .with-scroll}
::: en-title
Навігація
:::

::: en-intro
 [Інтро](#)
:::

::: en-intro
 [Концептуальний приклад](#example-0)
:::

::: en-file
 [index](#example-0--index-php)
:::

::: en-file
 [Output](#example-0--Output-txt)
:::

::: en-intro
 [Життєвий приклад](#example-1)
:::

::: en-file
 [index](#example-1--index-php)
:::

::: en-file
 [Output](#example-1--Output-txt)
:::
:::::::::::

**Складність:** []{.fa-stars}

**Популярність:** []{.fa-stars}

**Застосування:** Сенс використання Легковаговика --- це економія
пам'яті. Тому, якщо в програмі немає такої проблеми, ви навряд чи
знайдете там приклади Легковаговика.

Патерн дуже рідко актуальний для PHP-програм, через саму природу ціїє
мови. Скрипти майже завжди працюють з даними програми частинами, ніколи
не завантажуючи усі наявні дані в пам'ять одночасно.

**Ознаки застосування патерна:** Легковаговик можна визначити за
створюваними методами класу, які повертають закешовані об'єкти, замість
створення нових.
::::::::::::::

::: {#nav-tab .nav .nav-tabs .nav-justified role="tablist"}
[Концептуальний приклад](#example-0){#example-0-tab .nav-item .nav-link
.active toggle="tab" role="tab" aria-controls="example-0"
aria-selected="true"} [Життєвий приклад](#example-1){#example-1-tab
.nav-item .nav-link toggle="tab" role="tab" aria-controls="example-1"
aria-selected="true"}
:::

::::: {#nav-tabContent .pattern-example-tab-content .tab-content}
::: {#example-0 .tab-pane .show .active role="tabpanel" aria-labelledby="example-0-tab" style="padding-top: 24px"}
## Концептуальний приклад {#example-0-title}

Цей приклад показує структуру патерна **Легковаговик**, а саме --- з
яких класів він складається, які ролі ці класи виконують і як вони
взаємодіють один з одним.

Після ознайомлення зі структурою, вам буде легше сприймати наступний
приклад, що розглядає реальний випадок використання патерна в світі PHP.

#### []{#example-0--index-php .anchor} **index.php:** Приклад структури патерна

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

#### []{#example-0--Output-txt .anchor} **Output.txt:** Результат виконання

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
## Життєвий приклад {#example-1-title}

#### []{#example-1--index-php .anchor} **index.php:** Приклад з реального світу

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

#### []{#example-1--Output-txt .anchor} **Output.txt:** Результат виконання

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
[Концептуальний приклад](#example-0){#example-0-tab-bottom .nav-item
.nav-link .active toggle="tab" role="tab" aria-controls="example-0"
aria-selected="true"} [Життєвий
приклад](#example-1){#example-1-tab-bottom .nav-item .nav-link
toggle="tab" role="tab" aria-controls="example-1" aria-selected="true"}
:::

::: next
#### Читаємо далі

[Замісник на PHP []{.fa
.fa-arrow-right}](../../proxy/php/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Повернутись назад

[[]{.fa .fa-arrow-left} Фасад на
PHP](../../facade/php/example.html){.btn .btn-default rel="prev"}
:::

## **Легковаговик** іншими мовами програмування

[![Легковаговик на
C#](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Легковаговик на C#"){.prog-lang-link}
[![Легковаговик на
C++](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Легковаговик на C++"){.prog-lang-link}
[![Легковаговик на
Go](../../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Легковаговик на Go"){.prog-lang-link}
[![Легковаговик на
Java](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Легковаговик на Java"){.prog-lang-link}
[![Легковаговик на
Python](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Легковаговик на Python"){.prog-lang-link}
[![Легковаговик на
Ruby](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Легковаговик на Ruby"){.prog-lang-link}
[![Легковаговик на
Rust](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Легковаговик на Rust"){.prog-lang-link}
[![Легковаговик на
Swift](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Легковаговик на Swift"){.prog-lang-link}
[![Легковаговик на
TypeScript](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Легковаговик на TypeScript"){.prog-lang-link}
:::::::::::::::::::::::::

::::::: {.prom .banner-sidebar .banner .banner-striped .banner-examples .banner-removable .banner-removable-patterns data-id="DP: Examples-IDE" creative-id="examples-sidebar-uk" position="sidebar"}
:::::: {.banner-inner style="font-size: 14px; line-height: 21px;"}
::: banner-examples-img
[![](../../../../images/patterns/banners/examples-ide91ca.png?id=3115b4b548fb96b75974e2de8f4f49bc){width="215"
height="158" loading="lazy"
srcset="/images/patterns/banners/examples-ide-2x.png?id=93c007a6d157b97c28eb213f59d716ae 2x"}](../../book.html)
:::

::: banner-examples-fade
[ Архів з прикладами](../../book.html){.btn .btn-secondary .btn-block}
:::

::: {.banner-examples-text style="margin-top: 1rem;"}
Придбай книгу **Занурення в Патерни** та отримай архів з десятками
детальних прикладів, які можна відкривати прямо в IDE.

[ Дізнатися більше...](../../book.html){.btn .btn-secondary .btn-block}
:::
::::::
:::::::
:::::::::::::::::::::::::::::::
::::::::::::::::::::::::::::::::
