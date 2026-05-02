:::::::::::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::::::::::::::::::::: {.refactoring .page .text}
::: breadcrumb
[](../index.html){.home} / [Refactoring](refactoring.html){.type} /
[Techniques](refactoring/techniques.html){.type} / [Simplifying
Conditional
Expressions](refactoring/techniques/simplifying-conditional-expressions.html){.type}
:::

# Consolidate Duplicate Conditional Fragments {#consolidate-duplicate-conditional-fragments .title}

::::: before-after-container
::: before
### Problem

Identical code can be found in all branches of a conditional.
:::

::: after
### Solution

Move the code outside of the conditional.
:::
:::::

:::::::::::::::::::::::::::: examples
::::::: {.before-after .before-after-container .java data-lang="java"}
:::: before
::: ba
Before
:::

``` {.code lang="java"}
if (isSpecialDeal()) {
  total = price * 0.95;
  send();
}
else {
  total = price * 0.98;
  send();
}
```
::::

:::: after
::: ba
After
:::

``` {.code lang="java"}
if (isSpecialDeal()) {
  total = price * 0.95;
}
else {
  total = price * 0.98;
}
send();
```
::::
:::::::

::::::: {.before-after .before-after-container .csharp data-lang="csharp"}
:::: before
::: ba
Before
:::

``` {.code lang="csharp"}
if (IsSpecialDeal()) 
{
  total = price * 0.95;
  Send();
}
else 
{
  total = price * 0.98;
  Send();
}
```
::::

:::: after
::: ba
After
:::

``` {.code lang="csharp"}
if (IsSpecialDeal())
{
  total = price * 0.95;
}
else
{
  total = price * 0.98;
}
Send();
```
::::
:::::::

::::::: {.before-after .before-after-container .php data-lang="php"}
:::: before
::: ba
Before
:::

``` {.code lang="php"}
if (isSpecialDeal()) {
  $total = $price * 0.95;
  send();
} else {
  $total = $price * 0.98;
  send();
}
```
::::

:::: after
::: ba
After
:::

``` {.code lang="php"}
if (isSpecialDeal()) {
  $total = $price * 0.95;
} else {
  $total = $price * 0.98;
}
send();
```
::::
:::::::

::::::: {.before-after .before-after-container .python data-lang="python"}
:::: before
::: ba
Before
:::

``` {.code lang="python"}
if isSpecialDeal():
    total = price * 0.95
    send()
else:
    total = price * 0.98
    send()
```
::::

:::: after
::: ba
After
:::

``` {.code lang="python"}
if isSpecialDeal():
    total = price * 0.95
else:
    total = price * 0.98
send()
```
::::
:::::::

::::::: {.before-after .before-after-container .typescript data-lang="typescript"}
:::: before
::: ba
Before
:::

``` {.code lang="typescript"}
if (isSpecialDeal()) {
  total = price * 0.95;
  send();
}
else {
  total = price * 0.98;
  send();
}
```
::::

:::: after
::: ba
After
:::

``` {.code lang="typescript"}
if (isSpecialDeal()) {
  total = price * 0.95;
}
else {
  total = price * 0.98;
}
send();
```
::::
:::::::
::::::::::::::::::::::::::::

### Why Refactor

Duplicate code is found inside all branches of a conditional, often as
the result of evolution of the code within the conditional branches.
Team development can be a contributing factor to this.

### Benefits

-   Code deduplication.

### How to Refactor

1.  If the duplicated code is at the beginning of the conditional
    branches, move the code to a place before the conditional.

2.  If the code is executed at the end of the branches, place it after
    the conditional.

3.  If the duplicate code is randomly situated inside the branches,
    first try to move the code to the beginning or end of the branch,
    depending on whether it changes the result of the subsequent code.

4.  If appropriate and the duplicate code is longer than one line, try
    using [Extract Method](extract-method.html).

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
[Duplicate Code](smells/duplicate-code.html)
:::
::::
:::::
::::::

::: next
#### Suivant

[Remove Control Flag []{.fa
.fa-arrow-right}](remove-control-flag.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Retour

[[]{.fa .fa-arrow-left} Consolidate Conditional
Expression](consolidate-conditional-expression.html){.btn .btn-default
rel="prev"}
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
