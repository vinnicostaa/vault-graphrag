::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::: {.main-content-container .center-content-container}
::::::::: {.page .text}
::: breadcrumb
[](../../index.html){.home} / [Refactoring](../refactoring.html){.type}
/ [Code Smells](../refactoring/smells.html){.type} / [Object-Orientation
Abusers](../refactoring/smells/oo-abusers.html){.type}
:::

# Temporary Field {#temporary-field .title}

### Signs and Symptoms

Temporary fields get their values (and thus are needed by objects) only
under certain circumstances. Outside of these circumstances, they're
empty.

<figure class="image">
<img
src="../../images/refactoring/content/smells/temporary-field-01e767.png?id=5e30a8144171693ee4894091762b9742"
style="aspect-ratio: 1.67;"
srcset="/images/refactoring/content/smells/temporary-field-01.png?id=5e30a8144171693ee4894091762b9742 500w,/images/refactoring/content/smells/temporary-field-01-1.5x.png?id=5079cd815f19d0a8938bc64c40f763ab 750w,/images/refactoring/content/smells/temporary-field-01-2x.png?id=1cf05ea67f1e3bd1c1f634af7408f67c 1000w"
sizes="(max-width: 720px) 100vw, 500px" data-fetchpriority="high"
width="500" height="300" />
</figure>

### Reasons for the Problem

Oftentimes, temporary fields are created for use in an algorithm that
requires a large amount of inputs. So instead of creating a large
number of parameters in the method, the programmer decides to create
fields for this data in the class. These fields are used only in the
algorithm and go unused the rest of the time.

This kind of code is tough to understand. You expect to see data in
object fields but for some reason they're almost always empty.

<figure class="image">
<img
src="../../images/refactoring/content/smells/temporary-field-02bc27.png?id=2cf98e02ffb0c1d36a98d48ba7bc5c45"
style="aspect-ratio: 1.67;"
srcset="/images/refactoring/content/smells/temporary-field-02.png?id=2cf98e02ffb0c1d36a98d48ba7bc5c45 500w,/images/refactoring/content/smells/temporary-field-02-1.5x.png?id=d29dd17bc354ec725ad3092a67cecfd8 750w,/images/refactoring/content/smells/temporary-field-02-2x.png?id=6c8e4384d9029cd3ec84cd330d5871eb 1000w"
sizes="(max-width: 720px) 100vw, 500px" loading="lazy" width="500"
height="300" />
</figure>

### Treatment

-   Temporary fields and all code operating on them can be put in a
    separate class via [Extract Class](../extract-class.html). In other
    words, you're creating a method object, achieving the same result as
    if you would perform [Replace Method with Method
    Object](../replace-method-with-method-object.html).

-   [Introduce Null Object](../introduce-null-object.html) and
    integrate it in place of the conditional code which was used to
    check the temporary field values for existence.

<figure class="image">
<img
src="../../images/refactoring/content/smells/temporary-field-032ad0.png?id=cf0e1c1e2a19745d23ca9e1d917d45cc"
style="aspect-ratio: 1.67;"
srcset="/images/refactoring/content/smells/temporary-field-03.png?id=cf0e1c1e2a19745d23ca9e1d917d45cc 500w,/images/refactoring/content/smells/temporary-field-03-1.5x.png?id=09f1df24d41f5f39b6b298b8a71701b7 750w,/images/refactoring/content/smells/temporary-field-03-2x.png?id=c633fd664958307bf296b09de72959dd 1000w"
sizes="(max-width: 720px) 100vw, 500px" loading="lazy" width="500"
height="300" />
</figure>

### Payoff

-   Better code clarity and organization.

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

[Refused Bequest []{.fa .fa-arrow-right}](refused-bequest.html){.btn
.btn-primary rel="next"}
:::

::: prev
#### Wróć

[[]{.fa .fa-arrow-left} Switch Statements](switch-statements.html){.btn
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
