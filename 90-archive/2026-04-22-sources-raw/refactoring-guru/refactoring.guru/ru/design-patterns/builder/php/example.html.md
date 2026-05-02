:::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [Паттерны
проектирования](../../../design-patterns.html){.type} /
[Строитель](../../builder.html){.type} / [PHP](../../php.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Строитель](../../../../images/patterns/cards/builder-mini6d24.png?id=19b95fd05e6469679752c0554b116815){srcset="/images/patterns/cards/builder-mini-2x.png?id=de6d0938678b86903a1426dddfdd13bf 2x"}
:::

# **Строитель** на PHP {#строитель-на-php .pattern-example-title-block-title}
::::

:::::::::::::: pattern-example-body
::: pattern-example-brief
**Строитель** --- это порождающий паттерн проектирования, который
позволяет создавать объекты пошагово.

В отличие от других порождающих паттернов, Строитель позволяет
производить различные продукты, используя один и тот же процесс
строительства.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Подробней о паттерне Строитель ](../../builder.html){.btn .btn-lg
.btn-primary}
:::

::::::::::: {.sidebar-navigation .with-scroll}
::: en-title
Навигация
:::

::: en-intro
 [Интро](#)
:::

::: en-intro
 [Концептуальный пример](#example-0)
:::

::: en-file
 [index](#example-0--index-php)
:::

::: en-file
 [Output](#example-0--Output-txt)
:::

::: en-intro
 [Пример из реальной жизни](#example-1)
:::

::: en-file
 [index](#example-1--index-php)
:::

::: en-file
 [Output](#example-1--Output-txt)
:::
:::::::::::

**Сложность:** []{.fa-stars}

**Популярность:** []{.fa-stars}

**Применимость:** Паттерн можно часто встретить в PHP-коде, особенно
там, где требуется пошаговое создание продуктов или конфигурация сложных
объектов.

**Признаки применения паттерна:** Строителя можно узнать в классе,
который имеет один создающий метод и несколько методов настройки
создаваемого продукта. Обычно, методы настройки вызывают для удобства
цепочкой (например,
`someBuilder->setValueA(1)->setValueB(2)->create()`).
::::::::::::::

::: {#nav-tab .nav .nav-tabs .nav-justified role="tablist"}
[Концептуальный пример](#example-0){#example-0-tab .nav-item .nav-link
.active toggle="tab" role="tab" aria-controls="example-0"
aria-selected="true"} [Пример из реальной
жизни](#example-1){#example-1-tab .nav-item .nav-link toggle="tab"
role="tab" aria-controls="example-1" aria-selected="true"}
:::

::::: {#nav-tabContent .pattern-example-tab-content .tab-content}
::: {#example-0 .tab-pane .show .active role="tabpanel" aria-labelledby="example-0-tab" style="padding-top: 24px"}
## Концептуальный пример {#example-0-title}

Этот пример показывает структуру паттерна **Строитель**, а именно --- из
каких классов он состоит, какие роли эти классы выполняют и как они
взаимодействуют друг с другом.

После ознакомления со структурой, вам будет легче воспринимать второй
пример, который рассматривает реальный случай использования паттерна в
мире PHP.

#### []{#example-0--index-php .anchor} **index.php:** Пример структуры паттерна

<figure class="code">
<pre class="code" lang="php"><code>&lt;?php

namespace RefactoringGuru\Builder\Conceptual;

/**
 * Интерфейс Строителя объявляет создающие методы для различных частей объектов
 * Продуктов.
 */
interface Builder
{
    public function producePartA(): void;

    public function producePartB(): void;

    public function producePartC(): void;
}

/**
 * Классы Конкретного Строителя следуют интерфейсу Строителя и предоставляют
 * конкретные реализации шагов построения. Ваша программа может иметь несколько
 * вариантов Строителей, реализованных по-разному.
 */
class ConcreteBuilder1 implements Builder
{
    private $product;

    /**
     * Новый экземпляр строителя должен содержать пустой объект продукта,
     * который используется в дальнейшей сборке.
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
     * Все этапы производства работают с одним и тем же экземпляром продукта.
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
     * Конкретные Строители должны предоставить свои собственные методы
     * получения результатов. Это связано с тем, что различные типы строителей
     * могут создавать совершенно разные продукты с разными интерфейсами.
     * Поэтому такие методы не могут быть объявлены в базовом интерфейсе
     * Строителя (по крайней мере, в статически типизированном языке
     * программирования). Обратите внимание, что PHP является динамически
     * типизированным языком, и этот метод может быть в базовом интерфейсе.
     * Однако мы не будем объявлять его здесь для ясности.
     *
     * Как правило, после возвращения конечного результата клиенту, экземпляр
     * строителя должен быть готов к началу производства следующего продукта.
     * Поэтому обычной практикой является вызов метода сброса в конце тела
     * метода getProduct. Однако такое поведение не является обязательным, вы
     * можете заставить своих строителей ждать явного запроса на сброс из кода
     * клиента, прежде чем избавиться от предыдущего результата.
     */
    public function getProduct(): Product1
    {
        $result = $this-&gt;product;
        $this-&gt;reset();

        return $result;
    }
}

/**
 * Имеет смысл использовать паттерн Строитель только тогда, когда ваши продукты
 * достаточно сложны и требуют обширной конфигурации.
 *
 * В отличие от других порождающих паттернов, различные конкретные строители
 * могут производить несвязанные продукты. Другими словами, результаты различных
 * строителей могут не всегда следовать одному и тому же интерфейсу.
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
 * Директор отвечает только за выполнение шагов построения в определённой
 * последовательности. Это полезно при производстве продуктов в определённом
 * порядке или особой конфигурации. Строго говоря, класс Директор необязателен,
 * так как клиент может напрямую управлять строителями.
 */
class Director
{
    /**
     * @var Builder
     */
    private $builder;

    /**
     * Директор работает с любым экземпляром строителя, который передаётся ему
     * клиентским кодом. Таким образом, клиентский код может изменить конечный
     * тип вновь собираемого продукта.
     */
    public function setBuilder(Builder $builder): void
    {
        $this-&gt;builder = $builder;
    }

    /**
     * Директор может строить несколько вариаций продукта, используя одинаковые
     * шаги построения.
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
 * Клиентский код создаёт объект-строитель, передаёт его директору, а затем
 * инициирует процесс построения. Конечный результат извлекается из объекта-
 * строителя.
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

    // Помните, что паттерн Строитель можно использовать без класса Директор.
    echo &quot;Custom product:\n&quot;;
    $builder-&gt;producePartA();
    $builder-&gt;producePartC();
    $builder-&gt;getProduct()-&gt;listParts();
}

$director = new Director();
clientCode($director);</code></pre>
</figure>

#### []{#example-0--Output-txt .anchor} **Output.txt:** Результат выполнения

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
## Пример из реальной жизни {#example-1-title}

Одним из лучших применений паттерна **Строитель** является конструктор
запросов SQL. Интерфейс Строителя определяет общие шаги, необходимые для
построения основного SQL-запроса. В тоже время Конкретные Строители,
соответствующие различным диалектам SQL, реализуют эти шаги, возвращая
части SQL-запросов, которые могут быть выполнены в данном движке базы
данных.

#### []{#example-1--index-php .anchor} **index.php:** Пример из реальной жизни

<figure class="code">
<pre class="code" lang="php"><code>&lt;?php

namespace RefactoringGuru\Builder\RealWorld;

/**
 * Интерфейс Строителя объявляет набор методов для сборки SQL-запроса.
 *
 * Все шаги построения возвращают текущий объект строителя, чтобы обеспечить
 * цепочку: $builder-&gt;select(...)-&gt;where(...)
 */
interface SQLQueryBuilder
{
    public function select(string $table, array $fields): SQLQueryBuilder;

    public function where(string $field, string $value, string $operator = &#39;=&#39;): SQLQueryBuilder;

    public function limit(int $start, int $offset): SQLQueryBuilder;

    // +100 других методов синтаксиса SQL...

    public function getSQL(): string;
}

/**
 * Каждый Конкретный Строитель соответствует определённому диалекту SQL и может
 * реализовать шаги построения немного иначе, чем остальные.
 *
 * Этот Конкретный Строитель может создавать SQL-запросы, совместимые с MySQL.
 */
class MysqlQueryBuilder implements SQLQueryBuilder
{
    protected $query;

    protected function reset(): void
    {
        $this-&gt;query = new \stdClass();
    }

    /**
     * Построение базового запроса SELECT.
     */
    public function select(string $table, array $fields): SQLQueryBuilder
    {
        $this-&gt;reset();
        $this-&gt;query-&gt;base = &quot;SELECT &quot; . implode(&quot;, &quot;, $fields) . &quot; FROM &quot; . $table;
        $this-&gt;query-&gt;type = &#39;select&#39;;

        return $this;
    }

    /**
     * Добавление условия WHERE.
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
     * Добавление ограничения LIMIT.
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
     * Получение окончательной строки запроса.
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
 * Этот Конкретный Строитель совместим с PostgreSQL. Хотя Postgres очень похож
 * на Mysql, в нем всё же есть ряд отличий. Чтобы повторно использовать общий
 * код, мы расширяем его от строителя MySQL, переопределяя некоторые шаги
 * построения.
 */
class PostgresQueryBuilder extends MysqlQueryBuilder
{
    /**
     * Помимо прочего, PostgreSQL имеет несколько иной синтаксис LIMIT.
     */
    public function limit(int $start, int $offset): SQLQueryBuilder
    {
        parent::limit($start, $offset);

        $this-&gt;query-&gt;limit = &quot; LIMIT &quot; . $start . &quot; OFFSET &quot; . $offset;

        return $this;
    }

    // + тонны других переопределений...
}


/**
 * Обратите внимание, что клиентский код непосредственно использует объект
 * строителя. Назначенный класс Директора в этом случае не нужен, потому что
 * клиентский код практически всегда нуждается в различных запросах, поэтому
 * последовательность шагов конструирования непросто повторно использовать.
 *
 * Поскольку все наши строители запросов создают продукты одного типа (это
 * строка), мы можем взаимодействовать со всеми строителями, используя их общий
 * интерфейс. Позднее, если мы реализуем новый класс Строителя, мы сможем
 * передать его экземпляр существующему клиентскому коду, не нарушая его,
 * благодаря интерфейсу SQLQueryBuilder.
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
 * Приложение выбирает подходящий тип строителя запроса в зависимости от текущей
 * конфигурации или настроек среды.
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

#### []{#example-1--Output-txt .anchor} **Output.txt:** Результат выполнения

<figure class="code">
<pre class="code" lang="output"><code>Testing MySQL query builder:
SELECT name, email, password FROM users WHERE age &gt; &#39;18&#39; AND age &lt; &#39;30&#39; LIMIT 10, 20;

Testing PostgresSQL query builder:
SELECT name, email, password FROM users WHERE age &gt; &#39;18&#39; AND age &lt; &#39;30&#39; LIMIT 10 OFFSET 20;</code></pre>
</figure>
:::
:::::

::: {#nav-tab .nav .nav-tabs .nav-tabs-bottom .nav-justified role="tablist"}
[Концептуальный пример](#example-0){#example-0-tab-bottom .nav-item
.nav-link .active toggle="tab" role="tab" aria-controls="example-0"
aria-selected="true"} [Пример из реальной
жизни](#example-1){#example-1-tab-bottom .nav-item .nav-link
toggle="tab" role="tab" aria-controls="example-1" aria-selected="true"}
:::

::: next
#### Читаем дальше

[Фабричный метод на PHP []{.fa
.fa-arrow-right}](../../factory-method/php/example.html){.btn
.btn-primary rel="next"}
:::

::: prev
#### Вернуться назад

[[]{.fa .fa-arrow-left} Абстрактная фабрика на
PHP](../../abstract-factory/php/example.html){.btn .btn-default
rel="prev"}
:::

## **Строитель** на других языках программирования

[![Строитель на
C#](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Строитель на C#"){.prog-lang-link}
[![Строитель на
C++](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Строитель на C++"){.prog-lang-link}
[![Строитель на
Go](../../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Строитель на Go"){.prog-lang-link}
[![Строитель на
Java](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Строитель на Java"){.prog-lang-link}
[![Строитель на
Python](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Строитель на Python"){.prog-lang-link}
[![Строитель на
Ruby](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Строитель на Ruby"){.prog-lang-link}
[![Строитель на
Rust](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Строитель на Rust"){.prog-lang-link}
[![Строитель на
Swift](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Строитель на Swift"){.prog-lang-link}
[![Строитель на
TypeScript](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Строитель на TypeScript"){.prog-lang-link}
:::::::::::::::::::::::::

::::::: {.prom .banner-sidebar .banner .banner-striped .banner-examples .banner-removable .banner-removable-patterns data-id="DP: Examples-IDE" creative-id="examples-sidebar-ru" position="sidebar"}
:::::: {.banner-inner style="font-size: 14px; line-height: 21px;"}
::: banner-examples-img
[![](../../../../images/patterns/banners/examples-ide91ca.png?id=3115b4b548fb96b75974e2de8f4f49bc){width="215"
height="158" loading="lazy"
srcset="/images/patterns/banners/examples-ide-2x.png?id=93c007a6d157b97c28eb213f59d716ae 2x"}](../../book.html)
:::

::: banner-examples-fade
[ Архив с примерами](../../book.html){.btn .btn-secondary .btn-block}
:::

::: {.banner-examples-text style="margin-top: 1rem;"}
Купи книгу **Погружение в Паттерны** и получи архив с десятками
детальных примеров, которые можно открывать прямо в IDE.

[ Узнать больше...](../../book.html){.btn .btn-secondary .btn-block}
:::
::::::
:::::::
:::::::::::::::::::::::::::::::
::::::::::::::::::::::::::::::::
