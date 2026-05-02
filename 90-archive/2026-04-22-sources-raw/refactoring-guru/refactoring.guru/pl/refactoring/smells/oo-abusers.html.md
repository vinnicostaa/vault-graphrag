::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::: {.page .text}
::: breadcrumb
[](../../../index.html){.home} /
[Refactoring](../../refactoring.html){.type} / [Code
Smells](../smells.html){.type}
:::

# Object-Orientation Abusers {#object-orientation-abusers .title}

All these smells are incomplete or incorrect application of
object-oriented programming principles.

::::::::::::::: {.relations .link-list}
::::: dl
::: dt
[Switch Statements](../../smells/switch-statements.html)
:::

::: dd
You have a complex `switch` operator or sequence of `if` statements.
:::
:::::

::::: dl
::: dt
[Temporary Field](../../smells/temporary-field.html)
:::

::: dd
Temporary fields get their values (and thus are needed by objects) only
under certain circumstances. Outside of these circumstances, they're
empty.
:::
:::::

::::: dl
::: dt
[Refused Bequest](../../smells/refused-bequest.html)
:::

::: dd
If a subclass uses only some of the methods and properties inherited
from its parents, the hierarchy is off-kilter. The unneeded methods may
simply go unused or be redefined and give off exceptions.
:::
:::::

::::: dl
::: dt
[Alternative Classes with Different
Interfaces](../../smells/alternative-classes-with-different-interfaces.html)
:::

::: dd
Two classes perform identical functions but have different method names.
:::
:::::
:::::::::::::::

::: next
#### Czytaj dalej

[Switch Statements []{.fa
.fa-arrow-right}](../../smells/switch-statements.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Wróć

[[]{.fa .fa-arrow-left} Data Clumps](../../smells/data-clumps.html){.btn
.btn-default rel="prev"}
:::
:::::::::::::::::::
::::::::::::::::::::
:::::::::::::::::::::
