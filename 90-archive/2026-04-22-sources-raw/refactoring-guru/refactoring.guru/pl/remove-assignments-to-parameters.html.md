::::::::::::::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::::::::::::::::::::::::: {.refactoring .page .text}
::: breadcrumb
[](../index.html){.home} / [Refactoring](refactoring.html){.type} /
[Techniques](refactoring/techniques.html){.type} / [Composing
Methods](refactoring/techniques/composing-methods.html){.type}
:::

# Remove Assignments to Parameters {#remove-assignments-to-parameters .title}

::::: before-after-container
::: before
### Problem

Some value is assigned to a parameter inside method's body.
:::

::: after
### Solution

Use a local variable instead of a parameter.
:::
:::::

:::::::::::::::::::::::::::: examples
::::::: {.before-after .before-after-container .java data-lang="java"}
:::: before
::: ba
Before
:::

``` {.code lang="java"}
int discount(int inputVal, int quantity) {
  if (quantity > 50) {
    inputVal -= 2;
  }
  // ...
}
```
::::

:::: after
::: ba
After
:::

``` {.code lang="java"}
int discount(int inputVal, int quantity) {
  int result = inputVal;
  if (quantity > 50) {
    result -= 2;
  }
  // ...
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
int Discount(int inputVal, int quantity) 
{
  if (quantity > 50) 
  {
    inputVal -= 2;
  }
  // ...
}
```
::::

:::: after
::: ba
After
:::

``` {.code lang="csharp"}
int Discount(int inputVal, int quantity) 
{
  int result = inputVal;
  
  if (quantity > 50) 
  {
    result -= 2;
  }
  // ...
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
function discount($inputVal, $quantity) {
  if ($quantity > 50) {
    $inputVal -= 2;
  }
  ...
```
::::

:::: after
::: ba
After
:::

``` {.code lang="php"}
function discount($inputVal, $quantity) {
  $result = $inputVal;
  if ($quantity > 50) {
    $result -= 2;
  }
  ...
```
::::
:::::::

::::::: {.before-after .before-after-container .python data-lang="python"}
:::: before
::: ba
Before
:::

``` {.code lang="python"}
def discount(inputVal, quantity):
    if quantity > 50:
        inputVal -= 2
    # ...
```
::::

:::: after
::: ba
After
:::

``` {.code lang="python"}
def discount(inputVal, quantity):
    result = inputVal
    if quantity > 50:
        result -= 2
    # ...
```
::::
:::::::

::::::: {.before-after .before-after-container .typescript data-lang="typescript"}
:::: before
::: ba
Before
:::

``` {.code lang="typescript"}
discount(inputVal: number, quantity: number): number {
  if (quantity > 50) {
    inputVal -= 2;
  }
  // ...
}
```
::::

:::: after
::: ba
After
:::

``` {.code lang="typescript"}
discount(inputVal: number, quantity: number): number {
  let result = inputVal;
  if (quantity > 50) {
    result -= 2;
  }
  // ...
}
```
::::
:::::::
::::::::::::::::::::::::::::

### Why Refactor

The reasons for this refactoring are the same as for [Split Temporary
Variable](split-temporary-variable.html), but in this case we're dealing
with a parameter, not a local variable.

First, if a parameter is passed via reference, then after the parameter
value is changed inside the method, this value is passed to the argument
that requested calling this method. Very often, this occurs accidentally
and leads to unfortunate effects. Even if parameters are usually
passed by value (and not by reference) in your programming language,
this coding quirk may alienate those who are unaccustomed to it.

Second, multiple assignments of different values to a single parameter
make it difficult for you to know what data should be contained in the
parameter at any particular point in time. The problem worsens if your
parameter and its contents are documented but the actual value is
capable of differing from what's expected inside the method.

### Benefits

-   Each element of the program should be responsible for only one
    thing. This makes code maintenance much easier going forward, since
    you can safely replace code without any side effects.

-   This refactoring helps to extract [repetitive code to separate
    methods](extract-method.html).

### How to Refactor

1.  Create a local variable and assign the initial value of your
    parameter.

2.  In all method code that follows this line, replace the parameter
    with your new local variable.

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

::::::::: row
::::: {.col-xs-12 .col-sm-6 .col-lg-6}
### Similar refactorings

:::: dl
::: dt
[Split Temporary Variable](split-temporary-variable.html)
:::
::::
:::::

::::: {.col-xs-12 .col-sm-6 .col-lg-6}
### Helps other refactorings

:::: dl
::: dt
[Extract Method](extract-method.html)
:::
::::
:::::
:::::::::

::: next
#### Czytaj dalej

[Replace Method with Method Object []{.fa
.fa-arrow-right}](replace-method-with-method-object.html){.btn
.btn-primary rel="next"}
:::

::: prev
#### Wróć

[[]{.fa .fa-arrow-left} Split Temporary
Variable](split-temporary-variable.html){.btn .btn-default rel="prev"}
:::
:::::::::::::::::::::::::::::::::::::::::::::

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
::::::::::::::::::::::::::::::::::::::::::::::::::
:::::::::::::::::::::::::::::::::::::::::::::::::::
