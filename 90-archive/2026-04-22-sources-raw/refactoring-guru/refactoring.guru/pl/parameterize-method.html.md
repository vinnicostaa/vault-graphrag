:::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::::::::::: {.refactoring .page .text}
::: breadcrumb
[](../index.html){.home} / [Refactoring](refactoring.html){.type} /
[Techniques](refactoring/techniques.html){.type} / [Simplifying Method
Calls](refactoring/techniques/simplifying-method-calls.html){.type}
:::

# Parameterize Method {#parameterize-method .title}

::::: before-after-container
::: before
### Problem

Multiple methods perform similar actions that are different only in
their internal values, numbers or operations.
:::

::: after
### Solution

Combine these methods by using a parameter that will pass the necessary
special value.
:::
:::::

:::::::::: examples
::::::::: {.before-after .before-after-container}
::::: before
::: ba
Before
:::

::: ba-container
![Parameterize Method -
Before](../images/refactoring/diagrams/Parameterize%20Method%20-%20Beforeb413.png?id=28369c5a763d56c4aa68ff0a9d2d6beb){width="171"}
:::
:::::

::::: after
::: ba
After
:::

::: ba-container
![Parameterize Method -
After](../images/refactoring/diagrams/Parameterize%20Method%20-%20After948e.png?id=fd1602ffde98a8fea3ad50cb63e14553){width="171"}
:::
:::::
:::::::::
::::::::::

### Why Refactor

If you have similar methods, you probably have duplicate code, with all
the consequences that this entails.

What's more, if you need to add yet another version of this
functionality, you will have to create yet another method. Instead, you
could simply run the existing method with a different parameter.

### Drawbacks

-   Sometimes this refactoring technique can be taken too far,
    resulting in a long and complicated common method instead of
    multiple simpler ones.

-   Also be careful when moving activation/deactivation of
    functionality to a parameter. This can eventually lead to
    creation of a large conditional operator that will need to be
    treated via [Replace Parameter with Explicit
    Methods](replace-parameter-with-explicit-methods.html).

### How to Refactor

1.  Create a new method with a parameter and move it to the code that's
    the same for all classes, by applying [Extract
    Method](extract-method.html). Note that sometimes only a certain
    part of methods is actually the same. In this case, refactoring
    consists of extracting only the same part to a new method.

2.  In the code of the new method, replace the special/differing value
    with a parameter.

3.  For each old method, find the places where it's called, replacing
    these calls with calls to the new method that include a parameter.
    Then delete the old method.

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
[Replace Parameter with Explicit
Methods](replace-parameter-with-explicit-methods.html)
:::
::::
:::::

::::::: {.col-xs-12 .col-sm-6 .col-lg-4}
### Similar refactorings

:::: dl
::: dt
[Extract Method](extract-method.html)
:::
::::

:::: dl
::: dt
[Form Template Method](form-template-method.html)
:::
::::
:::::::

::::: {.col-xs-12 .col-sm-6 .col-lg-4}
### Eliminates smell

:::: dl
::: dt
[Duplicate Code](smells/duplicate-code.html)
:::
::::
:::::
::::::::::::::

::: next
#### Czytaj dalej

[Replace Parameter with Explicit Methods []{.fa
.fa-arrow-right}](replace-parameter-with-explicit-methods.html){.btn
.btn-primary rel="next"}
:::

::: prev
#### Wróć

[[]{.fa .fa-arrow-left} Separate Query from
Modifier](separate-query-from-modifier.html){.btn .btn-default
rel="prev"}
:::
::::::::::::::::::::::::::::::::

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
:::::::::::::::::::::::::::::::::::::
::::::::::::::::::::::::::::::::::::::
