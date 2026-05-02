:::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::: {.refactoring .page .text}
::: breadcrumb
[](../index.html){.home} / [Refactoring](refactoring.html){.type} /
[Techniques](refactoring/techniques.html){.type} / [Simplifying Method
Calls](refactoring/techniques/simplifying-method-calls.html){.type}
:::

# Remove Setting Method {#remove-setting-method .title}

::::: before-after-container
::: before
### Problem

The value of a field should be set only when it\'s created, and not
change at any time after that.
:::

::: after
### Solution

So remove methods that set the field\'s value.
:::
:::::

:::::::::: examples
::::::::: {.before-after .before-after-container}
::::: before
::: ba
Before
:::

::: ba-container
![Remove Setting Method -
Before](../images/refactoring/diagrams/Remove%20Setting%20Method%20-%20Before0679.png?id=6f5c0d77289a8f59a2bb89032d5bc55f){width="190"}
:::
:::::

::::: after
::: ba
After
:::

::: ba-container
![Remove Setting Method -
After](../images/refactoring/diagrams/Remove%20Setting%20Method%20-%20Aftercb50.png?id=5e75fe3df982c4da469ddceb96530058){width="190"}
:::
:::::
:::::::::
::::::::::

### Why Refactor

You want to prevent any changes to the value of a field.

### How to Refactor

1.  The value of a field should be changeable only in the constructor.
    If the constructor doesn\'t contain a parameter for setting the
    value, add one.

2.  Find all setter calls.

    -   If a setter call is located right after a call for the
        constructor of the current class, move its argument to the
        constructor call and remove the setter.

    -   Replace setter calls in the constructor with direct access to
        the field.

3.  Delete the setter.

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
### Helps other refactorings

:::: dl
::: dt
[Change Reference to Value](change-reference-to-value.html)
:::
::::
:::::
::::::

::: next
#### 다음을 보세요

[Hide Method []{.fa .fa-arrow-right}](hide-method.html){.btn
.btn-primary rel="next"}
:::

::: prev
#### 돌아가기

[[]{.fa .fa-arrow-left} Introduce Parameter
Object](introduce-parameter-object.html){.btn .btn-default rel="prev"}
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
