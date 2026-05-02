::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::: {.main-content-container .center-content-container}
::::::::: {.page .text}
::: breadcrumb
[](../../index.html){.home} / [Refactoring](../refactoring.html){.type}
/ [Code Smells](../refactoring/smells.html){.type} /
[Dispensables](../refactoring/smells/dispensables.html){.type}
:::

# Speculative Generality {#speculative-generality .title}

### Signs and Symptoms

There's an unused class, method, field or parameter.

<figure class="image">
<img
src="../../images/refactoring/content/smells/speculative-generality-01d899.png?id=c804fce5c6c5c34b4d9389fcb2aa60aa"
style="aspect-ratio: 1.67;"
srcset="/images/refactoring/content/smells/speculative-generality-01.png?id=c804fce5c6c5c34b4d9389fcb2aa60aa 500w,/images/refactoring/content/smells/speculative-generality-01-1.5x.png?id=475f4713f626920ce866b378e350abc8 750w,/images/refactoring/content/smells/speculative-generality-01-2x.png?id=7875dde405dfa433685fa9040da39b5e 1000w"
sizes="(max-width: 720px) 100vw, 500px" data-fetchpriority="high"
width="500" height="300" />
</figure>

### Reasons for the Problem

Sometimes code is created "just in case" to support anticipated future
features that never get implemented. As a result, code becomes hard to
understand and support.

### Treatment

-   For removing unused abstract classes, try [Collapse
    Hierarchy](../collapse-hierarchy.html).

-   Unnecessary delegation of functionality to another class can be
    eliminated via [Inline Class](../inline-class.html).

-   Unused methods? Use [Inline Method](../inline-method.html) to get
    rid of them.

-   Methods with unused parameters should be given a look with the help
    of [Remove Parameter](../remove-parameter.html).

-   Unused fields can be simply deleted.

<figure class="image">
<img
src="../../images/refactoring/content/smells/speculative-generality-02bab3.png?id=e9d0e8a6170b6d0d0be9cca44175fe44"
style="aspect-ratio: 1.67;"
srcset="/images/refactoring/content/smells/speculative-generality-02.png?id=e9d0e8a6170b6d0d0be9cca44175fe44 500w,/images/refactoring/content/smells/speculative-generality-02-1.5x.png?id=f9c73fb909b254c3451fe73b2dde4548 750w,/images/refactoring/content/smells/speculative-generality-02-2x.png?id=017235ad164cb220c99f21f201872d29 1000w"
sizes="(max-width: 720px) 100vw, 500px" loading="lazy" width="500"
height="300" />
</figure>

### Payoff

-   Slimmer code.

-   Easier support.

### When to Ignore

-   If you're working on a framework, it's eminently reasonable to
    create functionality not used in the framework itself, as long as
    the functionality is needed by the frameworks's users.

-   Before deleting elements, make sure that they aren't used in unit
    tests. This happens if tests need a way to get certain internal
    information from a class or perform special testing-related actions.

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

[Couplers []{.fa
.fa-arrow-right}](../refactoring/smells/couplers.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Retour

[[]{.fa .fa-arrow-left} Dead Code](dead-code.html){.btn .btn-default
rel="prev"}
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
