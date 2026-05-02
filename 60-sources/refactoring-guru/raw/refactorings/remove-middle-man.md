::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::::::: {.refactoring .page .text}
::: breadcrumb
[](index.html){.home} / [Refactoring](refactoring.html){.type} /
[Techniques](refactoring/techniques.html){.type} / [Moving Features
between
Objects](refactoring/techniques/moving-features-between-objects.html){.type}
:::

# Remove Middle Man {#remove-middle-man .title}

::::: before-after-container
::: before
### Problem

A class has too many methods that simply delegate to other objects.
:::

::: after
### Solution

Delete these methods and force the client to call the end methods
directly.
:::
:::::

:::::::::: examples
::::::::: {.before-after .before-after-container}
::::: before
::: ba
Before
:::

::: ba-container
![Remove Middle Man -
Before](images/refactoring/diagrams/Remove%20Middle%20Man%20-%20Before921c.png?id=f51110f3e0d4423b3f9088e92fc3dce4){width="153"}
:::
:::::

::::: after
::: ba
After
:::

::: ba-container
![Remove Middle Man -
After](images/refactoring/diagrams/Remove%20Middle%20Man%20-%20After0412.png?id=f7de1016e76545f7c51af09463ce5f4c){width="415"}
:::
:::::
:::::::::
::::::::::

### Why Refactor

To describe this technique, we'll use the terms from [Hide
Delegate](hide-delegate.html), which are:

-   *Server* is the object to which the client has direct access.

-   *Delegate* is the end object that contains the functionality needed
    by the client.

There are two types of problems:

1.  The *server-class* doesn't do anything itself and simply creates
    needless complexity. In this case, give thought to whether this
    class is needed at all.

2.  Every time a new feature is added to the *delegate*, you need to
    create a delegating method for it in the *server-class*. If a lot of
    changes are made, this will be rather tiresome.

### How to Refactor

1.  Create a getter for accessing the *delegate-class* object from the
    *server-class* object.

2.  Replace calls to delegating methods in the *server-class* with
    direct calls for methods in the *delegate-class*.

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

::::::::: row
::::: {.col-xs-12 .col-sm-6 .col-lg-6}
### Anti-refactoring

:::: dl
::: dt
[Hide Delegate](hide-delegate.html)
:::
::::
:::::

::::: {.col-xs-12 .col-sm-6 .col-lg-6}
### Eliminates smell

:::: dl
::: dt
[Middle Man](smells/middle-man.html)
:::
::::
:::::
:::::::::

::: next
#### Read next

[Introduce Foreign Method []{.fa
.fa-arrow-right}](introduce-foreign-method.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Return

[[]{.fa .fa-arrow-left} Hide Delegate](hide-delegate.html){.btn
.btn-default rel="prev"}
:::
:::::::::::::::::::::::::::

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
::::::::::::::::::::::::::::::::
:::::::::::::::::::::::::::::::::
