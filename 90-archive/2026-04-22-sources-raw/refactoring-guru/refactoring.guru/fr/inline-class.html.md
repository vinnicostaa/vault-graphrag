::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::::::::::: {.refactoring .page .text}
::: breadcrumb
[](../index.html){.home} / [Refactoring](refactoring.html){.type} /
[Techniques](refactoring/techniques.html){.type} / [Moving Features
between
Objects](refactoring/techniques/moving-features-between-objects.html){.type}
:::

# Inline Class {#inline-class .title}

::::: before-after-container
::: before
### Problem

A class does almost nothing and isn't responsible for anything, and no
additional responsibilities are planned for it.
:::

::: after
### Solution

Move all features from the class to another one.
:::
:::::

:::::::::: examples
::::::::: {.before-after .before-after-container}
::::: before
::: ba
Before
:::

::: ba-container
![Inline Class -
Before](../images/refactoring/diagrams/Inline%20Class%20-%20Beforebf8b.png?id=717d80baaa902d09acbaa55fd0539638){width="509"}
:::
:::::

::::: after
::: ba
After
:::

::: ba-container
![Inline Class -
After](../images/refactoring/diagrams/Inline%20Class%20-%20After5bcb.png?id=7cb7db0fe2ab0d17f067d411f9e98f15){width="209"}
:::
:::::
:::::::::
::::::::::

### Why Refactor

-   Often this technique is needed after the features of one class are
    "transplanted" to other classes, leaving that class with little to
    do.

### Benefits

-   Eliminating needless classes frees up operating memory on the
    computer---and bandwidth in your head.

### How to Refactor

1.  In the recipient class, create the public fields and methods present
    in the donor class. Methods should refer to the equivalent methods
    of the donor class.

2.  Replace all references to the donor class with references to the
    fields and methods of the recipient class.

3.  Now test the program and make sure that no errors have been added.
    If tests show that everything is working A-OK, start using [Move
    Method](move-method.html) and [Move Field](move-field.html) to
    completely transplant all functionality to the recipient class from
    the original one. Continue doing so until the original class is
    completely empty.

4.  Delete the original class.

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

::::::::::::: row
::::: {.col-xs-12 .col-sm-6 .col-lg-6}
### Anti-refactoring

:::: dl
::: dt
[Extract Class](extract-class.html)
:::
::::
:::::

::::::::: {.col-xs-12 .col-sm-6 .col-lg-6}
### Eliminates smell

:::: dl
::: dt
[Shotgun Surgery](smells/shotgun-surgery.html)
:::
::::

:::: dl
::: dt
[Lazy Class](smells/lazy-class.html)
:::
::::

:::: dl
::: dt
[Speculative Generality](smells/speculative-generality.html)
:::
::::
:::::::::
:::::::::::::

::: next
#### Suivant

[Hide Delegate []{.fa .fa-arrow-right}](hide-delegate.html){.btn
.btn-primary rel="next"}
:::

::: prev
#### Retour

[[]{.fa .fa-arrow-left} Extract Class](extract-class.html){.btn
.btn-default rel="prev"}
:::
:::::::::::::::::::::::::::::::

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
::::::::::::::::::::::::::::::::::::
:::::::::::::::::::::::::::::::::::::
