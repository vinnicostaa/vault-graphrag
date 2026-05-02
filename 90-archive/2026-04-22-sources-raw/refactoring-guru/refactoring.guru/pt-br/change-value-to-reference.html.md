:::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::: {.refactoring .page .text}
::: breadcrumb
[](../index.html){.home} / [Refactoring](refactoring.html){.type} /
[Techniques](refactoring/techniques.html){.type} / [Organizing
Data](refactoring/techniques/organizing-data.html){.type}
:::

# Change Value to Reference {#change-value-to-reference .title}

::::: before-after-container
::: before
### Problem

So you have many identical instances of a single class that you need to
replace with a single object.
:::

::: after
### Solution

Convert the identical objects to a single reference object.
:::
:::::

:::::::::: examples
::::::::: {.before-after .before-after-container}
::::: before
::: ba
Before
:::

::: ba-container
![Change Value to Reference -
Before](../images/refactoring/diagrams/Change%20Value%20to%20Reference%20-%20Beforefa98.png?id=b2e65e5bb87366e8195bab6933c15250){width="396"}
:::
:::::

::::: after
::: ba
After
:::

::: ba-container
![Change Value to Reference -
After](../images/refactoring/diagrams/Change%20Value%20to%20Reference%20-%20Afterfe5d.png?id=20d3bdea32264097859011bacb4ff19f){width="396"}
:::
:::::
:::::::::
::::::::::

### Why Refactor

In many systems, objects can be classified as either values or
references.

-   **References**: when one real-world object corresponds to only one
    object in the program. References are usually
    user/order/product/etc. objects.

-   **Values**: one real-world object corresponds to multiple objects in
    the program. These objects could be dates, phone numbers, addresses,
    colors, and the like.

The selection of reference vs. value isn't always clear-cut. Sometimes
there's a simple value with a small amount of unchanging data. Then it
becomes necessary to add changeable data and pass these changes every
time the object is accessed. In this case it becomes necessary to
convert it to a reference.

### Benefits

-   An object contains all the most current information about a
    particular entity. If the object is changed in one part of the
    program, these changes are accessible from the other parts of the
    program that make use of the object.

### Drawbacks

-   References are much harder to implement.

### How to Refactor

1.  Use [Replace Constructor with Factory
    Method](replace-constructor-with-factory-method.html) on the class
    from which the references are to be generated.

2.  Determine which object will be responsible for providing access to
    references. Instead of creating a new object, when you need one you
    now need to get it from a storage object or static dictionary field.

3.  Determine whether references will be created in advance or
    dynamically as necessary. If objects are created in advance, make
    sure to load them before use.

4.  Change the factory method so that it returns a reference. If objects
    are created in advance, decide how to handle errors when a
    non-existent object is requested. You may also need to use [Rename
    Method](rename-method.html) to inform that the method returns only
    existing objects.

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
[Change Reference to Value](change-reference-to-value.html)
:::
::::
:::::
::::::

::: next
#### Leia a seguir

[Change Reference to Value []{.fa
.fa-arrow-right}](change-reference-to-value.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Voltar

[[]{.fa .fa-arrow-left} Replace Data Value with
Object](replace-data-value-with-object.html){.btn .btn-default
rel="prev"}
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
