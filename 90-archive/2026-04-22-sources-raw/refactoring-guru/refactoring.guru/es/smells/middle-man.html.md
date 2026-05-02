::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::: {.main-content-container .center-content-container}
::::::::: {.page .text}
::: breadcrumb
[](../../index.html){.home} / [Refactoring](../refactoring.html){.type}
/ [Code Smells](../refactoring/smells.html){.type} /
[Couplers](../refactoring/smells/couplers.html){.type}
:::

# Middle Man {#middle-man .title}

### Signs and Symptoms

If a class performs only one action, delegating work to another class,
why does it exist at all?

<figure class="image">
<img
src="../../images/refactoring/content/smells/middle-man-01d4ce.png?id=14c65845c4e0cf03e7e9e48108090c98"
style="aspect-ratio: 1.67;"
srcset="/images/refactoring/content/smells/middle-man-01.png?id=14c65845c4e0cf03e7e9e48108090c98 500w,/images/refactoring/content/smells/middle-man-01-1.5x.png?id=9fd227d0b8d3e082fef36f02df56d907 750w,/images/refactoring/content/smells/middle-man-01-2x.png?id=a1a99f8b475b719d9f894aa613515761 1000w"
sizes="(max-width: 720px) 100vw, 500px" data-fetchpriority="high"
width="500" height="300" />
</figure>

### Reasons for the Problem

This smell can be the result of overzealous elimination of [Message
Chains](message-chains.html).

In other cases, it can be the result of the useful work of a class being
gradually moved to other classes. The class remains as an empty shell
that doesn't do anything other than delegate.

### Treatment

-   If most of a method's classes delegate to another class, [Remove
    Middle Man](../remove-middle-man.html) is in order.

### Payoff

-   Less bulky code.

<figure class="image">
<img
src="../../images/refactoring/content/smells/middle-man-02455a.png?id=f507c0fd9a7bde8df8c22b9027d0a404"
style="aspect-ratio: 1.67;"
srcset="/images/refactoring/content/smells/middle-man-02.png?id=f507c0fd9a7bde8df8c22b9027d0a404 500w,/images/refactoring/content/smells/middle-man-02-1.5x.png?id=4ff17bc08cd0dd6f84473f971873f154 750w,/images/refactoring/content/smells/middle-man-02-2x.png?id=41869f090e8263d46e708778fe64059c 1000w"
sizes="(max-width: 720px) 100vw, 500px" loading="lazy" width="500"
height="300" />
</figure>

### When to Ignore

Don't delete middle man that have been created for a reason:

-   A middle man may have been added to avoid interclass dependencies.

-   Some design patterns create middle man on purpose (such as
    [Proxy](../design-patterns/proxy.html) or
    [Decorator](../design-patterns/decorator.html)).

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

[Other Smells []{.fa
.fa-arrow-right}](../refactoring/smells/other.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Volver

[[]{.fa .fa-arrow-left} Message Chains](message-chains.html){.btn
.btn-default rel="prev"}
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
