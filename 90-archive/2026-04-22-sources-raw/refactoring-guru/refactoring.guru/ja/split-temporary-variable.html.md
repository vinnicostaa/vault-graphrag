:::::::::::::::::::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::::::::::::::::::::::::::::: {.refactoring .page .text}
::: breadcrumb
[](../index.html){.home} / [Refactoring](refactoring.html){.type} /
[Techniques](refactoring/techniques.html){.type} / [Composing
Methods](refactoring/techniques/composing-methods.html){.type}
:::

# Split Temporary Variable {#split-temporary-variable .title}

::::: before-after-container
::: before
### Problem

You have a local variable that\'s used to store various intermediate
values inside a method (except for cycle variables).
:::

::: after
### Solution

Use different variables for different values. Each variable should be
responsible for only one particular thing.
:::
:::::

:::::::::::::::::::::::::::: examples
::::::: {.before-after .before-after-container .java data-lang="java"}
:::: before
::: ba
Before
:::

``` {.code lang="java"}
double temp = 2 * (height + width);
System.out.println(temp);
temp = height * width;
System.out.println(temp);
```
::::

:::: after
::: ba
After
:::

``` {.code lang="java"}
final double perimeter = 2 * (height + width);
System.out.println(perimeter);
final double area = height * width;
System.out.println(area);
```
::::
:::::::

::::::: {.before-after .before-after-container .csharp data-lang="csharp"}
:::: before
::: ba
Before
:::

``` {.code lang="csharp"}
double temp = 2 * (height + width);
Console.WriteLine(temp);
temp = height * width;
Console.WriteLine(temp);
```
::::

:::: after
::: ba
After
:::

``` {.code lang="csharp"}
readonly double perimeter = 2 * (height + width);
Console.WriteLine(perimeter);
readonly double area = height * width;
Console.WriteLine(area);
```
::::
:::::::

::::::: {.before-after .before-after-container .php data-lang="php"}
:::: before
::: ba
Before
:::

``` {.code lang="php"}
$temp = 2 * ($this->height + $this->width);
echo $temp;
$temp = $this->height * $this->width;
echo $temp;
```
::::

:::: after
::: ba
After
:::

``` {.code lang="php"}
$perimeter = 2 * ($this->height + $this->width);
echo $perimeter;
$area = $this->height * $this->width;
echo $area;
```
::::
:::::::

::::::: {.before-after .before-after-container .python data-lang="python"}
:::: before
::: ba
Before
:::

``` {.code lang="python"}
temp = 2 * (height + width)
print(temp)
temp = height * width
print(temp)
```
::::

:::: after
::: ba
After
:::

``` {.code lang="python"}
perimeter = 2 * (height + width)
print(perimeter)
area = height * width
print(area)
```
::::
:::::::

::::::: {.before-after .before-after-container .typescript data-lang="typescript"}
:::: before
::: ba
Before
:::

``` {.code lang="typescript"}
let temp = 2 * (height + width);
console.log(temp);
temp = height * width;
console.log(temp);
```
::::

:::: after
::: ba
After
:::

``` {.code lang="typescript"}
const perimeter = 2 * (height + width);
console.log(perimeter);
const area = height * width;
console.log(area);
```
::::
:::::::
::::::::::::::::::::::::::::

### Why Refactor

If you\'re skimping on the number of variables inside a function and
reusing them for various unrelated purposes, you\'re sure to encounter
problems as soon as you need to make changes to the code containing the
variables. You will have to recheck each case of variable use to make
sure that the correct values are used.

### Benefits

-   Each component of the program code should be responsible for one and
    one thing only. This makes it much easier to maintain the code,
    since you can easily replace any particular thing without fear of
    unintended effects.

-   Code becomes more readable. If a variable was created long ago in a
    rush, it probably has a name that doesn\'t explain anything: `k`,
    `a2`, `value`, etc. But you can fix this situation by naming the new
    variables in an understandable, self-explanatory way. Such names
    might resemble `customerTaxValue`, `cityUnemploymentRate`,
    `clientSalutationString` and the like.

-   This refactoring technique is useful if you anticipate using
    [Extract Method](extract-method.html) later.

### How to Refactor

1.  Find the first place in the code where the variable is given a
    value. Here you should rename the variable with a name that
    corresponds to the value being assigned.

2.  Use the new name instead of the old one in places where this value
    of the variable is used.

3.  Repeat as needed for places where the variable is assigned a
    different value.

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

:::::::::::::: row
::::: {.col-xs-12 .col-sm-6 .col-lg-4}
### Anti-refactoring

:::: dl
::: dt
[Inline Temp](inline-temp.html)
:::
::::
:::::

::::::: {.col-xs-12 .col-sm-6 .col-lg-4}
### Similar refactorings

:::: dl
::: dt
[Extract Variable](extract-variable.html)
:::
::::

:::: dl
::: dt
[Remove Assignments to
Parameters](remove-assignments-to-parameters.html)
:::
::::
:::::::

::::: {.col-xs-12 .col-sm-6 .col-lg-4}
### Helps other refactorings

:::: dl
::: dt
[Extract Method](extract-method.html)
:::
::::
:::::
::::::::::::::

::: next
#### 次を読む

[Remove Assignments to Parameters []{.fa
.fa-arrow-right}](remove-assignments-to-parameters.html){.btn
.btn-primary rel="next"}
:::

::: prev
#### 戻る

[[]{.fa .fa-arrow-left} Replace Temp with
Query](replace-temp-with-query.html){.btn .btn-default rel="prev"}
:::
::::::::::::::::::::::::::::::::::::::::::::::::::

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
:::::::::::::::::::::::::::::::::::::::::::::::::::::::
::::::::::::::::::::::::::::::::::::::::::::::::::::::::
