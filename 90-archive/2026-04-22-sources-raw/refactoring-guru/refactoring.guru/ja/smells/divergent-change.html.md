::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::: {.main-content-container .center-content-container}
::::::::: {.page .text}
::: breadcrumb
[](../../index.html){.home} / [Refactoring](../refactoring.html){.type}
/ [Code Smells](../refactoring/smells.html){.type} / [Change
Preventers](../refactoring/smells/change-preventers.html){.type}
:::

# Divergent Change {#divergent-change .title}

> *Divergent Change* resembles [Shotgun Surgery](shotgun-surgery.html)
> but is actually the opposite smell. *Divergent Change* is when many
> changes are made to a single class. *Shotgun Surgery* refers to when a
> single change is made to multiple classes simultaneously.

### Signs and Symptoms

You find yourself having to change many unrelated methods when you make
changes to a class. For example, when adding a new product type you have
to change the methods for finding, displaying, and ordering products.

<figure class="image">
<img
src="../../images/refactoring/content/smells/divergent-change-010f98.png?id=d62e68e1778d67bf82ff74064c24de33"
style="aspect-ratio: 1.67;"
srcset="/images/refactoring/content/smells/divergent-change-01.png?id=d62e68e1778d67bf82ff74064c24de33 500w,/images/refactoring/content/smells/divergent-change-01-1.5x.png?id=166d6b9318e5bcaa76ea673e0346d58e 750w,/images/refactoring/content/smells/divergent-change-01-2x.png?id=1c7d20737703941d1e3f7ad85e180578 1000w"
sizes="(max-width: 720px) 100vw, 500px" data-fetchpriority="high"
width="500" height="300" />
</figure>

### Reasons for the Problem

Often these divergent modifications are due to poor program structure or
\"copypasta programming".

### Treatment

-   Split up the behavior of the class via [Extract
    Class](../extract-class.html).

-   If different classes have the same behavior, you may want to combine
    the classes through inheritance ([Extract
    Superclass](../extract-superclass.html) and [Extract
    Subclass](../extract-subclass.html)).

<figure class="image">
<img
src="../../images/refactoring/content/smells/divergent-change-02aacb.png?id=21b6fd7cba36f123c09497cb8f5a5625"
style="aspect-ratio: 1.67;"
srcset="/images/refactoring/content/smells/divergent-change-02.png?id=21b6fd7cba36f123c09497cb8f5a5625 500w,/images/refactoring/content/smells/divergent-change-02-1.5x.png?id=1c18d8c9f8a40887e60041d4d5027532 750w,/images/refactoring/content/smells/divergent-change-02-2x.png?id=581f6218d8a2393ece88419ad60831da 1000w"
sizes="(max-width: 720px) 100vw, 500px" loading="lazy" width="500"
height="300" />
</figure>

### Payoff

-   Improves code organization.

-   Reduces code duplication.

-   Simplifies support.

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

[Shotgun Surgery []{.fa .fa-arrow-right}](shotgun-surgery.html){.btn
.btn-primary rel="next"}
:::

::: prev
#### 戻る

[[]{.fa .fa-arrow-left} Change
Preventers](../refactoring/smells/change-preventers.html){.btn
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
