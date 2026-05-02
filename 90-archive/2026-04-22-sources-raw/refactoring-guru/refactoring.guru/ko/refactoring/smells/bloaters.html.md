:::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::: {.page .text}
::: breadcrumb
[](../../../index.html){.home} /
[Refactoring](../../refactoring.html){.type} / [Code
Smells](../smells.html){.type}
:::

# Bloaters {#bloaters .title}

Bloaters are code, methods and classes that have increased to such
gargantuan proportions that they\'re hard to work with. Usually these
smells don\'t crop up right away, rather they accumulate over time as
the program evolves (and especially when nobody makes an effort to
eradicate them).

:::::::::::::::::: {.relations .link-list}
::::: dl
::: dt
[Long Method](../../smells/long-method.html)
:::

::: dd
A method contains too many lines of code. Generally, any method longer
than ten lines should make you start asking questions.
:::
:::::

::::: dl
::: dt
[Large Class](../../smells/large-class.html)
:::

::: dd
A class contains many fields/methods/lines of code.
:::
:::::

::::: dl
::: dt
[Primitive Obsession](../../smells/primitive-obsession.html)
:::

::: dd
-   Use of primitives instead of small objects for simple tasks (such as
    currency, ranges, special strings for phone numbers, etc.)
-   Use of constants for coding information (such as a constant
    `USER_ADMIN_ROLE = 1` for referring to users with administrator
    rights.)
-   Use of string constants as field names for use in data arrays.
:::
:::::

::::: dl
::: dt
[Long Parameter List](../../smells/long-parameter-list.html)
:::

::: dd
More than three or four parameters for a method.
:::
:::::

::::: dl
::: dt
[Data Clumps](../../smells/data-clumps.html)
:::

::: dd
Sometimes different parts of the code contain identical groups of
variables (such as parameters for connecting to a database). These
clumps should be turned into their own classes.
:::
:::::
::::::::::::::::::

::: next
#### 다음을 보세요

[Long Method []{.fa
.fa-arrow-right}](../../smells/long-method.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### 돌아가기

[[]{.fa .fa-arrow-left} Code Smells](../smells.html){.btn .btn-default
rel="prev"}
:::
::::::::::::::::::::::
:::::::::::::::::::::::
::::::::::::::::::::::::
