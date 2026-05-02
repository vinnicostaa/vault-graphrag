:::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [Паттерны
проектирования](../../../design-patterns.html){.type} /
[Компоновщик](../../composite.html){.type} /
[PHP](../../php.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Компоновщик](../../../../images/patterns/cards/composite-minib543.png?id=a369d98d18b417f255d04568fd0131b8){srcset="/images/patterns/cards/composite-mini-2x.png?id=3f7f811fefeb0b64f6774746eb42af09 2x"}
:::

# **Компоновщик** на PHP {#компоновщик-на-php .pattern-example-title-block-title}
::::

:::::::::::::: pattern-example-body
::: pattern-example-brief
**Компоновщик** --- это структурный паттерн, который позволяет создавать
дерево объектов и работать с ним так же, как и с единичным объектом.

Компоновщик давно стал синонимом всех задач, связанных с построением
дерева объектов. Все операции компоновщика основаны на рекурсии и
«суммировании» результатов на ветвях дерева.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Подробней о паттерне Компоновщик ](../../composite.html){.btn .btn-lg
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

**Применимость:** Паттерн Компоновщик встречается в любых задачах,
которые связаны с построением дерева. Самый простой пример --- составные
элементы DOM-дерева, которые тоже можно рассматривать как поддерево.

**Признаки применения паттерна:** Если из объектов строится древовидная
структура, и со всеми объектами дерева, как и с самим деревом работают
через общий интерфейс.
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

Этот пример показывает структуру паттерна **Компоновщик**, а именно ---
из каких классов он состоит, какие роли эти классы выполняют и как они
взаимодействуют друг с другом.

После ознакомления со структурой, вам будет легче воспринимать второй
пример, который рассматривает реальный случай использования паттерна в
мире PHP.

#### []{#example-0--index-php .anchor} **index.php:** Пример структуры паттерна

<figure class="code">
<pre class="code" lang="php"><code>&lt;?php

namespace RefactoringGuru\Composite\Conceptual;

/**
 * Базовый класс Компонент объявляет общие операции как для простых, так и для
 * сложных объектов структуры.
 */
abstract class Component
{
    /**
     * @var Component|null
     */
    protected $parent;

    /**
     * При необходимости базовый Компонент может объявить интерфейс для
     * установки и получения родителя компонента в древовидной структуре. Он
     * также может предоставить некоторую реализацию по умолчанию для этих
     * методов.
     */
    public function setParent(?Component $parent)
    {
        $this-&gt;parent = $parent;
    }

    public function getParent(): Component
    {
        return $this-&gt;parent;
    }

    /**
     * В некоторых случаях целесообразно определить операции управления
     * потомками прямо в базовом классе Компонент. Таким образом, вам не нужно
     * будет предоставлять конкретные классы компонентов клиентскому коду, даже
     * во время сборки дерева объектов. Недостаток такого подхода в том, что эти
     * методы будут пустыми для компонентов уровня листа.
     */
    public function add(Component $component): void
    {
    }

    public function remove(Component $component): void
    {
    }

    /**
     * Вы можете предоставить метод, который позволит клиентскому коду понять,
     * может ли компонент иметь вложенные объекты.
     */
    public function isComposite(): bool
    {
        return false;
    }

    /**
     * Базовый Компонент может сам реализовать некоторое поведение по умолчанию
     * или поручить это конкретным классам, объявив метод, содержащий поведение
     * абстрактным.
     */
    abstract public function operation(): string;
}

/**
 * Класс Лист представляет собой конечные объекты структуры. Лист не может иметь
 * вложенных компонентов.
 *
 * Обычно объекты Листьев выполняют фактическую работу, тогда как объекты
 * Контейнера лишь делегируют работу своим подкомпонентам.
 */
class Leaf extends Component
{
    public function operation(): string
    {
        return &quot;Leaf&quot;;
    }
}

/**
 * Класс Контейнер содержит сложные компоненты, которые могут иметь вложенные
 * компоненты. Обычно объекты Контейнеры делегируют фактическую работу своим
 * детям, а затем «суммируют» результат.
 */
class Composite extends Component
{
    /**
     * @var \SplObjectStorage
     */
    protected $children;

    public function __construct()
    {
        $this-&gt;children = new \SplObjectStorage();
    }

    /**
     * Объект контейнера может как добавлять компоненты в свой список вложенных
     * компонентов, так и удалять их, как простые, так и сложные.
     */
    public function add(Component $component): void
    {
        $this-&gt;children-&gt;attach($component);
        $component-&gt;setParent($this);
    }

    public function remove(Component $component): void
    {
        $this-&gt;children-&gt;detach($component);
        $component-&gt;setParent(null);
    }

    public function isComposite(): bool
    {
        return true;
    }

    /**
     * Контейнер выполняет свою основную логику особым образом. Он проходит
     * рекурсивно через всех своих детей, собирая и суммируя их результаты.
     * Поскольку потомки контейнера передают эти вызовы своим потомкам и так
     * далее, в результате обходится всё дерево объектов.
     */
    public function operation(): string
    {
        $results = [];
        foreach ($this-&gt;children as $child) {
            $results[] = $child-&gt;operation();
        }

        return &quot;Branch(&quot; . implode(&quot;+&quot;, $results) . &quot;)&quot;;
    }
}

/**
 * Клиентский код работает со всеми компонентами через базовый интерфейс.
 */
function clientCode(Component $component)
{
    // ...

    echo &quot;RESULT: &quot; . $component-&gt;operation();

    // ...
}

/**
 * Таким образом, клиентский код может поддерживать простые компоненты-листья...
 */
$simple = new Leaf();
echo &quot;Client: I&#39;ve got a simple component:\n&quot;;
clientCode($simple);
echo &quot;\n\n&quot;;

/**
 * ...а также сложные контейнеры.
 */
$tree = new Composite();
$branch1 = new Composite();
$branch1-&gt;add(new Leaf());
$branch1-&gt;add(new Leaf());
$branch2 = new Composite();
$branch2-&gt;add(new Leaf());
$tree-&gt;add($branch1);
$tree-&gt;add($branch2);
echo &quot;Client: Now I&#39;ve got a composite tree:\n&quot;;
clientCode($tree);
echo &quot;\n\n&quot;;

/**
 * Благодаря тому, что операции управления потомками объявлены в базовом классе
 * Компонента, клиентский код может работать как с простыми, так и со сложными
 * компонентами, вне зависимости от их конкретных классов.
 */
function clientCode2(Component $component1, Component $component2)
{
    // ...

    if ($component1-&gt;isComposite()) {
        $component1-&gt;add($component2);
    }
    echo &quot;RESULT: &quot; . $component1-&gt;operation();

    // ...
}

echo &quot;Client: I don&#39;t need to check the components classes even when managing the tree:\n&quot;;
clientCode2($tree, $simple);</code></pre>
</figure>

#### []{#example-0--Output-txt .anchor} **Output.txt:** Результат выполнения

<figure class="code">
<pre class="code" lang="output"><code>Client: I get a simple component:
RESULT: Leaf

Client: Now I get a composite tree:
RESULT: Branch(Branch(Leaf+Leaf)+Branch(Leaf))

Client: I don&#39;t need to check the components classes even when managing the tree::
RESULT: Branch(Branch(Leaf+Leaf)+Branch(Leaf)+Leaf)</code></pre>
</figure>
:::

::: {#example-1 .tab-pane role="tabpanel" aria-labelledby="example-1-tab" style="padding-top: 24px"}
## Пример из реальной жизни {#example-1-title}

Паттерн **Компоновщик** может упростить работу с любыми древовидными
рекурсивными структурами. Примером такой структуры является DOM-дерево
HTML. Например, в то время как различные входные элементы могут служить
листьями, сложные элементы, такие как формы и наборы полей, играют роль
контейнеров.

Имея это в виду, вы можете использовать паттерн Компоновщик для
применения различных типов поведения ко всему дереву HTML точно так же,
как и к его внутренним элементам, не привязывая ваш код к конкретным
классам дерева DOM. Примерами такого поведения может быть рендеринг
элементов DOM, их экспорт в различные форматы, проверка достоверности их
частей и т. д.

С паттерном Компоновщик вам не нужно проверять, является ли тип элемента
простым или сложным, перед реализацией поведения. В зависимости от типа
элемента, оно либо сразу же выполняется, либо передаётся всем дочерним
элементам.

#### []{#example-1--index-php .anchor} **index.php:** Пример из реальной жизни

<figure class="code">
<pre class="code" lang="php"><code>&lt;?php

namespace RefactoringGuru\Composite\RealWorld;

/**
 * Базовый класс Компонент объявляет интерфейс для всех конкретных компонентов,
 * как простых, так и сложных.
 *
 * В нашем примере мы сосредоточимся на поведении рендеринга элементов DOM.
 */
abstract class FormElement
{
    /**
     * Можно предположить, что всем элементам DOM будут нужны эти 3 поля.
     */
    protected $name;
    protected $title;
    protected $data;

    public function __construct(string $name, string $title)
    {
        $this-&gt;name = $name;
        $this-&gt;title = $title;
    }

    public function getName(): string
    {
        return $this-&gt;name;
    }

    public function setData($data): void
    {
        $this-&gt;data = $data;
    }

    public function getData(): array
    {
        return $this-&gt;data;
    }

    /**
     * Каждый конкретный элемент DOM должен предоставить свою реализацию
     * рендеринга, но мы можем с уверенностью предположить, что все они будут
     * возвращать строки.
     */
    abstract public function render(): string;
}

/**
 * Это компонент-Лист. Как и все Листья, он не может иметь вложенных
 * компонентов.
 */
class Input extends FormElement
{
    private $type;

    public function __construct(string $name, string $title, string $type)
    {
        parent::__construct($name, $title);
        $this-&gt;type = $type;
    }

    /**
     * Поскольку у компонентов-Листьев нет вложенных компонентов, которые могут
     * выполнять за них основную часть работы, обычно Листья делают большую
     * часть тяжёлой работы внутри паттерна Компоновщик.
     */
    public function render(): string
    {
        return &quot;&lt;label for=\&quot;{$this-&gt;name}\&quot;&gt;{$this-&gt;title}&lt;/label&gt;\n&quot; .
            &quot;&lt;input name=\&quot;{$this-&gt;name}\&quot; type=\&quot;{$this-&gt;type}\&quot; value=\&quot;{$this-&gt;data}\&quot;&gt;\n&quot;;
    }
}

/**
 * Базовый класс Контейнер реализует инфраструктуру для управления дочерними
 * объектами, повторно используемую всеми Конкретными Контейнерами.
 */
abstract class FieldComposite extends FormElement
{
    /**
     * @var FormElement[]
     */
    protected $fields = [];

    /**
     * Методы добавления/удаления подобъектов.
     */
    public function add(FormElement $field): void
    {
        $name = $field-&gt;getName();
        $this-&gt;fields[$name] = $field;
    }

    public function remove(FormElement $component): void
    {
        $this-&gt;fields = array_filter($this-&gt;fields, function ($child) use ($component) {
            return $child != $component;
        });
    }

    /**
     * В то время как метод Листа просто выполняет эту работу, метод Контейнера
     * почти всегда должен учитывать его подобъекты.
     *
     * В этом случае контейнер может принимать структурированные данные.
     *
     * @param array $data
     */
    public function setData($data): void
    {
        foreach ($this-&gt;fields as $name =&gt; $field) {
            if (isset($data[$name])) {
                $field-&gt;setData($data[$name]);
            }
        }
    }

    /**
     * Та же логика применима и к получателю. Он возвращает структурированные
     * данные самого контейнера (если они есть), а также все дочерние данные.
     */
    public function getData(): array
    {
        $data = [];

        foreach ($this-&gt;fields as $name =&gt; $field) {
            $data[$name] = $field-&gt;getData();
        }

        return $data;
    }

    /**
     * Базовая реализация рендеринга Контейнера просто объединяет результаты
     * всех дочерних элементов. Конкретные Контейнеры смогут повторно
     * использовать эту реализацию в своих реальных реализациях рендеринга.
     */
    public function render(): string
    {
        $output = &quot;&quot;;

        foreach ($this-&gt;fields as $name =&gt; $field) {
            $output .= $field-&gt;render();
        }

        return $output;
    }
}

/**
 * Элемент fieldset представляет собой Конкретный Контейнер.
 */
class Fieldset extends FieldComposite
{
    public function render(): string
    {
        // Обратите внимание, как комбинированный результат рендеринга потомков
        // включается в тег fieldset.
        $output = parent::render();

        return &quot;&lt;fieldset&gt;&lt;legend&gt;{$this-&gt;title}&lt;/legend&gt;\n$output&lt;/fieldset&gt;\n&quot;;
    }
}

/**
 * Так же как и элемент формы.
 */
class Form extends FieldComposite
{
    protected $url;

    public function __construct(string $name, string $title, string $url)
    {
        parent::__construct($name, $title);
        $this-&gt;url = $url;
    }

    public function render(): string
    {
        $output = parent::render();
        return &quot;&lt;form action=\&quot;{$this-&gt;url}\&quot;&gt;\n&lt;h3&gt;{$this-&gt;title}&lt;/h3&gt;\n$output&lt;/form&gt;\n&quot;;
    }
}

/**
 * Клиентский код получает удобный интерфейс для построения сложных древовидных
 * структур.
 */
function getProductForm(): FormElement
{
    $form = new Form(&#39;product&#39;, &quot;Add product&quot;, &quot;/product/add&quot;);
    $form-&gt;add(new Input(&#39;name&#39;, &quot;Name&quot;, &#39;text&#39;));
    $form-&gt;add(new Input(&#39;description&#39;, &quot;Description&quot;, &#39;text&#39;));

    $picture = new Fieldset(&#39;photo&#39;, &quot;Product photo&quot;);
    $picture-&gt;add(new Input(&#39;caption&#39;, &quot;Caption&quot;, &#39;text&#39;));
    $picture-&gt;add(new Input(&#39;image&#39;, &quot;Image&quot;, &#39;file&#39;));
    $form-&gt;add($picture);

    return $form;
}

/**
 * Структура формы может быть заполнена данными из разных источников. Клиент не
 * должен проходить через все поля формы, чтобы назначить данные различным
 * полям, так как форма сама может справиться с этим.
 */
function loadProductData(FormElement $form)
{
    $data = [
        &#39;name&#39; =&gt; &#39;Apple MacBook&#39;,
        &#39;description&#39; =&gt; &#39;A decent laptop.&#39;,
        &#39;photo&#39; =&gt; [
            &#39;caption&#39; =&gt; &#39;Front photo.&#39;,
            &#39;image&#39; =&gt; &#39;photo1.png&#39;,
        ],
    ];

    $form-&gt;setData($data);
}

/**
 * Клиентский код может работать с элементами формы, используя абстрактный
 * интерфейс. Таким образом, не имеет значения, работает ли клиент с простым
 * компонентом или сложным составным деревом.
 */
function renderProduct(FormElement $form)
{
    // ..

    echo $form-&gt;render();

    // ..
}

$form = getProductForm();
loadProductData($form);
renderProduct($form);</code></pre>
</figure>

#### []{#example-1--Output-txt .anchor} **Output.txt:** Результат выполнения

<figure class="code">
<pre class="code" lang="output"><code>&lt;form action=&quot;/product/add&quot;&gt;
&lt;h3&gt;Add product&lt;/h3&gt;
&lt;label for=&quot;name&quot;&gt;Name&lt;/label&gt;
&lt;input name=&quot;name&quot; type=&quot;text&quot; value=&quot;Apple MacBook&quot;&gt;
&lt;label for=&quot;description&quot;&gt;Description&lt;/label&gt;
&lt;input name=&quot;description&quot; type=&quot;text&quot; value=&quot;A decent laptop.&quot;&gt;
&lt;fieldset&gt;&lt;legend&gt;Product photo&lt;/legend&gt;
&lt;label for=&quot;caption&quot;&gt;Caption&lt;/label&gt;
&lt;input name=&quot;caption&quot; type=&quot;text&quot; value=&quot;Front photo.&quot;&gt;
&lt;label for=&quot;image&quot;&gt;Image&lt;/label&gt;
&lt;input name=&quot;image&quot; type=&quot;file&quot; value=&quot;photo1.png&quot;&gt;
&lt;/fieldset&gt;
&lt;/form&gt;</code></pre>
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

[Декоратор на PHP []{.fa
.fa-arrow-right}](../../decorator/php/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Вернуться назад

[[]{.fa .fa-arrow-left} Мост на PHP](../../bridge/php/example.html){.btn
.btn-default rel="prev"}
:::

## **Компоновщик** на других языках программирования

[![Компоновщик на
C#](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Компоновщик на C#"){.prog-lang-link}
[![Компоновщик на
C++](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Компоновщик на C++"){.prog-lang-link}
[![Компоновщик на
Go](../../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Компоновщик на Go"){.prog-lang-link}
[![Компоновщик на
Java](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Компоновщик на Java"){.prog-lang-link}
[![Компоновщик на
Python](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Компоновщик на Python"){.prog-lang-link}
[![Компоновщик на
Ruby](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Компоновщик на Ruby"){.prog-lang-link}
[![Компоновщик на
Rust](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Компоновщик на Rust"){.prog-lang-link}
[![Компоновщик на
Swift](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Компоновщик на Swift"){.prog-lang-link}
[![Компоновщик на
TypeScript](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Компоновщик на TypeScript"){.prog-lang-link}
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
