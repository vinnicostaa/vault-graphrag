:::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::::::::: {.refactoring .page .text}
::: breadcrumb
[](../index.html){.home} / [Refactoring](refactoring.html){.type} /
[Techniques](refactoring/techniques.html){.type} / [Simplifying Method
Calls](refactoring/techniques/simplifying-method-calls.html){.type}
:::

# Add Parameter {#add-parameter .title}

::::: before-after-container
::: before
### Problem

A method doesn\'t have enough data to perform certain actions.
:::

::: after
### Solution

Create a new parameter to pass the necessary data.
:::
:::::

:::::::::: examples
::::::::: {.before-after .before-after-container}
::::: before
::: ba
Before
:::

::: ba-container
![Add Parameter -
Before](../images/refactoring/diagrams/Add%20Parameter%20-%20Before41d0.png?id=a060d5ffb8661cee995f02ead0d8547b){width="171"}
:::
:::::

::::: after
::: ba
After
:::

::: ba-container
![Add Parameter -
After](../images/refactoring/diagrams/Add%20Parameter%20-%20After1916.png?id=c49cf53fd9ebf39daa1ec27bb897a5cf){width="171"}
:::
:::::
:::::::::
::::::::::

### Why Refactor

You need to make changes to a method and these changes require adding
information or data that was previously not available to the method.

### Benefits

-   The choice here is between adding a new parameter and adding a new
    private field that contains the data needed by the method. A
    parameter is preferable when you need some occasional or frequently
    changing data for which there\'s no point in holding it in an object
    all of the time. In this case, the refactoring will pay off.
    Otherwise, add a private field and fill it with the necessary data
    before calling the method.

### Drawbacks

-   Adding a new parameter is always easier than removing it, which is
    why parameter lists frequently balloon to grotesque sizes. This
    smell is known as the [Long Parameter
    List](smells/long-parameter-list.html).

-   If you need to add a new parameter, sometimes this means that your
    class doesn\'t contain the necessary data or the existing parameters
    don\'t contain the necessary related data. In both cases, the best
    solution is to consider moving data to the main class or to other
    classes whose objects are already accessible from inside the method.

### How to Refactor

1.  See whether the method is defined in a superclass or subclass. If
    the method is present in them, you will need to repeat all the steps
    in these classes as well.

2.  The following step is critical for keeping your program functional
    during the refactoring process. Create a new method by copying the
    old one and add the necessary parameter to it. Replace the code for
    the old method with a call to the new method. You can plug in any
    value to the new parameter (such as `null` for objects or a zero for
    numbers).

3.  Find all references to the old method and replace them with
    references to the new method.

4.  Delete the old method. Deletion isn\'t possible if the old method is
    part of the public interface. If that\'s the case, mark the old
    method as deprecated.

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

:::::::::::: row
::::: {.col-xs-12 .col-sm-6 .col-lg-4}
### Anti-refactoring

:::: dl
::: dt
[Remove Parameter](remove-parameter.html)
:::
::::
:::::

::::: {.col-xs-12 .col-sm-6 .col-lg-4}
### Similar refactorings

:::: dl
::: dt
[Rename Method](rename-method.html)
:::
::::
:::::

::::: {.col-xs-12 .col-sm-6 .col-lg-4}
### Helps other refactorings

:::: dl
::: dt
[Introduce Parameter Object](introduce-parameter-object.html)
:::
::::
:::::
::::::::::::

::: next
#### 次を読む

[Remove Parameter []{.fa .fa-arrow-right}](remove-parameter.html){.btn
.btn-primary rel="next"}
:::

::: prev
#### 戻る

[[]{.fa .fa-arrow-left} Rename Method](rename-method.html){.btn
.btn-default rel="prev"}
:::
::::::::::::::::::::::::::::::

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
:::::::::::::::::::::::::::::::::::
::::::::::::::::::::::::::::::::::::
