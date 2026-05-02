:::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::::::::: {.refactoring .page .text}
::: breadcrumb
[](index.html){.home} / [Refactoring](refactoring.html){.type} /
[Techniques](refactoring/techniques.html){.type} / [Dealing with
Generalization](refactoring/techniques/dealing-with-generalization.html){.type}
:::

# Collapse Hierarchy {#collapse-hierarchy .title}

::::: before-after-container
::: before
### Problem

You have a class hierarchy in which a subclass is practically the same
as its superclass.
:::

::: after
### Solution

Merge the subclass and superclass.
:::
:::::

:::::::::: examples
::::::::: {.before-after .before-after-container}
::::: before
::: ba
Before
:::

::: ba-container
![Collapse Hierarchy -
Before](images/refactoring/diagrams/Collapse%20Hierarchy%20-%20Beforea922.png?id=e95d97b3a9d564fbdba5ec0b76748f88){width="152"}
:::
:::::

::::: after
::: ba
After
:::

::: ba-container
![Collapse Hierarchy -
After](images/refactoring/diagrams/Collapse%20Hierarchy%20-%20After1803.png?id=db86997e7b68eaac829168048cd02a8b){width="152"}
:::
:::::
:::::::::
::::::::::

### Why Refactor

Your program has grown over time and a subclass and superclass have
become practically the same. A feature was removed from a subclass, a
method was moved to the superclass\... and now you have two look-alike
classes.

### Benefits

-   Program complexity is reduced. Fewer classes mean fewer things to
    keep straight in your head and fewer breakable moving parts to worry
    about during future code changes.

-   Navigating through your code is easier when methods are defined in
    one class early. You don't need to comb through the entire hierarchy
    to find a particular method.

### When Not to Use

-   Does the class hierarchy that you're refactoring have more than one
    subclass? If so, after refactoring is complete, the remaining
    subclasses should become the inheritors of the class in which the
    hierarchy was collapsed.

-   But keep in mind that this can lead to violations of the *Liskov
    substitution principle*. For example, if your program emulates city
    transport networks and you accidentally collapse the `Transport`
    superclass into the `Car` subclass, then the `Plane` class may
    become the inheritor of `Car`. Oops!

### How to Refactor

1.  Select which class is easier to remove: the superclass or its
    subclass.

2.  Use [Pull Up Field](pull-up-field.html) and [Pull Up
    Method](pull-up-method.html) if you decide to get rid of the
    subclass. If you choose to eliminate the superclass, go for [Push
    Down Field](push-down-field.html) and [Push Down
    Method](push-down-method.html).

3.  Replace all uses of the class that you're deleting with the class to
    which the fields and methods are to be migrated. Often this will be
    code for creating classes, variable and parameter typing, and
    documentation in code comments.

4.  Delete the empty class.

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

[ Let\'s see...](refactoring/course.html){.btn .btn-secondary}
:::
:::::

:::::::::::: row
:::::: {.col-xs-12 .col-sm-6 .col-lg-6}
### Similar refactorings

::::: dl
::: dt
[Inline Class](inline-class.html)
:::

::: dd
*Collapse Hierarchy* is a variation of [Inline
Class](inline-class.html), where the code moves to superclass or
subclass.
:::
:::::
::::::

::::::: {.col-xs-12 .col-sm-6 .col-lg-6}
### Eliminates smell

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
:::::::
::::::::::::

::: next
#### Read next

[Form Template Method []{.fa
.fa-arrow-right}](form-template-method.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Return

[[]{.fa .fa-arrow-left} Extract Interface](extract-interface.html){.btn
.btn-default rel="prev"}
:::
::::::::::::::::::::::::::::::

:::::: {.prom .banner-sidebar .banner-removable .banner-removable-refactoring data-id="Ref: Sidebar" creative-id="standard-sidebar-en" position="sidebar"}
::::: ban
::: {.image .product-image}
[Sale!]{.banner-discount}
[![](images/refactoring/course/course-cover-en1b63.jpg?id=ac6ad836a27c4a8d579244f15c48a8b4){width="300"
height="300" loading="lazy"}](refactoring/course.html)
:::

::: {.banner-text .banner-text-en}
This refactoring is part of the much bigger **Refactoring Course**.

[ Learn more...](refactoring/course.html){.btn .btn-secondary
.btn-block}
:::
:::::
::::::
:::::::::::::::::::::::::::::::::::
::::::::::::::::::::::::::::::::::::
