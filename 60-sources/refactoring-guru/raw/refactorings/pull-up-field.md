:::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::::::::: {.refactoring .page .text}
::: breadcrumb
[](index.html){.home} / [Refactoring](refactoring.html){.type} /
[Techniques](refactoring/techniques.html){.type} / [Dealing with
Generalization](refactoring/techniques/dealing-with-generalization.html){.type}
:::

# Pull Up Field {#pull-up-field .title}

::::: before-after-container
::: before
### Problem

Two classes have the same field.
:::

::: after
### Solution

Remove the field from subclasses and move it to the superclass.
:::
:::::

:::::::::: examples
::::::::: {.before-after .before-after-container}
::::: before
::: ba
Before
:::

::: ba-container
![Pull Up Field -
Before](images/refactoring/diagrams/Pull%20Up%20Field%20-%20Before41a3.png?id=8174b96aa69ce1541271a14b97cc3e33){width="452"}
:::
:::::

::::: after
::: ba
After
:::

::: ba-container
![Pull Up Field -
After](images/refactoring/diagrams/Pull%20Up%20Field%20-%20After12cb.png?id=c97f930072aba9f00c03ba140797de19){width="452"}
:::
:::::
:::::::::
::::::::::

### Why Refactor

Subclasses grew and developed separately, causing identical (or nearly
identical) fields and methods to appear.

### Benefits

-   Eliminates duplication of fields in subclasses.

-   Eases subsequent relocation of duplicate methods, if they exist,
    from subclasses to a superclass.

### How to Refactor

1.  Make sure that the fields are used for the same needs in subclasses.

2.  If the fields have different names, give them the same name and
    replace all references to the fields in existing code.

3.  Create a field with the same name in the superclass. Note that if
    the fields were private, the superclass field should be protected.

4.  Remove the fields from the subclasses.

5.  You may want to consider using [Self Encapsulate
    Field](self-encapsulate-field.html) for the new field, in order to
    hide it behind access methods.

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

:::::::::::: row
::::: {.col-xs-12 .col-sm-6 .col-lg-4}
### Anti-refactoring

:::: dl
::: dt
[Push Down Field](push-down-field.html)
:::
::::
:::::

::::: {.col-xs-12 .col-sm-6 .col-lg-4}
### Similar refactorings

:::: dl
::: dt
[Pull Up Method](pull-up-method.html)
:::
::::
:::::

::::: {.col-xs-12 .col-sm-6 .col-lg-4}
### Eliminates smell

:::: dl
::: dt
[Duplicate Code](smells/duplicate-code.html)
:::
::::
:::::
::::::::::::

::: next
#### Read next

[Pull Up Method []{.fa .fa-arrow-right}](pull-up-method.html){.btn
.btn-primary rel="next"}
:::

::: prev
#### Return

[[]{.fa .fa-arrow-left} Dealing with
Generalization](refactoring/techniques/dealing-with-generalization.html){.btn
.btn-default rel="prev"}
:::
::::::::::::::::::::::::::::::

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
:::::::::::::::::::::::::::::::::::
::::::::::::::::::::::::::::::::::::
