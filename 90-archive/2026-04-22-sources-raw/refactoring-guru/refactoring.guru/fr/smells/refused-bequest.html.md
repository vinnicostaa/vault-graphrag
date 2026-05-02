::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::: {.main-content-container .center-content-container}
::::::::: {.page .text}
::: breadcrumb
[](../../index.html){.home} / [Refactoring](../refactoring.html){.type}
/ [Code Smells](../refactoring/smells.html){.type} / [Object-Orientation
Abusers](../refactoring/smells/oo-abusers.html){.type}
:::

# Refused Bequest {#refused-bequest .title}

### Signs and Symptoms

If a subclass uses only some of the methods and properties inherited
from its parents, the hierarchy is off-kilter. The unneeded methods may
simply go unused or be redefined and give off exceptions.

<figure class="image">
<img
src="../../images/refactoring/content/smells/refused-bequest-014f59.png?id=7a1d79e75a3836c22ec865d72c98664e"
style="aspect-ratio: 1.67;"
srcset="/images/refactoring/content/smells/refused-bequest-01.png?id=7a1d79e75a3836c22ec865d72c98664e 500w,/images/refactoring/content/smells/refused-bequest-01-1.5x.png?id=90bf2279c14d74bc0e63b9bcc3b69808 750w,/images/refactoring/content/smells/refused-bequest-01-2x.png?id=d2e31b7b9fa3326118817b8e0c65e435 1000w"
sizes="(max-width: 720px) 100vw, 500px" data-fetchpriority="high"
width="500" height="300" />
</figure>

### Reasons for the Problem

Someone was motivated to create inheritance between classes only by the
desire to reuse the code in a superclass. But the superclass and
subclass are completely different.

<figure class="image">
<img
src="../../images/refactoring/content/smells/refused-bequest-02c0d0.png?id=f9b0affd4bbf6fec22c05783fc75562e"
style="aspect-ratio: 1.67;"
srcset="/images/refactoring/content/smells/refused-bequest-02.png?id=f9b0affd4bbf6fec22c05783fc75562e 500w,/images/refactoring/content/smells/refused-bequest-02-1.5x.png?id=28b34af7a9319c2d15c32e4a5890c835 750w,/images/refactoring/content/smells/refused-bequest-02-2x.png?id=33b42b7d51bca13f27e4933d24f82751 1000w"
sizes="(max-width: 720px) 100vw, 500px" loading="lazy" width="500"
height="300" />
</figure>

### Treatment

-   If inheritance makes no sense and the subclass really does have
    nothing in common with the superclass, eliminate inheritance in
    favor of [Replace Inheritance with
    Delegation](../replace-inheritance-with-delegation.html).

-   If inheritance is appropriate, get rid of unneeded fields and
    methods in the subclass. Extract all fields and methods needed by
    the subclass from the parent class, put them in a new superclass,
    and set both classes to inherit from it ([Extract
    Superclass](../extract-superclass.html)).

<figure class="image">
<img
src="../../images/refactoring/content/smells/refused-bequest-03ba1b.png?id=2a84293620fa1caf4329fca1f4a44e08"
style="aspect-ratio: 1.67;"
srcset="/images/refactoring/content/smells/refused-bequest-03.png?id=2a84293620fa1caf4329fca1f4a44e08 500w,/images/refactoring/content/smells/refused-bequest-03-1.5x.png?id=9c9b198ba12e2bb4a4e73ea3df19de8b 750w,/images/refactoring/content/smells/refused-bequest-03-2x.png?id=6990ba0083e3de07881bd551928e3a79 1000w"
sizes="(max-width: 720px) 100vw, 500px" loading="lazy" width="500"
height="300" />
</figure>

### Payoff

-   Improves code clarity and organization. You will no longer have to
    wonder why the `Dog` class is inherited from the `Chair` class (even
    though they both have 4 legs).

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
#### Suivant

[Alternative Classes with Different Interfaces []{.fa
.fa-arrow-right}](alternative-classes-with-different-interfaces.html){.btn
.btn-primary rel="next"}
:::

::: prev
#### Retour

[[]{.fa .fa-arrow-left} Temporary Field](temporary-field.html){.btn
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
