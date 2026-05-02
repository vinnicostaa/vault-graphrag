::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::::: {.page .text}
::: breadcrumb
[](../../../index.html){.home} /
[Refactoring](../../refactoring.html){.type} / [Code
Smells](../smells.html){.type}
:::

# Dispensables {#dispensables .title}

A dispensable is something pointless and unneeded whose absence would
make the code cleaner, more efficient and easier to understand.

::::::::::::::::::::: {.relations .link-list}
::::: dl
::: dt
[Comments](../../smells/comments.html)
:::

::: dd
A method is filled with explanatory comments.
:::
:::::

::::: dl
::: dt
[Duplicate Code](../../smells/duplicate-code.html)
:::

::: dd
Two code fragments look almost identical.
:::
:::::

::::: dl
::: dt
[Lazy Class](../../smells/lazy-class.html)
:::

::: dd
Understanding and maintaining classes always costs time and money. So if
a class doesn\'t do enough to earn your attention, it should be deleted.
:::
:::::

::::: dl
::: dt
[Data Class](../../smells/data-class.html)
:::

::: dd
A data class refers to a class that contains only fields and crude
methods for accessing them (getters and setters). These are simply
containers for data used by other classes. These classes don\'t contain
any additional functionality and can\'t independently operate on the
data that they own.
:::
:::::

::::: dl
::: dt
[Dead Code](../../smells/dead-code.html)
:::

::: dd
A variable, parameter, field, method or class is no longer used (usually
because it\'s obsolete).
:::
:::::

::::: dl
::: dt
[Speculative Generality](../../smells/speculative-generality.html)
:::

::: dd
There\'s an unused class, method, field or parameter.
:::
:::::
:::::::::::::::::::::

::: next
#### 다음을 보세요

[Comments []{.fa .fa-arrow-right}](../../smells/comments.html){.btn
.btn-primary rel="next"}
:::

::: prev
#### 돌아가기

[[]{.fa .fa-arrow-left} Parallel Inheritance
Hierarchies](../../smells/parallel-inheritance-hierarchies.html){.btn
.btn-default rel="prev"}
:::
:::::::::::::::::::::::::
::::::::::::::::::::::::::
:::::::::::::::::::::::::::
