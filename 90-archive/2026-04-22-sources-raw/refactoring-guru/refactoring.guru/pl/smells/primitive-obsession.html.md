::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::: {.main-content-container .center-content-container}
::::::::: {.page .text}
::: breadcrumb
[](../../index.html){.home} / [Refactoring](../refactoring.html){.type}
/ [Code Smells](../refactoring/smells.html){.type} /
[Bloaters](../refactoring/smells/bloaters.html){.type}
:::

# Primitive Obsession {#primitive-obsession .title}

### Signs and Symptoms

-   Use of primitives instead of small objects for simple tasks (such as
    currency, ranges, special strings for phone numbers, etc.)

-   Use of constants for coding information (such as a constant
    `USER_ADMIN_ROLE = 1` for referring to users with administrator
    rights.)

-   Use of string constants as field names for use in data arrays.

<figure class="image">
<img
src="../../images/refactoring/content/smells/primitive-obsession-0177ec.png?id=73e1c5b08ea68a7ad7a66832644e6696"
style="aspect-ratio: 1.67;"
srcset="/images/refactoring/content/smells/primitive-obsession-01.png?id=73e1c5b08ea68a7ad7a66832644e6696 500w,/images/refactoring/content/smells/primitive-obsession-01-1.5x.png?id=128163a863560b18aa19640c18468bfb 750w,/images/refactoring/content/smells/primitive-obsession-01-2x.png?id=e13be298e4b8d9d4a987972dfc777f4b 1000w"
sizes="(max-width: 720px) 100vw, 500px" data-fetchpriority="high"
width="500" height="300" />
</figure>

### Reasons for the Problem

Like most other smells, primitive obsessions are born in moments of
weakness. "Just a field for storing some data!" the programmer said.
Creating a primitive field is so much easier than making a whole new
class, right? And so it was done. Then another field was needed and
added in the same way. Lo and behold, the class became huge and
unwieldy.

Primitives are often used to "simulate" types. So instead of a separate
data type, you have a set of numbers or strings that form the list of
allowable values for some entity. Easy-to-understand names are then
given to these specific numbers and strings via constants, which is why
they're spread wide and far.

Another example of poor primitive use is field simulation. The class
contains a large array of diverse data and string constants (which are
specified in the class) are used as array indices for getting this data.

### Treatment

-   If you have a large variety of primitive fields, it may be
    possible to logically group some of them into their own class. Even
    better, move the behavior associated with this data into the class
    too. For this task, try [Replace Data Value with
    Object](../replace-data-value-with-object.html).

    <figure class="image">
    <img
    src="../../images/refactoring/content/smells/primitive-obsession-02edeb.png?id=69dfd06eec8d6053e9d88c03ecc78044"
    style="aspect-ratio: 1.67;"
    srcset="/images/refactoring/content/smells/primitive-obsession-02.png?id=69dfd06eec8d6053e9d88c03ecc78044 500w,/images/refactoring/content/smells/primitive-obsession-02-1.5x.png?id=6531a2995361be154aa01d81aa9340a0 750w,/images/refactoring/content/smells/primitive-obsession-02-2x.png?id=255bbb1e0b340ce3b62a5898e8edc75a 1000w"
    sizes="(max-width: 720px) 100vw, 500px" loading="lazy" width="500"
    height="300" />
    </figure>

-   If the values of primitive fields are used in method parameters, go
    with [Introduce Parameter
    Object](../introduce-parameter-object.html) or [Preserve Whole
    Object](../preserve-whole-object.html).

-   When complicated data is coded in variables, use [Replace Type Code
    with Class](../replace-type-code-with-class.html), [Replace Type
    Code with Subclasses](../replace-type-code-with-subclasses.html) or
    [Replace Type Code with
    State/Strategy](../replace-type-code-with-state-strategy.html).

-   If there are arrays among the variables, use [Replace Array with
    Object](../replace-array-with-object.html).

<figure class="image">
<img
src="../../images/refactoring/content/smells/primitive-obsession-03b918.png?id=ab0a8c7b6188265bb9890424e679af2f"
style="aspect-ratio: 1.67;"
srcset="/images/refactoring/content/smells/primitive-obsession-03.png?id=ab0a8c7b6188265bb9890424e679af2f 500w,/images/refactoring/content/smells/primitive-obsession-03-1.5x.png?id=c3dd77a95422b4727db752c6634cf57f 750w,/images/refactoring/content/smells/primitive-obsession-03-2x.png?id=bec1080bcf2be14eeda69b0090d5d3fb 1000w"
sizes="(max-width: 720px) 100vw, 500px" loading="lazy" width="500"
height="300" />
</figure>

### Payoff

-   Code becomes more flexible thanks to use of objects instead of
    primitives.

-   Better understandability and organization of code. Operations on
    particular data are in the same place, instead of being
    scattered. No more guessing about the reason for all these strange
    constants and why they're in an array.

-   Easier finding of duplicate code.

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

[Long Parameter List []{.fa
.fa-arrow-right}](long-parameter-list.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Wróć

[[]{.fa .fa-arrow-left} Large Class](large-class.html){.btn .btn-default
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
