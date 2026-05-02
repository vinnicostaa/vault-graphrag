:::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::: {.refactoring .page .text}
::: breadcrumb
[](../index.html){.home} / [Refactoring](refactoring.html){.type} /
[Techniques](refactoring/techniques.html){.type} / [Simplifying Method
Calls](refactoring/techniques/simplifying-method-calls.html){.type}
:::

# Separate Query from Modifier {#separate-query-from-modifier .title}

::::: before-after-container
::: before
### Problem

Do you have a method that returns a value but also changes something
inside an object?
:::

::: after
### Solution

Split the method into two separate methods. As you would expect, one of
them should return the value and the other one modifies the object.
:::
:::::

:::::::::: examples
::::::::: {.before-after .before-after-container}
::::: before
::: ba
Before
:::

::: ba-container
![Separate Query from Modifier -
Before](../images/refactoring/diagrams/Separate%20Query%20from%20Modifier%20-%20Before109c.png?id=b28976c01f67d02e8b4574b1c825d10d){width="415"}
:::
:::::

::::: after
::: ba
After
:::

::: ba-container
![Separate Query from Modifier -
After](../images/refactoring/diagrams/Separate%20Query%20from%20Modifier%20-%20After3356.png?id=c2b8e0722c1c8500302326ab60db80e5){width="227"}
:::
:::::
:::::::::
::::::::::

### Why Refactor

This factoring technique implements *Command and Query Responsibility
Segregation*. This principle tells us to separate code responsible for
getting data from code that changes something inside an object.

Code for getting data is named a *query*. Code for changing things in
the *visible state* of an object is named a *modifier*. When a *query*
and *modifier* are combined, you don\'t have a way to get data without
making changes to its condition. In other words, you ask a question and
can change the answer even as it\'s being received. This problem becomes
even more severe when the person calling the query may not know about
the method\'s \"side effects\", which often leads to runtime errors.

But remember that side effects are dangerous only in the case of
*modifiers* that change the **visible** state of an object. These could
be, for example, fields accessible from an object\'s public interface,
entry in a database, in files, etc. If a *modifier* only caches a
complex operation and saves it within the private field of a class, it
can hardly cause any side effects.

### Benefits

-   If you have a *query* that doesn\'t change the state of your
    program, you can call it as many times as you like without having to
    worry about unintended changes in the result caused by the mere fact
    of you calling the method.

### Drawbacks

-   In some cases it\'s convenient to get data after performing a
    command. For example, when deleting something from a database you
    want to know how many rows were deleted.

### How to Refactor

1.  Create a new *query method* to return what the original method did.

2.  Change the original method so that it returns only the result of
    calling the new *query method*.

3.  Replace all references to the original method with a call to the
    *query method*. Immediately before this line, place a call to the
    *modifier method*. This will save you from side effects in case if
    the original method was used in a condition of a conditional
    operator or loop.

4.  Get rid of the value-returning code in the original method, which
    now has become a proper *modifier method*.

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
[Replace Temp with Query](replace-temp-with-query.html)
:::
::::
:::::
::::::

::: next
#### 다음을 보세요

[Parameterize Method []{.fa
.fa-arrow-right}](parameterize-method.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### 돌아가기

[[]{.fa .fa-arrow-left} Remove Parameter](remove-parameter.html){.btn
.btn-default rel="prev"}
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
