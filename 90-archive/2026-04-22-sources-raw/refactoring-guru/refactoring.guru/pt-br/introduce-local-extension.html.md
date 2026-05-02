:::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::: {.refactoring .page .text}
::: breadcrumb
[](../index.html){.home} / [Refactoring](refactoring.html){.type} /
[Techniques](refactoring/techniques.html){.type} / [Moving Features
between
Objects](refactoring/techniques/moving-features-between-objects.html){.type}
:::

# Introduce Local Extension {#introduce-local-extension .title}

::::: before-after-container
::: before
### Problem

A utility class doesn't contain some methods that you need. But you
can't add these methods to the class.
:::

::: after
### Solution

Create a new class containing the methods and make it either the child
or wrapper of the utility class.
:::
:::::

:::::::::: examples
::::::::: {.before-after .before-after-container}
::::: before
::: ba
Before
:::

::: ba-container
![Introduce Local Extension -
Before](../images/refactoring/diagrams/Introduce%20Local%20Extension%20-%20Before766c.png?id=3ba103e769691215bed0a50e1de1b80f){width="209"}
:::
:::::

::::: after
::: ba
After
:::

::: ba-container
![Introduce Local Extension -
After](../images/refactoring/diagrams/Introduce%20Local%20Extension%20-%20After041f.png?id=44bc24e5c4b86e28a9dfc9c67c9e21d1){width="209"}
:::
:::::
:::::::::
::::::::::

### Why Refactor

The class that you're using doesn't have the methods that you need.
What's worse, you can't add these methods (because the classes are in a
third-party library, for example). There are two ways out:

-   Create a **subclass** from the relevant class, containing the
    methods and inheriting everything else from the parent class. This
    way is easier but is sometimes blocked by the utility class itself
    (due to `final`).

-   Create a **wrapper** class that contains all the new methods and
    elsewhere will delegate to the related object from the utility
    class. This method is more work since you need not only code to
    maintain the relationship between the wrapper and utility object,
    but also a large number of simple delegating methods in order to
    emulate the public interface of the utility class.

### Benefits

-   By moving additional methods to a separate extension class (wrapper
    or subclass), you avoid gumming up client classes with code that
    doesn't fit. Program components are more coherent and are more
    reusable.

### How to Refactor

1.  Create a new extension class:

    -   Option A: Make it a child of the utility class.

    -   Option B: If you have decided to make a wrapper, create a field
        in it for storing the utility class object to which delegation
        will be made. When using this option, you will need to also
        create methods that repeat the public methods of the utility
        class and contain simple delegation to the methods of the
        utility object.

2.  Create a constructor that uses the parameters of the constructor of
    the utility class.

3.  Also create an alternative "converting" constructor that takes only
    the object of the original class in its parameters. This will help
    to substitute the extension for the objects of the original class.

4.  Create new extended methods in the class. Move foreign methods from
    other classes to this class or else delete the foreign methods if
    their functionality is already present in the extension.

5.  Replace use of the utility class with the new extension class in
    places where its functionality is needed.

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

::: next
#### Leia a seguir

[Organizing Data []{.fa
.fa-arrow-right}](refactoring/techniques/organizing-data.html){.btn
.btn-primary rel="next"}
:::

::: prev
#### Voltar

[[]{.fa .fa-arrow-left} Introduce Foreign
Method](introduce-foreign-method.html){.btn .btn-default rel="prev"}
:::
::::::::::::::::::::

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
:::::::::::::::::::::::::
::::::::::::::::::::::::::
