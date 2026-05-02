:::::::::::::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::::::::::::::::::::::: {.refactoring .page .text}
::: breadcrumb
[](../index.html){.home} / [Refactoring](refactoring.html){.type} /
[Techniques](refactoring/techniques.html){.type} / [Composing
Methods](refactoring/techniques/composing-methods.html){.type}
:::

# Inline Temp {#inline-temp .title}

::::: before-after-container
::: before
### Problem

You have a temporary variable that's assigned the result of a simple
expression and nothing more.
:::

::: after
### Solution

Replace the references to the variable with the expression itself.
:::
:::::

:::::::::::::::::::::::::::: examples
::::::: {.before-after .before-after-container .java data-lang="java"}
:::: before
::: ba
Before
:::

``` {.code lang="java"}
boolean hasDiscount(Order order) {
  double basePrice = order.basePrice();
  return basePrice > 1000;
}
```
::::

:::: after
::: ba
After
:::

``` {.code lang="java"}
boolean hasDiscount(Order order) {
  return order.basePrice() > 1000;
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
bool HasDiscount(Order order)
{
  double basePrice = order.BasePrice();
  return basePrice > 1000;
}
```
::::

:::: after
::: ba
After
:::

``` {.code lang="csharp"}
bool HasDiscount(Order order)
{
  return order.BasePrice() > 1000;
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
$basePrice = $anOrder->basePrice();
return $basePrice > 1000;
```
::::

:::: after
::: ba
After
:::

``` {.code lang="php"}
return $anOrder->basePrice() > 1000;
```
::::
:::::::

::::::: {.before-after .before-after-container .python data-lang="python"}
:::: before
::: ba
Before
:::

``` {.code lang="python"}
def hasDiscount(order):
    basePrice = order.basePrice()
    return basePrice > 1000
```
::::

:::: after
::: ba
After
:::

``` {.code lang="python"}
def hasDiscount(order):
    return order.basePrice() > 1000
```
::::
:::::::

::::::: {.before-after .before-after-container .typescript data-lang="typescript"}
:::: before
::: ba
Before
:::

``` {.code lang="typescript"}
hasDiscount(order: Order): boolean {
  let basePrice: number = order.basePrice();
  return basePrice > 1000;
}
```
::::

:::: after
::: ba
After
:::

``` {.code lang="typescript"}
hasDiscount(order: Order): boolean {
  return order.basePrice() > 1000;
}
```
::::
:::::::
::::::::::::::::::::::::::::

### Why Refactor

Inline local variables are almost always used as part of [Replace Temp
with Query](replace-temp-with-query.html) or to pave the way for
[Extract Method](extract-method.html).

### Benefits

-   This refactoring technique offers almost no benefit in and of
    itself. However, if the variable is assigned the result of a method,
    you can marginally improve the readability of the program by getting
    rid of the unnecessary variable.

### Drawbacks

-   Sometimes seemingly useless temps are used to cache the result of an
    expensive operation that's reused several times. So before using
    this refactoring technique, make sure that simplicity won't come at
    the cost of performance.

### How to Refactor

1.  Find all places that use the variable. Instead of the variable, use
    the expression that had been assigned to it.

2.  Delete the declaration of the variable and its assignment line.

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

:::::::: row
::::::: {.col-xs-12 .col-sm-6 .col-lg-12}
### Helps other refactorings

:::: dl
::: dt
[Replace Temp with Query](replace-temp-with-query.html)
:::
::::

:::: dl
::: dt
[Extract Method](extract-method.html)
:::
::::
:::::::
::::::::

::: next
#### Leer siguiente

[Replace Temp with Query []{.fa
.fa-arrow-right}](replace-temp-with-query.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Volver

[[]{.fa .fa-arrow-left} Extract Variable](extract-variable.html){.btn
.btn-default rel="prev"}
:::
::::::::::::::::::::::::::::::::::::::::::::

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
:::::::::::::::::::::::::::::::::::::::::::::::::
::::::::::::::::::::::::::::::::::::::::::::::::::
