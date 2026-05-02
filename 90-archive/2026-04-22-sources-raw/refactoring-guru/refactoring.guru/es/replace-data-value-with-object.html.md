:::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::: {.refactoring .page .text}
::: breadcrumb
[](../index.html){.home} / [Refactoring](refactoring.html){.type} /
[Techniques](refactoring/techniques.html){.type} / [Organizing
Data](refactoring/techniques/organizing-data.html){.type}
:::

# Replace Data Value with Object {#replace-data-value-with-object .title}

::::: before-after-container
::: before
### Problem

A class (or group of classes) contains a data field. The field has its
own behavior and associated data.
:::

::: after
### Solution

Create a new class, place the old field and its behavior in the class,
and store the object of the class in the original class.
:::
:::::

:::::::::: examples
::::::::: {.before-after .before-after-container}
::::: before
::: ba
Before
:::

::: ba-container
![Replace Data Value with Object -
Before](../images/refactoring/diagrams/Replace%20Data%20Value%20with%20Object%20-%20Before08a7.png?id=f9ecd087d0e9e71ec8be6622ac36e573){width="190"}
:::
:::::

::::: after
::: ba
After
:::

::: ba-container
![Replace Data Value with Object -
After](../images/refactoring/diagrams/Replace%20Data%20Value%20with%20Object%20-%20After5a05.png?id=26b91a5742429b46df4885713fea0a74){width="434"}
:::
:::::
:::::::::
::::::::::

### Why Refactor

This refactoring is basically a special case of [Extract
Class](extract-class.html). What makes it different is the cause of the
refactoring.

In [Extract Class](extract-class.html), we have a single class that's
responsible for different things and we want to split up its
responsibilities.

With replacement of a data value with an object, we have a primitive
field (number, string, etc.) that's no longer so simple due to growth of
the program and now has associated data and behaviors. On the one hand,
there's nothing scary about these fields in and of themselves. However,
this fields-and-behaviors family can be present in several classes
simultaneously, creating duplicate code.

Therefore, for all this we create a new class and move both the field
and the related data and behaviors to it.

### Benefits

-   Improves relatedness inside classes. Data and the relevant behaviors
    are inside a single class.

### How to Refactor

Before you begin with refactoring, see if there are direct references to
the field from within the class. If so, use [Self Encapsulate
Field](self-encapsulate-field.html) in order to hide it in the original
class.

1.  Create a new class and copy your field and relevant getter to it. In
    addition, create a constructor that accepts the simple value of the
    field. This class won't have a setter since each new field value
    that's sent to the original class will create a new value object.

2.  In the original class, change the field type to the new class.

3.  In the getter in the original class, invoke the getter of the
    associated object.

4.  In the setter, create a new value object. You may need to also
    create a new object in the constructor if initial values had been
    set there for the field previously.

### Next Steps

After applying this refactoring technique, it's wise to apply [Change
Value to Reference](change-value-to-reference.html) on the field that
contains the object. This allows storing a reference to a single object
that corresponds to a value instead of storing dozens of objects for one
and the same value.

Most often this approach is needed when you want to have one object be
responsible for one real-world object (such as users, orders, documents
and so forth). At the same time, this approach won't be useful for
objects such as dates, money, ranges, etc.

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

::: next
#### Leer siguiente

[Change Value to Reference []{.fa
.fa-arrow-right}](change-value-to-reference.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Volver

[[]{.fa .fa-arrow-left} Self Encapsulate
Field](self-encapsulate-field.html){.btn .btn-default rel="prev"}
:::
::::::::::::::::::::

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
:::::::::::::::::::::::::
::::::::::::::::::::::::::
