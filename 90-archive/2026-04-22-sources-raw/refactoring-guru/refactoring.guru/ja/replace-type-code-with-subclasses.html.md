:::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::::::::::: {.refactoring .page .text}
::: breadcrumb
[](../index.html){.home} / [Refactoring](refactoring.html){.type} /
[Techniques](refactoring/techniques.html){.type} / [Organizing
Data](refactoring/techniques/organizing-data.html){.type}
:::

# Replace Type Code with Subclasses {#replace-type-code-with-subclasses .title}

> **What\'s type code?** Type code occurs when, instead of a separate
> data type, you have a set of numbers or strings that form a list of
> allowable values for some entity. Often these specific numbers and
> strings are given understandable names via constants, which is the
> reason for why such type code is encountered so much.

::::: before-after-container
::: before
### Problem

You have a coded type that directly affects program behavior (values of
this field trigger various code in conditionals).
:::

::: after
### Solution

Create subclasses for each value of the coded type. Then extract the
relevant behaviors from the original class to these subclasses. Replace
the control flow code with polymorphism.
:::
:::::

:::::::::: examples
::::::::: {.before-after .before-after-container}
::::: before
::: ba
Before
:::

::: ba-container
![Replace Type Code with Subclasses -
Before](../images/refactoring/diagrams/Replace%20Type%20Code%20with%20Subclasses%20-%20Before4bb7.png?id=5fa75a3c084933dcb4d1d1f0c60e56d3){width="152"}
:::
:::::

::::: after
::: ba
After
:::

::: ba-container
![Replace Type Code with Subclasses -
After](../images/refactoring/diagrams/Replace%20Type%20Code%20with%20Subclasses%20-%20After2670.png?id=9a9b00b277bd590e01589a441f50410e){width="302"}
:::
:::::
:::::::::
::::::::::

### Why Refactor

This refactoring technique is a more complicated twist on [Replace Type
Code with Class](replace-type-code-with-class.html).

As in the first refactoring method, you have a set of simple values that
constitute all the allowed values for a field. Although these values are
often specified as constants and have understandable names, their use
makes your code very error-prone since they\'re still primitives in
effect. For example, you have a method that accepts one of these values
in the parameters. At a certain moment, instead of the constant
`USER_TYPE_ADMIN` with the value `"ADMIN"`, the method receives the same
string in lower case (`"admin"`), which will cause execution of
something else that the author (you) didn\'t intend.

Here we\'re dealing with control flow code such as the conditionals
`if`, `switch` and `?:`. In other words, fields with coded values (such
as `$user->type === self::USER_TYPE_ADMIN`) are used inside the
conditions of these operators. If we were to use [Replace Type Code with
Class](replace-type-code-with-class.html) here, all these control flow
constructions would be best moved to a class responsible for the data
type. Ultimately, this would of course create a type class very similar
to the original one, with the same problems as well.

### Benefits

-   Delete the control flow code. Instead of a bulky `switch` in the
    original class, move the code to appropriate subclasses. This
    improves adherence to the *Single Responsibility Principle* and
    makes the program more readable in general.

-   If you need to add a new value for a coded type, all you need to do
    is add a new subclass without touching the existing code (cf. the
    *Open/Closed Principle*).

-   By replacing type code with classes, we pave the way for type
    hinting for methods and fields at the level of the programming
    language. This wouldn\'t be possible using simple numeric or string
    values contained in a coded type.

### When Not to Use

-   This technique isn\'t applicable if you already have a class
    hierarchy. You can\'t create a dual hierarchy via inheritance in
    object-oriented programming. Still, you can replace type code via
    composition instead of inheritance. To do so, use [Replace Type Code
    with State/Strategy](replace-type-code-with-state-strategy.html).

-   If the values of type code can change after an object is created,
    avoid this technique. We would have to somehow replace the class of
    the object itself on the fly, which isn\'t possible. Still, an
    alternative in this case too would be [Replace Type Code with
    State/Strategy](replace-type-code-with-state-strategy.html).

### How to Refactor

1.  Use [Self Encapsulate Field](self-encapsulate-field.html) to create
    a getter for the field that contains type code.

2.  Make the superclass constructor private. Create a static factory
    method with the same parameters as the superclass constructor. It
    must contain the parameter that will take the starting values of the
    coded type. Depending on this parameter, the factory method will
    create objects of various subclasses. To do so, in its code you must
    create a large conditional but, at least, it\'ll be the only one
    when it\'s truly necessary; otherwise, subclasses and polymorphism
    will do.

3.  Create a unique subclass for each value of the coded type. In it,
    redefine the getter of the coded type so that it returns the
    corresponding value of the coded type.

4.  Delete the field with type code from the superclass. Make its getter
    abstract.

5.  Now that you have subclasses, you can start to move the fields and
    methods from the superclass to corresponding subclasses (with the
    help of [Push Down Field](push-down-field.html) and [Push Down
    Method](push-down-method.html)).

6.  When everything possible has been moved, use [Replace Conditional
    with Polymorphism](replace-conditional-with-polymorphism.html) in
    order to get rid of conditions that use the type code once and for
    all.

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

:::::::::::::: row
::::: {.col-xs-12 .col-sm-6 .col-lg-4}
### Anti-refactoring

:::: dl
::: dt
[Replace Subclass with Fields](replace-subclass-with-fields.html)
:::
::::
:::::

::::::: {.col-xs-12 .col-sm-6 .col-lg-4}
### Similar refactorings

:::: dl
::: dt
[Replace Type Code with Class](replace-type-code-with-class.html)
:::
::::

:::: dl
::: dt
[Replace Type Code with
State/Strategy](replace-type-code-with-state-strategy.html)
:::
::::
:::::::

::::: {.col-xs-12 .col-sm-6 .col-lg-4}
### Eliminates smell

:::: dl
::: dt
[Primitive Obsession](smells/primitive-obsession.html)
:::
::::
:::::
::::::::::::::

::: next
#### 次を読む

[Replace Type Code with State/Strategy []{.fa
.fa-arrow-right}](replace-type-code-with-state-strategy.html){.btn
.btn-primary rel="next"}
:::

::: prev
#### 戻る

[[]{.fa .fa-arrow-left} Replace Type Code with
Class](replace-type-code-with-class.html){.btn .btn-default rel="prev"}
:::
::::::::::::::::::::::::::::::::

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
:::::::::::::::::::::::::::::::::::::
::::::::::::::::::::::::::::::::::::::
