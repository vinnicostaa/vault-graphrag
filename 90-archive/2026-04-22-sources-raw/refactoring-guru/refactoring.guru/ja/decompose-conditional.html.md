:::::::::::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::::::::::::::::::::: {.refactoring .page .text}
::: breadcrumb
[](../index.html){.home} / [Refactoring](refactoring.html){.type} /
[Techniques](refactoring/techniques.html){.type} / [Simplifying
Conditional
Expressions](refactoring/techniques/simplifying-conditional-expressions.html){.type}
:::

# Decompose Conditional {#decompose-conditional .title}

::::: before-after-container
::: before
### Problem

You have a complex conditional (`if-then`/`else` or `switch`).
:::

::: after
### Solution

Decompose the complicated parts of the conditional into separate
methods: the condition, `then` and `else`.
:::
:::::

:::::::::::::::::::::::::::: examples
::::::: {.before-after .before-after-container .java data-lang="java"}
:::: before
::: ba
Before
:::

``` {.code lang="java"}
if (date.before(SUMMER_START) || date.after(SUMMER_END)) {
  charge = quantity * winterRate + winterServiceCharge;
}
else {
  charge = quantity * summerRate;
}
```
::::

:::: after
::: ba
After
:::

``` {.code lang="java"}
if (isSummer(date)) {
  charge = summerCharge(quantity);
}
else {
  charge = winterCharge(quantity);
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
if (date < SUMMER_START || date > SUMMER_END) 
{
  charge = quantity * winterRate + winterServiceCharge;
}
else 
{
  charge = quantity * summerRate;
}
```
::::

:::: after
::: ba
After
:::

``` {.code lang="csharp"}
if (isSummer(date))
{
  charge = SummerCharge(quantity);
}
else 
{
  charge = WinterCharge(quantity);
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
if ($date->before(SUMMER_START) || $date->after(SUMMER_END)) {
  $charge = $quantity * $winterRate + $winterServiceCharge;
} else {
  $charge = $quantity * $summerRate;
}
```
::::

:::: after
::: ba
After
:::

``` {.code lang="php"}
if (isSummer($date)) {
  $charge = summerCharge($quantity);
} else {
  $charge = winterCharge($quantity);
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
if date.before(SUMMER_START) or date.after(SUMMER_END):
    charge = quantity * winterRate + winterServiceCharge
else:
    charge = quantity * summerRate
```
::::

:::: after
::: ba
After
:::

``` {.code lang="python"}
if isSummer(date):
    charge = summerCharge(quantity)
else:
    charge = winterCharge(quantity)
```
::::
:::::::

::::::: {.before-after .before-after-container .typescript data-lang="typescript"}
:::: before
::: ba
Before
:::

``` {.code lang="typescript"}
if (date.before(SUMMER_START) || date.after(SUMMER_END)) {
  charge = quantity * winterRate + winterServiceCharge;
}
else {
  charge = quantity * summerRate;
}
```
::::

:::: after
::: ba
After
:::

``` {.code lang="typescript"}
if (isSummer(date)) {
  charge = summerCharge(quantity);
}
else {
  charge = winterCharge(quantity);
}
```
::::
:::::::
::::::::::::::::::::::::::::

### Why Refactor

The longer a piece of code is, the harder it\'s to understand. Things
become even more hard to understand when the code is filled with
conditions:

-   While you\'re busy figuring out what the code in the `then` block
    does, you forget what the relevant condition was.

-   While you\'re busy parsing `else`, you forget what the code in
    `then` does.

### Benefits

-   By extracting conditional code to clearly named methods, you make
    life easier for the person who\'ll be maintaining the code later
    (such as you, two months from now!).

-   This refactoring technique is also applicable for short expressions
    in conditions. The string `isSalaryDay()` is much prettier and more
    descriptive than code for comparing dates.

### How to Refactor

1.  Extract the conditional to a separate method via [Extract
    Method](extract-method.html).

2.  Repeat the process for the `then` and `else` blocks.

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

:::::: row
::::: {.col-xs-12 .col-sm-6 .col-lg-12}
### Eliminates smell

:::: dl
::: dt
[Long Method](smells/long-method.html)
:::
::::
:::::
::::::

::: next
#### 次を読む

[Consolidate Conditional Expression []{.fa
.fa-arrow-right}](consolidate-conditional-expression.html){.btn
.btn-primary rel="next"}
:::

::: prev
#### 戻る

[[]{.fa .fa-arrow-left} Simplifying Conditional
Expressions](refactoring/techniques/simplifying-conditional-expressions.html){.btn
.btn-default rel="prev"}
:::
::::::::::::::::::::::::::::::::::::::::::

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
:::::::::::::::::::::::::::::::::::::::::::::::
::::::::::::::::::::::::::::::::::::::::::::::::
