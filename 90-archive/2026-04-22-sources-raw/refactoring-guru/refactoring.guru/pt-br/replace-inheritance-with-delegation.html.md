::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::::::: {.refactoring .page .text}
::: breadcrumb
[](../index.html){.home} / [Refactoring](refactoring.html){.type} /
[Techniques](refactoring/techniques.html){.type} / [Dealing with
Generalization](refactoring/techniques/dealing-with-generalization.html){.type}
:::

# Replace Inheritance with Delegation {#replace-inheritance-with-delegation .title}

::::: before-after-container
::: before
### Problem

You have a subclass that uses only a portion of the methods of its
superclass (or it's not possible to inherit superclass data).
:::

::: after
### Solution

Create a field and put a superclass object in it, delegate methods to
the superclass object, and get rid of inheritance.
:::
:::::

:::::::::: examples
::::::::: {.before-after .before-after-container}
::::: before
::: ba
Before
:::

::: ba-container
![Replace Inheritance with Delegation -
Before](../images/refactoring/diagrams/Replace%20Inheritance%20with%20Delegation%20-%20Before7f4b.png?id=ea9631c50eaae9a19a51f039b44e35bf){width="152"}
:::
:::::

::::: after
::: ba
After
:::

::: ba-container
![Replace Inheritance with Delegation -
After](../images/refactoring/diagrams/Replace%20Inheritance%20with%20Delegation%20-%20Afterf4ed.png?id=7c6d23661792658feae42d051b2fd7ea){width="453"}
:::
:::::
:::::::::
::::::::::

### Why Refactor

Replacing inheritance with composition can substantially improve class
design if:

-   Your subclass violates the *Liskov substitution principle*, i.e., if
    inheritance was implemented only to combine common code but not
    because the subclass is an extension of the superclass.

-   The subclass uses only a portion of the methods of the superclass.
    In this case, it's only a matter of time before someone calls a
    superclass method that he or she wasn't supposed to call.

In essence, this refactoring technique splits both classes and makes the
superclass the helper of the subclass, not its parent. Instead of
inheriting all superclass methods, the subclass will have only the
necessary methods for delegating to the methods of the superclass
object.

### Benefits

-   A class doesn't contain any unneeded methods inherited from the
    superclass.

-   Various objects with various implementations can be put in the
    delegate field. In effect you get the
    [Strategy](design-patterns/strategy.html) design pattern.

### Drawbacks

-   You have to write many simple delegating methods.

### How to Refactor

1.  Create a field in the subclass for holding the superclass. During
    the initial stage, place the current object in it.

2.  Change the subclass methods so that they use the superclass object
    instead of `this`.

3.  For methods inherited from the superclass that are called in the
    client code, create simple delegating methods in the subclass.

4.  Remove the inheritance declaration from the subclass.

5.  Change the initialization code of the field in which the former
    superclass is stored by creating a new object.

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
[Replace Delegation with
Inheritance](replace-delegation-with-inheritance.html)
:::
::::
:::::

::::: {.col-xs-12 .col-sm-6 .col-lg-6}
### Implements design pattern

:::: dl
::: dt
[Strategy](design-patterns/strategy.html)
:::
::::
:::::
:::::::::

::: next
#### Leia a seguir

[Replace Delegation with Inheritance []{.fa
.fa-arrow-right}](replace-delegation-with-inheritance.html){.btn
.btn-primary rel="next"}
:::

::: prev
#### Voltar

[[]{.fa .fa-arrow-left} Form Template
Method](form-template-method.html){.btn .btn-default rel="prev"}
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
