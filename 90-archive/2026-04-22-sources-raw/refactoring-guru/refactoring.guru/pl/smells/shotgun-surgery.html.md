::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::: {.main-content-container .center-content-container}
::::::::: {.page .text}
::: breadcrumb
[](../../index.html){.home} / [Refactoring](../refactoring.html){.type}
/ [Code Smells](../refactoring/smells.html){.type} / [Change
Preventers](../refactoring/smells/change-preventers.html){.type}
:::

# Shotgun Surgery {#shotgun-surgery .title}

> *Shotgun Surgery* resembles [Divergent Change](divergent-change.html)
> but is actually the opposite smell. *Divergent Change* is when many
> changes are made to a single class. *Shotgun Surgery* refers to when a
> single change is made to multiple classes simultaneously.

### Signs and Symptoms

Making any modifications requires that you make many small changes to
many different classes.

<figure class="image">
<img
src="../../images/refactoring/content/smells/shotgun-surgery-01e5e8.png?id=9cc1117a6d787364788e152a3adb6a53"
style="aspect-ratio: 1.67;"
srcset="/images/refactoring/content/smells/shotgun-surgery-01.png?id=9cc1117a6d787364788e152a3adb6a53 500w,/images/refactoring/content/smells/shotgun-surgery-01-1.5x.png?id=38debad74fb67588357f07c149e76569 750w,/images/refactoring/content/smells/shotgun-surgery-01-2x.png?id=01431b43fcaee83fade53530a3dd91ab 1000w"
sizes="(max-width: 720px) 100vw, 500px" data-fetchpriority="high"
width="500" height="300" />
</figure>

### Reasons for the Problem

A single responsibility has been split up among a large number of
classes. This can happen after overzealous application of [Divergent
Change](divergent-change.html).

<figure class="image">
<img
src="../../images/refactoring/content/smells/shotgun-surgery-02b284.png?id=48f8a4a0f17d112e02ae73bacaed43fa"
style="aspect-ratio: 1.67;"
srcset="/images/refactoring/content/smells/shotgun-surgery-02.png?id=48f8a4a0f17d112e02ae73bacaed43fa 500w,/images/refactoring/content/smells/shotgun-surgery-02-1.5x.png?id=fce8ce3dacc4177bbc18af12a39da6cd 750w,/images/refactoring/content/smells/shotgun-surgery-02-2x.png?id=a35426ca3f6e64857e66b2fdeb395870 1000w"
sizes="(max-width: 720px) 100vw, 500px" loading="lazy" width="500"
height="300" />
</figure>

### Treatment

-   Use [Move Method](../move-method.html) and [Move
    Field](../move-field.html) to move existing class behaviors into a
    single class. If there's no class appropriate for this, create a new
    one.

-   If moving code to the same class leaves the original classes almost
    empty, try to get rid of these now-redundant classes via [Inline
    Class](../inline-class.html).

<figure class="image">
<img
src="../../images/refactoring/content/smells/shotgun-surgery-030760.png?id=cf013f14eb5cde98bd48595a1c9836a9"
style="aspect-ratio: 1.67;"
srcset="/images/refactoring/content/smells/shotgun-surgery-03.png?id=cf013f14eb5cde98bd48595a1c9836a9 500w,/images/refactoring/content/smells/shotgun-surgery-03-1.5x.png?id=c29cddda01b23e0dd21d8b12f5937f84 750w,/images/refactoring/content/smells/shotgun-surgery-03-2x.png?id=259b00413f0f8be143ead703c74b7e38 1000w"
sizes="(max-width: 720px) 100vw, 500px" loading="lazy" width="500"
height="300" />
</figure>

### Payoff

-   Better organization.

-   Less code duplication.

-   Easier maintenance.

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
#### Czytaj dalej

[Parallel Inheritance Hierarchies []{.fa
.fa-arrow-right}](parallel-inheritance-hierarchies.html){.btn
.btn-primary rel="next"}
:::

::: prev
#### Wróć

[[]{.fa .fa-arrow-left} Divergent Change](divergent-change.html){.btn
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
