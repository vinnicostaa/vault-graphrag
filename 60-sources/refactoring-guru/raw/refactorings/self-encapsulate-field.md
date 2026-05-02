::::::::::::::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::::::::::::::::::::::::: {.refactoring .page .text}
::: breadcrumb
[](index.html){.home} / [Refactoring](refactoring.html){.type} /
[Techniques](refactoring/techniques.html){.type} / [Organizing
Data](refactoring/techniques/organizing-data.html){.type}
:::

# Self Encapsulate Field {#self-encapsulate-field .title}

> Self-encapsulation is distinct from ordinary [Encapsulate
> Field](encapsulate-field.html): the refactoring technique given here
> is performed on a private field.

::::: before-after-container
::: before
### Problem

You use direct access to private fields inside a class.
:::

::: after
### Solution

Create a getter and setter for the field, and use only them for
accessing the field.
:::
:::::

::::::::::::::::::::::: examples
::::::: {.before-after .before-after-container .java data-lang="java"}
:::: before
::: ba
Before
:::

``` {.code lang="java"}
class Range {
  private int low, high;
  boolean includes(int arg) {
    return arg >= low && arg <= high;
  }
}
```
::::

:::: after
::: ba
After
:::

``` {.code lang="java"}
class Range {
  private int low, high;
  boolean includes(int arg) {
    return arg >= getLow() && arg <= getHigh();
  }
  int getLow() {
    return low;
  }
  int getHigh() {
    return high;
  }
}
```
::::
:::::::

::::::: {.before-after .before-after-container .csharp data-lang="csharp"}
:::: before
::: ba
Before
:::

``` {.code lang="csharp"}
class Range 
{
  private int low, high;
  
  bool Includes(int arg) 
  {
    return arg >= low && arg <= high;
  }
}
```
::::

:::: after
::: ba
After
:::

``` {.code lang="csharp"}
class Range 
{
  private int low, high;
  
  int Low {
    get { return low; }
  }
  int High {
    get { return high; }
  }
  
  bool Includes(int arg) 
  {
    return arg >= Low && arg <= High;
  }
}
```
::::
:::::::

::::::: {.before-after .before-after-container .php data-lang="php"}
:::: before
::: ba
Before
:::

``` {.code lang="php"}
private $low;
private $high;

function includes($arg) {
  return $arg >= $this->low && $arg <= $this->high;
}
```
::::

:::: after
::: ba
After
:::

``` {.code lang="php"}
private $low;
private $high;

function includes($arg) {
  return $arg >= $this->getLow() && $arg <= $this->getHigh();
}
function getLow() {
  return $this->low;
}
function getHigh() {
  return $this->high;
}
```
::::
:::::::

::::::: {.before-after .before-after-container .typescript data-lang="typescript"}
:::: before
::: ba
Before
:::

``` {.code lang="typescript"}
class Range {
  private low: number
  private high: number;
  includes(arg: number): boolean {
    return arg >= low && arg <= high;
  }
}
```
::::

:::: after
::: ba
After
:::

``` {.code lang="typescript"}
class Range {
  private low: number
  private high: number;
  includes(arg: number): boolean {
    return arg >= getLow() && arg <= getHigh();
  }
  getLow(): number {
    return low;
  }
  getHigh(): number {
    return high;
  }
}
```
::::
:::::::
:::::::::::::::::::::::

### Why Refactor

Sometimes directly accessing a private field inside a class just isn't
flexible enough. You want to be able to initiate a field value when the
first query is made or perform certain operations on new values of the
field when they're assigned, or maybe do all this in various ways in
subclasses.

### Benefits

-   *Indirect access to fields* is when a field is acted on via access
    methods (getters and setters). This approach is much more flexible
    than *direct access to fields*.

    -   First, you can perform complex operations when data in the field
        is set or received. *Lazy initialization* and *validation of
        field values* are easily implemented inside field getters and
        setters.

    -   Second and more crucially, you can redefine getters and setters
        in subclasses.

-   You have the option of not implementing a setter for a field at all.
    The field value will be specified only in the constructor, thus
    making the field unchangeable throughout the entire object lifespan.

### Drawbacks

-   When *direct access to fields* is used, code looks simpler and more
    presentable, although flexibility is diminished.

### How to Refactor

1.  Create a getter (and optional setter) for the field. They should be
    either `protected` or `public`.

2.  Find all direct invocations of the field and replace them with
    getter and setter calls.

::::: {.prom .banner-content .banner .banner-striped .banner-with-image data-id="Ref: Part of the ebook" creative-id="animated-en" position="content_bottom"}
::: banner-image
Your browser does not support HTML video.
:::

::: banner-text
### Tired of reading? {#tired-of-reading .title}

No wonder, it takes [7 hours]{.blue} to read all of the text we have
here.

Try our interactive course on refactoring. It offers a less tedious
approach to learning new stuff.

[ Let\'s see...](refactoring/course.html){.btn .btn-secondary}
:::
:::::

:::::::::::::: row
:::::: {.col-xs-12 .col-sm-6 .col-lg-6}
### Similar refactorings

::::: dl
::: dt
[Encapsulate Field](encapsulate-field.html)
:::

::: dd
Hide public fields, provide getters and setters.
:::
:::::
::::::

::::::::: {.col-xs-12 .col-sm-6 .col-lg-6}
### Helps other refactorings

:::: dl
::: dt
[Duplicate Observed Data](duplicate-observed-data.html)
:::
::::

:::: dl
::: dt
[Replace Type Code with
Subclasses](replace-type-code-with-subclasses.html)
:::
::::

:::: dl
::: dt
[Replace Type Code with
State/Strategy](replace-type-code-with-state-strategy.html)
:::
::::
:::::::::
::::::::::::::

::: next
#### Read next

[Replace Data Value with Object []{.fa
.fa-arrow-right}](replace-data-value-with-object.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Return

[[]{.fa .fa-arrow-left} Organizing
Data](refactoring/techniques/organizing-data.html){.btn .btn-default
rel="prev"}
:::
:::::::::::::::::::::::::::::::::::::::::::::

:::::: {.prom .banner-sidebar .banner-removable .banner-removable-refactoring data-id="Ref: Sidebar" creative-id="standard-sidebar-en" position="sidebar"}
::::: ban
::: {.image .product-image}
[Sale!]{.banner-discount}
[![](images/refactoring/course/course-cover-en1b63.jpg?id=ac6ad836a27c4a8d579244f15c48a8b4){width="300"
height="300" loading="lazy"}](refactoring/course.html)
:::

::: {.banner-text .banner-text-en}
This refactoring is part of the much bigger **Refactoring Course**.

[ Learn more...](refactoring/course.html){.btn .btn-secondary
.btn-block}
:::
:::::
::::::
::::::::::::::::::::::::::::::::::::::::::::::::::
:::::::::::::::::::::::::::::::::::::::::::::::::::
