::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::: {.main-content-container .center-content-container}
::::::::: {.page .text}
::: breadcrumb
[](../../index.html){.home} / [Refactoring](../refactoring.html){.type}
/ [Code Smells](../refactoring/smells.html){.type} /
[Bloaters](../refactoring/smells/bloaters.html){.type}
:::

# Long Parameter List {#long-parameter-list .title}

### Signs and Symptoms

More than three or four parameters for a method.

<figure class="image">
<img
src="../../images/refactoring/content/smells/long-parameter-list-015e37.png?id=06fad4adaf485cfaa569e66c20f268eb"
style="aspect-ratio: 1.67;"
srcset="/images/refactoring/content/smells/long-parameter-list-01.png?id=06fad4adaf485cfaa569e66c20f268eb 500w,/images/refactoring/content/smells/long-parameter-list-01-1.5x.png?id=7e78b50f369819e051b3aa450a183659 750w,/images/refactoring/content/smells/long-parameter-list-01-2x.png?id=d964f68180e89b6312726c7a5719e35d 1000w"
sizes="(max-width: 720px) 100vw, 500px" data-fetchpriority="high"
width="500" height="300" />
</figure>

### Reasons for the Problem

A long list of parameters might happen after several types of algorithms
are merged in a single method. A long list may have been created to
control which algorithm will be run and how.

Long parameter lists may also be the byproduct of efforts to make
classes more independent of each other. For example, the code for
creating specific objects needed in a method was moved from the
method to the code for calling the method, but the created objects are
passed to the method as parameters. Thus the original class no longer
knows about the relationships between objects, and dependency has
decreased. But if several of these objects are created, each of them
will require its own parameter, which means a longer parameter list.

It's hard to understand such lists, which become contradictory and
hard to use as they grow longer. Instead of a long list of parameters, a
method can use the data of its own object. If the current object doesn't
contain all necessary data, another object (which will get the necessary
data) can be passed as a method parameter.

### Treatment

-   Check what values are passed to parameters. If some of the arguments
    are just results of method calls of another object, use [Replace
    Parameter with Method
    Call](../replace-parameter-with-method-call.html). This object
    can be placed in the field of its own class or passed as a method
    parameter.

-   Instead of passing a group of data received from another object as
    parameters, pass the object itself to the method, by using [Preserve
    Whole Object](../preserve-whole-object.html).

-   But if these parameters are coming from different sources, you can
    pass them as a single parameter object via [Introduce Parameter
    Object](../introduce-parameter-object.html).

<figure class="image">
<img
src="../../images/refactoring/content/smells/long-parameter-list-020f28.png?id=7571291fcaea939ed137400cbe0f3c02"
style="aspect-ratio: 1.67;"
srcset="/images/refactoring/content/smells/long-parameter-list-02.png?id=7571291fcaea939ed137400cbe0f3c02 500w,/images/refactoring/content/smells/long-parameter-list-02-1.5x.png?id=f933042e101fee9bb5424c308c62daeb 750w,/images/refactoring/content/smells/long-parameter-list-02-2x.png?id=90514c8d76f4eae7b76439778b82c778 1000w"
sizes="(max-width: 720px) 100vw, 500px" loading="lazy" width="500"
height="300" />
</figure>

### Payoff

-   More readable, shorter code.

-   Refactoring may reveal previously unnoticed duplicate code.

### When to Ignore

-   Don't get rid of parameters if doing so would cause unwanted
    dependency between classes.

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

[Data Clumps []{.fa .fa-arrow-right}](data-clumps.html){.btn
.btn-primary rel="next"}
:::

::: prev
#### Wróć

[[]{.fa .fa-arrow-left} Primitive
Obsession](primitive-obsession.html){.btn .btn-default rel="prev"}
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
