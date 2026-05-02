:::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::: {.refactoring .page .text}
::: breadcrumb
[](../index.html){.home} / [Refactoring](refactoring.html){.type} /
[Techniques](refactoring/techniques.html){.type} / [Dealing with
Generalization](refactoring/techniques/dealing-with-generalization.html){.type}
:::

# Extract Interface {#extract-interface .title}

::::: before-after-container
::: before
### Problem

Multiple clients are using the same part of a class interface. Another
case: part of the interface in two classes is the same.
:::

::: after
### Solution

Move this identical portion to its own interface.
:::
:::::

:::::::::: examples
::::::::: {.before-after .before-after-container}
::::: before
::: ba
Before
:::

::: ba-container
![Extract Interface -
Before](../images/refactoring/diagrams/Extract%20Interface%20-%20Before543f.png?id=fc2bba34908c16e2b86c5c9cc8c424b1){width="209"}
:::
:::::

::::: after
::: ba
After
:::

::: ba-container
![Extract Interface -
After](../images/refactoring/diagrams/Extract%20Interface%20-%20After2e06.png?id=2692cc5b979184941c55080bc89226b4){width="209"}
:::
:::::
:::::::::
::::::::::

### Why Refactor

1.  Interfaces are very apropos when classes play special roles in
    different situations. Use [Extract
    Interface](extract-interface.html) to explicitly indicate which
    role.

2.  Another convenient case arises when you need to describe the
    operations that a class performs on its server. If it's planned to
    eventually allow use of servers of multiple types, all servers must
    implement the interface.

### Good to Know

There's a certain resemblance between [Extract
Superclass](extract-superclass.html) and [Extract
Interface](extract-interface.html).

Extracting an interface allows isolating only common interfaces, not
common code. In other words, if classes contain [Duplicate
Code](smells/duplicate-code.html), extracting the interface won't help
you to deduplicate.

All the same, this problem can be mitigated by applying [Extract
Class](extract-class.html) to move the behavior that contains the
duplication to a separate component and delegating all the work to it.
If the common behavior is large in size, you can always use [Extract
Superclass](extract-superclass.html). This is even easier, of course,
but remember that if you take this path you will get only one parent
class.

### How to Refactor

1.  Create an empty interface.

2.  Declare common operations in the interface.

3.  Declare the necessary classes as implementing the interface.

4.  Change type declarations in the client code to use the new
    interface.

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
### Similar refactorings

:::: dl
::: dt
[Extract Superclass](extract-superclass.html)
:::
::::
:::::
::::::

::: next
#### Leia a seguir

[Collapse Hierarchy []{.fa
.fa-arrow-right}](collapse-hierarchy.html){.btn .btn-primary rel="next"}
:::

::: prev
#### Voltar

[[]{.fa .fa-arrow-left} Extract
Superclass](extract-superclass.html){.btn .btn-default rel="prev"}
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
