::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::::::::::::: {.refactoring .page .text}
::: breadcrumb
[](../index.html){.home} / [Refactoring](refactoring.html){.type} /
[Techniques](refactoring/techniques.html){.type} / [Dealing with
Generalization](refactoring/techniques/dealing-with-generalization.html){.type}
:::

# Push Down Method {#push-down-method .title}

::::: before-after-container
::: before
### Problem

Is behavior implemented in a superclass used by only one (or a few)
subclasses?
:::

::: after
### Solution

Move this behavior to the subclasses.
:::
:::::

:::::::::: examples
::::::::: {.before-after .before-after-container}
::::: before
::: ba
Before
:::

::: ba-container
![Push Down Method -
Before](../images/refactoring/diagrams/Push%20Down%20Method%20-%20Beforea9e3.png?id=ed31b8dcbc3c98fc668ce95cff9f76a8){width="452"}
:::
:::::

::::: after
::: ba
After
:::

::: ba-container
![Push Down Method -
After](../images/refactoring/diagrams/Push%20Down%20Method%20-%20Aftere373.png?id=dcf9788fec0c13fb8e60fe494d481529){width="452"}
:::
:::::
:::::::::
::::::::::

### Why Refactor

At first a certain method was meant to be universal for all classes but
in reality is used in only one subclass. This situation can occur when
planned features fail to materialize.

Such situations can also occur after partial extraction (or removal) of
functionality from a class hierarchy, leaving a method that's used in
only one subclass.

If you see that a method is needed by more than one subclass, but not
all of them, it may be useful to create an intermediate subclass and
move the method to it. This allows avoiding the code duplication that
would result from pushing a method down to all subclasses.

### Benefits

-   Improves class coherence. A method is located where you expect to
    see it.

### How to Refactor

1.  Declare the method in a subclass and copy its code from the
    superclass.

2.  Remove the method from the superclass.

3.  Find all places where the method is used and verify that it's called
    from the necessary subclass.

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

::::::::::::::: row
::::: {.col-xs-12 .col-sm-6 .col-lg-3}
### Anti-refactoring

:::: dl
::: dt
[Pull Up Method](pull-up-method.html)
:::
::::
:::::

::::: {.col-xs-12 .col-sm-6 .col-lg-3}
### Similar refactorings

:::: dl
::: dt
[Push Down Field](push-down-field.html)
:::
::::
:::::

::::: {.col-xs-12 .col-sm-6 .col-lg-3}
### Helps other refactorings

:::: dl
::: dt
[Extract Subclass](extract-subclass.html)
:::
::::
:::::

::::: {.col-xs-12 .col-sm-6 .col-lg-3}
### Eliminates smell

:::: dl
::: dt
[Refused Bequest](smells/refused-bequest.html)
:::
::::
:::::
:::::::::::::::

::: next
#### Suivant

[Push Down Field []{.fa .fa-arrow-right}](push-down-field.html){.btn
.btn-primary rel="next"}
:::

::: prev
#### Retour

[[]{.fa .fa-arrow-left} Pull Up Constructor
Body](pull-up-constructor-body.html){.btn .btn-default rel="prev"}
:::
:::::::::::::::::::::::::::::::::

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
::::::::::::::::::::::::::::::::::::::
:::::::::::::::::::::::::::::::::::::::
