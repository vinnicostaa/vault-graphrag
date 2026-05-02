::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::: {.main-content-container .center-content-container}
::::::::: {.page .text}
::: breadcrumb
[](../../index.html){.home} / [Refactoring](../refactoring.html){.type}
/ [Code Smells](../refactoring/smells.html){.type} / [Object-Orientation
Abusers](../refactoring/smells/oo-abusers.html){.type}
:::

# Alternative Classes with Different Interfaces {#alternative-classes-with-different-interfaces .title}

### Signs and Symptoms

Two classes perform identical functions but have different method names.

<figure class="image">
<img
src="../../images/refactoring/content/smells/alternative-classes-with-different-interfaces-011fe4.png?id=e5fccb2e5390e0a62b5c9f56029bd361"
style="aspect-ratio: 1.67;"
srcset="/images/refactoring/content/smells/alternative-classes-with-different-interfaces-01.png?id=e5fccb2e5390e0a62b5c9f56029bd361 500w,/images/refactoring/content/smells/alternative-classes-with-different-interfaces-01-1.5x.png?id=41cee7e73c74ea76fd0e5a65157aef1d 750w,/images/refactoring/content/smells/alternative-classes-with-different-interfaces-01-2x.png?id=00f0e52d679514e0c16e836e7cee5c24 1000w"
sizes="(max-width: 720px) 100vw, 500px" data-fetchpriority="high"
width="500" height="300" />
</figure>

### Reasons for the Problem

The programmer who created one of the classes probably didn\'t know that
a functionally equivalent class already existed.

### Treatment

Try to put the interface of classes in terms of a common denominator:

-   [Rename Method](../rename-method.html)s to make them identical in
    all alternative classes.

-   [Move Method](../move-method.html), [Add
    Parameter](../add-parameter.html) and [Parameterize
    Method](../parameterize-method.html) to make the signature and
    implementation of methods the same.

-   If only part of the functionality of the classes is duplicated, try
    using [Extract Superclass](../extract-superclass.html). In this
    case, the existing classes will become subclasses.

-   After you have determined which treatment method to use and
    implemented it, you may be able to delete one of the classes.

### Payoff

-   You get rid of unnecessary duplicated code, making the resulting
    code less bulky.

-   Code becomes more readable and understandable (you no longer have to
    guess the reason for creation of a second class performing the exact
    same functions as the first one).

<figure class="image">
<img
src="../../images/refactoring/content/smells/alternative-classes-with-different-interfaces-02ece4.png?id=669874e082965799a70076a120288c6a"
style="aspect-ratio: 1.67;"
srcset="/images/refactoring/content/smells/alternative-classes-with-different-interfaces-02.png?id=669874e082965799a70076a120288c6a 500w,/images/refactoring/content/smells/alternative-classes-with-different-interfaces-02-1.5x.png?id=720363b1a9f666cc0d41345ccf4e4de3 750w,/images/refactoring/content/smells/alternative-classes-with-different-interfaces-02-2x.png?id=db011d16b1dcea2e68d252eb435e63ef 1000w"
sizes="(max-width: 720px) 100vw, 500px" loading="lazy" width="500"
height="300" />
</figure>

### When to Ignore

-   Sometimes merging classes is impossible or so difficult as to be
    pointless. One example is when the alternative classes are in
    different libraries that each have their own version of the class.

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

[Change Preventers []{.fa
.fa-arrow-right}](../refactoring/smells/change-preventers.html){.btn
.btn-primary rel="next"}
:::

::: prev
#### 돌아가기

[[]{.fa .fa-arrow-left} Refused Bequest](refused-bequest.html){.btn
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
