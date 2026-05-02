::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::::::::: {.refactoring .page .text}
::: breadcrumb
[](index.html){.home} / [Refactoring](refactoring.html){.type} /
[Techniques](refactoring/techniques.html){.type} / [Organizing
Data](refactoring/techniques/organizing-data.html){.type}
:::

# Replace Type Code with Class {#replace-type-code-with-class .title}

> **What's type code?** Type code occurs when, instead of a separate
> data type, you have a set of numbers or strings that form a list of
> allowable values for some entity. Often these specific numbers and
> strings are given understandable names via constants, which is the
> reason for why such type code is encountered so much.

::::: before-after-container
::: before
### Problem

A class has a field that contains type code. The values of this type
aren't used in operator conditions and don't affect the behavior of the
program.
:::

::: after
### Solution

Create a new class and use its objects instead of the type code values.
:::
:::::

:::::::::: examples
::::::::: {.before-after .before-after-container}
::::: before
::: ba
Before
:::

::: ba-container
![Replace Type Code with Class -
Before](images/refactoring/diagrams/Replace%20Type%20Code%20with%20Class%20-%20Before7f24.png?id=1b492169f36a1b9da5bb6d9e6f857f45){width="152"}
:::
:::::

::::: after
::: ba
After
:::

::: ba-container
![Replace Type Code with Class -
After](images/refactoring/diagrams/Replace%20Type%20Code%20with%20Class%20-%20After8bf2.png?id=c3be25afb117027a6d56cc489d0e7ae6){width="152"}
:::
:::::
:::::::::
::::::::::

### Why Refactor

One of the most common reasons for type code is working with databases,
when a database has fields in which some complex concept is coded with a
number or string.

For example, you have the class `User` with the field `user_role`, which
contains information about the access privileges of each user, whether
administrator, editor, or ordinary user. So in this case, this
information is coded in the field as `A`, `E`, and `U` respectively.

What are the shortcomings of this approach? The field setters often
don't check which value is sent, which can cause big problems when
someone sends unintended or wrong values to these fields.

In addition, type verification is impossible for these fields. It's
possible to send any number or string to them, which won't be type
checked by your IDE and even allow your program to run (and crash
later).

### Benefits

-   We want to turn sets of primitive values---which is what coded types
    are---into full-fledged classes with all the benefits that
    object-oriented programming has to offer.

-   By replacing type code with classes, we allow type hinting for
    values passed to methods and fields at the level of the programming
    language.

    For example, while the compiler previously didn't see difference
    between your numeric constant and some arbitrary number when a value
    is passed to a method, now when data that doesn't fit the indicated
    type class is passed, you're warned of the error inside your IDE.

-   Thus we make it possible to move code to the classes of the type. If
    you needed to perform complex manipulations with type values
    throughout the whole program, now this code can "live" inside one or
    multiple type classes.

### When Not to Use

If the values of a coded type are used inside control flow structures
(`if`, `switch`, etc.) and control a class behavior, you should use one
of the two refactoring techniques for type code:

-   [Replace Type Code with
    Subclasses](replace-type-code-with-subclasses.html)

-   [Replace Type Code with
    State/Strategy](replace-type-code-with-state-strategy.html)

### How to Refactor

1.  Create a new class and give it a new name that corresponds to the
    purpose of the coded type. Here we'll call it *type class*.

2.  Copy the field containing type code to the *type class* and make it
    private. Then create a getter for the field. A value will be set for
    this field only from the constructor.

3.  For each value of the coded type, create a static method in *type
    class*. It'll be creating a new *type class* object corresponding to
    this value of the coded type.

4.  In the original class, replace the type of the coded field with
    *type class*. Create a new object of this type in the constructor as
    well as in the field setter. Change the field getter so that it
    calls the *type class* getter.

5.  Replace any mentions of values of the coded type with calls of the
    relevant *type class* static methods.

6.  Remove the coded type constants from the original class.

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

::::::::::: row
::::::: {.col-xs-12 .col-sm-6 .col-lg-6}
### Similar refactorings

:::: dl
::: dt
[Replace Type Code with
Subclasses](replace-type-code-with-subclasses.html)
:::
::::

:::: dl
::: dt
[Replace Type Code with
State/Strategy](replace-type-code-with-state-strategy.html)
:::
::::
:::::::

::::: {.col-xs-12 .col-sm-6 .col-lg-6}
### Eliminates smell

:::: dl
::: dt
[Primitive Obsession](smells/primitive-obsession.html)
:::
::::
:::::
:::::::::::

::: next
#### Read next

[Replace Type Code with Subclasses []{.fa
.fa-arrow-right}](replace-type-code-with-subclasses.html){.btn
.btn-primary rel="next"}
:::

::: prev
#### Return

[[]{.fa .fa-arrow-left} Encapsulate
Collection](encapsulate-collection.html){.btn .btn-default rel="prev"}
:::
:::::::::::::::::::::::::::::

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
::::::::::::::::::::::::::::::::::
:::::::::::::::::::::::::::::::::::
