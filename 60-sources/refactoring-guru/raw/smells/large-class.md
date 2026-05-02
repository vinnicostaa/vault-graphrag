::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::: {.main-content-container .center-content-container}
::::::::: {.page .text}
::: breadcrumb
[](../index.html){.home} / [Refactoring](../refactoring.html){.type} /
[Code Smells](../refactoring/smells.html){.type} /
[Bloaters](../refactoring/smells/bloaters.html){.type}
:::

# Large Class {#large-class .title}

### Signs and Symptoms

A class contains many fields/methods/lines of code.

<figure class="image">
<img
src="../images/refactoring/content/smells/large-class-01eaa7.png?id=acac82f25cc90aaa413c2daefebf0e4b"
style="aspect-ratio: 1.67;"
srcset="/images/refactoring/content/smells/large-class-01.png?id=acac82f25cc90aaa413c2daefebf0e4b 500w,/images/refactoring/content/smells/large-class-01-1.5x.png?id=ba2cf337d9a765204a1b313169f65cf8 750w,/images/refactoring/content/smells/large-class-01-2x.png?id=44aea94399b8bd6398a01b46b5bc7f29 1000w"
sizes="(max-width: 720px) 100vw, 500px" data-fetchpriority="high"
width="500" height="300" />
</figure>

### Reasons for the Problem

Classes usually start small. But over time, they get bloated as the
program grows.

As is the case with long methods as well, programmers usually find it
mentally less taxing to place a new feature in an existing class than to
create a new class for the feature.

<figure class="image">
<img
src="../images/refactoring/content/smells/large-class-027a0b.png?id=973b37334ae57489945a88b9327f81e3"
style="aspect-ratio: 1.67;"
srcset="/images/refactoring/content/smells/large-class-02.png?id=973b37334ae57489945a88b9327f81e3 500w,/images/refactoring/content/smells/large-class-02-1.5x.png?id=4b4846ad906c16339481236196214bcb 750w,/images/refactoring/content/smells/large-class-02-2x.png?id=f51627abdfb96fad29cb114d00795fec 1000w"
sizes="(max-width: 720px) 100vw, 500px" loading="lazy" width="500"
height="300" />
</figure>

### Treatment

When a class is wearing too many (functional) hats, think about
splitting it up:

-   [Extract Class](../extract-class.html) helps if part of the behavior
    of the large class can be spun off into a separate component.

-   [Extract Subclass](../extract-subclass.html) helps if part of the
    behavior of the large class can be implemented in different ways or
    is used in rare cases.

-   [Extract Interface](../extract-interface.html) helps if it's
    necessary to have a list of the operations and behaviors that the
    client can use.

-   If a large class is responsible for the graphical interface, you may
    try to move some of its data and behavior to a separate domain
    object. In doing so, it may be necessary to store copies of some
    data in two places and keep the data consistent. [Duplicate Observed
    Data](../duplicate-observed-data.html) offers a way to do this.

<figure class="image">
<img
src="../images/refactoring/content/smells/large-class-030a79.png?id=f0a0109f731dbc420ffe385cb658f0de"
style="aspect-ratio: 1.67;"
srcset="/images/refactoring/content/smells/large-class-03.png?id=f0a0109f731dbc420ffe385cb658f0de 500w,/images/refactoring/content/smells/large-class-03-1.5x.png?id=8f7537669b0af86e9e77928004ad865f 750w,/images/refactoring/content/smells/large-class-03-2x.png?id=2e497ff65fc035f0d51f908361daee78 1000w"
sizes="(max-width: 720px) 100vw, 500px" loading="lazy" width="500"
height="300" />
</figure>

### Payoff

-   Refactoring of these classes spares developers from needing to
    remember a large number of attributes for a class.

-   In many cases, splitting large classes into parts avoids duplication
    of code and functionality.

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
#### Read next

[Primitive Obsession []{.fa
.fa-arrow-right}](primitive-obsession.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Return

[[]{.fa .fa-arrow-left} Long Method](long-method.html){.btn .btn-default
rel="prev"}
:::
:::::::::

:::::: {.prom .banner-sidebar .banner-removable .banner-removable-refactoring data-id="Ref: Sidebar" creative-id="standard-sidebar-en" position="sidebar"}
::::: ban
::: {.image .product-image}
[Sale!]{.banner-discount}
[![](../images/refactoring/course/course-cover-en1b63.jpg?id=ac6ad836a27c4a8d579244f15c48a8b4){width="300"
height="300" loading="lazy"}](../refactoring/course.html)
:::

::: {.banner-text .banner-text-en}
This code smell is part of the much bigger **Refactoring Course**.

[ Learn more...](../refactoring/course.html){.btn .btn-secondary
.btn-block}
:::
:::::
::::::
::::::::::::::
:::::::::::::::
