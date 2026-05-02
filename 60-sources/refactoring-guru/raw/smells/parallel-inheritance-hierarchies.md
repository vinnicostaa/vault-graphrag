::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::: {.main-content-container .center-content-container}
::::::::: {.page .text}
::: breadcrumb
[](../index.html){.home} / [Refactoring](../refactoring.html){.type} /
[Code Smells](../refactoring/smells.html){.type} / [Change
Preventers](../refactoring/smells/change-preventers.html){.type}
:::

# Parallel Inheritance Hierarchies {#parallel-inheritance-hierarchies .title}

### Signs and Symptoms

Whenever you create a subclass for a class, you find yourself needing to
create a subclass for another class.

<figure class="image">
<img
src="../images/refactoring/content/smells/parallel-inheritance-hierarchies-0187af.png?id=9167875f5f0e80256edcc8fcaaed3563"
style="aspect-ratio: 1.67;"
srcset="/images/refactoring/content/smells/parallel-inheritance-hierarchies-01.png?id=9167875f5f0e80256edcc8fcaaed3563 500w,/images/refactoring/content/smells/parallel-inheritance-hierarchies-01-1.5x.png?id=e2129d5eb8da79a491ed0c7d802e8d1c 750w,/images/refactoring/content/smells/parallel-inheritance-hierarchies-01-2x.png?id=975e6a0589795c59b47ed3aa122beead 1000w"
sizes="(max-width: 720px) 100vw, 500px" data-fetchpriority="high"
width="500" height="300" />
</figure>

### Reasons for the Problem

All was well as long as the hierarchy stayed small. But with new classes
being added, making changes has become harder and harder.

### Treatment

-   You may de-duplicate parallel class hierarchies in two steps. First,
    make instances of one hierarchy refer to instances of another
    hierarchy. Then, remove the hierarchy in the referred class, by
    using [Move Method](../move-method.html) and [Move
    Field](../move-field.html).

### Payoff

-   Reduces code duplication.

-   Can improve organization of code.

<figure class="image">
<img
src="../images/refactoring/content/smells/parallel-inheritance-hierarchies-02b2cf.png?id=4dca6795d3d087b23ad1027298d6f1dd"
style="aspect-ratio: 1.67;"
srcset="/images/refactoring/content/smells/parallel-inheritance-hierarchies-02.png?id=4dca6795d3d087b23ad1027298d6f1dd 500w,/images/refactoring/content/smells/parallel-inheritance-hierarchies-02-1.5x.png?id=ef89bd3332ebb11a6c6974a9e8017779 750w,/images/refactoring/content/smells/parallel-inheritance-hierarchies-02-2x.png?id=b45e8dde4f4abbe2f0d329964c921960 1000w"
sizes="(max-width: 720px) 100vw, 500px" loading="lazy" width="500"
height="300" />
</figure>

### When to Ignore

-   Sometimes having parallel class hierarchies is just a way to avoid
    even bigger mess with program architecture. If you find that your
    attempts to de-duplicate hierarchies produce even uglier code, just
    step out, revert all of your changes and get used to that code.

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

[Dispensables []{.fa
.fa-arrow-right}](../refactoring/smells/dispensables.html){.btn
.btn-primary rel="next"}
:::

::: prev
#### Return

[[]{.fa .fa-arrow-left} Shotgun Surgery](shotgun-surgery.html){.btn
.btn-default rel="prev"}
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
