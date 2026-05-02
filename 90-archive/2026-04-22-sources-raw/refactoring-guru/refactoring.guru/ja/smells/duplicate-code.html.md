::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::: {.main-content-container .center-content-container}
::::::::: {.page .text}
::: breadcrumb
[](../../index.html){.home} / [Refactoring](../refactoring.html){.type}
/ [Code Smells](../refactoring/smells.html){.type} /
[Dispensables](../refactoring/smells/dispensables.html){.type}
:::

# Duplicate Code {#duplicate-code .title}

### Signs and Symptoms

Two code fragments look almost identical.

<figure class="image">
<img
src="../../images/refactoring/content/smells/duplicate-code-013568.png?id=16fe591195fa50073551852b3d44844e"
style="aspect-ratio: 1.67;"
srcset="/images/refactoring/content/smells/duplicate-code-01.png?id=16fe591195fa50073551852b3d44844e 500w,/images/refactoring/content/smells/duplicate-code-01-1.5x.png?id=fc62042bc009d4890e7b1d37fe1d5817 750w,/images/refactoring/content/smells/duplicate-code-01-2x.png?id=9e462bc4bd52927cf45cfc7dbc5907af 1000w"
sizes="(max-width: 720px) 100vw, 500px" data-fetchpriority="high"
width="500" height="300" />
</figure>

### Reasons for the Problem

Duplication usually occurs when multiple programmers are working on
different parts of the same program at the same time. Since they\'re
working on different tasks, they may be unaware their colleague has
already written similar code that could be repurposed for their own
needs.

There\'s also more subtle duplication, when specific parts of code look
different but actually perform the same job. This kind of duplication
can be hard to find and fix.

Sometimes duplication is purposeful. When rushing to meet deadlines and
the existing code is \"almost right\" for the job, novice programmers
may not be able to resist the temptation of copying and pasting the
relevant code. And in some cases, the programmer is simply too lazy to
de-clutter.

### Treatment

-   If the same code is found in two or more methods in the same class:
    use [Extract Method](../extract-method.html) and place calls for the
    new method in both places.

    <figure class="image">
    <img
    src="../../images/refactoring/content/smells/duplicate-code-020123.png?id=50d92af3defe2c2688f66cde102c9c09"
    style="aspect-ratio: 1.67;"
    srcset="/images/refactoring/content/smells/duplicate-code-02.png?id=50d92af3defe2c2688f66cde102c9c09 500w,/images/refactoring/content/smells/duplicate-code-02-1.5x.png?id=224c1f9b44e12cfc3ef7b39e6230d53c 750w,/images/refactoring/content/smells/duplicate-code-02-2x.png?id=5b9325ca1b0369ec3423808380fa9022 1000w"
    sizes="(max-width: 720px) 100vw, 500px" loading="lazy" width="500"
    height="300" />
    </figure>

-   If the same code is found in two subclasses of the same level:

    -   Use [Extract Method](../extract-method.html) for both classes,
        followed by [Pull Up Field](../pull-up-field.html) for the
        fields used in the method that you\'re pulling up.

    -   If the duplicate code is inside a constructor, use [Pull Up
        Constructor Body](../pull-up-constructor-body.html).

    -   If the duplicate code is similar but not completely identical,
        use [Form Template Method](../form-template-method.html).

    -   If two methods do the same thing but use different algorithms,
        select the best algorithm and apply [Substitute
        Algorithm](../substitute-algorithm.html).

-   If duplicate code is found in two different classes:

    -   If the classes aren\'t part of a hierarchy, use [Extract
        Superclass](../extract-superclass.html) in order to create a
        single superclass for these classes that maintains all the
        previous functionality.

    -   If it\'s difficult or impossible to create a superclass, use
        [Extract Class](../extract-class.html) in one class and use the
        new component in the other.

-   If a large number of conditional expressions are present and perform
    the same code (differing only in their conditions), merge these
    operators into a single condition using [Consolidate Conditional
    Expression](../consolidate-conditional-expression.html) and use
    [Extract Method](../extract-method.html) to place the condition in a
    separate method with an easy-to-understand name.

-   If the same code is performed in all branches of a conditional
    expression: place the identical code outside of the condition tree
    by using [Consolidate Duplicate Conditional
    Fragments](../consolidate-duplicate-conditional-fragments.html).

### Payoff

-   Merging duplicate code simplifies the structure of your code and
    makes it shorter.

-   Simplification + shortness = code that\'s easier to simplify and
    cheaper to support.

<figure class="image">
<img
src="../../images/refactoring/content/smells/duplicate-code-038497.png?id=bd88b98ff5e5e1b5a4019cb0a50df9f5"
style="aspect-ratio: 1.67;"
srcset="/images/refactoring/content/smells/duplicate-code-03.png?id=bd88b98ff5e5e1b5a4019cb0a50df9f5 500w,/images/refactoring/content/smells/duplicate-code-03-1.5x.png?id=6c3566b366c660b727083df7f491a38d 750w,/images/refactoring/content/smells/duplicate-code-03-2x.png?id=33df6a84eddb7c888f6757d4d80d5e20 1000w"
sizes="(max-width: 720px) 100vw, 500px" loading="lazy" width="500"
height="300" />
</figure>

### When to Ignore

-   In very rare cases, merging two identical fragments of code can make
    the code less intuitive and obvious.

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

[Lazy Class []{.fa .fa-arrow-right}](lazy-class.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### 戻る

[[]{.fa .fa-arrow-left} Comments](comments.html){.btn .btn-default
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
