::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::::::::::::: {.refactoring .page .text}
::: breadcrumb
[](../index.html){.home} / [Refactoring](refactoring.html){.type} /
[Techniques](refactoring/techniques.html){.type} / [Dealing with
Generalization](refactoring/techniques/dealing-with-generalization.html){.type}
:::

# Push Down Field {#push-down-field .title}

::::: before-after-container
::: before
### Problem

Is a field used only in a few subclasses?
:::

::: after
### Solution

Move the field to these subclasses.
:::
:::::

:::::::::: examples
::::::::: {.before-after .before-after-container}
::::: before
::: ba
Before
:::

::: ba-container
![Push Down Field -
Before](../images/refactoring/diagrams/Push%20Down%20Field%20-%20Before4e32.png?id=79ff2461236c616fcfbc5b0ab8abf5d9){width="452"}
:::
:::::

::::: after
::: ba
After
:::

::: ba-container
![Push Down Field -
After](../images/refactoring/diagrams/Push%20Down%20Field%20-%20After4461.png?id=b48b45a448c27122df61dab2c2451227){width="452"}
:::
:::::
:::::::::
::::::::::

### Why Refactor

Although it was planned to use a field universally for all classes, in
reality the field is used only in some subclasses. This situation can
occur when planned features fail to pan out, for example.

This can also occur due to extraction (or removal) of part of the
functionality of class hierarchies.

### Benefits

-   Improves internal class coherency. A field is located where it\'s
    actually used.

-   When moving to several subclasses simultaneously, you can develop
    the fields independently of each other. This does create code
    duplication, yes, so push down fields only when you really do intend
    to use the fields in different ways.

### How to Refactor

1.  Declare a field in all the necessary subclasses.

2.  Remove the field from the superclass.

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
::::: {.col-xs-12 .col-sm-6 .col-lg-3}
### Anti-refactoring

:::: dl
::: dt
[Pull Up Field](pull-up-field.html)
:::
::::
:::::

::::: {.col-xs-12 .col-sm-6 .col-lg-3}
### Similar refactorings

:::: dl
::: dt
[Push Down Method](push-down-method.html)
:::
::::
:::::

::::: {.col-xs-12 .col-sm-6 .col-lg-3}
### Helps other refactorings

:::: dl
::: dt
[Extract Subclass](extract-subclass.html)
:::
::::
:::::

::::: {.col-xs-12 .col-sm-6 .col-lg-3}
### Eliminates smell

:::: dl
::: dt
[Refused Bequest](smells/refused-bequest.html)
:::
::::
:::::
:::::::::::::::

::: next
#### 次を読む

[Extract Subclass []{.fa .fa-arrow-right}](extract-subclass.html){.btn
.btn-primary rel="next"}
:::

::: prev
#### 戻る

[[]{.fa .fa-arrow-left} Push Down Method](push-down-method.html){.btn
.btn-default rel="prev"}
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
