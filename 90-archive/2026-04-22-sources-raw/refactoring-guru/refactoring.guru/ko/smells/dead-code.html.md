::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::: {.main-content-container .center-content-container}
::::::::: {.page .text}
::: breadcrumb
[](../../index.html){.home} / [Refactoring](../refactoring.html){.type}
/ [Code Smells](../refactoring/smells.html){.type} /
[Dispensables](../refactoring/smells/dispensables.html){.type}
:::

# Dead Code {#dead-code .title}

### Signs and Symptoms

A variable, parameter, field, method or class is no longer used (usually
because it\'s obsolete).

<figure class="image">
<img
src="../../images/refactoring/content/smells/dead-code-01fd46.png?id=418685bee5de933c472c48efcb5b67a0"
style="aspect-ratio: 1.67;"
srcset="/images/refactoring/content/smells/dead-code-01.png?id=418685bee5de933c472c48efcb5b67a0 500w,/images/refactoring/content/smells/dead-code-01-1.5x.png?id=264efc49269071a3781dd96a1994c249 750w,/images/refactoring/content/smells/dead-code-01-2x.png?id=d9981d853e7e855cf0454fc32aab85a6 1000w"
sizes="(max-width: 720px) 100vw, 500px" data-fetchpriority="high"
width="500" height="300" />
</figure>

### Reasons for the Problem

When requirements for the software have changed or corrections have been
made, nobody had time to clean up the old code.

Such code could also be found in complex conditionals, when one of the
branches becomes unreachable (due to error or other circumstances).

### Treatment

The quickest way to find dead code is to use a good
[IDE](https://en.wikipedia.org/wiki/Integrated_development_environment).

-   Delete unused code and unneeded files.

-   In the case of an unnecessary class, [Inline
    Class](../inline-class.html) or [Collapse
    Hierarchy](../collapse-hierarchy.html) can be applied if a subclass
    or superclass is used.

-   To remove unneeded parameters, use [Remove
    Parameter](../remove-parameter.html).

<figure class="image">
<img
src="../../images/refactoring/content/smells/dead-code-025e37.png?id=b368f23b7cc88340933b761cf2ad1954"
style="aspect-ratio: 1.67;"
srcset="/images/refactoring/content/smells/dead-code-02.png?id=b368f23b7cc88340933b761cf2ad1954 500w,/images/refactoring/content/smells/dead-code-02-1.5x.png?id=c84149b6f42ea2e4ab59b653276627c6 750w,/images/refactoring/content/smells/dead-code-02-2x.png?id=bf78ebf643d890b41b60058d8e67d845 1000w"
sizes="(max-width: 720px) 100vw, 500px" loading="lazy" width="500"
height="300" />
</figure>

### Payoff

-   Reduced code size.

-   Simpler support.

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
#### 다음을 보세요

[Speculative Generality []{.fa
.fa-arrow-right}](speculative-generality.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### 돌아가기

[[]{.fa .fa-arrow-left} Data Class](data-class.html){.btn .btn-default
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
