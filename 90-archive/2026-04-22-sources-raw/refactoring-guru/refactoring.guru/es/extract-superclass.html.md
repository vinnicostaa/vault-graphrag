::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::::::: {.refactoring .page .text}
::: breadcrumb
[](../index.html){.home} / [Refactoring](refactoring.html){.type} /
[Techniques](refactoring/techniques.html){.type} / [Dealing with
Generalization](refactoring/techniques/dealing-with-generalization.html){.type}
:::

# Extract Superclass {#extract-superclass .title}

::::: before-after-container
::: before
### Problem

You have two classes with common fields and methods.
:::

::: after
### Solution

Create a shared superclass for them and move all the identical fields
and methods to it.
:::
:::::

:::::::::: examples
::::::::: {.before-after .before-after-container}
::::: before
::: ba
Before
:::

::: ba-container
![Extract Superclass -
Before](../images/refactoring/diagrams/Extract%20Superclass%20-%20Beforef473.png?id=e8192774aeefddece5b3c1a7a868127d){width="209"}
:::
:::::

::::: after
::: ba
After
:::

::: ba-container
![Extract Superclass -
After](../images/refactoring/diagrams/Extract%20Superclass%20-%20After12ec.png?id=1cff46d95be1632df8af715e14ea88c9){width="509"}
:::
:::::
:::::::::
::::::::::

### Why Refactor

One type of code duplication occurs when two classes perform similar
tasks in the same way, or perform similar tasks in different ways.
Objects offer a built-in mechanism for simplifying such situations via
inheritance. But oftentimes this similarity remains unnoticed until
classes are created, necessitating that an inheritance structure be
created later.

### Benefits

-   Code deduplication. Common fields and methods now "live" in one
    place only.

### When Not to Use

-   You can not apply this technique to classes that already have a
    superclass.

### How to Refactor

1.  Create an abstract superclass.

2.  Use [Pull Up Field](pull-up-field.html), [Pull Up
    Method](pull-up-method.html), and [Pull Up Constructor
    Body](pull-up-constructor-body.html) to move the common
    functionality to a superclass. Start with the fields, since in
    addition to the common fields you will need to move the fields that
    are used in the common methods.

3.  Look for places in the client code where use of subclasses can be
    replaced with your new class (such as in type declarations).

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

::::::::: row
::::: {.col-xs-12 .col-sm-6 .col-lg-6}
### Similar refactorings

:::: dl
::: dt
[Extract Interface](extract-interface.html)
:::
::::
:::::

::::: {.col-xs-12 .col-sm-6 .col-lg-6}
### Eliminates smell

:::: dl
::: dt
[Duplicate Code](smells/duplicate-code.html)
:::
::::
:::::
:::::::::

::: next
#### Leer siguiente

[Extract Interface []{.fa .fa-arrow-right}](extract-interface.html){.btn
.btn-primary rel="next"}
:::

::: prev
#### Volver

[[]{.fa .fa-arrow-left} Extract Subclass](extract-subclass.html){.btn
.btn-default rel="prev"}
:::
:::::::::::::::::::::::::::

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
::::::::::::::::::::::::::::::::
:::::::::::::::::::::::::::::::::
