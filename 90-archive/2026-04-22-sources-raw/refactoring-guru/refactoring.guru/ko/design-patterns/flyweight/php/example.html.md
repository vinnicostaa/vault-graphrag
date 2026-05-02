:::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [디자인
패턴들](../../../design-patterns.html){.type} /
[플라이웨이트](../../flyweight.html){.type} /
[PHP](../../php.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![플라이웨이트](../../../../images/patterns/cards/flyweight-minie3d8.png?id=422ca8d2f90614dce810a8812c626698){srcset="/images/patterns/cards/flyweight-mini-2x.png?id=857ca676fff84317bb0dab76abfce08e 2x"}
:::

# PHP로 작성된 **플라이웨이트** {#php로-작성된-플라이웨이트 .pattern-example-title-block-title}
::::

:::::::::::::: pattern-example-body
::: pattern-example-brief
**플라이웨이트**는 구조 패턴이며 프로그램들이 객체들의 메모리 소비를
낮게 유지하여 방대한 양의 객체들을 지원할 수 있도록 합니다.

이 패턴은 여러 객체 사이의 객체 상태를 공유하여 위를 달성합니다. 다르게
설명하자면 플라이웨이트는 다른 객체들이 공통으로 사용하는 데이터를
캐싱하여 RAM을 절약합니다.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ 플라이웨이트에 대하여 더 자세히 알아보세요
](../../flyweight.html){.btn .btn-lg .btn-primary}
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
 [실제 사례 예시](#example-1)
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

**사용 사례들:** 플라이웨이트 패턴은 언어의 특성상 PHP 앱에서 거의
사용되지 않습니다. PHP 스크립트는 일반적으로 앱 데이터의 일부와 함께
작동하며 모든 앱 데이터를 동시에 메모리에 로드하지 않습니다.

**식별:** 플라이웨이트는 새로운 객체들 대신 캐싱 된 객체들을 반환하는
생성 메서드의 유무로 식별될 수 있습니다.
::::::::::::::

::: {#nav-tab .nav .nav-tabs .nav-justified role="tablist"}
[개념적인 예시](#example-0){#example-0-tab .nav-item .nav-link .active
toggle="tab" role="tab" aria-controls="example-0" aria-selected="true"}
[실제 사례 예시](#example-1){#example-1-tab .nav-item .nav-link
toggle="tab" role="tab" aria-controls="example-1" aria-selected="true"}
:::

::::: {#nav-tabContent .pattern-example-tab-content .tab-content}
::: {#example-0 .tab-pane .show .active role="tabpanel" aria-labelledby="example-0-tab" style="padding-top: 24px"}
## 개념적인 예시 {#example-0-title}

이 예시는 **플라이웨이트**의 구조를 보여주고 다음 질문에 중점을 둡니다:

-   패턴은 어떤 클래스들로 구성되어 있나요?
-   이 클래스들은 어떤 역할을 하나요?
-   패턴의 요소들은 어떻게 서로 연관되어 있나요?

이 패턴의 구조를 배우면 실제 PHP 사용 사례를 기반으로 하는 다음 예시를
더욱 쉽게 이해할 수 있을 것입니다.

#### []{#example-0--index-php .anchor} **index.php:** 개념적인 예시

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

#### []{#example-0--Output-txt .anchor} **Output.txt:** 실행 결과

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
## 실제 사례 예시 {#example-1-title}

시작하기 전에 PHP에서의 **플라이웨이트** 패턴의 실제 응용은 매우
드물다는 점에 유의하세요. 이것은 PHP의 단일 스레드 특성에서 비롯됩니다:
앱의 모든 객체를 동시에 같은 스레드에 저장하면 안 됩니다. 이 예시의
아이디어는 진정성 있는 예시가 아니며, 위에 언급된 RAM 문제는 앱을 다르게
구조화하여 해결할 수 있으나 그래도 예시를 통하여 패턴의 개념이 실제
사례와 어떻게 작동하는지 보여줍니다.

이 예시에서 플라이웨이트 패턴은 고양이 전용 동물병원의 동물
데이터베이스에 있는 객체들의 RAM 사용량을 최소화하는 데 사용됩니다.
데이터베이스의 각 레코드는 `Cat`객체로 표시되며 그의 데이터는 두
부분으로 구성됩니다:

1.  애완동물의 이름, 나이 및 소유자 정보와 같은 독특한 (공유한 상태의)
    데이터.
2.  고양이 품종 이름, 색상, 질감 등과 같은 공유된 (고유한 상태의)
    데이터.

첫 번째 부분은 콘텍스트 역할을 하는 `Cat` 클래스의 내부에 직접
저장됩니다. 그러나 두 번째 부분은 별도로 저장되어 여러 고양이가 공유할
수 있습니다. 이 공유 가능한 데이터는 `Cat­Variation` 클래스 내에
있습니다. 유사한 기능을 가진 모든 고양이는 중복 데이터를 그들의 각 해당
객체에 저장하는 대신 같은 `Cat­Variation` 클래스에 연결됩니다.

#### []{#example-1--index-php .anchor} **index.php:** 실제 사례 예시

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

#### []{#example-1--Output-txt .anchor} **Output.txt:** 실행 결과

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
[개념적인 예시](#example-0){#example-0-tab-bottom .nav-item .nav-link
.active toggle="tab" role="tab" aria-controls="example-0"
aria-selected="true"} [실제 사례 예시](#example-1){#example-1-tab-bottom
.nav-item .nav-link toggle="tab" role="tab" aria-controls="example-1"
aria-selected="true"}
:::

::: next
#### 다음을 보세요

[PHP로 작성된 프록시 []{.fa
.fa-arrow-right}](../../proxy/php/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### 돌아가기

[[]{.fa .fa-arrow-left} PHP로 작성된
퍼사드](../../facade/php/example.html){.btn .btn-default rel="prev"}
:::

## 다른 언어로 작성된 **플라이웨이트**

[![C#으로 작성된
플라이웨이트](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "C#으로 작성된 플라이웨이트"){.prog-lang-link}
[![C++로 작성된
플라이웨이트](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "C++로 작성된 플라이웨이트"){.prog-lang-link}
[![Go로 작성된
플라이웨이트](../../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Go로 작성된 플라이웨이트"){.prog-lang-link}
[![자바로 작성된
플라이웨이트](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "자바로 작성된 플라이웨이트"){.prog-lang-link}
[![파이썬으로 작성된
플라이웨이트](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "파이썬으로 작성된 플라이웨이트"){.prog-lang-link}
[![루비로 작성된
플라이웨이트](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "루비로 작성된 플라이웨이트"){.prog-lang-link}
[![러스트로 작성된
플라이웨이트](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "러스트로 작성된 플라이웨이트"){.prog-lang-link}
[![스위프트로 작성된
플라이웨이트](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "스위프트로 작성된 플라이웨이트"){.prog-lang-link}
[![타입스크립트로 작성된
플라이웨이트](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "타입스크립트로 작성된 플라이웨이트"){.prog-lang-link}
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
