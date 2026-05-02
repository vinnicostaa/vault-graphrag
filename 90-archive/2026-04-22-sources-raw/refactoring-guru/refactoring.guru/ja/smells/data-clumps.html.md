::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::: {.main-content-container .center-content-container}
::::::::: {.page .text}
::: breadcrumb
[](../../index.html){.home} / [Refactoring](../refactoring.html){.type}
/ [Code Smells](../refactoring/smells.html){.type} /
[Bloaters](../refactoring/smells/bloaters.html){.type}
:::

# Data Clumps {#data-clumps .title}

### Signs and Symptoms

Sometimes different parts of the code contain identical groups of
variables (such as parameters for connecting to a database). These
clumps should be turned into their own classes.

<figure class="image">
<img
src="../../images/refactoring/content/smells/data-clumps-018c21.png?id=9d8a38ce942720cee728797eba695a2f"
style="aspect-ratio: 1.67;"
srcset="/images/refactoring/content/smells/data-clumps-01.png?id=9d8a38ce942720cee728797eba695a2f 500w,/images/refactoring/content/smells/data-clumps-01-1.5x.png?id=148bb6bd87a02e66d83bc11bda3ae7e6 750w,/images/refactoring/content/smells/data-clumps-01-2x.png?id=64c7f4113e3c06f10dbec825833fa190 1000w"
sizes="(max-width: 720px) 100vw, 500px" data-fetchpriority="high"
width="500" height="300" />
</figure>

### Reasons for the Problem

Often these data groups are due to poor program structure or \"copypasta
programming".

If you want to make sure whether or not some data is a data clump, just
delete one of the data values and see whether the other values still
make sense. If this isn\'t the case, this is a good sign that this group
of variables should be combined into an object.

### Treatment

-   If repeating data comprises the fields of a class, use [Extract
    Class](../extract-class.html) to move the fields to their own class.

-   If the same data clumps are passed in the parameters of methods, use
    [Introduce Parameter Object](../introduce-parameter-object.html) to
    set them off as a class.

-   If some of the data is passed to other methods, think about passing
    the entire data object to the method instead of just individual
    fields. [Preserve Whole Object](../preserve-whole-object.html) will
    help with this.

-   Look at the code used by these fields. It may be a good idea to move
    this code to a data class.

<figure class="image">
<img
src="../../images/refactoring/content/smells/data-clumps-02ec1e.png?id=cfb0a8fa64a983473dd31527e28ae158"
style="aspect-ratio: 1.67;"
srcset="/images/refactoring/content/smells/data-clumps-02.png?id=cfb0a8fa64a983473dd31527e28ae158 500w,/images/refactoring/content/smells/data-clumps-02-1.5x.png?id=2dbf0e8e5960f6797d32abbbc1579d3f 750w,/images/refactoring/content/smells/data-clumps-02-2x.png?id=195f24da42dbd21508aa9ef46a57abba 1000w"
sizes="(max-width: 720px) 100vw, 500px" loading="lazy" width="500"
height="300" />
</figure>

### Payoff

-   Improves understanding and organization of code. Operations on
    particular data are now gathered in a single place, instead of
    haphazardly throughout the code.

-   Reduces code size.

<figure class="image">
<img
src="../../images/refactoring/content/smells/data-clumps-038987.png?id=c170bbdea77b7d4a26947ef328b70adc"
style="aspect-ratio: 1.67;"
srcset="/images/refactoring/content/smells/data-clumps-03.png?id=c170bbdea77b7d4a26947ef328b70adc 500w,/images/refactoring/content/smells/data-clumps-03-1.5x.png?id=88875edb97b4047aa9bfb2fd319c0f19 750w,/images/refactoring/content/smells/data-clumps-03-2x.png?id=2b4a70e09a6236715a9bc4bd4663508b 1000w"
sizes="(max-width: 720px) 100vw, 500px" loading="lazy" width="500"
height="300" />
</figure>

### When to Ignore

-   Passing an entire object in the parameters of a method, instead of
    passing just its values (primitive types), may create an undesirable
    dependency between the two classes.

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

[Object-Orientation Abusers []{.fa
.fa-arrow-right}](../refactoring/smells/oo-abusers.html){.btn
.btn-primary rel="next"}
:::

::: prev
#### 戻る

[[]{.fa .fa-arrow-left} Long Parameter
List](long-parameter-list.html){.btn .btn-default rel="prev"}
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
