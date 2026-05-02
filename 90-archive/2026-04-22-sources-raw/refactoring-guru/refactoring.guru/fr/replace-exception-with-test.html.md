:::::::::::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::::::::::::::::::::: {.refactoring .page .text}
::: breadcrumb
[](../index.html){.home} / [Refactoring](refactoring.html){.type} /
[Techniques](refactoring/techniques.html){.type} / [Simplifying Method
Calls](refactoring/techniques/simplifying-method-calls.html){.type}
:::

# Replace Exception with Test {#replace-exception-with-test .title}

::::: before-after-container
::: before
### Problem

You throw an exception in a place where a simple test would do the job?
:::

::: after
### Solution

Replace the exception with a condition test.
:::
:::::

:::::::::::::::::::::::::::: examples
::::::: {.before-after .before-after-container .java data-lang="java"}
:::: before
::: ba
Before
:::

``` {.code lang="java"}
double getValueForPeriod(int periodNumber) {
  try {
    return values[periodNumber];
  } catch (ArrayIndexOutOfBoundsException e) {
    return 0;
  }
}
```
::::

:::: after
::: ba
After
:::

``` {.code lang="java"}
double getValueForPeriod(int periodNumber) {
  if (periodNumber >= values.length) {
    return 0;
  }
  return values[periodNumber];
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
double GetValueForPeriod(int periodNumber) 
{
  try 
  {
    return values[periodNumber];
  } 
  catch (IndexOutOfRangeException e) 
  {
    return 0;
  }
}
```
::::

:::: after
::: ba
After
:::

``` {.code lang="csharp"}
double GetValueForPeriod(int periodNumber) 
{
  if (periodNumber >= values.Length) 
  {
    return 0;
  }
  return values[periodNumber];
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
function getValueForPeriod($periodNumber) {
  try {
    return $this->values[$periodNumber];
  } catch (ArrayIndexOutOfBoundsException $e) {
    return 0;
  }
}
```
::::

:::: after
::: ba
After
:::

``` {.code lang="php"}
function getValueForPeriod($periodNumber) {
  if ($periodNumber >= count($this->values)) {
    return 0;
  }
  return $this->values[$periodNumber];
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
def getValueForPeriod(periodNumber):
    try:
        return values[periodNumber]
    except IndexError:
        return 0
```
::::

:::: after
::: ba
After
:::

``` {.code lang="python"}
def getValueForPeriod(self, periodNumber):
    if periodNumber >= len(self.values):
        return 0
    return self.values[periodNumber]
```
::::
:::::::

::::::: {.before-after .before-after-container .typescript data-lang="typescript"}
:::: before
::: ba
Before
:::

``` {.code lang="typescript"}
getValueForPeriod(periodNumber: number): number {
  try {
    return values[periodNumber];
  } catch (ArrayIndexOutOfBoundsException e) {
    return 0;
  }
}
```
::::

:::: after
::: ba
After
:::

``` {.code lang="typescript"}
getValueForPeriod(periodNumber: number): number {
  if (periodNumber >= values.length) {
    return 0;
  }
  return values[periodNumber];
}
```
::::
:::::::
::::::::::::::::::::::::::::

### Why Refactor

Exceptions should be used to handle irregular behavior related to an
unexpected error. They shouldn't serve as a replacement for testing. If
an exception can be avoided by simply verifying a condition before
running, then do so. Exceptions should be reserved for real errors.

For instance, you entered a minefield and triggered a mine there,
resulting in an exception; the exception was successfully handled and
you were lifted through the air to safety beyond the mine field. But you
could have avoided this all by simply reading the warning sign in front
of the minefield to begin with.

### Benefits

-   A simple conditional can sometimes be more obvious than exception
    handling code.

### How to Refactor

1.  Create a conditional for an edge case and move it before the
    try/catch block.

2.  Move code from the `catch` section inside this conditional.

3.  In the `catch` section, place the code for throwing a usual unnamed
    exception and run all the tests.

4.  If no exceptions were thrown during the tests, get rid of the
    `try`/`catch` operator.

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
### Similar refactorings

:::: dl
::: dt
[Replace Error Code with
Exception](replace-error-code-with-exception.html)
:::
::::
:::::
::::::

::: next
#### Suivant

[Dealing with Generalization []{.fa
.fa-arrow-right}](refactoring/techniques/dealing-with-generalization.html){.btn
.btn-primary rel="next"}
:::

::: prev
#### Retour

[[]{.fa .fa-arrow-left} Replace Error Code with
Exception](replace-error-code-with-exception.html){.btn .btn-default
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
