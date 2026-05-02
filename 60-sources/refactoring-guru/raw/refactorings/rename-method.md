::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::::::::::: {.refactoring .page .text}
::: breadcrumb
[](index.html){.home} / [Refactoring](refactoring.html){.type} /
[Techniques](refactoring/techniques.html){.type} / [Simplifying Method
Calls](refactoring/techniques/simplifying-method-calls.html){.type}
:::

# Rename Method {#rename-method .title}

::::: before-after-container
::: before
### Problem

The name of a method doesn't explain what the method does.
:::

::: after
### Solution

Rename the method.
:::
:::::

:::::::::: examples
::::::::: {.before-after .before-after-container}
::::: before
::: ba
Before
:::

::: ba-container
![Rename Method -
Before](images/refactoring/diagrams/Rename%20Method%20-%20Before175a.png?id=7943798ae9db6b5b232860eed6262462){width="171"}
:::
:::::

::::: after
::: ba
After
:::

::: ba-container
![Rename Method -
After](images/refactoring/diagrams/Rename%20Method%20-%20Aftere5be.png?id=62b4e6747951bbbacba3ede379fef200){width="171"}
:::
:::::
:::::::::
::::::::::

### Why Refactor

Perhaps a method was poorly named from the very beginning---for example,
someone created the method in a rush and didn't give proper care to
naming it well.

Or perhaps the method was well named at first but as its functionality
grew, the method name stopped being a good descriptor.

### Benefits

-   Code readability. Try to give the new method a name that reflects
    what it does. Something like `createOrder()`,
    `renderCustomerInfo()`, etc.

### How to Refactor

1.  See whether the method is defined in a superclass or subclass. If
    so, you must repeat all steps in these classes too.

2.  The next method is important for maintaining the functionality of
    the program during the refactoring process. Create a new method with
    a new name. Copy the code of the old method to it. Delete all the
    code in the old method and, instead of it, insert a call for the new
    method.

3.  Find all references to the old method and replace them with
    references to the new one.

4.  Delete the old method. If the old method is part of a public
    interface, don't perform this step. Instead, mark the old method as
    deprecated.

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

::::::::::::: row
::::::: {.col-xs-12 .col-sm-6 .col-lg-6}
### Similar refactorings

:::: dl
::: dt
[Add Parameter](add-parameter.html)
:::
::::

:::: dl
::: dt
[Remove Parameter](remove-parameter.html)
:::
::::
:::::::

::::::: {.col-xs-12 .col-sm-6 .col-lg-6}
### Eliminates smell

:::: dl
::: dt
[Alternative Classes with Different
Interfaces](smells/alternative-classes-with-different-interfaces.html)
:::
::::

:::: dl
::: dt
[Comments](smells/comments.html)
:::
::::
:::::::
:::::::::::::

::: next
#### Read next

[Add Parameter []{.fa .fa-arrow-right}](add-parameter.html){.btn
.btn-primary rel="next"}
:::

::: prev
#### Return

[[]{.fa .fa-arrow-left} Simplifying Method
Calls](refactoring/techniques/simplifying-method-calls.html){.btn
.btn-default rel="prev"}
:::
:::::::::::::::::::::::::::::::

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
::::::::::::::::::::::::::::::::::::
:::::::::::::::::::::::::::::::::::::
