:::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../index.html){.home} / [Design
Patterns](../../../design-patterns.html){.type} /
[Prototype](../../prototype.html){.type} / [PHP](../../php.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Prototype](../../../images/patterns/cards/prototype-mini6540.png?id=bc3046bb39ff36574c08d49839fd1c8e){srcset="/images/patterns/cards/prototype-mini-2x.png?id=b871f789a736e7efbb1cd082d2de6398 2x"}
:::

# **Prototype** in PHP {#prototype-in-php .pattern-example-title-block-title}
::::

:::::::::::::: pattern-example-body
::: pattern-example-brief
**Prototype** is a creational design pattern that allows cloning
objects, even complex ones, without coupling to their specific classes.

All prototype classes should have a common interface that makes it
possible to copy objects even if their concrete classes are unknown.
Prototype objects can produce full copies since objects of the same
class can access each other's private fields.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Learn more about Prototype ](../../prototype.html){.btn .btn-lg
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

**Usage examples:** The Prototype pattern is available in PHP [out of
the box](http://php.net/manual/en/language.oop5.cloning.php). You can
use the `clone` keyword to create an exact copy of an object. To add
cloning support to a class, you need to implement a `__clone` method.

**Identification:** The prototype can be easily recognized by a `clone`
or `copy` methods, etc.
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

This example illustrates the structure of the **Prototype** design
pattern and focuses on the following questions:

-   What classes does it consist of?
-   What roles do these classes play?
-   In what way the elements of the pattern are related?

After learning about the pattern's structure it'll be easier for you to
grasp the following example, based on a real-world PHP use case.

#### []{#example-0--index-php .anchor} **index.php:** Conceptual example

<figure class="code">
<pre class="code" lang="php"><code>&lt;?php

namespace RefactoringGuru\Prototype\Conceptual;

/**
 * The example class that has cloning ability. We&#39;ll see how the values of field
 * with different types will be cloned.
 */
class Prototype
{
    public $primitive;
    public $component;
    public $circularReference;

    /**
     * PHP has built-in cloning support. You can `clone` an object without
     * defining any special methods as long as it has fields of primitive types.
     * Fields containing objects retain their references in a cloned object.
     * Therefore, in some cases, you might want to clone those referenced
     * objects as well. You can do this in a special `__clone()` method.
     */
    public function __clone()
    {
        $this-&gt;component = clone $this-&gt;component;

        // Cloning an object that has a nested object with backreference
        // requires special treatment. After the cloning is completed, the
        // nested object should point to the cloned object, instead of the
        // original object.
        $this-&gt;circularReference = clone $this-&gt;circularReference;
        $this-&gt;circularReference-&gt;prototype = $this;
    }
}

class ComponentWithBackReference
{
    public $prototype;

    /**
     * Note that the constructor won&#39;t be executed during cloning. If you have
     * complex logic inside the constructor, you may need to execute it in the
     * `__clone` method as well.
     */
    public function __construct(Prototype $prototype)
    {
        $this-&gt;prototype = $prototype;
    }
}

/**
 * The client code.
 */
function clientCode()
{
    $p1 = new Prototype();
    $p1-&gt;primitive = 245;
    $p1-&gt;component = new \DateTime();
    $p1-&gt;circularReference = new ComponentWithBackReference($p1);

    $p2 = clone $p1;
    if ($p1-&gt;primitive === $p2-&gt;primitive) {
        echo &quot;Primitive field values have been carried over to a clone. Yay!\n&quot;;
    } else {
        echo &quot;Primitive field values have not been copied. Booo!\n&quot;;
    }
    if ($p1-&gt;component === $p2-&gt;component) {
        echo &quot;Simple component has not been cloned. Booo!\n&quot;;
    } else {
        echo &quot;Simple component has been cloned. Yay!\n&quot;;
    }

    if ($p1-&gt;circularReference === $p2-&gt;circularReference) {
        echo &quot;Component with back reference has not been cloned. Booo!\n&quot;;
    } else {
        echo &quot;Component with back reference has been cloned. Yay!\n&quot;;
    }

    if ($p1-&gt;circularReference-&gt;prototype === $p2-&gt;circularReference-&gt;prototype) {
        echo &quot;Component with back reference is linked to original object. Booo!\n&quot;;
    } else {
        echo &quot;Component with back reference is linked to the clone. Yay!\n&quot;;
    }
}

clientCode();</code></pre>
</figure>

#### []{#example-0--Output-txt .anchor} **Output.txt:** Execution result

<figure class="code">
<pre class="code" lang="output"><code>Primitive field values have been carried over to a clone. Yay!
Simple component has been cloned. Yay!
Component with back reference has been cloned. Yay!
Component with back reference is linked to the clone. Yay!</code></pre>
</figure>
:::

::: {#example-1 .tab-pane role="tabpanel" aria-labelledby="example-1-tab" style="padding-top: 24px"}
## Real World Example {#example-1-title}

The **Prototype** pattern provides a convenient way of replicating
existing objects instead of trying to reconstruct the objects by copying
all of their fields directly. The direct approach not only couples you
to the classes of the objects being cloned, but also doesn't allow you
to copy the contents of the private fields. The Prototype pattern lets
you perform the cloning within the context of the cloned class, where
the access to the class' private fields isn't restricted.

This example shows you how to clone a complex Page object using the
Prototype pattern. The Page class has lots of private fields, which will
be carried over to the cloned object thanks to the Prototype pattern.

#### []{#example-1--index-php .anchor} **index.php:** Real world example

<figure class="code">
<pre class="code" lang="php"><code>&lt;?php

namespace RefactoringGuru\Prototype\RealWorld;

/**
 * Prototype.
 */
class Page
{
    private $title;

    private $body;

    /**
     * @var Author
     */
    private $author;

    private $comments = [];

    /**
     * @var \DateTime
     */
    private $date;

    // +100 private fields.

    public function __construct(string $title, string $body, Author $author)
    {
        $this-&gt;title = $title;
        $this-&gt;body = $body;
        $this-&gt;author = $author;
        $this-&gt;author-&gt;addToPage($this);
        $this-&gt;date = new \DateTime();
    }

    public function addComment(string $comment): void
    {
        $this-&gt;comments[] = $comment;
    }

    /**
     * You can control what data you want to carry over to the cloned object.
     *
     * For instance, when a page is cloned:
     * - It gets a new &quot;Copy of ...&quot; title.
     * - The author of the page remains the same. Therefore we leave the
     * reference to the existing object while adding the cloned page to the list
     * of the author&#39;s pages.
     * - We don&#39;t carry over the comments from the old page.
     * - We also attach a new date object to the page.
     */
    public function __clone()
    {
        $this-&gt;title = &quot;Copy of &quot; . $this-&gt;title;
        $this-&gt;author-&gt;addToPage($this);
        $this-&gt;comments = [];
        $this-&gt;date = new \DateTime();
    }
}

class Author
{
    private $name;

    /**
     * @var Page[]
     */
    private $pages = [];

    public function __construct(string $name)
    {
        $this-&gt;name = $name;
    }

    public function addToPage(Page $page): void
    {
        $this-&gt;pages[] = $page;
    }
}

/**
 * The client code.
 */
function clientCode()
{
    $author = new Author(&quot;John Smith&quot;);
    $page = new Page(&quot;Tip of the day&quot;, &quot;Keep calm and carry on.&quot;, $author);

    // ...

    $page-&gt;addComment(&quot;Nice tip, thanks!&quot;);

    // ...

    $draft = clone $page;
    echo &quot;Dump of the clone. Note that the author is now referencing two objects.\n\n&quot;;
    print_r($draft);
}

clientCode();</code></pre>
</figure>

#### []{#example-1--Output-txt .anchor} **Output.txt:** Execution result

<figure class="code">
<pre class="code" lang="output"><code>Dump of the clone. Note that the author is now referencing two objects.

RefactoringGuru\Prototype\RealWorld\Page Object
(
    [title:RefactoringGuru\Prototype\RealWorld\Page:private] =&gt; Copy of Tip of the day
    [body:RefactoringGuru\Prototype\RealWorld\Page:private] =&gt; Keep calm and carry on.
    [author:RefactoringGuru\Prototype\RealWorld\Page:private] =&gt; RefactoringGuru\Prototype\RealWorld\Author Object
        (
            [name:RefactoringGuru\Prototype\RealWorld\Author:private] =&gt; John Smith
            [pages:RefactoringGuru\Prototype\RealWorld\Author:private] =&gt; Array
                (
                    [0] =&gt; RefactoringGuru\Prototype\RealWorld\Page Object
                        (
                            [title:RefactoringGuru\Prototype\RealWorld\Page:private] =&gt; Tip of the day
                            [body:RefactoringGuru\Prototype\RealWorld\Page:private] =&gt; Keep calm and carry on.
                            [author:RefactoringGuru\Prototype\RealWorld\Page:private] =&gt; RefactoringGuru\Prototype\RealWorld\Author Object
 *RECURSION*
                            [comments:RefactoringGuru\Prototype\RealWorld\Page:private] =&gt; Array
                                (
                                    [0] =&gt; Nice tip, thanks!
                                )

                            [date:RefactoringGuru\Prototype\RealWorld\Page:private] =&gt; DateTime Object
                                (
                                    [date] =&gt; 2018-06-04 14:50:39.306237
                                    [timezone_type] =&gt; 3
                                    [timezone] =&gt; UTC
                                )

                        )

                    [1] =&gt; RefactoringGuru\Prototype\RealWorld\Page Object
 *RECURSION*
                )

        )

    [comments:RefactoringGuru\Prototype\RealWorld\Page:private] =&gt; Array
        (
        )

    [date:RefactoringGuru\Prototype\RealWorld\Page:private] =&gt; DateTime Object
        (
            [date] =&gt; 2018-06-04 14:50:39.306272
            [timezone_type] =&gt; 3
            [timezone] =&gt; UTC
        )

)</code></pre>
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

[Singleton in PHP []{.fa
.fa-arrow-right}](../../singleton/php/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Return

[[]{.fa .fa-arrow-left} Factory Method in
PHP](../../factory-method/php/example.html){.btn .btn-default
rel="prev"}
:::

## **Prototype** in Other Languages

[![Prototype in
C#](../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Prototype in C#"){.prog-lang-link}
[![Prototype in
C++](../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Prototype in C++"){.prog-lang-link}
[![Prototype in
Go](../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Prototype in Go"){.prog-lang-link}
[![Prototype in
Java](../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Prototype in Java"){.prog-lang-link}
[![Prototype in
Python](../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Prototype in Python"){.prog-lang-link}
[![Prototype in
Ruby](../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Prototype in Ruby"){.prog-lang-link}
[![Prototype in
Rust](../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Prototype in Rust"){.prog-lang-link}
[![Prototype in
Swift](../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Prototype in Swift"){.prog-lang-link}
[![Prototype in
TypeScript](../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Prototype in TypeScript"){.prog-lang-link}
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
