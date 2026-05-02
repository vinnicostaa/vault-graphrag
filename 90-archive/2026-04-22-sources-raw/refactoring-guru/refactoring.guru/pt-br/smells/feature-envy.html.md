::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::: {.main-content-container .center-content-container}
::::::::: {.page .text}
::: breadcrumb
[](../../index.html){.home} / [Refactoring](../refactoring.html){.type}
/ [Code Smells](../refactoring/smells.html){.type} /
[Couplers](../refactoring/smells/couplers.html){.type}
:::

# Feature Envy {#feature-envy .title}

### Signs and Symptoms

A method accesses the data of another object more than its own data.

<figure class="image">
<img
src="../../images/refactoring/content/smells/feature-envy-010982.png?id=f520a24562e3f4b7848eca94792c329f"
style="aspect-ratio: 1.67;"
srcset="/images/refactoring/content/smells/feature-envy-01.png?id=f520a24562e3f4b7848eca94792c329f 500w,/images/refactoring/content/smells/feature-envy-01-1.5x.png?id=8f02ea7aa51cd62bef6763bcecabec9b 750w,/images/refactoring/content/smells/feature-envy-01-2x.png?id=4329322703e5af5b3ef7faefd17c4750 1000w"
sizes="(max-width: 720px) 100vw, 500px" data-fetchpriority="high"
width="500" height="300" />
</figure>

### Reasons for the Problem

This smell may occur after fields are moved to a data class. If this is
the case, you may want to move the operations on data to this class as
well.

### Treatment

As a basic rule, if things change at the same time, you should keep them
in the same place. Usually data and functions that use this data are
changed together (although exceptions are possible).

-   If a method clearly should be moved to another place, use [Move
    Method](../move-method.html).

-   If only part of a method accesses the data of another object, use
    [Extract Method](../extract-method.html) to move the part in
    question.

-   If a method uses functions from several other classes, first
    determine which class contains most of the data used. Then place the
    method in this class along with the other data. Alternatively, use
    [Extract Method](../extract-method.html) to split the method into
    several parts that can be placed in different places in different
    classes.

<figure class="image">
<img
src="../../images/refactoring/content/smells/feature-envy-025fc9.png?id=a90a3545498c7c22e605ceeb1f23d005"
style="aspect-ratio: 1.67;"
srcset="/images/refactoring/content/smells/feature-envy-02.png?id=a90a3545498c7c22e605ceeb1f23d005 500w,/images/refactoring/content/smells/feature-envy-02-1.5x.png?id=56b1c3639436656391810a4122cc488c 750w,/images/refactoring/content/smells/feature-envy-02-2x.png?id=641faecaeb0d422232c0bcc6346352c5 1000w"
sizes="(max-width: 720px) 100vw, 500px" loading="lazy" width="500"
height="300" />
</figure>

### Payoff

-   Less code duplication (if the data handling code is put in a central
    place).

-   Better code organization (methods for handling data are next to the
    actual data).

<figure class="image">
<img
src="../../images/refactoring/content/smells/feature-envy-03a90e.png?id=ea63eeab9eda1910348d0930c8592780"
style="aspect-ratio: 1.67;"
srcset="/images/refactoring/content/smells/feature-envy-03.png?id=ea63eeab9eda1910348d0930c8592780 500w,/images/refactoring/content/smells/feature-envy-03-1.5x.png?id=9556986c5ae21dd7e2367d44d28e3258 750w,/images/refactoring/content/smells/feature-envy-03-2x.png?id=d8d24af45285db63e68560788e6240bc 1000w"
sizes="(max-width: 720px) 100vw, 500px" loading="lazy" width="500"
height="300" />
</figure>

### When to Ignore

-   Sometimes behavior is purposefully kept separate from the class that
    holds the data. The usual advantage of this is the ability to
    dynamically change the behavior (see
    [Strategy](../design-patterns/strategy.html),
    [Visitor](../design-patterns/visitor.html) and other patterns).

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
#### Leia a seguir

[Inappropriate Intimacy []{.fa
.fa-arrow-right}](inappropriate-intimacy.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Voltar

[[]{.fa
.fa-arrow-left} Couplers](../refactoring/smells/couplers.html){.btn
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
