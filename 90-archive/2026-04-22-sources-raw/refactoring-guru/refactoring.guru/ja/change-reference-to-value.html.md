:::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::: {.refactoring .page .text}
::: breadcrumb
[](../index.html){.home} / [Refactoring](refactoring.html){.type} /
[Techniques](refactoring/techniques.html){.type} / [Organizing
Data](refactoring/techniques/organizing-data.html){.type}
:::

# Change Reference to Value {#change-reference-to-value .title}

::::: before-after-container
::: before
### Problem

You have a reference object that\'s too small and infrequently changed
to justify managing its life cycle.
:::

::: after
### Solution

Turn it into a value object.
:::
:::::

:::::::::: examples
::::::::: {.before-after .before-after-container}
::::: before
::: ba
Before
:::

::: ba-container
![Change Reference to Value -
Before](../images/refactoring/diagrams/Change%20Reference%20to%20Value%20-%20Before2033.png?id=584dcba345161853463376cef73ab205){width="377"}
:::
:::::

::::: after
::: ba
After
:::

::: ba-container
![Change Reference to Value -
After](../images/refactoring/diagrams/Change%20Reference%20to%20Value%20-%20After05e8.png?id=08ac04b5ee1d4845fb6ebe409a4a6614){width="377"}
:::
:::::
:::::::::
::::::::::

### Why Refactor

Inspiration to switch from a reference to a value may come from the
inconvenience of working with the reference. References require
management on your part:

-   They always require requesting the necessary object from storage.

-   References in memory may be inconvenient to work with.

-   Working with references is particularly difficult, compared to
    values, on distributed and parallel systems.

Values are especially useful if you would rather have unchangeable
objects than objects whose state may change during their lifetime.

### Benefits

-   One important property of objects is that they should be
    unchangeable. The same result should be received for each query that
    returns an object value. If this is true, no problems arise if there
    are many objects representing the same thing.

-   Values are much easier to implement.

### Drawbacks

-   If a value is changeable, make sure if any object changes that the
    values in all the other objects representing the same entity are
    updated. This is so burdensome that it\'s easier to create a
    reference for this purpose.

### How to Refactor

1.  Make the object unchangeable. The object shouldn\'t have any setters
    or other methods that change its state and data ([Remove Setting
    Method](remove-setting-method.html) may help here). The only place
    where data should be assigned to the fields of a value object is a
    constructor.

2.  Create a comparison method to be able to compare two values.

3.  Check whether you can delete the factory method and make the object
    constructor public.

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
### Anti-refactoring

:::: dl
::: dt
[Change Value to Reference](change-value-to-reference.html)
:::
::::
:::::
::::::

::: next
#### 次を読む

[Replace Array with Object []{.fa
.fa-arrow-right}](replace-array-with-object.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### 戻る

[[]{.fa .fa-arrow-left} Change Value to
Reference](change-value-to-reference.html){.btn .btn-default rel="prev"}
:::
::::::::::::::::::::::::

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
:::::::::::::::::::::::::::::
::::::::::::::::::::::::::::::
