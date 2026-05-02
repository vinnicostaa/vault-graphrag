:::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [Patrones de
diseño](../../../design-patterns.html){.type} /
[Composite](../../composite.html){.type} / [PHP](../../php.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Composite](../../../../images/patterns/cards/composite-minib543.png?id=a369d98d18b417f255d04568fd0131b8){srcset="/images/patterns/cards/composite-mini-2x.png?id=3f7f811fefeb0b64f6774746eb42af09 2x"}
:::

# **Composite** en PHP {#composite-en-php .pattern-example-title-block-title}
::::

:::::::::::::: pattern-example-body
::: pattern-example-brief
**Composite** es un patrón de diseño estructural que permite componer
objetos en una estructura en forma de árbol y trabajar con ella como si
fuera un objeto único.

El patrón Composite se convirtió en una solución muy popular para la
mayoría de problemas que requieren la creación de una estructura de
árbol. La gran característica del Composite es la capacidad para
ejecutar métodos de forma recursiva por toda la estructura de árbol y
recapitular los resultados.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Aprende más sobre el patrón Composite ](../../composite.html){.btn
.btn-lg .btn-primary}
:::

::::::::::: {.sidebar-navigation .with-scroll}
::: en-title
Navegación
:::

::: en-intro
 [Intro](#)
:::

::: en-intro
 [Ejemplo conceptual](#example-0)
:::

::: en-file
 [index](#example-0--index-php)
:::

::: en-file
 [Output](#example-0--Output-txt)
:::

::: en-intro
 [Ejemplo del mundo real](#example-1)
:::

::: en-file
 [index](#example-1--index-php)
:::

::: en-file
 [Output](#example-1--Output-txt)
:::
:::::::::::

**Complejidad:** []{.fa-stars}

**Popularidad:** []{.fa-stars}

**Ejemplos de uso:** El patrón Composite se utiliza habitualmente al
trabajar con árboles de objetos. El ejemplo más sencillo sería aplicar
el patrón a elementos del árbol DOM, trabajando con elementos compuestos
y simples del árbol de la misma manera.

**Identificación:** El Composite es fácil de reconocer por los métodos
de comportamiento que toman una instancia del mismo tipo
abstracto/interfaz y lo hacen una estructura de árbol.
::::::::::::::

::: {#nav-tab .nav .nav-tabs .nav-justified role="tablist"}
[Ejemplo conceptual](#example-0){#example-0-tab .nav-item .nav-link
.active toggle="tab" role="tab" aria-controls="example-0"
aria-selected="true"} [Ejemplo del mundo
real](#example-1){#example-1-tab .nav-item .nav-link toggle="tab"
role="tab" aria-controls="example-1" aria-selected="true"}
:::

::::: {#nav-tabContent .pattern-example-tab-content .tab-content}
::: {#example-0 .tab-pane .show .active role="tabpanel" aria-labelledby="example-0-tab" style="padding-top: 24px"}
## Ejemplo conceptual {#example-0-title}

Este ejemplo ilustra la estructura del patrón de diseño **Composite** y
se centra en las siguientes preguntas:

-   ¿De qué clases se compone?
-   ¿Qué papeles juegan esas clases?
-   ¿De qué forma se relacionan los elementos del patrón?

Después de conocer la estructura del patrón, será más fácil comprender
el siguiente ejemplo basado en un caso de uso real de PHP.

#### []{#example-0--index-php .anchor} **index.php:** Ejemplo conceptual

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

#### []{#example-0--Output-txt .anchor} **Output.txt:** Resultado de la ejecución

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
## Ejemplo del mundo real {#example-1-title}

El patrón **Composite** puede agilizar el trabajo con cualquier
estructura recursiva con forma de árbol. El árbol DOM HTML es un ejemplo
de dicha estructura. Por ejemplo, mientras varios elementos de entrada
pueden actuar como hojas, los elementos complejos como formas y grupos
de campo (fieldsets) juegan el papel de compuestos.

Con esto en mente, puedes utilizar el patrón Composite para aplicar
varios comportamientos a todo el árbol HTML, del mismo modo que a sus
elementos internos, sin acoplar tu código a clases concretas del árbol
DOM. Ejemplos de estos comportamientos serían representar los elementos
DOM, exportarlos a varios formatos, validar sus partes, etc.

Con el patrón Composite, no tienes que comprobar si se trata del tipo de
elemento simple o complejo antes de ejecutar el comportamiento.
Dependiendo del tipo de elemento, se ejecuta directamente o se pasa a
todos los hijos del elemento.

#### []{#example-1--index-php .anchor} **index.php:** Ejemplo del mundo real

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

#### []{#example-1--Output-txt .anchor} **Output.txt:** Resultado de la ejecución

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
[Ejemplo conceptual](#example-0){#example-0-tab-bottom .nav-item
.nav-link .active toggle="tab" role="tab" aria-controls="example-0"
aria-selected="true"} [Ejemplo del mundo
real](#example-1){#example-1-tab-bottom .nav-item .nav-link toggle="tab"
role="tab" aria-controls="example-1" aria-selected="true"}
:::

::: next
#### Leer siguiente

[Decorator en PHP []{.fa
.fa-arrow-right}](../../decorator/php/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Volver

[[]{.fa .fa-arrow-left} Bridge en
PHP](../../bridge/php/example.html){.btn .btn-default rel="prev"}
:::

## **Composite** en otros lenguajes

[![Composite en
C#](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Composite en C#"){.prog-lang-link}
[![Composite en
C++](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Composite en C++"){.prog-lang-link}
[![Composite en
Go](../../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Composite en Go"){.prog-lang-link}
[![Composite en
Java](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Composite en Java"){.prog-lang-link}
[![Composite en
Python](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Composite en Python"){.prog-lang-link}
[![Composite en
Ruby](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Composite en Ruby"){.prog-lang-link}
[![Composite en
Rust](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Composite en Rust"){.prog-lang-link}
[![Composite en
Swift](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Composite en Swift"){.prog-lang-link}
[![Composite en
TypeScript](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Composite en TypeScript"){.prog-lang-link}
:::::::::::::::::::::::::

::::::: {.prom .banner-sidebar .banner .banner-striped .banner-examples .banner-removable .banner-removable-patterns data-id="DP: Examples-IDE" creative-id="examples-sidebar-es" position="sidebar"}
:::::: {.banner-inner style="font-size: 14px; line-height: 21px;"}
::: banner-examples-img
[![](../../../../images/patterns/banners/examples-ide91ca.png?id=3115b4b548fb96b75974e2de8f4f49bc){width="215"
height="158" loading="lazy"
srcset="/images/patterns/banners/examples-ide-2x.png?id=93c007a6d157b97c28eb213f59d716ae 2x"}](../../book.html)
:::

::: banner-examples-fade
[ Archivo con ejemplos de código](../../book.html){.btn .btn-secondary
.btn-block}
:::

::: {.banner-examples-text style="margin-top: 1rem;"}
Compra el libro electrónico **Sumérgete en los patrones de diseño** para
tener acceso al archivo con decenas de ejemplos detallados que puedes
abrir en tu IDE.

[ Saber más...](../../book.html){.btn .btn-secondary .btn-block}
:::
::::::
:::::::
:::::::::::::::::::::::::::::::
::::::::::::::::::::::::::::::::
