:::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::: {.page .text}
::: breadcrumb
[](../../../index.html){.home} /
[Refactoring](../../refactoring.html){.type} / [Code
Smells](../smells.html){.type}
:::

# Change Preventers {#change-preventers .title}

These smells mean that if you need to change something in one place in
your code, you have to make many changes in other places too. Program
development becomes much more complicated and expensive as a result.

:::::::::::: {.relations .link-list}
::::: dl
::: dt
[Divergent Change](../../smells/divergent-change.html)
:::

::: dd
You find yourself having to change many unrelated methods when you make
changes to a class. For example, when adding a new product type you
have to change the methods for finding, displaying, and ordering
products.
:::
:::::

::::: dl
::: dt
[Shotgun Surgery](../../smells/shotgun-surgery.html)
:::

::: dd
Making any modifications requires that you make many small changes to
many different classes.
:::
:::::

::::: dl
::: dt
[Parallel Inheritance
Hierarchies](../../smells/parallel-inheritance-hierarchies.html)
:::

::: dd
Whenever you create a subclass for a class, you find yourself needing to
create a subclass for another class.
:::
:::::
::::::::::::

::: next
#### Czytaj dalej

[Divergent Change []{.fa
.fa-arrow-right}](../../smells/divergent-change.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Wróć

[[]{.fa .fa-arrow-left} Alternative Classes with Different
Interfaces](../../smells/alternative-classes-with-different-interfaces.html){.btn
.btn-default rel="prev"}
:::
::::::::::::::::
:::::::::::::::::
::::::::::::::::::
