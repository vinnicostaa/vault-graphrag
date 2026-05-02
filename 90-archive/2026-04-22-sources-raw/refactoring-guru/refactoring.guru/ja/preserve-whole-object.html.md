::::::::::::::::::::::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::::::::::::::::::::::::::::::::: {.refactoring .page .text}
::: breadcrumb
[](../index.html){.home} / [Refactoring](refactoring.html){.type} /
[Techniques](refactoring/techniques.html){.type} / [Simplifying Method
Calls](refactoring/techniques/simplifying-method-calls.html){.type}
:::

# Preserve Whole Object {#preserve-whole-object .title}

::::: before-after-container
::: before
### Problem

You get several values from an object and then pass them as parameters
to a method.
:::

::: after
### Solution

Instead, try passing the whole object.
:::
:::::

:::::::::::::::::::::::::::: examples
::::::: {.before-after .before-after-container .java data-lang="java"}
:::: before
::: ba
Before
:::

``` {.code lang="java"}
int low = daysTempRange.getLow();
int high = daysTempRange.getHigh();
boolean withinPlan = plan.withinRange(low, high);
```
::::

:::: after
::: ba
After
:::

``` {.code lang="java"}
boolean withinPlan = plan.withinRange(daysTempRange);
```
::::
:::::::

::::::: {.before-after .before-after-container .csharp data-lang="csharp"}
:::: before
::: ba
Before
:::

``` {.code lang="csharp"}
int low = daysTempRange.GetLow();
int high = daysTempRange.GetHigh();
bool withinPlan = plan.WithinRange(low, high);
```
::::

:::: after
::: ba
After
:::

``` {.code lang="csharp"}
bool withinPlan = plan.WithinRange(daysTempRange);
```
::::
:::::::

::::::: {.before-after .before-after-container .php data-lang="php"}
:::: before
::: ba
Before
:::

``` {.code lang="php"}
$low = $daysTempRange->getLow();
$high = $daysTempRange->getHigh();
$withinPlan = $plan->withinRange($low, $high);
```
::::

:::: after
::: ba
After
:::

``` {.code lang="php"}
$withinPlan = $plan->withinRange($daysTempRange);
```
::::
:::::::

::::::: {.before-after .before-after-container .python data-lang="python"}
:::: before
::: ba
Before
:::

``` {.code lang="python"}
low = daysTempRange.getLow()
high = daysTempRange.getHigh()
withinPlan = plan.withinRange(low, high)
```
::::

:::: after
::: ba
After
:::

``` {.code lang="python"}
withinPlan = plan.withinRange(daysTempRange)
```
::::
:::::::

::::::: {.before-after .before-after-container .typescript data-lang="typescript"}
:::: before
::: ba
Before
:::

``` {.code lang="typescript"}
let low = daysTempRange.getLow();
let high = daysTempRange.getHigh();
let withinPlan = plan.withinRange(low, high);
```
::::

:::: after
::: ba
After
:::

``` {.code lang="typescript"}
let withinPlan = plan.withinRange(daysTempRange);
```
::::
:::::::
::::::::::::::::::::::::::::

### Why Refactor

The problem is that each time before your method is called, the methods
of the future parameter object must be called. If these methods or the
quantity of data obtained for the method are changed, you will need to
carefully find a dozen such places in the program and implement these
changes in each of them.

After you apply this refactoring technique, the code for getting all
necessary data will be stored in one place---the method itself.

### Benefits

-   Instead of a hodgepodge of parameters, you see a single object with
    a comprehensible name.

-   If the method needs more data from an object, you won\'t need to
    rewrite all the places where the method is used---merely inside the
    method itself.

### Drawbacks

-   Sometimes this transformation causes a method to become less
    flexible: previously the method could get data from many different
    sources but now, because of refactoring, we\'re limiting its use to
    only objects with a particular interface.

### How to Refactor

1.  Create a parameter in the method for the object from which you can
    get the necessary values.

2.  Now start removing the old parameters from the method one by one,
    replacing them with calls to the relevant methods of the parameter
    object. Test the program after each replacement of a parameter.

3.  Delete the getter code from the parameter object that had preceded
    the method call.

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

::::::::::::::::: row
::::::: {.col-xs-12 .col-sm-6 .col-lg-6}
### Similar refactorings

:::: dl
::: dt
[Introduce Parameter Object](introduce-parameter-object.html)
:::
::::

:::: dl
::: dt
[Replace Parameter with Method
Call](replace-parameter-with-method-call.html)
:::
::::
:::::::

::::::::::: {.col-xs-12 .col-sm-6 .col-lg-6}
### Eliminates smell

:::: dl
::: dt
[Primitive Obsession](smells/primitive-obsession.html)
:::
::::

:::: dl
::: dt
[Long Parameter List](smells/long-parameter-list.html)
:::
::::

:::: dl
::: dt
[Long Method](smells/long-method.html)
:::
::::

:::: dl
::: dt
[Data Clumps](smells/data-clumps.html)
:::
::::
:::::::::::
:::::::::::::::::

::: next
#### 次を読む

[Replace Parameter with Method Call []{.fa
.fa-arrow-right}](replace-parameter-with-method-call.html){.btn
.btn-primary rel="next"}
:::

::: prev
#### 戻る

[[]{.fa .fa-arrow-left} Replace Parameter with Explicit
Methods](replace-parameter-with-explicit-methods.html){.btn .btn-default
rel="prev"}
:::
:::::::::::::::::::::::::::::::::::::::::::::::::::::

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
::::::::::::::::::::::::::::::::::::::::::::::::::::::::::
:::::::::::::::::::::::::::::::::::::::::::::::::::::::::::
