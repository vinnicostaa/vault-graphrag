::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::: {.main-content-container .center-content-container}
::::::::: {.page .text}
::: breadcrumb
[](../../index.html){.home} / [Refactoring](../refactoring.html){.type}
/ [Code Smells](../refactoring/smells.html){.type} / [Object-Orientation
Abusers](../refactoring/smells/oo-abusers.html){.type}
:::

# Switch Statements {#switch-statements .title}

### Signs and Symptoms

You have a complex `switch` operator or sequence of `if` statements.

<figure class="image">
<img
src="../../images/refactoring/content/smells/switch-statements-013ae1.png?id=1c5a538d451c3d2b2e43d0d4f53b994b"
style="aspect-ratio: 1.67;"
srcset="/images/refactoring/content/smells/switch-statements-01.png?id=1c5a538d451c3d2b2e43d0d4f53b994b 500w,/images/refactoring/content/smells/switch-statements-01-1.5x.png?id=77cd896e1d6bcf499db5eb5acdda01e4 750w,/images/refactoring/content/smells/switch-statements-01-2x.png?id=da7f9cad4c4bb8bf5d6910aba08680be 1000w"
sizes="(max-width: 720px) 100vw, 500px" data-fetchpriority="high"
width="500" height="300" />
</figure>

### Reasons for the Problem

Relatively rare use of `switch` and `case` operators is one of the
hallmarks of object-oriented code. Often code for a single `switch` can
be scattered in different places in the program. When a new condition is
added, you have to find all the `switch` code and modify it.

As a rule of thumb, when you see `switch` you should think of
polymorphism.

### Treatment

-   To isolate `switch` and put it in the right class, you may need
    [Extract Method](../extract-method.html) and then [Move
    Method](../move-method.html).

-   If a `switch` is based on type code, such as when the program's
    runtime mode is switched, use [Replace Type Code with
    Subclasses](../replace-type-code-with-subclasses.html) or [Replace
    Type Code with
    State/Strategy](../replace-type-code-with-state-strategy.html).

-   After specifying the inheritance structure, use [Replace Conditional
    with Polymorphism](../replace-conditional-with-polymorphism.html).

-   If there aren't too many conditions in the operator and they all
    call same method with different parameters, polymorphism will be
    superfluous. If this case, you can break that method into multiple
    smaller methods with [Replace Parameter with Explicit
    Methods](../replace-parameter-with-explicit-methods.html) and change
    the `switch` accordingly.

-   If one of the conditional options is `null`, use [Introduce Null
    Object](../introduce-null-object.html).

### Payoff

-   Improved code organization.

<figure class="image">
<img
src="../../images/refactoring/content/smells/switch-statements-029112.png?id=29ec9ad9c6d760d2b940a68a1d23c4be"
style="aspect-ratio: 1.67;"
srcset="/images/refactoring/content/smells/switch-statements-02.png?id=29ec9ad9c6d760d2b940a68a1d23c4be 500w,/images/refactoring/content/smells/switch-statements-02-1.5x.png?id=ba92702333e88eed74ee9866f9ac9d0b 750w,/images/refactoring/content/smells/switch-statements-02-2x.png?id=b7c9163ebe366c956f1aa64c4b3312e4 1000w"
sizes="(max-width: 720px) 100vw, 500px" loading="lazy" width="500"
height="300" />
</figure>

### When to Ignore

-   When a `switch` operator performs simple actions, there's no reason
    to make code changes.

-   Often `switch` operators are used by factory design patterns
    ([Factory Method](../design-patterns/factory-method.html) or
    [Abstract Factory](../design-patterns/abstract-factory.html)) to
    select a created class.

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

[Temporary Field []{.fa .fa-arrow-right}](temporary-field.html){.btn
.btn-primary rel="next"}
:::

::: prev
#### Volver

[[]{.fa .fa-arrow-left} Object-Orientation
Abusers](../refactoring/smells/oo-abusers.html){.btn .btn-default
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
