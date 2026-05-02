::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::::::::::::: {.refactoring .page .text}
::: breadcrumb
[](index.html){.home} / [Refactoring](refactoring.html){.type} /
[Techniques](refactoring/techniques.html){.type} / [Dealing with
Generalization](refactoring/techniques/dealing-with-generalization.html){.type}
:::

# Pull Up Method {#pull-up-method .title}

::::: before-after-container
::: before
### Problem

Your subclasses have methods that perform similar work.
:::

::: after
### Solution

Make the methods identical and then move them to the relevant
superclass.
:::
:::::

:::::::::: examples
::::::::: {.before-after .before-after-container}
::::: before
::: ba
Before
:::

::: ba-container
![Pull Up Method -
Before](images/refactoring/diagrams/Pull%20Up%20Method%20-%20Beforee533.png?id=1af4e8b520779e6b72d6d91115f6373f){width="452"}
:::
:::::

::::: after
::: ba
After
:::

::: ba-container
![Pull Up Method -
After](images/refactoring/diagrams/Pull%20Up%20Method%20-%20After9072.png?id=57193e5593115571e969abe471dfd9aa){width="452"}
:::
:::::
:::::::::
::::::::::

### Why Refactor

Subclasses grew and developed independently of one another, causing
identical (or nearly identical) fields and methods.

### Benefits

-   Gets rid of duplicate code. If you need to make changes to a method,
    it's better to do so in a single place than have to search for all
    duplicates of the method in subclasses.

-   This refactoring technique can also be used if, for some reason, a
    subclass redefines a superclass method but performs what's
    essentially the same work.

### How to Refactor

1.  Investigate similar methods in superclasses. If they aren't
    identical, format them to match each other.

2.  If methods use a different set of parameters, put the parameters in
    the form that you want to see in the superclass.

3.  Copy the method to the superclass. Here you may find that the method
    code uses fields and methods that exist only in subclasses and
    therefore aren't available in the superclass. To solve this, you
    can:

    -   For fields: use either [Pull Up Field](pull-up-field.html) or
        Self-[Encapsulate Field](encapsulate-field.html) to create
        getters and setters in subclasses; then declare these getters
        abstractly in the superclass.

    -   For methods: use either [Pull Up Method](pull-up-method.html) or
        declare abstract methods for them in the superclass (note that
        your class will become abstract if it wasn't previously).

4.  Remove the methods from the subclasses.

5.  Check the locations in which the method is called. In some places
    you may be able to replace use of a subclass with the superclass.

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

::::::::::::::: row
::::: {.col-xs-12 .col-sm-6 .col-lg-3}
### Anti-refactoring

:::: dl
::: dt
[Push Down Method](push-down-method.html)
:::
::::
:::::

::::: {.col-xs-12 .col-sm-6 .col-lg-3}
### Similar refactorings

:::: dl
::: dt
[Pull Up Field](pull-up-field.html)
:::
::::
:::::

::::: {.col-xs-12 .col-sm-6 .col-lg-3}
### Helps other refactorings

:::: dl
::: dt
[Form Template Method](form-template-method.html)
:::
::::
:::::

::::: {.col-xs-12 .col-sm-6 .col-lg-3}
### Eliminates smell

:::: dl
::: dt
[Duplicate Code](smells/duplicate-code.html)
:::
::::
:::::
:::::::::::::::

::: next
#### Read next

[Pull Up Constructor Body []{.fa
.fa-arrow-right}](pull-up-constructor-body.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Return

[[]{.fa .fa-arrow-left} Pull Up Field](pull-up-field.html){.btn
.btn-default rel="prev"}
:::
:::::::::::::::::::::::::::::::::

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
::::::::::::::::::::::::::::::::::::::
:::::::::::::::::::::::::::::::::::::::
