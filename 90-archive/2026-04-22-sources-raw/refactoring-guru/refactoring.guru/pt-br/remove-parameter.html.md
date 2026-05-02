::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::::::::::::: {.refactoring .page .text}
::: breadcrumb
[](../index.html){.home} / [Refactoring](refactoring.html){.type} /
[Techniques](refactoring/techniques.html){.type} / [Simplifying Method
Calls](refactoring/techniques/simplifying-method-calls.html){.type}
:::

# Remove Parameter {#remove-parameter .title}

::::: before-after-container
::: before
### Problem

A parameter isn't used in the body of a method.
:::

::: after
### Solution

Remove the unused parameter.
:::
:::::

:::::::::: examples
::::::::: {.before-after .before-after-container}
::::: before
::: ba
Before
:::

::: ba-container
![Remove Parameter -
Before](../images/refactoring/diagrams/Remove%20Parameter%20-%20Before1916.png?id=c49cf53fd9ebf39daa1ec27bb897a5cf){width="171"}
:::
:::::

::::: after
::: ba
After
:::

::: ba-container
![Remove Parameter -
After](../images/refactoring/diagrams/Remove%20Parameter%20-%20After4dba.png?id=93177c2a159087d73ebef60cc8c158a3){width="171"}
:::
:::::
:::::::::
::::::::::

### Why Refactor

Every parameter in a method call forces the programmer reading it to
figure out what information is found in this parameter. And if a
parameter is entirely unused in the method body, this "noggin
scratching" is for naught.

And in any case, additional parameters are extra code that has to be
run.

Sometimes we add parameters with an eye to the future, anticipating
changes to the method for which the parameter might be needed. All the
same, experience shows that it's better to add a parameter only when
it's genuinely needed. After all, anticipated changes often remain just
that---anticipated.

### Benefits

-   A method contains only the parameters that it truly requires.

### When Not to Use

-   If the method is implemented in different ways in subclasses or in a
    superclass, and your parameter is used in those implementations,
    leave the parameter as-is.

### How to Refactor

1.  See whether the method is defined in a superclass or subclass. If
    so, is the parameter used there? If the parameter is used in one of
    these implementations, hold off on this refactoring technique.

2.  The next step is important for keeping the program functional during
    the refactoring process. Create a new method by copying the old one
    and delete the relevant parameter from it. Replace the code of the
    old method with a call to the new one.

3.  Find all references to the old method and replace them with
    references to the new method.

4.  Delete the old method. Don't perform this step if the old method is
    part of a public interface. In this case, mark the old method as
    deprecated.

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

::::::::::::::: row
::::: {.col-xs-12 .col-sm-6 .col-lg-3}
### Anti-refactoring

:::: dl
::: dt
[Add Parameter](add-parameter.html)
:::
::::
:::::

::::: {.col-xs-12 .col-sm-6 .col-lg-3}
### Similar refactorings

:::: dl
::: dt
[Rename Method](rename-method.html)
:::
::::
:::::

::::: {.col-xs-12 .col-sm-6 .col-lg-3}
### Helps other refactorings

:::: dl
::: dt
[Replace Parameter with Method
Call](replace-parameter-with-method-call.html)
:::
::::
:::::

::::: {.col-xs-12 .col-sm-6 .col-lg-3}
### Eliminates smell

:::: dl
::: dt
[Speculative Generality](smells/speculative-generality.html)
:::
::::
:::::
:::::::::::::::

::: next
#### Leia a seguir

[Separate Query from Modifier []{.fa
.fa-arrow-right}](separate-query-from-modifier.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Voltar

[[]{.fa .fa-arrow-left} Add Parameter](add-parameter.html){.btn
.btn-default rel="prev"}
:::
:::::::::::::::::::::::::::::::::

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
::::::::::::::::::::::::::::::::::::::
:::::::::::::::::::::::::::::::::::::::
