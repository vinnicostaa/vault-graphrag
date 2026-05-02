::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::::::::: {.refactoring .page .text}
::: breadcrumb
[](../index.html){.home} / [Refactoring](refactoring.html){.type} /
[Techniques](refactoring/techniques.html){.type} / [Moving Features
between
Objects](refactoring/techniques/moving-features-between-objects.html){.type}
:::

# Hide Delegate {#hide-delegate .title}

::::: before-after-container
::: before
### Problem

The client gets object B from a field or method of object А. Then the
client calls a method of object B.
:::

::: after
### Solution

Create a new method in class A that delegates the call to object B. Now
the client doesn\'t know about, or depend on, class B.
:::
:::::

:::::::::: examples
::::::::: {.before-after .before-after-container}
::::: before
::: ba
Before
:::

::: ba-container
![Hide Delegate -
Before](../images/refactoring/diagrams/Hide%20Delegate%20-%20Before0412.png?id=f7de1016e76545f7c51af09463ce5f4c){width="415"}
:::
:::::

::::: after
::: ba
After
:::

::: ba-container
![Hide Delegate -
After](../images/refactoring/diagrams/Hide%20Delegate%20-%20After921c.png?id=f51110f3e0d4423b3f9088e92fc3dce4){width="153"}
:::
:::::
:::::::::
::::::::::

### Why Refactor

To start with, let\'s look at terminology:

-   *Server* is the object to which the client has direct access.

-   *Delegate* is the end object that contains the functionality needed
    by the client.

A call chain appears when a client requests an object from another
object, then the second object requests another one, and so on. These
sequences of calls involve the client in navigation along the class
structure. Any changes in these interrelationships will require changes
on the client side.

### Benefits

-   Hides delegation from the client. The less that the client code
    needs to know about the details of relationships between objects,
    the easier it\'s to make changes to your program.

### Drawbacks

-   If you need to create an excessive number of delegating methods,
    *server-class* risks becoming an unneeded go-between, leading to an
    excess of [Middle Man](smells/middle-man.html).

### How to Refactor

1.  For each method of the *delegate-class* called by the client, create
    a method in the *server-class* that delegates the call to the
    *delegate-class*.

2.  Change the client code so that it calls the methods of the
    *server-class*.

3.  If your changes free the client from needing the *delegate-class*,
    you can remove the access method to the *delegate-class* from the
    *server-class* (the method that was originally used to get the
    *delegate-class*).

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

::::::::::: row
::::: {.col-xs-12 .col-sm-6 .col-lg-6}
### Anti-refactoring

:::: dl
::: dt
[Remove Middle Man](remove-middle-man.html)
:::
::::
:::::

::::::: {.col-xs-12 .col-sm-6 .col-lg-6}
### Eliminates smell

:::: dl
::: dt
[Message Chains](smells/message-chains.html)
:::
::::

:::: dl
::: dt
[Inappropriate Intimacy](smells/inappropriate-intimacy.html)
:::
::::
:::::::
:::::::::::

::: next
#### 다음을 보세요

[Remove Middle Man []{.fa .fa-arrow-right}](remove-middle-man.html){.btn
.btn-primary rel="next"}
:::

::: prev
#### 돌아가기

[[]{.fa .fa-arrow-left} Inline Class](inline-class.html){.btn
.btn-default rel="prev"}
:::
:::::::::::::::::::::::::::::

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
::::::::::::::::::::::::::::::::::
:::::::::::::::::::::::::::::::::::
