:::::::::::::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::::::::::::::::::::::: {.refactoring .page .text}
::: breadcrumb
[](../index.html){.home} / [Refactoring](refactoring.html){.type} /
[Techniques](refactoring/techniques.html){.type} / [Moving Features
between
Objects](refactoring/techniques/moving-features-between-objects.html){.type}
:::

# Extract Class {#extract-class .title}

::::: before-after-container
::: before
### Problem

When one class does the work of two, awkwardness results.
:::

::: after
### Solution

Instead, create a new class and place the fields and methods responsible
for the relevant functionality in it.
:::
:::::

:::::::::: examples
::::::::: {.before-after .before-after-container}
::::: before
::: ba
Before
:::

::: ba-container
![Extract Class -
Before](../images/refactoring/diagrams/Extract%20Class%20-%20Before5bcb.png?id=7cb7db0fe2ab0d17f067d411f9e98f15){width="209"}
:::
:::::

::::: after
::: ba
After
:::

::: ba-container
![Extract Class -
After](../images/refactoring/diagrams/Extract%20Class%20-%20Afterbf8b.png?id=717d80baaa902d09acbaa55fd0539638){width="509"}
:::
:::::
:::::::::
::::::::::

### Why Refactor

Classes always start out clear and easy to understand. They do their job
and mind their own business as it were, without butting into the work of
other classes. But as the program expands, a method is added and then a
field\... and eventually, some classes are performing more
responsibilities than ever envisioned.

### Benefits

-   This refactoring method will help maintain adherence to the *Single
    Responsibility Principle*. The code of your classes will be more
    obvious and understandable.

-   Single-responsibility classes are more reliable and tolerant of
    changes. For example, say that you have a class responsible for ten
    different things. When you change this class to make it better for
    one thing, you risk breaking it for the nine others.

### Drawbacks

-   If you "overdo it" with this refactoring technique, you will have to
    resort to [Inline Class](inline-class.html).

### How to Refactor

Before starting, decide on how exactly you want to split up the
responsibilities of the class.

1.  Create a new class to contain the relevant functionality.

2.  Create a relationship between the old class and the new one.
    Optimally, this relationship is unidirectional; this allows reusing
    the second class without any issues. Nonetheless, if you think
    that a two-way relationship is necessary, this can always be set up.

3.  Use [Move Field](move-field.html) and [Move
    Method](move-method.html) for each field and method that you have
    decided to move to the new class. For methods, start with private
    ones in order to reduce the risk of making a large number of errors.
    Try to relocate just a little bit at a time and test the results
    after each move, in order to avoid a pileup of error-fixing at the
    very end.

    After you're done moving, take one more look at the resulting
    classes. An old class with changed responsibilities may be renamed
    for increased clarity. Check again to see whether you can get rid of
    two-way class relationships, if any are present.

4.  Also give thought to accessibility to the new class from the
    outside. You can hide the class from the client entirely by
    making it private, managing it via the fields from the old class.
    Alternatively, you can make it a public one by allowing the
    client to change values directly. Your decision here depends on how
    safe it's for the behavior of the old class when unexpected direct
    changes are made to the values in the new class.

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

:::::::::::::::::::::::::: row
::::: {.col-xs-12 .col-sm-6 .col-lg-4}
### Anti-refactoring

:::: dl
::: dt
[Inline Class](inline-class.html)
:::
::::
:::::

::::::: {.col-xs-12 .col-sm-6 .col-lg-4}
### Similar refactorings

:::: dl
::: dt
[Extract Subclass](extract-subclass.html)
:::
::::

:::: dl
::: dt
[Replace Data Value with Object](replace-data-value-with-object.html)
:::
::::
:::::::

::::::::::::::::: {.col-xs-12 .col-sm-6 .col-lg-4}
### Eliminates smell

:::: dl
::: dt
[Duplicate Code](smells/duplicate-code.html)
:::
::::

:::: dl
::: dt
[Large Class](smells/large-class.html)
:::
::::

:::: dl
::: dt
[Divergent Change](smells/divergent-change.html)
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
[Temporary Field](smells/temporary-field.html)
:::
::::

:::: dl
::: dt
[Inappropriate Intimacy](smells/inappropriate-intimacy.html)
:::
::::
:::::::::::::::::
::::::::::::::::::::::::::

::: next
#### Czytaj dalej

[Inline Class []{.fa .fa-arrow-right}](inline-class.html){.btn
.btn-primary rel="next"}
:::

::: prev
#### Wróć

[[]{.fa .fa-arrow-left} Move Field](move-field.html){.btn .btn-default
rel="prev"}
:::
::::::::::::::::::::::::::::::::::::::::::::

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
:::::::::::::::::::::::::::::::::::::::::::::::::
::::::::::::::::::::::::::::::::::::::::::::::::::
