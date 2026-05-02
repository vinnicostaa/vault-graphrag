::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::::::::::::: {.refactoring .page .text}
::: breadcrumb
[](../index.html){.home} / [Refactoring](refactoring.html){.type} /
[Techniques](refactoring/techniques.html){.type} / [Simplifying Method
Calls](refactoring/techniques/simplifying-method-calls.html){.type}
:::

# Introduce Parameter Object {#introduce-parameter-object .title}

::::: before-after-container
::: before
### Problem

Your methods contain a repeating group of parameters.
:::

::: after
### Solution

Replace these parameters with an object.
:::
:::::

:::::::::: examples
::::::::: {.before-after .before-after-container}
::::: before
::: ba
Before
:::

::: ba-container
![Introduce Parameter Object -
Before](../images/refactoring/diagrams/Introduce%20Parameter%20Object%20-%20Before9531.png?id=023aa35781e4c8d16e0faf7172646d1e){width="359"}
:::
:::::

::::: after
::: ba
After
:::

::: ba-container
![Introduce Parameter Object -
After](../images/refactoring/diagrams/Introduce%20Parameter%20Object%20-%20After803b.png?id=b5ed29b5678759399a83c229b1837730){width="359"}
:::
:::::
:::::::::
::::::::::

### Why Refactor

Identical groups of parameters are often encountered in multiple
methods. This causes code duplication of both the parameters themselves
and of related operations. By consolidating parameters in a single
class, you can also move the methods for handling this data there as
well, freeing the other methods from this code.

### Benefits

-   More readable code. Instead of a hodgepodge of parameters, you see a
    single object with a comprehensible name.

-   Identical groups of parameters scattered here and there create their
    own kind of code duplication: while identical code isn't being
    called, identical groups of parameters and arguments are constantly
    encountered.

### Drawbacks

-   If you move only data to a new class and don't plan to move any
    behaviors or related operations there, this begins to smell of a
    [Data Class](smells/data-class.html).

### How to Refactor

1.  Create a new class that will represent your group of parameters.
    Make the class immutable.

2.  In the method that you want to refactor, use [Add
    Parameter](add-parameter.html), which is where your parameter object
    will be passed. In all method calls, pass the object created from
    old method parameters to this parameter.

3.  Now start deleting old parameters from the method one by one,
    replacing them in the code with fields of the parameter object. Test
    the program after each parameter replacement.

4.  When done, see whether there's any point in moving a part of the
    method (or sometimes even the whole method) to a parameter object
    class. If so, use [Move Method](move-method.html) or [Extract
    Method](extract-method.html).

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

::::::::::::::: row
::::: {.col-xs-12 .col-sm-6 .col-lg-6}
### Similar refactorings

:::: dl
::: dt
[Preserve Whole Object](preserve-whole-object.html)
:::
::::
:::::

::::::::::: {.col-xs-12 .col-sm-6 .col-lg-6}
### Eliminates smell

:::: dl
::: dt
[Long Parameter List](smells/long-parameter-list.html)
:::
::::

:::: dl
::: dt
[Data Clumps](smells/data-clumps.html)
:::
::::

:::: dl
::: dt
[Primitive Obsession](smells/primitive-obsession.html)
:::
::::

:::: dl
::: dt
[Long Method](smells/long-method.html)
:::
::::
:::::::::::
:::::::::::::::

::: next
#### Leia a seguir

[Remove Setting Method []{.fa
.fa-arrow-right}](remove-setting-method.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Voltar

[[]{.fa .fa-arrow-left} Replace Parameter with Method
Call](replace-parameter-with-method-call.html){.btn .btn-default
rel="prev"}
:::
:::::::::::::::::::::::::::::::::

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
::::::::::::::::::::::::::::::::::::::
:::::::::::::::::::::::::::::::::::::::
