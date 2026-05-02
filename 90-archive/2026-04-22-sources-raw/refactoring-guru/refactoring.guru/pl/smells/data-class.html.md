::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::: {.main-content-container .center-content-container}
::::::::: {.page .text}
::: breadcrumb
[](../../index.html){.home} / [Refactoring](../refactoring.html){.type}
/ [Code Smells](../refactoring/smells.html){.type} /
[Dispensables](../refactoring/smells/dispensables.html){.type}
:::

# Data Class {#data-class .title}

### Signs and Symptoms

A data class refers to a class that contains only fields and crude
methods for accessing them (getters and setters). These are simply
containers for data used by other classes. These classes don't contain
any additional functionality and can't independently operate on the data
that they own.

<figure class="image">
<img
src="../../images/refactoring/content/smells/data-class-01949b.png?id=2ea1583b05a194a056d27ac559545318"
style="aspect-ratio: 1.67;"
srcset="/images/refactoring/content/smells/data-class-01.png?id=2ea1583b05a194a056d27ac559545318 500w,/images/refactoring/content/smells/data-class-01-1.5x.png?id=e816a7b99f2cbef399ea43b1af66125b 750w,/images/refactoring/content/smells/data-class-01-2x.png?id=2beb8150d4ba31ca37d6515495ceff2d 1000w"
sizes="(max-width: 720px) 100vw, 500px" data-fetchpriority="high"
width="500" height="300" />
</figure>

### Reasons for the Problem

It's a normal thing when a newly created class contains only a few
public fields (and maybe even a handful of getters/setters). But the
true power of objects is that they can contain behavior types or
operations on their data.

### Treatment

-   If a class contains public fields, use [Encapsulate
    Field](../encapsulate-field.html) to hide them from direct access
    and require that access be performed via getters and setters only.

-   Use [Encapsulate Collection](../encapsulate-collection.html) for
    data stored in collections (such as arrays).

-   Review the client code that uses the class. In it, you may find
    functionality that would be better located in the data class
    itself. If this is the case, use [Move Method](../move-method.html)
    and [Extract Method](../extract-method.html) to migrate this
    functionality to the data class.

-   After the class has been filled with well thought-out methods, you
    may want to get rid of old methods for data access that give overly
    broad access to the class data. For this, [Remove Setting
    Method](../remove-setting-method.html) and [Hide
    Method](../hide-method.html) may be helpful.

<figure class="image">
<img
src="../../images/refactoring/content/smells/data-class-024cf2.png?id=db0eb15f9f229bafd8423b2cfd09f910"
style="aspect-ratio: 1.67;"
srcset="/images/refactoring/content/smells/data-class-02.png?id=db0eb15f9f229bafd8423b2cfd09f910 500w,/images/refactoring/content/smells/data-class-02-1.5x.png?id=f2f0339a74ba638bfbd8d57317396f43 750w,/images/refactoring/content/smells/data-class-02-2x.png?id=fb9b6d670232d6effe790980e6b388ec 1000w"
sizes="(max-width: 720px) 100vw, 500px" loading="lazy" width="500"
height="300" />
</figure>

### Payoff

-   Improves understanding and organization of code. Operations on
    particular data are now gathered in a single place, instead of
    haphazardly throughout the code.

-   Helps you to spot duplication of client code.

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

[Dead Code []{.fa .fa-arrow-right}](dead-code.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Wróć

[[]{.fa .fa-arrow-left} Lazy Class](lazy-class.html){.btn .btn-default
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
