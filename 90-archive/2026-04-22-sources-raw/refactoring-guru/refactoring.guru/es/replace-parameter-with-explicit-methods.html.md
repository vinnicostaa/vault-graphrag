:::::::::::::::::::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::::::::::::::::::::::::::::: {.refactoring .page .text}
::: breadcrumb
[](../index.html){.home} / [Refactoring](refactoring.html){.type} /
[Techniques](refactoring/techniques.html){.type} / [Simplifying Method
Calls](refactoring/techniques/simplifying-method-calls.html){.type}
:::

# Replace Parameter with Explicit Methods {#replace-parameter-with-explicit-methods .title}

::::: before-after-container
::: before
### Problem

A method is split into parts, each of which is run depending on the
value of a parameter.
:::

::: after
### Solution

Extract the individual parts of the method into their own methods and
call them instead of the original method.
:::
:::::

:::::::::::::::::::::::::::: examples
::::::: {.before-after .before-after-container .java data-lang="java"}
:::: before
::: ba
Before
:::

``` {.code lang="java"}
void setValue(String name, int value) {
  if (name.equals("height")) {
    height = value;
    return;
  }
  if (name.equals("width")) {
    width = value;
    return;
  }
  Assert.shouldNeverReachHere();
}
```
::::

:::: after
::: ba
After
:::

``` {.code lang="java"}
void setHeight(int arg) {
  height = arg;
}
void setWidth(int arg) {
  width = arg;
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
void SetValue(string name, int value) 
{
  if (name.Equals("height")) 
  {
    height = value;
    return;
  }
  if (name.Equals("width")) 
  {
    width = value;
    return;
  }
  Assert.Fail();
}
```
::::

:::: after
::: ba
After
:::

``` {.code lang="csharp"}
void SetHeight(int arg) 
{
  height = arg;
}
void SetWidth(int arg) 
{
  width = arg;
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
function setValue($name, $value) {
  if ($name === "height") {
    $this->height = $value;
    return;
  }
  if ($name === "width") {
    $this->width = $value;
    return;
  }
  assert("Should never reach here");
}
```
::::

:::: after
::: ba
After
:::

``` {.code lang="php"}
function setHeight($arg) {
  $this->height = $arg;
}
function setWidth($arg) {
  $this->width = $arg;
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
def output(self, name):
    if name == "banner"
        # Print the banner.
        # ...
    if name == "info"
        # Print the info.
        # ...
```
::::

:::: after
::: ba
After
:::

``` {.code lang="python"}
def outputBanner(self):
    # Print the banner.
    # ...

def outputInfo(self):
    # Print the info.
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
 setValue(name: string, value: number): void {
  if (name.equals("height")) {
    height = value;
    return;
  }
  if (name.equals("width")) {
    width = value;
    return;
  }
  
}
```
::::

:::: after
::: ba
After
:::

``` {.code lang="typescript"}
setHeight(arg: number): void {
  height = arg;
}
setWidth(arg: number): number {
  width = arg;
}
```
::::
:::::::
::::::::::::::::::::::::::::

### Why Refactor

A method containing parameter-dependent variants has grown massive.
Non-trivial code is run in each branch and new variants are added very
rarely.

### Benefits

-   Improves code readability. It's much easier to understand the
    purpose of `startEngine()` than `setValue("engineEnabled", true)`.

### When Not to Use

-   Don't replace a parameter with explicit methods if a method is
    rarely changed and new variants aren't added inside it.

### How to Refactor

1.  For each variant of the method, create a separate method. Run these
    methods based on the value of a parameter in the main method.

2.  Find all places where the original method is called. In these
    places, place a call for one of the new parameter-dependent
    variants.

3.  When no calls to the original method remain, delete it.

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
[Parameterize Method](parameterize-method.html)
:::
::::
:::::

::::: {.col-xs-12 .col-sm-6 .col-lg-4}
### Similar refactorings

:::: dl
::: dt
[Replace Conditional with
Polymorphism](replace-conditional-with-polymorphism.html)
:::
::::
:::::

::::::: {.col-xs-12 .col-sm-6 .col-lg-4}
### Eliminates smell

:::: dl
::: dt
[Switch Statements](smells/switch-statements.html)
:::
::::

:::: dl
::: dt
[Long Method](smells/long-method.html)
:::
::::
:::::::
::::::::::::::

::: next
#### Leer siguiente

[Preserve Whole Object []{.fa
.fa-arrow-right}](preserve-whole-object.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Volver

[[]{.fa .fa-arrow-left} Parameterize
Method](parameterize-method.html){.btn .btn-default rel="prev"}
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
