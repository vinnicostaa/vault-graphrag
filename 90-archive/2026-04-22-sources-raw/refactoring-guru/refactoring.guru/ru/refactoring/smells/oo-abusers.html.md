::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::: {.page .text}
::: breadcrumb
[](../../../index.html){.home} /
[Рефакторинг](../../refactoring.html){.type} / [Запахи
кода](../smells.html){.type}
:::

# Нарушители объектно-ориентированного дизайна {#нарушители-объектно-ориентированного-дизайна .title}

Все эти запахи являют собой неполное или неправильное использование
возможностей объектно-ориентированного программирования.

::::::::::::::: {.relations .link-list}
::::: dl
::: dt
[Switch Statements](../../smells/switch-statements.html)
:::

::: dd
У вас есть сложный оператор `switch` или последовательность `if`-ов.
:::
:::::

::::: dl
::: dt
[Temporary Field](../../smells/temporary-field.html)
:::

::: dd
Временные поля --- это поля, которые нужны объекту только при
определённых обстоятельствах. Только тогда они заполняются какими-то
значениями, оставаясь пустыми в остальное время.
:::
:::::

::::: dl
::: dt
[Refused Bequest](../../smells/refused-bequest.html)
:::

::: dd
Если подкласс использует лишь малую часть унаследованных методов и
свойств суперкласса, это является признаком неправильной иерархии. При
этом ненужные методы могут просто не использоваться либо быть
переопределёнными и выбрасывать исключения.
:::
:::::

::::: dl
::: dt
[Alternative Classes with Different
Interfaces](../../smells/alternative-classes-with-different-interfaces.html)
:::

::: dd
Два класса выполняют одинаковые функции, но имеют разные названия
методов.
:::
:::::
:::::::::::::::

::: next
#### Читаем дальше

[Операторы switch []{.fa
.fa-arrow-right}](../../smells/switch-statements.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Вернуться назад

[[]{.fa .fa-arrow-left} Группы
данных](../../smells/data-clumps.html){.btn .btn-default rel="prev"}
:::
:::::::::::::::::::
::::::::::::::::::::
:::::::::::::::::::::
