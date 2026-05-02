:::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [Паттерны
проектирования](../../../design-patterns.html){.type} /
[Легковес](../../flyweight.html){.type} / [PHP](../../php.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Легковес](../../../../images/patterns/cards/flyweight-minie3d8.png?id=422ca8d2f90614dce810a8812c626698){srcset="/images/patterns/cards/flyweight-mini-2x.png?id=857ca676fff84317bb0dab76abfce08e 2x"}
:::

# **Легковес** на PHP {#легковес-на-php .pattern-example-title-block-title}
::::

:::::::::::::: pattern-example-body
::: pattern-example-brief
**Легковес** --- это структурный паттерн, который экономит память,
благодаря разделению общего состояния, вынесенного в один объект, между
множеством объектов.

Легковес позволяет экономить память, кешируя одинаковые данные,
используемые в разных объектах.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Подробней о паттерне Легковес ](../../flyweight.html){.btn .btn-lg
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

**Применимость:** Весь смысл использования Легковеса --- в экономии
памяти. Поэтому, если в приложении нет такой проблемы, то вы вряд ли
найдёте там примеры Легковеса.

Паттерн особенно редко актуален для PHP-приложений, из-за самой природы
языка. Скрипты почти всегда работают с данными приложения частями,
никогда не загружая все данные в память одновременно.

**Признаки применения паттерна:** Легковес можно определить по создающим
методам класса, которые возвращают закешированные объекты, вместо
создания новых.
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

Этот пример показывает структуру паттерна **Легковес**, а именно --- из
каких классов он состоит, какие роли эти классы выполняют и как они
взаимодействуют друг с другом.

После ознакомления со структурой, вам будет легче воспринимать второй
пример, который рассматривает реальный случай использования паттерна в
мире PHP.

#### []{#example-0--index-php .anchor} **index.php:** Пример структуры паттерна

<figure class="code">
<pre class="code" lang="php"><code>&lt;?php

namespace RefactoringGuru\Flyweight\Conceptual;

/**
 * Легковес хранит общую часть состояния (также называемую внутренним
 * состоянием), которая принадлежит нескольким реальным бизнес-объектам.
 * Легковес принимает оставшуюся часть состояния (внешнее состояние, уникальное
 * для каждого объекта)  через его параметры метода.
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
 * Фабрика Легковесов создает объекты-Легковесы и управляет ими. Она
 * обеспечивает правильное разделение легковесов. Когда клиент запрашивает
 * легковес, фабрика либо возвращает существующий экземпляр, либо создает новый,
 * если он ещё не существует.
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
     * Возвращает хеш строки Легковеса для данного состояния.
     */
    private function getKey(array $state): string
    {
        ksort($state);

        return implode(&quot;_&quot;, $state);
    }

    /**
     * Возвращает существующий Легковес с заданным состоянием или создает новый.
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
 * Клиентский код обычно создает кучу предварительно заполненных легковесов на
 * этапе инициализации приложения.
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

    // Клиентский код либо сохраняет, либо вычисляет внешнее состояние и
    // передает его методам легковеса.
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

#### []{#example-0--Output-txt .anchor} **Output.txt:** Результат выполнения

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
## Пример из реальной жизни {#example-1-title}

Прежде чем мы начнём, обратите внимание, что реальное применение
паттерна **Легковес** на PHP встречается довольно редко. Это связано с
однопоточным характером PHP, где вы не должны хранить ВСЕ объекты вашего
приложения в памяти одновременно в одном потоке. Хотя замысел этого
примера только наполовину серьёзен, и вся проблема с ОЗУ может быть
решена, если приложение структурировать по-другому, он всё же наглядно
показывает концепцию паттерна, как он работает в реальном мире. Итак, я
вас предупредил. Теперь давайте начнём.

В этом примере паттерн Легковес применяется для минимизации
использования оперативной памяти объектами в базе данных животных
ветеринарной клиники только для кошек. Каждую запись в базе данных
представляет объект-`Кот`. Его данные состоят из двух частей:

1.  Уникальные (внешние) данные: имя кота, возраст и инфо о владельце.
2.  Общие (внутренние) данные: название породы, цвет, текстура и т. д.

Первая часть хранится непосредственно внутри класса `Кот`, который
играет роль контекста. Вторая часть, однако, хранится отдельно и может
совместно использоваться разными котами. Эти совместно используемые
данные находятся внутри класса `РазновидностиКотов`. Все коты, имеющие
схожие признаки, привязаны к одному и тому же классу
`РазновидностиКотов`, вместо того чтобы хранить повторяющиеся данные в
каждом из своих объектов.

#### []{#example-1--index-php .anchor} **index.php:** Пример из реальной жизни

<figure class="code">
<pre class="code" lang="php"><code>&lt;?php

namespace RefactoringGuru\Flyweight\RealWorld;

/**
 * Объекты Легковеса представляют данные, разделяемые несколькими объектами
 * Кошек. Это сочетание породы, цвета, текстуры и т.д.
 */
class CatVariation
{
    /**
     * Так называемое «внутреннее» состояние.
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
     * Этот метод отображает информацию о кошке. Метод принимает внешнее
     * состояние в качестве аргументов. Остальная часть состояния хранится
     * внутри полей Легковеса.
     *
     * Возможно, вы удивлены, почему мы поместили основную логику кошки в класс
     * РазновидностейКошек вместо того, чтобы держать её в классе Кошки. Я
     * согласен, это звучит странно.
     *
     * Имейте в виду, что в реальной жизни паттерн Легковес может быть либо
     * реализован с самого начала, либо принудительно применён к существующему
     * приложению, когда разработчики понимают, что они столкнулись с проблемой
     * ОЗУ.
     *
     * Во втором случае вы получаете такие же классы, как у нас. Мы как бы
     * «отрефакторили» идеальное приложение, где все данные изначально
     * находились внутри класса Кошки. Если бы мы реализовывали Легковес с
     * самого начала, названия наших классов могли бы быть другими и более
     * определёнными. Например, Кошка и КонтекстКошки.
     *
     * Однако действительная причина, по которой основное поведение должно
     * проживать в классе Легковеса, заключается в том, что у вас может вообще
     * не быть объявленного класса Контекста. Контекстные данные могут храниться
     * в массиве или какой-то другой, более эффективной структуре данных.
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
 * Контекст хранит данные, уникальные для каждой кошки.
 *
 * Создавать отдельный класс для хранения контекста необязательно и не всегда
 * целесообразно. Контекст может храниться внутри громоздкой структуры данных в
 * коде Клиента и при необходимости передаваться в методы легковеса.
 */
class Cat
{
    /**
     * Так называемое «внешнее» состояние.
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
     * Поскольку объекты Контекста не владеют всем своим состоянием, иногда для
     * удобства вы можете реализовать несколько вспомогательных методов
     * (например, для сравнения нескольких объектов Контекста между собой).
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
     * Кроме того, Контекст может определять несколько методов быстрого доступа,
     * которые делегируют исполнение объекту-Легковесу. Эти методы могут быть
     * остатками реальных методов, извлечённых в класс Легковеса во время
     * массивного рефакторинга к паттерну Легковес.
     */
    public function render(): void
    {
        $this-&gt;variation-&gt;renderProfile($this-&gt;name, $this-&gt;age, $this-&gt;owner);
    }
}

/**
 * Фабрика Легковесов хранит объекты Контекст и Легковес, эффективно скрывая
 * любое упоминание о паттерне Легковес от клиента.
 */
class CatDataBase
{
    /**
     * Список объектов-кошек (Контексты).
     */
    private $cats = [];

    /**
     * Список вариаций кошки (Легковесы).
     */
    private $variations = [];

    /**
     * При добавлении кошки в базу данных мы сначала ищем существующую вариацию
     * кошки.
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
     * Возвращаем существующий вариант (Легковеса) по указанным данным или
     * создаём новый, если он ещё не существует.
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
     * Эта функция помогает генерировать уникальные ключи массива.
     */
    private function getKey(array $data): string
    {
        return md5(implode(&quot;_&quot;, $data));
    }

    /**
     * Ищем кошку в базе данных, используя заданные параметры запроса.
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
 * Клиентский код.
 */
$db = new CatDataBase();

echo &quot;Client: Let&#39;s see what we have in \&quot;cats.csv\&quot;.\n&quot;;

// Чтобы увидеть реальный эффект паттерна, вы должны иметь большую базу данных с
// несколькими миллионами записей. Не стесняйтесь экспериментировать с кодом,
// чтобы увидеть реальные масштабы паттерна.
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

#### []{#example-1--Output-txt .anchor} **Output.txt:** Результат выполнения

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
[Концептуальный пример](#example-0){#example-0-tab-bottom .nav-item
.nav-link .active toggle="tab" role="tab" aria-controls="example-0"
aria-selected="true"} [Пример из реальной
жизни](#example-1){#example-1-tab-bottom .nav-item .nav-link
toggle="tab" role="tab" aria-controls="example-1" aria-selected="true"}
:::

::: next
#### Читаем дальше

[Заместитель на PHP []{.fa
.fa-arrow-right}](../../proxy/php/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Вернуться назад

[[]{.fa .fa-arrow-left} Фасад на
PHP](../../facade/php/example.html){.btn .btn-default rel="prev"}
:::

## **Легковес** на других языках программирования

[![Легковес на
C#](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Легковес на C#"){.prog-lang-link}
[![Легковес на
C++](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Легковес на C++"){.prog-lang-link}
[![Легковес на
Go](../../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Легковес на Go"){.prog-lang-link}
[![Легковес на
Java](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Легковес на Java"){.prog-lang-link}
[![Легковес на
Python](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Легковес на Python"){.prog-lang-link}
[![Легковес на
Ruby](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Легковес на Ruby"){.prog-lang-link}
[![Легковес на
Rust](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Легковес на Rust"){.prog-lang-link}
[![Легковес на
Swift](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Легковес на Swift"){.prog-lang-link}
[![Легковес на
TypeScript](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Легковес на TypeScript"){.prog-lang-link}
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
