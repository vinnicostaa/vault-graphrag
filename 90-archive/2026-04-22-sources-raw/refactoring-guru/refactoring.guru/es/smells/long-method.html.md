::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::: {.main-content-container .center-content-container}
::::::::: {.page .text}
::: breadcrumb
[](../../index.html){.home} / [Refactoring](../refactoring.html){.type}
/ [Code Smells](../refactoring/smells.html){.type} /
[Bloaters](../refactoring/smells/bloaters.html){.type}
:::

# Long Method {#long-method .title}

### Signs and Symptoms

A method contains too many lines of code. Generally, any method longer
than ten lines should make you start asking questions.

<figure class="image">
<img
src="../../images/refactoring/content/smells/long-method-012095.png?id=ba3b4a6d8ef25a8f676543cee5e1e019"
style="aspect-ratio: 1.67;"
srcset="/images/refactoring/content/smells/long-method-01.png?id=ba3b4a6d8ef25a8f676543cee5e1e019 500w,/images/refactoring/content/smells/long-method-01-1.5x.png?id=0aba5fa284aac80c0c3c909fca9ff51c 750w,/images/refactoring/content/smells/long-method-01-2x.png?id=fd9479c2221526f284c9b14fb94f9169 1000w"
sizes="(max-width: 720px) 100vw, 500px" data-fetchpriority="high"
width="500" height="300" />
</figure>

### Reasons for the Problem

Like the Hotel California, something is always being added to a method
but nothing is ever taken out. Since it's easier to write code than to
read it, this "smell" remains unnoticed until the method turns into an
ugly, oversized beast.

Mentally, it's often harder to create a new method than to add to an
existing one: "But it's just two lines, there's no use in creating a
whole method just for that\..." Which means that another line is added
and then yet another, giving birth to a tangle of spaghetti code.

### Treatment

As a rule of thumb, if you feel the need to comment on something inside
a method, you should take this code and put it in a new method. Even a
single line can and should be split off into a separate method, if it
requires explanations. And if the method has a descriptive name, nobody
will need to look at the code to see what it does.

<figure class="image">
<img
src="../../images/refactoring/content/smells/long-method-027655.png?id=274350a92b305ae79848ab40b3bdb0cb"
style="aspect-ratio: 1.67;"
srcset="/images/refactoring/content/smells/long-method-02.png?id=274350a92b305ae79848ab40b3bdb0cb 500w,/images/refactoring/content/smells/long-method-02-1.5x.png?id=dbb0cb724dd6beefe6474b86ae33913c 750w,/images/refactoring/content/smells/long-method-02-2x.png?id=beba19e840bf4763e85f006ef79cc89c 1000w"
sizes="(max-width: 720px) 100vw, 500px" loading="lazy" width="500"
height="300" />
</figure>

-   To reduce the length of a method body, use [Extract
    Method](../extract-method.html).

-   If local variables and parameters interfere with extracting a
    method, use [Replace Temp with
    Query](../replace-temp-with-query.html), [Introduce Parameter
    Object](../introduce-parameter-object.html) or [Preserve Whole
    Object](../preserve-whole-object.html).

-   If none of the previous recipes help, try moving the entire method
    to a separate object via [Replace Method with Method
    Object](../replace-method-with-method-object.html).

-   Conditional operators and loops are a good clue that code can be
    moved to a separate method. For conditionals, use [Decompose
    Conditional](../decompose-conditional.html). If loops are in the
    way, try [Extract Method](../extract-method.html).

### Payoff

-   Among all types of object-oriented code, classes with short methods
    live longest. The longer a method or function is, the harder it
    becomes to understand and maintain it.

-   In addition, long methods offer the perfect hiding place for
    unwanted duplicate code.

<figure class="image">
<img
src="../../images/refactoring/content/smells/long-method-03ca39.png?id=82ce2d388aa14bdae4e8f62b875f0259"
style="aspect-ratio: 1.67;"
srcset="/images/refactoring/content/smells/long-method-03.png?id=82ce2d388aa14bdae4e8f62b875f0259 500w,/images/refactoring/content/smells/long-method-03-1.5x.png?id=fd71378acb56437e26c4b9b5654a3f28 750w,/images/refactoring/content/smells/long-method-03-2x.png?id=ba563da468cf42a704ff53da2e89b6d5 1000w"
sizes="(max-width: 720px) 100vw, 500px" loading="lazy" width="500"
height="300" />
</figure>

### Performance

Does an increase in the number of methods hurt performance, as many
people claim? In almost all cases the impact is so negligible that it's
not even worth worrying about.

Plus, now that you have clear and understandable code, you're more
likely to find truly effective methods for restructuring code and
getting real performance gains if the need ever arises.

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
#### Leer siguiente

[Large Class []{.fa .fa-arrow-right}](large-class.html){.btn
.btn-primary rel="next"}
:::

::: prev
#### Volver

[[]{.fa
.fa-arrow-left} Bloaters](../refactoring/smells/bloaters.html){.btn
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
