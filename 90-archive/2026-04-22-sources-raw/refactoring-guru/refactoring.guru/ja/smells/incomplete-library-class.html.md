::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::: {.main-content-container .center-content-container}
::::::::: {.page .text}
::: breadcrumb
[](../../index.html){.home} / [Refactoring](../refactoring.html){.type}
/ [Code Smells](../refactoring/smells.html){.type} / [Other
Smells](../refactoring/smells/other.html){.type}
:::

# Incomplete Library Class {#incomplete-library-class .title}

### Signs and Symptoms

Sooner or later,
[libraries](https://en.wikipedia.org/wiki/Library_(computing)) stop
meeting user needs. The only solution to the problem---changing the
library---is often impossible since the library is read-only.

<figure class="image">
<img
src="../../images/refactoring/content/smells/incomplete-library-class-0152c5.png?id=ca51f740f7fd39b7de1430b64cae9f8c"
style="aspect-ratio: 1.67;"
srcset="/images/refactoring/content/smells/incomplete-library-class-01.png?id=ca51f740f7fd39b7de1430b64cae9f8c 500w,/images/refactoring/content/smells/incomplete-library-class-01-1.5x.png?id=e0efa4b241a785afba5f0e85672dbe4a 750w,/images/refactoring/content/smells/incomplete-library-class-01-2x.png?id=25c39ccf56423153b7c977c57943af54 1000w"
sizes="(max-width: 720px) 100vw, 500px" data-fetchpriority="high"
width="500" height="300" />
</figure>

### Reasons for the Problem

The author of the library hasn\'t provided the features you need or has
refused to implement them.

### Treatment

-   To introduce a few methods to a library class, use [Introduce
    Foreign Method](../introduce-foreign-method.html).

-   For big changes in a class library, use [Introduce Local
    Extension](../introduce-local-extension.html).

### Payoff

-   Reduces code duplication (instead of creating your own library from
    scratch, you can still piggy-back off an existing one).

<figure class="image">
<img
src="../../images/refactoring/content/smells/incomplete-library-class-02b14e.png?id=05a8d9c631d43a3fb256196f366fd089"
style="aspect-ratio: 1.67;"
srcset="/images/refactoring/content/smells/incomplete-library-class-02.png?id=05a8d9c631d43a3fb256196f366fd089 500w,/images/refactoring/content/smells/incomplete-library-class-02-1.5x.png?id=bb28ce39729bc28b39030ed6b0ff60b0 750w,/images/refactoring/content/smells/incomplete-library-class-02-2x.png?id=cb204d62084939b3d9e6f97d5d3662ee 1000w"
sizes="(max-width: 720px) 100vw, 500px" loading="lazy" width="500"
height="300" />
</figure>

### When to Ignore

-   Extending a library can generate additional work if the changes to
    the library involve changes in code.

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

[Techniques []{.fa
.fa-arrow-right}](../refactoring/techniques.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### 戻る

[[]{.fa .fa-arrow-left} Other
Smells](../refactoring/smells/other.html){.btn .btn-default rel="prev"}
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
