:::::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::::::::::::::: {.refactoring .page .text}
::: breadcrumb
[](../index.html){.home} / [Refactoring](refactoring.html){.type} /
[Techniques](refactoring/techniques.html){.type} / [Moving Features
between
Objects](refactoring/techniques/moving-features-between-objects.html){.type}
:::

# Move Field {#move-field .title}

::::: before-after-container
::: before
### Problem

A field is used more in another class than in its own class.
:::

::: after
### Solution

Create a field in a new class and redirect all users of the old field to
it.
:::
:::::

:::::::::: examples
::::::::: {.before-after .before-after-container}
::::: before
::: ba
Before
:::

::: ba-container
![Move Field -
Before](../images/refactoring/diagrams/Move%20Field%20-%20Before2fe2.png?id=b58c81b01a0c4ef8659f92cc64fa51a8){width="134"}
:::
:::::

::::: after
::: ba
After
:::

::: ba-container
![Move Field -
After](../images/refactoring/diagrams/Move%20Field%20-%20After4c93.png?id=d7c21af94ec9df17575373bae745e96e){width="134"}
:::
:::::
:::::::::
::::::::::

### Why Refactor

Often fields are moved as part of the [Extract
Class](extract-class.html) technique. Deciding which class to leave the
field in can be tough. Here is our rule of thumb: **put a field in the
same place as the methods that use it** (or else where most of these
methods are).

This rule will help in other cases when a field is simply located in the
wrong place.

### How to Refactor

1.  If the field is public, refactoring will be much easier if you make
    the field private and provide public access methods (for this, you
    can use [Encapsulate Field](encapsulate-field.html)).

2.  Create the same field with access methods in the recipient class.

3.  Decide how you will refer to the recipient class. You may already
    have a field or method that returns the appropriate object; if not,
    you will need to write a new method or field to store the object of
    the recipient class.

4.  Replace all references to the old field with appropriate calls to
    methods in the recipient class. If the field isn\'t private, take
    care of this in the superclass and subclasses.

5.  Delete the field in the original class.

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

:::::::::::::::::: row
::::: {.col-xs-12 .col-sm-6 .col-lg-4}
### Similar refactorings

:::: dl
::: dt
[Move Method](move-method.html)
:::
::::
:::::

::::::: {.col-xs-12 .col-sm-6 .col-lg-4}
### Helps other refactorings

:::: dl
::: dt
[Extract Class](extract-class.html)
:::
::::

:::: dl
::: dt
[Inline Class](inline-class.html)
:::
::::
:::::::

::::::::: {.col-xs-12 .col-sm-6 .col-lg-4}
### Eliminates smell

:::: dl
::: dt
[Shotgun Surgery](smells/shotgun-surgery.html)
:::
::::

:::: dl
::: dt
[Parallel Inheritance
Hierarchies](smells/parallel-inheritance-hierarchies.html)
:::
::::

:::: dl
::: dt
[Inappropriate Intimacy](smells/inappropriate-intimacy.html)
:::
::::
:::::::::
::::::::::::::::::

::: next
#### 次を読む

[Extract Class []{.fa .fa-arrow-right}](extract-class.html){.btn
.btn-primary rel="next"}
:::

::: prev
#### 戻る

[[]{.fa .fa-arrow-left} Move Method](move-method.html){.btn .btn-default
rel="prev"}
:::
::::::::::::::::::::::::::::::::::::

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
:::::::::::::::::::::::::::::::::::::::::
::::::::::::::::::::::::::::::::::::::::::
