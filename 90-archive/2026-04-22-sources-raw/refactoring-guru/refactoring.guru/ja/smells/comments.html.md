::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::: {.main-content-container .center-content-container}
::::::::: {.page .text}
::: breadcrumb
[](../../index.html){.home} / [Refactoring](../refactoring.html){.type}
/ [Code Smells](../refactoring/smells.html){.type} /
[Dispensables](../refactoring/smells/dispensables.html){.type}
:::

# Comments {#comments .title}

### Signs and Symptoms

A method is filled with explanatory comments.

<figure class="image">
<img
src="../../images/refactoring/content/smells/comments-01cb86.png?id=584958123f3b902e0ad0895d879509b9"
style="aspect-ratio: 1.67;"
srcset="/images/refactoring/content/smells/comments-01.png?id=584958123f3b902e0ad0895d879509b9 500w,/images/refactoring/content/smells/comments-01-1.5x.png?id=c6c70d9f1f12d481d8cabfc726a7fbf7 750w,/images/refactoring/content/smells/comments-01-2x.png?id=15fe22a84b974b19a752ad169ae999ae 1000w"
sizes="(max-width: 720px) 100vw, 500px" data-fetchpriority="high"
width="500" height="300" />
</figure>

### Reasons for the Problem

Comments are usually created with the best of intentions, when the
author realizes that his or her code isn\'t intuitive or obvious. In
such cases, comments are like a deodorant masking the smell of fishy
code that could be improved.

> The best comment is a good name for a method or class.

If you feel that a code fragment can\'t be understood without comments,
try to change the code structure in a way that makes comments
unnecessary.

### Treatment

-   If a comment is intended to explain a complex expression, the
    expression should be split into understandable subexpressions using
    [Extract Variable](../extract-variable.html).

-   If a comment explains a section of code, this section can be turned
    into a separate method via [Extract Method](../extract-method.html).
    The name of the new method can be taken from the comment text
    itself, most likely.

-   If a method has already been extracted, but comments are still
    necessary to explain what the method does, give the method a
    self-explanatory name. Use [Rename Method](../rename-method.html)
    for this.

-   If you need to assert rules about a state that\'s necessary for the
    system to work, use [Introduce
    Assertion](../introduce-assertion.html).

### Payoff

-   Code becomes more intuitive and obvious.

<figure class="image">
<img
src="../../images/refactoring/content/smells/comments-02b499.png?id=266f82bb7081957d409ae690c2c66483"
style="aspect-ratio: 1.67;"
srcset="/images/refactoring/content/smells/comments-02.png?id=266f82bb7081957d409ae690c2c66483 500w,/images/refactoring/content/smells/comments-02-1.5x.png?id=6451f6937bda39e8f101367df8dec883 750w,/images/refactoring/content/smells/comments-02-2x.png?id=57a83d2b705347aa0d0a6d197a1f9d3c 1000w"
sizes="(max-width: 720px) 100vw, 500px" loading="lazy" width="500"
height="300" />
</figure>

### When to Ignore

Comments can sometimes be useful:

-   When explaining **why** something is being implemented in a
    particular way.

-   When explaining complex algorithms (when all other methods for
    simplifying the algorithm have been tried and come up short).

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

[Duplicate Code []{.fa .fa-arrow-right}](duplicate-code.html){.btn
.btn-primary rel="next"}
:::

::: prev
#### 戻る

[[]{.fa
.fa-arrow-left} Dispensables](../refactoring/smells/dispensables.html){.btn
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
