:::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::: {.refactoring .page .text}
::: breadcrumb
[](../index.html){.home} / [Refactoring](refactoring.html){.type} /
[Techniques](refactoring/techniques.html){.type} / [Organizing
Data](refactoring/techniques/organizing-data.html){.type}
:::

# Replace Subclass with Fields {#replace-subclass-with-fields .title}

::::: before-after-container
::: before
### Problem

You have subclasses differing only in their (constant-returning)
methods.
:::

::: after
### Solution

Replace the methods with fields in the parent class and delete the
subclasses.
:::
:::::

:::::::::: examples
::::::::: {.before-after .before-after-container}
::::: before
::: ba
Before
:::

::: ba-container
![Replace Subclass with Fields -
Before](../images/refactoring/diagrams/Replace%20Subclass%20with%20Fields%20-%20Beforec050.png?id=ea6525cc6b55e1a03fdb35def943c675){width="415"}
:::
:::::

::::: after
::: ba
After
:::

::: ba-container
![Replace Subclass with Fields -
After](../images/refactoring/diagrams/Replace%20Subclass%20with%20Fields%20-%20After22d1.png?id=bd1b29687aa333b3adbeb2bfb3e78614){width="152"}
:::
:::::
:::::::::
::::::::::

### Why Refactor

Sometimes refactoring is just the ticket for avoiding type code.

In one such case, a hierarchy of subclasses may be different only in the
values returned by particular methods. These methods aren\'t even the
result of computation, but are strictly set out in the methods
themselves or in the fields returned by the methods. To simplify the
class architecture, this hierarchy can be compressed into a single class
containing one or several fields with the necessary values, based on the
situation.

These changes may become necessary after moving a large amount of
functionality from a class hierarchy to another place. The current
hierarchy is no longer so valuable and its subclasses are now just dead
weight.

### Benefits

-   Simplifies system architecture. Creating subclasses is overkill if
    all you want to do is to return different values in different
    methods.

### How to Refactor

1.  Apply [Replace Constructor with Factory
    Method](replace-constructor-with-factory-method.html) to the
    subclasses.

2.  Replace subclass constructor calls with superclass factory method
    calls.

3.  In the superclass, declare fields for storing the values of each of
    the subclass methods that return constant values.

4.  Create a protected superclass constructor for initializing the new
    fields.

5.  Create or modify the existing subclass constructors so that they
    call the new constructor of the parent class and pass the relevant
    values to it.

6.  Implement each constant method in the parent class so that it
    returns the value of the corresponding field. Then remove the method
    from the subclass.

7.  If the subclass constructor has additional functionality, use
    [Inline Method](inline-method.html) to incorporate the constructor
    into the superclass factory method.

8.  Delete the subclass.

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
[Replace Type Code with
Subclasses](replace-type-code-with-subclasses.html)
:::
::::
:::::
::::::

::: next
#### 다음을 보세요

[Simplifying Conditional Expressions []{.fa
.fa-arrow-right}](refactoring/techniques/simplifying-conditional-expressions.html){.btn
.btn-primary rel="next"}
:::

::: prev
#### 돌아가기

[[]{.fa .fa-arrow-left} Replace Type Code with
State/Strategy](replace-type-code-with-state-strategy.html){.btn
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
