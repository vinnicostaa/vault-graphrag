::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::::::: {.refactoring .page .text}
::: breadcrumb
[](../index.html){.home} / [Refactoring](refactoring.html){.type} /
[Techniques](refactoring/techniques.html){.type} / [Dealing with
Generalization](refactoring/techniques/dealing-with-generalization.html){.type}
:::

# Form Template Method {#form-template-method .title}

::::: before-after-container
::: before
### Problem

Your subclasses implement algorithms that contain similar steps in the
same order.
:::

::: after
### Solution

Move the algorithm structure and identical steps to a superclass, and
leave implementation of the different steps in the subclasses.
:::
:::::

:::::::::: examples
::::::::: {.before-after .before-after-container}
::::: before
::: ba
Before
:::

::: ba-container
![Form Template Method -
Before](../images/refactoring/diagrams/Form%20Template%20Method%20-%20Beforee8c2.png?id=858de51c181ba956e5ecf63e3a88fda2){width="696"}
:::
:::::

::::: after
::: ba
After
:::

::: ba-container
![Form Template Method -
After](../images/refactoring/diagrams/Form%20Template%20Method%20-%20After9b6d.png?id=7a2977c70d9fcd80c96307a877343828){width="640"}
:::
:::::
:::::::::
::::::::::

### Why Refactor

Subclasses are developed in parallel, sometimes by different people,
which leads to code duplication, errors, and difficulties in code
maintenance, since each change must be made in all subclasses.

### Benefits

-   Code duplication doesn't always refer to cases of simple copy/paste.
    Often duplication occurs at a higher level, such as when you have a
    method for sorting numbers and a method for sorting object
    collections that are differentiated only by the comparison of
    elements. Creating a template method eliminates this duplication by
    merging the shared algorithm steps in a superclass and leaving just
    the differences in the subclasses.

-   Forming a template method is an example of the *Open/Closed
    Principle* in action. When a new algorithm version appears, you need
    only to create a new subclass; no changes to existing code are
    required.

### How to Refactor

1.  Split algorithms in the subclasses into their constituent parts
    described in separate methods. [Extract Method](extract-method.html)
    can help with this.

2.  The resulting methods that are identical for all subclasses can be
    moved to a superclass via [Pull Up Method](pull-up-method.html).

3.  The non-similar methods can be given consistent names via [Rename
    Method](rename-method.html).

4.  Move the signatures of non-similar methods to a superclass as
    abstract ones by using [Pull Up Method](pull-up-method.html). Leave
    their implementations in the subclasses.

5.  And finally, pull up the main method of the algorithm to the
    superclass. Now it should work with the method steps described in
    the superclass, both real and abstract.

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
### Implements design pattern

:::: dl
::: dt
[Metoda szablonowa](design-patterns/template-method.html)
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
#### Czytaj dalej

[Replace Inheritance with Delegation []{.fa
.fa-arrow-right}](replace-inheritance-with-delegation.html){.btn
.btn-primary rel="next"}
:::

::: prev
#### Wróć

[[]{.fa .fa-arrow-left} Collapse
Hierarchy](collapse-hierarchy.html){.btn .btn-default rel="prev"}
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
