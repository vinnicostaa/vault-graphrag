::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::: {.main-content-container .center-content-container}
::::::::: {.page .text}
::: breadcrumb
[](../../index.html){.home} / [Refactoring](../refactoring.html){.type}
/ [Code Smells](../refactoring/smells.html){.type} /
[Couplers](../refactoring/smells/couplers.html){.type}
:::

# Message Chains {#message-chains .title}

### Signs and Symptoms

In code you see a series of calls resembling `$a->b()->c()->d()`

<figure class="image">
<img
src="../../images/refactoring/content/smells/message-chains-01e234.png?id=c290ab1d348b3e6ab500c0b949f3d3f8"
style="aspect-ratio: 1.67;"
srcset="/images/refactoring/content/smells/message-chains-01.png?id=c290ab1d348b3e6ab500c0b949f3d3f8 500w,/images/refactoring/content/smells/message-chains-01-1.5x.png?id=66d768c6d2df06a8744c861b48b5b404 750w,/images/refactoring/content/smells/message-chains-01-2x.png?id=63332ad44f028e0d60f42d3a56e0280a 1000w"
sizes="(max-width: 720px) 100vw, 500px" data-fetchpriority="high"
width="500" height="300" />
</figure>

### Reasons for the Problem

A message chain occurs when a client requests another object, that
object requests yet another one, and so on. These chains mean that the
client is dependent on navigation along the class structure. Any changes
in these relationships require modifying the client.

### Treatment

-   To delete a message chain, use [Hide
    Delegate](../hide-delegate.html).

-   Sometimes it's better to think of why the end object is being used.
    Perhaps it would make sense to use [Extract
    Method](../extract-method.html) for this functionality and move it
    to the beginning of the chain, by using [Move
    Method](../move-method.html).

<figure class="image">
<img
src="../../images/refactoring/content/smells/message-chains-02cbe3.png?id=d348325f450e592900b1a4a2ed960b53"
style="aspect-ratio: 1.67;"
srcset="/images/refactoring/content/smells/message-chains-02.png?id=d348325f450e592900b1a4a2ed960b53 500w,/images/refactoring/content/smells/message-chains-02-1.5x.png?id=277dfdf5360730a6e4babf0c4732cd29 750w,/images/refactoring/content/smells/message-chains-02-2x.png?id=07214cc9363ca16dcbaab4bb6e1edeef 1000w"
sizes="(max-width: 720px) 100vw, 500px" loading="lazy" width="500"
height="300" />
</figure>

### Payoff

-   Reduces dependencies between classes of a chain.

-   Reduces the amount of bloated code.

<figure class="image">
<img
src="../../images/refactoring/content/smells/message-chains-03d2e4.png?id=e651ac11f057e3e2e7c7786fc4051a66"
style="aspect-ratio: 1.67;"
srcset="/images/refactoring/content/smells/message-chains-03.png?id=e651ac11f057e3e2e7c7786fc4051a66 500w,/images/refactoring/content/smells/message-chains-03-1.5x.png?id=4a3dc90e543359c0f886d46d05b5b1e5 750w,/images/refactoring/content/smells/message-chains-03-2x.png?id=2e0e5bdf1e249a09f9c8e67f01de6bd1 1000w"
sizes="(max-width: 720px) 100vw, 500px" loading="lazy" width="500"
height="300" />
</figure>

### When to Ignore

-   Overly aggressive delegate hiding can cause code in which it's hard
    to see where the functionality is actually occurring. Which is
    another way of saying, avoid the [Middle Man](middle-man.html) smell
    as well.

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

[ Let\'s see...](../../refactoring/course.html){.btn .btn-secondary}
:::
:::::

::: next
#### Leer siguiente

[Middle Man []{.fa .fa-arrow-right}](middle-man.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Volver

[[]{.fa .fa-arrow-left} Inappropriate
Intimacy](inappropriate-intimacy.html){.btn .btn-default rel="prev"}
:::
:::::::::

:::::: {.prom .banner-sidebar .banner-removable .banner-removable-refactoring data-id="Ref: Sidebar" creative-id="standard-sidebar-en" position="sidebar"}
::::: ban
::: {.image .product-image}
[Sale!]{.banner-discount}
[![](../../images/refactoring/course/course-cover-en1b63.jpg?id=ac6ad836a27c4a8d579244f15c48a8b4){width="300"
height="300" loading="lazy"}](../../refactoring/course.html)
:::

::: {.banner-text .banner-text-en}
This code smell is part of the much bigger **Refactoring Course**.

[ Learn more...](../../refactoring/course.html){.btn .btn-secondary
.btn-block}
:::
:::::
::::::
::::::::::::::
:::::::::::::::
