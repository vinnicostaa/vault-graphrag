::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::: {.main-content-container .center-content-container}
::::::::: {.page .text}
::: breadcrumb
[](../index.html){.home} / [Refactoring](../refactoring.html){.type} /
[Code Smells](../refactoring/smells.html){.type} /
[Couplers](../refactoring/smells/couplers.html){.type}
:::

# Inappropriate Intimacy {#inappropriate-intimacy .title}

### Signs and Symptoms

One class uses the internal fields and methods of another class.

<figure class="image">
<img
src="../images/refactoring/content/smells/inappropriate-intimacy-017f9d.png?id=31bf185f4ff946f13e28d27d377a4b6c"
style="aspect-ratio: 1.67;"
srcset="/images/refactoring/content/smells/inappropriate-intimacy-01.png?id=31bf185f4ff946f13e28d27d377a4b6c 500w,/images/refactoring/content/smells/inappropriate-intimacy-01-1.5x.png?id=c41452986a03eb3fb0af7992184f8b22 750w,/images/refactoring/content/smells/inappropriate-intimacy-01-2x.png?id=1857242bb9cf7b2ca50327897d256711 1000w"
sizes="(max-width: 720px) 100vw, 500px" data-fetchpriority="high"
width="500" height="300" />
</figure>

### Reasons for the Problem

Keep a close eye on classes that spend too much time together. Good
classes should know as little about each other as possible. Such classes
are easier to maintain and reuse.

### Treatment

-   The simplest solution is to use [Move Method](../move-method.html)
    and [Move Field](../move-field.html) to move parts of one class to
    the class in which those parts are used. But this works only if the
    first class truly doesn't need these parts.

    <figure class="image">
    <img
    src="../images/refactoring/content/smells/inappropriate-intimacy-024036.png?id=3f23c8df6eb8cf91b46e39fa912ff85c"
    style="aspect-ratio: 1.67;"
    srcset="/images/refactoring/content/smells/inappropriate-intimacy-02.png?id=3f23c8df6eb8cf91b46e39fa912ff85c 500w,/images/refactoring/content/smells/inappropriate-intimacy-02-1.5x.png?id=4c3b0852df877ca82d7fe12d61235b0a 750w,/images/refactoring/content/smells/inappropriate-intimacy-02-2x.png?id=f3ac66384b14f074ed36b6d9c3720b20 1000w"
    sizes="(max-width: 720px) 100vw, 500px" loading="lazy" width="500"
    height="300" />
    </figure>

-   Another solution is to use [Extract Class](../extract-class.html)
    and [Hide Delegate](../hide-delegate.html) on the class to make the
    code relations "official".

-   If the classes are mutually interdependent, you should use [Change
    Bidirectional Association to
    Unidirectional](../change-bidirectional-association-to-unidirectional.html).

-   If this "intimacy" is between a subclass and the superclass,
    consider [Replace Delegation with
    Inheritance](../replace-delegation-with-inheritance.html).

<figure class="image">
<img
src="../images/refactoring/content/smells/inappropriate-intimacy-03b31e.png?id=de33e2285073feaabd1a81cffdcd386c"
style="aspect-ratio: 1.67;"
srcset="/images/refactoring/content/smells/inappropriate-intimacy-03.png?id=de33e2285073feaabd1a81cffdcd386c 500w,/images/refactoring/content/smells/inappropriate-intimacy-03-1.5x.png?id=a2b1cf219e0a4a3056c4e7116b973d85 750w,/images/refactoring/content/smells/inappropriate-intimacy-03-2x.png?id=51bfcec36fb123f146857099555dc478 1000w"
sizes="(max-width: 720px) 100vw, 500px" loading="lazy" width="500"
height="300" />
</figure>

### Payoff

-   Improves code organization.

-   Simplifies support and code reuse.

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

[Message Chains []{.fa .fa-arrow-right}](message-chains.html){.btn
.btn-primary rel="next"}
:::

::: prev
#### Return

[[]{.fa .fa-arrow-left} Feature Envy](feature-envy.html){.btn
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
