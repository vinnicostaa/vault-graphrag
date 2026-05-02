:::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::: {.refactoring .page .text}
::: breadcrumb
[](../index.html){.home} / [Refactoring](refactoring.html){.type} /
[Techniques](refactoring/techniques.html){.type} / [Simplifying
Conditional
Expressions](refactoring/techniques/simplifying-conditional-expressions.html){.type}
:::

# Remove Control Flag {#remove-control-flag .title}

::::: before-after-container
::: before
### Problem

You have a boolean variable that acts as a control flag for multiple
boolean expressions.
:::

::: after
### Solution

Instead of the variable, use `break`, `continue` and `return`.
:::
:::::

### Why Refactor

Control flags date back to the days of yore, when "proper" programmers
always had one entry point for their functions (the function declaration
line) and one exit point (at the very end of the function).

In modern programming languages this style tic is obsolete, since we
have special operators for modifying the control flow in loops and other
complex constructions:

-   `break`: stops loop

-   `continue`: stops execution of the current loop branch and goes to
    check the loop conditions in the next iteration

-   `return`: stops execution of the entire function and returns its
    result if given in the operator

### Benefits

-   Control flag code is often much more ponderous than code written
    with control flow operators.

### How to Refactor

1.  Find the value assignment to the control flag that causes the exit
    from the loop or current iteration.

2.  Replace it with `break`, if this is an exit from a loop;
    `continue`, if this is an exit from an iteration, or `return`, if
    you need to return this value from the function.

3.  Remove the remaining code and checks associated with the control
    flag.

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
#### Czytaj dalej

[Replace Nested Conditional with Guard Clauses []{.fa
.fa-arrow-right}](replace-nested-conditional-with-guard-clauses.html){.btn
.btn-primary rel="next"}
:::

::: prev
#### Wróć

[[]{.fa .fa-arrow-left} Consolidate Duplicate Conditional
Fragments](consolidate-duplicate-conditional-fragments.html){.btn
.btn-default rel="prev"}
:::
::::::::::::

:::::: {.prom .banner-sidebar .banner-removable .banner-removable-refactoring data-id="Ref: Sidebar" creative-id="standard-sidebar-en" position="sidebar"}
::::: ban
::: {.image .product-image}
[Sale!]{.banner-discount}
[![](../images/refactoring/course/course-cover-en1b63.jpg?id=ac6ad836a27c4a8d579244f15c48a8b4){width="300"
height="300" loading="lazy"}](../refactoring/course.html)
:::

::: {.banner-text .banner-text-en}
This refactoring is part of the much bigger **Refactoring Course**.

[ Learn more...](../refactoring/course.html){.btn .btn-secondary
.btn-block}
:::
:::::
::::::
:::::::::::::::::
::::::::::::::::::
