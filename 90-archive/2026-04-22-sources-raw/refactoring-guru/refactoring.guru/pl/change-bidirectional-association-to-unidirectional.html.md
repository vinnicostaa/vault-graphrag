::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::::::: {.refactoring .page .text}
::: breadcrumb
[](../index.html){.home} / [Refactoring](refactoring.html){.type} /
[Techniques](refactoring/techniques.html){.type} / [Organizing
Data](refactoring/techniques/organizing-data.html){.type}
:::

# Change Bidirectional Association to Unidirectional {#change-bidirectional-association-to-unidirectional .title}

::::: before-after-container
::: before
### Problem

You have a bidirectional association between classes, but one of the
classes doesn't use the other's features.
:::

::: after
### Solution

Remove the unused association.
:::
:::::

:::::::::: examples
::::::::: {.before-after .before-after-container}
::::: before
::: ba
Before
:::

::: ba-container
![Change Bidirectional Association to Unidirectional -
Before](../images/refactoring/diagrams/Change%20Bidirectional%20Association%20to%20Unidirectional%20-%20Before4c70.png?id=9fc2b8d6c54255295d2ef62d9e015c0a){width="396"}
:::
:::::

::::: after
::: ba
After
:::

::: ba-container
![Change Bidirectional Association to Unidirectional -
After](../images/refactoring/diagrams/Change%20Bidirectional%20Association%20to%20Unidirectional%20-%20After3a90.png?id=0340aa25a071d7529343ebe3dd469ffd){width="396"}
:::
:::::
:::::::::
::::::::::

### Why Refactor

A bidirectional association is generally harder to maintain than a
unidirectional one, requiring additional code for properly creating and
deleting the relevant objects. This makes the program more complicated.

In addition, an improperly implemented bidirectional association can
cause problems for garbage collection (in turn leading to memory
bloat by unused objects).

Example: the garbage collector removes objects from memory that are no
longer referenced by other objects. Let's say that an object pair
`User`-`Order` was created, used, and then abandoned. But these objects
won't be cleared from memory since they still refer to each other. That
said, this problem is becoming less important thanks to advances in
programming languages, which now automatically identify unused object
references and remove them from memory.

There's also the problem of interdependency between classes. In a
bidirectional association, the two classes must know about each other,
meaning that they can't be used separately. If many of these
associations are present, different parts of the program become too
dependent on each other and any changes in one component may affect the
other components.

### Benefits

-   Simplifies the class that doesn't need the relationship. Less code
    equals less code maintenance.

-   Reduces dependency between classes. Independent classes are
    easier to maintain since any changes to a class affect only that
    class.

### How to Refactor

1.  Make sure that one of the following is true for your classes:

    -   No association is used.

    -   There's another way to get the associated object, such through a
        database query.

    -   The associated object can be passed as an argument to the
        methods that use it.

2.  Depending on your situation, use of a field that contains an
    association with another object should be replaced by a parameter or
    method call for getting the object in a different way.

3.  Delete the code that assigns the associated object to the field.

4.  Delete the now-unused field.

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

::::::::: row
::::: {.col-xs-12 .col-sm-6 .col-lg-6}
### Anti-refactoring

:::: dl
::: dt
[Change Unidirectional Association to
Bidirectional](change-unidirectional-association-to-bidirectional.html)
:::
::::
:::::

::::: {.col-xs-12 .col-sm-6 .col-lg-6}
### Eliminates smell

:::: dl
::: dt
[Inappropriate Intimacy](smells/inappropriate-intimacy.html)
:::
::::
:::::
:::::::::

::: next
#### Czytaj dalej

[Replace Magic Number with Symbolic Constant []{.fa
.fa-arrow-right}](replace-magic-number-with-symbolic-constant.html){.btn
.btn-primary rel="next"}
:::

::: prev
#### Wróć

[[]{.fa .fa-arrow-left} Change Unidirectional Association to
Bidirectional](change-unidirectional-association-to-bidirectional.html){.btn
.btn-default rel="prev"}
:::
:::::::::::::::::::::::::::

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
::::::::::::::::::::::::::::::::
:::::::::::::::::::::::::::::::::
