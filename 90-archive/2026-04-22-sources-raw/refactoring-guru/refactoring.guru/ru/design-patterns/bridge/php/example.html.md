:::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [Паттерны
проектирования](../../../design-patterns.html){.type} /
[Мост](../../bridge.html){.type} / [PHP](../../php.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Мост](../../../../images/patterns/cards/bridge-mini9e60.png?id=b389101d8ee8e23ffa1b534c704d0774){srcset="/images/patterns/cards/bridge-mini-2x.png?id=2622384cf623ed150ee9c21a0812dd87 2x"}
:::

# **Мост** на PHP {#мост-на-php .pattern-example-title-block-title}
::::

:::::::::::::: pattern-example-body
::: pattern-example-brief
**Мост** --- это структурный паттерн, который разделяет бизнес-логику
или большой класс на несколько отдельных иерархий, которые потом можно
развивать отдельно друг от друга.

Одна из этих иерархий (абстракция) получит ссылку на объекты другой
иерархии (реализация) и будет делегировать им основную работу. Благодаря
тому, что все реализации будут следовать общему интерфейсу, их можно
будет взаимозаменять внутри абстракции.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Подробней о паттерне Мост ](../../bridge.html){.btn .btn-lg
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

**Применимость:** Паттерн Мост особенно полезен когда вам приходится
поддерживать несколько типов баз данных или работать с разными
поставщиками похожего API (например, cloud-сервисы, социальные сети
и т. д.)

**Признаки применения паттерна:** Если в программе чётко выделены классы
«управления» и несколько видов классов «платформ», причём управляющие
объекты делегируют выполнение платформам, то можно сказать, что у вас
используется Мост.
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

Этот пример показывает структуру паттерна **Мост**, а именно --- из
каких классов он состоит, какие роли эти классы выполняют и как они
взаимодействуют друг с другом.

После ознакомления со структурой, вам будет легче воспринимать второй
пример, который рассматривает реальный случай использования паттерна в
мире PHP.

#### []{#example-0--index-php .anchor} **index.php:** Пример структуры паттерна

<figure class="code">
<pre class="code" lang="php"><code>&lt;?php

namespace RefactoringGuru\Bridge\Conceptual;

/**
 * Абстракция устанавливает интерфейс для «управляющей» части двух иерархий
 * классов. Она содержит ссылку на объект из иерархии Реализации и делегирует
 * ему всю настоящую работу.
 */
class Abstraction
{
    /**
     * @var Implementation
     */
    protected $implementation;

    public function __construct(Implementation $implementation)
    {
        $this-&gt;implementation = $implementation;
    }

    public function operation(): string
    {
        return &quot;Abstraction: Base operation with:\n&quot; .
            $this-&gt;implementation-&gt;operationImplementation();
    }
}

/**
 * Можно расширить Абстракцию без изменения классов Реализации.
 */
class ExtendedAbstraction extends Abstraction
{
    public function operation(): string
    {
        return &quot;ExtendedAbstraction: Extended operation with:\n&quot; .
            $this-&gt;implementation-&gt;operationImplementation();
    }
}

/**
 * Реализация устанавливает интерфейс для всех классов реализации. Он не должен
 * соответствовать интерфейсу Абстракции. На практике оба интерфейса могут быть
 * совершенно разными. Как правило, интерфейс Реализации предоставляет только
 * примитивные операции, в то время как Абстракция определяет операции более
 * высокого уровня, основанные на этих примитивах.
 */
interface Implementation
{
    public function operationImplementation(): string;
}

/**
 * Каждая Конкретная Реализация соответствует определённой платформе и реализует
 * интерфейс Реализации с использованием API этой платформы.
 */
class ConcreteImplementationA implements Implementation
{
    public function operationImplementation(): string
    {
        return &quot;ConcreteImplementationA: Here&#39;s the result on the platform A.\n&quot;;
    }
}

class ConcreteImplementationB implements Implementation
{
    public function operationImplementation(): string
    {
        return &quot;ConcreteImplementationB: Here&#39;s the result on the platform B.\n&quot;;
    }
}

/**
 * За исключением этапа инициализации, когда объект Абстракции связывается с
 * определённым объектом Реализации, клиентский код должен зависеть только от
 * класса Абстракции. Таким образом, клиентский код может поддерживать любую
 * комбинацию абстракции и реализации.
 */
function clientCode(Abstraction $abstraction)
{
    // ...

    echo $abstraction-&gt;operation();

    // ...
}

/**
 * Клиентский код должен работать с любой предварительно сконфигурированной
 * комбинацией абстракции и реализации.
 */
$implementation = new ConcreteImplementationA();
$abstraction = new Abstraction($implementation);
clientCode($abstraction);

echo &quot;\n&quot;;

$implementation = new ConcreteImplementationB();
$abstraction = new ExtendedAbstraction($implementation);
clientCode($abstraction);</code></pre>
</figure>

#### []{#example-0--Output-txt .anchor} **Output.txt:** Результат выполнения

<figure class="code">
<pre class="code" lang="output"><code>Abstraction: Base operation with:
ConcreteImplementationA: Here&#39;s the result on the platform A.

ExtendedAbstraction: Extended operation with:
ConcreteImplementationB: Here&#39;s the result on the platform B.</code></pre>
</figure>
:::

::: {#example-1 .tab-pane role="tabpanel" aria-labelledby="example-1-tab" style="padding-top: 24px"}
## Пример из реальной жизни {#example-1-title}

В этом примере иерархия Страницы выступает как Абстракция, а иерархия
Рендера как Реализация. Объекты класса Страница монтируют веб-страницы
определённого типа, используя базовые элементы, которые предоставляются
объектом Рендер, прикреплённым к этой странице. Обе иерархии классов
разделены, поэтому можно добавить новый класс Рендер без изменения
классов страниц и наоборот.

#### []{#example-1--index-php .anchor} **index.php:** Пример из реальной жизни

<figure class="code">
<pre class="code" lang="php"><code>&lt;?php

namespace RefactoringGuru\Bridge\RealWorld;

/**
 * Абстракция.
 */
abstract class Page
{
    /**
     * @var Renderer
     */
    protected $renderer;

    /**
     * Обычно Абстракция инициализируется одним из объектов Реализации.
     */
    public function __construct(Renderer $renderer)
    {
        $this-&gt;renderer = $renderer;
    }

    /**
     * Паттерн Мост позволяет динамически заменять присоединённый объект
     * Реализации.
     */
    public function changeRenderer(Renderer $renderer): void
    {
        $this-&gt;renderer = $renderer;
    }

    /**
     * Поведение «вида» остаётся абстрактным, так как оно предоставляется только
     * классами Конкретной Абстракции.
     */
    abstract public function view(): string;
}

/**
 * Эта Конкретная Абстракция создаёт простую страницу.
 */
class SimplePage extends Page
{
    protected $title;
    protected $content;

    public function __construct(Renderer $renderer, string $title, string $content)
    {
        parent::__construct($renderer);
        $this-&gt;title = $title;
        $this-&gt;content = $content;
    }

    public function view(): string
    {
        return $this-&gt;renderer-&gt;renderParts([
            $this-&gt;renderer-&gt;renderHeader(),
            $this-&gt;renderer-&gt;renderTitle($this-&gt;title),
            $this-&gt;renderer-&gt;renderTextBlock($this-&gt;content),
            $this-&gt;renderer-&gt;renderFooter()
        ]);
    }
}

/**
 * Эта Конкретная Абстракция создаёт более сложную страницу.
 */
class ProductPage extends Page
{
    protected $product;

    public function __construct(Renderer $renderer, Product $product)
    {
        parent::__construct($renderer);
        $this-&gt;product = $product;
    }

    public function view(): string
    {
        return $this-&gt;renderer-&gt;renderParts([
            $this-&gt;renderer-&gt;renderHeader(),
            $this-&gt;renderer-&gt;renderTitle($this-&gt;product-&gt;getTitle()),
            $this-&gt;renderer-&gt;renderTextBlock($this-&gt;product-&gt;getDescription()),
            $this-&gt;renderer-&gt;renderImage($this-&gt;product-&gt;getImage()),
            $this-&gt;renderer-&gt;renderTextBlock(&#39;$&#39; . number_format($this-&gt;product-&gt;getPrice(), 2)),
            $this-&gt;renderer-&gt;renderLink(&quot;/cart/add/&quot; . $this-&gt;product-&gt;getId(), &quot;Add to cart&quot;),
            $this-&gt;renderer-&gt;renderFooter()
        ]);
    }
}

/**
 * Вспомогательный класс для класса ProductPage.
 */
class Product
{
    private $id, $title, $description, $image, $price;

    public function __construct(
        string $id,
        string $title,
        string $description,
        string $image,
        float $price
    ) {
        $this-&gt;id = $id;
        $this-&gt;title = $title;
        $this-&gt;description = $description;
        $this-&gt;image = $image;
        $this-&gt;price = $price;
    }

    public function getId(): string
    {
        return $this-&gt;id;
    }

    public function getTitle(): string
    {
        return $this-&gt;title;
    }

    public function getDescription(): string
    {
        return $this-&gt;description;
    }

    public function getImage(): string
    {
        return $this-&gt;image;
    }

    public function getPrice(): float
    {
        return $this-&gt;price;
    }
}


/**
 * Реализация объявляет набор «реальных», «под капотом», «платформенных»
 * методов.
 *
 * В этом случае Реализация перечисляет методы рендеринга, которые используются
 * для создания веб-страниц. Разные Абстракции могут использовать разные методы
 * Реализации.
 */
interface Renderer
{
    public function renderTitle(string $title): string;

    public function renderTextBlock(string $text): string;

    public function renderImage(string $url): string;

    public function renderLink(string $url, string $title): string;

    public function renderHeader(): string;

    public function renderFooter(): string;

    public function renderParts(array $parts): string;
}

/**
 * Эта Конкретная Реализация отображает веб-страницу в виде HTML.
 */
class HTMLRenderer implements Renderer
{
    public function renderTitle(string $title): string
    {
        return &quot;&lt;h1&gt;$title&lt;/h1&gt;&quot;;
    }

    public function renderTextBlock(string $text): string
    {
        return &quot;&lt;div class=&#39;text&#39;&gt;$text&lt;/div&gt;&quot;;
    }

    public function renderImage(string $url): string
    {
        return &quot;&lt;img src=&#39;$url&#39;&gt;&quot;;
    }

    public function renderLink(string $url, string $title): string
    {
        return &quot;&lt;a href=&#39;$url&#39;&gt;$title&lt;/a&gt;&quot;;
    }

    public function renderHeader(): string
    {
        return &quot;&lt;html&gt;&lt;body&gt;&quot;;
    }

    public function renderFooter(): string
    {
        return &quot;&lt;/body&gt;&lt;/html&gt;&quot;;
    }

    public function renderParts(array $parts): string
    {
        return implode(&quot;\n&quot;, $parts);
    }
}

/**
 * Эта Конкретная Реализация отображает веб-страницу в виде строк JSON.
 */
class JsonRenderer implements Renderer
{
    public function renderTitle(string $title): string
    {
        return &#39;&quot;title&quot;: &quot;&#39; . $title . &#39;&quot;&#39;;
    }

    public function renderTextBlock(string $text): string
    {
        return &#39;&quot;text&quot;: &quot;&#39; . $text . &#39;&quot;&#39;;
    }

    public function renderImage(string $url): string
    {
        return &#39;&quot;img&quot;: &quot;&#39; . $url . &#39;&quot;&#39;;
    }

    public function renderLink(string $url, string $title): string
    {
        return &#39;&quot;link&quot;: {&quot;href&quot;: &quot;&#39; . $url . &#39;&quot;, &quot;title&quot;: &quot;&#39; . $title . &#39;&quot;}&#39;;
    }

    public function renderHeader(): string
    {
        return &#39;&#39;;
    }

    public function renderFooter(): string
    {
        return &#39;&#39;;
    }

    public function renderParts(array $parts): string
    {
        return &quot;{\n&quot; . implode(&quot;,\n&quot;, array_filter($parts)) . &quot;\n}&quot;;
    }
}

/**
 * Клиентский код имеет дело только с объектами Абстракции.
 */
function clientCode(Page $page)
{
    // ...

    echo $page-&gt;view();

    // ...
}

/**
 * Клиентский код может выполняться с любой предварительно сконфигурированной
 * комбинацией Абстракция+Реализация.
 */
$HTMLRenderer = new HTMLRenderer();
$JSONRenderer = new JsonRenderer();

$page = new SimplePage($HTMLRenderer, &quot;Home&quot;, &quot;Welcome to our website!&quot;);
echo &quot;HTML view of a simple content page:\n&quot;;
clientCode($page);
echo &quot;\n\n&quot;;

/**
 * При необходимости Абстракция может заменить связанную Реализацию во время
 * выполнения.
 */
$page-&gt;changeRenderer($JSONRenderer);
echo &quot;JSON view of a simple content page, rendered with the same client code:\n&quot;;
clientCode($page);
echo &quot;\n\n&quot;;


$product = new Product(
    &quot;123&quot;,
    &quot;Star Wars, episode1&quot;,
    &quot;A long time ago in a galaxy far, far away...&quot;,
    &quot;/images/star-wars.jpeg&quot;,
    39.95
);

$page = new ProductPage($HTMLRenderer, $product);
echo &quot;HTML view of a product page, same client code:\n&quot;;
clientCode($page);
echo &quot;\n\n&quot;;

$page-&gt;changeRenderer($JSONRenderer);
echo &quot;JSON view of a simple content page, with the same client code:\n&quot;;
clientCode($page);</code></pre>
</figure>

#### []{#example-1--Output-txt .anchor} **Output.txt:** Результат выполнения

<figure class="code">
<pre class="code" lang="output"><code>HTML view of a simple content page:
&lt;html&gt;&lt;body&gt;
&lt;h1&gt;Home&lt;/h1&gt;
&lt;div class=&#39;text&#39;&gt;Welcome to our website!&lt;/div&gt;
&lt;/body&gt;&lt;/html&gt;

JSON view of a simple content page, rendered with the same client code:
{
&quot;title&quot;: &quot;Home&quot;,
&quot;text&quot;: &quot;Welcome to our website!&quot;
}

HTML view of a product page, same client code:
&lt;html&gt;&lt;body&gt;
&lt;h1&gt;Star Wars, episode1&lt;/h1&gt;
&lt;div class=&#39;text&#39;&gt;A long time ago in a galaxy far, far away...&lt;/div&gt;
&lt;img src=&#39;/images/star-wars.jpeg&#39;&gt;
&lt;a href=&#39;/cart/add/123&#39;&gt;Add to cart&lt;/a&gt;
&lt;/body&gt;&lt;/html&gt;

JSON view of a simple content page, with the same client code:
{
&quot;title&quot;: &quot;Star Wars, episode1&quot;,
&quot;text&quot;: &quot;A long time ago in a galaxy far, far away...&quot;,
&quot;img&quot;: &quot;/images/star-wars.jpeg&quot;,
&quot;link&quot;: {&quot;href&quot;: &quot;/cart/add/123&quot;, &quot;title&quot;: &quot;Add to cart&quot;}
}</code></pre>
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

[Компоновщик на PHP []{.fa
.fa-arrow-right}](../../composite/php/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Вернуться назад

[[]{.fa .fa-arrow-left} Адаптер на
PHP](../../adapter/php/example.html){.btn .btn-default rel="prev"}
:::

## **Мост** на других языках программирования

[![Мост на
C#](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Мост на C#"){.prog-lang-link}
[![Мост на
C++](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Мост на C++"){.prog-lang-link}
[![Мост на
Go](../../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Мост на Go"){.prog-lang-link}
[![Мост на
Java](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Мост на Java"){.prog-lang-link}
[![Мост на
Python](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Мост на Python"){.prog-lang-link}
[![Мост на
Ruby](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Мост на Ruby"){.prog-lang-link}
[![Мост на
Rust](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Мост на Rust"){.prog-lang-link}
[![Мост на
Swift](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Мост на Swift"){.prog-lang-link}
[![Мост на
TypeScript](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Мост на TypeScript"){.prog-lang-link}
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
