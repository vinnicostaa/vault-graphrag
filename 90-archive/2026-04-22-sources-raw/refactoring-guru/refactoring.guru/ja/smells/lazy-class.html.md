::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::: {.main-content-container .center-content-container}
::::::::: {.page .text}
::: breadcrumb
[](../../index.html){.home} / [Refactoring](../refactoring.html){.type}
/ [Code Smells](../refactoring/smells.html){.type} /
[Dispensables](../refactoring/smells/dispensables.html){.type}
:::

# Lazy Class {#lazy-class .title}

### Signs and Symptoms

Understanding and maintaining classes always costs time and money. So if
a class doesn\'t do enough to earn your attention, it should be deleted.

<figure class="image">
<img
src="../../images/refactoring/content/smells/lazy-class-0148c1.png?id=efec5911dfaaa3ba69d3eb4dab03fd3c"
style="aspect-ratio: 1.67;"
srcset="/images/refactoring/content/smells/lazy-class-01.png?id=efec5911dfaaa3ba69d3eb4dab03fd3c 500w,/images/refactoring/content/smells/lazy-class-01-1.5x.png?id=4b9cf29aa6991dc8050debc5494a2a92 750w,/images/refactoring/content/smells/lazy-class-01-2x.png?id=3b5b07bc60eb98c883fc68c5e1a05aed 1000w"
sizes="(max-width: 720px) 100vw, 500px" data-fetchpriority="high"
width="500" height="300" />
</figure>

### Reasons for the Problem

Perhaps a class was designed to be fully functional but after some of
the refactoring it has become ridiculously small.

Or perhaps it was designed to support future development work that never
got done.

### Treatment

-   Components that are near-useless should be given the [Inline
    Class](../inline-class.html) treatment.

-   For subclasses with few functions, try [Collapse
    Hierarchy](../collapse-hierarchy.html).

<figure class="image">
<img
src="../../images/refactoring/content/smells/lazy-class-02adb6.png?id=393302f2bd27ba0197660caea274ae23"
style="aspect-ratio: 1.67;"
srcset="/images/refactoring/content/smells/lazy-class-02.png?id=393302f2bd27ba0197660caea274ae23 500w,/images/refactoring/content/smells/lazy-class-02-1.5x.png?id=72b0880a8673312f80bd10755cdeae74 750w,/images/refactoring/content/smells/lazy-class-02-2x.png?id=d46dd63f159b40aa266ccbdbefb319bd 1000w"
sizes="(max-width: 720px) 100vw, 500px" loading="lazy" width="500"
height="300" />
</figure>

### Payoff

-   Reduced code size.

-   Easier maintenance.

### When to Ignore

-   Sometimes a *Lazy Class* is created in order to delineate intentions
    for future development, In this case, try to maintain a balance
    between clarity and simplicity in your code.

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
#### 次を読む

[Data Class []{.fa .fa-arrow-right}](data-class.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### 戻る

[[]{.fa .fa-arrow-left} Duplicate Code](duplicate-code.html){.btn
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
