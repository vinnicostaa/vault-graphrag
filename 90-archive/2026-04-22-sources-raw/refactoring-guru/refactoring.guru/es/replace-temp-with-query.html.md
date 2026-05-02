::::::::::::::::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::::::::::::::::::::::::::: {.refactoring .page .text}
::: breadcrumb
[](../index.html){.home} / [Refactoring](refactoring.html){.type} /
[Techniques](refactoring/techniques.html){.type} / [Composing
Methods](refactoring/techniques/composing-methods.html){.type}
:::

# Replace Temp with Query {#replace-temp-with-query .title}

::::: before-after-container
::: before
### Problem

You place the result of an expression in a local variable for later use
in your code.
:::

::: after
### Solution

Move the entire expression to a separate method and return the result
from it. Query the method instead of using a variable. Incorporate the
new method in other methods, if necessary.
:::
:::::

:::::::::::::::::::::::::::: examples
::::::: {.before-after .before-after-container .java data-lang="java"}
:::: before
::: ba
Before
:::

``` {.code lang="java"}
double calculateTotal() {
  double basePrice = quantity * itemPrice;
  if (basePrice > 1000) {
    return basePrice * 0.95;
  }
  else {
    return basePrice * 0.98;
  }
}
```
::::

:::: after
::: ba
After
:::

``` {.code lang="java"}
double calculateTotal() {
  if (basePrice() > 1000) {
    return basePrice() * 0.95;
  }
  else {
    return basePrice() * 0.98;
  }
}
double basePrice() {
  return quantity * itemPrice;
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
double CalculateTotal() 
{
  double basePrice = quantity * itemPrice;
  
  if (basePrice > 1000) 
  {
    return basePrice * 0.95;
  }
  else 
  {
    return basePrice * 0.98;
  }
}
```
::::

:::: after
::: ba
After
:::

``` {.code lang="csharp"}
double CalculateTotal() 
{
  if (BasePrice() > 1000) 
  {
    return BasePrice() * 0.95;
  }
  else 
  {
    return BasePrice() * 0.98;
  }
}
double BasePrice() 
{
  return quantity * itemPrice;
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
$basePrice = $this->quantity * $this->itemPrice;
if ($basePrice > 1000) {
  return $basePrice * 0.95;
} else {
  return $basePrice * 0.98;
}
```
::::

:::: after
::: ba
After
:::

``` {.code lang="php"}
if ($this->basePrice() > 1000) {
  return $this->basePrice() * 0.95;
} else {
  return $this->basePrice() * 0.98;
}

...

function basePrice() {
  return $this->quantity * $this->itemPrice;
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
def calculateTotal():
    basePrice = quantity * itemPrice
    if basePrice > 1000:
        return basePrice * 0.95
    else:
        return basePrice * 0.98
```
::::

:::: after
::: ba
After
:::

``` {.code lang="python"}
def calculateTotal():
    if basePrice() > 1000:
        return basePrice() * 0.95
    else:
        return basePrice() * 0.98

def basePrice():
    return quantity * itemPrice
```
::::
:::::::

::::::: {.before-after .before-after-container .typescript data-lang="typescript"}
:::: before
::: ba
Before
:::

``` {.code lang="typescript"}
 calculateTotal(): number {
  let basePrice = quantity * itemPrice;
  if (basePrice > 1000) {
    return basePrice * 0.95;
  }
  else {
    return basePrice * 0.98;
  }
}
```
::::

:::: after
::: ba
After
:::

``` {.code lang="typescript"}
calculateTotal(): number {
  if (basePrice() > 1000) {
    return basePrice() * 0.95;
  }
  else {
    return basePrice() * 0.98;
  }
}
basePrice(): number {
  return quantity * itemPrice;
}
```
::::
:::::::
::::::::::::::::::::::::::::

### Why Refactor

This refactoring can lay the groundwork for applying [Extract
Method](extract-method.html) for a portion of a very long method.

The same expression may sometimes be found in other methods as well,
which is one reason to consider creating a common method.

### Benefits

-   Code readability. It's much easier to understand the purpose of the
    method `getTax()` than the line `orderPrice() * 0.2`.

-   Slimmer code via deduplication, if the line being replaced is used
    in multiple methods.

### Good to Know

#### Performance

This refactoring may prompt the question of whether this approach is
liable to cause a performance hit. The honest answer is: yes, it is,
since the resulting code may be burdened by querying a new method. But
with today's fast CPUs and excellent compilers, the burden will almost
always be minimal. By contrast, readable code and the ability to reuse
this method in other places in program code---thanks to this refactoring
approach---are very noticeable benefits.

Nonetheless, if your temp variable is used to cache the result of a
truly time-consuming expression, you may want to stop this refactoring
after extracting the expression to a new method.

### How to Refactor

1.  Make sure that a value is assigned to the variable once and only
    once within the method. If not, use [Split Temporary
    Variable](split-temporary-variable.html) to ensure that the variable
    will be used only to store the result of your expression.

2.  Use [Extract Method](extract-method.html) to place the expression of
    interest in a new method. Make sure that this method only returns a
    value and doesn't change the state of the object. If the method
    affects the visible state of the object, use [Separate Query from
    Modifier](separate-query-from-modifier.html).

3.  Replace the variable with a query to your new method.

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

::::::::::: row
::::: {.col-xs-12 .col-sm-6 .col-lg-6}
### Similar refactorings

:::: dl
::: dt
[Extract Method](extract-method.html)
:::
::::
:::::

::::::: {.col-xs-12 .col-sm-6 .col-lg-6}
### Eliminates smell

:::: dl
::: dt
[Long Method](smells/long-method.html)
:::
::::

:::: dl
::: dt
[Duplicate Code](smells/duplicate-code.html)
:::
::::
:::::::
:::::::::::

::: next
#### Leer siguiente

[Split Temporary Variable []{.fa
.fa-arrow-right}](split-temporary-variable.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Volver

[[]{.fa .fa-arrow-left} Inline Temp](inline-temp.html){.btn .btn-default
rel="prev"}
:::
:::::::::::::::::::::::::::::::::::::::::::::::

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
::::::::::::::::::::::::::::::::::::::::::::::::::::
:::::::::::::::::::::::::::::::::::::::::::::::::::::
