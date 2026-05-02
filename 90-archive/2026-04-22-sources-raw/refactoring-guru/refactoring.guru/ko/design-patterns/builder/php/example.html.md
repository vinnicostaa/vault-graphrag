:::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [디자인
패턴들](../../../design-patterns.html){.type} /
[빌더](../../builder.html){.type} / [PHP](../../php.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![빌더](../../../../images/patterns/cards/builder-mini6d24.png?id=19b95fd05e6469679752c0554b116815){srcset="/images/patterns/cards/builder-mini-2x.png?id=de6d0938678b86903a1426dddfdd13bf 2x"}
:::

# PHP로 작성된 **빌더** {#php로-작성된-빌더 .pattern-example-title-block-title}
::::

:::::::::::::: pattern-example-body
::: pattern-example-brief
**빌더**는 복잡한 객체들을 단계별로 생성할 수 있도록 하는 생성 디자인
패턴입니다.

다른 생성 패턴과 달리 빌더 패턴은 제품들에 공통 인터페이스를 요구하지
않습니다. 이를 통해 같은 생성공정을 사용하여 다양한 제품을 생산할 수
있습니다.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ 빌더에 대하여 더 자세히 알아보세요 ](../../builder.html){.btn .btn-lg
.btn-primary}
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

**사용 예시들:** 빌더 패턴은 PHP 개발자들에게 잘 알려진 패턴이며, 가능한
설정 옵션이 많은 객체를 만들어야 할 때 특히 유용합니다.

**식별법:** 빌더 패턴은 하나의 생성 메서드와 결과 객체를 설정하기 위한
여러 메서드가 있는 클래스가 있습니다. 또 빌더 메서드들은 자주 사슬식
연결을 지원합니다 (예:
`someBuilder->​setValueA(1)->​setValueB(2)->​create()`{style="white-space: normal; display:inline;"}).
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

이 예시는 **빌더** 디자인 패턴의 구조를 보여주고 다음 질문에 중점을
둡니다:

-   패턴은 어떤 클래스들로 구성되어 있나요?
-   이 클래스들은 어떤 역할을 하나요?
-   패턴의 요소들은 어떻게 서로 연관되어 있나요?

이 패턴의 구조를 배우면 실제 PHP 사용 사례를 기반으로 하는 다음 예시를
더욱 쉽게 이해할 수 있을 것입니다.

#### []{#example-0--index-php .anchor} **index.php:** 개념적인 예시

<figure class="code">
<pre class="code" lang="php"><code>&lt;?php

namespace RefactoringGuru\Builder\Conceptual;

/**
 * The Builder interface specifies methods for creating the different parts of
 * the Product objects.
 */
interface Builder
{
    public function producePartA(): void;

    public function producePartB(): void;

    public function producePartC(): void;
}

/**
 * The Concrete Builder classes follow the Builder interface and provide
 * specific implementations of the building steps. Your program may have several
 * variations of Builders, implemented differently.
 */
class ConcreteBuilder1 implements Builder
{
    private $product;

    /**
     * A fresh builder instance should contain a blank product object, which is
     * used in further assembly.
     */
    public function __construct()
    {
        $this-&gt;reset();
    }

    public function reset(): void
    {
        $this-&gt;product = new Product1();
    }

    /**
     * All production steps work with the same product instance.
     */
    public function producePartA(): void
    {
        $this-&gt;product-&gt;parts[] = &quot;PartA1&quot;;
    }

    public function producePartB(): void
    {
        $this-&gt;product-&gt;parts[] = &quot;PartB1&quot;;
    }

    public function producePartC(): void
    {
        $this-&gt;product-&gt;parts[] = &quot;PartC1&quot;;
    }

    /**
     * Concrete Builders are supposed to provide their own methods for
     * retrieving results. That&#39;s because various types of builders may create
     * entirely different products that don&#39;t follow the same interface.
     * Therefore, such methods cannot be declared in the base Builder interface
     * (at least in a statically typed programming language). Note that PHP is a
     * dynamically typed language and this method CAN be in the base interface.
     * However, we won&#39;t declare it there for the sake of clarity.
     *
     * Usually, after returning the end result to the client, a builder instance
     * is expected to be ready to start producing another product. That&#39;s why
     * it&#39;s a usual practice to call the reset method at the end of the
     * `getProduct` method body. However, this behavior is not mandatory, and
     * you can make your builders wait for an explicit reset call from the
     * client code before disposing of the previous result.
     */
    public function getProduct(): Product1
    {
        $result = $this-&gt;product;
        $this-&gt;reset();

        return $result;
    }
}

/**
 * It makes sense to use the Builder pattern only when your products are quite
 * complex and require extensive configuration.
 *
 * Unlike in other creational patterns, different concrete builders can produce
 * unrelated products. In other words, results of various builders may not
 * always follow the same interface.
 */
class Product1
{
    public $parts = [];

    public function listParts(): void
    {
        echo &quot;Product parts: &quot; . implode(&#39;, &#39;, $this-&gt;parts) . &quot;\n\n&quot;;
    }
}

/**
 * The Director is only responsible for executing the building steps in a
 * particular sequence. It is helpful when producing products according to a
 * specific order or configuration. Strictly speaking, the Director class is
 * optional, since the client can control builders directly.
 */
class Director
{
    /**
     * @var Builder
     */
    private $builder;

    /**
     * The Director works with any builder instance that the client code passes
     * to it. This way, the client code may alter the final type of the newly
     * assembled product.
     */
    public function setBuilder(Builder $builder): void
    {
        $this-&gt;builder = $builder;
    }

    /**
     * The Director can construct several product variations using the same
     * building steps.
     */
    public function buildMinimalViableProduct(): void
    {
        $this-&gt;builder-&gt;producePartA();
    }

    public function buildFullFeaturedProduct(): void
    {
        $this-&gt;builder-&gt;producePartA();
        $this-&gt;builder-&gt;producePartB();
        $this-&gt;builder-&gt;producePartC();
    }
}

/**
 * The client code creates a builder object, passes it to the director and then
 * initiates the construction process. The end result is retrieved from the
 * builder object.
 */
function clientCode(Director $director)
{
    $builder = new ConcreteBuilder1();
    $director-&gt;setBuilder($builder);

    echo &quot;Standard basic product:\n&quot;;
    $director-&gt;buildMinimalViableProduct();
    $builder-&gt;getProduct()-&gt;listParts();

    echo &quot;Standard full featured product:\n&quot;;
    $director-&gt;buildFullFeaturedProduct();
    $builder-&gt;getProduct()-&gt;listParts();

    // Remember, the Builder pattern can be used without a Director class.
    echo &quot;Custom product:\n&quot;;
    $builder-&gt;producePartA();
    $builder-&gt;producePartC();
    $builder-&gt;getProduct()-&gt;listParts();
}

$director = new Director();
clientCode($director);</code></pre>
</figure>

#### []{#example-0--Output-txt .anchor} **Output.txt:** 실행 결과

<figure class="code">
<pre class="code" lang="output"><code>Standard basic product:
Product parts: PartA1

Standard full featured product:
Product parts: PartA1, PartB1, PartC1

Custom product:
Product parts: PartA1, PartC1</code></pre>
</figure>
:::

::: {#example-1 .tab-pane role="tabpanel" aria-labelledby="example-1-tab" style="padding-top: 24px"}
## 실제 사례 예시 {#example-1-title}

**빌더** 패턴의 최고의 적용사례 중 하나는 SQL 쿼리 빌더입니다. 빌더
인터페이스는 일반 SQL 쿼리를 생성하는 데 필요한 공통 단계들을
정의합니다. 반면에 다른 SQL 언어 유형들에 해당하는 구상 빌더들은 특정
데이터베이스 엔진에서 실행할 수 있는 SQL 쿼리들의 일부 부분들을 반환하여
이러한 단계들을 구현합니다.

#### []{#example-1--index-php .anchor} **index.php:** 실제 사례 예시

<figure class="code">
<pre class="code" lang="php"><code>&lt;?php

namespace RefactoringGuru\Builder\RealWorld;

/**
 * The Builder interface declares a set of methods to assemble an SQL query.
 *
 * All of the construction steps are returning the current builder object to
 * allow chaining: $builder-&gt;select(...)-&gt;where(...)
 */
interface SQLQueryBuilder
{
    public function select(string $table, array $fields): SQLQueryBuilder;

    public function where(string $field, string $value, string $operator = &#39;=&#39;): SQLQueryBuilder;

    public function limit(int $start, int $offset): SQLQueryBuilder;

    // +100 other SQL syntax methods...

    public function getSQL(): string;
}

/**
 * Each Concrete Builder corresponds to a specific SQL dialect and may implement
 * the builder steps a little bit differently from the others.
 *
 * This Concrete Builder can build SQL queries compatible with MySQL.
 */
class MysqlQueryBuilder implements SQLQueryBuilder
{
    protected $query;

    protected function reset(): void
    {
        $this-&gt;query = new \stdClass();
    }

    /**
     * Build a base SELECT query.
     */
    public function select(string $table, array $fields): SQLQueryBuilder
    {
        $this-&gt;reset();
        $this-&gt;query-&gt;base = &quot;SELECT &quot; . implode(&quot;, &quot;, $fields) . &quot; FROM &quot; . $table;
        $this-&gt;query-&gt;type = &#39;select&#39;;

        return $this;
    }

    /**
     * Add a WHERE condition.
     */
    public function where(string $field, string $value, string $operator = &#39;=&#39;): SQLQueryBuilder
    {
        if (!in_array($this-&gt;query-&gt;type, [&#39;select&#39;, &#39;update&#39;, &#39;delete&#39;])) {
            throw new \Exception(&quot;WHERE can only be added to SELECT, UPDATE OR DELETE&quot;);
        }
        $this-&gt;query-&gt;where[] = &quot;$field $operator &#39;$value&#39;&quot;;

        return $this;
    }

    /**
     * Add a LIMIT constraint.
     */
    public function limit(int $start, int $offset): SQLQueryBuilder
    {
        if (!in_array($this-&gt;query-&gt;type, [&#39;select&#39;])) {
            throw new \Exception(&quot;LIMIT can only be added to SELECT&quot;);
        }
        $this-&gt;query-&gt;limit = &quot; LIMIT &quot; . $start . &quot;, &quot; . $offset;

        return $this;
    }

    /**
     * Get the final query string.
     */
    public function getSQL(): string
    {
        $query = $this-&gt;query;
        $sql = $query-&gt;base;
        if (!empty($query-&gt;where)) {
            $sql .= &quot; WHERE &quot; . implode(&#39; AND &#39;, $query-&gt;where);
        }
        if (isset($query-&gt;limit)) {
            $sql .= $query-&gt;limit;
        }
        $sql .= &quot;;&quot;;
        return $sql;
    }
}

/**
 * This Concrete Builder is compatible with PostgreSQL. While Postgres is very
 * similar to Mysql, it still has several differences. To reuse the common code,
 * we extend it from the MySQL builder, while overriding some of the building
 * steps.
 */
class PostgresQueryBuilder extends MysqlQueryBuilder
{
    /**
     * Among other things, PostgreSQL has slightly different LIMIT syntax.
     */
    public function limit(int $start, int $offset): SQLQueryBuilder
    {
        parent::limit($start, $offset);

        $this-&gt;query-&gt;limit = &quot; LIMIT &quot; . $start . &quot; OFFSET &quot; . $offset;

        return $this;
    }

    // + tons of other overrides...
}


/**
 * Note that the client code uses the builder object directly. A designated
 * Director class is not necessary in this case, because the client code needs
 * different queries almost every time, so the sequence of the construction
 * steps cannot be easily reused.
 *
 * Since all our query builders create products of the same type (which is a
 * string), we can interact with all builders using their common interface.
 * Later, if we implement a new Builder class, we will be able to pass its
 * instance to the existing client code without breaking it thanks to the
 * SQLQueryBuilder interface.
 */
function clientCode(SQLQueryBuilder $queryBuilder)
{
    // ...

    $query = $queryBuilder
        -&gt;select(&quot;users&quot;, [&quot;name&quot;, &quot;email&quot;, &quot;password&quot;])
        -&gt;where(&quot;age&quot;, 18, &quot;&gt;&quot;)
        -&gt;where(&quot;age&quot;, 30, &quot;&lt;&quot;)
        -&gt;limit(10, 20)
        -&gt;getSQL();

    echo $query;

    // ...
}


/**
 * The application selects the proper query builder type depending on a current
 * configuration or the environment settings.
 */
// if ($_ENV[&#39;database_type&#39;] == &#39;postgres&#39;) {
//     $builder = new PostgresQueryBuilder(); } else {
//     $builder = new MysqlQueryBuilder(); }
//
// clientCode($builder);


echo &quot;Testing MySQL query builder:\n&quot;;
clientCode(new MysqlQueryBuilder());

echo &quot;\n\n&quot;;

echo &quot;Testing PostgresSQL query builder:\n&quot;;
clientCode(new PostgresQueryBuilder());</code></pre>
</figure>

#### []{#example-1--Output-txt .anchor} **Output.txt:** 실행 결과

<figure class="code">
<pre class="code" lang="output"><code>Testing MySQL query builder:
SELECT name, email, password FROM users WHERE age &gt; &#39;18&#39; AND age &lt; &#39;30&#39; LIMIT 10, 20;

Testing PostgresSQL query builder:
SELECT name, email, password FROM users WHERE age &gt; &#39;18&#39; AND age &lt; &#39;30&#39; LIMIT 10 OFFSET 20;</code></pre>
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

[PHP로 작성된 팩토리 메서드 []{.fa
.fa-arrow-right}](../../factory-method/php/example.html){.btn
.btn-primary rel="next"}
:::

::: prev
#### 돌아가기

[[]{.fa .fa-arrow-left} PHP로 작성된 추상
팩토리](../../abstract-factory/php/example.html){.btn .btn-default
rel="prev"}
:::

## 다른 언어로 작성된 **빌더**

[![C#으로 작성된
빌더](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "C#으로 작성된 빌더"){.prog-lang-link}
[![C++로 작성된
빌더](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "C++로 작성된 빌더"){.prog-lang-link}
[![Go로 작성된
빌더](../../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Go로 작성된 빌더"){.prog-lang-link}
[![자바로 작성된
빌더](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "자바로 작성된 빌더"){.prog-lang-link}
[![파이썬으로 작성된
빌더](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "파이썬으로 작성된 빌더"){.prog-lang-link}
[![루비로 작성된
빌더](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "루비로 작성된 빌더"){.prog-lang-link}
[![러스트로 작성된
빌더](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "러스트로 작성된 빌더"){.prog-lang-link}
[![스위프트로 작성된
빌더](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "스위프트로 작성된 빌더"){.prog-lang-link}
[![타입스크립트로 작성된
빌더](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "타입스크립트로 작성된 빌더"){.prog-lang-link}
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
