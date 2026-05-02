:::::::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::::::::::::::::: {.refactoring .page .text}
::: breadcrumb
[](../index.html){.home} / [Refactoring](refactoring.html){.type} /
[Techniques](refactoring/techniques.html){.type} / [Simplifying Method
Calls](refactoring/techniques/simplifying-method-calls.html){.type}
:::

# Replace Parameter with Method Call {#replace-parameter-with-method-call .title}

::::: before-after-container
::: before
### Problem

Calling a query method and passing its results as the parameters of
another method, while that method could call the query directly.
:::

::: after
### Solution

Instead of passing the value through a parameter, try placing a query
call inside the method body.
:::
:::::

:::::::::::::::::::::::::::: examples
::::::: {.before-after .before-after-container .java data-lang="java"}
:::: before
::: ba
Before
:::

``` {.code lang="java"}
int basePrice = quantity * itemPrice;
double seasonDiscount = this.getSeasonalDiscount();
double fees = this.getFees();
double finalPrice = discountedPrice(basePrice, seasonDiscount, fees);
```
::::

:::: after
::: ba
After
:::

``` {.code lang="java"}
int basePrice = quantity * itemPrice;
double finalPrice = discountedPrice(basePrice);
```
::::
:::::::

::::::: {.before-after .before-after-container .csharp data-lang="csharp"}
:::: before
::: ba
Before
:::

``` {.code lang="csharp"}
int basePrice = quantity * itemPrice;
double seasonDiscount = this.GetSeasonalDiscount();
double fees = this.GetFees();
double finalPrice = DiscountedPrice(basePrice, seasonDiscount, fees);
```
::::

:::: after
::: ba
After
:::

``` {.code lang="csharp"}
int basePrice = quantity * itemPrice;
double finalPrice = DiscountedPrice(basePrice);
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
$seasonDiscount = $this->getSeasonalDiscount();
$fees = $this->getFees();
$finalPrice = $this->discountedPrice($basePrice, $seasonDiscount, $fees);
```
::::

:::: after
::: ba
After
:::

``` {.code lang="php"}
$basePrice = $this->quantity * $this->itemPrice;
$finalPrice = $this->discountedPrice($basePrice);
```
::::
:::::::

::::::: {.before-after .before-after-container .python data-lang="python"}
:::: before
::: ba
Before
:::

``` {.code lang="python"}
basePrice = quantity * itemPrice
seasonalDiscount = self.getSeasonalDiscount()
fees = self.getFees()
finalPrice = discountedPrice(basePrice, seasonalDiscount, fees)
```
::::

:::: after
::: ba
After
:::

``` {.code lang="python"}
basePrice = quantity * itemPrice
finalPrice = discountedPrice(basePrice)
```
::::
:::::::

::::::: {.before-after .before-after-container .typescript data-lang="typescript"}
:::: before
::: ba
Before
:::

``` {.code lang="typescript"}
let basePrice = quantity * itemPrice;
const seasonDiscount = this.getSeasonalDiscount();
const fees = this.getFees();
const finalPrice = discountedPrice(basePrice, seasonDiscount, fees);
```
::::

:::: after
::: ba
After
:::

``` {.code lang="typescript"}
let basePrice = quantity * itemPrice;
let finalPrice = discountedPrice(basePrice);
```
::::
:::::::
::::::::::::::::::::::::::::

### Why Refactor

A long list of parameters is hard to understand. In addition, calls to
such methods often resemble a series of cascades, with winding and
exhilarating value calculations that are hard to navigate yet have to be
passed to the method. So if a parameter value can be calculated with the
help of a method, do this inside the method itself and get rid of the
parameter.

### Benefits

-   We get rid of unneeded parameters and simplify method calls. Such
    parameters are often created not for the project as it\'s now, but
    with an eye for future needs that may never come.

### Drawbacks

-   You may need the parameter tomorrow for other needs\... making you
    rewrite the method.

### How to Refactor

1.  Make sure that the value-getting code doesn\'t use parameters from
    the current method, since they\'ll be unavailable from inside
    another method. If so, moving the code isn\'t possible.

2.  If the relevant code is more complicated than a single method or
    function call, use [Extract Method](extract-method.html) to isolate
    this code in a new method and make the call simple.

3.  In the code of the main method, replace all references to the
    parameter being replaced with calls to the method that gets the
    value.

4.  Use [Remove Parameter](remove-parameter.html) to eliminate the
    now-unused parameter.

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
#### 次を読む

[Introduce Parameter Object []{.fa
.fa-arrow-right}](introduce-parameter-object.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### 戻る

[[]{.fa .fa-arrow-left} Preserve Whole
Object](preserve-whole-object.html){.btn .btn-default rel="prev"}
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
