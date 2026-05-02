:::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::: {.refactoring .page .text}
::: breadcrumb
[](../index.html){.home} / [Refactoring](refactoring.html){.type} /
[Techniques](refactoring/techniques.html){.type} / [Organizing
Data](refactoring/techniques/organizing-data.html){.type}
:::

# Change Unidirectional Association to Bidirectional {#change-unidirectional-association-to-bidirectional .title}

::::: before-after-container
::: before
### Problem

You have two classes that each need to use the features of the other,
but the association between them is only unidirectional.
:::

::: after
### Solution

Add the missing association to the class that needs it.
:::
:::::

:::::::::: examples
::::::::: {.before-after .before-after-container}
::::: before
::: ba
Before
:::

::: ba-container
![Change Unidirectional Association to Bidirectional -
Before](../images/refactoring/diagrams/Change%20Unidirectional%20Association%20to%20Bidirectional%20-%20Before914d.png?id=a31d8472bbb2be95836c8b5e2334ac77){width="396"}
:::
:::::

::::: after
::: ba
After
:::

::: ba-container
![Change Unidirectional Association to Bidirectional -
After](../images/refactoring/diagrams/Change%20Unidirectional%20Association%20to%20Bidirectional%20-%20After01d6.png?id=058ba44a44610bcb9caa4c23f60ad0bc){width="396"}
:::
:::::
:::::::::
::::::::::

### Why Refactor

Originally the classes had a unidirectional association. But with time,
client code needed access to both sides of the association.

### Benefits

-   If a class needs a reverse association, you can simply calculate it.
    But if these calculations are complex, it\'s better to keep the
    reverse association.

### Drawbacks

-   Bidirectional associations are much harder to implement and maintain
    than unidirectional ones.

-   Bidirectional associations make classes interdependent. With a
    unidirectional association, one of them can be used independently of
    the other.

### How to Refactor

1.  Add a field for holding the reverse association.

2.  Decide which class will be \"dominant\". This class will contain the
    methods that create or update the association as elements are added
    or changed, establishing the association in its class and calling
    the utility methods for establishing the association in the
    associated object.

3.  Create a utility method for establishing the association in the
    \"non-dominant\" class. The method should use what it\'s given in
    parameters to complete the field. Give the method an obvious name so
    that it isn\'t used later for any other purposes.

4.  If old methods for controlling the unidirectional association were
    in the \"dominant\" class, complement them with calls to utility
    methods from the associated object.

5.  If the old methods for controlling the association were in the
    \"non-dominant\" class, create the methods in the \"dominant\"
    class, call them, and delegate execution to them.

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
[Change Bidirectional Association to
Unidirectional](change-bidirectional-association-to-unidirectional.html)
:::
::::
:::::
::::::

::: next
#### 다음을 보세요

[Change Bidirectional Association to Unidirectional []{.fa
.fa-arrow-right}](change-bidirectional-association-to-unidirectional.html){.btn
.btn-primary rel="next"}
:::

::: prev
#### 돌아가기

[[]{.fa .fa-arrow-left} Duplicate Observed
Data](duplicate-observed-data.html){.btn .btn-default rel="prev"}
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
