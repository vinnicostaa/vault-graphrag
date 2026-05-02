:::::::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::::::::::::::::: {.refactoring .page .text}
::: breadcrumb
[](../index.html){.home} / [Refactoring](refactoring.html){.type} /
[Techniques](refactoring/techniques.html){.type} / [Organizing
Data](refactoring/techniques/organizing-data.html){.type}
:::

# Replace Magic Number with Symbolic Constant {#replace-magic-number-with-symbolic-constant .title}

::::: before-after-container
::: before
### Problem

Your code uses a number that has a certain meaning to it.
:::

::: after
### Solution

Replace this number with a constant that has a human-readable name
explaining the meaning of the number.
:::
:::::

:::::::::::::::::::::::::::: examples
::::::: {.before-after .before-after-container .java data-lang="java"}
:::: before
::: ba
Before
:::

``` {.code lang="java"}
double potentialEnergy(double mass, double height) {
  return mass * height * 9.81;
}
```
::::

:::: after
::: ba
After
:::

``` {.code lang="java"}
static final double GRAVITATIONAL_CONSTANT = 9.81;

double potentialEnergy(double mass, double height) {
  return mass * height * GRAVITATIONAL_CONSTANT;
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
double PotentialEnergy(double mass, double height) 
{
  return mass * height * 9.81;
}
```
::::

:::: after
::: ba
After
:::

``` {.code lang="csharp"}
const double GRAVITATIONAL_CONSTANT = 9.81;

double PotentialEnergy(double mass, double height) 
{
  return mass * height * GRAVITATIONAL_CONSTANT;
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
function potentialEnergy($mass, $height) {
  return $mass * $height * 9.81;
}
```
::::

:::: after
::: ba
After
:::

``` {.code lang="php"}
define("GRAVITATIONAL_CONSTANT", 9.81);

function potentialEnergy($mass, $height) {
  return $mass * $height * GRAVITATIONAL_CONSTANT;
}
```
::::
:::::::

::::::: {.before-after .before-after-container .python data-lang="python"}
:::: before
::: ba
Before
:::

``` {.code lang="python"}
def potentialEnergy(mass, height):
    return mass * height * 9.81
```
::::

:::: after
::: ba
After
:::

``` {.code lang="python"}
GRAVITATIONAL_CONSTANT = 9.81

def potentialEnergy(mass, height):
    return mass * height * GRAVITATIONAL_CONSTANT
```
::::
:::::::

::::::: {.before-after .before-after-container .typescript data-lang="typescript"}
:::: before
::: ba
Before
:::

``` {.code lang="typescript"}
potentialEnergy(mass: number, height: number): number {
  return mass * height * 9.81;
}
```
::::

:::: after
::: ba
After
:::

``` {.code lang="typescript"}
static const GRAVITATIONAL_CONSTANT = 9.81;

potentialEnergy(mass: number, height: number): number {
  return mass * height * GRAVITATIONAL_CONSTANT;
}
```
::::
:::::::
::::::::::::::::::::::::::::

### Why Refactor

A magic number is a numeric value that's encountered in the source but
has no obvious meaning. This "anti-pattern" makes it harder to
understand the program and refactor the code.

Yet more difficulties arise when you need to change this magic number.
Find and replace won't work for this: the same number may be used for
different purposes in different places, meaning that you will have to
verify every line of code that uses this number.

### Benefits

-   The symbolic constant can serve as live documentation of the meaning
    of its value.

-   It's much easier to change the value of a constant than to search
    for this number throughout the entire codebase, without the risk of
    accidentally changing the same number used elsewhere for a different
    purpose.

-   Reduce duplicate use of a number or string in the code. This is
    especially important when the value is complicated and long (such as
    `3.14159` or `0xCAFEBABE`).

### Good to Know

#### Not all numbers are magical.

If the purpose of a number is obvious, there's no need to replace it. A
classic example is:

<figure class="code">
<pre class="code"><code>for (i = 0; i &lt; сount; i++) { ... }</code></pre>
</figure>

#### Alternatives

1.  Sometimes a magic number can be replaced with method calls. For
    example, if you have a magic number that signifies the number of
    elements in a collection, you don't need to use it for checking the
    last element of the collection. Instead, use the standard method for
    getting the collection length.

2.  Magic numbers are sometimes used as type code. Say that you have two
    types of users and you use a number field in a class to specify
    which is which: administrators are `1` and ordinary users are `2`.

    In this case, you should use one of the refactoring methods to avoid
    type code:

    -   [Replace Type Code with
        Class](replace-type-code-with-class.html)

    -   [Replace Type Code with
        Subclasses](replace-type-code-with-subclasses.html)

    -   [Replace Type Code with
        State/Strategy](replace-type-code-with-state-strategy.html)

### How to Refactor

1.  Declare a constant and assign the value of the magic number to it.

2.  Find all mentions of the magic number.

3.  For each of the numbers that you find, double-check that the magic
    number in this particular case corresponds to the purpose of the
    constant. If yes, replace the number with your constant. This is an
    important step, since the same number can mean absolutely different
    things (and replaced with different constants, as the case may be).

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

[ Let\'s see...](../refactoring/course.html){.btn .btn-secondary}
:::
:::::

::: next
#### Leia a seguir

[Encapsulate Field []{.fa .fa-arrow-right}](encapsulate-field.html){.btn
.btn-primary rel="next"}
:::

::: prev
#### Voltar

[[]{.fa .fa-arrow-left} Change Bidirectional Association to
Unidirectional](change-bidirectional-association-to-unidirectional.html){.btn
.btn-default rel="prev"}
:::
::::::::::::::::::::::::::::::::::::::

:::::: {.prom .banner-sidebar .banner-removable .banner-removable-refactoring data-id="Ref: Sidebar" creative-id="standard-sidebar-en" position="sidebar"}
::::: ban
::: {.image .product-image}
[Sale!]{.banner-discount}
[![](../images/refactoring/course/course-cover-en1b63.jpg?id=ac6ad836a27c4a8d579244f15c48a8b4){width="300"
height="300" loading="lazy"}](../refactoring/course.html)
:::

::: {.banner-text .banner-text-en}
This refactoring is part of the much bigger **Refactoring Course**.

[ Learn more...](../refactoring/course.html){.btn .btn-secondary
.btn-block}
:::
:::::
::::::
:::::::::::::::::::::::::::::::::::::::::::
::::::::::::::::::::::::::::::::::::::::::::
