:::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::: {.refactoring .page .text}
::: breadcrumb
[](../index.html){.home} / [Refactoring](refactoring.html){.type} /
[Techniques](refactoring/techniques.html){.type} / [Simplifying Method
Calls](refactoring/techniques/simplifying-method-calls.html){.type}
:::

# Hide Method {#hide-method .title}

::::: before-after-container
::: before
### Problem

A method isn't used by other classes or is used only inside its own
class hierarchy.
:::

::: after
### Solution

Make the method private or protected.
:::
:::::

:::::::::: examples
::::::::: {.before-after .before-after-container}
::::: before
::: ba
Before
:::

::: ba-container
![Hide Method -
Before](../images/refactoring/diagrams/Hide%20Method%20-%20Before37bb.png?id=598219cd5a4b26974b091534b2dc89ae){width="134"}
:::
:::::

::::: after
::: ba
After
:::

::: ba-container
![Hide Method -
After](../images/refactoring/diagrams/Hide%20Method%20-%20After5926.png?id=2f7f1435a959a0a611778513e5bc4365){width="134"}
:::
:::::
:::::::::
::::::::::

### Why Refactor

Quite often, the need to hide methods for getting and setting values is
due to development of a richer interface that provides additional
behavior, especially if you started with a class that added little
beyond mere data encapsulation.

As new behavior is built into the class, you may find that public getter
and setter methods are no longer necessary and can be hidden. If you
make getter or setter methods private and apply direct access to
variables, you can delete the method.

### Benefits

-   Hiding methods makes it easier for your code to evolve. When you
    change a private method, you only need to worry about how to not
    break the current class since you know that the method can't be used
    anywhere else.

-   By making methods private, you underscore the importance of the
    public interface of the class and of the methods that remain public.

### How to Refactor

1.  Regularly try to find methods that can be made private. Static code
    analysis and good unit test coverage can offer a big leg up.

2.  Make each method as private as possible.

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

:::::: row
::::: {.col-xs-12 .col-sm-6 .col-lg-12}
### Eliminates smell

:::: dl
::: dt
[Data Class](smells/data-class.html)
:::
::::
:::::
::::::

::: next
#### Leer siguiente

[Replace Constructor with Factory Method []{.fa
.fa-arrow-right}](replace-constructor-with-factory-method.html){.btn
.btn-primary rel="next"}
:::

::: prev
#### Volver

[[]{.fa .fa-arrow-left} Remove Setting
Method](remove-setting-method.html){.btn .btn-default rel="prev"}
:::
::::::::::::::::::::::::

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
:::::::::::::::::::::::::::::
::::::::::::::::::::::::::::::
