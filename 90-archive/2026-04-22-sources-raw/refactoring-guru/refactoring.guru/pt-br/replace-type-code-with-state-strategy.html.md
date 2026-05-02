:::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::::::::::::: {.refactoring .page .text}
::: breadcrumb
[](../index.html){.home} / [Refactoring](refactoring.html){.type} /
[Techniques](refactoring/techniques.html){.type} / [Organizing
Data](refactoring/techniques/organizing-data.html){.type}
:::

# Replace Type Code with State/Strategy {#replace-type-code-with-statestrategy .title}

> **What's type code?** Type code occurs when, instead of a separate
> data type, you have a set of numbers or strings that form a list of
> allowable values for some entity. Often these specific numbers and
> strings are given understandable names via constants, which is the
> reason for why such type code is encountered so much.

::::: before-after-container
::: before
### Problem

You have a coded type that affects behavior but you can't use subclasses
to get rid of it.
:::

::: after
### Solution

Replace type code with a state object. If it's necessary to replace a
field value with type code, another state object is "plugged in".
:::
:::::

:::::::::: examples
::::::::: {.before-after .before-after-container}
::::: before
::: ba
Before
:::

::: ba-container
![Replace Type Code with State-Strategy -
Before](../images/refactoring/diagrams/Replace%20Type%20Code%20with%20State-Strategy%20-%20Before4bb7.png?id=5fa75a3c084933dcb4d1d1f0c60e56d3){width="152"}
:::
:::::

::::: after
::: ba
After
:::

::: ba-container
![Replace Type Code with State-Strategy -
After](../images/refactoring/diagrams/Replace%20Type%20Code%20with%20State-Strategy%20-%20After15f4.png?id=1fb190a5b389c0a88e7335648b869748){width="452"}
:::
:::::
:::::::::
::::::::::

### Why Refactor

You have type code and it affects the behavior of a class, therefore we
can't use [Replace Type Code with
Class](replace-type-code-with-class.html).

Type code affects the behavior of a class but we can't create subclasses
for the coded type due to the existing class hierarchy or other reasons.
Thus means that we can't apply [Replace Type Code with
Subclasses](replace-type-code-with-subclasses.html).

### Benefits

-   This refactoring technique is a way out of situations when a field
    with a coded type changes its value during the object's lifetime. In
    this case, replacement of the value is made via replacement of the
    state object to which the original class refers.

-   If you need to add a new value of a coded type, all you need to do
    is to add a new state subclass without altering the existing code
    (cf. the *Open/Closed Principle*).

### Drawbacks

-   If you have a simple case of type code but you use this refactoring
    technique anyway, you will have many extra (and unneeded) classes.

### Good to Know

Implementation of this refactoring technique can make use of one of two
design patterns: **State** or **Strategy**. Implementation is the same
no matter which pattern you choose. So which pattern should you pick in
a particular situation?

If you're trying to split a conditional that controls the selection of
algorithms, use Strategy.

But if each value of the coded type is responsible not only for
selecting an algorithm but for the whole condition of the class, class
state, field values, and many other actions, State is better for the
job.

### How to Refactor

1.  Use [Self Encapsulate Field](self-encapsulate-field.html) to create
    a getter for the field that contains type code.

2.  Create a new class and give it an understandable name that fits the
    purpose of the type code. This class will be playing the role of
    *state* (or *strategy*). In it, create an abstract coded field
    getter.

3.  Create subclasses of the state class for each value of the coded
    type. In each subclass, redefine the getter of the coded field so
    that it returns the corresponding value of the coded type.

4.  In the abstract state class, create a static factory method that
    accepts the value of the coded type as a parameter. Depending on
    this parameter, the factory method will create objects of various
    states. For this, in its code create a large conditional; it'll be
    the only one when refactoring is complete.

5.  In the original class, change the type of the coded field to the
    state class. In the field's setter, call the factory state method
    for getting new state objects.

6.  Now you can start to move the fields and methods from the superclass
    to the corresponding state subclasses (using [Push Down
    Field](push-down-field.html) and [Push Down
    Method](push-down-method.html)).

7.  When everything moveable has been moved, use [Replace Conditional
    with Polymorphism](replace-conditional-with-polymorphism.html) in
    order to get rid of conditionals that use type code once and for
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

:::::::::::::::: row
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
Subclasses](replace-type-code-with-subclasses.html)
:::
::::
:::::::

::::::: {.col-xs-12 .col-sm-6 .col-lg-4}
### Implements design pattern

:::: dl
::: dt
[State](design-patterns/state.html)
:::
::::

:::: dl
::: dt
[Strategy](design-patterns/strategy.html)
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
::::::::::::::::

::: next
#### Leia a seguir

[Replace Subclass with Fields []{.fa
.fa-arrow-right}](replace-subclass-with-fields.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Voltar

[[]{.fa .fa-arrow-left} Replace Type Code with
Subclasses](replace-type-code-with-subclasses.html){.btn .btn-default
rel="prev"}
:::
::::::::::::::::::::::::::::::::::

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
:::::::::::::::::::::::::::::::::::::::
::::::::::::::::::::::::::::::::::::::::
