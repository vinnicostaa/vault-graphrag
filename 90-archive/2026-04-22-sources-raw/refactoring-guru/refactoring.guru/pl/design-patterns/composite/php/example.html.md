:::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [Wzorce
projektowe](../../../design-patterns.html){.type} /
[Kompozyt](../../composite.html){.type} / [PHP](../../php.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Kompozyt](../../../../images/patterns/cards/composite-minib543.png?id=a369d98d18b417f255d04568fd0131b8){srcset="/images/patterns/cards/composite-mini-2x.png?id=3f7f811fefeb0b64f6774746eb42af09 2x"}
:::

# **Kompozyt** w języku PHP {#kompozyt-w-języku-php .pattern-example-title-block-title}
::::

:::::::::::::: pattern-example-body
::: pattern-example-brief
**Kompozyt** to strukturalny wzorzec projektowy umożliwiający
komponowanie struktury drzewiastej z obiektów i traktowanie jej jak
pojedynczy obiekt.

Kompozyt stał się dość popularnym rozwiązaniem wielu problemów gdzie w
grę wchodzi struktura drzewa. Zaletą tego wzorca jest możliwość
uruchamiania metod rekurencyjnie na wszystkich elementach struktury i
sumowanie wyników ich działania.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Dowiedz się więcej o Kompozyt ](../../composite.html){.btn .btn-lg
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

**Przykłady użycia:** Wzorzec Kompozyt jest często stosowany w
kontekście drzew obiektów. Najprostszym przykładem jest zastosowanie
tego wzorca wobec elementów drzewa Obiektowego Modelu Dokumentu (ang.
DOM), gdzie złożone i proste obiekty struktury traktowane są tak samo.

**Identyfikacja:** Jeśli klasy wszystkich obiektów w drzewie należą do
jednej hierarchii to najprawdopodobniej mamy do czynienia z kompozytem.
Jeśli dodatkowo metody tych klas delegują zadania obiektom-dzieciom
wchodzącym w skład tego drzewa i robią to za pośrednictwem klasy bazowej
lub bazowego interfejsu hierarchii, to na pewno jest to kompozyt.
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

Poniższy przykład ilustruje strukturę wzorca **Kompozyt** ze szczególnym
naciskiem na następujące kwestie:

-   Z jakich składa się klas?
-   Jakie role pełnią te klasy?
-   W jaki sposób elementy wzorca są ze sobą powiązane?

Poznawszy strukturę wzorca będzie ci łatwiej zrozumieć następujący
przykład, oparty na prawdziwym przypadku użycia PHP.

#### []{#example-0--index-php .anchor} **index.php:** Przykład koncepcyjny

<figure class="code">
<pre class="code" lang="php"><code>&lt;?php

namespace RefactoringGuru\Composite\Conceptual;

/**
 * The base Component class declares common operations for both simple and
 * complex objects of a composition.
 */
abstract class Component
{
    /**
     * @var Component|null
     */
    protected $parent;

    /**
     * Optionally, the base Component can declare an interface for setting and
     * accessing a parent of the component in a tree structure. It can also
     * provide some default implementation for these methods.
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
     * In some cases, it would be beneficial to define the child-management
     * operations right in the base Component class. This way, you won&#39;t need to
     * expose any concrete component classes to the client code, even during the
     * object tree assembly. The downside is that these methods will be empty
     * for the leaf-level components.
     */
    public function add(Component $component): void
    {
    }

    public function remove(Component $component): void
    {
    }

    /**
     * You can provide a method that lets the client code figure out whether a
     * component can bear children.
     */
    public function isComposite(): bool
    {
        return false;
    }

    /**
     * The base Component may implement some default behavior or leave it to
     * concrete classes (by declaring the method containing the behavior as
     * &quot;abstract&quot;).
     */
    abstract public function operation(): string;
}

/**
 * The Leaf class represents the end objects of a composition. A leaf can&#39;t have
 * any children.
 *
 * Usually, it&#39;s the Leaf objects that do the actual work, whereas Composite
 * objects only delegate to their sub-components.
 */
class Leaf extends Component
{
    public function operation(): string
    {
        return &quot;Leaf&quot;;
    }
}

/**
 * The Composite class represents the complex components that may have children.
 * Usually, the Composite objects delegate the actual work to their children and
 * then &quot;sum-up&quot; the result.
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
     * A composite object can add or remove other components (both simple or
     * complex) to or from its child list.
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
     * The Composite executes its primary logic in a particular way. It
     * traverses recursively through all its children, collecting and summing
     * their results. Since the composite&#39;s children pass these calls to their
     * children and so forth, the whole object tree is traversed as a result.
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
 * The client code works with all of the components via the base interface.
 */
function clientCode(Component $component)
{
    // ...

    echo &quot;RESULT: &quot; . $component-&gt;operation();

    // ...
}

/**
 * This way the client code can support the simple leaf components...
 */
$simple = new Leaf();
echo &quot;Client: I&#39;ve got a simple component:\n&quot;;
clientCode($simple);
echo &quot;\n\n&quot;;

/**
 * ...as well as the complex composites.
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
 * Thanks to the fact that the child-management operations are declared in the
 * base Component class, the client code can work with any component, simple or
 * complex, without depending on their concrete classes.
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

#### []{#example-0--Output-txt .anchor} **Output.txt:** Wynik działania

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
## Przykład z prawdziwego życia {#example-1-title}

Wzorzec **Kompozyt** może usprawnić działanie z dowolną rekursywną
drzewiastą strukturą. Drzewo HTML DOM jest przykładem takiej
struktury --- poszczególne pola służące wprowadzaniu danych mogą pełnić
rolę liści, zaś złożone elementy --- jak formularze i zestawy
formularzy --- pełnią rolę kompozytów.

Mając to na uwadze, można skorzystać ze wzorca Kompozyt w celu nadania
różnych funkcjonalności całemu drzewu HTML w taki sam sposób jak ich
pojedynczym elementom bez konieczności sprzęgania swojego kodu z
konkretnymi klasami drzewa DOM. Przykłady takich funkcjonalności to
renderowanie elementów DOM, eksportowanie ich do różnych formatów,
walidacja ich części, itp.

Stosując wzorzec Kompozyt nie trzeba sprawdzać czy dany element jest
typu prostego czy złożonego przed uruchomieniem jakiejś jego funkcji. W
zależności od typu elementu, wywołanie albo zostanie przetworzone od
razu, albo przekazane dalej wzdłuż kolejnych węzłów drzewa.

#### []{#example-1--index-php .anchor} **index.php:** Przykład z prawdziwego życia

<figure class="code">
<pre class="code" lang="php"><code>&lt;?php

namespace RefactoringGuru\Composite\RealWorld;

/**
 * The base Component class declares an interface for all concrete components,
 * both simple and complex.
 *
 * In our example, we&#39;ll be focusing on the rendering behavior of DOM elements.
 */
abstract class FormElement
{
    /**
     * We can anticipate that all DOM elements require these 3 fields.
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
     * Each concrete DOM element must provide its rendering implementation, but
     * we can safely assume that all of them are returning strings.
     */
    abstract public function render(): string;
}

/**
 * This is a Leaf component. Like all the Leaves, it can&#39;t have any children.
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
     * Since Leaf components don&#39;t have any children that may handle the bulk of
     * the work for them, usually it is the Leaves who do the most of the heavy-
     * lifting within the Composite pattern.
     */
    public function render(): string
    {
        return &quot;&lt;label for=\&quot;{$this-&gt;name}\&quot;&gt;{$this-&gt;title}&lt;/label&gt;\n&quot; .
            &quot;&lt;input name=\&quot;{$this-&gt;name}\&quot; type=\&quot;{$this-&gt;type}\&quot; value=\&quot;{$this-&gt;data}\&quot;&gt;\n&quot;;
    }
}

/**
 * The base Composite class implements the infrastructure for managing child
 * objects, reused by all Concrete Composites.
 */
abstract class FieldComposite extends FormElement
{
    /**
     * @var FormElement[]
     */
    protected $fields = [];

    /**
     * The methods for adding/removing sub-objects.
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
     * Whereas a Leaf&#39;s method just does the job, the Composite&#39;s method almost
     * always has to take its sub-objects into account.
     *
     * In this case, the composite can accept structured data.
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
     * The same logic applies to the getter. It returns the structured data of
     * the composite itself (if any) and all the children data.
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
     * The base implementation of the Composite&#39;s rendering simply combines
     * results of all children. Concrete Composites will be able to reuse this
     * implementation in their real rendering implementations.
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
 * The fieldset element is a Concrete Composite.
 */
class Fieldset extends FieldComposite
{
    public function render(): string
    {
        // Note how the combined rendering result of children is incorporated
        // into the fieldset tag.
        $output = parent::render();

        return &quot;&lt;fieldset&gt;&lt;legend&gt;{$this-&gt;title}&lt;/legend&gt;\n$output&lt;/fieldset&gt;\n&quot;;
    }
}

/**
 * And so is the form element.
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
 * The client code gets a convenient interface for building complex tree
 * structures.
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
 * The form structure can be filled with data from various sources. The Client
 * doesn&#39;t have to traverse through all form fields to assign data to various
 * fields since the form itself can handle that.
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
 * The client code can work with form elements using the abstract interface.
 * This way, it doesn&#39;t matter whether the client works with a simple component
 * or a complex composite tree.
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

#### []{#example-1--Output-txt .anchor} **Output.txt:** Wynik działania

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
[Przykład koncepcyjny](#example-0){#example-0-tab-bottom .nav-item
.nav-link .active toggle="tab" role="tab" aria-controls="example-0"
aria-selected="true"} [Przykład z prawdziwego
życia](#example-1){#example-1-tab-bottom .nav-item .nav-link
toggle="tab" role="tab" aria-controls="example-1" aria-selected="true"}
:::

::: next
#### Czytaj dalej

[Dekorator w języku PHP []{.fa
.fa-arrow-right}](../../decorator/php/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Wróć

[[]{.fa .fa-arrow-left} Most w języku
PHP](../../bridge/php/example.html){.btn .btn-default rel="prev"}
:::

## **Kompozyt** w innych językach

[![Kompozyt w języku
C#](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Kompozyt w języku C#"){.prog-lang-link}
[![Kompozyt w języku
C++](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Kompozyt w języku C++"){.prog-lang-link}
[![Kompozyt w języku
Go](../../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Kompozyt w języku Go"){.prog-lang-link}
[![Kompozyt w języku
Java](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Kompozyt w języku Java"){.prog-lang-link}
[![Kompozyt w języku
Python](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Kompozyt w języku Python"){.prog-lang-link}
[![Kompozyt w języku
Ruby](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Kompozyt w języku Ruby"){.prog-lang-link}
[![Kompozyt w języku
Rust](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Kompozyt w języku Rust"){.prog-lang-link}
[![Kompozyt w języku
Swift](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Kompozyt w języku Swift"){.prog-lang-link}
[![Kompozyt w języku
TypeScript](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Kompozyt w języku TypeScript"){.prog-lang-link}
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
