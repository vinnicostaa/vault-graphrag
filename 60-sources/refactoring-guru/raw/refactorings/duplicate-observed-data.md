::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::::::: {.refactoring .page .text}
::: breadcrumb
[](index.html){.home} / [Refactoring](refactoring.html){.type} /
[Techniques](refactoring/techniques.html){.type} / [Organizing
Data](refactoring/techniques/organizing-data.html){.type}
:::

# Duplicate Observed Data {#duplicate-observed-data .title}

::::: before-after-container
::: before
### Problem

Is domain data stored in classes responsible for the GUI?
:::

::: after
### Solution

Then it's a good idea to separate the data into separate classes,
ensuring connection and synchronization between the domain class and the
GUI.
:::
:::::

:::::::::: examples
::::::::: {.before-after .before-after-container}
::::: before
::: ba
Before
:::

::: ba-container
![Duplicate Observed Data -
Before](images/refactoring/diagrams/Duplicate%20Observed%20Data%20-%20Befored907.png?id=bd26ed6d7d9921504165fd46b7f6124c){width="227"}
:::
:::::

::::: after
::: ba
After
:::

::: ba-container
![Duplicate Observed Data -
After](images/refactoring/diagrams/Duplicate%20Observed%20Data%20-%20After5bfb.png?id=b86fa0ff2f9ff7c76d4a183607153458){width="227"}
:::
:::::
:::::::::
::::::::::

### Why Refactor

You want to have multiple interface views for the same data (for
example, you have both a desktop app and a mobile app). If you fail to
separate the GUI from the domain, you will have a very hard time
avoiding code duplication and a large number of mistakes.

### Benefits

-   You split responsibility between business logic classes and
    presentation classes (cf. the *Single Responsibility Principle*),
    which makes your program more readable and understandable.

-   If you need to add a new interface view, create new presentation
    classes; you don't need to touch the code of the business logic (cf.
    the *Open/Closed Principle*).

-   Now different people can work on the business logic and the user
    interfaces.

### When Not to Use

-   This refactoring technique, which in its classic form is performed
    using the [Observer](design-patterns/observer.html) template, isn't
    applicable for web apps, where all classes are recreated between
    queries to the web server.

-   All the same, the general principle of extracting business logic
    into separate classes can be justified for web apps as well. But
    this will be implemented using different refactoring techniques
    depending on how your system is designed.

### How to Refactor

1.  Hide direct access to domain data in the *GUI class*. For this, it's
    best to use [Self Encapsulate Field](self-encapsulate-field.html).
    So you create the getters and setters for this data.

2.  In handlers for *GUI class* events, use setters to set new field
    values. This will let you pass these values to the associated
    *domain object*.

3.  Create a domain class and copy necessary fields from the *GUI class*
    to it. Create getters and seters for all these fields.

4.  Create an Observer pattern for these two classes:

    -   In the *domain class*, create an array for storing observer
        objects (*GUI objects*), as well as methods for registering,
        deleting and notifying them.

    -   In the *GUI class*, create a field for storing references to the
        *domain class* as well as the `update()` method, which will be
        reacting to changes in the object and update the values of
        fields in the *GUI class*. Note that value updates should be
        established directly in the method, in order to avoid recursion.

    -   In the *GUI class* constructor, create an instance of *domain
        class* and save it in the field you have created. Register the
        *GUI object* as an observer in the *domain object*.

    -   In the setters for *domain class* fields, call the method for
        notifying the observer (in other words, method for updating in
        the *GUI class*), in order to pass the new values to the GUI.

    -   Change the setters of the *GUI class* fields so that they set
        new values in the domain object directly. Watch out to make sure
        that values aren't set through a *domain class*
        setter---otherwise infinite recursion will result.

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

::::::::: row
::::: {.col-xs-12 .col-sm-6 .col-lg-6}
### Implements design pattern

:::: dl
::: dt
[Observer](design-patterns/observer.html)
:::
::::
:::::

::::: {.col-xs-12 .col-sm-6 .col-lg-6}
### Eliminates smell

:::: dl
::: dt
[Large Class](smells/large-class.html)
:::
::::
:::::
:::::::::

::: next
#### Read next

[Change Unidirectional Association to Bidirectional []{.fa
.fa-arrow-right}](change-unidirectional-association-to-bidirectional.html){.btn
.btn-primary rel="next"}
:::

::: prev
#### Return

[[]{.fa .fa-arrow-left} Replace Array with
Object](replace-array-with-object.html){.btn .btn-default rel="prev"}
:::
:::::::::::::::::::::::::::

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
::::::::::::::::::::::::::::::::
:::::::::::::::::::::::::::::::::
